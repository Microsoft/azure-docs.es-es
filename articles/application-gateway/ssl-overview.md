---
title: Habilitación de TLS de extremo a extremo en Azure Application Gateway
description: En este artículo se proporciona información general sobre la compatibilidad de Application Gateway con el protocolo TLS de extremo a extremo.
services: application-gateway
author: surajmb
ms.service: application-gateway
ms.topic: conceptual
ms.date: 08/21/2020
ms.author: victorh
ms.openlocfilehash: 3d714b579bebb096745a47410da3f8f458e27161
ms.sourcegitcommit: 5b6acff3d1d0603904929cc529ecbcfcde90d88b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88723306"
---
# <a name="overview-of-tls-termination-and-end-to-end-tls-with-application-gateway"></a>Introducción a la terminación TLS y a TLS de extremo a extremo con Application Gateway

Seguridad de la capa de transporte (TLS), antes conocido como Capa de sockets seguros (SSL), es la tecnología de seguridad estándar para establecer un vínculo cifrado entre un servidor web y un explorador. Este vínculo garantiza que todos los datos que se pasan entre el servidor web y los exploradores siguen siendo privados y continúan estando cifrados. Application Gateway admite tanto la terminación TLS en la puerta de enlace como el cifrado TLS de un extremo a otro.

## <a name="tls-termination"></a>Finalización de TLS

Application Gateway admite la terminación TLS en la puerta de enlace, tras lo cual el tráfico fluye normalmente sin cifrar a los servidores back-end. Llevar a cabo la terminación TLS en la puerta de enlace de aplicaciones tiene una serie de ventajas:

- **Rendimiento mejorado**: el mayor rendimiento al realizar el descifrado TLS es el protocolo de enlace inicial. Para mejorar el rendimiento, el servidor que realiza el descifrado almacena en caché los identificadores de sesión TLS y administra los vales de sesión TLS. Si esto se hace en la puerta de enlace de aplicaciones, todas las solicitudes del mismo cliente pueden usar los valores almacenados en caché. Si se hace en los servidores back-end, cada vez que las solicitudes del cliente vayan a un servidor diferente, el cliente tendrá que volver a autenticarse. El uso de tickets TLS puede ayudar a mitigar este problema, pero no todos los clientes los admiten y pueden resultar difícil de configurar y administrar.
- **Mejor utilización de los servidores back-end**: el procesamiento SSL/TLS hace un uso intensivo de CPU, y este uso cada vez es mayor a medida que aumentan los tamaños de clave. La eliminación de este trabajo de los servidores back-end les permite centrarse en aquello en lo que son más eficientes: entregar contenido.
- **Enrutamiento inteligente**: mediante el descifrado del tráfico, la puerta de enlace de aplicaciones tiene acceso al contenido de la solicitud, como encabezados, URI, etc., y puede utilizar estos datos para enrutar las solicitudes.
- **Administración de certificados**: los certificados solo se deben comprar e instalar en la puerta de enlace de aplicaciones y no en todos los servidores back-end. Esto ahorra tiempo y dinero.

Para configurar la terminación TLS, debe agregar un certificado TLS/SSL al cliente de escucha para permitir a Application Gateway derivar una clave simétrica según la especificación del protocolo TLS/SSL. A continuación, la clave simétrica se usa para cifrar y descifrar el tráfico que se envía a la puerta de enlace. El certificado TLS/SSL debe tener el formato Personal Information Exchange (PFX). Este formato de archivo permite la exportación de la clave privada, lo que es necesario para que la puerta de enlace de aplicaciones pueda realizar el cifrado y descifrado del tráfico.

> [!IMPORTANT] 
> El certificado del cliente de escucha requiere que se cargue toda la cadena de certificados (el certificado raíz de la entidad de certificación, los certificados intermedios y el certificado de hoja) para establecer la cadena de confianza. 


> [!NOTE] 
>
> La puerta de enlace de aplicaciones no proporciona ninguna funcionalidad para crear un nuevo certificado o enviar una solicitud de certificado a una entidad de certificación.

Para que la conexión TLS para funcione, deberá asegurarse de que el certificado TLS/SSL cumple las condiciones siguientes:

- La fecha y hora actuales están dentro del intervalo de fechas "Válido desde" y "Válido hasta" en el certificado.
- El "nombre común" del certificado coincide con el encabezado de host en la solicitud. Por ejemplo, si el cliente realiza una solicitud a `https://www.contoso.com/`, el certificado común debe ser `www.contoso.com`.

### <a name="certificates-supported-for-tls-termination"></a>Certificados compatibles con la terminación TLS

Application Gateway admite los siguientes tipos de certificados:

- Certificado CA (entidad de certificación): un certificado CA es un certificado digital emitido por una entidad de certificación (CA)
- Certificado EV (validación extendida): un certificado EV es un certificado que cumple las pautas de certificado estándar del sector. Esto cambiará a verde la barra del localizador del explorador y publicará también el nombre de la compañía.
- Certificado comodín: este certificado admite cualquier número de subdominios en función de *. site.com, donde el subdominio reemplazaría el asterisco (*). Sin embargo, no admite site.com, por lo que en caso de que los usuarios accedan a su sitio web sin escribir las "www" iniciales, el certificado comodín no cubrirá eso.
- Certificados autofirmados: los exploradores de cliente no confían en estos certificados y avisarán al usuario de que el certificado del servicio virtual no forma parte de una cadena de confianza. Los certificados autofirmados son buenos para las pruebas o los entornos donde los administradores controlan a los clientes y pueden omitir de forma segura las alertas de seguridad del explorador. Las cargas de trabajo de producción nunca deben usar certificados autofirmados.

Para más información, consulte la [configuración de la terminación TLS con una puerta de enlace de aplicaciones](https://docs.microsoft.com/azure/application-gateway/create-ssl-portal).

### <a name="size-of-the-certificate"></a>Tamaño del certificado
Consulte la sección [Límites de Application Gateway](https://docs.microsoft.com/azure/azure-resource-manager/management/azure-subscription-service-limits#application-gateway-limits) para conocer el tamaño máximo de certificado TLS/SSL admitido.

## <a name="end-to-end-tls-encryption"></a>Cifrado TLS de un extremo a otro

Es posible que no desee una comunicación sin cifrar con los servidores back-end. Puede que tenga que cumplir requisitos de seguridad y cumplimiento normativo o quizás la aplicación solo acepte una conexión segura. Azure App Gateway tiene cifrado TLS de un extremo a otro para ajustarse a estos requisitos.

El cifrado TLS de un extremo a otro permite cifrar y transmitir datos confidenciales de forma segura al back-end mientras usa las características de equilibrio de carga de nivel 7 de Application Gateway. Entre estas características se incluyen la afinidad de la sesión basada en las cookies, el enrutamiento basado en direcciones URL, la compatibilidad para el enrutamiento basado en sitios, la capacidad para reescribir o insertar encabezados X-Forwarded-*, etc.

Cuando se configura con el modo de comunicación de TLS de un extremo a otro, Application Gateway finaliza las sesiones TLS en la puerta de enlace y descifra el tráfico de usuario. Después, aplica las reglas configuradas para seleccionar una instancia de grupo de back-end adecuada en la que enrutar el tráfico. Posteriormente, Application Gateway inicia una nueva conexión TLS al servidor back-end y vuelve a cifrar los datos mediante el certificado de clave pública del servidor back-end antes de transmitir la solicitud al back-end. Cualquier respuesta del servidor web pasa por el mismo proceso en su regreso al usuario final. El protocolo TLS de un extremo a otro se habilita estableciendo el valor de protocolo de [Configuración HTTP del back-end](https://docs.microsoft.com/azure/application-gateway/configuration-overview#http-settings) en HTTPS, que luego se aplica a un grupo de back-end.

En el caso de la SKU de WAF v1 y Application Gateway, la directiva de TLS se aplica al tráfico de front-end y de back-end. En el front-end, Application Gateway actúa como el servidor y exige la directiva. En el back-end, Application Gateway actúa como cliente y envía la información de protocolo o cifrado como la preferencia durante el protocolo de enlace TLS.

En el caso de la SKU de WAF v2 y Application Gateway, la directiva de TLS se aplica solo al tráfico de front-end y se ofrecen todos los cifrados en el servidor back-end, que tiene control para seleccionar cifrados específicos y la versión de TLS durante el enlace.

Application Gateway solo se comunica con los servidores back-end que han incluido su certificado en la lista de permitidos con Application Gateway o cuyos certificados están firmados por autoridades de entidad de certificación perfectamente conocidas y el nombre común del certificado coincide con el nombre del host en la configuración del back-end HTTP. Estas incluyen los servicios de Azure de confianza, como Azure App Service/Web Apps y Azure API Management.

Si los certificados de los miembros del grupo de back-end no están firmados por autoridades de entidad de certificación perfectamente conocidas, cada instancia del grupo de back-end con la funcionalidad TLS de un extremo a otro habilitada debe configurarse con un certificado para permitir una comunicación segura. Al agregar el certificado, se garantiza que la puerta de enlace de aplicaciones solo se comunique con instancias back-end conocidas. Esto protege aún más la comunicación de un extremo a otro.

> [!NOTE] 
>
> El certificado agregado en **Configuración HTTP del back-end** para autenticar los servidores back-end puede ser el mismo que el certificado agregado al **cliente de escucha** para la terminación TLS en la puerta de enlace de aplicaciones o diferente, para mayor seguridad.

![Escenario de TLS de un extremo a otro][1]

En este ejemplo, las solicitudes mediante TLS1.2 se enrutan a los servidores back-end en Pool1 con TLS de un extremo a otro.

## <a name="end-to-end-tls-and-allow-listing-of-certificates"></a>TLS de un extremo a otro e incorporación a listas de permitidos de certificados

Application Gateway solo se comunica con instancias de back-end conocidas, que tienen en la lista de permitidos su certificado con Application Gateway. Existen algunas diferencias en el proceso de configuración de TLS de un extremo a otro con respecto a la versión de Application Gateway utilizada. En la siguiente sección se explican individualmente.

## <a name="end-to-end-tls-with-the-v1-sku"></a>TLS de un extremo a otro con la SKU v1

Para habilitar TLS de un extremo a otro con los servidores back-end y para que Application Gateway enrute las solicitudes hacia ellos, los sondeos de estado deben completarse correctamente y devolver una respuesta correcta.

En el caso de los sondeos de estado de HTTPS, la SKU de Application Gateway v1 usa una coincidencia exacta del certificado de autenticación (clave pública del certificado del servidor back-end y no del certificado raíz) que se va a cargar en la configuración HTTP.

Solo se permiten conexiones en back-ends conocidos y en la lista de permitidos. Los sondeos de estado consideran incorrectos los back-ends restantes. Los certificados autofirmados son para fines de prueba que y no se recomiendan para cargas de trabajo de producción. Antes de poder usar estos certificados, deben estar en la lista de permitidos con Application Gateway, tal y como se describe en los pasos anteriores.

> [!NOTE]
> La configuración del certificado raíz de confianza y de autenticación no es necesaria para servicios de Azure de confianza, como Azure App Service. Estos servicios se consideran de confianza de forma predeterminada.

## <a name="end-to-end-tls-with-the-v2-sku"></a>TLS de un extremo a otro con la SKU v2

Los certificados de autenticación han dejado de usarse, y se han reemplazado por certificados raíz de confianza en la SKU v2 de Application Gateway. Funcionan de forma similar a los certificados de autenticación, aunque con algunas diferencias importantes:

- Los certificados firmados por autoridades de entidad de certificación conocidas cuyo CN coincida con el nombre de host de la configuración de back-end HTTP no requieren ningún paso adicional para que el protocolo TLS de extremo a extremo funcione. 

   Por ejemplo, si los certificados de back-end están emitidos por una entidad de certificación conocida y tienen un CN de contoso.com, y también se establece el campo de host de la configuración de HTTP de back-end en contoso.com, no se requiere ningún otro paso adicional. Puede establecer el protocolo de configuración HTTP del back-end en HTTPS y tanto el sondeo de estado como la ruta de acceso de los datos estarían habilitados para TLS. Si utiliza Azure App Service u otros servicios web de Azure como back-end, estos también serían de confianza de manera implícita y no se necesitarían pasos adicionales para TLS de extremo a extremo.
   
> [!NOTE] 
>
> Para que un certificado TLS/SSL sea de confianza, ese certificado del servidor back-end debe haber sido emitido por una entidad de certificación conocida. Si el certificado no lo emitió una entidad de certificación de confianza, Application Gateway comprobará si el certificado de la entidad de certificación emisora lo emitió una entidad de certificación de confianza y así sucesivamente hasta encontrar una entidad de certificación de confianza (momento en el que se establecerá una conexión segura y de confianza) o hasta no encontrar una entidad de certificación de confianza (en cuyo caso Application Gateway marcará el back-end como incorrecto). Por lo tanto, se recomienda que el certificado de servidor de back-end contenga tanto la CA raíz como la intermedia.

- Si el certificado está autofirmado o lo han firmado intermediarios desconocidos, tendrá que definir un certificado raíz de confianza para habilitar TLS de un extremo a otro en la SKU v2. La instancia de Application Gateway se comunica únicamente con back-ends cuyo certificado raíz del certificado del servidor coincida con alguno de la lista de certificados raíz de confianza en la configuración HTTP de back-end asociada con el grupo.

- Además de la coincidencia del certificado raíz, Application Gateway v2 también valida si el valor de host especificado en la configuración de HTTP del back-end coincide con el del nombre común (CN) presentado por el certificado TLS/SSL del servidor back-end. Cuando se intenta establecer una conexión TLS con el back-end, Application Gateway v2 establece la extensión Indicación de nombre de servidor (SNI) en el host especificado en la configuración de HTTP de back-end.

- Si se elige la opción de **elegir el nombre de host a partir del destino de back-end** en lugar del campo de host en la configuración de HTTP de back-end, el encabezado SNI siempre se establece en el nombre de dominio completo del grupo del back-end y el CN del certificado TLS/SSL del servidor back-end debe coincidir con su nombre de dominio completo. Los miembros del grupo de back-end con direcciones IP no se admiten en este escenario.

- El certificado raíz es un certificado raíz codificado en base64 a partir de los certificados de servidor de back-end.

## <a name="sni-differences-in-the-v1-and-v2-sku"></a>Diferencias de SNI en las SKU v1 y v2

Como se ha indicado anteriormente, Application Gateway finaliza el tráfico TLS desde el cliente en el cliente de escucha de Application Gateway (que denominaremos conexión front-end), descifra el tráfico, aplica las reglas necesarias para determinar el servidor back-end al que se tiene que reenviar la solicitud y establece una nueva sesión de TLS con el servidor back-end (que denominaremos conexión back-end).

En las tablas siguientes se describen las diferencias de SNI entre las SKU v1 y v2 en lo que respecta a las conexiones front-end y back-end.

### <a name="frontend-tls-connection-client-to-application-gateway"></a>Conexión TLS front-end (cliente a Application Gateway)

---
Escenario | v1 | v2 |
| --- | --- | --- |
| Si el cliente especifica un encabezado SNI y todos los clientes de escucha multisitio están habilitados con la marca "Requerir SNI" | Devuelve el certificado adecuado y, si el sitio no existe (según server_name), se restablece la conexión. | Devuelve el certificado adecuado si está disponible; de lo contrario, devuelve el certificado del primer cliente de escucha HTTPS configurado (en el orden).|
| Si el cliente no especifica un encabezado SNI y si todos los encabezados multisitio están habilitados con la marca "Requerir SNI" | Restablece la conexión. | Devuelve el certificado del primer cliente de escucha HTTPS configurado (en el orden).
| Si el cliente no especifica el encabezado SNI y si hay un cliente de escucha básico configurado con un certificado | Devuelve el certificado configurado en el cliente de escucha básico al cliente (certificado predeterminado o de reserva). | Devuelve el certificado del primer cliente de escucha HTTPS configurado (en el orden). |

### <a name="backend-tls-connection-application-gateway-to-the-backend-server"></a>Conexión TLS de back-end (Application Gateway al servidor back-end)

#### <a name="for-probe-traffic"></a>Para el tráfico de sondeo

---
Escenario | v1 | v2 |
| --- | --- | --- |
| Encabezado SNI (server_name) durante el enlace TLS como FQDN | Se establece como FQDN desde el grupo de back-end. Según el estándar [RFC 6066](https://tools.ietf.org/html/rfc6066), no se permiten direcciones IPv4 y IPv6 literales en el nombre de host de SNI. <br> **Nota:** El FQDN del grupo de back-end debe resolver el DNS en la dirección IP del servidor back-end (pública o privada) | El encabezado SNI (server_name) se establece como el nombre de host del sondeo personalizado asociado a la configuración HTTP (si se configuró) o del nombre de host mencionado en la configuración HTTP o del FQDN mencionado en el grupo de back-end. El orden de prioridad es el siguiente: sondeo personalizado > configuración HTTP > grupo de back-end. <br> **Nota:** Si los nombres de host configurados en la configuración HTTP y el sondeo personalizado son diferentes, de acuerdo con la prioridad, la SNI se establecerá como el nombre de host del sondeo personalizado.
| Si la dirección del grupo de back-end es una dirección IP (v1) o si el nombre de host del sondeo personalizado está configurado como dirección IP (v2) | No se establecerá la SNI (server_name). <br> **Nota:** En este caso, el servidor back-end debe ser capaz de devolver un certificado predeterminado o de reserva, que debe incluirse en la lista de permitidos en la configuración HTTP como certificado de autenticación. Si no hay ningún certificado predeterminado o de reserva configurado en el servidor back-end y se espera la SNI, el servidor podría restablecer la conexión y provocar errores de sondeo. | En el orden de prioridad mencionado anteriormente, si se tiene una dirección IP como nombre de host, no se establecerá la SNI de conformidad con el estándar [RFC 6066](https://tools.ietf.org/html/rfc6066). <br> **Nota:** Tampoco se establecerá la SNI en sondeos de v2 si no se ha configurado ningún sondeo personalizado y no se ha establecido ningún nombre de host en la configuración HTTP o en el grupo de back-end |

> [!NOTE] 
> Si no se configura un sondeo personalizado, Application Gateway envía un sondeo predeterminado con este formato: \<protocol\>://127.0.0.1:\<port\>/. Por ejemplo, para un sondeo HTTPS predeterminado, se enviará como https://127.0.0.1:443/. Tenga en cuenta que la dirección 127.0.0.1 mencionada aquí solo se usa como encabezado host HTTP y, de conformidad con el estándar RFC 6066, no se usará como encabezado SNI. Para más información acerca de los errores de sondeo de estado, consulte la [guía de solución de problemas de estado del back-end](application-gateway-backend-health-troubleshooting.md).

#### <a name="for-live-traffic"></a>Para el tráfico en vivo

---
Escenario | v1 | v2 |
| --- | --- | --- |
| Encabezado SNI (server_name) durante el enlace TLS como FQDN | Se establece como FQDN desde el grupo de back-end. Según el estándar [RFC 6066](https://tools.ietf.org/html/rfc6066), no se permiten direcciones IPv4 y IPv6 literales en el nombre de host de SNI. <br> **Nota:** El FQDN del grupo de back-end debe resolver el DNS en la dirección IP del servidor back-end (pública o privada) | El encabezado SNI (server_name) se establece como nombre de host desde la configuración HTTP; de lo contrario, si se selecciona la opción *PickHostnameFromBackendAddress* o si no se menciona ningún nombre de host, se establecerá como el FQDN en la configuración del grupo de back-end.
| Si la dirección del grupo de back-end es una dirección IP o el nombre de host no está establecido en la configuración HTTP | No se establecerá la SNI de conformidad con el estándar [RFC 6066](https://tools.ietf.org/html/rfc6066) si la entrada del grupo de back-end no es un FQDN | La SNI se establecerá como el nombre de host del FQDN de entrada del cliente y el CN del certificado de back-end debe coincidir con este nombre de host.

## <a name="next-steps"></a>Pasos siguientes

Después aprender acerca de TLS de un extremo a otro, consulte [Configuración de TLS de un extremo a otro con Application Gateway mediante PowerShell](application-gateway-end-to-end-ssl-powershell.md) para crear una puerta de enlace de aplicaciones utilizando TLS de un extremo a otro.

<!--Image references-->

[1]: ./media/ssl-overview/scenario.png
