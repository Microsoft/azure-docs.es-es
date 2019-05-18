---
title: Usar la CLI de máquina virtual de SQL Azure para configurar el grupo de disponibilidad AlwaysOn para SQL Server en una máquina virtual de Azure
description: 'Use la CLI de Azure para crear el clúster de conmutación por error de Windows, el agente de escucha del grupo de disponibilidad y el equilibrador de carga interno en una máquina virtual de SQL Server en Azure. '
services: virtual-machines-windows
documentationcenter: na
author: MashaMSFT
manager: craigg
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 02/12/2019
ms.author: mathoma
ms.reviewer: jroth
ms.openlocfilehash: 5efbe874bbf3c1c4081eb7a2c76c1be5a3358ec8
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65518978"
---
# <a name="use-azure-sql-vm-cli-to-configure-always-on-availability-group-for-sql-server-on-an-azure-vm"></a>Usar la CLI de máquina virtual de SQL Azure para configurar el grupo de disponibilidad AlwaysOn para SQL Server en una máquina virtual de Azure
En este artículo se describe cómo usar [CLI de máquina virtual de Azure SQL](/cli/azure/sql/vm?view=azure-cli-latest/) para implementar un clúster de conmutación por error de Windows (WSFC) y agregar las máquinas virtuales de SQL Server al clúster, así como para crear el equilibrador de carga interno y el agente de escucha para un grupo de disponibilidad Always On.  La implementación real del grupo de disponibilidad Always On todavía se realiza manualmente a través de SQL Server Management Studio (SSMS). 

## <a name="prerequisites"></a>Requisitos previos
Para automatizar la instalación de un grupo de disponibilidad AlwaysOn mediante la CLI de máquina virtual de SQL Azure, ya debe tener los siguientes requisitos previos: 
- Una [suscripción a Azure](https://azure.microsoft.com/free/).
- Un grupo de recursos con un controlador de dominio. 
- Uno o más unido al dominio [máquinas virtuales en Azure ejecutando SQL Server 2016 (o posterior) Enterprise edition](https://docs.microsoft.com/azure/virtual-machines/windows/sql/virtual-machines-windows-portal-sql-server-provision) en el *mismo conjunto de disponibilidad o zonas de disponibilidad diferentes* que han sido [registrado con el proveedor de recursos de VM de SQL](virtual-machines-windows-sql-ahb.md#register-sql-server-vm-with-sql-resource-provider).  
- [Azure CLI](/cli/azure/install-azure-cli). 
- (No se usa para cualquier entidad) disponible de dos direcciones IP, uno para el equilibrador de carga interno y otro para el agente de escucha del grupo de disponibilidad dentro de la misma subred que el grupo de disponibilidad. Si se utiliza un equilibrador de carga existente, se necesita una única dirección IP disponible para el agente de escucha del grupo de disponibilidad. 

## <a name="permissions"></a>Permisos
Los permisos de cuenta siguientes son necesarios para configurar el grupo de disponibilidad AlwaysOn mediante la CLI de máquina virtual de SQL Azure. 

- Una cuenta de usuario de dominio existente al que tenga permiso para 'Crear objeto de equipo' en el dominio.  Por ejemplo, una cuenta de administrador de dominio normalmente tiene permisos suficientes (p. ej.: account@domain.com). _Esta cuenta también debe formar parte del grupo de administradores local en cada máquina virtual para crear el clúster._
- La cuenta de usuario de dominio que controla el servicio de SQL Server. 
 
## <a name="step-1---create-storage-account-as-a-cloud-witness"></a>Paso 1: creación de cuenta de almacenamiento como un testigo en la nube
El clúster necesita una cuenta de almacenamiento para que actúe como el testigo en la nube. Puede usar cualquier cuenta de almacenamiento existente o puede crear una nueva cuenta de almacenamiento. Si desea usar una cuenta de almacenamiento existente, vaya a la siguiente sección. 

El fragmento de código siguiente crea la cuenta de almacenamiento: 
```azurecli-interactive
# Create the storage account
# example: az storage account create -n 'cloudwitness' -g SQLVM-RG -l 'West US' `
#  --sku Standard_LRS --kind StorageV2 --access-tier Hot --https-only true

az storage account create -n <name> -g <resource group name> -l <region ex:eastus> `
  --sku Standard_LRS --kind StorageV2 --access-tier Hot --https-only true
```

   >[!TIP]
   > Puede ver el error `az sql: 'vm' is not in the 'az sql' command group` si usa una versión obsoleta de CLI de Azure. Descargue el [versión más reciente de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli-windows?view=azure-cli-latest) para solucionar este error.

## <a name="step-2---define-windows-failover-cluster-metadata"></a>Paso 2: definir los metadatos del clúster de conmutación por error de Windows
La CLI de máquina virtual de Azure SQL [grupo de máquinas virtuales de sql de az](https://docs.microsoft.com/cli/azure/sql/vm/group?view=azure-cli-latest) comando grupo administra los metadatos del servicio de clúster de conmutación por error de Windows (WSFC) que hospeda el grupo de disponibilidad. Metadatos del clúster incluyen el dominio de Active Directory, las cuentas de clúster, las cuentas de almacenamiento que se usará como el testigo en la nube y la versión de SQL Server. Use [crear grupo de máquinas virtuales de sql de az](https://docs.microsoft.com/cli/azure/sql/vm/group?view=azure-cli-latest#az-sql-vm-group-create) para definir los metadatos del wsfc para que cuando se agrega la primera máquina virtual de SQL Server, se crea el clúster, tal como se define. 

El siguiente fragmento de código define los metadatos para el clúster:
```azurecli-interactive
# Define the cluster metadata
# example: az sql vm group create -n Cluster -l 'West US' -g SQLVM-RG `
#  --image-offer SQL2017-WS2016 --image-sku Enterprise --domain-fqdn domain.com `
#  --operator-acc vmadmin@domain.com --bootstrap-acc vmadmin@domain.com --service-acc sqlservice@domain.com `
#  --sa-key '4Z4/i1Dn8/bpbseyWX' `
#  --storage-account 'https://cloudwitness.blob.core.windows.net/'

az sql vm group create -n <cluster name> -l <region ex:eastus> -g <resource group name> `
  --image-offer <SQL2016-WS2016 or SQL2017-WS2016> --image-sku Enterprise --domain-fqdn <FQDN ex: domain.com> `
  --operator-acc <domain account ex: testop@domain.com> --bootstrap-acc <domain account ex:bootacc@domain.com> `
  --service-acc <service account ex: testservice@domain.com> `
  --sa-key '<PublicKey>' `
  --storage-account '<ex:https://cloudwitness.blob.core.windows.net/>'
```

## <a name="step-3---add-sql-server-vms-to-cluster"></a>Paso 3: agregar las máquinas virtuales de SQL Server en clúster
Al agregar la primera máquina virtual de SQL Server al clúster, crea el clúster. El [az sql vm agregar a grupo](https://docs.microsoft.com/cli/azure/sql/vm?view=azure-cli-latest#az-sql-vm-add-to-group) comando crea el clúster con el nombre indicado anteriormente, instala el rol de clúster en las máquinas virtuales de SQL Server y los agrega al clúster. Los usos posteriores de la `az sql vm add-to-group` comando agrega máquinas virtuales de SQL Server adicionales para el clúster recién creado. 

El fragmento de código siguiente crea el clúster y agrega la primera máquina virtual de SQL Server: 

```azurecli-interactive
# Add SQL Server VMs to cluster
# example: az sql vm add-to-group -n SQLVM1 -g SQLVM-RG --sqlvm-group Cluster `
#  -b Str0ngAzur3P@ssword! -p Str0ngAzur3P@ssword! -s Str0ngAzur3P@ssword!
# example: az sql vm add-to-group -n SQLVM2 -g SQLVM-RG --sqlvm-group Cluster `
#  -b Str0ngAzur3P@ssword! -p Str0ngAzur3P@ssword! -s Str0ngAzur3P@ssword!

az sql vm add-to-group -n <VM1 Name> -g <Resource Group Name> --sqlvm-group <cluster name> `
  -b <bootstrap account password> -p <operator account password> -s <service account password>
az sql vm add-to-group -n <VM2 Name> -g <Resource Group Name> --sqlvm-group <cluster name> `
  -b <bootstrap account password> -p <operator account password> -s <service account password>
```
Use este comando para agregar cualquier otra VM de SQL Server al clúster, solo se modifica el `-n` parámetro para el nombre de la máquina virtual de SQL Server. 

## <a name="step-4---create-availability-group"></a>Paso 4: crear el grupo de disponibilidad
Crear manualmente el grupo de disponibilidad como de costumbre, ya sea mediante [SQL Server Management Studio](/sql/database-engine/availability-groups/windows/use-the-availability-group-wizard-sql-server-management-studio), [PowerShell](/sql/database-engine/availability-groups/windows/create-an-availability-group-sql-server-powershell), o [Transact-SQL](/sql/database-engine/availability-groups/windows/create-an-availability-group-transact-sql). 

  >[!IMPORTANT]
  > Hacer **no** crear un agente de escucha en este momento, ya que esto se realiza a través de la CLI de Azure en las secciones siguientes.  

## <a name="step-5---create-internal-load-balancer"></a>Paso 5: crear el equilibrador de carga interno

La escucha de grupo (AG) de disponibilidad AlwaysOn requiere un equilibrador de carga interno de Azure (ILB). El ILB proporciona una dirección IP "flotante" para la escucha de grupo de disponibilidad que permite conmutar por error y volver a conectarse de manera más rápida. Si las máquinas virtuales con SQL Server de un grupo de disponibilidad forman parte del mismo conjunto de disponibilidad, puede usar una instancia de Load Balancer básico; en caso contrario, deberá usar Standard Load Balancer.  **El ILB debe estar en la misma red virtual que las instancias de VM con SQL Server.** 

El siguiente fragmento de código crea el equilibrador de carga interno:

```azurecli-interactive
# Create the Internal Load Balancer
# example: az network lb create --name sqlILB -g SQLVM-RG --sku Standard `
# --vnet-name SQLVMvNet --subnet default

az network lb create --name sqlILB -g <resource group name> --sku Standard `
  --vnet-name <VNet Name> --subnet <subnet name>
```

  >[!IMPORTANT]
  > El recurso de IP pública de cada VM con SQL Server debe tener una SKU estándar para que sea compatible con Standard Load Balancer. Para determinar la SKU del recurso de IP pública de la máquina virtual, vaya a su **grupo de recursos**, seleccione su recurso de **dirección IP pública** para la VM con SQL Server deseada y busque el valor que aparece debajo de **SKU** en el panel **Información general**.  

## <a name="step-6---create-availability-group-listener"></a>Paso 6: creación del agente de escucha del grupo de disponibilidad
Una vez que se creó manualmente el grupo de disponibilidad, puede crear el agente de escucha mediante [az sql vm-del agente de escucha](/cli/azure/sql/vm/group/ag-listener?view=azure-cli-latest#az-sql-vm-group-ag-listener-create). 


- El **Id. de recurso de subred** es el valor de `/subnets/<subnetname>` anexada al identificador de recurso del recurso de red virtual. Para identificar el identificador de recurso de subred, realice lo siguiente:
   1. Vaya al grupo de recursos en el [portal Azure](https://portal.azure.com). 
   1. Seleccione el recurso de red virtual. 
   1. Seleccione **propiedades** en el **configuración** panel. 
   1. Identificar el identificador de recurso para la red virtual y anexe `/subnets/<subnetname>`al final de la aplicación para crear el identificador de recurso de subred. Por ejemplo:
        - Es mi Id. de recurso de red virtual: `/subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet`
        - Mi nombre de subred es `default`.
        - Por lo tanto, mi Id. de recurso de subred es: `/subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet/subnets/default`


El fragmento de código siguiente creará el agente de escucha del grupo de disponibilidad:

```azurecli-interactive
# Create the AG listener
# example: az sql vm group ag-listener create -n AGListener -g SQLVM-RG `
#  --ag-name SQLAG --group-name Cluster --ip-address 10.0.0.27 `
#  --load-balancer sqlilb --probe-port 59999  `
#  --subnet /subscriptions/a1a1-1a11a/resourceGroups/SQLVM-RG/providers/Microsoft.Network/virtualNetworks/SQLVMvNet/subnets/default `
#  --sqlvms sqlvm1 sqlvm2

az sql vm group ag-listener create -n <listener name> -g <resource group name> `
  --ag-name <availability group name> --group-name <cluster name> --ip-address <ag listener IP address> `
  --load-balancer <lbname> --probe-port <Load Balancer probe port, default 59999>  `
  --subnet <subnet resource id> `
  --sqlvms <names of SQL VM's hosting AG replicas ex: sqlvm1 sqlvm2>
```

## <a name="modify-number-of-replicas-in-availability-group"></a>Modificar el número de réplicas en el grupo de disponibilidad
Hay un nivel adicional de complejidad al implementar un grupo de disponibilidad en máquinas virtuales de SQL Server hospedadas en Azure, como los recursos se administran ahora mediante el proveedor de recursos y por la `virtual machine group`. Por lo tanto, al agregar o quitar las réplicas para el grupo de disponibilidad, hay un paso adicional de actualizar los metadatos de agente de escucha con información acerca de las máquinas virtuales de SQL Server. Por lo tanto, cuando se modifica el número de réplicas del grupo de disponibilidad, también debe usar el [actualización del agente de escucha de grupo de disponibilidad de grupo de az sql vm](/cli/azure/sql/vm/group/ag-listener?view=azure-cli-2018-03-01-hybrid#az-sql-vm-group-ag-listener-update) comando para actualizar el agente de escucha con los metadatos de las máquinas virtuales de SQL Server. 


### <a name="add-a-replica"></a>Adición de una réplica

Para agregar una nueva réplica al grupo de disponibilidad, haga lo siguiente:

1. Agregar la máquina virtual de SQL Server al clúster:
   ```azurecli-interactive
   # Add SQL Server VM to the Cluster
   # example: az sql vm add-to-group -n SQLVM3 -g SQLVM-RG --sqlvm-group Cluster `
   # -b Str0ngAzur3P@ssword! -p Str0ngAzur3P@ssword! -s Str0ngAzur3P@ssword!

   az sql vm add-to-group -n <VM3 Name> -g <Resource Group Name> --sqlvm-group <cluster name> `
   -b <bootstrap account password> -p <operator account password> -s <service account password>
   ```
1. Utilizar SQL Server Management Studio (SSMS) para agregar la instancia de SQL Server como una réplica del grupo de disponibilidad.
1. Agregue los metadatos de la máquina virtual de SQL Server para el agente de escucha:
   ```azurecli-interactive
   # Update the listener metadata with the new VM
   # example: az sql vm group ag-listener update -n AGListener `
   # -g sqlvm-rg --group-name Cluster --sqlvms sqlvm1 sqlvm2 sqlvm3

   az sql vm group ag-listener update -n <Listener> `
   -g <RG name> --group-name <cluster name> --sqlvms <SQL VMs, along with new SQL VM>
   ```

### <a name="remove-a-replica"></a>Eliminación de una réplica

Para quitar una réplica del grupo de disponibilidad, haga lo siguiente:

1. Quitar la réplica del grupo de disponibilidad mediante SQL Server Management Studio (SSMS). 
1. Quitar los metadatos de la máquina virtual de SQL Server desde el agente de escucha:
   ```azurecli-interactive
   # Update the listener metadata by removing the VM from the SQLVMs list
   # example: az sql vm group ag-listener update -n AGListener `
   # -g sqlvm-rg --group-name Cluster --sqlvms sqlvm1 sqlvm2

   az sql vm group ag-listener update -n <Listener> `
   -g <RG name> --group-name <cluster name> --sqlvms <SQL VMs that remain>
   ```
1. Quitar la máquina virtual de SQL Server del clúster:
   ```azurecli-interactive
   # Remove SQL VM from cluster
   # example: az sql vm remove-from-group --name SQLVM3 --resource-group SQLVM-RG

   az sql vm remove-from-group --name <SQL VM name> --resource-group <RG name> 
   ```

## <a name="remove-availability-group-listener"></a>Eliminación de la escucha de grupo de disponibilidad
Si posteriormente necesita quitar el agente de escucha del grupo de disponibilidad configurado con la CLI de Azure, debe pasar a través del proveedor de recursos de VM de SQL. Puesto que la escucha se registra mediante este proveedor, no basta con eliminarla mediante SQL Server Management Studio. Realmente debería eliminarse mediante el proveedor de recursos de VM de SQL mediante la CLI de Azure. De esta manera, se eliminan los metadatos de la escucha de grupo de disponibilidad del proveedor y se elimina físicamente la escucha del grupo de disponibilidad. 

El fragmento de código siguiente elimina la escucha de grupo de disponibilidad de SQL del proveedor de recursos de SQL y del grupo de disponibilidad: 

```azurecli-interactive
# Remove the AG listener
# example: az sql vm group ag-listener delete --group-name Cluster --name AGListener --resource-group SQLVM-RG

az sql vm group ag-listener delete --group-name <cluster name> --name <listener name > --resource-group <resource group name>
```

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos. 

* [Introducción a las máquinas virtuales con SQL Server](virtual-machines-windows-sql-server-iaas-overview.md)
* [Preguntas frecuentes sobre las máquinas virtuales con SQL Server](virtual-machines-windows-sql-server-iaas-faq.md)
* [Notas de la versión de las máquinas virtuales con SQL Server](virtual-machines-windows-sql-server-iaas-release-notes.md)
* [Cambio de los modelos de licencia para una VM con SQL Server](virtual-machines-windows-sql-ahb.md)
* [Información general de grupos de disponibilidad Always On &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server)   
* [Configuración de una instancia de servidor para grupos de disponibilidad Always On &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/configuration-of-a-server-instance-for-always-on-availability-groups-sql-server)   
* [Administración de un grupo de disponibilidad &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/administration-of-an-availability-group-sql-server)   
* [Supervisión de grupos de disponibilidad &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/monitoring-of-availability-groups-sql-server)
* [Información general sobre instrucciones Transact-SQL para grupos de disponibilidad Always On &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/transact-sql-statements-for-always-on-availability-groups)   
* [Introducción a los PowerShell Cmdlets para grupos de disponibilidad Always On &#40;SQL Server&#41;](/sql/database-engine/availability-groups/windows/overview-of-powershell-cmdlets-for-always-on-availability-groups-sql-server)  
