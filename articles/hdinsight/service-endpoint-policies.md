---
title: 'Configuración de directivas de punto de conexión de servicio: Azure HDInsight'
description: Aprenda a configurar directivas de punto de conexión de servicio para la red virtual con Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 07/15/2020
ms.openlocfilehash: f2752e5ab2bf7c2926ec9e2c0e4929eab91ed377
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88530977"
---
# <a name="configure-virtual-network-service-endpoint-policies-for-azure-hdinsight"></a>Configuración de directivas de punto de conexión de servicio en la red virtual con Azure HDInsight

En este artículo se proporciona información sobre la implementación de directivas de punto de conexión de servicio en redes virtuales con Azure HDInsight.

## <a name="background"></a>Información previa

Azure HDInsight le permite crear clústeres en su red virtual propia. Si necesita permitir el tráfico saliente de la red virtual a otros servicios de Azure, como cuentas de almacenamiento, puede crear [directivas de punto de conexión de servicio](../virtual-network/virtual-network-service-endpoint-policies-overview.md). Las directivas de punto de conexión de servicio que se crean mediante Azure Portal solo permiten crear una directiva para una sola cuenta, todas las cuentas de una suscripción o todas las cuentas de un grupo de recursos.

Sin embargo, como servicio administrado, Azure HDInsight colecciona los archivos de datos y de registro de cada clúster en cuentas de almacenamiento específicas en cada región. Para que estos datos lleguen a HDInsight desde la red virtual, es necesario crear directivas de punto de conexión de servicio que permitan el tráfico saliente a puntos de colección de datos específicos administrados por Azure HDInsight.

## <a name="service-endpoint-policies-for-hdinsight"></a>Directivas de punto de conexión de servicio para HDInsight

Estas directivas de punto de conexión de servicio admiten la siguiente funcionalidad:

- Colección de registros y telemetría durante la creación de clústeres, la ejecución de trabajos y las operaciones de plataforma, como el escalado.
- Adjuntado de discos duros virtuales (VHD) a los nodos de clúster recién creados para aprovisionar software y bibliotecas en el clúster.

Si no se crean directivas de punto de conexión de servicio para habilitar este flujo de datos, se puede producir un error en la creación del clúster y Azure HDInsight no podrá ofrecer soporte técnico a los clústeres.

## <a name="create-service-endpoint-policies-for-hdinsight"></a>Creación de directivas de punto de conexión de servicio para HDInsight

Asegúrese de que las directivas de punto de conexión de servicio correctas están conectadas a la red virtual antes de crear clústeres. De lo contrario puede producirse un error en la creación del clúster o de otro tipo.

Utilice el siguiente proceso para crear las directivas de punto de conexión de servicio necesarias:

1. Decida la región en la que va a crear el clúster de HDInsight.
1. Busque esa región en la [lista de recursos de la directiva de punto de conexión de servicio](https://github.com/Azure-Samples/hdinsight-enterprise-security/blob/main/hdinsight-service-endpoint-policy-resources.json), que ofrece todos los grupos de recursos para las cuentas de almacenamiento de administración de HDInsight.
1. Seleccione la lista de grupos de recursos para su región. A continuación se muestra un ejemplo de los recursos para `Canada Central`:

    ```json
    "Canada Central":[
        "/subscriptions/235d341f-7fb9-435c-9bdc-034b7306c9b4/resourceGroups/Default-Storage-WestUS",
        "/subscriptions/da0c4c68-9283-4f88-9c35-18f7bd72fbdd/resourceGroups/GenevaWarmPathManageRG",
        "/subscriptions/6a853a41-3423-4167-8d9c-bcf37dc72818/resourceGroups/GenevaWarmPathManageRG",
        "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/Default-Storage-CanadaCentral",
        "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/cancstorage",
        "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/GenevaWarmPathManageRG"
    ],
    ```

1. Inserte la lista de grupos de recursos en el script de instalación escrito en la CLI de Azure o Azure PowerShell.

    ```azurecli
    $subscriptionId = "<subscription id>"
    $rgName="<resource group name> "
    $location="<location name>"
    $vnetName="<vnet name>"
    $subnetName="<subnet name>"
    $sepName="<service endpoint policy name>"
    $sepDefName="<service endpoint policy definition name>"
    
    # Set to the right subscription ID
    az account set --subscription $subscriptionId
    
    # setup service endpoint on the virtual network subnet
    az network vnet subnet update -g $rgName --vnet-name $vnetName -n $subnetName --service-endpoints Microsoft.Storage
    
    # Create Service Endpoint Policy
    az network service-endpoint policy create -g $rgName  -n $sepName -l $location
    
    # Insert the list of HDInsight owned resources for the region your clusters will be created in.
    [String[]]$resources = @("/subscriptions/235d341f-7fb9-435c-9bdc-034b7306c9b4/resourceGroups/Default-Storage-WestUS",`
    "/subscriptions/da0c4c68-9283-4f88-9c35-18f7bd72fbdd/resourceGroups/GenevaWarmPathManageRG",`
    "/subscriptions/6a853a41-3423-4167-8d9c-bcf37dc72818/resourceGroups/GenevaWarmPathManageRG",`
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/Default-Storage-CanadaCentral",`
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/cancstorage",`
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/GenevaWarmPathManageRG")
    
    #Assign service resources to the SEP policy.
    az network service-endpoint policy-definition create -g $rgName --policy-name $sepName -n $sepDefName --service "Microsoft.Storage" --service-resources $resources
    
    # Associate a subnet to the service endpoint policy just created. If there is a delay in updating it to subnet, you can use the Azure portal to associate the policy with the subnet.
    az network vnet subnet update -g $rgName --vnet-name $vnetName -n $subnetName --service-endpoint-policy $sepName
    ```

    Si prefiere configurar la directiva de punto de conexión de servicio mediante PowerShell, use el siguiente fragmento de código.
    
    ```powershell
    #Script to assign SEP 
    $subscriptionId = "<subscription id>"
    $rgName = "<resource group name>"
    $vnetName = "<vnet name>"
    $subnetName = "<subnet Name"
    $location = "Canada Central"
    
    # Connect to your Azure Account
    Connect-AzAccount
    
    # Select the Subscription that you want to use
    Select-AzSubscription -SubscriptionId $subscriptionId
    
    # Retrieve VNet Config
    $vnet = Get-AzVirtualNetwork -ResourceGroupName $rgName -Name $vnetName
    
    # Retrieve Subnet Config
    $subnet = Get-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet
    
    # Insert the list of HDInsight owned resources for the region your clusters will be created in.
    [String[]]$resources = @("/subscriptions/235d341f-7fb9-435c-9bdc-034b7306c9b4/resourceGroups/Default-Storage-WestUS",
    "/subscriptions/da0c4c68-9283-4f88-9c35-18f7bd72fbdd/resourceGroups/GenevaWarmPathManageRG",
    "/subscriptions/6a853a41-3423-4167-8d9c-bcf37dc72818/resourceGroups/GenevaWarmPathManageRG",
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/Default-Storage-CanadaCentral",
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/cancstorage",
    "/subscriptions/c8845df8-14d1-4a46-b6dd-e0c44ae400b0/resourceGroups/GenevaWarmPathManageRG")
    
    #Declare service endpoint policy definition
    $sepDef = New-AzServiceEndpointPolicyDefinition -Name "SEPHDICanadaCentral" -Description "Service Endpoint Policy Definition" -Service "Microsoft.Storage" -ServiceResource $resources
    
    # Service Endpoint Policy
    $sep= New-AzServiceEndpointPolicy -ResourceGroupName $rgName -Name "SEPHDICanadaCentral" -Location $location -ServiceEndpointPolicyDefinition $sepDef
    
    # Associate a subnet to the service endpoint policy just created. If there is a delay in updating it to subnet, you can use the Azure portal to associate the policy with the subnet.
    Set-AzVirtualNetworkSubnetConfig -Name $subnetName -VirtualNetwork $vnet -AddressPrefix $subnet.AddressPrefix -ServiceEndpointPolicy $sep
    ```

## <a name="next-steps"></a>Pasos siguientes

* [Arquitectura de red virtual de Azure HDInsight](hdinsight-virtual-network-architecture.md)
* [Configuración de una aplicación virtual de red](./network-virtual-appliance.md)
