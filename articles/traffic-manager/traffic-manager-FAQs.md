---
title: Preguntas más frecuentes de Azure Traffic Manager
description: Este artículo ofrece respuestas a las preguntas más frecuentes sobre Traffic Manager.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/26/2019
ms.author: duau
ms.openlocfilehash: 86758c355566fb67ebd8a606068e2044e0b8bd64
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400184"
---
# <a name="traffic-manager-frequently-asked-questions-faq"></a>Preguntas más frecuentes (P+F) sobre Traffic Manager

## <a name="traffic-manager-basics"></a>Introducción a Traffic Manager

### <a name="what-ip-address-does-traffic-manager-use"></a>¿Qué dirección IP usa el Administrador de tráfico?

Tal y como se explica en la sección sobre el [funcionamiento de Traffic Manager](../traffic-manager/traffic-manager-how-it-works.md), este servicio funciona en el nivel de DNS. Envía respuestas DNS para dirigir a los clientes al punto de conexión de servicio adecuado. Después, los clientes se conectan directamente al punto de conexión de servicio, y no a través del Administrador de tráfico.

Por lo tanto, este servicio no proporciona un punto de conexión o una dirección IP para que los clientes puedan conectarse. Si desea una dirección IP estática para el servicio, esta debe configurarse en el servicio y no en Traffic Manager.

### <a name="what-types-of-traffic-can-be-routed-using-traffic-manager"></a>¿Qué tipos de tráfico se pueden enrutar mediante Traffic Manager?
Como se explica en [Funcionamiento de Traffic Manager](../traffic-manager/traffic-manager-how-it-works.md), un punto de conexión de Traffic Manager puede ser cualquier servicio con acceso a Internet que esté hospedado dentro o fuera de Azure. Por lo tanto, Traffic Manager puede redirigir el tráfico que se origina desde la red pública de Internet hacia un conjunto de puntos de conexión que también tienen acceso a Internet. Si tiene puntos de conexión que están dentro de una red privada (por ejemplo, una versión interna de [Azure Load Balancer](../load-balancer/components.md#frontend-ip-configurations)) o tiene usuarios que realizan solicitudes DNS desde estas redes internas, no puede usar Traffic Manager para enrutar este tráfico.

### <a name="does-traffic-manager-support-sticky-sessions"></a>¿Admite Traffic Manager sesiones temporales?

Tal y como se explica en la sección sobre el [funcionamiento de Traffic Manager](../traffic-manager/traffic-manager-how-it-works.md), este servicio funciona en el nivel de DNS. para dirigir a los clientes al punto de conexión de servicio adecuado. Los clientes se conectan directamente al punto de conexión de servicio, y no a través de Traffic Manager. Por tanto, Traffic Manager no ve el tráfico HTTP que circula entre el cliente y el servidor.

Además, la dirección IP de origen de la consulta de DNS que recibe Traffic Manager pertenece al servicio DNS recursivo, no al cliente. Por lo tanto, Traffic Manager no tiene forma de realizar un seguimiento de los clientes individuales y no puede implementar sesiones temporales. Esta limitación es común en todos los sistemas de administración de tráfico basado en DNS; no se trata de una limitación específica de Traffic Manager.

### <a name="why-am-i-seeing-an-http-error-when-using-traffic-manager"></a>¿Por qué obtengo un error HTTP al utilizar Traffic Manager?

Tal y como se explica en la sección sobre el [funcionamiento de Traffic Manager](../traffic-manager/traffic-manager-how-it-works.md), este servicio funciona en el nivel de DNS. para dirigir a los clientes al punto de conexión de servicio adecuado. Después, los clientes se conectan directamente al punto de conexión de servicio, y no a través del Administrador de tráfico. Traffic Manager no ve el tráfico HTTP que circula entre cliente y servidor. Por tanto, todos los errores HTTP que obtenga deben provenir de la aplicación. Para que el cliente pueda conectarse a la aplicación, todos los pasos de la resolución DNS deben estar completos. Esto incluye cualquier interacción que tenga Traffic Manager en el flujo de tráfico de la aplicación.

Por tanto, debe llevar a cabo una investigación más extensa centrándose en la aplicación.

El encabezado host HTTP enviado desde el explorador del cliente es la fuente más habitual de problemas. Asegúrese de que la aplicación está configurada para aceptar el encabezado host correcto para el nombre de dominio que use. Para los puntos de conexión que utilizan Azure App Service, consulte [Configuración de un nombre de dominio personalizado para una aplicación web en Azure App Service mediante Traffic Manager](../app-service/configure-domain-traffic-manager.md).

### <a name="what-is-the-performance-impact-of-using-traffic-manager"></a>¿Cómo afecta al rendimiento el uso del Administrador de tráfico?

Tal y como se explica en la sección sobre el [funcionamiento de Traffic Manager](../traffic-manager/traffic-manager-how-it-works.md), este servicio funciona en el nivel de DNS. Puesto que los clientes se conectan directamente a los puntos de conexión de servicio, el rendimiento no se verá afectado cuando se utiliza el Administrador de tráfico una vez establecida la conexión.

Como Traffic Manager se integra con las aplicaciones en el nivel de DNS, hay que insertar una búsqueda DNS adicional en la cadena de resolución DNS. El impacto del Administrador de tráfico en el tiempo de resolución DNS es mínimo. Traffic Manager usa una red global de servidores de nombres y redes de difusión por proximidad [anycast](https://en.wikipedia.org/wiki/Anycast) para asegurarse de que siempre se enruten las consultas de DNS en el servidor de nombres disponible más cercano. Además, como las respuestas DNS se almacenan en caché, la latencia DNS adicional que se genera por utilizar Administrador de tráfico se aplica solo a una fracción de las sesiones.

El método Rendimiento enruta el tráfico al punto de conexión más cercano disponible. Como resultado, el impacto en el rendimiento general asociado a este método debería ser mínimo. Cualquier aumento en la latencia de DNS se debería reemplazar por una menor latencia de red al punto de conexión.

### <a name="what-application-protocols-can-i-use-with-traffic-manager"></a>¿Qué protocolos de aplicación puedo usar con el Administrador de tráfico?

Tal y como se explica en la sección sobre el [funcionamiento de Traffic Manager](../traffic-manager/traffic-manager-how-it-works.md), este servicio funciona en el nivel de DNS. Una vez finalizada la búsqueda DNS, los clientes se conectan directamente al punto de conexión de la aplicación, y no a través del Administrador de tráfico. Por tanto, la conexión puede usar cualquier protocolo de aplicación. Si selecciona TCP como el protocolo de supervisión, la supervisión del estado del punto de conexión de Traffic Manager puede realizarse sin usar ningún protocolo de aplicación. Si decide comprobar el estado mediante un protocolo de aplicación, el punto de conexión necesita ser capaz de responder a las solicitudes HTTP o HTTPS GET.

### <a name="can-i-use-traffic-manager-with-a-naked-domain-name"></a>¿Puedo usar Traffic Manager con un nombre de dominio desnudo?

Sí. Para obtener información sobre cómo crear un registro de alias para el código Apex del nombre de dominio para hacer referencia a un perfil de Azure Traffic Manager, consulte [Configuración de un registro de alias para admitir nombres de dominio de Apex con Traffic Manager](../dns/tutorial-alias-tm.md).

### <a name="does-traffic-manager-consider-the-client-subnet-address-when-handling-dns-queries"></a>¿Traffic Manager considera la dirección de subred de cliente cuando controla las consultas de DNS? 

Sí. Además de la dirección IP de origen de la consulta de DNS que recibe (que generalmente es la dirección IP de la resolución DNS), al realizar búsquedas con los métodos de enrutamiento geográfico, de rendimiento y de subred, Traffic Manager también tiene en cuenta la dirección de subred del cliente si la resolución que realiza la consulta en nombre del usuario final la incluye en la consulta.  
En concreto, [RFC 7871: subred de cliente en consultas de DNS](https://tools.ietf.org/html/rfc7871) que proporciona un [mecanismo de extensión para DNS (EDNS0)](https://tools.ietf.org/html/rfc2671) que puede pasar la dirección de subred de cliente desde las resoluciones que la admiten.

### <a name="what-is-dns-ttl-and-how-does-it-impact-my-users"></a>¿Qué es el TTL de DNS y cómo afecta a mis usuarios?

Cuando una consulta de DNS llega a Traffic Manager, establece un valor en la respuesta denominada período de vida (TTL). Este valor, cuya unidad está en segundos, indica a las resoluciones DNS de bajada cuánto tiempo se debe almacenar en caché esta respuesta. Aunque no se garantiza que las resoluciones DNS almacenen en caché este resultado, hacerlo les permite responder a cualquier consulta posterior de la memoria caché en lugar de ir a los servidores DNS de Traffic Manager. Esto afecta a las respuestas de la manera siguiente:

- Un TTL más alto reduce el número de consultas que llegan a los servidores DNS de Traffic Manager, que puede reducir el costo de un cliente ya que el número de consultas que se atiende es un uso facturable.
- Un TTL más alto puede reducir potencialmente el tiempo que se tarda en realizar una búsqueda de DNS.
- Un TTL más alto también significa que sus datos no reflejan la información de estado más reciente que Traffic Manager ha obtenido mediante sus agentes de sondeo.

### <a name="how-high-or-low-can-i-set-the-ttl-for-traffic-manager-responses"></a>¿Cuál es el límite superior e inferior en que puedo establecer el TTL para las respuestas de Traffic Manager?

Puede establecer, en un nivel de perfil, el TTL de DNS tan bajo como 0 segundos y tan alto como 2 147 483 647 segundos (el intervalo máximo compatible con [RFC-1035](https://www.ietf.org/rfc/rfc1035.txt )). Un TTL de 0 significa que las resoluciones DNS de bajada no almacenan en caché las respuestas de las consultas y se espera que todas las consultas lleguen a los servidores DNS de Traffic Manager para su resolución.

### <a name="how-can-i-understand-the-volume-of-queries-coming-to-my-profile"></a>¿Cómo puedo comprender el volumen de las consultas que llegan a mi perfil? 

Una de las métricas que proporciona Traffic Manager es el número de consultas a las que responde un perfil. Puede obtener esta información en una agregación de nivel de perfil o dividirla aún más para ver el volumen de consultas en las que se devolvieron puntos de conexión específicos. Además, puede configurar alertas para recibir una notificación si el volumen de respuestas de consulta supera las condiciones que haya configurado. Para obtener más detalles, consulte [Métricas y alertas de Traffic Manager](traffic-manager-metrics-alerts.md).

## <a name="traffic-manager-geographic-traffic-routing-method"></a>Método de enrutamiento del tráfico geográfico de Traffic Manager

### <a name="what-are-some-use-cases-where-geographic-routing-is-useful"></a>¿En qué casos de uso el enrutamiento geográfico resulta útil?

El tipo de enrutamiento geográfico se puede usar en cualquier escenario en el que el cliente de Azure necesite distinguir a sus usuarios en función de las regiones geográficas. Por ejemplo, si usa el método de enrutamiento de tráfico geográfico, puede darle a los usuarios provenientes de regiones específicas una experiencia de usuario distinta de las de otras regiones. Otro ejemplo es cumplir los mandatos de soberanía de datos locales que requieren que los usuarios de una región específica sean atendidos solo por puntos de conexión de dicha región.

### <a name="how-do-i-decide-if-i-should-use-performance-routing-method-or-geographic-routing-method"></a>¿Cómo decido si debo usar el método de enrutamiento por rendimiento o el método de enrutamiento geográfico?

La diferencia clave entre estos dos populares métodos de enrutamiento es que en el método de enrutamiento por rendimiento el objetivo principal es enviar tráfico al punto de conexión que puede proporcionar la latencia más baja al llamador, mientras que en el método de enrutamiento geográfico el objetivo principal es aplicar una geovalla para los llamadores de manera que pueda enrutarlos deliberadamente a un punto de conexión determinado. La superposición se produce porque hay una correlación entre la proximidad geográfica y una latencia menor, aunque este no es siempre el caso. Puede haber un punto de conexión en una ubicación geográfica diferente que puede ofrecer una mejor experiencia de latencia para el llamador y, en ese caso, el enrutamiento por rendimiento enviará al usuario a ese punto de conexión, pero el enrutamiento geográfico siempre lo enviará al punto de conexión que se haya asignado para su región geográfica. Para aclarar esto, considere el ejemplo siguiente: con el enrutamiento geográfico, puede realizar asignaciones poco comunes, como enviar todo el tráfico de Asia a puntos de conexión en Estados Unidos y todo el tráfico de Estados Unidos a puntos de conexión en Asia. En ese caso, el enrutamiento geográfico hará deliberadamente exactamente lo que le haya indicado que haga mediante la configuración, y la optimización por rendimiento no es una consideración. 
>[!NOTE]
>Puede haber escenarios en los que necesite funcionalidades de enrutamiento tanto por rendimiento como geográfico. Para estos escenarios, los perfiles anidados pueden ser una excelente opción. Por ejemplo, puede configurar un perfil principal con enrutamiento geográfico donde todo el tráfico de Norteamérica se envía a un perfil anidado con puntos de conexión en EE. UU. y usar el enrutamiento por rendimiento para enviar ese tráfico al mejor punto de conexión dentro de ese conjunto. 

### <a name="what-are-the-regions-that-are-supported-by-traffic-manager-for-geographic-routing"></a>¿Cuáles son las regiones que son compatibles con Traffic Manager para el enrutamiento geográfica?

Puede encontrar la jerarquía de países o regiones utilizada por Traffic Manager [aquí](traffic-manager-geographic-regions.md). Aunque esta página se mantiene actualizada con cualquier cambio que se realice, puede recuperar mediante programación la misma información mediante la [API de REST de Azure Traffic Manager](https://docs.microsoft.com/rest/api/trafficmanager/). 

### <a name="how-does-traffic-manager-determine-where-a-user-is-querying-from"></a>¿Cómo determina Traffic Manager desde dónde consulta un usuario?

Traffic Manager busca la dirección IP de origen de la consulta (probablemente se trata de una resolución DNS local que realiza la consulta en nombre del usuario) y usa una dirección IP interna a la asignación de la región para determinar la ubicación. Esta asignación se actualiza de forma continuada para tener en cuenta los cambios de Internet. 

### <a name="is-it-guaranteed-that-traffic-manager-can-correctly-determine-the-exact-geographic-location-of-the-user-in-every-case"></a>¿Está garantizado que Traffic Manager puede determinar correctamente la ubicación geográfica exacta del usuario en cada caso?

No, Traffic Manager no puede garantizar que la región geográfica que se infiere de la dirección IP de origen de una consulta de DNS siempre corresponderá a la ubicación del usuario debido a las razones siguientes:

- En primer lugar, tal como se describía en la P+F anterior, la dirección IP de origen que vemos es la de una resolución DNS que realiza la búsqueda en nombre del usuario. Si bien la ubicación geográfica de la resolución DNS es un buen proxy para la ubicación geográfica del usuario, también puede ser distinta dependiendo de la huella del servicio de resolución DNS y el servicio de resolución DNS específico que eligió usar un cliente. Por ejemplo, un cliente ubicado en Malasia podría especificar en la configuración de su dispositivo que se use un servicio de resolución DNS cuyo servidor DNS en Singapur se podría detectar para controlar las resoluciones de consulta de ese usuario o dispositivo. En ese caso, Traffic Manager solo puede ver la dirección IP de la resolución que corresponda a la ubicación de Singapur. Además, consulte la P+F anterior relacionada con la compatibilidad de direcciones de subred de cliente en esta página.

- En segundo lugar, Traffic Manager usa un mapa interno para traducir la dirección IP a la región geográfica. Si bien este mapa se valida y actualiza constantemente para aumentar la precisión y responder al carácter evolutivo de Internet, de todos modos existe la posibilidad de que nuestra información no sea una representación exacta de la ubicación geográfica de todas las direcciones IP.

###  <a name="does-an-endpoint-need-to-be-physically-located-in-the-same-region-as-the-one-it-is-configured-with-for-geographic-routing"></a>¿El punto de conexión debe encontrarse físicamente en la misma región que la configurada para el enrutamiento geográfico?

No, la ubicación del punto de conexión no impone restricción alguna sobre qué las regiones pueden asignarse a él. Por ejemplo, un punto de conexión en la región de Azure del centro de EE. UU. puede hacer que todos los usuarios de la India se redirijan a él.

### <a name="can-i-assign-geographic-regions-to-endpoints-in-a-profile-that-is-not-configured-to-do-geographic-routing"></a>¿Se pueden asignar regiones geográficas a puntos de conexión en un perfil que no está configurado para el enrutamiento geográfico?

Sí, si el método de enrutamiento de un perfil no es geográfico, puede usar la [API de REST de Azure Traffic Manager](https://docs.microsoft.com/rest/api/trafficmanager/) para asignar las regiones geográficas a puntos de conexión de dicho perfil. En el caso de perfiles de tipo de enrutamiento no geográficos, se omite esta configuración. Si cambia después este perfil al tipo de enrutamiento geográfico, Traffic Manager puede usar esas asignaciones.

### <a name="why-am-i-getting-an-error-when-i-try-to-change-the-routing-method-of-an-existing-profile-to-geographic"></a>¿Por qué cuando intento cambiar el método de enrutamiento de un perfil existente a geográfico obtengo un error?

Todos los puntos de conexión en un perfil con una necesidad de enrutamiento geográfico deben tener al menos una región asignada a ellos. Para convertir un perfil existente al tipo de enrutamiento geográfico, primero debe asociar las regiones geográficas a todos sus puntos de conexión mediante la [API de REST de Azure Traffic Manager](https://docs.microsoft.com/rest/api/trafficmanager/) antes de cambiar el tipo de enrutamiento a geográfico. Si usa el portal, tendrá que eliminar primero los puntos de conexión, cambiar el método de enrutamiento del perfil a geográfico y, después, agregar los puntos de conexión junto con su asignación de región geográfica.

### <a name="why-is-it-strongly-recommended-that-customers-create-nested-profiles-instead-of-endpoints-under-a-profile-with-geographic-routing-enabled"></a>¿Por qué se recomienda encarecidamente que los clientes creen perfiles anidados en lugar de puntos de conexión en un perfil con el enrutamiento geográfico habilitado?

Una región puede asignarse a un único punto de conexión dentro de un perfil si está utilizando el método de enrutamiento geográfico. Si ese punto de conexión no es un tipo anidado con un perfil secundario asociado a él, si ese punto de conexión es incorrecto, Traffic Manager le sigue enviando tráfico ya que la alternativa de no enviar ningún tráfico es peor. Traffic Manager no realiza la conmutación por error a otro punto de conexión, incluso cuando la región asignada es la "principal" de la región asignada al punto de conexión con estado incorrecto (por ejemplo, si un punto de conexión con la región España tiene el estado incorrecto, no se realizará la conmutación por error a otro punto de conexión que tenga asignada la región Europa). Esto se hace para asegurarse de que Traffic Manager respeta los límites geográficos que un cliente ha configurado en su perfil. Para obtener el beneficio de la conmutación por error a otro punto de conexión cuando un punto de conexión es incorrecto, se recomienda que las regiones geográficas se asignen a perfiles anidados con varios puntos de conexión dentro de ellos en lugar de a puntos de conexión individuales. De este modo, si se produce un error en un punto de conexión del perfil secundario anidado, el tráfico puede conmutar por error a otro punto de conexión en el mismo perfil secundario anidado.

### <a name="are-there-any-restrictions-on-the-api-version-that-supports-this-routing-type"></a>¿Existen restricciones en la versión de API que admite este tipo de enrutamiento?

Sí, solo la API versión 2017-03-01 y más recientes admiten el tipo de enrutamiento geográfico. Las versiones anteriores de la API no se pueden usar para crear perfiles de tipo de enrutamiento geográfico ni para asignar regiones geográficas a puntos de conexión. Si se usa una versión anterior de la API para recuperar perfiles desde una suscripción de Azure, no se devuelve ningún perfil de tipo de enrutamiento geográfico. Además, al usar las versiones anteriores de la API, cualquier perfil devuelto que tenga puntos de conexión con una asignación de región geográfica no muestra su asignación de región geográfica.

## <a name="traffic-manager-subnet-traffic-routing-method"></a>Métodos de enrutamiento del tráfico de subredes de Traffic Manager

### <a name="what-are-some-use-cases-where-subnet-routing-is-useful"></a>¿En qué casos de uso el enrutamiento de subredes resulta útil?

El enrutamiento de subredes le permite diferenciar la experiencia que proporciona a grupos específicos de usuarios que se identifican mediante la dirección IP de origen de sus solicitudes de DNS. Un ejemplo sería mostrar contenido diferente si los usuarios se están conectando a un sitio web desde su sede corporativa. Otro sería restringir a los usuarios de ciertos ISP para tener acceso solo a los puntos de conexión que admiten solo conexiones IPv4 si esos ISP tienen un rendimiento inferior al esperado cuando se usa IPv6.
Otra razón para utilizar el método de enrutamiento de subredes es en conjunción con otros perfiles en un conjunto de perfiles anidados. Por ejemplo, si quiere usar el método de enrutamiento geográfico para establecer una geovalla alrededor de sus usuarios, pero para un ISP específico quiere usar un método de enrutamiento diferente, puede tener un perfil con el método de enrutamiento de subredes como perfil primario y reemplazar ese ISP para utilizar un perfil secundario específico y tener el perfil geográfico estándar para todos los demás.

### <a name="how-does-traffic-manager-know-the-ip-address-of-the-end-user"></a>¿Cómo sabe Traffic Manager la dirección IP del usuario final?

Los dispositivos de usuario final suelen utilizar una resolución DNS para realizar la búsqueda de DNS en su nombre. La dirección IP saliente de estas resoluciones es lo que Traffic Manager ve como la dirección IP de origen. Además, el método de enrutamiento de subredes también buscar ver si se ha pasado información de Subred de Cliente Ampliada EDNS0 (ECS) con la solicitud. Si hay información de ECS, esa es la dirección que se usará para determinar el enrutamiento. Si dicha información no existe, la dirección IP de origen de la consulta se usa para fines de enrutamiento.

### <a name="how-can-i-specify-ip-addresses-when-using-subnet-routing"></a>¿Cómo se pueden especificar direcciones IP al usar el enrutamiento de subredes?

Las direcciones IP para asociar a un punto de conexión se pueden especificar de dos maneras. En primer lugar, puede utilizar la notación octeto decimal de cuatro puntos con direcciones de inicio y final para especificar el intervalo (por ejemplo, 1.2.3.4-5.6.7.8 o 3.4.5.6-3.4.5.6). En segundo lugar, puede usar la notación CIDR para especificar el intervalo (por ejemplo, 1.2.3.0/24). Puede especificar varios intervalos y puede usar ambos tipos de notación en un conjunto de intervalos. Sin embargo, se aplican algunas restricciones.

-    No puede haber superposición de intervalos de direcciones, ya que cada IP solo puede mapearse a un único punto de conexión.
-    La dirección de inicio no puede superar la dirección final
-    En el caso de la notación CIDR, la dirección IP antes de "/" debe ser la dirección de inicio de ese intervalo (por ejemplo, 1.2.3.0/24 es válido, pero 1.2.3.4.4/24 no es válido).

### <a name="how-can-i-specify-a-fallback-endpoint-when-using-subnet-routing"></a>¿Cómo se puede especificar un punto de conexión de reserva al usar el enrutamiento de subredes?

En un perfil con el enrutamiento de subredes, si tiene un punto de conexión sin ninguna subred asignada, cualquier solicitud que no coincida con otros puntos de conexión se dirigirá aquí. Se recomienda encarecidamente tener este punto de conexión de reserva en el perfil, ya que Traffic Manager devolverá una respuesta NXDOMAIN si llega una solicitud y no está asignado a ningún punto de conexión o si se asigna a un punto de conexión dañado.

### <a name="what-happens-if-an-endpoint-is-disabled-in-a-subnet-routing-type-profile"></a>¿Qué ocurre si se deshabilita un punto de conexión en un perfil de tipo de enrutamiento de subredes?

En un perfil con enrutamiento de subredes, si tiene un punto de conexión deshabilitado, Traffic Manager se comportará como si no existieran ese punto de conexión ni las asignaciones de subredes. Si se recibe una consulta que habría coincidido con su asignación de direcciones IP y el punto de conexión está deshabilitado, Traffic Manager devolverá un punto de conexión de reserva (uno sin asignaciones); o bien, si dicho punto de conexión no existe, devolverá una respuesta NXDOMAIN.

## <a name="traffic-manager-multivalue-traffic-routing-method"></a>Método de enrutamiento del tráfico de varios valores de Traffic Manager

### <a name="what-are-some-use-cases-where-multivalue-routing-is-useful"></a>¿En qué casos de uso el enrutamiento multivalor resulta útil?

El enrutamiento de varios valores devuelve múltiples puntos de conexión válidos en una sola respuesta de consulta. La principal ventaja de esto es que, si un punto de conexión está dañado, el cliente tiene más opciones para volver a intentarlo sin hacer otra llamada DNS (lo que podría devolver el mismo valor desde una caché ascendente). Esto se aplica a aplicaciones sensibles a la disponibilidad que desean minimizar el tiempo de inactividad.
Otro uso del método de enrutamiento de varios valores es si un punto de conexión tiene doble conexión a direcciones IPv4 e IPv6 y desea dar al llamador ambas opciones para que elija cuándo iniciar una conexión con el punto de conexión.

### <a name="how-many-endpoints-are-returned-when-multivalue-routing-is-used"></a>¿Cuántos puntos de conexión se devuelven cuando se usa el enrutamiento de varios valores?

Puede especificar el número máximo de puntos de conexión que se devolverán y MultiValue devolverá no más que esos puntos de conexión en buen estado cuando se reciba una consulta. El valor máximo posible para esta configuración es 10.

### <a name="will-i-get-the-same-set-of-endpoints-when-multivalue-routing-is-used"></a>¿Obtendré el mismo conjunto de puntos de conexión cuando se use el enrutamiento de varios valores?

No podemos garantizar la devolución del mismo conjunto de puntos de conexión en cada consulta. Esto también se ve afectado por el hecho de que algunos de los puntos de conexión podrían estar dañados y no se incluirían en la respuesta

## <a name="real-user-measurements"></a>Real User Measurements

### <a name="what-are-the-benefits-of-using-real-user-measurements"></a>¿Cuáles son las ventajas de usar Real User Measurements?

Cuando se usa el método de enrutamiento de rendimiento, Traffic Manager selecciona la mejor región de Azure para que se conecte el usuario final, para lo cual inspecciona la IP de origen y la subred de cliente EDNS (si se pasa) y vuelve a comprobar la latencia de red que mantiene el servicio de inteligencia. Real User Measurements mejora esta operación para la base de usuarios finales ya que su experiencia contribuye a esta tabla de latencia, además de garantizar que esta tabla abarca adecuadamente las redes de usuario final desde donde los usuarios finales se conectan a Azure. De este modo, el usuario final realiza enrutamientos con una mayor precisión.

### <a name="can-i-use-real-user-measurements-with-non-azure-regions"></a>¿Se puede usar Real User Measurements con regiones que no son de Azure?

Real User Measurements mide la latencia para llegar a las regiones de Azure e informa únicamente a este respecto. Si va a usar el enrutamiento basado en el rendimiento con puntos de conexión hospedados en regiones que no son de Azure, también puede beneficiarse de esta característica al disponer de una mayor información sobre la latencia de la región de Azure representativa que había seleccionado para asociarse con este punto de conexión.

### <a name="which-routing-method-benefits-from-real-user-measurements"></a>¿Qué método de enrutamiento se beneficia de Real User Measurements?

La información adicional obtenida mediante Real User Measurements solo es aplicable a perfiles que usan el método de enrutamiento de rendimiento. El vínculo Mediciones de usuario reales está disponible en todos los perfiles cuando se visualiza a través de Azure Portal.

### <a name="do-i-need-to-enable-real-user-measurements-each-profile-separately"></a>¿Es necesario habilitar Real User Measurements para cada perfil por separado?

No, solo es necesario habilitarla una vez por suscripción y toda la información de latencia medida y notificada estará disponible para todos los perfiles.

### <a name="how-do-i-turn-off-real-user-measurements-for-my-subscription"></a>¿Cómo se desactiva Real User Measurements en una suscripción?

Puede dejar de acumular gastos relacionados con Real User Measurements cuando deja de recopilar y enviar medidas de latencia desde la aplicación cliente. Por ejemplo, cuando se inserta JavaScript de medida en páginas web, puede dejar de usar esta característica quitando el código JavaScript o bien desactivando su invocación cuando se representa la página.

También puede eliminar la clave para desactivar las Mediciones de usuario reales. Una vez eliminada la clave, cualquier medida que se envíe a Traffic Manager con esta clave se descartará.

### <a name="can-i-use-real-user-measurements-with-client-applications-other-than-web-pages"></a>¿Se puede usar Real User Measurements con aplicaciones cliente que no sean páginas web?

Sí, Real User Measurements está diseñada para ingerir datos recopilados a través de diferentes tipos de clientes de usuario final. Esta pregunta frecuente se actualizará a medida que se admitan nuevos tipos de aplicaciones cliente.

### <a name="how-many-measurements-are-made-each-time-my-real-user-measurements-enabled-web-page-is-rendered"></a>¿Cuántas medidas se realizan cada vez que se representa la página web habilitada para Real User Measurements?

Cuando se usa Real User Measurements con el código JavaScript de medida proporcionado, cada representación de página da lugar a la realización de seis medidas. Estas se notifican luego al servicio Traffic Manager. Esta característica se cobra en función del número de medidas notificadas al servicio Traffic Manager. Por ejemplo, si el usuario navega fuera de su página web mientras se está realizando la medida, pero antes de que se notifique, esas medidas no se tienen en cuenta en la facturación.

### <a name="is-there-a-delay-before-real-user-measurements-script-runs-in-my-webpage"></a>¿Existe un retraso antes de que el script de Real User Measurements se ejecute en la página web?

No, no hay ningún retraso programado antes de invocar el script.

### <a name="can-i-use-real-user-measurements-with-only-the-azure-regions-i-want-to-measure"></a>¿Se pueden usar las Mediciones de usuario reales solo con las regiones que se quieren medir?

No, cada vez que se invoque, el script de Real User Measurements mide un conjunto de seis regiones de Azure, según determina el servicio. Este conjunto varía entre distintas invocaciones y, cuando se producen muchas de estas invocaciones, la cobertura de medida se distribuye entre distintas regiones de Azure.

### <a name="can-i-limit-the-number-of-measurements-made-to-a-specific-number"></a>¿Se puede limitar el número de medidas realizadas a un número específico?

El código JavaScript de medida se inserta dentro de la página web y se tiene control completo sobre cuándo empezar a usarlo y dejar de usarlo. Siempre y cuando el servicio Traffic Manager reciba una solicitud de una lista de regiones de Azure que se van a medir, se devuelve un conjunto de regiones.

### <a name="can-i-see-the-measurements-taken-by-my-client-application-as-part-of-real-user-measurements"></a>¿Se pueden ver las medidas realizadas por mis aplicaciones cliente como parte de Real User Measurements?

Dado que la lógica de medida se ejecuta desde la aplicación cliente, se tiene control completo sobre lo que sucede, como ver las medidas de latencia. Traffic Manager no informa de una vista agregada de las medidas recibidas con la clave vinculada a su suscripción.

### <a name="can-i-modify-the-measurement-script-provided-by-traffic-manager"></a>¿Se puede modificar el script de medida proporcionado por Traffic Manager?

Mientras se tiene el control de lo que está insertado en la página web, se recomienda firmemente no realizar cambios en el script de medida para garantizar que las latencias se miden y notifican correctamente.

### <a name="will-it-be-possible-for-others-to-see-the-key-i-use-with-real-user-measurements"></a>¿Podrán ver otros usuarios la clave que se usa con Real User Measurements?

Cuando se inserta el script de medida en una página web, otros usuarios podrán ver el script y la clave de Real User Measurements (RUM). Sin embargo, es importante saber que esta clave es diferente del identificador de suscripción y que la genera Traffic Manager con este único fin. El hecho de conocer la clave RUM no pone en peligro la seguridad de la cuenta de Azure.

### <a name="can-others-abuse-my-rum-key"></a>¿Pueden otros usuarios hacer un uso inapropiado de la clave RUM?

Aunque otras personas pueden usar su clave para enviar información incorrecta a Azure, unas cuantas medidas erróneas no cambiarán el enrutamiento, dado que es algo que se tiene en cuenta junto con todas las demás medidas que se reciben. Si necesita cambiar las claves, puede volver a generarlas. En ese momento, la clave antigua se descarta.

### <a name="do-i-need-to-put-the-measurement-javascript-in-all-my-web-pages"></a>¿Es necesario poner el código JavaScript de medida en todas las páginas web?

Real User Measurements ofrece más valor conforme aumenta el número de medidas. Dicho esto, es decisión suya ponerlo en todas las páginas web o en unas cuantas. Nuestra recomendación es comenzar a ponerlo en la página más visitada, donde se espera que un usuario permanezca cinco o más segundos.

### <a name="can-information-about-my-end-users-be-identified-by-traffic-manager-if-i-use-real-user-measurements"></a>¿Puede Traffic Manager identificar la información sobre mis usuarios finales si uso Real User Measurements?

Cuando se usa el código JavaScript de medida proporcionado, Traffic Manager puede ver la dirección IP del cliente de los usuarios finales y la dirección IP de origen de la resolución DNS que usan. Traffic Manager usa la dirección IP del cliente solo después de haberla truncado para no poder identificar el usuario final específico que envió la medida.

### <a name="does-the-webpage-measuring-real-user-measurements-need-to-be-using-traffic-manager-for-routing"></a>¿Es necesario que la página web que realiza las medidas de usuarios reales use Traffic Manager para el enrutamiento?

No, no es necesario que use Traffic Manager. El lado de enrutamiento de Traffic Manager funciona de forma independiente de la parte Real User Measurement y, aunque es una buena idea tenerlos a ambos en la misma propiedad web, no es necesario que lo estén.

### <a name="do-i-need-to-host-any-service-on-azure-regions-to-use-with-real-user-measurements"></a>¿Es necesario hospedar algún servicio en regiones de Azure para usar Real User Measurements?

No, no es necesario hospedar ningún componente del lado servidor en Azure para que funcionen las Mediciones de usuario reales. Azure hospeda y administra la imagen de píxel único descargada por el código JavaScript de medida y el servicio que lo ejecuta en diferentes regiones de Azure. 

### <a name="will-my-azure-bandwidth-usage-increase-when-i-use-real-user-measurements"></a>¿Aumentará el uso de ancho de banda de Azure debido a la utilización de Real User Measurements?

Como se mencionó en la respuesta anterior, Azure es propietario de los componentes del lado servidor de Real User Measurements, y también los administra. Esto significa que el uso de ancho de banda de Azure no aumentará porque use Real User Measurements. Tenga en cuenta que aquí no se incluye el uso de ancho de banda al margen de los cargos de Azure. Para reducir el ancho de banda usado, se descarga una imagen de un único píxel para medir la latencia a una región de Azure. 

## <a name="traffic-view"></a>Traffic View

### <a name="what-does-traffic-view-do"></a>¿Cómo funciona Traffic View?

Traffic View es una característica de Traffic Manager que le ayuda a comprender mejor a sus usuarios y su experiencia. Emplea las consultas que recibe Traffic Manager y las tablas de inteligencia de latencia de red que el servicio mantiene para proporcionarle lo siguiente:

- Las regiones desde donde los usuarios se conectan a los puntos de conexión de Azure.
- El volumen de usuarios que se conectan desde estas regiones.
- Las regiones de Azure a las que se enrutan.
- Su experiencia de latencia en estas regiones de Azure.

Esta información está disponible para que la utilice mediante una superposición de mapa geográfico y vistas tabulares del portal, y también está disponible como datos sin procesar para su descarga.

### <a name="how-can-i-benefit-from-using-traffic-view"></a>¿Cuál es el beneficio de usar Traffic View?

Traffic View le proporciona una vista general del tráfico que reciben los perfiles de Traffic Manager. En concreto, se puede usar para entender desde dónde se conecta la base de usuarios e, igualmente importante, cuál es su experiencia de latencia media. Luego, esta información puede usarse para buscar áreas en las que es necesario centrarse, por ejemplo, la expansión de la superficie de Azure a una región que pueda prestar servicio a esos usuarios con una latencia más baja. Otra información que se puede obtener del uso de Traffic View es ver los patrones de tráfico a diferentes regiones. Esto puede ayudarle a tomar decisiones sobre si aumentar o reducir la inversión en esas regiones.

### <a name="how-is-traffic-view-different-from-the-traffic-manager-metrics-available-through-azure-monitor"></a>¿En qué se diferencia Traffic View de las métricas de Traffic Manager disponibles mediante Azure Monitor?

Azure Monitor se puede usar para comprender en un nivel agregado el tráfico que recibe el perfil y sus puntos de conexión. También permite realizar el seguimiento del estado de mantenimiento de los puntos de conexión mediante la exposición de los resultados de comprobación de mantenimiento. Cuando sea necesario ir más allá y comprender la experiencia del usuario final que se conecta a Azure en un nivel regional, Traffic View puede usarse con esta finalidad.

### <a name="does-traffic-view-use-edns-client-subnet-information"></a>¿Usa Traffic View información de subred de cliente de EDNS?

Las consultas de DNS que sirve Azure Traffic Manager consideran la información de ECS para aumentar la exactitud del enrutamiento. Sin embargo, cuando se crea el conjunto de datos que muestra desde dónde se conectan los usuarios, Vista de tráfico solo usa la dirección IP de la resolución DNS.

### <a name="how-many-days-of-data-does-traffic-view-use"></a>¿Cuántos días de datos usa Traffic View?

La salida de Traffic View incluye los datos de los siete días anteriores al día en que usted ve los datos. Como es una ventana móvil, cada vez que acceda a Traffic View verá los datos más recientes.

### <a name="how-does-traffic-view-handle-external-endpoints"></a>¿Cómo controla Traffic View los puntos de conexión externos?

Cuando se usan puntos de conexión externos hospedados fuera de regiones de Azure en un perfil de Traffic Manager, puede elegir asignarlos a una región de Azure que funcione como un servidor proxy por sus características de latencia (de hecho, esto es necesario si se usa el método de enrutamiento de rendimiento). Si tiene esta asignación de región de Azure, las métricas de latencia de dicha región se usarán al crear la salida de Vista de tráfico. Si no se especifica ninguna región de Azure, no habrá información de latencia para esos puntos de conexión externos.

### <a name="do-i-need-to-enable-traffic-view-for-each-profile-in-my-subscription"></a>¿Es necesario habilitar Traffic View en cada perfil de la suscripción?

Durante el periodo de versión preliminar, Vista de tráfico estaba habilitado en el nivel de suscripción. Como parte de las mejoras que hemos realizado antes de la disponibilidad general, ahora puede habilitar Vista de tráfico en un nivel de perfil, lo que permite una habilitación más granular de esta característica. De forma predeterminada, Vista de tráfico se deshabilitará para un perfil.

>[!NOTE]
>Si habilitó Vista de tráfico en un nivel de suscripción durante la versión preliminar, ahora debe volver a habilitarla para cada perfil de la suscripción.
 
### <a name="how-can-i-turn-off-traffic-view"></a>¿Cómo se desactiva Traffic View?

Puede desactivar Vista de tráfico para cualquier perfil mediante el Portal o la API de REST. 

### <a name="how-does-traffic-view-billing-work"></a>¿Cómo funciona la facturación de Traffic View?

El precio de Traffic View se basa en el número de puntos de datos usados para crear la salida. Actualmente, el único tipo de datos admitido son las consultas que recibe su perfil. Además, solo se le factura por el procesamiento que se haya realizado cuando se tiene Traffic View habilitado. Esto significa que, si habilita Traffic View durante un período de tiempo de un mes y lo desactiva en otros momentos, solo los puntos de datos procesados mientras tenía habilitada la característica se tienen en cuenta para la facturación.

## <a name="traffic-manager-endpoints"></a>Puntos de conexión del Administrador de tráfico

### <a name="can-i-use-traffic-manager-with-endpoints-from-multiple-subscriptions"></a>¿Puedo usar el Administrador de tráfico con puntos de conexión de varias suscripciones?

No se pueden usar puntos de conexión de varias suscripciones con Azure Web Apps. Azure Web Apps requiere que cualquier nombre de dominio personalizado usado con Web Apps se use únicamente en una suscripción. No es posible usar Web Apps desde varias suscripciones con el mismo nombre de dominio.

Para otros tipos de punto de conexión, es posible usar el Administrador de tráfico con puntos de conexión de más de una suscripción. En Resource Manager, pueden agregarse puntos de conexión de cualquier suscripción al Administrador de tráfico, siempre y cuando la persona que configura el perfil de este servicio tenga acceso de lectura al punto de conexión. Estos permisos pueden concederse mediante la funcionalidad de [control de acceso basado en roles de Azure (RBAC de Azure)](../role-based-access-control/role-assignments-portal.md). Los puntos de conexión de otras suscripciones se pueden agregar mediante [Azure PowerShell](https://docs.microsoft.com/powershell/module/az.trafficmanager/new-aztrafficmanagerendpoint) o la [CLI de Azure](https://docs.microsoft.com/cli/azure/network/traffic-manager/endpoint?view=azure-cli-latest#az-network-traffic-manager-endpoint-create).

### <a name="can-i-use-traffic-manager-with-cloud-service-staging-slots"></a>¿Puedo usar Traffic Manager con espacios de ensayo de servicio en la nube?

Sí. Los espacios de ensayo de servicio en la nube se pueden configurar en Traffic Manager como puntos de conexión externos. Las comprobaciones de estado se siguen cobrando a la tarifa Puntos de conexión de Azure.

### <a name="does-traffic-manager-support-ipv6-endpoints"></a>¿Admite el Administrador de tráfico puntos de conexión IPv6?

Traffic Manager no proporciona actualmente servidores de nombres que admitan direcciones IPv6. Sin embargo, aún se puede usar Traffic Manager con clientes IPv6 que se conecten a puntos de conexión IPv6. Un cliente no envía solicitudes de DNS directamente a Traffic Manager. En su lugar, el cliente usa un servicio DNS recursivo. Un cliente solo IPv6 envía solicitudes al servicio DNS recursivo por medio de IPv6. Después, el servicio recursivo debería poder entrar en contacto con los servidores de nombres de Traffic Manager mediante IPv4.

Traffic Manager responde con el nombre DNS o la dirección IP del punto de conexión. Para admitir un punto de conexión IPv6, hay dos opciones. Puede agregar el punto de conexión como un nombre de DNS que tenga asociado un registro AAAA y Traffic Manager comprobará su estado y lo devolverá como un tipo de registro CNAME en la respuesta de la consulta. También puede agregar ese punto de conexión directamente con la dirección IPv6 y Traffic Manager devolverá un registro de tipo AAAA en la respuesta de la consulta.

### <a name="can-i-use-traffic-manager-with-more-than-one-web-app-in-the-same-region"></a>¿Puedo usar el Administrador de tráfico con más de una aplicación web en la misma región?

Normalmente, el Administrador de tráfico se utiliza para dirigir el tráfico a las aplicaciones implementadas en diferentes regiones. Sin embargo, también se puede utilizar en los casos en que una aplicación tenga más de una implementación en la misma región. Los puntos de conexión de Azure de Traffic Manager no permiten que se agregue más de un punto de conexión de aplicación web de la misma región de Azure al mismo perfil de Traffic Manager.

### <a name="how-do-i-move-my-traffic-manager-profiles-azure-endpoints-to-a-different-resource-group-or-subscription"></a>¿Cómo muevo los puntos de conexión de Azure de mi perfil de Traffic Manager a un grupo de recursos o suscripción diferente?

El seguimiento de los puntos de conexión de Azure que están asociados a un perfil de Traffic Manager se realiza mediante sus identificadores de recurso. Cuando un recurso de Azure que se usa como punto de conexión (por ejemplo, una dirección IP pública, un servicio en la nube clásico, una aplicación web u otro perfil de Traffic Manager usado de forma anidada) se mueve a otro grupo de recursos o suscripción, su identificador de recurso cambia. Actualmente, en este caso, debe actualizar el perfil de Traffic Manager. Para ello, debe eliminar primero los puntos de conexión y, después, volver a agregarlos al perfil.

## <a name="traffic-manager-endpoint-monitoring"></a>Supervisión de puntos de conexión de Traffic Manager

### <a name="is-traffic-manager-resilient-to-azure-region-failures"></a>¿Traffic Manager es resistente a errores de región de Azure?

Traffic Manager es un componente clave de la entrega de aplicaciones de alta disponibilidad en Azure.
Para ofrecer alta disponibilidad, Traffic Manager debe tener un grado de disponibilidad excepcionalmente alto y ser resistente a errores regionales.

Por diseño, los componentes de Traffic Manager son resistentes a un error completo de cualquier región de Azure. Esta resistencia se aplica a todos los componentes del Administrador de tráfico: los servidores de nombres DNS, la API, la capa de almacenamiento y el servicio de supervisión del punto de conexión.

En el improbable caso de una interrupción de toda una región de Azure, se espera que Traffic Manager continúe funcionando con normalidad. Las aplicaciones implementadas en varias regiones de Azure pueden confiar en Traffic Manager para dirigir el tráfico a una instancia disponible de su aplicación.

### <a name="how-does-the-choice-of-resource-group-location-affect-traffic-manager"></a>¿Cómo afecta al Administrador de tráfico la elección de la ubicación del grupo de recursos?

El Administrador de tráfico es servicio global individual. No es regional. La elección de la ubicación del grupo de recursos no supone ninguna diferencia con respecto a los perfiles del Administrador de tráfico implementados en dicho grupo de recursos.

Azure Resource Manager requiere que todos los grupos de recursos especifiquen una ubicación, que determina la ubicación predeterminada de los recursos implementados en dicho grupo de recursos. Cuando se crea un perfil de Traffic Manager, se crea en un grupo de recursos. Todos los perfiles de Traffic Manager usan **global** como ubicación, lo que omite la configuración predeterminada del grupo de recursos.

### <a name="how-do-i-determine-the-current-health-of-each-endpoint"></a>¿Cómo se determina el estado actual de cada punto de conexión?

El estado de supervisión actual de cada punto de conexión, junto con el del perfil global, se muestra en el Portal de Azure. Esta información también está disponible a través de la [API de REST](https://msdn.microsoft.com/library/azure/mt163667.aspx), los [cmdlets de PowerShell](https://docs.microsoft.com/powershell/module/az.trafficmanager) y la [CLI de Azure multiplataforma](../cli-install-nodejs.md) del Monitor de tráfico.

También puede usar Azure Monitor para hacer un seguimiento del mantenimiento de los puntos de conexión y ver una representación visual de los mismos. Para más información sobre cómo usar Azure Monitor, consulte la [documentación de supervisión de Azure](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-metrics).

### <a name="can-i-monitor-https-endpoints"></a>¿Puedo supervisar puntos de conexión HTTPS?

Sí. El Administrador de tráfico admite el sondeo a través de HTTPS. Configure **HTTPS** como protocolo de la configuración de supervisión.

Traffic Manager no puede proporcionar ninguna validación de certificado, incluidos:

* No se validan certificados del servidor
* No se validan certificados SNI del servidor
* No se admiten certificados de cliente

### <a name="do-i-use-an-ip-address-or-a-dns-name-when-adding-an-endpoint"></a>¿Uso una dirección IP o un nombre DNS al agregar un punto de conexión?

Traffic Manager admite la adición de puntos de conexión mediante tres formas de referencia: como nombre DNS, como una dirección IPv4 y como una dirección IPv6. Si el punto de conexión se agrega como una dirección IPv4 o IPv6, la respuesta de la consulta será, respectivamente, de tipo de registro A o AAAA. Si el punto de conexión se agregó como un nombre DNS, la respuesta de la consulta será de tipo de registro CNAME. Solo está permitido agregar puntos de conexión como direcciones IPv4 o IPv6 si el punto de conexión es de tipo **Externo**.
Todos los métodos de enrutamiento y la configuración de supervisión son compatibles con los tres tipos de direcciones de punto de conexión.

### <a name="what-types-of-ip-addresses-can-i-use-when-adding-an-endpoint"></a>¿Qué tipos de direcciones IP puedo usar al agregar un punto de conexión?

Traffic Manager permite usar direcciones IPv4 o IPv6 para especificar los puntos de conexión. Hay algunas restricciones que se enumeran a continuación:

- No se permiten direcciones que correspondan a espacios de direcciones IP privadas reservadas. Estas direcciones incluyen las indicadas en RFC 1918, RFC 6890, RFC 5737, RFC 3068, RFC 2544 y RFC 5771
- La dirección no puede contener números de puerto (puede especificar los puertos que se usarán en las opciones de configuración del perfil)
- Dos puntos de conexión en el mismo perfil no pueden tener la misma dirección IP de destino

### <a name="can-i-use-different-endpoint-addressing-types-within-a-single-profile"></a>¿Puedo usar tipos de direccionamiento de punto de conexión diferentes con un único punto de conexión?

No, Traffic Manager no permite combinar tipos de direccionamiento de punto de conexión dentro de un perfil, excepto en el caso de un perfil con el tipo de enrutamiento de varios valores donde puede mezclar tipos de direccionamiento IPv4 y IPv6

### <a name="what-happens-when-an-incoming-querys-record-type-is-different-from-the-record-type-associated-with-the-addressing-type-of-the-endpoints"></a>¿Qué ocurre cuando el tipo de registro de la consulta entrante es diferente del tipo de registro asociado con el tipo de direccionamiento de los puntos de conexión?

Cuando se recibe una consulta con un perfil, Traffic Manager busca primero el punto de conexión que debe devolverse según el método de enrutamiento especificado y el estado de mantenimiento de los puntos de conexión. A continuación, examina el tipo de registro solicitado en la consulta entrante y el tipo de registro asociado con el punto de conexión antes de devolver una respuesta basada en la tabla siguiente.

En el caso de perfiles con cualquier método de enrutamiento que no sea de varios valores:

|Solicitud de consulta entrante|     Tipo de punto de conexión|     Respuesta proporcionada|
|--|--|--|
|ANY |    A / AAAA / CNAME |    Punto de conexión de destino| 
|A |    A / CNAME |    Punto de conexión de destino|
|A |    AAAA |    NODATA |
|AAAA |    AAAA / CNAME |    Punto de conexión de destino|
|AAAA |    A |    NODATA |
|CNAME |    CNAME |    Punto de conexión de destino|
|CNAME     |A / AAAA |    NODATA |
|

En el caso de los perfiles con el método de enrutamiento establecido en varios valores:

|Solicitud de consulta entrante|     Tipo de punto de conexión |    Respuesta proporcionada|
|--|--|--|
|ANY |    Combinación de A y AAAA |    Extremos de destino|
|A |    Combinación de A y AAAA |    Solo puntos de conexión de destino de tipo A|
|AAAA    |Combinación de A y AAAA|     Solo puntos de conexión de destino de tipo AAAA|
|CNAME |    Combinación de A y AAAA |    NODATA |

### <a name="can-i-use-a-profile-with-ipv4--ipv6-addressed-endpoints-in-a-nested-profile"></a>¿Puedo usar un perfil con puntos de conexión con direcciones IPv4 / IPv6 en un perfil anidado?

Sí, con la excepción de que un perfil de tipo varios valores no puede ser un perfil primario en un conjunto de perfiles anidados.

### <a name="i-stopped-an-web-application-endpoint-in-my-traffic-manager-profile-but-i-am-not-receiving-any-traffic-even-after-i-restarted-it-how-can-i-fix-this"></a>Detuve un punto de conexión de una aplicación web en mi perfil de Traffic Manager, pero no recibo tráfico ni siquiera después de haberlo reiniciado. ¿Cómo lo puedo corregir?

Cuando se detiene un punto de conexión de una aplicación web de Azure, Traffic Manager deja de comprobar su mantenimiento y reinicia las comprobaciones de estado solo una vez que detecta que se reinició el punto de conexión. Para evitar este retraso, deshabilite y vuelva a habilitar ese punto de conexión en el perfil de Traffic Manager después de reiniciar el punto de conexión.

### <a name="can-i-use-traffic-manager-even-if-my-application-does-not-have-support-for-http-or-https"></a>¿Puedo usar Traffic Manager incluso si mi aplicación no tiene compatibilidad para HTTP o HTTPS?

Sí. Puede especificar TCP como el protocolo de supervisión y Traffic Manager puede iniciar una conexión TCP y esperar una respuesta del punto de conexión. Si el punto de conexión responde a la solicitud de conexión con una respuesta para establecer la conexión, en el período de tiempo de expiración, entonces ese punto de conexión se marca como correcto.

### <a name="what-specific-responses-are-required-from-the-endpoint-when-using-tcp-monitoring"></a>¿Qué respuestas específicas se necesitan del punto de conexión cuando se usa la supervisión TCP?

Cuando se usa la supervisión TCP, Traffic Manager inicia un protocolo de enlace TCP de tres direcciones mediante el envío de una solicitud SYN al punto de conexión en el puerto especificado. Después, espera durante un período de tiempo (como se ha especificado en la configuración del tiempo de expiración) para obtener una respuesta SYN-ACK del punto de conexión.

- Si se recibe una respuesta SYN-ACK en el período de tiempo de expiración especificado en la configuración de supervisión, ese punto de conexión se considera correcto. Una confirmación de FIN o FIN-ACK es la respuesta esperada de Traffic Manager cuando finaliza un socket de manera normal.
- Si se recibe una respuesta SYN-ACK después del tiempo de expiración especificado, Traffic Manager responderá con un RST para restablecer la conexión.

### <a name="how-fast-does-traffic-manager-move-my-users-away-from-an-unhealthy-endpoint"></a>¿Con qué rapidez Traffic Manager mueve mis usuarios de un punto de conexión incorrecto?

Traffic Manager proporciona varias opciones que pueden ayudarle a controlar el comportamiento de conmutación por error de su perfil de Traffic Manager de la manera siguiente:

- Puede especificar que Traffic Manager sondee los puntos de conexión más frecuentemente estableciendo el intervalo de sondeo en 10 segundos. Esto garantiza que cualquier punto de conexión que vaya a ser incorrecto se detecte lo antes posible. 
- Puede especificar cuánto tiempo esperar antes de que se agote el tiempo de espera de una solicitud de comprobación de estado (valor de tiempo de espera mínimo es 5 segundos).
- Puede especificar cuántos errores pueden producirse antes de que el punto de conexión se marque como incorrecto. Este valor puede ser tan bajo como 0, en cuyo caso el punto de conexión se marca como incorrecto tan pronto como se produzca un error en la primera comprobación de estado. En cambio, con el valor mínimo de 0 para el número de errores permitido puede provocar que los puntos de conexión se quiten de la rotación debido a cualquier problema transitorio que pueda producirse en el momento del sondeo.
- Puede especificar el período de vida (TTL) para que la respuesta DNS sea tan baja como 0. Al hacer esto, las resoluciones DNS no pueden almacenar en caché la respuesta y cada nueva solicitud obtiene una respuesta que incorpora la información de estado más actualizada que tiene Traffic Manager.

Con estas opciones, Traffic Manager puede proporcionar conmutaciones por error en 10 segundos después de que un punto de conexión se vuelva incorrecto y se realice una consulta DNS en el perfil correspondiente.

### <a name="how-can-i-specify-different-monitoring-settings-for-different-endpoints-in-a-profile"></a>¿Cómo puedo especificar diferentes opciones de supervisión para los diferentes puntos de conexión de un perfil?

La configuración de supervisión de Traffic Manager se encuentra en el nivel de perfil. Si necesita usar una configuración de supervisión diferente solo para un punto de conexión, puede realizarse teniendo ese punto de conexión como un [perfil anidado](traffic-manager-nested-profiles.md) cuya configuración de supervisión sea diferente de la del perfil principal.

### <a name="how-can-i-assign-http-headers-to-the-traffic-manager-health-checks-to-my-endpoints"></a>¿Cómo puedo asignar encabezados HTTP para las comprobaciones de estado de Traffic Manager a mis puntos de conexión?

Traffic Manager permite especificar encabezados personalizados en las comprobaciones de estado de HTTP(S) que inicia en sus puntos de conexión. Si desea especificar un encabezado personalizado, puede hacerlo en el nivel de perfil (se aplica a todos los puntos de conexión) o especificarlo en el nivel del punto de conexión. Si un encabezado se define en ambos niveles, el especificado en el nivel de punto de conexión reemplazará el del perfil del nivel uno.
Un caso de uso común para esto es la especificación de encabezados para que las solicitudes de Traffic Manager se enruten correctamente a un punto de conexión hospedado en un entorno de varios inquilinos. Otro caso de uso de esto es identificar solicitudes de Traffic Manager desde registros de solicitudes HTTP(S) de un punto de conexión

### <a name="what-host-header-do-endpoint-health-checks-use"></a>¿Qué encabezado host se utiliza en las comprobaciones de estado de punto de conexión?

Si no se proporciona ningún valor de encabezado de host personalizado, el encabezado de host utilizado por Traffic Manager es el nombre DNS del destino del punto de conexión configurado en el perfil, si está disponible.

### <a name="what-are-the-ip-addresses-from-which-the-health-checks-originate"></a>¿Cuáles son las direcciones IP desde las que proceden las comprobaciones de estado?

Haga clic [aquí](https://azuretrafficmanagerdata.blob.core.windows.net/probes/azure/probe-ip-ranges.json) para ver el archivo JSON en que se enumeran las direcciones IP de las que pueden originarse las comprobaciones de estado de Traffic Manager. Revise las direcciones IP enumeradas para asegurarse de que se permiten conexiones entrantes de estas direcciones IP en los puntos de conexión para comprobar su estado de mantenimiento.

### <a name="how-many-health-checks-to-my-endpoint-can-i-expect-from-traffic-manager"></a>¿Cuántas comprobaciones de estado en mi punto de conexión puedo esperar de Traffic Manager?

El número de comprobaciones de estado de Traffic Manager que llega a su punto de conexión depende de lo siguiente:

- el valor que haya establecido para el intervalo de supervisión (un intervalo más pequeño significa más solicitudes que llegan a su punto de conexión en cualquier período de tiempo determinado);
- el número de ubicaciones desde donde se originan las comprobaciones de estado (las direcciones IP desde donde puede esperar estas comprobaciones se muestran en las preguntas más frecuentes anteriores).

### <a name="how-can-i-get-notified-if-one-of-my-endpoints-goes-down"></a>¿Cómo puedo recibir notificación de si uno de mis puntos de conexión deja de funcionar?

Una de las métricas que proporciona Traffic Manager es el estado de mantenimiento de los puntos de conexión de un perfil. Puede ver esto como un agregado de todos los puntos de conexión dentro de un perfil (por ejemplo, el 75 % de los puntos de conexión tienen un estado correcto) o en un nivel por punto de conexión. Las métricas de Traffic Manager se exponen a través de Azure Monitor y puede usar sus [funcionalidades de alerta](../monitoring-and-diagnostics/monitor-alerts-unified-usage.md) para recibir notificaciones cuando se produzca un cambio en el estado de mantenimiento del punto de conexión. Para obtener más detalles, consulte [Métricas y alertas de Traffic Manager](traffic-manager-metrics-alerts.md).  

## <a name="traffic-manager-nested-profiles"></a>Perfiles anidados de Traffic Manager

### <a name="how-do-i-configure-nested-profiles"></a>¿Cómo se configuran los perfiles anidados?

Los perfiles anidados de Traffic Manager se pueden configurar mediante Azure Resource Manager y las API de REST del portal clásico, cmdlets de Azure PowerShell y comandos de la CLI de Azure multiplataforma. También son compatibles con el nuevo Azure Portal.

### <a name="how-many-layers-of-nesting-does-traffic-manger-support"></a>¿Cuántas capas de anidamiento admite el Administrador de tráfico?

Puede anidar perfiles con hasta 10 niveles de profundidad. No se permiten "bucles".

### <a name="can-i-mix-other-endpoint-types-with-nested-child-profiles-in-the-same-traffic-manager-profile"></a>¿Puedo combinar otros tipos de puntos de conexión con perfiles secundarios anidados, en el mismo perfil del Administrador de tráfico?

Sí. No hay ninguna restricción en el modo de combinar puntos de conexión de diferentes tipos dentro de un perfil.

### <a name="how-does-the-billing-model-apply-for-nested-profiles"></a>¿Cómo se aplica el modelo de facturación para perfiles anidados?

El uso de perfiles anidados no afecta de forma negativa a los precios.

La facturación de Traffic Manager tiene dos componentes: comprobaciones de estado del punto de conexión y millones de consultas DNS.

* Comprobaciones de estado del punto de conexión: cuando un perfil secundario se configura como punto de conexión de un perfil primario no se aplica ningún cargo. La supervisión de los puntos de conexión del perfil secundario se factura de la manera habitual.
* Consultas de DNS: cada consulta se cuenta una sola vez. Las consultas a un perfil primario que devuelven un punto de conexión de un perfil secundario solo se cuentan con respecto al perfil primario.

Para obtener todos los detalles, consulte la [página de precios de Traffic Manager](https://azure.microsoft.com/pricing/details/traffic-manager/).

### <a name="is-there-a-performance-impact-for-nested-profiles"></a>¿Se ve afectado el rendimiento por el uso de perfiles anidados?

No. No hay ningún efecto sobre el rendimiento derivado del uso de perfiles anidados.

Los servidores de nombres de Traffic Manager atravesarán la jerarquía de perfil internamente cuando se procese cada consulta de DNS. Una consulta de DNS realizada a un perfil primario puede recibir una respuesta de DNS con un punto de conexión de un perfil secundario. Se usará un único registro CNAME tanto si usa un solo perfil como si usa perfiles anidados. No hay necesidad de crear un registro CNAME para cada perfil de la jerarquía.

### <a name="how-does-traffic-manager-compute-the-health-of-a-nested-endpoint-in-a-parent-profile"></a>¿Cómo calcula Traffic Manager el estado de un punto de conexión anidado en un perfil primario?

El perfil primario no realiza comprobaciones de estado en el perfil secundario directamente. En su lugar, el estado de los puntos de conexión del perfil secundario se usa para calcular el estado general del perfil secundario. Esta información se propaga hacia arriba en la jerarquía del perfil anidado para determinar el estado del punto de conexión anidado. El perfil primario usa este estado agregado para determinar si se puede dirigir el tráfico al perfil secundario.

En la tabla siguiente se describe el comportamiento de las comprobaciones de estado de Traffic Manager para un punto de conexión anidado.

| Estado de supervisión de perfiles secundarios | Estado de supervisión de extremos principales | Notas |
| --- | --- | --- |
| Deshabilitado. Se ha deshabilitado el perfil secundario. |Detenido |El estado del extremo primario es “Detenido”, no “Deshabilitado”. El estado “Deshabilitado” se reserva para indicar que el usuario ha deshabilitado el extremo en el perfil principal. |
| Degradado. Al menos un punto de conexión de perfil secundario se encuentra en estado Degradado. |En línea: el número de puntos de conexión en línea en el perfil secundario es, como mínimo, el valor de MinChildEndpoints.<BR>CheckingEndpoint: el número de puntos de conexión en línea más CheckingEndpoint en el perfil secundario es, como mínimo, el valor de MinChildEndpoints.<BR>Degradado: en caso contrario. |El tráfico se enruta a un punto de conexión con estado CheckingEndpoint. Si MinChildEndpoints se establece demasiado alto, el punto de conexión siempre se degradará. |
| En línea. Al menos un punto de conexión de perfil secundario se encuentra en estado En línea. Ningún punto de conexión está en estado Degradado. |Consulte más arriba. | |
| CheckingEndpoints. Al menos un punto de conexión de perfil secundario es "CheckingEndpoint". Ningún punto de conexión está "En línea" o "Degradado". |Igual que el anterior. | |
| Inactivo. Todos los puntos de conexión de perfil secundario están en estado "Deshabilitado" o "Detenido", o bien se trata de un perfil sin ningún punto de conexión. |Detenido | |

## <a name="next-steps"></a>Pasos siguientes:

- Obtenga más información sobre la [supervisión del punto de conexión y la conmutación por error automática](../traffic-manager/traffic-manager-monitoring.md) de Traffic Manager.
- Obtenga más información sobre los [métodos de enrutamiento del tráfico](../traffic-manager/traffic-manager-routing-methods.md) de Traffic Manager.
