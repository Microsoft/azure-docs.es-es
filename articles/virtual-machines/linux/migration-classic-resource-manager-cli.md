---
title: Migración de VM a Resource Manager mediante la CLI de Azure
description: Este artículo le guía por los pasos de migración de recursos compatible con la plataforma desde el modelo de implementación clásica hasta Azure Resource Manager mediante la CLI de Azure.
author: tanmaygore
manager: vashan
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 02/06/2020
ms.author: tagore
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7537d4c19d6e196aeaa30f8c542e7b3e4e8702c1
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88505973"
---
# <a name="migrate-iaas-resources-from-classic-to-azure-resource-manager-by-using-azure-cli"></a>Migración de recursos de IaaS de la implementación clásica a Azure Resource Manager con la CLI de Azure

> [!IMPORTANT]
> En la actualidad, aproximadamente el 90 % de las máquinas virtuales de IaaS usan [Azure Resource Manager](https://azure.microsoft.com/features/resource-manager/). A partir del 28 de febrero de 2020, las máquinas virtuales clásicas han quedado en desuso y se van a retirar por completo el 1 de marzo de 2023. [Obtenga más información]( https://aka.ms/classicvmretirement) sobre esta caída en desuso y [cómo se va a ver afectado](../classic-vm-deprecation.md#how-does-this-affect-me).

En estos pasos se describe cómo utilizar los comandos de la interfaz de la línea de comandos (CLI) de Azure para migrar recursos de infraestructura como servicio (IaaS) del modelo de implementación clásica al modelo de implementación de Azure Resource Manager. El artículo requiere la [CLI de Azure clásica](/cli/azure/install-classic-cli). Dado que la CLI de Azure solo se puede aplicar a los recursos de Azure Resource Manager, no se puede utilizar para esta migración.

> [!NOTE]
> Todas las operaciones que se describen aquí son idempotentes. Si tiene un problema diferente de una función no admitida o un error de configuración, se recomienda que vuelva a intentar la operación de preparación, anulación o confirmación. De ese modo, la plataforma intentará de nuevo la acción.
> 
> 

<br>
Este es un diagrama de flujo para identificar el orden en que tienen que ejecutarse durante un proceso de migración.

![Screenshot that shows the migration steps](../windows/media/migration-classic-resource-manager/migration-flow.png)

## <a name="step-1-prepare-for-migration"></a>Paso 1: Preparación para la migración
Estos son algunos de los procedimientos recomendados a la hora de evaluar la migración de recursos de IaaS desde el modelo clásico a Resource Manager:

* Consulte la [lista de configuraciones o características no admitidas](../windows/migration-classic-resource-manager-overview.md). Si tiene máquinas virtuales que usan configuraciones o características no admitidas, se recomienda esperar a que se anuncie dicha compatibilidad. Como alternativa, puede quitar esa característica o salir de esa configuración para habilitar la migración si se ajusta a sus necesidades.
* Si tiene actualmente scripts automatizados que implementan la infraestructura y las aplicaciones, intente crear una configuración de prueba similar usando esos scripts para la migración. También puede configurar entornos de ejemplo mediante el Portal de Azure.

> [!IMPORTANT]
> Las puertas de enlace de aplicaciones no se admiten actualmente para realizar migraciones del modelo clásico al de Resource Manager. Si desea migrar una red virtual clásica con una instancia de Application Gateway, quite la puerta de enlace antes de ejecutar una operación de preparación para mover la red. Después, cuando termine el proceso de migración, vuelva a conectar la puerta de enlace en Azure Resource Manager. 
>
>Las puertas de enlace de ExpressRoute que se conectan con circuitos de ExpressRoute en otra suscripción no se pueden migrar automáticamente. En esos casos, quite la puerta de enlace de ExpressRoute, migre la red virtual y vuelva a crear la puerta de enlace. Para obtener más información, consulte [Migración de circuitos de ExpressRoute y las redes virtuales asociadas del modelo de implementación clásica a Resource Manager](../../expressroute/expressroute-migration-classic-resource-manager.md).
> 
> 

## <a name="step-2-set-your-subscription-and-register-the-provider"></a>Paso 2: Establecimiento de la suscripción y registro del proveedor
Para los escenarios de migración, debe configurar el entorno para el modelo clásico y el de Resource Manager. [Instale la CLI de Azure](/cli/azure/install-classic-cli) y [seleccione la suscripción](/cli/azure/authenticate-azure-cli).

Inicie sesión en su cuenta.

```azurecli
azure login
```

Seleccione la suscripción de Azure con el siguiente comando.

```azurecli
azure account set "<azure-subscription-name>"
```

> [!NOTE]
> El registro es un paso que solo se realiza una vez pero que es necesario antes de intentar la migración. Si no se registra, verá el siguiente mensaje de error: 
> 
> *BadRequest: La suscripción no está registrada para la migración.* 
> 
> 

Regístrese con el proveedor de recursos de migración mediante el comando siguiente. Tenga en cuenta que, en algunos casos, se agota el tiempo de espera de este comando. Sin embargo, el registro se realizará correctamente.

```azurecli
azure provider register Microsoft.ClassicInfrastructureMigrate
```

Espere cinco minutos a que finalice el registro. Puede comprobar el estado de la aprobación con el siguiente comando. Asegúrese de que RegistrationState sea `Registered` antes de continuar.

```azurecli
azure provider show Microsoft.ClassicInfrastructureMigrate
```

Ahora cambie CLI al modo `asm` .

```azurecli
azure config mode asm
```

## <a name="step-3-make-sure-you-have-enough-azure-resource-manager-virtual-machine-vcpus-in-the-azure-region-of-your-current-deployment-or-vnet"></a>Paso 3: Asegurarse de tener suficientes unidades vCPU de máquina virtual de Azure Resource Manager en la región de Azure de su VNET o implementación actual
Para este paso debe cambiar a modo `arm` . Para ello, ejecute el siguiente comando.

```azurecli
azure config mode arm
```

Puede usar el siguiente comando de CLI para comprobar el número de unidades vCPU que tiene actualmente en Azure Resource Manager. Para obtener más información sobre las cuotas de vCPU, vea [Límites y Azure Resource Manager](../../azure-resource-manager/management/azure-subscription-service-limits.md#managing-limits).

```azurecli
azure vm list-usage -l "<Your VNET or Deployment's Azure region"
```

Una vez comprobado este paso, puede volver a cambiar a modo `asm` .

```azurecli
azure config mode asm
```

## <a name="step-4-option-1---migrate-virtual-machines-in-a-cloud-service"></a>Paso 4: Opción 1: Migración de máquinas virtuales de un servicio en la nube
Obtenga la lista de servicios en la nube mediante el siguiente comando y seleccione luego el servicio en la nube que quiera migrar. Tenga en cuenta que si las máquinas virtuales del servicio en la nube están en una red virtual o tienen roles web o de trabajo, recibirá un mensaje de error.

```azurecli
azure service list
```

Ejecute el siguiente comando para obtener el nombre de la implementación del servicio en la nube desde la salida detallada. En la mayoría de los casos, el nombre de la implementación es el mismo que el nombre del servicio de nube.

```azurecli
azure service show <serviceName> -vv
```

Primero, valide si puede migrar el servicio en la nube con los siguientes comandos:

```shell
azure service deployment validate-migration <serviceName> <deploymentName> new "" "" ""
```

Prepare las máquinas virtuales del servicio en la nube para la migración. Tiene dos opciones para elegir.

Si quiere migrar las máquinas virtuales a una red virtual creada en una plataforma, use el siguiente comando.

```azurecli
azure service deployment prepare-migration <serviceName> <deploymentName> new "" "" ""
```

Si quiere migrar a una red virtual existente en el modelo de implementación de Resource Manager, utilice el siguiente comando.

```azurecli
azure service deployment prepare-migration <serviceName> <deploymentName> existing <destinationVNETResourceGroupName> <subnetName> <vnetName>
```

Una vez finalizada la operación de preparación, puede consultar la salida detallada para obtener el estado de migración de las máquinas virtuales y asegurarse de que están en estado `Prepared` .

```azurecli
azure vm show <vmName> -vv
```

Compruebe la configuración de los recursos preparados mediante la CLI o el Portal de Azure. Si no está preparado para la migración y desea volver al estado anterior, utilice el siguiente comando.

```azurecli
azure service deployment abort-migration <serviceName> <deploymentName>
```

Si la configuración preparada parece correcta, puede continuar y confirmar los recursos mediante el siguiente comando.

```azurecli
azure service deployment commit-migration <serviceName> <deploymentName>
```

## <a name="step-4-option-2----migrate-virtual-machines-in-a-virtual-network"></a>Paso 4: Opción 2: Migración de máquinas virtuales en una red virtual
Seleccione la red virtual que quiere migrar. Tenga en cuenta que si la red virtual contiene roles web o de trabajo, o bien máquinas virtuales con configuraciones no admitidas, recibirá un mensaje de error de validación.

Obtenga todas las redes virtuales de la suscripción con el siguiente comando.

```azurecli
azure network vnet list
```

El resultado tendrá un aspecto similar al siguiente:

![Captura de pantalla de la línea de comandos con todo el nombre de red virtual resaltado.](../media/virtual-machines-linux-cli-migration-classic-resource-manager/vnet.png)

En el ejemplo anterior, **virtualNetworkName** es el nombre entero **"Group classicubuntu16 classicubuntu16"** .

En primer lugar, valide si puede migrar la red virtual con el siguiente comando:

```shell
azure network vnet validate-migration <virtualNetworkName>
```

Prepare la red virtual de su elección para la migración mediante el siguiente comando:

```azurecli
azure network vnet prepare-migration <virtualNetworkName>
```

Compruebe la configuración de las máquinas virtuales preparadas mediante la CLI o el Portal de Azure. Si no está preparado para la migración y desea volver al estado anterior, utilice el siguiente comando.

```azurecli
azure network vnet abort-migration <virtualNetworkName>
```

Si la configuración preparada parece correcta, puede continuar y confirmar los recursos mediante el siguiente comando.

```azurecli
azure network vnet commit-migration <virtualNetworkName>
```

## <a name="step-5-migrate-a-storage-account"></a>Paso 5: Migración de una cuenta de almacenamiento
Cuando haya terminado de migrar las máquinas virtuales, se recomienda migrar la cuenta de almacenamiento.

Prepare la cuenta de almacenamiento para la migración mediante el siguiente comando.

```azurecli
azure storage account prepare-migration <storageAccountName>
```

Compruebe la configuración de la cuenta de almacenamiento preparada mediante la CLI o el Portal de Azure. Si no está preparado para la migración y desea volver al estado anterior, utilice el siguiente comando.

```azurecli
azure storage account abort-migration <storageAccountName>
```

Si la configuración preparada parece correcta, puede continuar y confirmar los recursos mediante el siguiente comando.

```azurecli
azure storage account commit-migration <storageAccountName>
```

## <a name="next-steps"></a>Pasos siguientes

* [Información general sobre la migración compatible con la plataforma de recursos de IaaS desde el modelo de implementación clásica a Azure Resource Manager](migration-classic-resource-manager-overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Profundización técnica en la migración compatible con la plataforma de la implementación clásica a la de Azure Resource Manager](../migration-classic-resource-manager-deep-dive.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Planning for migration of IaaS resources from classic to Azure Resource Manager](migration-classic-resource-manager-plan.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (Planificación de la migración de recursos de IaaS del modelo clásico a Azure Resource Manager)
* [Migración de recursos de IaaS de la implementación clásica a Resource Manager con Azure PowerShell](../windows/migration-classic-resource-manager-ps.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Herramientas de la comunidad para ayudar con la migración de recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](../windows/migration-classic-resource-manager-community-tools.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
* [Revisión de los errores más comunes en la migración](migration-classic-resource-manager-errors.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Revisión de las preguntas más frecuentes acerca de cómo migrar recursos de IaaS del modelo de implementación clásica a Azure Resource Manager](../migration-classic-resource-manager-faq.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
