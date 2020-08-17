---
title: Configuración del acceso basado en red virtual para la cuenta de Azure Cosmos
description: En este documento se describen los pasos necesarios para configurar un punto de conexión de servicio de red virtual en Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.topic: how-to
ms.date: 06/04/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli
ms.openlocfilehash: 995e5a1a87ee332c48641f42c4134e3e58f11cfa
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495427"
---
# <a name="configure-access-from-virtual-networks-vnet"></a>Configuración del acceso desde redes virtuales (VNET)

Puede configurar las cuentas de Azure Cosmos DB para permitir el acceso solo desde una subred específica de una red virtual de Azure. Para limitar el acceso a una cuenta de Azure Cosmos DB con conexiones desde una subred de una red virtual:

1. Habilite la subred para enviar la identidad de la subred y de la red virtual a Azure Cosmos DB. Para lograrlo, habilite un punto de conexión de servicio para Azure Cosmos DB en la subred específica.

1. Agregue una regla en la cuenta de Azure Cosmos DB para especificar la subred como origen desde el cual se puede tener acceso a la cuenta.

> [!NOTE]
> Una vez que un punto de conexión de servicio para su cuenta de Azure Cosmos DB esté habilitada en una subred, el origen del tráfico que llegue a Azure Cosmos DB cambia de una dirección IP pública a una red virtual y subred. La conmutación de tráfico se aplica a cualquier cuenta de Azure Cosmos DB a la que se tenga acceso desde esta subred. Si las cuentas de Azure Cosmos DB tienen un firewall basado en IP para permitir esta subred, las solicitudes de la subred habilitada por el servicio ya no coincidirán con las reglas de firewall de IP y se rechazarán.
>
> Para más información, consulte los pasos descritos en la sección [Migración desde una regla de firewall de IP a una lista de control de acceso de red virtual](#migrate-from-firewall-to-vnet) de este artículo.

En las secciones siguientes se describe cómo configurar un punto de conexión de servicio de red virtual para una cuenta de Azure Cosmos DB.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="configure-a-service-endpoint-by-using-the-azure-portal"></a><a id="configure-using-portal"></a>Configuración de un punto de conexión de servicio en Azure Portal

### <a name="configure-a-service-endpoint-for-an-existing-azure-virtual-network-and-subnet"></a>Configuración de un punto de conexión de servicio para una subred y red virtual de Azure existentes

1. En la hoja **Todos los recursos**, busque la cuenta de Azure Cosmos DB que desea proteger.

1. Seleccione **Firewalls y redes virtuales** en el menú de configuración y elija Permitir el acceso en **Redes seleccionadas**.

1. Para conceder acceso a la subred de una red virtual existente, en **Redes virtuales**, seleccione **Add existing Azure virtual network** (Agregar red virtual de Azure existente).

1. Seleccione la **suscripción** en la que quiere agregar una red virtual de Azure. Seleccione las **redes virtuales** y las **subredes** de Azure a las que desea conceder acceso a la cuenta de Azure Cosmos DB. A continuación, seleccione **Habilitar** para habilitar las redes seleccionadas con los puntos de conexión de servicio para "Microsoft.AzureCosmosDB". Cuando haya finalizado, seleccione **Agregar**.

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/choose-subnet-and-vnet.png" alt-text="Selección de la red virtual y la subred":::

1. Después de que la cuenta de Azure Cosmos DB esté habilitada para obtener acceso desde una red virtual, solo permitirá el tráfico desde esta subred seleccionada. La red virtual y subred que agregó deben aparecer como se muestra en la captura de pantalla siguiente:

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/vnet-and-subnet-configured-successfully.png" alt-text="Red virtual y subred configuradas correctamente":::

> [!NOTE]
> Para habilitar los puntos de conexión de servicio de red virtual, necesitará los siguientes permisos de suscripción:
>   * Suscripción con red virtual: colaborador de red
>   * Suscripción con cuenta de Azure Cosmos DB: colaborador de cuenta de DocumentDB
>   * Si la red virtual y su cuenta de Azure Cosmos DB están en suscripciones diferentes, asegúrese de que la suscripción que tiene la red virtual también tiene el proveedor de recursos `Microsoft.DocumentDB` registrado. Para registrar un proveedor de recursos, vea el artículo [Tipos y proveedores de recursos de Azure](../azure-resource-manager/management/resource-providers-and-types.md).

Estas son las instrucciones para registrar la suscripción con el proveedor de recursos.

### <a name="configure-a-service-endpoint-for-a-new-azure-virtual-network-and-subnet"></a>Configuración de un punto de conexión de servicio para una subred y red virtual de Azure nuevas

1. En la hoja **Todos los recursos**, busque la cuenta de Azure Cosmos DB que desea proteger.  

1. Seleccione **Firewalls y redes virtuales de Azure** en el menú de configuración y elija Permitir el acceso en **Redes seleccionadas**.  

1. Para conceder acceso a una red virtual de Azure nueva, en **Redes virtuales**, seleccione **Agregar nueva red virtual**.  

1. Proporcione los detalles necesarios para crear una red virtual y, a continuación, seleccione **Crear**. La subred se creará con un punto de conexión de servicio para "Microsoft.AzureCosmosDB" habilitado.

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/choose-subnet-and-vnet-new-vnet.png" alt-text="Selección de la red virtual y la subred para una red virtual nueva":::

Si otros servicios de Azure, como Azure Cognitive Search, usan la cuenta de Azure Cosmos DB, o si se accede a ella desde Stream Analytics o Power BI, permita el acceso al seleccionar **Accept connections from within global Azure datacenters** (Aceptar conexiones desde centros de datos globales de Azure).

Para garantizar el acceso a las métricas de Azure Cosmos DB desde el portal, debe habilitar las opciones **Allow access from Azure portal** (Permitir acceso desde Azure Portal). Para obtener más información sobre estas opciones, consulte el artículo [Configure an IP firewall](how-to-configure-firewall.md) (Configuración de un firewall de dirección IP). Después de habilitar el acceso, seleccione **Guardar** para guardar la configuración.

## <a name="remove-a-virtual-network-or-subnet"></a><a id="remove-vnet-or-subnet"></a>Eliminación de una red virtual o subred

1. En la hoja **Todos los recursos**, busque la cuenta de Azure Cosmos DB a la que ha asignado los puntos de conexión de servicio.  

1. Seleccione **Firewalls y redes virtuales** en el menú de configuración.  

1. Para eliminar una regla de subred o red virtual, seleccione **...** junto a la red virtual o subred y luego **Eliminar**.

   :::image type="content" source="./media/how-to-configure-vnet-service-endpoint/remove-a-vnet.png" alt-text="Eliminación de una red virtual":::

1. Seleccione **Guardar** para aplicar los cambios.

## <a name="configure-a-service-endpoint-by-using-azure-powershell"></a><a id="configure-using-powershell"></a>Configuración de un punto de conexión de servicio mediante Azure PowerShell

> [!NOTE]
> Al usar PowerShell o la CLI de Azure, asegúrese de especificar la lista completa de filtros IP y ACL de red virtual en los parámetros, no solo los que se deben agregar.

Use los pasos siguientes para configurar un punto de conexión de servicio de una cuenta de Azure Cosmos DB mediante Azure PowerShell:  

1. Instale [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps) e [inicie sesión](https://docs.microsoft.com/powershell/azure/authenticate-azureps).  

1. Habilite el punto de conexión de servicio de una subred existente de una red virtual.  

   ```powershell
   $resourceGroupName = "<Resource group name>"
   $vnetName = "<Virtual network name>"
   $subnetName = "<Subnet name>"
   $subnetPrefix = "<Subnet address range>"
   $serviceEndpoint = "Microsoft.AzureCosmosDB"

   Get-AzVirtualNetwork `
      -ResourceGroupName $resourceGroupName `
      -Name $vnetName | Set-AzVirtualNetworkSubnetConfig `
      -Name $subnetName `
      -AddressPrefix $subnetPrefix `
      -ServiceEndpoint $serviceEndpoint | Set-AzVirtualNetwork
   ```

1. Obtenga información sobre la red virtual.

   ```powershell
   $vnet = Get-AzVirtualNetwork `
      -ResourceGroupName $resourceGroupName `
      -Name $vnetName

   $subnetId = $vnet.Id + "/subnets/" + $subnetName
   ```

1. Prepare una regla de red virtual de Cosmos DB.

   ```powershell
   $vnetRule = New-AzCosmosDBVirtualNetworkRule `
      -Id $subnetId
   ```

1. Actualice las propiedades de la cuenta de Azure Cosmos DB con la nueva configuración de punto de conexión de red virtual: 

   ```powershell
   $accountName = "<Cosmos DB account name>"

   Update-AzCosmosDBAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $accountName `
      -EnableVirtualNetwork $true `
      -VirtualNetworkRuleObject @($vnetRule)
   ```

1. Ejecute el siguiente comando para verificar que la cuenta de Azure Cosmos DB se ha actualizado con el punto de conexión de servicio de red virtual que configuró en el paso anterior:

   ```powershell
   $account = Get-AzCosmosDBAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $accountName

   $account.IsVirtualNetworkFilterEnabled
   $account.VirtualNetworkRules
   ```

## <a name="configure-a-service-endpoint-by-using-the-azure-cli"></a><a id="configure-using-cli"></a>Configuración de un punto de conexión de servicio con la CLI de Azure

Las cuentas de Azure Cosmos se pueden configurar para puntos de conexión de servicio cuando se crean o actualizan posteriormente si la subred ya está configurada para ellos. Los puntos de conexión de servicio también se pueden habilitar en la cuenta de Cosmos donde la subred todavía no esté configurada para ellos y empezarán a funcionar cuando la subred se configure más adelante. Esta flexibilidad permite a los administradores que no tienen acceso a la cuenta de Cosmos y a los recursos de la red virtual hacer que sus configuraciones sean independientes entre sí.

### <a name="create-a-new-cosmos-account-and-connect-it-to-a-back-end-subnet-for-a-new-virtual-network"></a>Creación de una nueva cuenta de Cosmos y conexión de esta cuenta a una subred back-end para una nueva red virtual

En este ejemplo, la red virtual y la subred se crean con los puntos de conexión de servicio habilitados para ambas cuando se crean.

```azurecli-interactive
# Create an Azure Cosmos Account with a service endpoint connected to a backend subnet

# Resource group and Cosmos account variables
resourceGroupName='MyResourceGroup'
location='West US 2'
accountName='mycosmosaccount'

# Variables for a new Virtual Network with two subnets
vnetName='myVnet'
frontEnd='FrontEnd'
backEnd='BackEnd'

# Create a resource group
az group create -n $resourceGroupName -l $location

# Create a virtual network with a front-end subnet
az network vnet create \
   -n $vnetName \
   -g $resourceGroupName \
   --address-prefix 10.0.0.0/16 \
   --subnet-name $frontEnd \
   --subnet-prefix 10.0.1.0/24

# Create a back-end subnet with service endpoints enabled for Cosmos DB
az network vnet subnet create \
   -n $backEnd \
   -g $resourceGroupName \
   --address-prefix 10.0.2.0/24 \
   --vnet-name $vnetName \
   --service-endpoints Microsoft.AzureCosmosDB

svcEndpoint=$(az network vnet subnet show -g $resourceGroupName -n $backEnd --vnet-name $vnetName --query 'id' -o tsv)

# Create a Cosmos DB account with default values and service endpoints
az cosmosdb create \
   -n $accountName \
   -g $resourceGroupName \
   --enable-virtual-network true \
   --virtual-network-rules $svcEndpoint
```

### <a name="connect-and-configure-a-cosmos-account-to-a-back-end-subnet-independently"></a>Conexión y configuración de una cuenta de Cosmos en una subred back-end de forma independiente

Este ejemplo está diseñado para mostrar cómo conectar una cuenta de Azure Cosmos a una nueva red virtual existente en la que aún no se ha configurado la subred para los puntos de conexión de servicio. Esto se hace mediante el parámetro `--ignore-missing-vnet-service-endpoint`. Esto permite que la configuración de la cuenta de Cosmos se complete sin errores antes de que se complete la configuración de la subred de la red virtual. Una vez completada la configuración de la subred, se podrá acceder a la cuenta de Cosmos mediante la subred configurada.

```azurecli-interactive
# Create an Azure Cosmos Account with a service endpoint connected to a backend subnet
# that is not yet enabled for service endpoints.

# Resource group and Cosmos account variables
resourceGroupName='MyResourceGroup'
location='West US 2'
accountName='mycosmosaccount'

# Variables for a new Virtual Network with two subnets
vnetName='myVnet'
frontEnd='FrontEnd'
backEnd='BackEnd'

# Create a resource group
az group create -n $resourceGroupName -l $location

# Create a virtual network with a front-end subnet
az network vnet create \
   -n $vnetName \
   -g $resourceGroupName \
   --address-prefix 10.0.0.0/16 \
   --subnet-name $frontEnd \
   --subnet-prefix 10.0.1.0/24

# Create a back-end subnet but without configuring service endpoints (--service-endpoints Microsoft.AzureCosmosDB)
az network vnet subnet create \
   -n $backEnd \
   -g $resourceGroupName \
   --address-prefix 10.0.2.0/24 \
   --vnet-name $vnetName

svcEndpoint=$(az network vnet subnet show -g $resourceGroupName -n $backEnd --vnet-name $vnetName --query 'id' -o tsv)

# Create a Cosmos DB account with default values
az cosmosdb create -n $accountName -g $resourceGroupName

# Add the virtual network rule but ignore the missing service endpoint on the subnet
az cosmosdb network-rule add \
   -n $accountName \
   -g $resourceGroupName \
   --virtual-network $vnetName \
   --subnet svcEndpoint \
   --ignore-missing-vnet-service-endpoint true

read -p'Press any key to now configure the subnet for service endpoints'

az network vnet subnet update \
   -n $backEnd \
   -g $resourceGroupName \
   --vnet-name $vnetName \
   --service-endpoints Microsoft.AzureCosmosDB
```

## <a name="port-range-when-using-direct-mode"></a>Intervalo de puertos al usar el modo directo

Cuando use los puntos de conexión de servicio con una cuenta de Azure Cosmos a través de una conexión de modo directo, debe asegurarse de que el intervalo de puertos TCP de 10000 a 20000 esté disponible.

## <a name="migrating-from-an-ip-firewall-rule-to-a-virtual-network-acl"></a><a id="migrate-from-firewall-to-vnet"></a>Migración desde una regla de firewall de IP a una lista de control de acceso de red virtual

Haga lo siguiente para cambiar una cuenta de Azure Cosmos DB de usar reglas de firewall de IP a usar puntos de conexión de servicio de red virtual.

Después de configurar una cuenta de Azure Cosmos DB para un punto de conexión de servicio de una subred, las solicitudes de esa subred se envían a Azure Cosmos DB con la información de origen de la red virtual y la subred en lugar de con una dirección IP pública de origen. Estas solicitudes dejarán de coincidir con un filtro IP configurado en la cuenta de Azure Cosmos DB, por lo que es necesario hacer lo siguiente para evitar que se produzcan tiempos de inactividad.

Antes de continuar, habilite el punto de conexión de servicio de Azure Cosmos DB en la red virtual y la subred realizando el paso "Habilite el punto de conexión de servicio de una subred existente de una red virtual" anterior.

1. Obtenga la información de red virtual y de subred:

   ```powershell
   $resourceGroupName = "myResourceGroup"
   $accountName = "mycosmosaccount"
   $vnetName = "myVnet"
   $subnetName = "mySubnet"

   $vnet = Get-AzVirtualNetwork `
      -ResourceGroupName $resourceGroupName `
      -Name $vnetName

   $subnetId = $vnet.Id + "/subnets/" + $subnetName
   ```

1. Prepare un nuevo objeto de regla de red virtual para la cuenta de Azure Cosmos DB:

   ```powershell
   $vnetRule = New-AzCosmosDBVirtualNetworkRule `
      -Id $subnetId
   ```

1. Actualice la cuenta de Azure Cosmos DB para habilitar el acceso de punto de conexión de servicio desde la subred:

   ```powershell
   Update-AzCosmosDBAccount `
      -ResourceGroupName $resourceGroupName `
      -Name $accountName `
      -EnableVirtualNetwork $true `
      -VirtualNetworkRuleObject @($vnetRule)
   ```

1. Repita los pasos anteriores con todas las cuentas de Azure Cosmos DB a las que se acceda desde la subred.

1. Quite la regla de firewall IP relativa a la subred de las reglas de firewall de la cuenta de Azure Cosmos DB.

## <a name="next-steps"></a>Pasos siguientes

* Para configurar un firewall para Azure Cosmos DB, consulte el artículo sobre la [compatibilidad con un firewall](firewall-support.md).
