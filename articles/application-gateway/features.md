---
title: Características de Azure Application Gateway
description: Obtenga información sobre las características de Azure Application Gateway.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: conceptual
ms.date: 04/07/2020
ms.author: victorh
ms.openlocfilehash: bbb78fd879bc5c6bb8c2624329a23d7137b11660
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89651989"
---
# <a name="azure-application-gateway-features"></a>Características de Azure Application Gateway

[Azure Application Gateway](overview.md) es un equilibrador de carga de tráfico web que permite administrar el tráfico a las aplicaciones web.

![Conceptos de Application Gateway](media/overview/figure1-720.png)

Application Gateway incluye las siguientes características:

- [Terminación de la Capa de sockets seguros (SSL/TLS)](#secure-sockets-layer-ssltls-termination)
- [Escalabilidad automática](#autoscaling)
- [Redundancia de zona](#zone-redundancy)
- [IP virtual estática](#static-vip)
- [Firewall de aplicaciones web](#web-application-firewall)
- [Controlador de entrada para AKS](#ingress-controller-for-aks)
- [Enrutamiento basado en dirección URL](#url-based-routing)
- [Hospedaje de varios sitios](#multiple-site-hosting)
- [Redireccionamiento](#redirection)
- [Afinidad de sesión](#session-affinity)
- [Tráfico de Websocket y HTTP/2](#websocket-and-http2-traffic)
- [Purga de la conexión](#connection-draining)
- [Páginas de error personalizadas](#custom-error-pages)
- [Reescritura de encabezados HTTP y URL](#rewrite-http-headers-and-url)
- [Ajuste de tamaño](#sizing)

## <a name="secure-sockets-layer-ssltls-termination"></a>Terminación de la Capa de sockets seguros (SSL)

Application Gateway admite la terminación SSL/TLS en la puerta de enlace, tras lo cual el tráfico fluye normalmente sin cifrar a los servidores back-end. Esta característica permite a los servidores web liberarse de la costosa sobrecarga de cifrado y descifrado. Sin embargo, en algunas ocasiones, la comunicación sin cifrar con los servidores no es una opción aceptable. Esto podría deberse a los requisitos de seguridad y cumplimiento normativo, o bien a la posibilidad de que la aplicación solo acepte una conexión segura. Para estas aplicaciones, Application Gateway admite el cifrado SSL/TLS de un extremo a otro.

Para obtener más información, consulte [Introducción a la terminación SSL y a SSL de extremo a extremo con Application Gateway](ssl-overview.md).

## <a name="autoscaling"></a>Escalado automático

La versión Standard_v2 de Application Gateway admite el escalado automático y puede escalarse o reducirse verticalmente en función de los cambiantes patrones de la carga de tráfico. La escalabilidad automática también elimina el requisito de tener elegir un tamaño de implementación o un número de instancias durante el aprovisionamiento. 

Para más información sobre las características de la versión Standard_v2 de Application Gateway, consulte el artículo sobre el [Escalado automático de la versión 2 del SKU](application-gateway-autoscaling-zone-redundant.md).

## <a name="zone-redundancy"></a>Redundancia de zona

La versión Standard_v2 de Application Gateway puede abarcar varias instancias de Availability Zones para ofrecer una mejor resistencia a errores y eliminar la necesidad de aprovisionar las puertas de enlace de aplicaciones independientes en cada zona.

## <a name="static-vip"></a>IP virtual estática

La puerta de enlace de aplicaciones de la versión Standard_v2 del SKU solo admite tipos de IP virtual estáticos. Con esto se garantiza que la IP virtual asociada a la puerta de enlace de aplicaciones no cambia durante la vigencia de Application Gateway.

## <a name="web-application-firewall"></a>Firewall de aplicaciones web

El firewall de aplicaciones web (WAF) es un servicio que ofrece una protección centralizada de las aplicaciones web contra las vulnerabilidades de seguridad más habituales. Firewall de aplicaciones web se basa en las reglas contenidas en el [conjunto de reglas básicas de OWASP (Open Web Application Security Project)](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project) 3.1 (solo WAF_v2), 3.0 y 2.2.9. 

Las aplicaciones web son cada vez más los objetivos de ataques malintencionados que aprovechan vulnerabilidades comunes conocidas, como ataques por inyección de código SQL o ataques de scripts de sitios, por nombrar unos pocos. Impedir tales ataques en el código de aplicación puede ser un verdadero desafío y requerir tareas rigurosas de mantenimiento, aplicación de revisiones y supervisión en varias capas de la topología de aplicación. Un firewall de aplicaciones web centralizado facilita enormemente la administración de la seguridad y proporciona mayor protección a los administradores de la aplicación frente a amenazas o intrusiones. Las soluciones de WAF también pueden reaccionar más rápido ante una amenaza de la seguridad aplicando revisiones que aborden una vulnerabilidad conocida en una ubicación central en lugar de proteger cada una de las aplicaciones web por separado. Las puertas de enlace de aplicaciones existentes pueden transformarse rápidamente en puertas de enlace con un firewall de aplicaciones web habilitado.

Para más información, consulte [¿Qué es el firewall de aplicaciones web?](../web-application-firewall/overview.md)

## <a name="ingress-controller-for-aks"></a>Controlador de entrada para AKS
El controlador de entrada de Application Gateway (AGIC) le permite utilizar Application Gateway como entrada para un clúster de [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/services/kubernetes-service/). 

El controlador de entrada se ejecuta como un pod en el clúster de AKS y consume [recursos de entrada de Kubernetes](https://kubernetes.io/docs/concepts/services-networking/ingress/) y los convierte en una configuración de Application Gateway que permite a la puerta de enlace equilibrar la carga de tráfico en los pods de Kubernetes. El controlador de entrada solo admite las SKU de Application Gateway Standard_v2 y WAF_v2. 

Para más información, consulte [Application Gateway Ingress Controller](ingress-controller-overview.md) (Controlador de entrada de Azure Application Gateway).

## <a name="url-based-routing"></a>Enrutamiento basado en dirección URL

El enrutamiento basado en URL le permite enrutar el tráfico a los grupos de servidores back-end en función de las direcciones URL de la solicitud. Uno de los escenarios podría ser enrutar las solicitudes de diferentes tipos de contenido a otro grupo diferente.

Por ejemplo, las solicitudes para `http://contoso.com/video/*` se enrutan a VideoServerPool y para `http://contoso.com/images/*` se enrutan a ImageServerPool. DefaultServerPool se selecciona si ninguno de los patrones de ruta de acceso coincide.

Para más información, consulte [Información general del enrutamiento basado en URL](url-route-overview.md).

## <a name="multiple-site-hosting"></a>Hospedaje de varios sitios

Con Application Gateway, puede configurar el enrutamiento basado en el nombre de host o el nombre de dominio de más de una aplicación web en la misma puerta de enlace de aplicación. Permite configurar una topología más eficaz para las implementaciones al agregar hasta 100 sitios web a una puerta de enlace de aplicaciones. Cada sitio web se puede dirigir a su propio grupo de back-end. Por ejemplo, tres dominios, contoso.com, fabrikam.com y adatum.com, señalan a la dirección IP de la puerta de enlace de aplicaciones. Crearía tres clientes de escucha multisitio y configuraría cada uno con la configuración respectiva de protocolo y puerto. 

Las solicitudes para `http://contoso.com` se enrutan a ContosoServerPool, y para `http://fabrikam.com` se enrutan a FabrikamServerPool, y así sucesivamente.

De forma similar, dos subdominios del mismo dominio primario pueden hospedarse en la misma implementación de la puerta de enlace de aplicaciones. Ejemplos del uso de subdominios podrían incluir `http://blog.contoso.com` y `http://app.contoso.com` hospedados en una única implementación de la puerta de enlace de aplicaciones. Para más información, consulte [Hospedaje de varios sitios de Application Gateway](multiple-site-overview.md).

También puede definir nombres de host con el carácter comodín en un cliente de escucha de varios sitios y hasta cinco nombres de host por cliente de escucha. Para obtener más información, consulte los [nombres de host comodín en el cliente de escucha (versión preliminar)](multiple-site-overview.md#wildcard-host-names-in-listener-preview).

## <a name="redirection"></a>Redireccionamiento

Un escenario común para muchas aplicaciones web es admitir el redireccionamiento automático de HTTP a HTTPS para asegurarse de que toda la comunicación entre la aplicación y sus usuarios se produce a través de una ruta de acceso cifrada.

En el pasado podría haber utilizado técnicas como la creación de un grupo dedicado cuya única finalidad sería redirigir las solicitudes que recibe de HTTP a HTTPS. Application Gateway admite la posibilidad de redirigir el tráfico en la puerta de enlace de aplicaciones. Esto simplifica la configuración de la aplicación, optimiza el uso de recursos y admite nuevos escenarios de redireccionamiento que incluyen el global y el redireccionamiento basado en la ruta de acceso. La compatibilidad del redireccionamiento de Application Gateway no se limita solo al redireccionamiento de HTTP a HTTPS. Se trata de un mecanismo de redireccionamiento genérico, por lo que puede redirigir desde y hacia cualquier puerto que defina mediante reglas. También admite la redirección a un sitio externo.

La compatibilidad con la redirección de Application Gateway ofrece las siguientes funcionalidades:

- Redireccionamiento global de un puerto a otro puerto en la puerta de enlace. Permite la redirección de HTTP a HTTPS en un sitio.
- Redirección basada en la ruta de acceso. Este tipo de redireccionamiento permite el redireccionamiento de HTTP a HTTPS solamente para un área de un sitio específico, por ejemplo un área de carro de la compra que se indica mediante `/cart/*`.
- Redireccionamiento a un sitio externo

Para más información, consulte [Introducción a la redirección de Application Gateway](redirect-overview.md).

## <a name="session-affinity"></a>Afinidad de sesión

La característica de afinidad de sesión basada en cookies es útil cuando se quiere mantener una sesión de usuario en el mismo servidor. Mediante el uso de cookies administradas por la puerta de enlace, Application Gateway puede dirigir el tráfico posterior de una sesión de usuario hasta el mismo servidor para su procesamiento. Esto es importante en aquellos casos en los que se guarda el estado de la sesión de forma local en el servidor para una sesión de usuario.

Para más información, consulte [Funcionamiento de una puerta de enlace de aplicaciones](how-application-gateway-works.md#modifications-to-the-request).

## <a name="websocket-and-http2-traffic"></a>Tráfico de Websocket y HTTP/2

Application Gateway proporciona compatibilidad nativa con los protocolos Websocket y HTTP/2. No hay ninguna opción de configuración que permita al usuario habilitar o deshabilitar la compatibilidad con WebSocket.

Los protocolos WebSocket y HTTP/2 permiten una comunicación dúplex completa entre un servidor y un cliente a través de una conexión TCP de larga duración. Esto permite una comunicación más interactiva entre el servidor web y el cliente, que puede ser bidireccional sin necesidad de realizar sondeos como en las implementaciones basadas en HTTP. Estos protocolos tienen, a diferencia de HTTP, una sobrecarga reducida y pueden reutilizar la misma conexión TCP para varias solicitudes y respuestas, con lo que se utilizan los recursos de una manera más eficaz. Estos protocolos están diseñados para utilizarse a través de los puertos HTTP tradicionales 80 y 443.

Para más información, consulte [Compatibilidad con WebSocket
](application-gateway-websocket.md) y [Compatibilidad con HTTP/2](configuration-listeners.md#http2-support).

## <a name="connection-draining"></a>Purga de la conexión

La purga de conexión ayuda a lograr la correcta eliminación de miembros del grupo de back-end durante las actualizaciones de servicio planeadas. Esta configuración se habilita a través de la configuración de http de back-end y se puede aplicar a todos los miembros de un grupo de back-end durante la creación de reglas. Una vez habilitado, Application Gateway garantiza que las instancias de anulación de registro de un grupo de back-end no reciban solicitudes nuevas y se permite que las solicitudes existentes se completen en un límite de tiempo configurado. Esto se aplica a las dos instancias de back-end que se quitan explícitamente del grupo de back-end mediante un cambio en la configuración del usuario y las instancias de back-end que se notifican como incorrectas según determinan los sondeos de estado. La única excepción a esto son las solicitudes enlazadas para las instancias de cancelación del registro, que se han eliminado explícitamente, debido a la afinidad de la sesión administrada por la puerta de enlace y siguen siendo procesadas por el proxy hasta las instancias de cancelación del registro.

Para más información, consulte [Introducción a la configuración de Application Gateway](configuration-http-settings.md#connection-draining).

## <a name="custom-error-pages"></a>Páginas de error personalizadas

Application Gateway permite crear páginas de error personalizadas, en lugar de mostrar las páginas de error predeterminadas. Mediante una página de error personalizada puede usar su propia marca y diseño.

Para obtener más información, consulte [Errores personalizados](custom-error.md).

## <a name="rewrite-http-headers-and-url"></a>Reescritura de encabezados HTTP y URL

Los encabezados HTTP permiten que el cliente y el servidor pasen información adicional con la solicitud o la respuesta. Volver a escribir estos encabezados HTTP le permitirá lograr varios escenarios importantes, como:

- Agregar campos de encabezado relacionados con la seguridad como HSTS/X-XSS-Protection.
- Quitar campos de encabezado de respuesta que pueden revelar información confidencial.
- Eliminar información del puerto desde los encabezados X-Forwarded-For.

Application Gateway y la SKU de WAF v2 admiten la posibilidad de agregar, quitar o actualizar los encabezados de solicitud y respuesta HTTP, mientras los paquetes de solicitud y respuesta se mueven entre los grupos de servidores backend y clientes. También se pueden reescribir las direcciones URL, los parámetros de cadena de consulta y el nombre de host. Con el enrutamiento basado en la ruta de acceso a las direcciones URL y la reescritura de direcciones URL, puede elegir entre enrutar las solicitudes a uno de los grupos de back-end en función de la ruta de acceso original, o bien la ruta de acceso reescrita, mediante la opción de evaluación repetida de la asignación de la ruta de acceso. 

También se ofrece la posibilidad de agregar condiciones a fin de garantizar que los encabezados o las direcciones URL especificados se reescriben solo cuando se cumplen ciertas condiciones. Estas condiciones se basan en la información de solicitud y respuesta.

Para más información, consulte el artículo sobre la [Reescritura de encabezados HTTP y direcciones URL](rewrite-http-headers-url.md).

## <a name="sizing"></a>Ajuste de tamaño

La versión Standard_v2 de Application Gateway se puede configurar para las implementaciones de tamaño fijo o de escalado automático. Esta SKU no ofrece tamaños de instancia diferentes. Para más información sobre el rendimiento y los precios de la versión 2, consulte [Escalabilidad automática v2](application-gateway-autoscaling-zone-redundant.md) y [Explicación de los precios](understanding-pricing.md).

La versión Standard de Application Gateway se ofrece en tres tamaños: **pequeño**, **mediano** y **grande**. Tamaños pequeños de instancia están pensados para escenarios de desarrollo y pruebas.

Para ver una lista completa de los límites de la puerta de enlace de aplicaciones, consulte el tema sobre los [límites de servicio de Application Gateway](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fapplication-gateway%2ftoc.json#application-gateway-limits).

En la tabla siguiente se muestra un promedio del rendimiento para cada instancia de puerta de enlace de aplicaciones v1 con descarga SSL habilitada:

| Tamaño de respuesta medio de página de back-end | Pequeña | Media | grande |
| --- | --- | --- | --- |
| 6 KB |7,5 Mbps |13 Mbps |50 Mbps |
| 100 KB |35 Mbps |100 Mbps |200 Mbps |

> [!NOTE]
> Se trata de valores aproximados para un rendimiento de puerta de enlace de aplicaciones. El rendimiento real depende de varios detalles del entorno, como el tamaño medio de página, la ubicación de las instancias de back-end y el tiempo de procesamiento para proporcionar una página. Para los números de rendimiento exactos, debe ejecutar sus propias pruebas. Estos valores solo se proporcionan para obtener instrucciones de planeamiento de capacidad.

## <a name="version-feature-comparison"></a>Comparación de las características de las versiones

Para una comparación de las características de la versión 1 y de la versión 2 de Application Gateway, consulte [Escalabilidad automática y Application Gateway con redundancia de zona v2](application-gateway-autoscaling-zone-redundant.md#feature-comparison-between-v1-sku-and-v2-sku)

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo funciona Application Gateway en [Funcionamiento de una puerta de enlace de aplicaciones](how-application-gateway-works.md).