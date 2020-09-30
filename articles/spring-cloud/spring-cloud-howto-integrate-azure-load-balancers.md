---
title: Tutorial; integración de Azure Spring Cloud con soluciones de equilibrio de carga de Azure
description: Integración de Azure Spring Cloud con soluciones de equilibrio de carga de Azure
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 04/20/2020
ms.custom: devx-track-java
ms.openlocfilehash: 02772f153cdda7e3f3c866c727d589e755e19033
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906937"
---
# <a name="integrate-azure-spring-cloud-with-azure-load-balance-solutions"></a>Integración de Azure Spring Cloud con soluciones de equilibrio de carga de Azure

**Este artículo se aplica a:** ✔️ Java ✔️ C#

Azure Spring Cloud admite microservicios en Azure.  El aumento del negocio puede requerir varios centros de datos con administración de varias instancias de Azure Spring Cloud.

Azure ya proporciona diferentes soluciones de equilibrio de carga. Existen tres opciones para integrar Azure Spring Cloud con soluciones de equilibrio de carga de Azure:

1.  Integración de Azure Spring Cloud con Azure Traffic Manager
2.  Integración de Azure Spring Cloud con Azure App Gateway
3.  Integración de Azure Spring Cloud con Azure Front Door

## <a name="prerequisites"></a>Prerrequisitos

* Azure Spring Cloud: [creación de un servicio Azure Spring Cloud](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart)
* Azure Traffic Manager: [creación de una instancia de Traffic Manager](https://docs.microsoft.com/azure/traffic-manager/quickstart-create-traffic-manager-profile/)
* Azure App Gateway: [creación de una instancia de Application Gateway](https://docs.microsoft.com/azure/application-gateway/quick-create-portal)
* Azure Front Door: [creación de una instancia de Front Door](https://docs.microsoft.com/azure/frontdoor/quickstart-create-front-door)

## <a name="integrate-azure-spring-cloud-with-azure-traffic-manager"></a>Integración de Azure Spring Cloud con Azure Traffic Manager

Para integrar Azure Spring Cloud con Traffic Manager, agregue sus puntos de conexión públicos como puntos de conexión de Traffic Manager y, a continuación, configure el dominio personalizado para Traffic Manager y Azure Spring Cloud.

### <a name="add-endpoint-in-traffic-manager"></a>Agregar punto de conexión en Traffic Manager
Agregue puntos de conexión en Traffic Manager:
1.  especifique el **Tipo** como *Punto de conexión externo*.
1.  Escriba el nombre de dominio completo (FQDN) de cada punto de conexión público de Azure Spring Cloud.
1. Haga clic en **OK**.

    ![Traffic Manager 1](media/spring-cloud-load-balancers/traffic-manager-1.png) ![Traffic Manager 2](media/spring-cloud-load-balancers/traffic-manager-2.png)

### <a name="configure-custom-domain"></a>Configuración de un dominio personalizado
Para finalizar la configuración:
1.  inicie sesión en el sitio web de su proveedor de dominios y cree una asignación de registros CNAME desde su dominio personalizado al nombre de dominio predeterminado de Azure de Traffic Manager.
1.  Siga las instrucciones sobre [cómo agregar un dominio personalizado a Azure Spring Cloud](spring-cloud-tutorial-custom-domain.md).
1. Agregue el enlace de dominio personalizado anterior a Traffic Manager a la instancia de App Service correspondiente de Azure Spring Cloud y cargue ahí el certificado.

    ![Traffic Manager 3](media/spring-cloud-load-balancers/traffic-manager-3.png)

## <a name="integrate-azure-spring-cloud-with-azure-app-gateway"></a>Integración de Azure Spring Cloud con Azure App Gateway

Para la integración con el servicio Azure Spring Cloud, complete las siguientes configuraciones:

### <a name="configure-backend-pool"></a>Configuración del grupo de back-end
1. Especifique **Tipo de destino** como *Dirección IP* o *FQDN*.
1. Escriba los puntos de conexión públicos de Azure Spring Cloud.

    ![App Gateway 1](media/spring-cloud-load-balancers/app-gateway-1.png)

### <a name="add-custom-probe"></a>Adición de sondeo personalizado
1. Seleccione **Sondeos de estado** y luego **Agregar** para abrir el cuadro de diálogo del **Sondeo** personalizado. 
1. El punto clave es seleccionar *Sí* para la opción **Seleccionar el nombre de host de la configuración de HTTP de back-end**.

    ![App Gateway 2](media/spring-cloud-load-balancers/app-gateway-2.png)

### <a name="configure-http-setting"></a>Configuración de HTTP
1.  Seleccione **Configuración de HTTP** y después **Agregar** para agregar una configuración de HTTP.
1.  **Reemplazar por un nuevo nombre de host**: seleccione *Sí*.
1.  **Reemplazo del nombre de host**: seleccione **Seleccionar el nombre de host del destino de back-end**.
1.  **Usar sondeo personalizado**: seleccione *Sí* y seleccione el sondeo personalizado creado anteriormente.

    ![App Gateway 3](media/spring-cloud-load-balancers/app-gateway-3.png)

## <a name="integrate-azure-spring-cloud-with-azure-front-door"></a>Integración de Azure Spring Cloud con Azure Front Door

Para integrar con el servicio Azure Spring Cloud y configurar el grupo de back-end, hay que realizar las acciones siguientes: 
1. **Agregar grupo back-end**.
1. Especifique el punto de conexión de back-end agregando el host.

    ![Front Door 1](media/spring-cloud-load-balancers/front-door-1.png)

1.  Especifique **Tipo de host de back-end** como *Host personalizado*.
1.  Escriba el FQDN de los puntos de conexión públicos de Azure Spring Cloud en **Nombre de host de back-end**.
1.  Acepte el **encabezado de host de back-end**, que es el mismo que el **nombre de host de back-end**.

    ![Front Door 2](media/spring-cloud-load-balancers/front-door-2.png)

## <a name="next-steps"></a>Pasos siguientes
* [Creación de un perfil de Traffic Manager](https://docs.microsoft.com/azure/traffic-manager/quickstart-create-traffic-manager-profile/)
* [Creación de una instancia de Application Gateway](https://docs.microsoft.com/azure/application-gateway/quick-create-portal)
* [Creación de una instancia de Front Door](https://docs.microsoft.com/azure/frontdoor/quickstart-create-front-door)
