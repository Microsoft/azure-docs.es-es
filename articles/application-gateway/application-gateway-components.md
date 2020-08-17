---
title: Componentes de Application Gateway
description: En este artículo se proporciona información sobre los distintos componentes de una puerta de enlace de aplicaciones
services: application-gateway
author: abshamsft
ms.service: application-gateway
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: absha
ms.openlocfilehash: 20d43666919f8528c25735592c2727601af10bbb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088094"
---
# <a name="application-gateway-components"></a>Componentes de Application Gateway

 Una puerta de enlace de aplicaciones sirve como el único punto de contacto para los clientes. Se encarga de distribuir el tráfico entrante de las aplicaciones entre varios grupos de servidores back-end, como máquinas virtuales de Azure, conjuntos de escalado de máquinas virtuales, Azure App Service y servidores externos y locales. Para ello, emplea varios componentes que se describen en este artículo.

![Componentes usados en una puerta de enlace de aplicaciones](./media/application-gateway-components/application-gateway-components.png)

## <a name="frontend-ip-addresses"></a>Direcciones IP de front-end

Una dirección IP de front-end es la dirección IP asociada con una puerta de enlace de aplicaciones. Puede configurar una puerta de enlace de aplicaciones para que tenga una dirección IP pública, una dirección IP privada o ambas. Una puerta de enlace de aplicaciones admite una dirección IP privada y una pública. La red virtual y la dirección IP pública deben estar en la misma ubicación que la puerta de enlace de aplicaciones. Después de crearla, la dirección IP de front-end se asocia con un cliente de escucha.

### <a name="static-versus-dynamic-public-ip-address"></a>Dirección IP pública estática frente a dinámica

La SKU de Azure Application Gateway V2 se puede configurar para admitir tanto una dirección IP interna estática como una dirección IP pública estática, o solo una dirección IP pública estática. No se puede configurar para admitir solo una dirección IP estática interna.

La SKU V1 se puede configurar para admitir una dirección IP interna estática o dinámica y una dirección IP pública dinámica. La dirección IP dinámica de Application Gateway no cambia en una puerta de enlace en ejecución. Puede cambiar solo cuando se detiene o inicia Gateway. No cambia cuando se producen errores del sistema, actualizaciones, actualizaciones del host de Azure, etc. 

El nombre DNS asociado a una puerta de enlace de aplicaciones no cambia durante el ciclo de vida de la puerta de enlace. Como resultado, debe usar un alias CNAME y hacer que apunte a la dirección DNS de la puerta de enlace de aplicaciones.

## <a name="listeners"></a>Clientes de escucha

Un cliente de escucha es una entidad lógica que comprueba si hay solicitudes de conexión entrantes. Un cliente de escucha acepta una solicitud si el protocolo, el puerto, el nombre de host y la dirección IP asociados a la solicitud coinciden con los mismos elementos asociados a la configuración del cliente de escucha.

Antes de usar una puerta de enlace de aplicaciones, debe agregar al menos un cliente de escucha. Puede haber varios clientes de escucha asociados a una puerta de enlace de aplicaciones, y se pueden usar con el mismo protocolo.

Una vez que un cliente de escucha detecta las solicitudes entrantes de clientes, la puerta de enlace de aplicaciones enruta estas solicitudes a los miembros del grupo de servidores back-end configurado en esta regla.

Los clientes de escucha admiten los siguientes puertos y protocolos.

### <a name="ports"></a>Puertos

Un puerto es donde un cliente de escucha realiza escuchas de la solicitud de cliente. Puede configurar puertos entre 1 y 65502 para la SKU v1 y entre 1 y 65199 para la SKU v2.

### <a name="protocols"></a>Protocolos

Application Gateway admite cuatro protocolos: HTTP, HTTPS, HTTP/2 y WebSocket:
>[!NOTE]
>La compatibilidad con el protocolo HTTP/2 está disponible únicamente para los clientes que se conectan a los agentes de escucha de la puerta de aplicaciones. La comunicación con grupos de servidores back-end es a través de HTTP/1.1. De forma predeterminada, HTTP/2 está deshabilitado. Pero puede elegir habilitarlo.

- Especifique entre los protocolos HTTP y HTTPS en la configuración del cliente de escucha.
- La compatibilidad con los [protocolos WebSockets y HTTP/2](features.md#websocket-and-http2-traffic) se proporciona de forma nativa, y la [compatibilidad con WebSocket](application-gateway-websocket.md) está habilitada de forma predeterminada. No hay ninguna opción de configuración que permita al usuario habilitar o deshabilitar la compatibilidad con WebSocket. Use WebSockets con clientes de escucha HTTP y HTTPS.

Use un cliente de escucha HTTPS para la terminación TLS. Un cliente de escucha HTTPS descarga el trabajo de cifrado y descifrado en la puerta de enlace de aplicaciones, por lo que los servidores web no se ven afectados por la sobrecarga.

### <a name="custom-error-pages"></a>Páginas de error personalizadas

Application Gateway permite crear páginas de error personalizadas, en lugar de mostrar las páginas de error predeterminadas. Mediante una página de error personalizada puede usar su propia marca y diseño. Application Gateway muestra una página de error personalizada cuando una solicitud no puede conectar con el back-end.

Para más información, consulte [Creación de páginas de error personalizadas de Application Gateway](custom-error.md).

### <a name="types-of-listeners"></a>Tipos de clientes de escucha

Hay dos tipos de clientes de escucha:

- **Básico**. Este tipo de cliente de escucha realiza escuchas en un único sitio de dominio, donde tiene una única asignación de DNS a la dirección IP de la puerta de enlace de aplicaciones. Esta configuración de cliente de escucha es necesaria cuando se realiza el hospedaje en un único sitio detrás de una puerta de enlace de aplicaciones.

- **Multisitio**. Esta configuración del agente de escucha es obligatoria cuando busca configurar el enrutamiento según el nombre de host o el nombre de dominio de más de una aplicación web en la misma puerta de enlace de aplicación. Permite configurar una topología más eficaz para las implementaciones al agregar hasta 100 sitios web a una puerta de enlace de aplicaciones. Cada sitio web se puede dirigir a su propio grupo de back-end. Por ejemplo, tres dominios, contoso.com, fabrikam.com y adatum.com, señalan a la dirección IP de la puerta de enlace de aplicaciones. Crearía tres [clientes de escucha multisitio](multiple-site-overview.md) y configuraría cada uno con la configuración respectiva de protocolo y puerto. 

    También puede definir nombres de host con el carácter comodín en un cliente de escucha de varios sitios y hasta cinco nombres de host por cliente de escucha. Para obtener más información, consulte los [nombres de host comodín en el cliente de escucha (versión preliminar)](multiple-site-overview.md#wildcard-host-names-in-listener-preview).

    Para obtener más información sobre cómo configurar un agente de escucha multisitio, consulte [Hospedaje multisitio en Application Gateway mediante Azure Portal](create-multiple-sites-portal.md).

Después de crear un cliente de escucha, asócielo a una regla de enrutamiento de solicitudes. Esta regla determina cómo se debe enrutar la solicitud recibida en el cliente de escucha hacia el back-end. La regla de enrutamiento de solicitud también contiene el grupo de back-end al que se va a enrutar y la configuración de HTTP donde se mencionan el puerto de back-end, el protocolo, etc.

## <a name="request-routing-rules"></a>Reglas de enrutamiento de solicitudes

Una regla de enrutamiento de solicitudes es un componente clave de una puerta de enlace de aplicaciones, ya que determina cómo se enruta el tráfico en el cliente de escucha. La regla enlaza el cliente de escucha, el grupo de servidores back-end y la configuración HTTP de back-end.

Cuando un cliente de escucha acepta una solicitud, la regla de enrutamiento de solicitudes reenvía la solicitud al back-end o la redirige a otra parte. Si la solicitud se reenvía al back-end, la regla de enrutamiento de solicitudes define a qué grupo de servidores back-end reenviarla. Esta regla determina también si se van a reescribir los encabezados de la solicitud. Un cliente de escucha se puede asociar a una regla.

Hay dos tipos de reglas de enrutamiento de solicitudes:

- **Básico**. Todas las solicitudes del cliente de escucha asociado (por ejemplo, blog.contoso.com/*) se reenvían al grupo back-end asociado mediante la configuración HTTP asociada.

- **Basadas en la ruta de acceso**. Esta regla de enrutamiento le permite enrutar las solicitudes del cliente de escucha asociado a un grupo back-end específico, según la dirección URL de la solicitud. Si la ruta de acceso de la dirección URL de una solicitud coincide con el patrón de ruta de acceso de una regla basada en la ruta de acceso, la regla enruta esa solicitud. El patrón de ruta de acceso solo se aplica a la ruta de acceso de dirección URL, no a sus parámetros de consulta. Si la ruta de acceso de dirección URL de una solicitud del cliente de escucha no coincide con ninguna de las reglas basadas en la ruta de acceso, enruta la solicitud al grupo back-end y la configuración de HTTP predeterminados.

Para más información, consulte [Enrutamiento basado en la dirección URL](url-route-overview.md).

### <a name="redirection-support"></a>Compatibilidad con la redirección

La regla de enrutamiento de solicitudes también permite redirigir el tráfico en la puerta de enlace de aplicaciones. Este es un mecanismo de redireccionamiento genérico, por lo que puede redirigir desde y hacia cualquier puerto que defina mediante reglas.

Puede elegir que el destino de redireccionamiento sea otro cliente de escucha (lo que puede ayudar el redireccionamiento automático de HTTP a HTTPS) o un sitio externo. También puede elegir que el redireccionamiento sea temporal o permanente, o anexar la ruta de acceso del URI y cadena de consulta a la dirección URL redirigida.

Para más información, consulte [Redireccionamiento del tráfico en la puerta de enlace de aplicaciones](redirect-overview.md).

### <a name="rewrite-http-headers-and-url"></a>Reescritura de encabezados HTTP y URL

Mediante las reglas de reescritura, puede agregar, quitar o actualizar encabezados de solicitud y respuesta HTTP(S), así como parámetros de ruta de acceso URL y cadena de consulta, dado que los paquetes de solicitud y respuesta se mueven entre el cliente y los grupos back-end a través de la puerta de enlace de aplicaciones.

Los parámetros de URL y encabezado se pueden establecer en valores estáticos o en otros encabezados y variables de servidor. Como consecuencia, sirve de ayuda en casos de uso importantes, como la extracción de direcciones IP de cliente, la eliminación de información confidencial sobre el back-end, la adición de más seguridad, etc.

Para más información, consulte [Rescritura de encabezados HTTP y direcciones URL en la puerta de enlace de aplicación](rewrite-http-headers-url.md).

## <a name="http-settings"></a>Configuración de HTTP

Una puerta de enlace de aplicaciones enruta el tráfico a los servidores back-end (especificados en la regla de enrutamiento de solicitudes que incluye la configuración de HTTP) mediante el número de puerto, el protocolo y demás configuración detallados en este componente.

El puerto y el protocolo usados en la configuración de HTTP determinan si el tráfico entre los servidores back-end y de puerta de enlace de aplicaciones está cifrado (con TLS de un extremo a otro) o sin cifrar.

Este componente también se usa para:

- Determinar si una sesión de usuario se va a mantener en el mismo servidor mediante la [afinidad de sesión basada en cookies](features.md#session-affinity).

- Quitar miembros del grupo back-end correctamente con la [purga de conexiones](features.md#connection-draining).

- Asociar un sondeo personalizado para supervisar el mantenimiento del back-end, establecer el intervalo de tiempo de expiración de las solicitudes, reemplazar el nombre de host y la ruta de acceso de la solicitud y proporcionar facilidad de uso con un clic para especificar la configuración del back-end de App Service.

## <a name="backend-pools"></a>Grupos back-end

Un grupo back-end enruta la solicitud a los servidores back-end, quienes la atienden. Los grupos back-end pueden contener:

- NIC
- Conjuntos de escalado de máquinas virtuales
- Direcciones IP públicas
- Direcciones IP internas
- FQDN
- Servidores back-end multiinquilino (por ejemplo, App Service)

Los miembros del grupo back-end de Application Gateway no están asociados a un conjunto de disponibilidad. Una puerta de enlace de aplicaciones puede comunicarse con instancias de fuera de la red virtual en la que se encuentra. Como resultado, los miembros de los grupos back-end pueden estar en los clústeres, en los centros de datos o fuera de Azure, siempre que haya conectividad IP.

Si usa direcciones IP internas como miembros del grupo back-end, debe usar [emparejamiento de redes virtuales](../virtual-network/virtual-network-peering-overview.md) o una [puerta de enlace VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md). El emparejamiento de red virtual se admite y es beneficioso para el tráfico de equilibrio de carga en otras redes virtuales.

Una puerta de enlace de aplicaciones también puede comunicarse con servidores locales cuando están conectados mediante Azure ExpressRoute o túneles VPN si se permite el tráfico.

Puede crear grupos back-end diferente para distintos tipos de solicitudes. Por ejemplo, puede crear un grupo back-end para las solicitudes generales y otro grupo back-end para las solicitudes a los microservicios de la aplicación.

## <a name="health-probes"></a>Sondeos de estado

De forma predeterminada, una puerta de enlace de aplicaciones supervisa el estado de todos los recursos de su grupo back-end y elimina automáticamente los que tienen un estado incorrecto. A continuación, supervisa las instancias incorrectas y las agrega de vuelta al grupo back-end en buen estado cuando está disponible y responde a los sondeos de estado.

Aparte del uso de la supervisión del sondeo de estado, también puede personalizar el sondeo de estado para adaptarlo a las necesidades de su aplicación. Los sondeos personalizados permiten un control más pormenorizado sobre la supervisión del estado. Cuando se usan sondeos personalizados, puede configurar un nombre de host personalizado, la ruta de acceso de la dirección URL, el intervalo de sondeo y la cantidad de respuestas erróneas que se aceptan antes de marcar la instancia del grupo de back-end como en mal estado, la coincidencia de códigos de estado personalizados y cuerpos de respuesta, etc. Se recomienda configurar sondeos personalizados para supervisar el estado de cada grupo back-end.

Para más información, consulte [Supervisión del estado de la puerta de enlace de aplicaciones](../application-gateway/application-gateway-probe-overview.md).

## <a name="next-steps"></a>Pasos siguientes

Creación de una puerta de enlace de aplicaciones:

* [En Azure Portal](quick-create-portal.md)
* [Mediante Azure PowerShell](quick-create-powershell.md)
* [Mediante la CLI de Azure](quick-create-cli.md).
