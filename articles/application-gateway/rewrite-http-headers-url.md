---
title: Reescritura de encabezados HTTP y direcciones URL con Azure Application Gateway | Microsoft Docs
description: En este artículo, se proporciona información general sobre la reescritura de encabezados HTTP y direcciones URL en Azure Application Gateway
services: application-gateway
author: surajmb
ms.service: application-gateway
ms.topic: conceptual
ms.date: 07/16/2020
ms.author: surmb
ms.openlocfilehash: 2ee34e1a7959aafa5db949b443fd58cca58719c6
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281198"
---
# <a name="rewrite-http-headers-and-url-with-application-gateway"></a>Reescritura de los encabezados HTTP y direcciones URL con Application Gateway

 Application Gateway permite volver a escribir el contenido seleccionado de las solicitudes y las respuestas. Con esta característica, puede traducir las direcciones URL y los parámetros de cadenas de consulta, además de modificar los encabezados de solicitud y respuesta. También permite agregar las condiciones necesarias para asegurarse de que los encabezados especificados y las direcciones URL se reescriben solo cuando se cumplen ciertas condiciones. Estas condiciones se basan en la información de solicitud y respuesta.

>[!NOTE]
>Las características de encabezados HTTP y reescritura de direcciones URL solo están disponibles para la [SKU de Application Gateway v2](application-gateway-autoscaling-zone-redundant.md)

## <a name="rewrite-types-supported"></a>Tipos de reescritura admitidos

### <a name="request-and-response-headers"></a>Encabezados de respuesta y solicitud

Los encabezados HTTP permiten que el cliente y el servidor pasen información adicional con la solicitud o la respuesta. Al reescribir estos encabezados, puede realizar tareas importantes, como agregar campos de encabezado relacionados con la seguridad (como HSTS y X-XSS-Protection), quitar campos de encabezado de respuesta que pueden revelar información confidencial o eliminar información de puertos de los encabezados X-Forwarded-For.

Application Gateway permite agregar, quitar o actualizar los encabezados de respuesta y de solicitudes HTTP, mientras los paquetes de solicitudes y respuestas se mueven entre los grupos de back-end y de cliente.

Para obtener información sobre cómo volver a escribir los encabezados de solicitud y respuesta con Application Gateway mediante Azure Portal, consulte [aquí](rewrite-url-portal.md).

![img](./media/rewrite-http-headers-url/header-rewrite-overview.png)


**Encabezados admitidos**

Puede volver a escribir todos los encabezados de las solicitudes y las respuestas, excepto para los encabezados Connection y Upgrade. También puede usar la puerta de enlace de aplicaciones para crear los encabezados personalizados y agregarlos a las solicitudes y respuestas que se enrutan por ella.

### <a name="url-path-and-query-string-preview"></a>Ruta de acceso URL y cadena de consulta (versión preliminar)

Con la funcionalidad de reescritura de direcciones URL en Application Gateway, puede:

* Volver a escribir el nombre de host, la ruta de acceso y la cadena de consulta de la dirección URL de solicitud. 

* Elegir volver a escribir la dirección URL de todas las solicitudes de un agente de escucha o solo de las que coincidan con una o varias de las condiciones que establezca. Estas condiciones se basan en las propiedades de solicitud y respuesta (solicitud, encabezado, encabezado de respuesta y variables de servidor).

* Elija la opción de enrutar la solicitud (seleccione el grupo de back-end) en función de la dirección URL original o la dirección URL reescrita.

Para obtener información sobre cómo reescribir la dirección URL con Application Gateway mediante Azure Portal, consulte [aquí](rewrite-url-portal.md).

![img](./media/rewrite-http-headers-url/url-rewrite-overview.png)

>[!NOTE]
> La característica de reescritura de direcciones URL está en versión preliminar y solo está disponible para las SKU Standard_v2 y WAF_v2 de Application Gateway. No es aconsejable en entornos de producción. Para obtener más información sobre las versiones preliminares, consulte los [términos de uso aquí](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="rewrite-actions"></a>Acciones de reescritura

Las acciones de reescritura se usan para especificar la dirección URL, los encabezados de solicitud o los encabezados de respuesta que se desea volver a escribir y el nuevo valor para el que se pretenden volver a escribir. El valor de una nueva dirección URL o un encabezado nuevo o existente se pueden establecer para estos tipos de valores:

* Texto
* Encabezado de la solicitud. Para especificar un encabezado de solicitud, tiene que usar la sintaxis siguiente: {http_req_*nombreDeEncabezado*}.
* Encabezado de respuesta. Para especificar un encabezado de respuesta, tiene que usar la sintaxis {http_req_*nombreDeEncabezado*}.
* Variable de servidor. Para especificar una variable de servidor, tiene que usar la sintaxis {var_*serverVariable*}. Consulte la lista de variables de servidor admitidas.
* Una combinación de texto, un encabezado de solicitud, un encabezado de respuesta y una variable de servidor. 

## <a name="rewrite-conditions"></a>Condiciones de reescritura

Puede utilizar condiciones de reescritura, una configuración opcional, para evaluar el contenido de las solicitudes y las respuestas HTTP(S), y realizar una reescritura cuando se cumplan una o varias condiciones. La puerta de enlace de aplicaciones utiliza estos tipos de variables para evaluar el contenido de las respuestas y las solicitudes:

* Encabezados HTTP en la solicitud
* Encabezados HTTP en la respuesta
* Variables de servidor de Application Gateway

Puede usar una condición para evaluar si una variable especificada está presente o coincide con un valor específico o con un patrón específico. 


### <a name="pattern-matching"></a>Coincidencia de patrones 

Application Gateway usa expresiones regulares para la coincidencia de patrones en la condición. Puede usar la [biblioteca de expresiones regulares compatibles de Perl (PCRE)](https://www.pcre.org/) si desea configurar la coincidencia de los patrones de expresión regular en las condiciones. Para obtener información sobre la sintaxis de las expresiones regulares, vea la [página principal de las expresiones regulares de Perl](https://perldoc.perl.org/perlre.html).

### <a name="capturing"></a>Capturado

Para capturar una subcadena para usarse más adelante, inserte paréntesis alrededor del subpatrón que coincida en la definición de la condición Regex. El primer par de paréntesis almacena su subcadena en 1, el segundo par en 2 y así sucesivamente. Puede usar tantos paréntesis como desee. Perl simplemente mantiene la definición de más variables numeradas para que represente estas cadenas capturadas. Algunos ejemplos de [ref](https://docstore.mik.ua/orelly/perl/prog3/ch05_07.htm): 

*  /(\d)(\d)/ # Coincide con dos dígitos y los captura en los grupos 1 y 2 

* /(\d+)/ # Coincide con uno o varios dígitos, y los captura todos en el grupo 1 

* /(\d)+/ # Coincide con un dígito una o varias veces, y captura el último en el grupo 1

Una vez capturado, puede hacer referencia a ellos en el conjunto de acciones con el siguiente formato:

* Para una captura de encabezado de solicitud, debe usar {http_req_headerName_groupNumber}. Por ejemplo, {http_req_User-Agent_1} o {http_req_User-Agent_2}
* Para una captura de encabezado de respuesta, debe usar {http_resp_headerName_groupNumber}. Por ejemplo, {http_resp_Location_1} o {http_resp_Location_2}
* En el caso de una variable de servidor, debe usar {var_serverVariableName_groupNumber}. Por ejemplo, {var_uri_path_1} o {var_uri_path_2}

Si desea utilizar el valor entero, no debe mencionar el número. Simplemente, use el formato {http_req_headerName}, etc. sin groupNumber.

## <a name="server-variables"></a>Variables de servidor

App Gateway usa variables de servidor para almacenar información útil sobre el servidor, la conexión con el cliente y la solicitud actual en la conexión. La dirección IP del cliente y el tipo de explorador web son ejemplos de la información almacenada. Las variables de servidor cambian dinámicamente, por ejemplo, cuando se carga una página nueva o cuando se publica un formulario. Puede usar estas variables para evaluar las condiciones de reescritura y volver a escribir los encabezados. Para usar el valor de las variables del servidor a fin de volver a escribir los encabezados, deberá especificar estas variables con la sintaxis {var_*nombreDeVariableDeServidor*}.

Application Gateway admite las siguientes variables de servidor:

|   Nombre de la variable    |                   Descripción                                           |
| ------------------------- | ------------------------------------------------------------ |
| add_x_forwarded_for_proxy | El campo de encabezado de solicitud de cliente X-Forwarded-For con la variable `client_ip` (consulte la explicación más adelante en esta tabla) anexada a él en el formato IP1, IP2, IP3 y así sucesivamente. Si el campo X-Forwarded-For no se encuentra en el encabezado de solicitud de cliente, la variable `add_x_forwarded_for_proxy` es igual que la variable `$client_ip`.   Esta variable es especialmente útil cuando se desea reescribir el encabezado X-Forwarded-For establecido por Application Gateway, de forma que contenga solo la dirección IP del cliente y no la información del puerto. |
| ciphers_supported         | Una lista de cifrados admitidos por el cliente.               |
| ciphers_used              | La cadena de cifrados que se usa para una conexión TLS establecida. |
| client_ip                 | La dirección IP del cliente desde la que la puerta de enlace de la aplicación recibió la solicitud. Si hay un proxy inverso antes de la puerta de enlace de aplicaciones y el cliente de origen, `client_ip` devolverá la dirección IP del proxy inverso. |
| client_port               | Puerto del cliente.                                             |
| client_tcp_rtt            | Información sobre la conexión TCP de cliente. Está disponible en los sistemas que admiten la opción de socket TCP_INFO. |
| client_user               | Al usar la autenticación HTTP, el nombre de usuario proporcionado para la autenticación. |
| host                      | En este orden de prioridad: nombre de host de la línea de la solicitud, nombre de host del campo de encabezado de la solicitud del “Host” o bien el nombre del servidor que coincida con una solicitud. Ejemplo: en la solicitud `http://contoso.com:8080/article.aspx?id=123&title=fabrikam`, el valor del host será `contoso.com`. |
| cookie_*name*             | El cookie de *nombre*.                                           |
| http_method               | Método usado para realizar la solicitud de URL. Por ejemplo, GET o POST. |
| http_status               | Estado de la sesión. Por ejemplo, 200, 400 o 403.           |
| http_version              | Protocolo de solicitud. Normalmente, HTTP/1.0, HTTP/1.1 o HTTP/2.0. |
| query_string              | La lista de pares de variable-valor que aparecen después de “?” en la dirección URL solicitada. Ejemplo: en la solicitud `http://contoso.com:8080/article.aspx?id=123&title=fabrikam`, el valor de query_string será `id=123&title=fabrikam`. |
| received_bytes            | La longitud de la solicitud (incluida la línea de la solicitud, el encabezado y el cuerpo de la solicitud). |
| request_query             | Los argumentos en la línea de la solicitud.                           |
| request_scheme            | El esquema de la solicitud: HTTP o HTTPS.                           |
| request_uri               | El URI original completo de la solicitud (con argumentos). Ejemplo: en la solicitud `http://contoso.com:8080/article.aspx?id=123&title=fabrikam*`, el valor request_uri será `/article.aspx?id=123&title=fabrikam` |
| sent_bytes                | El número de bytes enviados a un cliente.                        |
| server_port               | El puerto del servidor que ha aceptado una solicitud.              |
| ssl_connection_protocol   | El protocolo de una conexión TLS establecida.               |
| ssl_enabled               | "On" si la conexión funciona en modo TLS. De lo contrario, una cadena vacía. |
| uri_path                  | Identifica el recurso específico en el host al que el cliente web quiere acceder. Esta es la parte del URI de solicitud sin los argumentos. Ejemplo: en la solicitud `http://contoso.com:8080/article.aspx?id=123&title=fabrikam`, el valor de uri_path será `/article.aspx`. |

 

## <a name="rewrite-configuration"></a>Configuración de la reescritura

Para configurar una regla de reescritura, debe crear un conjunto de reglas de reescritura y agregarle su configuración.

Un conjunto de reglas de reescritura contiene:

* **Asociación de la regla de enrutamiento de solicitudes:** la configuración de reescritura se asocia al agente de escucha de origen mediante la regla de enrutamiento. Cuando usa una regla de enrutamiento básica, la configuración de reescritura se asocia a un agente de escucha de origen y es una reescritura de encabezados global. Cuando usa una regla de enrutamiento basada en rutas, la configuración de reescritura de encabezados se define en el mapa de rutas de direcciones URL. En este caso, solo se aplica al área específica de la ruta de acceso de un sitio. Puede crear varios conjuntos de reescrituras y aplicar cada conjunto de reescrituras a varios agentes de escucha. Pero solo puede aplicar un conjunto de reescritura a un agente de escucha específico.

* **Condición de reescritura**: se trata de una configuración opcional. Las condiciones de reescritura evalúan el contenido de las solicitudes y respuestas HTTP(S). La acción de reescritura tendrá lugar si la solicitud o respuesta HTTP(S) coinciden con la condición de reescritura. Si asocia más de una condición con una acción, la acción se produce solo cuando se cumplen todas las condiciones. En otras palabras, se trata de una operación AND lógica.

* **Tipo de reescritura**:  hay tres tipos de reescrituras disponibles:
   * Reescritura de encabezados de solicitud 
   * Reescritura de encabezados de respuesta
   * Dirección URL de reescritura: el tipo de reescritura de URL tiene tres componentes
      * **Ruta de acceso URL**: valor en el que se va a volver a escribir la ruta de acceso. 
      * **Cadena de consulta de URL**: valor en el que se va a volver a escribir la cadena de consulta. 
      * **Volver a evaluar el mapa de ruta de acceso**: se usa para determinar si el mapa de rutas de acceso de dirección URL se va a volver a evaluar o no. Si se deja desactivado, se usará la ruta de dirección URL original para hacer coincidir el patrón de ruta de acceso en el mapa de ruta de acceso de direcciones URL. Si se establece en true, el mapa de ruta de acceso de direcciones URL se volverá a evaluar para comprobar si coincide con la ruta de acceso reescrita. Si se habilita este modificador, ayuda a enrutar la solicitud a otro grupo de back-end posterior a la reescritura.

### <a name="common-scenarios-for-header-rewrite"></a>Escenarios comunes de la reescritura de encabezados

#### <a name="remove-port-information-from-the-x-forwarded-for-header"></a>Eliminación de la información del puerto desde el encabezado X-Forwarded-For

Application Gateway inserta un encabezado X-Forwarded-For en todas las solicitudes antes de reenviar las solicitudes al back-end. Este encabezado es una lista de puertos IP separados por comas. Puede haber escenarios en que los servidores back-end necesiten solo los encabezados que van a contener las direcciones IP. Puede usar la reescritura de encabezados para quitar la información del puerto desde el encabezado X-Forwarded-For. Una manera de hacerlo es establecer el encabezado en la variable de servidor add_x_forwarded_for_proxy: Como alternativa, también puede usar la variable client_ip:

![Eliminación del puerto](./media/rewrite-http-headers-url/remove-port.png)


#### <a name="modify-a-redirection-url"></a>Modificación de una dirección URL de redirección

Cuando una aplicación back-end envía una respuesta de redireccionamiento, es posible que le interese redirigir el cliente a una dirección URL diferente de la especificada por la aplicación back-end. Por ejemplo, es posible que quiera hacerlo cuando un servicio de aplicaciones se hospeda detrás de una puerta de enlace de aplicaciones y requiere que el cliente realice un redireccionamiento a su ruta de acceso relativa. (Por ejemplo, un redireccionamiento de contoso.azurewebsites.net/path1 a contoso.azurewebsites.net/path2).

Dado que App Service es un servicio multiinquilino, usa el encabezado de host en la solicitud para enrutar la solicitud al punto de conexión correcto. Los servicios de aplicaciones tienen un nombre de dominio predeterminado, *. azurewebsites.net, (por ejemplo, contoso.azurewebsites.net) que es diferente del nombre de dominio de la puerta de enlace de aplicaciones (por ejemplo, contoso.com). Dado que la solicitud original desde el cliente tiene el nombre de dominio (contoso.com) de la puerta de enlace de aplicaciones como el nombre de host, la puerta de enlace de aplicaciones cambia el nombre de host a contoso.azurewebsites.net. Realiza este cambio para que el servicio de aplicaciones pueda enrutar la solicitud al punto de conexión correcto.

Cuando el servicio de aplicaciones envía una respuesta de redireccionamiento, usa el mismo nombre de host en el encabezado de ubicación de su respuesta que el que aparece en la solicitud que recibe de la puerta de enlace de aplicaciones. Así pues, el cliente hará la solicitud directamente a contoso.azurewebsites.net/path2, en lugar de pasar por la puerta de enlace de aplicaciones (contoso.com/path2). No es conveniente omitir la puerta de enlace de aplicaciones.

Para resolver este problema, puede establecer el nombre de host en el encabezado de ubicación en el nombre de dominio de la puerta de enlace de aplicaciones.

Estos son los pasos para reemplazar el nombre de host:

1. Cree una regla de reescritura con una condición que evalúe si el encabezado de ubicación en la respuesta contiene azurewebsites.net. Escriba el patrón `(https?):\/\/.*azurewebsites\.net(.*)$`.
2. Realice una acción para volver a escribir el encabezado de ubicación de modo que tenga el nombre de host de la puerta de enlace de aplicaciones. Para ello, escriba `{http_resp_Location_1}://contoso.com{http_resp_Location_2}` como valor del encabezado. O bien, puede usar la variable de servidor `host` para establecer el nombre de host de modo que coincida con la solicitud original.

![Modificación del encabezado de ubicación](./media/rewrite-http-headers-url/app-service-redirection.png)

#### <a name="implement-security-http-headers-to-prevent-vulnerabilities"></a>Implementación de los encabezados HTTP de seguridad para evitar las vulnerabilidades

Puede corregir varias vulnerabilidades de seguridad si implementa los encabezados necesarios en la respuesta de la aplicación. Estos encabezados de seguridad incluyen X-XSS-Protection Strict-Transport-Security y Content-Security-Policy. Puede usar Application Gateway para establecer estos encabezados para todas las respuestas.

![Encabezado de seguridad](./media/rewrite-http-headers-url/security-header.png)

### <a name="delete-unwanted-headers"></a>Eliminación de los encabezados no deseados

Es posible que desee quitar los encabezados que revelen información confidencial de una respuesta HTTP. Por ejemplo, es posible que desee quitar información como el nombre del servidor back-end, el sistema operativo o los detalles de la biblioteca. Puede usar la puerta de enlace de aplicaciones para quitar estos encabezados:

![Eliminación del encabezado](./media/rewrite-http-headers-url/remove-headers.png)

#### <a name="check-for-the-presence-of-a-header"></a>Comprobación de la presencia de un encabezado

Puede evaluar un encabezado de respuesta o de solicitud HTTP para comprobar la presencia de una variable de servidor o encabezado. Esta evaluación es útil cuando desea realizar una reescritura de encabezado solo si está presente un encabezado determinado.

![Comprobación de la presencia de un encabezado](./media/rewrite-http-headers-url/check-presence.png)

### <a name="common-scenarios-for-url-rewrite"></a>Escenarios comunes de la reescritura de direcciones URL

#### <a name="parameter-based-path-selection"></a>Selección de ruta basada en parámetros

En los escenarios en los que desee elegir el grupo de back-end en función del valor de un encabezado, parte de la dirección URL o la cadena de consulta de la solicitud, puede usar la combinación de la funcionalidad de reescritura de direcciones URL y el enrutamiento basado en rutas de acceso.  Por ejemplo, si tiene un sitio web de compras y la categoría de producto se pasa como una cadena de consulta en la dirección URL, y desea enrutar la solicitud al back-end en función de la cadena de consulta, entonces:

**Paso 1:**  cree un mapa de ruta de acceso tal como se muestra en la imagen siguiente.

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario1-1.png" alt-text="Escenario de reescritura de URL 1-1.":::

**Paso 2 (a):** cree un conjunto de reescritura que tenga tres reglas de reescritura: 

* La primera regla tiene una condición que comprueba la variable *query_string* para *category=shoes* y tiene una acción que reescribe la ruta de acceso de dirección URL en /*listing1* y tiene habilitada la opción **Volver a evaluar el mapa de ruta de acceso**

* La segunda regla tiene una condición que comprueba la variable *query_string* para *category=bags* y tiene una acción que vuelve a escribir la ruta de acceso de la dirección URL en/*listing2* y tiene habilitada la opción **Volver a evaluar el mapa de ruta de acceso**

* La tercera regla tiene una condición que comprueba la variable *query_string* para *category= accessories* y tiene una acción que vuelve a escribir la ruta de acceso de la dirección URL en /*listing3* y tiene habilitada la opción **Volver a evaluar el mapa de ruta de acceso**

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario1-2.png" alt-text="Escenario de reescritura de URL 1-2.":::

 

**Paso 2 (b):** asocie este conjunto de reescritura con la ruta de acceso predeterminada de la regla basada en la ruta de acceso anterior

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario1-3.png" alt-text="Escenario de reescritura de URL 1-3.":::

Ahora, si el usuario solicita *contoso.com/listing?category=any*, se hará coincidir con la ruta de acceso predeterminada, ya que ninguno de los patrones de ruta de acceso del mapa de ruta de acceso (/listing1,/listing2,/listing3) coincidirá. Como asoció el conjunto de reescritura anterior con esta ruta de acceso, este conjunto de reescritura se evaluará. Dado que la cadena de consulta no coincidirá con la condición en ninguna de las tres reglas de reescritura de este conjunto de reescritura, no se llevará a cabo ninguna acción de reescritura y, por tanto, la solicitud se enrutará sin cambios al back-end asociado a la ruta de acceso predeterminada (que es *GenericList*).

 Si el usuario solicita *contoso.com/listing?category=shoes,* de nuevo se buscarán coincidencias en la ruta de acceso predeterminada. Sin embargo, en este caso, la condición de la primera regla coincidirá y, por lo tanto, se ejecutará la acción asociada a la condición, que volverá a escribir la ruta de acceso de la dirección URL para /*listing1* y volverá a evaluar el mapa de ruta de acceso. Cuando se vuelva a evaluar el mapa de ruta de acceso, la solicitud coincidirá ahora con la ruta de acceso asociada al patrón */listing1* y la solicitud se enrutará al back-end asociado a este patrón, que es ShoesListBackendPool

>[!NOTE]
>Este escenario se puede extender a cualquier valor de encabezado o cookie, ruta de dirección URL, cadena de consulta o variables de servidor según la condición definida y, esencialmente, permite enrutar las solicitudes en función de esas condiciones.

#### <a name="rewrite-query-string-parameters-based-on-the-url"></a>Reescritura de los parámetros de cadena de consulta en función de la dirección URL

Considere un escenario de un sitio web de compras en el que el vínculo visible del usuario deba ser sencillo y legible, pero el servidor back-end necesite los parámetros de la cadena de consulta para mostrar el contenido correcto.

En ese caso, Application Gateway puede capturar los parámetros de la dirección URL y agregar pares de clave-valor de cadena de consulta a partir de los de la dirección URL. Por ejemplo, supongamos que el usuario desea reescribir `https://www.contoso.com/fashion/shirts` a `https://www.contoso.com/buy.aspx?category=fashion&product=shirts`; para ello, puede usar la siguiente configuración de reescritura de direcciones URL.

**Condición**: si la variable de servidor `uri_path` es igual al patrón `/(.+)/(.+)`

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario2-1.png" alt-text="Escenario de reescritura de URL 2-1.":::

**Acción**: establezca la ruta de acceso de la dirección URL en `buy.aspx` y la cadena de consulta en `category={var_uri_path_1}&product={var_uri_path_2}`

:::image type="content" source="./media/rewrite-http-headers-url/url-scenario2-2.png" alt-text="Escenario de reescritura de URL 2-2.":::

Para obtener una guía paso a paso para lograr el escenario descrito anteriormente, consulte [Reescritura de dirección URL con Application Gateway mediante Azure Portal](rewrite-url-portal.md)

### <a name="url-rewrite-vs-url-redirect"></a>Reescritura de direcciones URL frente a redirección de direcciones URL

En el caso de la reescritura de direcciones URL, Application Gateway reescribe la dirección URL antes de que la solicitud se envíe al back-end. Eso no cambiará lo que los usuarios ven en el explorador, porque los cambios se ocultan al usuario.

En el caso de la redirección de direcciones URL, Application Gateway envía una respuesta de redirección al cliente con la nueva dirección URL. Esto, a su vez, requiere que el cliente reenvíe su solicitud a la nueva dirección URL proporcionada en la redirección. La dirección URL que el usuario ve en el explorador se actualizará a la nueva dirección URL.

:::image type="content" source="./media/rewrite-http-headers-url/url-rewrite-vs-redirect.png" alt-text="Reescritura frente a redirección.":::

## <a name="limitations"></a>Limitaciones

- Si una respuesta tiene más de un encabezado con el mismo nombre, volver a escribir el valor de uno de esos encabezados dará como resultado la eliminación de los demás en la respuesta. Normalmente, esto puede suceder con el encabezado Set-Cookie, ya que puede tener más de uno en una respuesta. Uno de estos escenarios es cuando se usa un servicio de aplicaciones con una puerta de enlace de aplicaciones y ha configurado la afinidad de sesión basada en cookies en la puerta de enlace de aplicaciones. En este caso, la respuesta contendrá dos encabezados Set-Cookie: uno utilizado por el servicio de aplicaciones, por ejemplo, `Set-Cookie: ARRAffinity=ba127f1caf6ac822b2347cc18bba0364d699ca1ad44d20e0ec01ea80cda2a735;Path=/;HttpOnly;Domain=sitename.azurewebsites.net`, y otro para la afinidad de la puerta de enlace de aplicaciones, es decir, `Set-Cookie: ApplicationGatewayAffinity=c1a2bd51lfd396387f96bl9cc3d2c516; Path=/`. Si se vuelve a escribir uno de los encabezados Set-Cookie en este escenario, podría quitar el otro encabezado Set-Cookie de la respuesta.
- No se admiten las reescrituras cuando la puerta de enlace de aplicaciones está configurada para redirigir las solicitudes o para mostrar una página de error personalizada.
- Los nombres de encabezado solo pueden contener caracteres alfanuméricos y símbolos específicos definidos en [RFC 7230](https://tools.ietf.org/html/rfc7230#page-27). Actualmente no se admite el carácter especial de subrayado (_) en los nombres de encabezado.
- No se pueden volver a escribir los encabezados de conexión y de actualización

## <a name="next-steps"></a>Pasos siguientes

- [Obtenga información sobre cómo reescribir encabezados HTTP con Application Gateway mediante Azure Portal](rewrite-http-headers-portal.md)
- [Obtenga información sobre cómo reescribir la dirección URL con Application Gateway mediante Azure Portal](rewrite-url-portal.md)
