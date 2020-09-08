---
title: Uso de PowerShell para administrar Traffic Manager en Azure
description: Con esta ruta de aprendizaje, empiece por el uso de Azure PowerShell para Traffic Manager.
services: traffic-manager
documentationcenter: na
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/16/2017
ms.author: duau
ms.openlocfilehash: 21076fe23301c189d9987f78706cc81691ce7a4f
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400575"
---
# <a name="using-powershell-to-manage-traffic-manager"></a>Uso de PowerShell para administrar Traffic Manager

Azure Resource Manager es la interfaz de administración de servicios preferida en Azure. Los perfiles de Traffic Manager se pueden administrar mediante las herramientas y las API basadas en Azure Resource Manager.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="resource-model"></a>Modelo de recursos

El Administrador de tráfico de Azure se configura con una colección de valores denominada perfil del Administrador de tráfico. Este perfil contiene la configuración de DNS, la de enrutamiento del tráfico, la de supervisión de puntos de conexión y una lista de los puntos de conexión de servicio a los que se enruta el tráfico.

Cada perfil de Traffic Manager se representa mediante un recurso de tipo "TrafficManagerProfiles". En el nivel de la API de REST, el URI de cada perfil es el siguiente:

`https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.Network/trafficManagerProfiles/{profile-name}?api-version={api-version}`

## <a name="setting-up-azure-powershell"></a>Configuración de Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

En estas instrucciones se usa PowerShell para Microsoft Azure. En el siguiente artículo se explica cómo instalar y configurar Azure PowerShell.

* [Instalación y configuración de Azure PowerShell](/powershell/azure/)

En los ejemplos de este artículo se da por supuesto que ya tiene un grupo de recursos. Puede crear uno mediante el siguiente comando:

```powershell
New-AzResourceGroup -Name MyRG -Location "West US"
```

> [!NOTE]
> Azure Resource Manager requiere que todos los grupos de recursos cuenten con una ubicación. Esta se utiliza como predeterminada para los recursos que se crean en ese grupo de recursos. Sin embargo, puesto que los recursos del perfil de Traffic Manager son globales y no regionales, la elección de la ubicación del grupo de recursos no afecta a Azure Traffic Manager.

## <a name="create-a-traffic-manager-profile"></a>Creación de un perfil del Administrador de tráfico

Para crear un perfil de Traffic Manager, use el cmdlet `New-AzTrafficManagerProfile`:

```powershell
$TmProfile = New-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName contoso -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
```

En la siguiente tabla se describen los parámetros:

| Parámetro | Descripción |
| --- | --- |
| Nombre |Nombre del recurso del perfil de Traffic Manager. Los perfiles del mismo grupo de recursos deben tener nombres únicos. Este nombre es independiente del nombre DNS que se utiliza para las consultas de DNS. |
| ResourceGroupName |Nombre del grupo de recursos que contiene el recurso de perfil. |
| TrafficRoutingMethod |Especifica el método de enrutamiento del tráfico que se usa para determinar qué punto de conexión se devuelve en respuesta a una consulta de DNS. Los valores posibles son "Performance", "Weighted" o "Priority". |
| RelativeDnsName |Especifica la parte correspondiente al nombre de host del nombre DNS proporcionado por este perfil de Traffic Manager. Este valor se combina con el nombre de dominio DNS usado por Azure Traffic Manager para formar el nombre de dominio completo (FQDN) del perfil. Por ejemplo, con el valor "contoso", se obtiene "contoso.trafficmanager.net". |
| TTL |Especifica el período de vida (TTL) de DNS, en segundos. Este TTL informa a las resoluciones DNS locales y a los clientes DNS de cuánto tiempo se guardan en memoria caché las respuestas DNS proporcionadas por este perfil de Traffic Manager. |
| MonitorProtocol |Especifica el protocolo que se va a usar para supervisar el estado de los puntos de conexión. Los valores posibles son "HTTP" y "HTTPS". |
| MonitorPort |Especifica el puerto TCP usado para supervisar el estado de los puntos de conexión. |
| MonitorPath |Especifica la ruta de acceso relativa al nombre de dominio de punto de conexión usado para sondear el estado del punto de conexión. |

El cmdlet crea un perfil de Traffic Manager en Azure y devuelve un objeto de perfil correspondiente a PowerShell. El perfil aún no contiene puntos de conexión. Para más información sobre cómo agregar puntos de conexión a un perfil de Traffic Manager, consulte Incorporación de puntos de conexión de Traffic Manager.

## <a name="get-a-traffic-manager-profile"></a>Obtención de un perfil del Administrador de tráfico

Para recuperar un objeto del perfil de Traffic Manager existente, use el cmdlet `Get-AzTrafficManagerProfle`:

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
```

Este cmdlet devuelve un objeto del perfil del Administrador de tráfico.

## <a name="update-a-traffic-manager-profile"></a>Actualización de un perfil de Traffic Manager

Para modificar perfiles de Traffic Manager, se sigue un proceso de tres pasos:

1. Recupere el perfil mediante `Get-AzTrafficManagerProfile` o use el devuelto por `New-AzTrafficManagerProfile`.
2. Modifique el perfil. Puede agregar y quitar puntos de conexión o cambiar los parámetros del perfil o el punto de conexión. Estos cambios son operaciones fuera de línea. Solo cambia el objeto el objeto local en memoria que representa el perfil.
3. Confirme los cambios mediante el cmdlet `Set-AzTrafficManagerProfile`.

Se pueden cambiar todas las propiedades del perfil, excepto RelativeDnsName para el perfil. Para cambiar RelativeDnsName, debe eliminar el perfil y crear uno con un nombre diferente.

En el ejemplo siguiente se muestra cómo cambiar el TTL del perfil:

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
$TmProfile.Ttl = 300
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```

Existen tres tipos de puntos de conexión de Administrador de tráfico:

1. Los **puntos de conexión de Azure** son servicios hospedados en Azure.
2. Los **puntos de conexión externos** son servicios hospedados fuera de Azure.
3. Los **puntos de conexión anidados** se usan para construir jerarquías anidadas de perfiles de Traffic Manager. Los puntos de conexión anidados hacen posibles configuraciones avanzadas de enrutamiento del tráfico para aplicaciones complejas.

En los tres casos, se pueden agregar puntos de conexión de dos maneras:

1. Mediante el proceso de tres pasos descrito antes. La ventaja de este método es que se pueden realizar varios cambios en el punto de conexión en una sola actualización.
2. Con el cmdlet New-AzTrafficManagerEndpoint. Este cmdlet agrega un punto de conexión a un perfil de Traffic Manager existente en una sola operación.

## <a name="adding-azure-endpoints"></a>Adición de puntos de conexión de Azure

Los puntos de conexión de Azure hacen referencia a servicios hospedados en Azure. Se admiten dos tipos de puntos de conexión de Azure:

1. Azure App Service
2. Recursos de PublicIpAddress de Azure (que pueden estar asociados a un equilibrador de carga o a una NIC de máquina virtual). PublicIpAddress debe tener un nombre DNS asignado para usarse en Traffic Manager.

En cada caso:

* El servicio se especifica mediante el parámetro '"targetResourceId" de `Add-AzTrafficManagerEndpointConfig` o `New-AzTrafficManagerEndpoint`.
* Los parámetros "Target" y "EndpointLocation" están implícitos en TargetResourceId.
* Especificar el valor de 'Weight' es opcional. La ponderación solo se usa si el perfil está configurado para usar el método de enrutamiento del tráfico "Weighted". En caso contrario, se pasan por alto. Si se especifica, el valor debe ser un número entre 1 y 1000. El valor predeterminado es 1.
* Especificar el valor de 'Priority' es opcional. Las prioridades solo se usan si el perfil está configurado para usar el método de enrutamiento del tráfico "Priority". En caso contrario, se pasan por alto. Los valores válidos son de 1 a 1000; los valores más bajos indican una prioridad más alta. Si se especifica para un punto de conexión, se debe especificar para todos los puntos de conexión. Si se omite, se aplican los valores predeterminados a partir de "1" en el orden en que se indican los puntos de conexión.

### <a name="example-1-adding-app-service-endpoints-using-add-aztrafficmanagerendpointconfig"></a>Ejemplo 1: Adición de puntos de conexión de App Service mediante `Add-AzTrafficManagerEndpointConfig`

En este ejemplo, vamos a crear un perfil de Traffic Manager y a agregar dos puntos de conexión de App Service utilizando el cmdlet `Add-AzTrafficManagerEndpointConfig`.

```powershell
$TmProfile = New-AzTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
$webapp1 = Get-AzWebApp -Name webapp1
Add-AzTrafficManagerEndpointConfig -EndpointName webapp1ep -TrafficManagerProfile $TmProfile -Type AzureEndpoints -TargetResourceId $webapp1.Id -EndpointStatus Enabled
$webapp2 = Get-AzWebApp -Name webapp2
Add-AzTrafficManagerEndpointConfig -EndpointName webapp2ep -TrafficManagerProfile $TmProfile -Type AzureEndpoints -TargetResourceId $webapp2.Id -EndpointStatus Enabled
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```
### <a name="example-2-adding-a-publicipaddress-endpoint-using-new-aztrafficmanagerendpoint"></a>Ejemplo 2: Adición de un punto de conexión de publicIpAddress mediante `New-AzTrafficManagerEndpoint`

En este ejemplo, se agrega un recurso de dirección IP pública al perfil de Traffic Manager. La dirección IP pública debe tener un nombre DNS configurado y puede enlazarse a la NIC de una máquina virtual o a un equilibrador de carga.

```powershell
$ip = Get-AzPublicIpAddress -Name MyPublicIP -ResourceGroupName MyRG
New-AzTrafficManagerEndpoint -Name MyIpEndpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled
```

## <a name="adding-external-endpoints"></a>Adición de puntos de conexión externos

Administrador de tráfico usa los puntos de conexión externos para dirigir el tráfico a los servicios hospedados fuera de Azure. Al igual que en el caso de los puntos de conexión de Azure, los puntos de conexión externos se pueden agregar mediante `Add-AzTrafficManagerEndpointConfig` seguido de `Set-AzTrafficManagerProfile` o `New-AzTrafficManagerEndpoint`.

Cuando se especifican puntos de conexión externos:

* Se debe especificar el nombre de dominio del punto de conexión con el parámetro Target.
* Si se usa el método de enrutamiento del tráfico "Performance", se necesita "EndpointLocation". De lo contrario, es opcional. El valor debe ser un [nombre de región de Azure válido](https://azure.microsoft.com/regions/).
* Los parámetros "Weight" y "Priority" son opcionales.

### <a name="example-1-adding-external-endpoints-using-add-aztrafficmanagerendpointconfig-and-set-aztrafficmanagerprofile"></a>Ejemplo 1: Adición de puntos de conexión externos mediante `Add-AzTrafficManagerEndpointConfig` y `Set-AzTrafficManagerProfile`

En este ejemplo, se crea un perfil de Traffic Manager, se agregan dos puntos de conexión externos y se confirman los cambios.

```powershell
$TmProfile = New-AzTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName myapp -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
Add-AzTrafficManagerEndpointConfig -EndpointName eu-endpoint -TrafficManagerProfile $TmProfile -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointLocation "North Europe" -EndpointStatus Enabled
Add-AzTrafficManagerEndpointConfig -EndpointName us-endpoint -TrafficManagerProfile $TmProfile -Type ExternalEndpoints -Target app-us.contoso.com -EndpointLocation "Central US" -EndpointStatus Enabled
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```

### <a name="example-2-adding-external-endpoints-using-new-aztrafficmanagerendpoint"></a>Ejemplo 2: Adición de puntos de conexión externos mediante `New-AzTrafficManagerEndpoint`

En este ejemplo, se agrega un punto de conexión externo a un perfil existente. El perfil se especifica mediante los nombres del grupo de recursos y del perfil.

```powershell
New-AzTrafficManagerEndpoint -Name eu-endpoint -ProfileName MyProfile -ResourceGroupName MyRG -Type ExternalEndpoints -Target app-eu.contoso.com -EndpointStatus Enabled
```

## <a name="adding-nested-endpoints"></a>Adición de puntos de conexión "Anidados"

Cada perfil del Administrador de tráfico especifica un único método de enrutamiento del tráfico. No obstante, hay escenarios en que se requiere un enrutamiento del tráfico más sofisticado que el proporcionado mediante un único perfil de Traffic Manager. Puede anidar perfiles de Traffic Manager para combinar las ventajas de más de un método de enrutamiento del tráfico. Los perfiles anidados permiten invalidar el comportamiento predeterminado de Traffic Manager para admitir implementaciones de aplicaciones más grandes y complejas. Para ejemplos más detallados, consulte [Perfiles anidados de Traffic Manager](traffic-manager-nested-profiles.md).

Los puntos de conexión anidados se configuran en el perfil primario, utilizando un tipo de punto de conexión específico: 'NestedEndpoints'. Cuando se especifican puntos de conexión anidados:

* El punto de conexión se debe especificar con el parámetro "targetResourceId".
* Si se usa el método de enrutamiento del tráfico "Performance", se necesita "EndpointLocation". De lo contrario, es opcional. El valor debe ser un [nombre de región de Azure válido](https://azure.microsoft.com/regions/).
* Los parámetros Weight y Priority son opcionales en cuanto a los puntos de conexión de Azure.
* El parámetro "MinChildEndpoints" es opcional. El valor predeterminado es 1. Si el número de puntos de conexión disponibles se encuentra por debajo de este umbral, el perfil primario considera que dicho perfil está "degradado" y desvía el tráfico a los demás puntos de conexión del perfil primario.

### <a name="example-1-adding-nested-endpoints-using-add-aztrafficmanagerendpointconfig-and-set-aztrafficmanagerprofile"></a>Ejemplo 1: Adición de puntos de conexión anidados mediante `Add-AzTrafficManagerEndpointConfig` y `Set-AzTrafficManagerProfile`

En este ejemplo, se crean un perfil primario y otro secundario de Traffic Manager, se agrega el secundario como punto de conexión anidado al primario y se confirman los cambios.

```powershell
$child = New-AzTrafficManagerProfile -Name child -ResourceGroupName MyRG -TrafficRoutingMethod Priority -RelativeDnsName child -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
$parent = New-AzTrafficManagerProfile -Name parent -ResourceGroupName MyRG -TrafficRoutingMethod Performance -RelativeDnsName parent -Ttl 30 -MonitorProtocol HTTP -MonitorPort 80 -MonitorPath "/"
Add-AzTrafficManagerEndpointConfig -EndpointName child-endpoint -TrafficManagerProfile $parent -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "North Europe" -MinChildEndpoints 2
Set-AzTrafficManagerProfile -TrafficManagerProfile $parent
```

Por brevedad, en este ejemplo, no se han agregado otros puntos de conexión al perfil primario ni al secundario.

### <a name="example-2-adding-nested-endpoints-using-new-aztrafficmanagerendpoint"></a>Ejemplo 2: Adición de puntos de conexión anidados mediante `New-AzTrafficManagerEndpoint`

En este ejemplo, agregamos un perfil secundario existente como punto de conexión anidado a un perfil primario existente. El perfil se especifica mediante los nombres del grupo de recursos y del perfil.

```powershell
$child = Get-AzTrafficManagerEndpoint -Name child -ResourceGroupName MyRG
New-AzTrafficManagerEndpoint -Name child-endpoint -ProfileName parent -ResourceGroupName MyRG -Type NestedEndpoints -TargetResourceId $child.Id -EndpointStatus Enabled -EndpointLocation "North Europe" -MinChildEndpoints 2
```

## <a name="adding-endpoints-from-another-subscription"></a>Adición de puntos de conexión desde otra suscripción

Traffic Manager puede trabajar con puntos de conexión procedentes de distintas suscripciones. Debe cambiar a la suscripción con el punto de conexión que se va a agregar para recuperar la entrada necesaria para Traffic Manager. A continuación, debe cambiar a la suscripción con el perfil de Traffic Manager y agregarle el punto de conexión. En el ejemplo siguiente se muestra cómo llevar a cabo este procedimiento con una dirección IP pública.

```powershell
Set-AzContext -SubscriptionId $EndpointSubscription
$ip = Get-AzPublicIpAddress -Name $IpAddressName -ResourceGroupName $EndpointRG

Set-AzContext -SubscriptionId $trafficmanagerSubscription
New-AzTrafficManagerEndpoint -Name $EndpointName -ProfileName $ProfileName -ResourceGroupName $TrafficManagerRG -Type AzureEndpoints -TargetResourceId $ip.Id -EndpointStatus Enabled
```

## <a name="update-a-traffic-manager-endpoint"></a>Actualización de un punto de conexión de Administrador de tráfico

Hay dos maneras de actualizar un punto de conexión del Administrador de tráfico existente:

1. Obtenga el perfil de Traffic Manager mediante `Get-AzTrafficManagerProfile`, actualice las propiedades de punto de conexión en el perfil y confirme los cambios con `Set-AzTrafficManagerProfile`. Este método tiene la ventaja de poder actualizar más de un punto de conexión en una sola operación.
2. Obtenga el perfil de Traffic Manager mediante `Get-AzTrafficManagerEndpoint`, actualice las propiedades de punto de conexión y confirme los cambios con `Set-AzTrafficManagerEndpoint`. Este método es más sencillo, ya que no requiere la indexación de la matriz de puntos de conexión en el perfil.

### <a name="example-1-updating-endpoints-using-get-aztrafficmanagerprofile-and-set-aztrafficmanagerprofile"></a>Ejemplo 1: Actualización de puntos de conexión mediante `Get-AzTrafficManagerProfile` y `Set-AzTrafficManagerProfile`

En este ejemplo, se va a modificar la prioridad de dos puntos de conexión en un perfil existente.

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name myprofile -ResourceGroupName MyRG
$TmProfile.Endpoints[0].Priority = 2
$TmProfile.Endpoints[1].Priority = 1
Set-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile
```

### <a name="example-2-updating-an-endpoint-using-get-aztrafficmanagerendpoint-and-set-aztrafficmanagerendpoint"></a>Ejemplo 2: Actualización de un punto de conexión mediante `Get-AzTrafficManagerEndpoint` y `Set-AzTrafficManagerEndpoint`

En este ejemplo, se modifica la ponderación de un solo punto de conexión en un perfil existente.

```powershell
$endpoint = Get-AzTrafficManagerEndpoint -Name myendpoint -ProfileName myprofile -ResourceGroupName MyRG -Type ExternalEndpoints
$endpoint.Weight = 20
Set-AzTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

## <a name="enabling-and-disabling-endpoints-and-profiles"></a>Habilitación y deshabilitación de puntos de conexión y perfiles

Administrador de tráfico permite que se habiliten y deshabiliten puntos de conexión individuales, además de permitir la habilitación y deshabilitación de perfiles completos.
Estos cambios pueden realizarse mediante la obtención, actualización o configuración los recursos del perfil o del punto de conexión. Para facilitar estas operaciones comunes, también se admiten a través de cmdlets dedicados.

### <a name="example-1-enabling-and-disabling-a-traffic-manager-profile"></a>Ejemplo 1: Habilitación y deshabilitación de un perfil de Traffic Manager

Para habilitar un perfil de Traffic Manager, use `Enable-AzTrafficManagerProfile`. El perfil se puede especificar mediante un objeto de perfil. El objeto de perfil se puede pasar a través de la canalización o mediante el parámetro "-TrafficManagerProfile". En este ejemplo, se especifica el perfil mediante el nombre del grupo de recursos y del perfil.

```powershell
Enable-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup
```

Para deshabilitar un perfil de Traffic Manager:

```powershell
Disable-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyResourceGroup
```

El cmdlet Disable-AzTrafficManagerProfile solicita confirmación. Se puede suprimir este mensaje con el parámetro "-Force".

### <a name="example-2-enabling-and-disabling-a-traffic-manager-endpoint"></a>Ejemplo 2: Habilitación y deshabilitación de un punto de conexión de Traffic Manager

Para habilitar un punto de conexión de Traffic Manager, use `Enable-AzTrafficManagerEndpoint`. Hay dos maneras de especificar el punto de conexión.

1. Mediante un objeto TrafficManagerEndpoint pasado a través de la canalización o con el parámetro "-TrafficManagerEndpoint"
2. Mediante el nombre del punto de conexión, el tipo de punto de conexión, el nombre del perfil y el nombre del grupo de recursos:

```powershell
Enable-AzTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG
```

De forma similar, para deshabilitar un punto de conexión de Administrador de tráfico:

```powershell
Disable-AzTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG -Force
```

Al igual que con `Disable-AzTrafficManagerProfile`, el cmdlet `Disable-AzTrafficManagerEndpoint` solicita confirmación. Se puede suprimir este mensaje con el parámetro "-Force".

## <a name="delete-a-traffic-manager-endpoint"></a>Eliminación de un punto de conexión de Administrador de tráfico

Para quitar puntos de conexión individuales, use el cmdlet `Remove-AzTrafficManagerEndpoint`:

```powershell
Remove-AzTrafficManagerEndpoint -Name MyEndpoint -Type AzureEndpoints -ProfileName MyProfile -ResourceGroupName MyRG
```

Este cmdlet solicita confirmación. Se puede suprimir este mensaje con el parámetro "-Force".

## <a name="delete-a-traffic-manager-profile"></a>Eliminación de un perfil del Administrador de tráfico

Para eliminar un perfil de Traffic Manager, use el cmdlet `Remove-AzTrafficManagerProfile`, especificando el nombre del perfil y el del grupo de recursos:

```powershell
Remove-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG [-Force]
```

Este cmdlet solicita confirmación. Se puede suprimir este mensaje con el parámetro "-Force".

El perfil que se va a eliminar también se puede especificar con un objeto de perfil:

```powershell
$TmProfile = Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG
Remove-AzTrafficManagerProfile -TrafficManagerProfile $TmProfile [-Force]
```

Se puede canalizar igualmente esta secuencia:

```powershell
Get-AzTrafficManagerProfile -Name MyProfile -ResourceGroupName MyRG | Remove-AzTrafficManagerProfile [-Force]
```

## <a name="next-steps"></a>Pasos siguientes

[Supervisión del Administrador de tráfico](traffic-manager-monitoring.md)

[Consideraciones de rendimiento sobre el Administrador de tráfico](traffic-manager-performance-considerations.md)
