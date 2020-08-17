---
title: 'Aumento de la cuota de punto de conexión: LUIS'
titleSuffix: Azure Cognitive Services
description: Language Understanding (LUIS) ofrece la posibilidad de aumentar la cuota de solicitud de punto de conexión más allá de la cuota de una clave única. Esto se hace al crear más claves para LUIS y agregarlas a la aplicación de LUIS en la página **Publicar**, en la sección **Resources and Keys** (Recursos y claves).
author: diberry
manager: nitinme
ms.custom: seodec18, devx-track-javascript
services: cognitive-services
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 08/20/2019
ms.author: diberry
ms.openlocfilehash: f509d5f6f6e794adeee67fe632518a89882c945c
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407921"
---
# <a name="use-microsoft-azure-traffic-manager-to-manage-endpoint-quota-across-keys"></a>Usar Microsoft Azure Traffic Manager para administrar la cuota de punto de conexión en las claves
Language Understanding (LUIS) ofrece la posibilidad de aumentar la cuota de solicitud de punto de conexión más allá de la cuota de una clave única. Esto se hace al crear más claves para LUIS y agregarlas a la aplicación de LUIS en la página **Publicar**, en la sección **Resources and Keys** (Recursos y claves).

La aplicación cliente tiene que administrar el tráfico en las claves, ya que LUIS no lo hace.

En este artículo, se explica cómo administrar el tráfico en las claves con Azure [Traffic Manager][traffic-manager-marketing]. Ya debe tener una aplicación de LUIS entrenada y publicada. Si no la tiene, siga el [inicio rápido](luis-get-started-create-app.md) del dominio creado previamente.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="connect-to-powershell-in-the-azure-portal"></a>Conectar con PowerShell en Azure Portal
En [Azure][azure-portal] Portal, abra la ventana de PowerShell. El icono de la ventana de PowerShell es el **>_** que se encuentra en la barra de navegación superior. Al usar PowerShell desde el portal, obtiene la última versión de PowerShell y se autentica. PowerShell en el portal requiere una cuenta de [Azure Storage](https://azure.microsoft.com/services/storage/).

![Captura de pantalla de Azure Portal con la ventana de PowerShell abierta](./media/traffic-manager/azure-portal-powershell.png)

En las siguientes secciones se usan [cmdlets de PowerShell de Traffic Manager](https://docs.microsoft.com/powershell/module/az.trafficmanager/#traffic_manager).

## <a name="create-azure-resource-group-with-powershell"></a>Crear el grupo de recursos de Azure con PowerShell
Antes de crear los recursos de Azure, cree un grupo de recursos para que contenga todos los recursos. Ponga al grupo de recursos el nombre `luis-traffic-manager` y use la región `West US`. La región del grupo de recursos almacena metadatos sobre el grupo. No ralentizará los recursos si están en otra región.

Cree un grupo de recursos con el cmdlet **[New-AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup)** :

```powerShell
New-AzResourceGroup -Name luis-traffic-manager -Location "West US"
```

## <a name="create-luis-keys-to-increase-total-endpoint-quota"></a>Crear claves de LUIS para aumentar la cuota total de punto de conexión
1. En Azure Portal, cree dos claves de **Language Understanding**, una en `West US` y otra en `East US`. Use el grupo de recursos existente, que ha creado en la sección anterior, denominado `luis-traffic-manager`.

    ![Captura de pantalla de Azure Portal con dos claves de LUIS en el grupo de recursos luis-traffic-manager](./media/traffic-manager/luis-keys.png)

2. En el sitio web de [LUIS][LUIS], en la sección **Administrar**, en la página **Recursos de Azure**, asigne claves a la aplicación y vuelva a publicar esta; para ello, seleccione el botón **Publicar** del menú superior derecho.

    En la dirección URL de ejemplo de la columna **Punto de conexión** se usa una solicitud GET con la clave de punto de conexión como un parámetro de consulta. Copie las dos nuevas direcciones URL de punto de conexión de las claves. Se usarán como parte de la configuración de Traffic Manager más adelante en este artículo.

## <a name="manage-luis-endpoint-requests-across-keys-with-traffic-manager"></a>Administrar las solicitudes de punto de conexión de LUIS en claves con Traffic Manager
Traffic Manager crea un punto de acceso de DNS para los puntos de conexión. No actúa como puerta de enlace o proxy, sino estrictamente en el nivel de DNS. En este ejemplo, no se cambian los registros de DNS. Se usa una biblioteca de DNS para comunicarse con Traffic Manager a fin de obtener el punto de conexión correcto para esa solicitud en concreto. _Cada_ solicitud para LUIS requiere primero una solicitud de Traffic Manager para determinar qué punto de conexión de LUIS se debe usar.

### <a name="polling-uses-luis-endpoint"></a>El sondeo usa el punto de conexión de LUIS
Traffic Manager sondea los puntos de conexión de forma periódica para asegurarse de que el punto de conexión sigue estando disponible. La dirección URL de Traffic Manager sondeada debe ser accesible con una solicitud GET y devolver 200. La dirección URL del punto de conexión en la página **Publicar** lo hace. Puesto que cada clave de punto de conexión tiene una ruta y parámetros de cadena de consulta diferentes, cada clave de punto de conexión necesita una ruta de sondeo diferente. Cada vez que Traffic Manager sondea, cuesta una solicitud de cuota. El parámetro de cadena de consulta **q** del punto de conexión de LUIS es la expresión que se envía a LUIS. Este parámetro, en lugar de enviar una expresión, se usa para agregar el sondeo de Traffic Manager en el registro de punto de conexión de LUIS como una técnica de depuración a la vez que se configura Traffic Manager.

Dado que cada punto de conexión de LUIS necesita su propia ruta de acceso, necesita su propio perfil de Traffic Manager. Para administrar varios perfiles, cree una arquitectura [_anidada_ de Traffic Manager](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-nested-profiles). Un perfil primario apunta a los perfiles secundarios y administra el tráfico entre ellos.

Una vez que Traffic Manager está configurado, no olvide cambiar la ruta de acceso para usar el parámetro de cadena de consulta logging=false, de modo que su registro no se rellene con el sondeo.

## <a name="configure-traffic-manager-with-nested-profiles"></a>Configurar Traffic Manager con perfiles anidados
En las siguientes secciones, creará dos perfiles secundarios, uno para la clave del este de LUIS y otro para la clave del oeste de LUIS. Después, se crea un perfil primario y se agregan los dos perfiles secundarios al perfil primario.

### <a name="create-the-east-us-traffic-manager-profile-with-powershell"></a>Crear el perfil de Traffic Manager Este de EE. UU. con PowerShell
Para crear el perfil de Traffic Manager Este de EE. UU., es necesario llevar a cabo varios pasos: crear el perfil, así como agregar y establecer el punto de conexión. Un perfil de Traffic Manager puede tener muchos puntos de conexión, pero cada uno de ellos tiene la misma ruta de acceso de validación. Dado que las direcciones URL de punto de conexión de LUIS para las suscripciones del este y oeste son diferentes debido a la clave de la región y el punto de conexión, cada punto de conexión de LUIS tiene que ser un punto de conexión único en el perfil.

1. Cree el perfil con el cmdlet **[New-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.trafficmanager/new-aztrafficmanagerprofile)** :

    Use el siguiente cmdlet para crear el perfil. Asegúrese de cambiar `appIdLuis` y `subscriptionKeyLuis`. El valor de subscriptionKey es para la clave de LUIS Este de EE. UU. Si la ruta de acceso no es correcta, incluidos el identificador de la aplicación de LUIS y la clave de punto de conexión, el sondeo de Traffic Manager es un estado de `degraded` dado que Traffic Manager no puede solicitar correctamente el punto de conexión de LUIS. Asegúrese de que el valor de `q` es `traffic-manager-east` para que pueda ver este valor en los registros de punto de conexión de LUIS.

    ```powerShell
    $eastprofile = New-AzTrafficManagerProfile -Name luis-profile-eastus -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-eastus -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/luis/v2.0/apps/<appID>?subscription-key=<subscriptionKey>&q=traffic-manager-east"
    ```

    En esta tabla, se explica cada variable del cmdlet:

    |Parámetro de configuración|Nombre o valor de la variable|Propósito|
    |--|--|--|
    |-Name|luis-profile-eastus|Nombre de Traffic Manager en Azure Portal|
    |-ResourceGroupName|luis-traffic-manager|Creado en la sección anterior|
    |-TrafficRoutingMethod|Rendimiento|Para más información, consulte [Métodos de enrutamiento de Traffic Manager][routing-methods]. Si usa el rendimiento, la solicitud de dirección URL a Traffic Manager debe proceder de la región del usuario. Si pasa a través de un bot de chat u otra aplicación, es responsabilidad de este imitar la región en la llamada a Traffic Manager. |
    |-RelativeDnsName|luis-dns-eastus|Es el subdominio del servicio: luis-dns-eastus.trafficmanager.net|
    |-Ttl|30|Intervalo de sondeo, 30 segundos|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|El puerto y el protocolo de LUIS son HTTPS/443|
    |-MonitorPath|`/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-east`|Reemplace `<appIdLuis>` y `<subscriptionKeyLuis>` con sus propios valores.|

    Una solicitud correcta no tiene ninguna respuesta.

2. Agregue el punto de conexión Este de EE. UU. con el cmdlet **[Add-AzTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/az.trafficmanager/add-aztrafficmanagerendpointconfig)** :

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName luis-east-endpoint -TrafficManagerProfile $eastprofile -Type ExternalEndpoints -Target eastus.api.cognitive.microsoft.com -EndpointLocation "eastus" -EndpointStatus Enabled
    ```
    En esta tabla, se explica cada variable del cmdlet:

    |Parámetro de configuración|Nombre o valor de la variable|Propósito|
    |--|--|--|
    |-EndpointName|luis-east-endpoint|Nombre del punto de conexión que se muestra en el perfil|
    |-TrafficManagerProfile|$eastprofile|Use el objeto de perfil que ha creado en el paso 1|
    |-Type|ExternalEndpoints|Para más información, consulte [Punto de conexión de Traffic Manager][traffic-manager-endpoints]. |
    |-Target|eastus.api.cognitive.microsoft.com|Este es el dominio del punto de conexión de LUIS.|
    |-EndpointLocation|"eastus"|Región del punto de conexión|
    |-EndpointStatus|habilitado|Habilitar el punto de conexión cuando se crea|

    Una respuesta correcta tiene el siguiente aspecto:

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-eastus
    Name                             : luis-profile-eastus
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-eastus
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /luis/v2.0/apps/<luis-app-id>?subscription-key=f0517d185bcf467cba5147d6260bb868&q=traffic-manager-east
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {luis-east-endpoint}
    ```

3. Establezca el punto de conexión Este de EE. UU. con el cmdlet **[Set-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.trafficmanager/set-aztrafficmanagerprofile)** :

    ```powerShell
    Set-AzTrafficManagerProfile -TrafficManagerProfile $eastprofile
    ```

    Una respuesta correcta será la misma que la del paso 2.

### <a name="create-the-west-us-traffic-manager-profile-with-powershell"></a>Crear el perfil de Traffic Manager Oeste de EE. UU. con PowerShell
Para crear el perfil de Traffic Manager Oeste de EE. UU., siga los mismos pasos: cree el perfil y agregue y establezca el punto de conexión.

1. Cree el perfil con el cmdlet **[New-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.TrafficManager/New-azTrafficManagerProfile)** :

    Use el siguiente cmdlet para crear el perfil. Asegúrese de cambiar `appIdLuis` y `subscriptionKeyLuis`. El valor de subscriptionKey es para la clave de LUIS Este de EE. UU. Si la ruta de acceso no es correcta, incluidos el identificador de la aplicación de LUIS y la clave de punto de conexión, el sondeo de Traffic Manager es un estado de `degraded` dado que Traffic Manager no puede solicitar correctamente el punto de conexión de LUIS. Asegúrese de que el valor de `q` es `traffic-manager-west` para que pueda ver este valor en los registros de punto de conexión de LUIS.

    ```powerShell
    $westprofile = New-AzTrafficManagerProfile -Name luis-profile-westus -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-westus -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-west"
    ```

    En esta tabla, se explica cada variable del cmdlet:

    |Parámetro de configuración|Nombre o valor de la variable|Propósito|
    |--|--|--|
    |-Name|luis-profile-westus|Nombre de Traffic Manager en Azure Portal|
    |-ResourceGroupName|luis-traffic-manager|Creado en la sección anterior|
    |-TrafficRoutingMethod|Rendimiento|Para más información, consulte [Métodos de enrutamiento de Traffic Manager][routing-methods]. Si usa el rendimiento, la solicitud de dirección URL a Traffic Manager debe proceder de la región del usuario. Si pasa a través de un bot de chat u otra aplicación, es responsabilidad de este imitar la región en la llamada a Traffic Manager. |
    |-RelativeDnsName|luis-dns-westus|Es el subdominio del servicio: luis-dns-westus.trafficmanager.net|
    |-Ttl|30|Intervalo de sondeo, 30 segundos|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|El puerto y el protocolo de LUIS son HTTPS/443|
    |-MonitorPath|`/luis/v2.0/apps/<appIdLuis>?subscription-key=<subscriptionKeyLuis>&q=traffic-manager-west`|Reemplace `<appId>` y `<subscriptionKey>` con sus propios valores. Recuerde que esta clave de punto de conexión es diferente de la clave de punto de conexión del este|

    Una solicitud correcta no tiene ninguna respuesta.

2. Agregue el punto de conexión Oeste de EE. UU. con el cmdlet **[Add-AzTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/az.TrafficManager/Add-azTrafficManagerEndpointConfig)** :

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName luis-west-endpoint -TrafficManagerProfile $westprofile -Type ExternalEndpoints -Target westus.api.cognitive.microsoft.com -EndpointLocation "westus" -EndpointStatus Enabled
    ```

    En esta tabla, se explica cada variable del cmdlet:

    |Parámetro de configuración|Nombre o valor de la variable|Propósito|
    |--|--|--|
    |-EndpointName|luis-west-endpoint|Nombre del punto de conexión que se muestra en el perfil|
    |-TrafficManagerProfile|$westprofile|Use el objeto de perfil que ha creado en el paso 1|
    |-Type|ExternalEndpoints|Para más información, consulte [Punto de conexión de Traffic Manager][traffic-manager-endpoints]. |
    |-Target|westus.api.cognitive.microsoft.com|Este es el dominio del punto de conexión de LUIS.|
    |-EndpointLocation|"westus"|Región del punto de conexión|
    |-EndpointStatus|habilitado|Habilitar el punto de conexión cuando se crea|

    Una respuesta correcta tiene el siguiente aspecto:

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-westus
    Name                             : luis-profile-westus
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-westus
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /luis/v2.0/apps/c3fc5d1e-5187-40cc-af0f-fbde328aa16b?subscription-key=e3605f07e3cc4bedb7e02698a54c19cc&q=traffic-manager-west
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {luis-west-endpoint}
    ```

3. Establezca el punto de conexión Oeste de EE. UU. con el cmdlet **[Set-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.TrafficManager/Set-azTrafficManagerProfile)** :

    ```powerShell
    Set-AzTrafficManagerProfile -TrafficManagerProfile $westprofile
    ```

    Una respuesta correcta es la misma que la del paso 2.

### <a name="create-parent-traffic-manager-profile"></a>Crear el perfil primario de Traffic Manager
Cree el perfil primario de Traffic Manager y vincule dos perfiles secundarios de Traffic Manager con el primario.

1. Cree el perfil primario con el cmdlet **[New-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.TrafficManager/New-azTrafficManagerProfile)** :

    ```powerShell
    $parentprofile = New-AzTrafficManagerProfile -Name luis-profile-parent -ResourceGroupName luis-traffic-manager -TrafficRoutingMethod Performance -RelativeDnsName luis-dns-parent -Ttl 30 -MonitorProtocol HTTPS -MonitorPort 443 -MonitorPath "/"
    ```

    En esta tabla, se explica cada variable del cmdlet:

    |Parámetro de configuración|Nombre o valor de la variable|Propósito|
    |--|--|--|
    |-Name|luis-profile-parent|Nombre de Traffic Manager en Azure Portal|
    |-ResourceGroupName|luis-traffic-manager|Creado en la sección anterior|
    |-TrafficRoutingMethod|Rendimiento|Para más información, consulte [Métodos de enrutamiento de Traffic Manager][routing-methods]. Si usa el rendimiento, la solicitud de dirección URL a Traffic Manager debe proceder de la región del usuario. Si pasa a través de un bot de chat u otra aplicación, es responsabilidad de este imitar la región en la llamada a Traffic Manager. |
    |-RelativeDnsName|luis-dns-parent|Es el subdominio del servicio: luis-dns-parent.trafficmanager.net|
    |-Ttl|30|Intervalo de sondeo, 30 segundos|
    |-MonitorProtocol<BR>-MonitorPort|HTTPS<br>443|El puerto y el protocolo de LUIS son HTTPS/443|
    |-MonitorPath|`/`|Esta ruta de acceso no es importante porque las rutas de acceso del punto de conexión secundario se usan en su lugar.|

    Una solicitud correcta no tiene ninguna respuesta.

2. Agregue el perfil secundario de Este de EE. UU. con el cmdlet **[Add-AzTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/az.TrafficManager/Add-azTrafficManagerEndpointConfig)** y el tipo **NestedEndpoints**:

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName child-endpoint-useast -TrafficManagerProfile $parentprofile -Type NestedEndpoints -TargetResourceId $eastprofile.Id -EndpointStatus Enabled -EndpointLocation "eastus" -MinChildEndpoints 1
    ```

    En esta tabla, se explica cada variable del cmdlet:

    |Parámetro de configuración|Nombre o valor de la variable|Propósito|
    |--|--|--|
    |-EndpointName|child-endpoint-useast|Perfil del este|
    |-TrafficManagerProfile|$parentprofile|Perfil que se asignará a este punto de conexión|
    |-Type|NestedEndpoints|Para más información, consulte [Add-AzTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/az.trafficmanager/Add-azTrafficManagerEndpointConfig). |
    |-TargetResourceId|$eastprofile.Id|Identificador del perfil secundario|
    |-EndpointStatus|habilitado|Estado del punto de conexión después de agregarlo al perfil primario|
    |-EndpointLocation|"eastus"|[Nombre de la región de Azure](https://azure.microsoft.com/global-infrastructure/regions/) del recurso|
    |-MinChildEndpoints|1|Número mínimo para los puntos de conexión secundarios|

    Una respuesta correcta tendrá el siguiente aspecto e incluirá el nuevo punto de conexión `child-endpoint-useast`:

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-parent
    Name                             : luis-profile-parent
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-parent
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {child-endpoint-useast}
    ```

3. Agregue el perfil secundario de Oeste de EE. UU. con el cmdlet **[Add-AzTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/az.TrafficManager/Add-azTrafficManagerEndpointConfig)** y el tipo **NestedEndpoints**:

    ```powerShell
    Add-AzTrafficManagerEndpointConfig -EndpointName child-endpoint-uswest -TrafficManagerProfile $parentprofile -Type NestedEndpoints -TargetResourceId $westprofile.Id -EndpointStatus Enabled -EndpointLocation "westus" -MinChildEndpoints 1
    ```

    En esta tabla, se explica cada variable del cmdlet:

    |Parámetro de configuración|Nombre o valor de la variable|Propósito|
    |--|--|--|
    |-EndpointName|child-endpoint-uswest|Perfil del oeste|
    |-TrafficManagerProfile|$parentprofile|Perfil que se asignará a este punto de conexión|
    |-Type|NestedEndpoints|Para más información, consulte [Add-AzTrafficManagerEndpointConfig](https://docs.microsoft.com/powershell/module/az.trafficmanager/Add-azTrafficManagerEndpointConfig). |
    |-TargetResourceId|$westprofile.Id|Identificador del perfil secundario|
    |-EndpointStatus|habilitado|Estado del punto de conexión después de agregarlo al perfil primario|
    |-EndpointLocation|"westus"|[Nombre de la región de Azure](https://azure.microsoft.com/global-infrastructure/regions/) del recurso|
    |-MinChildEndpoints|1|Número mínimo para los puntos de conexión secundarios|

    Una respuesta correcta tendrá el aspecto siguiente e incluirá tanto el punto de conexión `child-endpoint-useast` anterior como el punto de conexión `child-endpoint-uswest` nuevo:

    ```console
    Id                               : /subscriptions/<azure-subscription-id>/resourceGroups/luis-traffic-manager/providers/Microsoft.Network/trafficManagerProfiles/luis-profile-parent
    Name                             : luis-profile-parent
    ResourceGroupName                : luis-traffic-manager
    RelativeDnsName                  : luis-dns-parent
    Ttl                              : 30
    ProfileStatus                    : Enabled
    TrafficRoutingMethod             : Performance
    MonitorProtocol                  : HTTPS
    MonitorPort                      : 443
    MonitorPath                      : /
    MonitorIntervalInSeconds         : 30
    MonitorTimeoutInSeconds          : 10
    MonitorToleratedNumberOfFailures : 3
    Endpoints                        : {child-endpoint-useast, child-endpoint-uswest}
    ```

4. Establezca los puntos de conexión con el cmdlet **[Set-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.TrafficManager/Set-azTrafficManagerProfile)** :

    ```powerShell
    Set-AzTrafficManagerProfile -TrafficManagerProfile $parentprofile
    ```

    Una respuesta correcta es la misma que la del paso 3.

### <a name="powershell-variables"></a>Variables de PowerShell
En las secciones anteriores, se crearon tres variables de PowerShell: `$eastprofile`, `$westprofile` y `$parentprofile`. Estas variables se usan hacia el final de la configuración de Traffic Manager. Si ha decidido no crear las variables, si se ha olvidado de hacerlo o si su ventana de PowerShell agota el tiempo de espera, puede usar el cmdlet de PowerShell **[Get-AzTrafficManagerProfile](https://docs.microsoft.com/powershell/module/az.TrafficManager/Get-azTrafficManagerProfile)** para volver a obtener el perfil y asignarlo a una variable.

Reemplace los elementos que aparecen entre corchetes angulares, `<>`, con los valores correctos para cada uno de los tres perfiles que necesita.

```powerShell
$<variable-name> = Get-AzTrafficManagerProfile -Name <profile-name> -ResourceGroupName luis-traffic-manager
```

## <a name="verify-traffic-manager-works"></a>Comprobar que Traffic Manager funciona
Para comprobar que los perfiles de Traffic Manager funcionan, estos deben tener el estado `Online`. Este estado se basa en la ruta de acceso de sondeo del punto de conexión.

### <a name="view-new-profiles-in-the-azure-portal"></a>Ver perfiles nuevos en Azure Portal
Puede comprobar que los tres perfiles se han creado si consulta los recursos del grupo de recursos `luis-traffic-manager`.

![Captura de pantalla del grupo de recursos de Azure luis-traffic-manager](./media/traffic-manager/traffic-manager-profiles.png)

### <a name="verify-the-profile-status-is-online"></a>Comprobar que el estado del perfil sea Online
Traffic Manager sondea la ruta de acceso de cada punto de conexión para asegurarse de que está en línea. Si lo está, el estado de los perfiles secundarios es `Online`. Esto se muestra en la **Introducción** de cada perfil.

![Captura de pantalla de la Introducción del perfil de Azure Traffic Manager en la que se muestra la Supervisión del estado Online](./media/traffic-manager/profile-status-online.png)

### <a name="validate-traffic-manager-polling-works"></a>Comprobar que el sondeo de Traffic Manager funciona
Otra forma de comprobar que el sondeo de Traffic Manager funciona es con los registros de puntos de conexión de LUIS. En la página de la lista de aplicaciones del sitio web de [LUIS][LUIS], exporte el registro de puntos de conexión de la aplicación. Dado que, a menudo, Traffic Manager sondea los dos puntos de conexión, hay tres entradas en los registros, incluso si solo han estado activados durante unos minutos. No olvide comprobar las entradas en las que la consulta comienza por `traffic-manager-`.

```console
traffic-manager-west    6/7/2018 19:19  {"query":"traffic-manager-west","intents":[{"intent":"None","score":0.944767}],"entities":[]}
traffic-manager-east    6/7/2018 19:20  {"query":"traffic-manager-east","intents":[{"intent":"None","score":0.944767}],"entities":[]}
```

### <a name="validate-dns-response-from-traffic-manager-works"></a>Comprobar que la respuesta de DNS de Traffic Manager funciona
Para comprobar que la respuesta de DNS devuelve un punto de conexión de LUIS, solicite el DNS del perfil primario de Traffic Manager mediante una biblioteca cliente de DNS. El nombre de DNS del perfil primario es `luis-dns-parent.trafficmanager.net`.

El siguiente código de Node.js realiza una solicitud al perfil primario y devuelve un punto de conexión de LUIS:

```javascript
const dns = require('dns');

dns.resolveAny('luis-dns-parent.trafficmanager.net', (err, ret) => {
  console.log('ret', ret);
});
```

La respuesta correcta con el punto de conexión de LUIS es:

```json
[
    {
        value: 'westus.api.cognitive.microsoft.com',
        type: 'CNAME'
    }
]
```

## <a name="use-the-traffic-manager-parent-profile"></a>Usar el perfil primario de Traffic Manager
Para administrar el tráfico a través de los puntos de conexión, debe insertar una llamada al DNS de Traffic Manager para buscar el punto de conexión de LUIS. Esta llamada se realiza para cada solicitud de punto de conexión de LUIS y tiene que simular la ubicación geográfica del usuario de la aplicación cliente de LUIS. Agregue el código de respuesta de DNS entre la aplicación cliente de LUIS y la solicitud a LUIS para la predicción del punto de conexión.

## <a name="resolving-a-degraded-state"></a>Resolver un estado degradado

Habilite los [registros de diagnóstico](../../traffic-manager/traffic-manager-diagnostic-logs.md) para Traffic Manager para ver por qué el estado del punto de conexión es degradado.

## <a name="clean-up"></a>Limpieza
Quite las dos claves de punto de conexión de LUIS, los tres perfiles de Traffic Manager y el grupo de recursos que contenía estos cinco recursos. Para ello, se usa Azure Portal. Elimine los cinco recursos de la lista de recursos y después elimine el grupo de recursos.

## <a name="next-steps"></a>Pasos siguientes

Consulte las opciones de [middleware](https://docs.microsoft.com/azure/bot-service/bot-builder-create-middleware?view=azure-bot-service-4.0&tabs=csaddmiddleware%2Ccsetagoverwrite%2Ccsmiddlewareshortcircuit%2Ccsfallback%2Ccsactivityhandler) de BotFramework v4 para comprender cómo se puede agregar este código de administración de tráfico a un bot de BotFramework.

[traffic-manager-marketing]: https://azure.microsoft.com/services/traffic-manager/
[traffic-manager-docs]: https://docs.microsoft.com/azure/traffic-manager/
[LUIS]: https://docs.microsoft.com/azure/cognitive-services/luis/luis-reference-regions#luis-website
[azure-portal]: https://portal.azure.com/
[azure-storage]: https://azure.microsoft.com/services/storage/
[routing-methods]: https://docs.microsoft.com/azure/traffic-manager/traffic-manager-routing-methods
[traffic-manager-endpoints]: https://docs.microsoft.com/azure/traffic-manager/traffic-manager-endpoint-types
