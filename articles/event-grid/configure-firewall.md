---
title: Configuración del firewall de IP para temas o dominios de Azure Event Grid
description: En este artículo se describe cómo configurar las opciones de firewall para los temas o dominios de Event Grid.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: df6098df4817ee6c47378704c25d07433d6b9480
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88509425"
---
# <a name="configure-ip-firewall-for-azure-event-grid-topics-or-domains"></a>Configuración del firewall de IP para temas o dominios de Azure Event Grid 
De forma predeterminada, el tema y el dominio son accesibles desde Internet siempre que la solicitud venga con una autenticación y una autorización válidas. Con el firewall de IP, puede restringirlo aún más a solo un conjunto de direcciones IPv4 o intervalos de direcciones IPv4 en la notación [CIDR (Enrutamiento de interdominios sin clases)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing). Los publicadores que se originen desde cualquier otra dirección IP se rechazarán y recibirán una respuesta 403 (Prohibido). Para más información sobre las características de seguridad de red admitidas por Event Grid, consulte [Seguridad de red para Event Grid](network-security.md).

En este artículo se describe cómo configurar las opciones de firewall de IP para los temas o dominios de Azure Event Grid.

## <a name="use-azure-portal"></a>Usar Azure Portal
En esta sección se muestra cómo usar Azure Portal para crear reglas de firewall de IP de entrada. Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para crear reglas de IP de entrada para **dominios**. 

1. En [Azure Portal](https://portal.azure.com), vaya a su tema o dominio de Event Grid y cambie a la pestaña **Redes**.
2. Seleccione **Redes públicas** para permitir que todas las redes, incluida Internet, tengan acceso al recurso. 

    Puede restringir el tráfico mediante reglas de firewall basadas en IP. Especifique una única dirección IPv4 o un intervalo de direcciones IP en la notación de enrutamiento de interdominios sin clases (CIDR). 

    ![Página de redes públicas](./media/configure-firewall/public-networks-page.png)
3. Seleccione **Private endpoints only** (Solo puntos de conexión privados) para permitir que sean solo las conexiones de puntos de conexión privados las que tengan acceso a este recurso. Use la pestaña **Conexiones de punto de conexión privado** de esta página para administrar las conexiones. 

    ![Página de redes públicas](./media/configure-firewall/private-endpoints-page.png)
4. Seleccione **Guardar** en la barra de herramientas. 



## <a name="use-azure-cli"></a>Uso de CLI de Azure
En esta sección se muestra cómo usar los comandos de la CLI de Azure para crear temas con reglas de IP de entrada. Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para crear reglas de IP de entrada para **dominios**. 


### <a name="prerequisites"></a>Requisitos previos
Actualice la extensión de Azure Event Grid para la CLI mediante el siguiente comando: 

```azurecli-interactive
az extension update -n eventgrid
```

Si la extensión no está instalada, ejecute el siguiente comando para instalarla: 

```azurecli-interactive
az extension add -n eventgrid
```

### <a name="enable-or-disable-public-network-access"></a>Habilitación o deshabilitación del acceso a la red pública
De forma predeterminada, el acceso de red pública está habilitado para los temas y los dominios. También puede habilitarlo explícitamente o deshabilitarlo. Puede restringir el tráfico mediante la configuración de reglas de firewall de IP de entrada. 

#### <a name="enable-public-network-access-while-creating-a-topic"></a>Habilitación del acceso a la red pública durante la creación de un tema

```azurecli-interactive
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location \
    --public-network-access enabled
```


#### <a name="disable-public-network-access-while-creating-a-topic"></a>Deshabilitación del acceso a la red pública durante la creación de un tema

```azurecli-interactive
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location \
    --public-network-access disabled
```

> [!NOTE]
> Cuando se deshabilita el acceso de red pública para un tema o un dominio, no se permite el tráfico a través de la red pública de Internet. Solo se permitirá el acceso a estos recursos a las conexiones de punto de conexión privado. 


#### <a name="enable-public-network-access-for-an-existing-topic"></a>Habilitación del acceso de red pública para un tema existente

```azurecli-interactive
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access enabled 
```

#### <a name="disable-public-network-access-for-an-existing-topic"></a>Deshabilitación del acceso de red pública para un tema existente 

```azurecli-interactive
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access disabled
```

### <a name="create-a-topic-with-single-inbound-ip-rule"></a>Creación de un tema con una sola regla de IP de entrada
El siguiente comando de ejemplo de la CLI crea un tema de Event Grid con reglas de IP de entrada. 

```azurecli-interactive
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR or CIDR MASK> allow 
```

### <a name="create-a-topic-with-multiple-inbound-ip-rules"></a>Creación de un tema con varias reglas de IP de entrada

El siguiente comando de ejemplo de la CLI crea un tema de Event Grid con dos reglas de IP de entrada en un solo paso: 

```azurecli-interactive
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR 1 or CIDR MASK 1> allow \
    --inbound-ip-rules <IP ADDR 2 or CIDR MASK 2> allow
```

### <a name="update-an-existing-topic-to-add-inbound-ip-rules"></a>Actualización de un tema existente para agregar reglas de IP de entrada
En este ejemplo se crea primero un tema de Event Grid y, a continuación, se agregan reglas de IP de entrada para el tema en un comando independiente. También se actualizan las reglas de IP de entrada que se establecieron en el segundo comando. 

```azurecli-interactive

# create the event grid topic first
az eventgrid topic create \
    --resource-group $resourceGroupName \
    --name $topicName \
    --location $location

# add inbound IP rules to an existing topic
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR or CIDR MASK> allow

# later, update topic with additional ip rules
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR 1 or CIDR MASK 1> allow \
    --inbound-ip-rules <IP ADDR 2 or CIDR MASK 2> allow
```

### <a name="remove-an-inbound-ip-rule"></a>Eliminación de una regla de IP de entrada
El siguiente comando quita la segunda regla que creó en el paso anterior mediante la especificación de solo la primera regla al actualizar la configuración. 

```azurecli-interactive
az eventgrid topic update \
    --resource-group $resourceGroupName \
    --name $topicName \
    --public-network-access enabled \
    --inbound-ip-rules <IP ADDR 1 or CIDR MASK 1> allow
```


## <a name="use-powershell"></a>Uso de PowerShell
En esta sección se muestra cómo usar los comandos de Azure PowerShell para crear temas de Azure Event Grid con reglas de firewall de IP de entrada. Los pasos que se muestran en esta sección son para los temas. Puede usar pasos similares para crear reglas de IP de entrada para **dominios**. 

### <a name="prerequisites"></a>Requisitos previos
Siga las instrucciones de [Procedimientos: Use el portal para crear una aplicación de Azure AD y una entidad de servicio que pueda acceder a los recursos](../active-directory/develop/howto-create-service-principal-portal.md) para crear una aplicación de Azure Active Directory y anote los siguientes valores:

- Id. de directorio (inquilino)
- Id. de aplicación (cliente)
- Secreto de la aplicación (cliente)

### <a name="prepare-token-and-headers-for-rest-api-calls"></a>Preparación de tokens y encabezados para las llamadas de la API REST 
Ejecute los siguientes comandos como requisitos previos para obtener un token de autenticación para usarlo con llamadas de la API REST y la autorización y otra información de encabezado. 

```azurepowershell-interactive
# replace <CLIENT ID> and <CLIENT SECRET>
$body = "grant_type=client_credentials&client_id=<CLIENT ID>&client_secret=<CLIENT SECRET>&resource=https://management.core.windows.net"

# get the authentication token. Replace <TENANT ID>
$Token = Invoke-RestMethod -Method Post `
    -Uri https://login.microsoftonline.com/<TENANT ID>/oauth2/token `
    -Body $body `
    -ContentType 'application/x-www-form-urlencoded'

# set authorization and content-type headers
$Headers = @{}
$Headers.Add("Authorization","$($Token.token_type) "+ " " + "$($Token.access_token)")
$Headers.Add("Content-Type","application/json")
```

### <a name="enable-public-network-access-for-an-existing-topic"></a>Habilitación del acceso de red pública para un tema existente
De forma predeterminada, el acceso de red pública está habilitado para los temas y los dominios. Puede restringir el tráfico mediante la configuración de reglas de firewall de IP de entrada. 

```azurepowershell-interactive
$body = @{"properties"=@{"publicNetworkAccess"="enabled"}} | ConvertTo-Json -Depth 5

Invoke-RestMethod -Method 'Patch' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body `
    | ConvertTo-Json -Depth 5
```

### <a name="disable-public-network-access-for-an-existing-topic"></a>Deshabilitación del acceso de red pública para un tema existente
Cuando se deshabilita el acceso de red pública para un tema o un dominio, no se permite el tráfico a través de la red pública de Internet. Solo se permitirá el acceso a estos recursos a las conexiones de punto de conexión privado. 

```azurepowershell-interactive
$body = @{"properties"=@{"publicNetworkAccess"="disabled"}} | ConvertTo-Json -Depth 5

Invoke-RestMethod -Method 'Patch' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body `
    | ConvertTo-Json -Depth 5
```

### <a name="create-an-event-grid-topic-with-inbound-rules-in-one-step"></a>Creación de un tema de Event Grid con reglas de entrada en un solo paso

```azurepowershell-interactive

# prepare the body for the REST PUT method. Notice that inbound IP rules are included. 
$body = @{"location"="<LOCATION>"; "sku"= @{"name"="basic"}; "properties"=@{"publicNetworkAccess"="enabled"; "inboundIpRules"=@(@{"ipmask"="<IP ADDR or CIDR MASK>";"action"="allow"})}} | ConvertTo-Json -Depth 5

# create the event grid topic with inbound IP rules
Invoke-RestMethod -Method 'Put' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body

# verify that the topic was created
Invoke-RestMethod -Method 'Get' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    | ConvertTo-Json -Depth 5
```


### <a name="create-event-grid-topic-first-and-then-add-inbound-ip-rules"></a>Creación del tema de Event Grid seguido de la incorporación de reglas de IP de entrada

```azurepowershell-interactive

# prepare the body for the REST PUT method. Notice that no inbound IP rules are specified. 
$body = @{"location"="<LOCATION>"; "sku"= @{"name"="basic"}; "properties"=@{"publicNetworkAccess"="enabled";}} | ConvertTo-Json -Depth 5

# create the Event Grid topic
Invoke-RestMethod -Method 'Put' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body`

# verify that the topic was created
Invoke-RestMethod -Method 'Get' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    | ConvertTo-Json -Depth 5

# prepare the body for REST PUT method. Notice that it includes inbound IP rules now. This feature is available in both basic and premium tiers.
$body = @{"location"="<LOCATION>"; "sku"= @{"name"="basic"}; "properties"=@{"publicNetworkAccess"="enabled"; "inboundIpRules"=@(@{"ipmask"="<IP ADDR or CIDR MASK>";"action"="allow"}, @{"ipmask"="<IP ADDR or CIDR MASK>";"action"="allow"})}} | ConvertTo-Json -Depth 5

# update the topic with inbound IP rules
Invoke-RestMethod -Method 'Put' `
    -Uri "https://management.azure.com/subscriptions/<AZURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    -Body $body

# verify that the topic was updated
Invoke-RestMethod -Method 'Get' 
    -Uri "https://management.azure.com/subscriptions/<AzURE SUBSCRIPTION ID>/resourceGroups/<RESOURCE GROUP NAME>/providers/Microsoft.EventGrid/topics/<EVENT GRID TOPIC NAME>?api-version=2020-06-01" `
    -Headers $Headers `
    | ConvertTo-Json -Depth 5

```

## <a name="next-steps"></a>Pasos siguientes

* Para información sobre la supervisión de las entregas de eventos, consulte [Supervisar la entrega de mensajes de Event Grid](monitor-event-delivery.md).
* Para más información sobre la clave de autenticación, vea [Seguridad y autenticación de Event Grid](security-authentication.md).
* Para más información acerca de la creación de una suscripción de Azure Event Grid, consulte [Esquema de suscripción de Event Grid](subscription-creation-schema.md).
* Para solucionar problemas de conectividad de red, consulte [Solucionar problemas de conectividad de red](troubleshoot-network-connectivity.md).
