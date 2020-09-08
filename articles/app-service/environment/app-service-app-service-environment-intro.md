---
title: Introducción a App Service Environment v1
description: Más información sobre las características de App Service Environment v1. Este documento solo se proporciona para los clientes que usan App Service Environment v1 heredado.
author: stefsch
ms.assetid: 78e6d4f5-da46-4eb5-a632-b5fdc17d2394
ms.topic: article
ms.date: 07/11/2017
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 878cef39ade0487c03dba83c9e3631fe3c72088f
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88962390"
---
# <a name="introduction-to-app-service-environment-v1"></a>Introducción a App Service Environment v1

> [!NOTE]
> Este artículo trata sobre App Service Environment v1.  Hay una versión más reciente de App Service Environment que resulta más fácil de usar y se ejecuta en una infraestructura más eficaz. Para aprender más sobre la nueva versión, consulte [Introducción a App Service Environment](intro.md).

## <a name="overview"></a>Información general

App Service Environment es una opción del plan de servicio [Premium][PremiumTier] de [Azure App Service](../overview.md) que proporciona un entorno plenamente aislado y dedicado para ejecutar de forma segura aplicaciones de Azure App Service a gran escala, lo que incluye Web Apps, Mobile Apps y API Apps.  

Los entornos de App Service son ideales para cargas de trabajo de aplicaciones que requieren:

* Muy grande escala
* Aislamiento y acceso a redes seguro

Los clientes pueden crear varios entornos de App Service en una o varias regiones de Azure.  Esto hace que sean perfectos para los niveles de aplicación sin estado de escalado horizontal en el respaldo de cargas de trabajo RPS elevadas.

Los entornos del Servicios de aplicaciones están aislados para ejecutar únicamente las aplicaciones de un solo cliente, y siempre se implementan en una red virtual.  Los clientes tienen un mayor control sobre el tráfico de red entrante y saliente de las aplicaciones, y las aplicaciones pueden establecer conexiones seguras de alta velocidad a los recursos corporativos locales a través de redes virtuales.

Para información general de cómo App Service Environment permite acceso de red seguro y a gran escala, consulte el [vídeo de profundización de AzureCon][AzureConDeepDive] sobre App Service Environment.

Para profundizar en el escalado horizontal con varias instancias de App Service Environment, consulte el artículo sobre cómo configurar una [superficie de aplicaciones con distribución geográfica][GeodistributedAppFootprint].

Para ver cómo se ha configurado la arquitectura de seguridad mostrada en la inmersión en AzureCon, consulte el artículo sobre la implementación de una [arquitectura de seguridad en capas](app-service-app-service-environment-layered-security.md) con entornos de App Service.

Las aplicaciones que se ejecutan en entornos de aplicación de servicio pueden tener su acceso validado por dispositivos de subida como firewalls de aplicación web (WAF).  En el artículo sobre la [configuración de un WAF para entornos de aplicación de servicio](app-service-app-service-environment-web-application-firewall.md) se trata este escenario.

[!INCLUDE [app-service-web-to-api-and-mobile](../../../includes/app-service-web-to-api-and-mobile.md)]

## <a name="dedicated-compute-resources"></a>Recursos de proceso dedicados

Todos los recursos de proceso de un entorno de App Service están dedicados exclusivamente a una sola suscripción, y se puede configurar un entorno de App Service con hasta cincuenta recursos de proceso (50) para uso exclusivo de una única aplicación.

Un entorno de este tipo se compone de un grupo de recursos de proceso front-end y de entre uno a tres grupos de recursos de proceso de trabajo.

El grupo de servidores front-end contiene los recursos de proceso responsables de la terminación TLS y del equilibrio de carga automático de las solicitudes de aplicación dentro de un entorno de App Service.

Cada grupo de trabajo contiene recursos de proceso asignados a [planes de App Service][AppServicePlan] que, a su vez, contienen una o más aplicaciones de Azure App Service.  Como en un entorno de App Service puede haber hasta tres grupos de trabajo distintos, tiene la flexibilidad de elegir diferentes recursos de proceso para cada grupo de trabajo.  

Esto permite, por ejemplo, crear un grupo de trabajo con recursos de proceso de menor eficacia para los planes de App Service destinados a aplicaciones de prueba o desarrollo.  Un segundo (o incluso tercer) grupo de trabajo podría usar recursos de proceso más eficaces en los planes de App Service que ejecuten aplicaciones de producción.

Para más información sobre la cantidad de recursos de proceso disponibles para los grupos front-end y de trabajo, consulte [Configuración de una instancia de App Service Environment][HowToConfigureanAppServiceEnvironment].  

Para más información sobre los tamaños de los recursos de proceso disponibles admitidos en App Service Environment, consulte la página [Precios de App Service][AppServicePricing] y revise las opciones que existen para este tipo de entornos en el plan de tarifa Premium.

## <a name="virtual-network-support"></a>Compatibilidad con redes virtuales

Se puede crear una instancia de App Service Environment **o bien** en una red virtual de Azure Resource Manager **o** en una del modelo de implementación clásica ([más información sobre las redes virtuales][MoreInfoOnVirtualNetworks]).  Puesto que los entornos de este tipo residen siempre en una red virtual y, más concretamente, en una subred de una red virtual, puede aprovechar las características de seguridad de las redes virtuales para controlar las comunicaciones de red entrantes y salientes.  

Una instancia de App Service Environment puede tener conexión a Internet con una dirección IP pública, o tener una conexión interna solo con la dirección de un equilibrador de carga interno (ILB) de Azure.

Puede usar [grupos de seguridad de red][NetworkSecurityGroups] para restringir las comunicaciones de red entrantes a la subred donde reside una instancia de App Service Environment.  Esto le permite ejecutar aplicaciones tras dispositivos y servicios ascendentes, como firewalls de aplicaciones web y proveedores de SaaS de red.

Las aplicaciones, además, suelen requerir acceso a recursos corporativos, como bases de datos internas y servicios web.  Un enfoque común consiste en poner estos extremos a disposición únicamente del flujo de tráfico de red interno de una red virtual de Azure.  Una vez que App Service Environment se une a la misma red virtual que los servicios internos, las aplicaciones que se ejecutan en el entorno pueden tener acceso a ellos, incluidos los puntos de conexión accesibles mediante conexiones [De sitio a sitio][SiteToSite] y [Azure ExpressRoute][ExpressRoute].

Para más información sobre cómo funciona App Service Environment con redes virtuales y locales, consulte los siguientes artículos sobre [Arquitectura de red][NetworkArchitectureOverview], [Control del tráfico de entrada][ControllingInboundTraffic] y [Protección de la conexión a los servidores back-end][SecurelyConnectingToBackends]. 

## <a name="getting-started"></a>Introducción

Para empezar a trabajar con App Service Environment, consulte [Creación de una instancia de App Service Environment][HowToCreateAnAppServiceEnvironment].

Para información general sobre la arquitectura de red de App Service Environment, consulte el artículo [Información general sobre la arquitectura de red][NetworkArchitectureOverview].

Para más información sobre el uso de App Service Environment con ExpressRoute, consulte el artículo [ExpressRoute y App Service Environment][NetworkConfigDetailsForExpressRoute].

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[PremiumTier]: https://azure.microsoft.com/pricing/details/app-service/
[MoreInfoOnVirtualNetworks]: ../../virtual-network/virtual-networks-faq.md
[AppServicePlan]: ../overview-hosting-plans.md
[HowToCreateAnAppServiceEnvironment]: app-service-web-how-to-create-an-app-service-environment.md
[LogicApps]: ../../logic-apps/logic-apps-overview.md
[AzureConDeepDive]:  https://azure.microsoft.com/documentation/videos/azurecon-2015-deploying-highly-scalable-and-secure-web-and-mobile-apps/
[GeodistributedAppFootprint]:  app-service-app-service-environment-geo-distributed-scale.md
[NetworkSecurityGroups]: ../../virtual-network/virtual-network-vnet-plan-design-arm.md
[SiteToSite]: ../../vpn-gateway/vpn-gateway-multi-site.md
[ExpressRoute]: https://azure.microsoft.com/services/expressroute/
[HowToConfigureanAppServiceEnvironment]:  app-service-web-configure-an-app-service-environment.md
[ControllingInboundTraffic]:  app-service-app-service-environment-control-inbound-traffic.md
[SecurelyConnectingToBackends]:  app-service-app-service-environment-securely-connecting-to-backend-resources.md
[NetworkArchitectureOverview]:  app-service-app-service-environment-network-architecture-overview.md
[NetworkConfigDetailsForExpressRoute]:  app-service-app-service-environment-network-configuration-expressroute.md
[AppServicePricing]: https://azure.microsoft.com/pricing/details/app-service/ 

<!-- IMAGES -->