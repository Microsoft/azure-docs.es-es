---
title: Registro en el proveedor de recursos de máquina virtual con SQL
description: Registre la máquina virtual de Azure SQL Server con el proveedor de recursos de máquina virtual con SQL para habilitar características para máquinas virtuales con SQL Server implementadas fuera de Azure Marketplace, además de para garantizar el cumplimiento y mejorar la administración.
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 11/13/2019
ms.author: mathoma
ms.reviewer: jroth
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 11e8a2fd709b40c68b90e5ed139f18997e4cb29e
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89396989"
---
# <a name="register-a-sql-server-vm-in-azure-with-the-sql-vm-resource-provider-rp"></a>Registro de una VM con SQL Server con el proveedor de recursos (RP) de VM con SQL Server
[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se describe cómo registrar una máquina virtual con SQL Server en Azure con el proveedor de recursos de máquina virtual con SQL. Al registrarse con el proveedor de recursos, se crea el **recurso** de _la máquina virtual SQL_ dentro de la suscripción, que es un recurso independiente del recurso de máquina virtual. Al anular el registro de la máquina virtual SQL Server del proveedor de recursos, se eliminará el **recurso** de la _máquina virtual SQL_ pero no se eliminará la máquina virtual real. 

La implementación de una imagen de Azure Marketplace de una máquina virtual con SQL Server mediante Azure Portal registra la máquina virtual con SQL Server con el proveedor de recursos. Sin embargo, si elige instalar automáticamente SQL Server en una máquina virtual de Azure, o aprovisionar una máquina virtual de Azure desde un disco duro virtual personalizado, debe registrar la VM con SQL Server con el proveedor de recursos para:

- **Ventajas de las características**: al registrar una máquina virtual con SQL Server con el proveedor de recursos, se desbloquean la [aplicación automática de revisiones](automated-patching.md), la [copia de seguridad automática](automated-backup.md) y las funcionalidades de supervisión y administración. También se desbloquean las [licencias](licensing-model-azure-hybrid-benefit-ahb-change.md) y la flexibilidad de [ediciones](change-sql-server-edition.md). Anteriormente, estas características solo estaban disponibles para las imágenes de máquina virtual con SQL Server implementadas desde Azure Marketplace. 

- **Cumplimiento**: el registro en el proveedor de recursos de máquina virtual con SQL ofrece un método simplificado para satisfacer el requisito de notificar a Microsoft que se ha habilitado Ventaja híbrida de Azure, según se especifica en los términos del producto. Con este proceso no es necesario administrar formularios de registro de licencias para cada recurso.  

- **Administración gratuita**: el registro en el proveedor de recursos de máquina virtual con SQL en los tres modos de administración es completamente gratuito. No hay costos adicionales asociados al proveedor de recursos o al cambio de modo de administración. 

- **Administración de licencias simplificada**: El registro en el proveedor de recursos de VM con SQL simplifica la administración de licencias de SQL Server y permite identificar rápidamente VM con SQL Server mediante Ventaja híbrida de Azure en [Azure portal](manage-sql-vm-portal.md), la CLI de Azure o PowerShell: 

   # <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

   ```azurecli-interactive
   $vms = az sql vm list | ConvertFrom-Json
   $vms | Where-Object {$_.sqlServerLicenseType -eq "AHUB"}
   ```

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```powershell-interactive
   Get-AzSqlVM | Where-Object {$_.LicenseType -eq 'AHUB'}
   ```

   ---

Para utilizar el proveedor de recursos de máquina virtual con SQL, primero debe [registrar la suscripción con el proveedor de recursos](#register-subscription-with-rp), lo que proporciona al proveedor de recursos la capacidad de crear recursos dentro de esa suscripción específica.

## <a name="prerequisites"></a>Requisitos previos

Para registrar una máquina virtual con SQL Server con el proveedor de recursos, necesita lo siguiente: 

- Una [suscripción de Azure](https://azure.microsoft.com/free/).
- Un modelo de recursos de Azure [máquina virtual SQL Server ](create-sql-vm-portal.md) implementado en la nube pública o de Azure Government. 
- La versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli) o [PowerShell](/powershell/azure/new-azureps-module-az). 

## <a name="management-modes"></a>Modos de administración

Si aún no se ha instalado la [extensión IaaS de SQL](sql-server-iaas-agent-extension-automate-management.md), el registro con el proveedor de recursos de máquina virtual con SQL instala automáticamente la extensión IaaS de SQL Server en uno de los tres modos de administración, especificado durante el proceso de registro. Si no se especifica el modo de administración, la extensión IaaS de SQL se instalará en el modo de administración completa.  

Si la extensión IaaS de SQL ya se ha instalado manualmente, ya estará en el modo de administración completa y el registro con el proveedor de recursos en modo completo no reiniciará el servicio SQL Server.

Los tres modos de administración son los siguientes:

- El modo **Lightweight** no requiere reiniciar SQL Server, pero solo permite cambiar la edición y el tipo de licencia de SQL Server. Use esta opción para máquinas virtuales con SQL Server con varias instancias o para participar en una instancia de clúster de conmutación por error (FCI). Si se usa el modo ligero, no se produce ningún impacto en la memoria ni en la CPU, ni conlleva costos asociados. Se recomienda registrar la máquina virtual con SQL Server en el modo ligero primero y, posteriormente, actualizar al modo completo durante una ventana de mantenimiento programado.  

- El modo **Full** ofrece todas las funcionalidades, pero requiere reiniciar los permisos de administrador del sistema y SQL Server. Esta es la opción que se instala de forma predeterminada al instalar la extensión IaaS de SQL manualmente. Úselo para administrar una VM con SQL Server con una sola instancia. El modo completo instala dos servicios de Windows que tienen un impacto mínimo en la memoria y la CPU (se pueden supervisar mediante el administrador de tareas). No hay ningún costo asociado al uso del modo de administración completa. 

- El modo **NoAgent** está dedicado a SQL Server 2008 y SQL Server 2008 R2 instalados en Windows Server 2008. Si se usa el modo NoAgent, ni la memoria ni la CPU resultan afectadas. No hay ningún costo asociado al uso del modo de administración NoAgent. 

Puede ver el modo actual del Agente de IaaS de SQL Server mediante PowerShell: 

  ```powershell-interactive
  # Get the SqlVirtualMachine
  $sqlvm = Get-AzSqlVM -Name $vm.Name  -ResourceGroupName $vm.ResourceGroupName
  $sqlvm.SqlManagementType
  ```

## <a name="register-subscription-with-rp"></a>Registro de la suscripción con el proveedor de recursos

Para registrar la máquina virtual de SQL Server con el proveedor de recursos de la máquina virtual de SQL, debe registrar primero su suscripción con el proveedor de recursos. Esto permite al proveedor de recursos de la máquina virtual con SQL crear recursos en la suscripción.  Para ello, puede usar Azure Portal, PowerShell o la CLI de Azure.

### <a name="azure-portal"></a>Azure portal

1. Abra Azure Portal y vaya a **Todos los servicios**. 
1. Vaya a **Suscripciones** y seleccione la suscripción que le interese.  
1. En la página **Suscripciones**, vaya a **Proveedores de recursos**. 
1. Escriba **sql** en el filtro para mostrar los proveedores de recursos relacionados con SQL. 
1. Seleccione **Registrar**, **Volver a registrar** o **Anular registro** para el proveedor **Microsoft.SqlVirtualMachine**, según la acción que quiera realizar. 

   ![Modificación del proveedor](./media/sql-vm-resource-provider-register/select-resource-provider-sql.png)


### <a name="command-line"></a>Línea de comandos

Registre el proveedor de recursos de VM con SQL en su suscripción a Azure mediante la CLI de Azure o PowerShell. 

# <a name="azure-cli"></a>[CLI de Azure](#tab/bash)

```azurecli-interactive
# Register the SQL VM resource provider to your subscription 
az provider register --namespace Microsoft.SqlVirtualMachine 
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell-interactive
# Register the SQL VM resource provider to your subscription
Register-AzResourceProvider -ProviderNamespace Microsoft.SqlVirtualMachine
```

---

## <a name="register-with-rp"></a>Registro con proveedor de recursos

### <a name="lightweight-management-mode"></a>Modo de administración ligera

Si la [extensión Agente de IaaS de SQL Server](sql-server-iaas-agent-extension-automate-management.md) no se ha instalado en la máquina virtual, se recomienda registrarse con el proveedor de recursos de máquina virtual con SQL en el modo de administración ligero. Esto instalará la extensión IaaS de SQL en [modo ligero](#management-modes) y evitará que se reinicie el servicio SQL Server. Después, puede actualizar al modo completo en cualquier momento, pero si lo hace, se reiniciará el servicio SQL Server, por lo que se recomienda esperar a una ventana de mantenimiento programado. 

Proporcione un tipo de licencia de SQL Server como pago por uso (`PAYG`) para pagar por uso, Ventaja híbrida de Azure (`AHUB`) para usar su propia licencia o recuperación ante desastres (`DR`) para activar [la licencia de réplica de recuperación ante desastres gratuita](business-continuity-high-availability-disaster-recovery-hadr-overview.md#free-dr-replica-in-azure).

Tanto las instancias del clúster de conmutación por error como las implementaciones de instancias múltiples solo se pueden registrar en el proveedor de recursos de máquina virtual con SQL en el modo ligero. 

# <a name="azure-cli"></a>[CLI de Azure](#tab/bash)

Registre una VM con SQL Server en modo ligero con la CLI de Azure: 

  ```azurecli-interactive
  # Register Enterprise or Standard self-installed VM in Lightweight mode
  az sql vm create --name <vm_name> --resource-group <resource_group_name> --location <vm_location> --license-type PAYG 
  ```


# <a name="powershell"></a>[PowerShell](#tab/powershell)

Registre una máquina virtual con SQL Server en modo ligero con PowerShell:  


  ```powershell-interactive
  # Get the existing compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
  # Register SQL VM with 'Lightweight' SQL IaaS agent
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
    -LicenseType PAYG -SqlManagementType LightWeight  
  ```

---

### <a name="full-management-mode"></a>Modo de administración completa


Si la extensión IaaS de SQL ya se ha instalado en la máquina virtual manualmente, puede registrar la máquina virtual con SQL Server en modo completo sin reiniciar el servicio SQL Server. **Sin embargo, si no se ha instalado la extensión IaaS de SQL, al registrarse en modo completo se instalará la extensión IaaS de SQL en modo completo y se reiniciará el servicio SQL Server. Proceda con precaución.**


Para registrar la VM de SQL Server directamente en modo completo (y posiblemente reiniciar el servicio de SQL Server), use el siguiente comando de PowerShell: 

  ```powershell-interactive
  # Get the existing  Compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
        
  # Register with SQL VM resource provider in full mode
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -SqlManagementType Full
  ```

### <a name="noagent-management-mode"></a>Modo de administración NoAgent 

SQL Server 2008 y 2008 R2 instalados en Windows Server 2008 (_no R2_) se pueden registrar con el proveedor de recursos de VM con SQL en [modo NoAgent](#management-modes). Esta opción garantiza el cumplimiento de normas y permite que la VM con SQL Server se supervise en Azure Portal con una funcionalidad limitada.

Especifique `AHUB`, `PAYG` o `DR` como valores de **sqlLicenseType** y `SQL2008-WS2008` o `SQL2008R2-WS2008` como valores de **sqlImageOffer**. 

Para registrar SQL Server 2008 o 2008 R2 en una instancia de Windows Server 2008, use el siguiente fragmento de código de la CLI de Azure o PowerShell: 


# <a name="azure-cli"></a>[CLI de Azure](#tab/bash)

Registre la máquina virtual con SQL Server 2008 en modo NoAgent con la CLI de Azure: 

  ```azurecli-interactive
   az sql vm create -n sqlvm -g myresourcegroup -l eastus |
   --license-type PAYG --sql-mgmt-type NoAgent 
   --image-sku Enterprise --image-offer SQL2008-WS2008
 ```
 
 
Registre la máquina virtual con SQL Server 2008 R2 en modo NoAgent con la CLI de Azure: 

  ```azurecli-interactive
   az sql vm create -n sqlvm -g myresourcegroup -l eastus |
   --license-type PAYG --sql-mgmt-type NoAgent 
   --image-sku Enterprise --image-offer SQL2008R2-WS2008
 ```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Registre la máquina virtual con SQL Server 2008 en modo NoAgent con PowerShell: 


  ```powershell-interactive
  # Get the existing compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
    -LicenseType PAYG -SqlManagementType NoAgent -Sku Standard -Offer SQL2008-WS2008
  ```
  
  Registre la máquina virtual con SQL Server 2008 R2 en modo NoAgent con PowerShell: 


  ```powershell-interactive
  # Get the existing compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
          
  New-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -Location $vm.Location `
    -LicenseType PAYG -SqlManagementType NoAgent -Sku Standard -Offer SQL2008R2-WS2008
  ```

---

## <a name="upgrade-to-full"></a>Actualizar a completo  

Las VM con SQL Server que tienen instalada la extensión IaaS *ligera* pueden actualizarse al modo _completo_ mediante Azure Portal, la CLI de Azure o PowerShell. Las máquinas virtuales con SQL Server en el modo _NoAgent_ pueden actualizar al modo _completo_ tras actualizar el sistema operativo a Windows 2008 R2 y versiones posteriores. No es posible cambiar a un modo inferior: para ello, deberá [anular el registro](#unregister-from-rp) de la máquina virtual con SQL Server del proveedor de recursos de máquina virtual con SQL. Al hacerlo, se quitará el **recurso** de la _máquina virtual SQL_, pero no se eliminará la máquina virtual real. 

Puede ver el modo actual del Agente de IaaS de SQL Server mediante PowerShell: 

  ```powershell-interactive
  # Get the SqlVirtualMachine
  $sqlvm = Get-AzSqlVM -Name $vm.Name  -ResourceGroupName $vm.ResourceGroupName
  $sqlvm.SqlManagementType
  ```

Para actualizar el modo del agente a Full: 


### <a name="azure-portal"></a>Azure portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya al recurso [Máquinas virtuales SQL](manage-sql-vm-portal.md#access-the-sql-virtual-machines-resource). 
1. Seleccione la VM con SQL Server y, luego, **Información general**. 
1. En el caso de las VM con SQL Server con los modos de IaaS NoAgent o Lightweight, seleccione el mensaje **Only license type and edition updates are available with the SQL IaaS extension** (Solo las actualizaciones de la edición y el tipo de licencia están disponibles con la extensión IaaS de SQL).

   ![Selecciones para cambiar el modo desde el portal](./media/sql-vm-resource-provider-register/change-sql-iaas-mode-portal.png)

1. Seleccione la casilla **Acepto reiniciar el servicio SQL Server en la máquina virtual** y, a continuación, seleccione **Confirmar** para actualizar el modo de IaaS a Full. 

    ![Casilla para aceptar el reinicio del servicio SQL Server en la máquina virtual](./media/sql-vm-resource-provider-register/enable-full-mode-iaas.png)

### <a name="command-line"></a>Línea de comandos

# <a name="azure-cli"></a>[CLI de Azure](#tab/bash)

Ejecute el siguiente fragmento de código de la CLI de Azure:

  ```azurecli-interactive
  # Update to full mode
  az sql vm update --name <vm_name> --resource-group <resource_group_name> --sql-mgmt-type full  
  ```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Ejecute el siguiente fragmento de código de PowerShell:

  ```powershell-interactive
  # Get the existing  Compute VM
  $vm = Get-AzVM -Name <vm_name> -ResourceGroupName <resource_group_name>
        
  # Register with SQL VM resource provider in full mode
  Update-AzSqlVM -Name $vm.Name -ResourceGroupName $vm.ResourceGroupName -SqlManagementType Full
  ```

---

## <a name="verify-registration-status"></a>Comprobación del estado de registro
Puede comprobar si su máquina virtual con SQL Server ya se ha registrado con el proveedor de recursos de máquina virtual con SQL mediante Azure Portal, la CLI de Azure o PowerShell. 

### <a name="azure-portal"></a>Azure portal 

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
1. Vaya a las [VM con SQL Server](manage-sql-vm-portal.md).
1. Seleccione la máquina con SQL Server en la lista. Si la máquina virtual con SQL Server no aparece en la lista, es probable que no se haya registrado con el proveedor de recursos de máquina virtual con SQL. 
1. Vea el valor de **Estado**. Si el valor de **Estado** es **Correcto**, la máquina virtual con SQL Server se ha registrado con el proveedor de recursos de máquina virtual con SQL correctamente. 

   ![Comprobación del estado con el registro en un proveedor de recursos de SQL](./media/sql-vm-resource-provider-register/verify-registration-status.png)

### <a name="command-line"></a>Línea de comandos

Verifique el estado actual de registro de la VM con SQL Server mediante la CLI de Azure o PowerShell. `ProvisioningState` mostrará `Succeeded` si el registro fue correcto. 

# <a name="azure-cli"></a>[CLI de Azure](#tab/bash)


  ```azurecli-interactive
  az sql vm show -n <vm_name> -g <resource_group>
 ```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

  ```powershell-interactive
  Get-AzSqlVM -Name <vm_name> -ResourceGroupName <resource_group>
  ```

---

Un error indica que la VM con SQL Server no se ha registrado en el proveedor de recursos. 


## <a name="unregister-from-rp"></a>Anular el registro con el proveedor de recursos

Para anular el registro de VM con SQL Server con el proveedor de recursos de VM con SQL, elimine el *recurso* de la máquina virtual de SQL mediante Azure Portal o la CLI de Azure. Al eliminar el *recurso* de la máquina virtual con SQL no se elimina la VM con SQL Server. Sin embargo, tenga cuidado y siga los pasos detenidamente, ya que es posible eliminar la máquina virtual involuntariamente al intentar quitar el *recurso*. 

Es necesario anular el registro de máquina virtual con SQL con el proveedor de recursos de VM con SQL para cambiar el modo de administración completa. 

### <a name="azure-portal"></a>Azure portal

Para anular el registro de MV con SQL Server con el proveedor de recursos mediante Azure Portal, siga estos pasos:

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com).
1. Vaya al recurso de VM con SQL. 
  
   ![Recurso máquinas virtuales SQL](./media/sql-vm-resource-provider-register/sql-vm-manage.png)

1. Seleccione **Eliminar**. 

   ![Eliminar proveedor de recursos de VM con SQL](./media/sql-vm-resource-provider-register/delete-sql-vm-resource-provider.png)

1. Escriba el nombre de la máquina virtual con SQL y **desactive la casilla situada junto a la máquina virtual**.

   ![Eliminar proveedor de recursos de VM con SQL](./media/sql-vm-resource-provider-register/confirm-delete-of-resource-uncheck-box.png)

   >[!WARNING]
   > Si no se borra la casilla situada junto al nombre de la máquina virtual, se *eliminará* la máquina virtual por completo. Borre la casilla para anular el registro de MV con SQL Server del proveedor de recursos, pero *no eliminar la máquina virtual real*. 

1. Seleccione **Eliminar** para confirmar la eliminación del *recurso* de máquina virtual con SQL y no la VM con SQL Server. 

### <a name="command-line"></a>Línea de comandos

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)
Para anular el registro de la VM con SQL Server del proveedor de recursos con la CLI de Azure, use el comando [az sql vm delete](/cli/azure/sql/vm?view=azure-cli-latest#az-sql-vm-delete). Esto quitará el *recurso* de la VM con SQL Server, pero no eliminará la máquina virtual. 


```azurecli-interactive
az sql vm delete 
  --name <SQL VM resource name> |
  --resource-group <Resource group name> |
  --yes 
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Para anular el registro de la VM con SQL Server del proveedor de recursos con PowerShell, use el comando [Remove-AzSqlVM](/powershell/module/az.sqlvirtualmachine/remove-azsqlvm). Esto quitará el *recurso* de la VM con SQL Server, pero no eliminará la máquina virtual. 

```powershell-interactive
Remove-AzSqlVM -ResourceGroupName <resource_group_name> -Name <VM_name>
```

---

## <a name="limitations"></a>Limitaciones

El proveedor de recursos de máquina virtual con SQL solo admite:
- Máquinas virtuales con SQL Server implementadas mediante Azure Resource Manager. No se admiten las máquinas virtuales con SQL Server implementadas con el modelo clásico. 
- Las máquinas virtuales SQL Server implementadas en la nube pública o Azure Government. No se admiten las implementaciones en otras nubes públicas o privadas. 


## <a name="frequently-asked-questions"></a>Preguntas más frecuentes 

**¿Debo registrar mi máquina virtual con SQL Server aprovisionada a partir de una imagen de SQL Server en Azure Marketplace?**

No. Microsoft registra automáticamente las máquinas virtuales aprovisionadas a partir de imágenes de SQL Server en Azure Marketplace. El registro con el proveedor de recursos de máquina virtual con SQL solo es necesario si la máquina virtual *no* se aprovisionó a partir de las imágenes de SQL Server en Azure Marketplace y SQL Server se instaló automáticamente.

**¿El proveedor de recursos de VM de SQL está disponible para todos los clientes?** 

Sí. Los clientes deben registrar sus máquinas virtuales con SQL Server con el proveedor de recursos de máquina virtual con SQL si no usaron una imagen de SQL Server de Azure Marketplace y, en su lugar, se instaló automáticamente SQL Server o si usan su propio VHD personalizado. Las máquinas virtuales propiedad de todos los tipos de suscripciones (directa, Contrato Enterprise y proveedor de soluciones en la nube) se pueden registrar con el proveedor de recursos de máquina virtual con SQL.

**¿Debo registrarme con el proveedor de recursos de máquina virtual con SQL si mi máquina virtual con SQL Server ya tiene instalada la extensión IaaS de SQL Server?**

Si la máquina virtual con SQL Server se instaló automáticamente y no se aprovisionó a partir de las imágenes de SQL Server de Azure Marketplace, debe registrarla con el proveedor de recursos de máquina virtual con SQL aunque haya instalado la extensión IaaS de SQL Server. Al registrarse con el proveedor de recursos de máquina virtual con SQL, se crea un nuevo recurso de tipo Microsoft.SqlVirtualMachine. La instalación de la extensión IaaS de SQL Server no crea ese recurso.

**¿Cuál es el modo de administración predeterminado al registrarse con el proveedor de recursos de máquina virtual con SQL?**

El modo de administración predeterminado al registrarse con el proveedor de recursos de máquina virtual con SQL es *Completo*. Si no se establece la propiedad de administración de SQL Server al registrarse con el proveedor de recursos de máquina virtual con SQL, se establecerá el modo de administración completa y se reiniciará el servicio SQL Server. Se recomienda registrar la máquina virtual con el proveedor de recursos de máquina virtual con SQL en el modo ligero primero y, posteriormente, actualizar al modo completo durante una ventana de mantenimiento. 

**¿Cuáles son los requisitos previos para registrarse con el proveedor de recursos de máquina virtual con SQL?**

No hay ningún requisito previo para el registro con el proveedor de recursos de VM de SQL en modo ligero o sin agente. El requisito previo para el registro con el proveedor de recursos de máquina virtual con SQL en modo completo es tener la extensión IaaS de SQL Server instalada en la máquina virtual, ya que de lo contrario el servicio SQL Server se reiniciará. 

**¿Puedo realizar el registro con el proveedor de recursos de máquina virtual con SQL si no tengo la extensión IaaS de SQL Server instalada en la máquina virtual?**

Sí, puede registrarse con el proveedor de recursos de máquina virtual con SQL en el modo de administración ligera si no tiene la extensión IaaS de SQL Server instalada en la máquina virtual. En el modo ligero, el proveedor de recursos de máquina virtual con SQL usará una aplicación de consola para comprobar la versión y la edición de la instancia de SQL Server. 

El modo de administración SQL predeterminado al registrarse con el proveedor de recursos de máquina virtual con SQL es _Completo_. Si no se establece la propiedad administración SQL al registrarse con el proveedor de recursos de VM de SQL, se establecerá el modo Administración completa. Se recomienda registrar la máquina virtual con el proveedor de recursos de máquina virtual con SQL en el modo ligero primero y, posteriormente, actualizar al modo completo durante una ventana de mantenimiento. 

**Durante el registro con el proveedor de recursos de máquina virtual con SQL, ¿se instalará un agente en mi máquina virtual?**

Sí, al realizar el registro con el proveedor de recursos de VM con SQL, se instalará un agente en la VM.

La extensión IaaS de SQL Server se basa en el agente para consultar los metadatos de SQL Server. La única vez que no se instala un agente es cuando el proveedor de recursos de la VM con SQL está registrado en modo NoAgent.

**Durante el registro con el proveedor de recursos de VM con SQL, ¿se reiniciará SQL Server en mi VM?**

Depende del modo especificado durante el registro. Si se especifica el modo ligero o NoAgent, el servicio SQL Server no se reiniciará. Sin embargo, si se especifica el modo de administración completa o se deja en blanco, se instalará la extensión IaaS de SQL en modo de administración completa, lo que hará que se reinicie el servicio SQL Server. 

**¿Cuál es la diferencia entre los modos de administración ligera y sin agente al registrarse con el proveedor de recursos de máquina virtual con SQL?** 

El modo de administración sin agente solo está disponible para SQL Server 2008 y SQL Server 2008 R2 en Windows Server 2008. Es el único modo de administración disponible para estas versiones. En todas las demás versiones de SQL Server, los dos modos de administración disponibles son ligera y completa. 

El modo sin agente requiere que el cliente establezca las propiedades de la versión y edición de SQL Server. El modo ligero consulta a la máquina virtual para encontrar la versión y la edición de la instancia de SQL Server.

**¿Puedo registrarme con el proveedor de recursos de máquina virtual con SQL sin especificar el tipo de licencia de SQL Server?**

No. El tipo de licencia de SQL Server no es una propiedad opcional cuando se registra con el proveedor de recursos de máquina virtual con SQL. Tiene que establecer el tipo de licencia de SQL Server como pago por uso o Ventaja híbrida de Azure al registrarse con el proveedor de recursos de máquina virtual con SQL en todos los modos de administración (sin agente, ligera y completa).

**¿Se puede actualizar la extensión IaaS de SQL Server del modo sin agente al modo completo?**

No. La actualización del modo de administración a completo o ligero no está disponible para el modo sin agente. Se trata de una limitación técnica de Windows Server 2008. Tendrá que actualizar primero el sistema operativo a Windows Server 2008 R2 o posterior y, a continuación, podrá actualizar al modo de administración completa. 

**¿Se puede actualizar la extensión IaaS de SQL Server del modo ligero al modo completo?**

Sí. La actualización del modo de administración de ligero a completo se admite desde PowerShell o Azure Portal. Requiere reiniciar el servicio SQL Server.

**¿Se puede cambiar la extensión de IaaS de SQL Server del modo completo a los modos inferiores de administración sin agente o ligera?**

No. No se admite cambiar a una versión inferior del modo de administración de la extensión IaaS de SQL Server. El modo de administración no se puede cambiar a una versión inferior, del modo completo al modo ligero o sin agente, ni del modo ligero al modo sin agente. 

Para cambiar el modo de administración completa a otro, [anule el registro](#unregister-from-rp) de la VM con SQL Server del proveedor de recursos de VM con SQL quitando el *recurso* de SQL Server y vuelva a registrar la VM con SQL Server con el proveedor de recursos de VM con SQL de nuevo en un modo de administración diferente.

**¿Puedo registrarme en un proveedor de recursos de máquina virtual con SQL desde Azure Portal?**

No. El registro en un proveedor de recursos de máquina virtual con SQL no está disponible en Azure Portal. El registro con el proveedor de recursos de máquina virtual con SQL solo se admite con la CLI de Azure o PowerShell. 

**¿Puedo registrar una máquina virtual con el proveedor de recursos de máquina virtual con SQL antes de instalar SQL Server?**

No. Una máquina virtual debe tener al menos una instancia de SQL Server (Motor de base de datos) para registrarse correctamente con el proveedor de recursos de máquina virtual con SQL. Si no hay ninguna instancia de SQL Server en la máquina virtual, el nuevo recurso Microsoft.SqlVirtualMachine tendrá un estado de error.

**¿Puedo registrar una máquina virtual en un proveedor de recursos de máquina virtual con SQL si hay varias instancias de SQL Server?**

Sí. El proveedor de recursos de máquina virtual con SQL solo registrará una instancia de SQL Server (Motor de base de datos). El proveedor de recursos de máquina virtual con SQL registrará la instancia de SQL Server predeterminada en caso de varias instancias. Si no hay ninguna instancia predeterminada, solo se admite el registro en modo ligero. Para actualizar del modo ligero al modo de administración completa, debe existir la instancia de SQL Server predeterminada o la máquina virtual debe tener solo una instancia de SQL Server con nombre.

**¿Se puede registrar una instancia de clúster de conmutación por error de SQL Server con el proveedor de recursos de máquina virtual con SQL?**

Sí. Las instancias de clúster de conmutación por error de SQL Server en una máquina virtual de Azure se pueden registrar con el proveedor de recursos de máquina virtual con SQL en modo ligero. Sin embargo, las instancias de clúster de conmutación por error de SQL Server no se pueden actualizar al modo de administración completa.

**¿Puedo registrar mi máquina virtual con el proveedor de recursos de máquina virtual con SQL si el grupo de disponibilidad AlwaysOn está configurado?**

Sí. No hay ninguna restricción para registrar una instancia de SQL Server instalada en una máquina virtual de Azure con el proveedor de recursos de máquina virtual con SQL si participa en una configuración de grupo de disponibilidad AlwaysOn.

**¿Cuál es el costo del registro en el proveedor de recursos de máquina virtual de SQL o de la actualización al modo de administración completa?**
Ninguno. No hay ningún precio asociado al registro en el proveedor de recursos de máquina virtual de SQL ni al uso de ninguno de los tres modos de manejabilidad. La administración de una VM con SQL Server con el proveedor de recursos es totalmente gratuita. 

**¿Como afecta el uso de los distintos modos de administración al rendimiento?**
El uso de los modos de manejabilidad *NoAgent* y *ligero* no tiene ningún impacto en el rendimiento. Si se usa el modo de manejabilidad *completa* desde dos servicios instalados en el sistema operativo, el impacto es mínimo. Estos se pueden supervisar mediante el administrador de tareas y se muestran en la consola de Servicios de Windows integrada. 

Los dos nombres de servicio son:
- `SqlIaaSExtensionQuery` (nombre para mostrar - `Microsoft SQL Server IaaS Query Service`)
- `SQLIaaSExtension` (nombre para mostrar - `Microsoft SQL Server IaaS Agent`)


## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos. 

* [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
* [Preguntas más frecuentes de SQL Server en máquinas virtuales Windows](frequently-asked-questions-faq.md)  
* [Orientación de precios de SQL Server para máquinas virtuales de Azure](pricing-guidance.md)
* [Notas de la versión de SQL Server en Azure Virtual Machines](../../database/doc-changes-updates-release-notes.md)
