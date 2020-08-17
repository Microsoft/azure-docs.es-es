---
title: Migrar a Administrador de recursos con PowerShell
description: Este artículo lo guía a través de la migración de recursos de IaaS compatible con la plataforma de recursos, como máquinas virtuales (VM), redes virtuales y cuentas de almacenamiento del modelo clásico al de Azure Resource Manager mediante comandos de Azure PowerShell.
author: tanmaygore
manager: vashan
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 02/06/2020
ms.author: tagore
ms.openlocfilehash: 026b869556e1bd49018b2afce27e732a3109b9fd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "86999135"
---
# <a name="migrate-iaas-resources-from-classic-to-azure-resource-manager-by-using-powershell"></a>Migración de recursos de IaaS del modelo clásico al de Azure Resource Manager mediante PowerShell

> [!IMPORTANT]
> En la actualidad, aproximadamente el 90 % de las máquinas virtuales de IaaS usan [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). A partir del 28 de febrero de 2020, las máquinas virtuales clásicas han quedado en desuso y se van a retirar por completo el 1 de marzo de 2023. [Obtenga más información]( https://aka.ms/classicvmretirement) sobre esta caída en desuso y [cómo se va a ver afectado](../classic-vm-deprecation.md#how-does-this-affect-me).

En estos pasos se describe cómo utilizar los comandos de Azure PowerShell para migrar los recursos de infraestructura como servicio (IaaS) desde el modelo de implementación clásica al modelo de implementación de Azure Resource Manager.

Si se quiere, también se pueden migrar recursos mediante la [CLI de Azure](../linux/migration-classic-resource-manager-cli.md).

* Lea el artículo [Migración compatible con la plataforma de recursos de IaaS del modelo clásico a Azure Resource Manager](migration-classic-resource-manager-overview.md)para obtener información sobre los escenarios de migración que se admiten.
* Si quiere obtener instrucciones detalladas y ver un tutorial de migración, consulte [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager](migration-classic-resource-manager-deep-dive.md).
* [Revisión de los errores más comunes en la migración](migration-classic-resource-manager-errors.md).

<br>
Este es un diagrama de flujo para identificar el orden en que tienen que ejecutarse los pasos durante un proceso de migración.

![Screenshot that shows the migration steps](media/migration-classic-resource-manager/migration-flow.png)

 

## <a name="step-1-plan-for-migration"></a>Paso 1: Planear la migración
Estos son algunos de los procedimientos recomendados a la hora de evaluar si se migran recursos de IaaS desde el modelo clásico a Resource Manager:

* Lea el artículo sobre [qué características y configuraciones se admiten y cuáles no](migration-classic-resource-manager-overview.md). Si tiene máquinas virtuales que usan configuraciones o características no admitidas, espere a que se anuncie dicha compatibilidad. De manera alternativa, si se ajusta a sus necesidades, quite esa característica o salga de esa configuración para habilitar la migración.
* Si tiene actualmente scripts automatizados que implementan la infraestructura y las aplicaciones, intente crear una configuración de prueba similar usando esos scripts para la migración. También puede configurar entornos de ejemplo mediante el Portal de Azure.

> [!IMPORTANT]
> Las puertas de enlace de aplicaciones no se admiten actualmente para realizar migraciones del modelo clásico al de Resource Manager. Si se quiere migrar una red virtual con una instancia de Application Gateway, quite la puerta de enlace antes de ejecutar una operación de preparación para mover la red. Después, cuando termine el proceso de migración, vuelva a conectar la puerta de enlace en Azure Resource Manager.
>
> Las puertas de enlace de Azure ExpressRoute que se conectan con circuitos de ExpressRoute en otra suscripción no se pueden migrar automáticamente. En esos casos, quite la puerta de enlace de ExpressRoute, migre la red virtual y vuelva a crear la puerta de enlace. Para obtener más información, vea [Migración de circuitos de ExpressRoute y las redes virtuales asociadas del modelo de implementación clásica al de Resource Manager](../../expressroute/expressroute-migration-classic-resource-manager.md).

## <a name="step-2-install-the-latest-version-of-powershell"></a>Paso 2: Instalar la versión más reciente de Azure PowerShell
Hay dos opciones principales para instalar Azure PowerShell: [Galería de PowerShell](https://www.powershellgallery.com/profiles/azure-sdk/) o [Instalador de plataforma web (WebPI)](https://aka.ms/webpi-azps). WebPI recibe actualizaciones mensuales. La galería de PowerShell recibe actualizaciones de forma continua. Este artículo se basa en los cmdlets de la versión 2.1.0 de Azure PowerShell.

Para ver las instrucciones de instalación, consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/).

## <a name="step-3-ensure-that-youre-an-administrator-for-the-subscription"></a>Paso 3: Asegurarse de que es administrador de la suscripción
Para realizar esta migración, debe estar agregado como coadministrador de la suscripción en [Azure Portal](https://portal.azure.com).

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el menú de **Central**, seleccione **Suscripción**. Si no lo ve, haga clic en **Todos los servicios**.
3. Busque la entrada de la suscripción adecuada y después examine el campo **MI ROL**. En el caso de un coadministrador, el valor debe ser _Administrador de cuenta_.

Si no puede agregar un coadministrador, póngase en contacto con un administrador del servicio o coadministrador de la suscripción para que le agreguen.

## <a name="step-4-set-your-subscription-and-sign-up-for-migration"></a>Paso 4: Establecer la suscripción y el registro para la migración
En primer lugar, inicie un símbolo del sistema de PowerShell. Para la migración, configure el entorno para el modelo clásico y el de Resource Manager.

Inicie sesión en su cuenta para el modelo de Resource Manager.

```powershell
    Connect-AzAccount
```

Puede encontrar las suscripciones disponibles ejecutando el siguiente comando:

```powershell
    Get-AzSubscription | Sort Name | Select Name
```

Establezca la suscripción de Azure para la sesión actual. En este ejemplo se establece el nombre de la suscripción predeterminado en **My Azure Subscription** (Mi suscripción de Azure). Reemplace el nombre de la suscripción de ejemplo por el suyo propio.

```powershell
    Select-AzSubscription –SubscriptionName "My Azure Subscription"
```

> [!NOTE]
> El registro es un paso que solo se realiza una vez, pero debe hacerlo antes de intentar la migración. Si no se registra, recibirá el siguiente mensaje de error:
>
> *BadRequest: La suscripción no está registrada para la migración.*

Regístrese con el proveedor de recursos de migración ejecutando el comando siguiente:

```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

Espere cinco minutos a que finalice el registro. Compruebe el estado de la aprobación con el comando siguiente:

```powershell
    Get-AzResourceProvider -ProviderNamespace Microsoft.ClassicInfrastructureMigrate
```

Asegúrese de que RegistrationState sea `Registered` antes de continuar.

Antes de cambiar al modelo de implementación clásico, asegúrese de tener suficientes unidades vCPU de máquina virtual de Azure Resource Manager en la región de Azure de su red virtual o implementación actual. Puede usar el siguiente comando de PowerShell para comprobar la cantidad de vCPU que tiene actualmente en Azure Resource Manager. Para obtener más información sobre las cuotas de vCPU, vea [Límites y Azure Resource Manager](../../azure-resource-manager/management/azure-subscription-service-limits.md#managing-limits).

En este ejemplo se comprueba la disponibilidad en la región del **oeste de EE. UU.** Reemplace el nombre de la región de ejemplo por el suyo propio.

```powershell
    Get-AzVMUsage -Location "West US"
```

Ahora inicie sesión en su cuenta para el modelo de implementación clásico.

```powershell
    Add-AzureAccount
```

Puede encontrar las suscripciones disponibles ejecutando el siguiente comando:

```powershell
    Get-AzureSubscription | Sort SubscriptionName | Select SubscriptionName
```

Establezca la suscripción de Azure para la sesión actual. En este ejemplo se establece la suscripción predeterminada en **My Azure Subscription** (Mi suscripción de Azure). Reemplace el nombre de la suscripción de ejemplo por el suyo propio.

```powershell
    Select-AzureSubscription –SubscriptionName "My Azure Subscription"
```


## <a name="step-5-run-commands-to-migrate-your-iaas-resources"></a>Paso 5: Ejecutar comandos para migrar los recursos de IaaS
* [Migración de máquinas virtuales en un servicio en la nube (no en una red virtual)](#step-51-option-1---migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network)
* [Migración de máquinas virtuales en una red virtual](#step-51-option-2---migrate-virtual-machines-in-a-virtual-network)
* [Migración de una cuenta de almacenamiento](#step-52-migrate-a-storage-account)

> [!NOTE]
> Todas las operaciones que se describen aquí son idempotentes. Si tiene un problema diferente de una función no admitida o un error de configuración, se recomienda que vuelva a intentar la operación de preparación, anulación o confirmación. La plataforma intenta nuevamente la acción.


### <a name="step-51-option-1---migrate-virtual-machines-in-a-cloud-service-not-in-a-virtual-network"></a>Paso 5.1: Opción 1: Migrar máquinas virtuales en un servicio en la nube (no en una red virtual)
Obtenga la lista de servicios en la nube con el comando siguiente. Después, seleccione el servicio en la nube que se quiere migrar. Si las máquinas virtuales del servicio en la nube están en una red virtual o si tienen roles web o de trabajo, el comando devolverá un mensaje de error.

```powershell
    Get-AzureService | ft Servicename
```

Obtenga el nombre de la implementación del servicio en la nube. En este ejemplo, el nombre de servicio es **My Service** (Mi servicio). Reemplace el nombre del servicio de ejemplo por el suyo propio.

```powershell
    $serviceName = "My Service"
    $deployment = Get-AzureDeployment -ServiceName $serviceName
    $deploymentName = $deployment.DeploymentName
```

Prepare las máquinas virtuales del servicio en la nube para la migración. Tiene dos opciones para elegir.

* **Opción 1: Migrar las máquinas virtuales a una red virtual creada en una plataforma**

    Primero, valide que puede migrar el servicio en la nube con los comandos siguientes:

    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    $validate.ValidationMessages
    ```

    El comando siguiente muestra cualquier advertencia y error que bloquee la migración. Si la validación se realiza correctamente, podrá pasar al paso de preparación siguiente.

    ```powershell
    Move-AzureService -Prepare -ServiceName $serviceName `
        -DeploymentName $deploymentName -CreateNewVirtualNetwork
    ```
* **Opción 2: Migrar a una red virtual existente en el modelo de implementación de Resource Manager**

    En este ejemplo se establece el nombre del grupo de recursos en **myResourceGroup**, el nombre de red virtual en **myVirtualNetwork** y el nombre de la subred en **mySubNet**. Reemplace los nombres del ejemplo por los nombres de sus propios recursos.

    ```powershell
    $existingVnetRGName = "myResourceGroup"
    $vnetName = "myVirtualNetwork"
    $subnetName = "mySubNet"
    ```

    En primer lugar, valide que puede migrar la red virtual con el comando siguiente:

    ```powershell
    $validate = Move-AzureService -Validate -ServiceName $serviceName `
        -DeploymentName $deploymentName -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName -VirtualNetworkName $vnetName -SubnetName $subnetName
    $validate.ValidationMessages
    ```

    El comando siguiente muestra cualquier advertencia y error que bloquee la migración. Si la validación se realiza correctamente, podrá continuar al paso de preparación siguiente:

    ```powershell
        Move-AzureService -Prepare -ServiceName $serviceName -DeploymentName $deploymentName `
        -UseExistingVirtualNetwork -VirtualNetworkResourceGroupName $existingVnetRGName `
        -VirtualNetworkName $vnetName -SubnetName $subnetName
    ```

Cuando la operación de preparación finalice correctamente con cualquiera de las opciones anteriores, consulte el estado de la migración de las máquinas virtuales. Asegúrese de que tienen el estado `Prepared`.

En este ejemplo se establece el nombre de la máquina virtual en **myVM**. Reemplace el nombre del ejemplo por su propio nombre de la máquina virtual.

```powershell
    $vmName = "myVM"
    $vm = Get-AzureVM -ServiceName $serviceName -Name $vmName
    $vm.VM.MigrationState
```

Compruebe la configuración de los recursos preparados mediante PowerShell o el Portal de Azure. Si no está preparado para la migración y quiere volver al estado anterior, utilice el comando siguiente:

```powershell
    Move-AzureService -Abort -ServiceName $serviceName -DeploymentName $deploymentName
```

Si la configuración preparada parece correcta, puede continuar y confirmar los recursos mediante el siguiente comando:

```powershell
    Move-AzureService -Commit -ServiceName $serviceName -DeploymentName $deploymentName
```

### <a name="step-51-option-2---migrate-virtual-machines-in-a-virtual-network"></a>Paso 5.1: Opción 2: Migrar máquinas virtuales en una red virtual

Para migrar máquinas virtuales de una red virtual, migre la red virtual. Las máquinas virtuales se migran automáticamente con la red virtual. Seleccione la red virtual que quiere migrar.
> [!NOTE]
> [Migre una sola máquina virtual](./create-vm-specialized-portal.md) creada mediante el modelo de implementación clásico creando una nueva máquina virtual de Resource Manager con Managed Disks mediante los archivos de VHD (SO y datos) de la máquina virtual.
<br>

> [!NOTE]
> El nombre de red virtual podría ser diferente del que se muestra en el portal nuevo. El nuevo Azure Portal muestra el nombre como `[vnet-name]` pero el nombre de red virtual real es de tipo `Group [resource-group-name] [vnet-name]`. Antes de empezar la migración, busque el nombre real de la red virtual con el comando `Get-AzureVnetSite | Select -Property Name` o consúltelo en el antiguo Azure Portal. 

En este ejemplo se establece el nombre de red virtual en **myVnet**. Reemplace el nombre de la red virtual de ejemplo por el suyo propio.

```powershell
    $vnetName = "myVnet"
```

> [!NOTE]
> Si la red virtual contiene roles web o de trabajo, o bien máquinas virtuales con configuraciones no admitidas, recibe un mensaje de error de validación.

En primer lugar, valide que puede migrar la red virtual con el comando siguiente:

```powershell
    Move-AzureVirtualNetwork -Validate -VirtualNetworkName $vnetName
```

El comando siguiente muestra cualquier advertencia y error que bloquee la migración. Si la validación se realiza correctamente, podrá continuar al paso de preparación siguiente:

```powershell
    Move-AzureVirtualNetwork -Prepare -VirtualNetworkName $vnetName
```

Compruebe la configuración de las máquinas virtuales preparadas mediante Azure PowerShell o Azure Portal. Si no está preparado para la migración y quiere volver al estado anterior, utilice el comando siguiente:

```powershell
    Move-AzureVirtualNetwork -Abort -VirtualNetworkName $vnetName
```

Si la configuración preparada parece correcta, puede continuar y confirmar los recursos mediante el siguiente comando:

```powershell
    Move-AzureVirtualNetwork -Commit -VirtualNetworkName $vnetName
```

### <a name="step-52-migrate-a-storage-account"></a>Paso 5.2: Migración de una cuenta de almacenamiento
Cuando haya terminado de migrar las máquinas virtuales, realice las comprobaciones siguientes de requisitos previos antes de migrar las cuentas de almacenamiento.

> [!NOTE]
> Si su cuenta de almacenamiento no tiene discos asociados ni datos de máquina virtual, puede ir directamente a la sección "Validación de las cuentas de almacenamiento y comienzo de la migración".

* Comprobación de requisitos previos si ha migrado cualquier máquina virtual o la cuenta de almacenamiento tiene recursos de disco:
    * Migración de máquinas virtuales cuyos discos se almacenan en la cuenta de almacenamiento.

        El comando siguiente devuelve las propiedades RoleName y DiskName de todos los discos de máquinas virtuales en la cuenta de almacenamiento. RoleName es el nombre de la máquina virtual a la que está conectado el disco. Si este comando devuelve discos, asegúrese de que las máquinas virtuales a las que están conectados estos discos se han migrado antes de migrar la cuenta de almacenamiento.
        ```powershell
         $storageAccountName = 'yourStorageAccountName'
          Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Select-Object -ExpandProperty AttachedTo -Property `
          DiskName | Format-List -Property RoleName, DiskName

        ```
    * Eliminación de discos de máquinas virtuales sin adjuntar almacenados en la cuenta de almacenamiento.

        Busque discos de máquinas virtuales sin adjuntar en la cuenta de almacenamiento con el comando siguiente:

        ```powershell
            $storageAccountName = 'yourStorageAccountName'
            Get-AzureDisk | where-Object {$_.MediaLink.Host.Contains($storageAccountName)} | Where-Object -Property AttachedTo -EQ $null | Format-List -Property DiskName  

        ```
        Si el comando anterior devuelve discos, elimínelos con el comando siguiente:

        ```powershell
           Remove-AzureDisk -DiskName 'yourDiskName'
        ```
    * Eliminación de imágenes de máquinas virtuales almacenadas en la cuenta de almacenamiento.

        El comando siguiente devuelve todas las imágenes de máquinas virtuales con los discos de SO almacenados en la cuenta de almacenamiento.
         ```powershell
            Get-AzureVmImage | Where-Object { $_.OSDiskConfiguration.MediaLink -ne $null -and $_.OSDiskConfiguration.MediaLink.Host.Contains($storageAccountName)`
                                    } | Select-Object -Property ImageName, ImageLabel
         ```
         El comando siguiente devuelve todas las imágenes de máquinas virtuales con discos de datos almacenados en la cuenta de almacenamiento.
         ```powershell

            Get-AzureVmImage | Where-Object {$_.DataDiskConfigurations -ne $null `
                                             -and ($_.DataDiskConfigurations | Where-Object {$_.MediaLink -ne $null -and $_.MediaLink.Host.Contains($storageAccountName)}).Count -gt 0 `
                                            } | Select-Object -Property ImageName, ImageLabel
         ```
        Elimine todas las imágenes de máquinas virtuales que se han devuelto usando este comando:
        ```powershell
        Remove-AzureVMImage -ImageName 'yourImageName'
        ```
* Validación de las cuentas de almacenamiento y comienzo de la migración.

    Valide cada cuenta de almacenamiento para la migración mediante el siguiente comando. En este ejemplo, el nombre de cuenta de almacenamiento es **myStorageAccount**. Reemplace el nombre de ejemplo por el nombre de su propia cuenta de almacenamiento.

    ```powershell
        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Validate -StorageAccountName $storageAccountName
    ```

    El paso siguiente consiste en preparar la cuenta de almacenamiento para la migración.

    ```powershell
        $storageAccountName = "myStorageAccount"
        Move-AzureStorageAccount -Prepare -StorageAccountName $storageAccountName
    ```

    Compruebe la configuración de la cuenta de almacenamiento preparada mediante Azure PowerShell o Azure Portal. Si no está preparado para la migración y quiere volver al estado anterior, utilice el comando siguiente:

    ```powershell
        Move-AzureStorageAccount -Abort -StorageAccountName $storageAccountName
    ```

    Si la configuración preparada parece correcta, puede continuar y confirmar los recursos mediante el siguiente comando:

    ```powershell
        Move-AzureStorageAccount -Commit -StorageAccountName $storageAccountName
    ```

## <a name="next-steps"></a>Pasos siguientes
* [Información general sobre la migración compatible con la plataforma de recursos de IaaS desde el modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager](migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-plan.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager)
* [Migración de recursos de IaaS de la implementación clásica a Azure Resource Manager con la CLI de Azure](../linux/migration-classic-resource-manager-cli.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Herramientas de la comunidad para ayudar con la migración de recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de los errores más comunes en la migración](migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de las preguntas más frecuentes acerca de cómo migrar recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-faq.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
