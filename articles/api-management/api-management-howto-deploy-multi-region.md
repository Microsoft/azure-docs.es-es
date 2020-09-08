---
title: Implementación de servicios de Azure API Management en varias regiones de Azure
titleSuffix: Azure API Management
description: Aprenda a implementar una instancia del servicio Azure API Management en varias regiones de Azure.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 04/20/2020
ms.author: apimpm
ms.openlocfilehash: 427ebfe865002612be2f9aeb9db416f5c2f41e52
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88065461"
---
# <a name="how-to-deploy-an-azure-api-management-service-instance-to-multiple-azure-regions"></a>Implementación de una instancia del servicio Azure API Management en varias regiones de Azure

Azure API Management admite la implementación en varias regiones, lo que permite a los publicadores de API distribuir un único servicio Azure API Management en el número de regiones de Azure admitidas. La característica multirregional ayuda a reducir la latencia de solicitud que perciben los usuarios de API distribuidos geográficamente, y mejora la disponibilidad del servicio en caso de que una región se quede sin conexión.

Inicialmente, un nuevo servicio Azure API Management contiene solo una [unidad][unit] en una única región de Azure, la región primaria. Se pueden agregar unidades adicionales a las regiones primaria o secundarias. Un componente de puerta de enlace de API Management se implementa en cada región primaria y secundaria seleccionada. Las solicitudes de la API entrantes se dirigen automáticamente a la región más cercana. Si una región se queda sin conexión, las solicitudes de la API se enrutarán automáticamente para evitar la región con errores hacia la siguiente puerta de enlace más cercana.

> [!NOTE]
> Solo el componente de puerta de enlace de API Management se implementa en todas las regiones. El componente de administración de servicios y el portal para desarrolladores se hospedan en la región primaria únicamente. Por lo tanto, en caso de interrupción de la región primaria, el acceso al portal para desarrolladores y la capacidad de cambiar la configuración (por ejemplo, agregar API y aplicar directivas) se verán impedidos hasta que la región primaria vuelva a estar en línea. Mientras la región principal esté sin conexión, las regiones secundarias seguirán atendiendo al tráfico de las API con la configuración más reciente disponible.

>[!IMPORTANT]
> La característica para permitir el almacenamiento de datos de clientes en una única región solo está disponible actualmente en la región Sudeste Asiático (Singapur) de la geoárea Asia Pacífico. En todas las demás regiones, los datos del cliente se almacenan en la geoárea.

[!INCLUDE [premium.md](../../includes/api-management-availability-premium.md)]

## <a name="deploy-api-management-service-to-a-new-region"></a><a name="add-region"> </a>Implementación del servicio API Management en una nueva región

> [!NOTE]
> Si todavía no ha creado una instancia del servicio API Management, consulte [Creación de una instancia del servicio API Management][create an api management service instance].

1. En Azure Portal, vaya al servicio API Management y haga clic en la entrada **Ubicaciones** del menú.
2. Haga clic en **+ Agregar** en la barra superior.
3. Seleccione la ubicación en la lista desplegable y establezca el número de unidades con el control deslizante.
4. Haga clic en el botón **Agregar** para confirmar.
5. Repita este proceso hasta que configure todas las ubicaciones.
6. Haga clic en **Guardar** en la barra superior para iniciar el proceso de implementación.

## <a name="delete-an-api-management-service-location"></a><a name="remove-region"> </a>Eliminación de una ubicación del servicio API Management

1. En Azure Portal, vaya al servicio API Management y haga clic en la entrada **Ubicaciones** del menú.
2. Para la ubicación que desee eliminar, abra el menú contextual mediante el botón **...** situado a la derecha de la tabla. Haga clic en la opción **Eliminar**.
3. Confirme la eliminación y haga clic en **Guardar** para aplicar los cambios.

## <a name="route-api-calls-to-regional-backend-services"></a><a name="route-backend"> </a>Enrutamiento de las llamadas API a servicios regionales back-end

Azure API Management incluye solo una dirección URL del servicio back-end. Aunque hay instancias de Azure API Management en varias regiones, la puerta de enlace de API sigue reenviando las solicitudes al mismo servicio back-end, que se implementa en una única región. En este caso, la ganancia de rendimiento procederá solo de las respuestas en caché a la solicitud en Azure API Management de una región específica, pero también puede generar una latencia alta al ponerse en contacto con el back-end de todo el mundo.

Para aprovechar completamente la distribución geográfica de su sistema, debe tener servicios back-end implementados en las mismas regiones que las instancias de Azure API Management. A continuación, mediante directivas y la propiedad `@(context.Deployment.Region)` puede enrutar el tráfico a las instancias locales de su back-end.

1. Vaya a la instancia de Azure API Management y haga clic en **API** en el menú izquierdo.
2. Seleccione la API que desee.
3. Haga clic en el **Editor de código** de la flecha desplegable de **Procesamiento de entrada**.

    ![Editor de código de API](./media/api-management-howto-deploy-multi-region/api-management-api-code-editor.png)

4. Use `set-backend` en combinación con directivas `choose` condicionales para construir una directiva de enrutamiento adecuada en la sección `<inbound> </inbound>` del archivo.

    Por ejemplo, el siguiente archivo XML sería válido para las regiones Oeste de EE. UU. y Este de Asia:

    ```xml
    <policies>
        <inbound>
            <base />
            <choose>
                <when condition="@("West US".Equals(context.Deployment.Region, StringComparison.OrdinalIgnoreCase))">
                    <set-backend-service base-url="http://contoso-us.com/" />
                </when>
                <when condition="@("East Asia".Equals(context.Deployment.Region, StringComparison.OrdinalIgnoreCase))">
                    <set-backend-service base-url="http://contoso-asia.com/" />
                </when>
                <otherwise>
                    <set-backend-service base-url="http://contoso-other.com/" />
                </otherwise>
            </choose>
        </inbound>
        <backend>
            <base />
        </backend>
        <outbound>
            <base />
        </outbound>
        <on-error>
            <base />
        </on-error>
    </policies>
    ```

> [!TIP]
> También puede adelantar sus servicios de back-end con [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/), dirigir las llamadas de las API a Traffic Manager y dejar que resuelva el enrutamiento automáticamente.

## <a name="use-custom-routing-to-api-management-regional-gateways"></a><a name="custom-routing"> </a>Uso del enrutamiento personalizado a puertas de enlace regionales de API Management

API Management enruta las solicitudes a una _puerta de enlace_ regional en función de la [latencia más baja](../traffic-manager/traffic-manager-routing-methods.md#performance). Aunque no es posible invalidar esta configuración en API Management, puede usar su propia instancia de Traffic Manager con reglas de enrutamiento personalizadas.

1. Cree su propio perfil de [Azure Traffic Manager](https://azure.microsoft.com/services/traffic-manager/).
1. Si usa un dominio personalizado, [úselo con Traffic Manager](../traffic-manager/traffic-manager-point-internet-domain.md) y no con el servicio API Management.
1. [Configuración de los puntos de conexión regionales de API Management en Traffic Manager](../traffic-manager/traffic-manager-manage-endpoints.md). Los puntos de conexión regionales siguen el patrón de dirección URL de `https://<service-name>-<region>-01.regional.azure-api.net`, por ejemplo `https://contoso-westus2-01.regional.azure-api.net`.
1. [Configure los puntos de conexión de estado regional de API Management en Traffic Manager](../traffic-manager/traffic-manager-monitoring.md). Los puntos de conexión de estado regional siguen el patrón de dirección URL de `https://<service-name>-<region>-01.regional.azure-api.net/status-0123456789abcdef`, por ejemplo `https://contoso-westus2-01.regional.azure-api.net/status-0123456789abcdef`.
1. Especifique [el método de enrutamiento](../traffic-manager/traffic-manager-routing-methods.md) de Traffic Manager.

[create an api management service instance]: get-started-create-service-instance.md
[get started with azure api management]: get-started-create-service-instance.md
[deploy an api management service instance to a new region]: #add-region
[delete an api management service instance from a region]: #remove-region
[unit]: https://azure.microsoft.com/pricing/details/api-management/
[premium]: https://azure.microsoft.com/pricing/details/api-management/
