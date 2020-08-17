---
title: Configuración de puntos de conexión privados para temas o dominios de Azure Event Grid
description: En este artículo se describe cómo configurar los puntos de conexión privados para los temas o dominios de Azure Event Grid.
ms.topic: how-to
ms.date: 07/07/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7796c4160cbf40a241db901e31cc91edc361b00d
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495291"
---
# <a name="configure-private-endpoints-for-azure-event-grid-topics-or-domains"></a>Configuración de puntos de conexión privados para temas o dominios de Azure Event Grid
Puede usar [puntos de conexión privados](../private-link/private-endpoint-overview.md) para permitir la entrada de eventos directamente desde su red virtual a sus temas y dominios de forma segura a través de un [vínculo privado](../private-link/private-link-overview.md) sin tener que ir a la red pública de Internet. El punto de conexión privado usa una dirección IP del espacio de direcciones de la red virtual para el tema o dominio. Para más información conceptual, consulte [Seguridad de las redes](network-security.md).

En este artículo se describe cómo configurar los puntos de conexión privados para los temas o dominios.

## <a name="use-azure-portal"></a>Usar Azure Portal 
En esta sección se muestra cómo usar Azure Portal para crear un punto de conexión privado para un tema o dominio.

> [!NOTE]
> Los pasos que se muestran en esta sección son principalmente para los temas. Puede usar pasos similares para crear puntos de conexión privados para **dominios**. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al tema o dominio.
2. Cambie a la pestaña **redes** de la página del tema. Seleccione **+ Punto de conexión privado** en la barra de herramientas.

    ![Agregar un punto de conexión privado](./media/configure-private-endpoints/add-button.png)
2. En la página **Conceptos básicos**, siga estos pasos: 
    1. Seleccione la **suscripción de Azure** donde desea crear el punto de conexión privado. 
    2. Seleccione un **grupo de recursos de Azure** para el punto de conexión privado. 
    3. Escriba un **nombre** para el punto de conexión. 
    4. Seleccione la **región** del punto de conexión. El punto de conexión privado debe estar en la misma región que la red virtual, pero puede estar en otra región distinta de la del recurso de Private Link (en este ejemplo, un tema de Event Grid). 
    5. Después, seleccione **Next (Siguiente): Recurso >** situado en la parte inferior de la página. 

      ![Punto de conexión privado: página Conceptos básicos](./media/configure-private-endpoints/basics-page.png)
3. En la página **Recurso**, siga estos pasos: 
    1. Como método de conexión, si selecciona **Conectarse a un recurso de Azure en mi directorio.** , siga estos pasos. En este ejemplo se muestra cómo conectarse a un recurso de Azure de su directorio. 
        1. Seleccione la **suscripción de Azure** en la que existe el **tema o dominio**. 
        1. En **Tipo de recurso**, seleccione **Microsoft.EventGrid/topics** o **Microsoft.EventGrid/domains** como **tipo de recurso**.
        2. En **Recurso**, seleccione un tema o dominio de la lista desplegable. 
        3. Confirme que el **subrecurso de destino** está establecido en **tema** o **dominio** (según el tipo de recurso que haya seleccionado).    
        4. Seleccione **Siguiente: Configuración >** situado en la parte inferior de la página. 

            ![Punto de conexión privado: página Recurso](./media/configure-private-endpoints/resource-page.png)
    2. Si selecciona **Connect to a resource using a resource ID or an alias** (Conectarse a un recurso mediante un id. de recurso o un alias), siga estos pasos:
        1. Especifique el identificador del recurso. Por ejemplo: `/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>`.  
        2. En **Recurso**, escriba **tema** o **dominio**. 
        3. (Opcional) Agregue un mensaje de solicitud. 
        4. Seleccione **Siguiente: Configuración >** situado en la parte inferior de la página. 

            ![Punto de conexión privado: página Recurso](./media/configure-private-endpoints/connect-azure-resource-id.png)
4. En la página **Configuración**, seleccione la subred de una red virtual en la que desee implementar el punto de conexión privado. 
    1. Seleccione una **red virtual**. En la lista desplegable, solo se muestran las redes virtuales de la suscripción y la ubicación seleccionadas actualmente. 
    2. Seleccione una **subred** de la red virtual que seleccionó. 
    3. Seleccione **Siguiente: Etiquetas** situado en la parte inferior de la página. 

    ![Punto de conexión privado: página Configuración](./media/configure-private-endpoints/configuration-page.png)
5. En la página **Etiquetas**, cree cualquier etiqueta (nombres y valores) que desee asociar al recurso de punto de conexión privado. Después, en la parte inferior de la página, seleccione el botón **Revisar y crear**. 
6. En **Revisar y crear**, revise toda la configuración y seleccione **Crear** para crear el punto de conexión privado. 

    ![Punto de conexión privado: página Revisar y crear](./media/configure-private-endpoints/review-create-page.png)
    

### <a name="manage-private-link-connection"></a>Administración de una conexión de vínculo privado

Cuando se crea un punto de conexión privado, se debe aprobar la conexión. Si el recurso para el que va a crear un punto de conexión privado está en el directorio, puede aprobar la solicitud de conexión siempre que tenga permisos suficientes. Si se va a conectar a un recurso de Azure en otro directorio, debe esperar a que el propietario de ese recurso apruebe la solicitud de conexión.

Hay cuatro estados de aprovisionamiento:

| Acción del servicio | Estado de punto de conexión privado del consumidor del servicio | Descripción |
|--|--|--|
| None | Pending | La conexión se crea manualmente y está pendiente de aprobación por parte del propietario del recurso de Private Link. |
| Aprobación | Aprobado | La conexión se aprobó de forma automática o manual y está lista para usarse. |
| Reject | Rechazada | El propietario del recurso de vínculo privado rechazó la conexión. |
| Remove | Escenario desconectado | El propietario del recurso del vínculo privado quitó la conexión, el punto de conexión privado se vuelve informativo y debe eliminarse para la limpieza. |
 
###  <a name="how-to-manage-a-private-endpoint-connection"></a>Administración de una conexión de punto de conexión privado
En las secciones siguientes se muestra cómo aprobar o rechazar una conexión de punto de conexión privado. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En la barra de búsqueda, escriba **Temas de Event Grid** o **Dominios de Event Grid**.
1. Seleccione el **tema** o **dominio** que desea administrar.
1. Seleccione la pestaña **Redes**.
1. Si hay alguna conexión pendiente, verá una conexión que aparece con el estado **Pendiente** como estado de aprovisionamiento. 

### <a name="to-approve-a-private-endpoint"></a>Aprobación de un punto de conexión privado
Puede aprobar un punto de conexión privado que esté pendiente. Para aprobarlo, siga estos pasos: 

> [!NOTE]
> Los pasos que se muestran en esta sección son principalmente para los temas. Puede usar pasos similares para aprobar puntos de conexión privados para **dominios**. 

1. Seleccione el **punto de conexión privado** que desea aprobar y seleccione **Aprobar** en la barra de herramientas.

    ![Punto de conexión privado: estado pendiente](./media/configure-private-endpoints/pending.png)
1. En el cuadro de diálogo **Aprobación de la conexión**, escriba un comentario (opcional), y seleccione **Sí**. 

    ![Punto de conexión privado: aprobación](./media/configure-private-endpoints/approve.png)
1. Confirme que el estado del punto de conexión privado aparece como **Aprobado**. 

    ![Punto de conexión privado: estado Aprobado](./media/configure-private-endpoints/approved-status.png)

### <a name="to-reject-a-private-endpoint"></a>Rechazo de un punto de conexión privado
Puede rechazar un punto de conexión privado que tenga el estado Pendiente o Aprobado. Para rechazarlo, siga estos pasos: 

> [!NOTE]
> Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para rechazar puntos de conexión privados para **dominios**. 

1. Seleccione el **punto de conexión privado** que desea rechazar y seleccione **Rechazar** en la barra de herramientas.

    ![Punto de conexión privado: rechazo](./media/configure-private-endpoints/reject-button.png)
1. En el cuadro de diálogo **Rechazo de la conexión**, escriba un comentario (opcional), y seleccione **Sí**. 

    ![Punto de conexión privado: rechazo](./media/configure-private-endpoints/reject.png)
1. Confirme que el estado del punto de conexión privado aparece como **Rechazado**. 

    ![Punto de conexión privado: estado Rechazado](./media/configure-private-endpoints/rejected-status.png)

    > [!NOTE]
    > No se puede aprobar un punto de conexión privado en Azure Portal una vez rechazado. 


## <a name="use-azure-cli"></a>Uso de CLI de Azure
Para crear un punto de conexión privado, use el método [az network private-endpoint create](/cli/azure/network/private-endpoint?view=azure-cli-latest#az-network-private-endpoint-create) que se indica en el siguiente ejemplo:

```azurecli-interactive
az network private-endpoint create \
    --resource-group <RESOURECE GROUP NAME> \
    --name <PRIVATE ENDPOINT NAME> \
    --vnet-name <VIRTUAL NETWORK NAME> \
    --subnet <SUBNET NAME> \
    --private-connection-resource-id "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAME> \
    --connection-name <PRIVATE LINK SERVICE CONNECTION NAME> \
    --location <LOCATION> \
    --group-ids topic
```

Para obtener descripciones de los parámetros utilizados en el ejemplo, consulte la documentación de [az network private-endpoint create](/cli/azure/network/private-endpoint?view=azure-cli-latest#az-network-private-endpoint-create). Tenga en cuenta los siguientes puntos de este ejemplo: 

- Para `private-connection-resource-id`, especifique el identificador del recurso del **tema** o **dominio**. En el ejemplo anterior se utiliza el tipo "tema".
- Para `group-ids`, especifique `topic` o `domain`. En el ejemplo anterior, se usa `topic`. 

Para eliminar un punto de conexión privado, use el método [az network private-endpoint delete](/cli/azure/network/private-endpoint?view=azure-cli-latest#az-network-private-endpoint-delete) que se indica en el siguiente ejemplo:

```azurecli-interactive
az network private-endpoint delete --resource-group <RESOURECE GROUP NAME> --name <PRIVATE ENDPOINT NAME>
```

> [!NOTE]
> Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para crear puntos de conexión privados para **dominios**. 



### <a name="prerequisites"></a>Requisitos previos
Actualice la extensión de Azure Event Grid para la CLI mediante el siguiente comando: 

```azurecli-interactive
az extension update -n eventgrid
```

Si la extensión no está instalada, ejecute el siguiente comando para instalarla: 

```azurecli-interactive
az extension add -n eventgrid
```

### <a name="create-a-private-endpoint"></a>Creación de un punto de conexión privado
Para crear un punto de conexión privado, use el método [az network private-endpoint create](/cli/azure/network/private-endpoint?view=azure-cli-latest#az-network-private-endpoint-create) que se indica en el siguiente ejemplo:

```azurecli-interactive
az network private-endpoint create \
    --resource-group <RESOURECE GROUP NAME> \
    --name <PRIVATE ENDPOINT NAME> \
    --vnet-name <VIRTUAL NETWORK NAME> \
    --subnet <SUBNET NAME> \
    --private-connection-resource-id "/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAME> \
    --connection-name <PRIVATE LINK SERVICE CONNECTION NAME> \
    --location <LOCATION> \
    --group-ids topic
```

Para obtener descripciones de los parámetros utilizados en el ejemplo, consulte la documentación de [az network private-endpoint create](/cli/azure/network/private-endpoint?view=azure-cli-latest#az-network-private-endpoint-create). Tenga en cuenta los siguientes puntos de este ejemplo: 

- Para `private-connection-resource-id`, especifique el identificador del recurso del **tema** o **dominio**. En el ejemplo anterior se utiliza el tipo "tema".
- Para `group-ids`, especifique `topic` o `domain`. En el ejemplo anterior, se usa `topic`. 

Para eliminar un punto de conexión privado, use el método [az network private-endpoint delete](/cli/azure/network/private-endpoint?view=azure-cli-latest#az-network-private-endpoint-delete) que se indica en el siguiente ejemplo:

```azurecli-interactive
az network private-endpoint delete --resource-group <RESOURECE GROUP NAME> --name <PRIVATE ENDPOINT NAME>
```

> [!NOTE]
> Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para crear puntos de conexión privados para **dominios**. 

#### <a name="sample-script"></a>Script de ejemplo
Este es un script de ejemplo que crea los siguientes recursos de Azure:

- Resource group
- Virtual network
- Una subred en la red virtual
- Tema de Azure Event Grid
- Un punto de conexión privado para el tema

> [!NOTE]
> Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para crear puntos de conexión privados para dominios.

```azurecli-interactive
subscriptionID="<AZURE SUBSCRIPTION ID>"
resourceGroupName="<RESOURCE GROUP NAME>"
location="<LOCATION>"
vNetName="<VIRTUAL NETWORK NAME>"
subNetName="<SUBNET NAME>"
topicName = "<TOPIC NAME>"
connectionName="<ENDPOINT CONNECTION NAME>"
endpointName=<ENDPOINT NAME>

# resource ID of the topic. replace <SUBSCRIPTION ID>, <RESOURCE GROUP NAME>, and <TOPIC NAME>
topicResourceID="/subscriptions/<SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<TOPIC NAME>"

# select subscription
az account set --subscription $subscriptionID

# create resource group
az group create --name $resourceGroupName --location $location

# create vnet 
az network vnet create \
    --resource-group $resourceGroupName \
    --name $vNetName \
    --address-prefix 10.0.0.0/16

# create subnet
az network vnet subnet create \
    --resource-group $resourceGroupName \
    --vnet-name $vNetName \
    --name $subNetName \
    --address-prefixes 10.0.0.0/24

# disable private endpoint network policies for the subnet
az network vnet subnet update \
    --resource-group $resourceGroupName \
    --vnet-name $vNetName \
    --name $subNetName \
    --disable-private-endpoint-network-policies true

# create event grid topic. update <LOCATION>
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location

# verify that the topic was created.
az eventgrid topic show \
    --resource-group $resourceGroupName \
    --name $topicName

# create private endpoint for the topic you created
az network private-endpoint create 
    --resource-group $resourceGroupName \
    --name $endpointName \
    --vnet-name $vNetName \
    --subnet $subNetName \
    --private-connection-resource-id $topicResourceID \
    --connection-name $connectionName \
    --location $location \
    --group-ids topic

# get topic 
az eventgrid topic show \
    --resource-group $resourceGroupName \
    --name $topicName

```

### <a name="approve-a-private-endpoint"></a>Aprobación de un punto de conexión privado
En el siguiente ejemplo de fragmento de código de la CLI se muestra cómo aprobar una conexión de punto de conexión privado. 

```azurecli-interactive
az eventgrid topic private-endpoint-connection approve \
    --resource-group $resourceGroupName \
    --topic-name $topicName \
    --name  $endpointName \
    --description "connection approved"
```


### <a name="reject-a-private-endpoint"></a>Rechazo de un punto de conexión privado
En el siguiente ejemplo de fragmento de código de la CLI se muestra cómo rechazar una conexión de punto de conexión privado. 

```azurecli-interactive
az eventgrid topic private-endpoint-connection reject \
    --resource-group $resourceGroupName \
    --topic-name $topicName \
    --name $endpointName \
    --description "Connection rejected"
```

### <a name="disable-public-network-access"></a>Deshabilitación del acceso a una red pública
De forma predeterminada, el acceso a la red pública está habilitado para un tema o dominio de Event Grid. Para permitir el acceso a través de puntos de conexión privados únicamente, deshabilite el acceso a la red pública con el siguiente comando:  

```azurecli-interactive
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access disabled
```


## <a name="use-powershell"></a>Uso de PowerShell
En esta sección se muestra cómo crear un punto de conexión privado para un tema o dominio mediante PowerShell. 

### <a name="prerequisite"></a>Requisito previo
Siga las instrucciones de [Procedimientos: Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos](../active-directory/develop/howto-create-service-principal-portal.md) para crear una aplicación de Azure Active Directory, y anote los valores de **Id. de directorio (inquilino)** , **Id. de aplicación (cliente)** y **Secreto de la aplicación (cliente)** . 

### <a name="prepare-token-and-headers-for-rest-api-calls"></a>Preparación de tokens y encabezados para las llamadas de la API REST 
Ejecute los siguientes comandos como requisitos previos para obtener un token de autenticación para usarlo con llamadas de la API REST y la autorización y otra información de encabezado. 

```azurepowershell-interactive
$body = "grant_type=client_credentials&client_id=<CLIENT ID>&client_secret=<CLIENT SECRET>&resource=https://management.core.windows.net"

# get authentication token
$Token = Invoke-RestMethod -Method Post `
    -Uri https://login.microsoftonline.com/<TENANT ID>/oauth2/token  `
    -Body $body  `
    -ContentType 'application/x-www-form-urlencoded' 

# set authorization and content-type headers
$Headers = @{}
$Headers.Add("Authorization","$($Token.token_type) "+ " " + "$($Token.access_token)")
```

### <a name="create-a-subnet-with-endpoint-network-policies-disabled"></a>Creación de una subred con directivas de red de puntos de conexión deshabilitadas

```azurepowershell-interactive

# create resource group
New-AzResourceGroup -ResourceGroupName <RESOURCE GROUP NAME>  -Location <LOCATION>

# create virtual network
$virtualNetwork = New-AzVirtualNetwork `
                    -ResourceGroupName <RESOURCE GROUP NAME> `
                    -Location <LOCATION> `
                    -Name <VIRTUAL NETWORK NAME> `
                    -AddressPrefix 10.0.0.0/16

# create subnet with endpoint network policy disabled
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
                    -Name example-privatelinksubnet `
                    -AddressPrefix 10.0.0.0/24 `
                    -PrivateEndpointNetworkPoliciesFlag "Disabled" `
                    -VirtualNetwork $virtualNetwork

# update virtual network
$virtualNetwork | Set-AzVirtualNetwork
```

### <a name="create-an-event-grid-topic-with-a-private-endpoint"></a>Creación de un tema de Event Grid con un punto de conexión privado

> [!NOTE]
> Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para crear puntos de conexión privados para **dominios**. 


```azurepowershell-interactive
$body = @{"location"="<LOCATION>"; "properties"=@{"publicNetworkAccess"="disabled"}} | ConvertTo-Json

# create topic
Invoke-RestMethod -Method 'Put'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-04-01-preview"  `
    -Headers $Headers  `
    -Body $body

# verify that the topic was created
$topic=Invoke-RestMethod -Method 'Get'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-04-01-preview"   `
    -Headers $Headers  

# create private link service connection
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
                                -Name "<PRIVATE LINK SERVICE CONNECTION NAME>" `
                                -PrivateLinkServiceId $topic.Id `
                                -GroupId "topic"

# get virtual network info 
$virtualNetwork = Get-AzVirtualNetwork  `
                    -ResourceGroupName  <RESOURCE GROUP NAME> `
                    -Name <VIRTUAL NETWORK NAME>

# get subnet info
$subnet = $virtualNetwork | Select -ExpandProperty subnets `
                             | Where-Object  {$_.Name -eq <SUBNET NAME>}  

# now, you are ready to create a private endpoint 
$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName <RESOURCE GROUP NAME>  `
                                        -Name <PRIVATE ENDPOINT NAME>   `
                                        -Location <LOCATION> `
                                        -Subnet  $subnet   `
                                        -PrivateLinkServiceConnection $privateEndpointConnection

# verify that the endpoint was created
Invoke-RestMethod -Method 'Get'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections?api-version=2020-04-01-preview"   `
    -Headers $Headers   `
    | ConvertTo-Json -Depth 5

```

Cuando compruebe que se ha creado el punto de conexión, debería ver un resultado similar al siguiente:

```json

{
  "value": [
    {
      "properties": {
        "privateEndpoint": {
          "id": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.Network/privateEndpoints/<PRIVATE ENDPOINT NAME>"
        },
        "groupIds": [
          "topic"
        ],
        "privateLinkServiceConnectionState": {
          "status": "Approved",
          "description": "Auto-approved",
          "actionsRequired": "None"
        },
        "provisioningState": "Succeeded"
      },
      "id": "/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>",
      "name": "myConnection",
      "type": "Microsoft.EventGrid/topics/privateEndpointConnections"
    }
  ]
}
```

### <a name="approve-a-private-endpoint-connection"></a>Aprobación de una conexión de punto de conexión privado
En el siguiente ejemplo de fragmento de código de PowerShell se muestra cómo aprobar un punto de conexión privado. 

> [!NOTE]
> Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para aprobar puntos de conexión privados para **dominios**. 

```azurepowershell-interactive
$approvedBody = @{"properties"=@{"privateLinkServiceConnectionState"=@{"status"="approved";"description"="connection approved";"actionsRequired"="none"}}} | ConvertTo-Json

# approve endpoint connection
Invoke-RestMethod -Method 'Put'  `
    -Uri "https://management.azure.com/subscriptions/<AzuRE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-04-01-preview"  `
    -Headers $Headers  `
    -Body $approvedBody

# confirm that the endpoint connection was approved
Invoke-RestMethod -Method 'Get'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-04-01-preview"  `
    -Headers $Headers

```

### <a name="reject-a-private-endpoint-connection"></a>Rechazo de una conexión de punto de conexión privado
En el siguiente ejemplo se muestra cómo rechazar un punto de conexión privado mediante PowerShell. Puede obtener el GUID del punto de conexión privado a partir del resultado del comando GET anterior. 

> [!NOTE]
> Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para rechazar puntos de conexión privados para **dominios**. 


```azurepowershell-interactive
$rejectedBody = @{"properties"=@{"privateLinkServiceConnectionState"=@{"status"="rejected";"description"="connection rejected";"actionsRequired"="none"}}} | ConvertTo-Json

# reject private endpoint
Invoke-RestMethod -Method 'Put'  `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-04-01-preview"  `
    -Headers $Headers  `
    -Body $rejectedBody

# confirm that endpoint was rejected
Invoke-RestMethod -Method 'Get' 
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>/privateEndpointConnections/<PRIVATE ENDPOINT NAME>.<GUID>?api-version=2020-04-01-preview" ` 
    -Headers $Headers
```

Puede aprobar la conexión incluso después de que esta se haya rechazado mediante la API. Si usa Azure Portal, no puede aprobar un punto de conexión que se haya rechazado. 

## <a name="next-steps"></a>Pasos siguientes
* Para más información sobre cómo configurar las opciones de firewall de IP, consulte [Configuración del firewall de IP para temas o dominios de Azure Event Grid](configure-firewall.md).
* Para solucionar problemas de conectividad de red, consulte [Solucionar problemas de conectividad de red](troubleshoot-network-connectivity.md).
