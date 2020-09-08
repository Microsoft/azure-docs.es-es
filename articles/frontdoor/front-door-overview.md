---
title: Azure Front Door | Microsoft Docs
description: En este artículo se ofrece información general sobre Azure Front Door. Averigüe si es la elección correcta para equilibrar la carga de tráfico de usuario de la aplicación.
services: frontdoor
documentationcenter: ''
author: duongau
editor: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/02/2020
ms.author: duau
ms.openlocfilehash: 003903a941b0d9ce36f28ce5e4d640e5746a7de3
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89378192"
---
# <a name="what-is-azure-front-door"></a>¿Qué es Azure Front Door?
Azure Front Door permite definir, administrar y supervisar el enrutamiento global para el tráfico web mediante la optimización para obtener el mejor rendimiento y la conmutación por error global rápida para alta disponibilidad. Con Front Door, las aplicaciones empresariales y de consumidor globales (de varias regiones) se pueden transformar en aplicaciones modernas personalizadas, sólidas y de alto rendimiento, API y contenido que lleguen a un público global mediante Azure.

Front Door funciona en la capa 7 o la capa HTTP/HTTPS, y usa el protocolo de difusión por proximidad con división TCP y la red global de Microsoft para mejorar la conectividad global. Por tanto, según la selección del método de enrutamiento en la configuración, puede asegurarse de que Front Door enruta las solicitudes de cliente al back-end de aplicación más rápido y disponible. Un back-end de aplicación es cualquier servicio accesible desde Internet hospedado dentro o fuera de Azure. Front Door proporciona una serie de [métodos de enrutamiento del tráfico](front-door-routing-methods.md) y [opciones de seguimiento de estado del back-end](front-door-health-probes.md) para satisfacer las distintas necesidades de las aplicaciones y los modelos de conmutación automática por error. Al igual que [Traffic Manager](../traffic-manager/traffic-manager-overview.md), Front Door es resistente a errores, incluidos los que afectan a una región completa de Azure.

>[!NOTE]
> Azure ofrece un conjunto de soluciones de equilibrio de carga completamente administradas para sus escenarios. Si busca un enrutamiento global basado en DNS y **no** tiene requisitos de finalización con el protocolo de seguridad de la capa de transporte (TLS) ("descarga SSL") o de procesamiento de niveles de aplicación por solicitud HTTP/HTTPS, revise [Traffic Manager](../traffic-manager/traffic-manager-overview.md). Si busca el equilibrio de carga entre los servidores de una región, para la capa de aplicación, revise [Application Gateway](../application-gateway/application-gateway-introduction.md) y para el equilibrio de carga del nivel de red, revise [Load Balancer](../load-balancer/load-balancer-overview.md). Sus escenarios integrales pueden beneficiarse de la combinación de estas soluciones según sea necesario.
>
> Si desea ver una comparación de las distintas opciones de equilibrio de carga de Azure, consulte [Información general sobre las opciones de equilibrio de carga en Azure](https://docs.microsoft.com/azure/architecture/guide/technology-choices/load-balancing-overview).

Las características siguientes se incluyen con Front Door:

## <a name="accelerate-application-performance"></a>Aceleración del rendimiento de las aplicaciones
Mediante el protocolo de difusión por proximidad basado en división TCP, Front Door garantiza que los usuarios finales se conecten inmediatamente al punto de presencia (POP) de Front Door más cercano. Con la red global de Microsoft para conectarse a los back-ends de aplicación desde puntos de presencia de Front Door, se asegura una mayor disponibilidad y confiabilidad al tiempo que se mantiene el rendimiento. Esta conectividad con el back-end también se basa en una latencia de red menor. Obtenga más información sobre las técnicas de enrutamiento de Front Door, como [División TCP](front-door-routing-architecture.md#splittcp) y el [Protocolo de difusión por proximidad](front-door-routing-architecture.md#anycast).

## <a name="increase-application-availability-with-smart-health-probes"></a>Aumento de la disponibilidad de la aplicación con sondeos de estado inteligentes

Front Door ofrece alta disponibilidad para las aplicaciones críticas mediante sus sondeos de estado inteligentes, la supervisión de la latencia y disponibilidad de los back-ends, y proporcionando conmutación automática por error rápida cuando un back-end deja de funcionar. Por tanto, puede realizar operaciones de mantenimiento planeado en las aplicaciones sin tiempo de inactividad. Front Door dirige el tráfico a back-ends alternativos mientras el mantenimiento está en curso.

## <a name="url-based-routing"></a>Enrutamiento basado en dirección URL
El enrutamiento basado en dirección URL permite enrutar el tráfico a los grupos de back-end en función de las rutas de acceso de dirección URL de la solicitud. Uno de los escenarios es el enrutamiento de solicitudes de diferentes tipos de contenido a diferentes grupos de back-end.

Por ejemplo, las solicitudes para `http://www.contoso.com/users/*` se enrutan a UserProfilePool y para `http://www.contoso.com/products/*` se enrutan a ProductInventoryPool.  Front Door permite incluso escenarios de comparación de rutas más complejos mediante el algoritmo de mejor coincidencia de modo que, si ninguno de los patrones de ruta de acceso coincide, se selecciona la regla de enrutamiento predeterminada para `http://www.contoso.com/*` y el tráfico se dirige a la regla de enrutamiento predeterminada de comodín. Obtenga más información en [Coincidencia de rutas](front-door-route-matching.md).

## <a name="multiple-site-hosting"></a>Hospedaje de varios sitios
El hospedaje de varios sitios permite configurar más de un sitio web en la misma configuración de Front Door. Esta característica permite configurar una topología más eficaz para las implementaciones al agregar diferentes sitios web a una misma configuración de Front Door. Según la arquitectura de la aplicación, puede configurar Azure Front Door para dirigir cada sitio web a su propio grupo de back-end o dirigir varios sitios web al mismo grupo de back-end. Por ejemplo, Front Door puede atender el tráfico para `images.contoso.com` y `videos.contoso.com` desde dos grupos de back-end llamados ImagePool y VideoPool. Como alternativa, los dos hosts de front-end se pueden configurar para dirigir el tráfico a un único grupo de back-end denominado MediaPool.

De forma similar, se pueden configurar dos dominios diferentes `www.contoso.com` y `www.fabrikam.com` en la misma instancia de Front Door.

## <a name="session-affinity"></a>Afinidad de sesión
La característica de afinidad de sesión basada en cookies es útil cuando se quiere mantener una sesión de usuario en el back-end de aplicación. Mediante las cookies administradas de Front Door, el tráfico posterior desde una sesión de usuario se dirige al mismo back-end de aplicación para su procesamiento. Esta característica es importante en aquellos casos en los que se guarda el estado de sesión de forma local en el back-end para una sesión de usuario.

## <a name="tls-termination"></a>Finalización de TLS
Front Door admite finalización TLS en el perímetro, es decir, los usuarios individuales pueden configurar una conexión TLS con entornos de Front Door en lugar de hacerlo a través de conexiones de largo alcance con el back-end de aplicación. Además, Front Door admite conectividad HTTP y HTTPS entre los entornos de Front Door y los back-ends. Por tanto, también se puede configurar el cifrado TLS de un extremo a otro. Por ejemplo, si Front Door para la carga de trabajo de la aplicación recibe más de 5 000 solicitudes en un minuto, debido a la reutilización de la conexión semiactiva, para los servicios activos solo establecerá, por ejemplo, unas 500 conexiones con el back-end de aplicación, lo que disminuye una parte significativa de la carga en los back-ends.

## <a name="custom-domains-and-certificate-management"></a>Dominios personalizados y administración de certificados
Cuando se usa Front Door para entregar contenido, se necesita un dominio personalizado si se quiere que el nombre de dominio propio sea visible en la dirección URL de Front Door. El hecho de tener un nombre de dominio visible puede ser cómodo para sus clientes y útil con fines de personalización de marca.
Front Door también admite HTTPS para los nombres de dominio personalizados. Para usar esta característica, elija certificados administrados por Front Door para el tráfico o cargue un certificado TLS/SSL personalizado propio.

## <a name="application-layer-security"></a>Seguridad en el nivel de aplicación
Azure Front Door permite crear reglas de firewall de aplicaciones web (WAF) personalizadas para el control de acceso con el fin de proteger la carga de trabajo HTTP/HTTPS frente a técnicas de explotación basadas en direcciones IP de cliente, código de país y parámetros HTTP. Además, Front Door permite crear reglas de limitación de velocidad para luchar contra el tráfico de bots malintencionados. Para más información sobre el firewall de aplicaciones web, consulte [¿Qué es el firewall de aplicaciones web de Azure?](../web-application-firewall/overview.md)

La propia plataforma Front Door está protegida con [Azure DDoS Protection](../virtual-network/ddos-protection-overview.md) básico. Para lograr una mayor protección, se puede habilitar Azure DDoS Protection estándar en las redes virtuales y proteger los recursos contra ataques de nivel de red (TCP/UDP) a través de la optimización automática y la mitigación. Front Door es un proxy inverso de nivel 7, solo permite que el tráfico web pase a los back-ends y bloquea otros tipos de tráfico de forma predeterminada.

## <a name="url-redirection"></a>Redirección de URL
Con la enorme presión en el sector de respaldar solamente la comunicación segura, se espera que las aplicaciones web redirijan automáticamente todo el tráfico HTTP a HTTPS. De esta forma, se garantiza que todas las comunicaciones entre los usuarios y la aplicación se realizan a través de una ruta de acceso cifrada. 

Tradicionalmente, los propietarios de aplicaciones han hecho frente a este requisito mediante la creación de un servicio dedicado, cuya única finalidad era redirigir las solicitudes que recibe en HTTP a HTTPS. Azure Front Door ofrece la posibilidad de redirigir el tráfico de HTTP a HTTPS. Esto simplifica la configuración de la aplicación, optimiza el uso de recursos y admite nuevos escenarios de redireccionamiento que incluyen el global y el redireccionamiento basado en la ruta de acceso. La redirección de URL desde Azure Front Door no se limita solo a la redirección de HTTP a HTTPS, sino también a la redirección a un nombre de host diferente, a una ruta de acceso diferente e, incluso, a una nueva cadena de consulta de la dirección URL.

Para más información, consulte cómo [redirigir el tráfico](front-door-url-redirect.md) con Azure Front Door.

## <a name="url-rewrite"></a>Reescritura de direcciones URL
Front Door admite la [reescritura de dirección URL](front-door-url-rewrite.md) al permitirle configurar una ruta de acceso de reenvío personalizada opcional que se usará al construir la solicitud para reenviar al back-end. Además, Front Door permite configurar el encabezado de Host para que se envíe al reenviar la solicitud al back-end.

## <a name="protocol-support---ipv6-and-http2-traffic"></a>Protocolos admitidos: IPv6 y tráfico HTTP/2
De forma nativa, Azure Front Door admite conectividad de IPv6-de un extremo a otro y también el protocolo HTTP/2. 

El protocolo HTTP/2 permite una comunicación dúplex completa entre los back-ends de aplicación y un cliente a través de una conexión TCP de larga duración. HTTP/2 permite una comunicación más interactiva entre el back-end y el cliente, que puede ser bidireccional sin necesidad de realizar sondeos como en las implementaciones basadas en HTTP. A diferencia de HTTP, el protocolo HTTP/2 tiene poca sobrecarga y puede reutilizar la misma conexión TCP para varias solicitudes o respuestas, lo que conlleva un uso más eficaz de los recursos. Obtenga más información sobre la [compatibilidad con HTTP/2 en Azure Front Door](front-door-http2.md).

## <a name="pricing"></a>Precios

Para obtener información sobre los precios, vea [Precios de Front Door](https://azure.microsoft.com/pricing/details/frontdoor/).

## <a name="whats-new"></a>Novedades

Suscríbase a la fuente RSS y vea las actualizaciones más recientes de las características de Azure Load Balancer en la página [Actualizaciones de Azure](https://azure.microsoft.com/updates/?category=networking&query=Azure%20Front%20Door).

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear una instancia de Front Door](quickstart-create-front-door.md).
- Más información acerca de cómo [funciona Front Door](front-door-routing-architecture.md).
