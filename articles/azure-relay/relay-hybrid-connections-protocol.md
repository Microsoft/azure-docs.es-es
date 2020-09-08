---
title: Guía del protocolo de conexiones híbridas de Retransmisión de Azure | Microsoft Docs
description: En este artículo se describen las interacciones en el lado cliente con el servicio de retransmisión de Conexiones híbridas para la conexión de clientes en los roles de agente de escucha y remitente.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: fec021d961a17102f8d979c61ee46af6b938f073
ms.sourcegitcommit: 2bab7c1cd1792ec389a488c6190e4d90f8ca503b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/17/2020
ms.locfileid: "88272016"
---
# <a name="azure-relay-hybrid-connections-protocol"></a>Protocolo de conexiones híbridas de Azure Relay

Relay de Azure es uno de los pilares básicos de las funcionalidades de la plataforma Azure Service Bus. La nueva funcionalidad _Conexiones híbridas_ de Relay es una evolución segura y de protocolo abierto basada en HTTP y WebSockets. Sustituye a la antigua característica de _BizTalk Services_ con el mismo nombre que se basaba en un protocolo propietario. La integración de Conexiones híbridas en Azure App Service seguirá funcionando de la misma forma.

El servicio Conexiones híbridas permite la comunicación de flujo binario bidireccional y el flujo de datagrama sencillo entre dos aplicaciones en red. Una o ambas partes pueden residir detrás de los firewalls o de los dispositivos NAT.

En este artículo se describen las interacciones en el lado cliente con el servicio de retransmisión de Conexiones híbridas para la conexión de clientes en los roles de agente de escucha y remitente. También se describe cómo los agentes de escucha aceptan nuevas conexiones y solicitudes.

## <a name="interaction-model"></a>Modelo de interacción

El servicio de retransmisión de Conexiones híbridas conecta dos partes mediante la creación de un punto de encuentro en la nube de Azure que ambas partes pueden detectar y al que pueden conectarse desde su propia red. Ese punto de encuentro se denomina "Conexión híbrida" en esta y otra documentación, en las API y también en Azure Portal. Al punto de conexión del servicio Conexiones híbridas nos referiremos como el "servicio" en el resto de este artículo.

El servicio permite la retransmisión de conexiones WebSocket y solicitudes y respuestas HTTP(S).

El modelo de interacción se basa en la nomenclatura que han establecido muchas otras API de red. Hay un agente de escucha que, primero, indica que está preparado para controlar las conexiones entrantes y, después, las acepta a medida que llegan. Por otro lado, un cliente se conecta con el agente de escucha, y espera que se acepte dicha conexión para establecer una ruta de comunicación bidireccional. "Conectar", "escuchar" y "aceptar" son los mismos términos que encontrará en la mayoría de las API de socket.

En cualquier modelo de comunicación retransmitida, alguna de las partes realiza conexiones salientes hacia un punto de conexión de servicio. Esto convierte también al "agente de escucha" en un "cliente" en uso coloquial, y puede provocar además otras sobrecargas de terminología. Por consiguiente, esta es la terminología precisa que se usa en Conexiones híbridas:

Los programas de ambos extremos de la conexión se denominan "clientes", ya que son clientes del servicio. El cliente que espera y acepta conexiones es el "agente de escucha" o que asume el "rol de escucha". El cliente que inicia una nueva conexión hacia un agente de escucha a través del servicio se denominará "remitente" o que asume el "rol de remitente".

### <a name="listener-interactions"></a>Interacciones del agente de escucha

El agente de escucha tiene cuatro interacciones con el servicio. Más adelante, en la sección de referencia de este artículo explicaremos todos los detalles de conexión.

Los mensajes de escucha, aceptación y solicitud se reciben del servicio. El agente de escucha envía las operaciones de revisión y ping.

#### <a name="listen-message"></a>Mensaje de escucha

Para indicar al servicio que un agente de escucha está listo para aceptar conexiones, crea una conexión WebSocket de salida. El protocolo de enlace de la conexión incluye el nombre de una conexión híbrida configurada en el espacio de nombres de Relay y un token de seguridad que concede el derecho de "escucha" a dicho nombre.

Cuando el servicio acepta WebSocket, se completa el registro y el WebSocket establecido se mantiene activo como "canal de control" para habilitar todas las interacciones posteriores. El servicio admite hasta 25 agentes de escucha simultáneos en una conexión híbrida. La cuota de AppHooks está por determinar.

En Conexiones híbridas, si hay dos o más agentes de escucha activos, las conexiones entrantes se reparten entre ellos en orden aleatorio, por lo que no se garantiza que la distribución sea equitativa.

#### <a name="accept-message"></a>Mensaje de aceptación

Cuando un remitente abre una nueva conexión en el servicio, este elige a uno de los agentes de escucha activos de la conexión híbrida y le envía una notificación. Esta notificación se envía al agente de escucha a través del canal de control abierto como un mensaje JSON. El mensaje contiene la dirección URL del punto de conexión de WebSocket al que debe conectarse el agente de escucha para aceptar la conexión.

La dirección URL puede, y debe, usarla directamente el agente de escucha sin tener que realizar ningún proceso adicional.
La información codificada solo es válida durante un breve período, básicamente, el tiempo que el remitente esté dispuesto a esperar a que se establezca la conexión de un extremo a otro. Se supone un límite máximo de 30 segundos. La dirección URL solo puede utilizarse para realizar un intento de conexión correcta. En cuanto se establece la conexión WebSocket con la dirección URL de encuentro, toda la actividad posterior en ese WebSocket se retransmite desde y hacia el remitente. Esto sucede sin intervención ni interpretación por parte del servicio.

### <a name="request-message"></a>Mensaje de solicitud

Además de las conexiones WebSocket, el agente de escucha puede recibir también marcos de solicitud HTTP de un remitente, si esta funcionalidad está habilitada explícitamente en la conexión híbrida.

Los agentes de escucha que se conectan a Conexiones híbridas con compatibilidad para HTTP deben controlar el gesto `request`. Un agente de escucha que no controla `request` y, por lo tanto, provoca errores repetidos de tiempo de espera mientras está conectado, PODRÍA incluirse en la lista de no permitidos del servicio en el futuro.

Los metadatos de encabezado de marco HTTP se convierten en JSON para que se puedan administrar más fácilmente en el marco del agente de escucha; también porque las bibliotecas de análisis de encabezados HTTP son más inusuales que los analizadores JSON. Los metadatos HTTP que solo son de interés para la relación entre el remitente y la puerta de enlace HTTP de retransmisión, incluida la información de autorización, no se reenvían. Los cuerpos de solicitud HTTP se transfieren de manera transparente como marcos binarios de WebSocket.

El agente de escucha puede responder a solicitudes HTTP mediante un gesto de respuesta equivalente.

El flujo de solicitud/respuesta usa el canal de control de forma predeterminada, pero se puede "actualizar" a un WebSocket de encuentro distinto siempre que sea necesario. Las distintas conexiones WebSocket mejoran el rendimiento de cada conversación de cliente, pero sobrecargan al agente de escucha con más conexiones que es necesario administrar, lo que no sería deseable para clientes ligeros.

En el canal de control, los cuerpos de solicitud y respuesta se limitan a 64 kB de tamaño como máximo. Los metadatos de encabezado HTTP se limitan a un total de 32 kB. Si la solicitud o la respuesta superan ese umbral, el agente de escucha se DEBE actualizar a un WebSocket de encuentro mediante un gesto equivalente a administrar la [aceptación](#accept-message).

Con las solicitudes, el servicio decide si se deben enrutar a través del canal de control. Esto incluye, entre otros, los casos en los que una solicitud supera los 64 kB (encabezados y cuerpo) directamente, o si la solicitud se envía con [ codificación de transferencia "fragmentada"](https://tools.ietf.org/html/rfc7230#section-4.1) y el servicio tiene alguna razón para esperar que la solicitud supere los 64 kB o que la lectura de la solicitud no sea instantánea. Si el servicio elige entregar la solicitud a través del punto de encuentro, solo se pasa la dirección de encuentro al agente de escucha.
El agente de escucha DEBE establecer el WebSocket de encuentro y el servicio entrega inmediatamente la solicitud completa, incluidos los cuerpos, a través de este. La respuesta también debe usar el WebSocket de encuentro.

En el caso de las solicitudes que llegan a través del canal de control, el agente de escucha decide si responder a través de dicho canal o del punto de encuentro. El servicio DEBE incluir una dirección de encuentro donde cada solicitud se enrute a través del canal de control. Esta dirección solo es válida para actualizaciones desde la solicitud actual.

Si el agente de escucha elige actualizar, se conecta y entrega inmediatamente la respuesta a través del socket de encuentro establecido.

Una vez que el WebSocket de encuentro se ha establecido, el agente de escucha debe MANTENERLO para el posterior tratamiento de las solicitudes y respuestas desde el mismo cliente. El servicio mantendrá el WebSocket mientras persista la conexión del socket HTTPS con el remitente, y enrutará todas las solicitudes sucesivas desde ese remitente a través del WebSocket mantenido. Si el agente de escucha decide descartar el WebSocket de encuentro por su parte, el servicio también descartará la conexión al remitente, con independencia de que pueda haber en curso una solicitud posterior.

#### <a name="renew-operation"></a>Operación de renovación

El token de seguridad que se debe utilizar para registrar el agente de escucha y mantener el canal de control puede expirar mientras que el agente de escucha está activo. La expiración del token no afecta a las conexiones en curso, pero hace que el servicio anule el canal de control en cuanto expire, o poco después. La operación de "renovación" es un mensaje JSON que puede enviar el agente de escucha para reemplazar el token asociado al canal de control. De este modo, el canal de control se puede mantener durante períodos prolongados.

#### <a name="ping-operation"></a>Operación de ping

Si el canal de control permanece inactivo durante mucho tiempo, los intermediarios del proceso, como los equilibradores de carga o los NAT, pueden anular la conexión TCP. La operación "ping" evita que esto suceda mediante el envío de una pequeña cantidad de datos en el canal que recuerda a todos los usuarios de la ruta de red que la conexión está pensada para que se mantenga activa. Además, sirve al agente de escucha como prueba de que está "activo". Si el ping falla, el canal de control debe considerarse inutilizable y debe volver a conectar el agente de escucha.

### <a name="sender-interaction"></a>Interacción del remitente

El remitente tiene dos interacciones con el servicio: conecta un socket web o envía solicitudes a través de HTTPS. Las solicitudes no se pueden enviar a través de un socket web con un rol de remitente.

#### <a name="connect-operation"></a>Operación de conexión

La operación de "conexión" abre un WebSocket en el servicio que proporciona el nombre de la conexión híbrida y un token de seguridad (opcional, pero obligatorio de manera predeterminada) que concede el permiso de "envío" a la cadena de consulta. Después, el servicio interactúa con el agente de escucha como se ha descrito anteriormente y este crea una conexión de encuentro que se combina con este WebSocket. Cuando el WebSocket se haya aceptado, las restantes interacciones de dicho WebSocket se realizarán con un agente de escucha conectado.

#### <a name="request-operation"></a>Operación de solicitud

En conexiones híbridas en las que la característica se ha habilitado, el remitente puede enviar solicitudes HTTP prácticamente ilimitadas a los agentes de escucha.

Excepto en el caso del token de acceso de retransmisión que está insertado en la cadena de consulta o en un encabezado HTTP de la solicitud, la retransmisión es totalmente transparente para todas las operaciones HTTP en la dirección de retransmisión y todos los sufijos de la ruta de la dirección de retransmisión, de forma que deja al agente de escucha con el control completo de la autorización de un extremo a otro e incluso de características de extensión HTTP como [CORS](https://www.w3.org/TR/cors/).

La autorización del remitente con el punto de conexión de retransmisión está activada de forma predeterminada, pero es OPCIONAL. El propietario de la conexión híbrida puede elegir permitir remitentes anónimos. El servicio interceptará, inspeccionará y quitará la información de autorización de la manera siguiente:

1. Si la cadena de consulta contiene una expresión `sb-hc-token`, la expresión SIEMPRE se quitará de la cadena de consulta. Se evaluará si la autorización de retransmisión está activada.
2. Si los encabezados de solicitud contienen un encabezado `ServiceBusAuthorization`, la expresión del encabezado SIEMPRE se quitará de la colección de encabezados.
   Se evaluará si la autorización de retransmisión está activada.
3. Solo si la autorización de retransmisión está activada, y si los encabezados de solicitud contienen un encabezado `Authorization`, y ninguna de las expresiones anteriores existe, el encabezado se evaluará y quitará. En caso contrario, el encabezado `Authorization` siempre se pasa como está.

Si no hay ningún agente de escucha activo, el servicio devolverá un código de error 502 "Puerta de enlace incorrecta". Si aparentemente el servicio no administra la solicitud, devolverá un error 504 "Tiempo de espera agotado para la puerta de enlace" al cabo de 60 segundos.

### <a name="interaction-summary"></a>Resumen de interacción

El resultado de este modelo de interacción es que el cliente remitente sale del protocolo de enlace con un WebSocket "limpio", que está conectado a un agente de escucha y que no necesita más preámbulos ni preparativos. Este modelo permite que casi todas las implementaciones existentes del cliente de WebSocket saquen provecho con facilidad del servicio Conexiones híbridas al especificar una dirección URL con una estructura correcta en el nivel de cliente de su WebSocket.

El WebSocket de la conexión de encuentro que el agente de escucha obtiene a través de la interacción de aceptación también está "limpio". Además, se puede entregar a cualquier implementación de servidor de WebSocket existente con una abstracción adicional mínima que distinga entre las operaciones de "aceptación" de los agentes de escucha de red local de su marco y las de "aceptación" remotas de conexiones híbridas.

El modelo de solicitud/respuesta HTTP proporciona al remitente un área de superficie de protocolo HTTP prácticamente ilimitada con un nivel de autorización OPCIONAL. El agente de escucha obtiene una sección de encabezado de solicitud HTTP analizada previamente que puede convertirse en una solicitud HTTP descendente o administrarse tal cual, donde los marcos binarios transportan los cuerpos HTTP. Las respuestas usan el mismo formato. Se pueden administrar interacciones con menos de 64 KB de cuerpos de solicitud y respuesta a través de un solo socket web que se comparte entre todos los remitentes. Las solicitudes y respuestas más grandes se pueden administrar con el modelo de encuentro.

## <a name="protocol-reference"></a>Referencia de protocolos

En esta sección se describen los detalles de las interacciones de protocolo que se han descrito anteriormente.

Todas las conexiones WebSocket se realizan en el puerto 443 como una actualización de la versión 1.1 de HTTPS, que normalmente la abstrae alguna API o marco de WebSocket. La descripción que ofrecemos en este documento sirve para cualquier tipo de implementación; no sugerimos ningún marco concreto.

### <a name="listener-protocol"></a>Protocolo del agente de escucha

El protocolo del agente de escucha está formado por dos movimientos de conexión y tres operaciones de mensaje.

#### <a name="listener-control-channel-connection"></a>Conexión de canal de control del agente de escucha

El canal de control se abre al crearse una conexión de WebSocket en:

`wss://{namespace-address}/$hc/{path}?sb-hc-action=...[&sb-hc-id=...]&sb-hc-token=...`

`namespace-address` es el nombre de dominio completo del espacio de nombres de Azure Relay que hospeda la conexión híbrida, normalmente del formulario `{myname}.servicebus.windows.net`.

Las opciones de los parámetros de cadena de consulta son las siguientes:

| Parámetro        | Obligatorio | Descripción
| ---------------- | -------- | -------------------------------------------
| `sb-hc-action`   | Sí      | Con el rol de agente de escucha, el parámetro debe ser **sb-hc-action=listen**.
| `{path}`         | Sí      | La ruta de acceso del espacio de nombres codificada con la dirección URL de la conexión híbrida preconfigurada en la que se registrará este agente de escucha. Esta expresión se anexa a la parte de la ruta de acceso `$hc/` fija.
| `sb-hc-token`    | Sí\*    | El agente de escucha debe proporcionar un token de acceso compartido de Service Bus válido y codificado para URL al espacio de nombres o a la conexión híbrida que concede el derecho de **escucha**.
| `sb-hc-id`       | No       | Este id. opcional especificado por el cliente permite realizar un seguimiento de diagnósticos completo.

Si se produce un error en la conexión de WebSocket porque la ruta de acceso de la conexión híbrida no está registrada, porque que falta un token, o no es válido, o por cualquier otro error, se utilizará el modelo de comentarios de estado normal de HTTP 1.1. La descripción del estado contiene un identificador de seguimiento de errores que se pueda comunicar al equipo de soporte técnico de Azure:

| Código | Error          | Descripción
| ---- | -------------- | -------------------------------------------------------------------
| 404  | No encontrado      | La ruta de acceso de la conexión híbrida no es válida o el formato de la URL base no es correcto.
| 401  | No autorizado   | Falta el token de seguridad, tiene una estructura incorrecta o no es válido.
| 403  | Prohibido      | El token de seguridad no es válido para esta ruta de acceso en esta acción.
| 500  | Error interno | Se ha producido un error en el servicio.

Si el servicio cierra intencionadamente la conexión de WebSocket después de haberse configurado inicialmente, el motivo se comunica mediante un código de error adecuado de protocolo WebSocket, junto con un mensaje de error descriptivo, que también incluye un identificador de seguimiento. El servicio no cerrará el canal de control si no se produce una condición de error. Todos los cierres "limpios" los controla el cliente.

| Estado de WS | Descripción
| --------- | -------------------------------------------------------------------------------
| 1001      | La ruta de acceso de Conexión híbrida se ha eliminado o deshabilitado.
| 1008      | El token de seguridad ha expirado, por lo que se ha infringido la directiva de autorización.
| 1011      | Se ha producido un error en el servicio.

#### <a name="accept-handshake"></a>Protocolo de enlace de aceptación

La notificación de aceptación la envía el servicio al agente de escucha a través del canal de control establecido previamente en forma de mensaje JSON en un marco de texto de WebSocket. No hay ninguna respuesta a este mensaje.

El mensaje contiene un objeto JSON denominado "accept", que define las siguientes propiedades:

* **address**: la cadena de dirección URL que se usará para establecer que el WebSocket del servicio acepte una conexión entrante.
* **id**: el identificador único para esta conexión. Si ha sido el cliente remitente quien ha suministrado el id., será el valor que ha especificado este; de lo contrario, será uno generado por el sistema.
* **connectHeaders**: todos los encabezados HTTP que ha suministrado el remitente al punto de conexión de Relay, que también incluyen los encabezados Sec-WebSocket-Protocol y Sec-WebSocket-Extensions.

```json
{
    "accept" : {
        "address" : "wss://dc-node.servicebus.windows.net:443/$hc/{path}?..."
        "id" : "4cb542c3-047a-4d40-a19f-bdc66441e736",
        "connectHeaders" : {
            "Host" : "...",
            "Sec-WebSocket-Protocol" : "...",
            "Sec-WebSocket-Extensions" : "..."
        }
     }
}
```

El agente de escucha usa la dirección URL especificada en el mensaje JSON para establecer el WebSocket con el objetivo de aceptar o rechazar el socket remitente.

##### <a name="accepting-the-socket"></a>Aceptación del socket

Para aceptarlo, el agente de escucha establece una conexión de WebSocket a la dirección proporcionada.

Si el mensaje "accept" contiene un encabezado `Sec-WebSocket-Protocol`, cabe esperar que el agente de escucha solo acepte el WebSocket si admite dicho protocolo. Además, establece el encabezado tal como se establece el WebSocket.

Lo mismo sucede con el encabezado `Sec-WebSocket-Extensions`. Si el marco admite una extensión, es necesario establecer el encabezado en la respuesta del lado servidor del protocolo de enlace `Sec-WebSocket-Extensions` que se necesita para la extensión.

La URL debe utilizarse tal cual para establecer el socket de aceptación, pero debe contener los parámetros siguientes:

| Parámetro      | Obligatorio | Descripción
| -------------- | -------- | -------------------------------------------------------------------
| `sb-hc-action` | Sí      | Para aceptar un socket, el parámetro tiene que ser `sb-hc-action=accept`
| `{path}`       | Sí      | (vea el párrafo siguiente)
| `sb-hc-id`     | No       | Consulte la descripción anterior del **identificador**.

`{path}` es la ruta de acceso del espacio de nombres con codificación URL de la conexión híbrida preconfigurada en la que se registra este agente de escucha. Esta expresión se anexa a la parte de la ruta de acceso `$hc/` fija.

La expresión `path` se puede ampliar con un sufijo y una expresión de cadena de consulta que vaya después del nombre registrado después de una barra diagonal de separación.
Esto permite que el cliente remitente pase los argumentos de envío al agente de escucha receptor cuando no sea posible incluir encabezados HTTP. La expectativa es que el marco del agente de escucha analice la parte fija y el nombre registrado de la ruta de acceso, y cree el resto, posiblemente sin ningún argumento de cadena de consulta con el prefijo `sb-`, disponible para la aplicación para decidir si acepta la conexión.

Para más información, consulte la sección "Protocolo del remitente", que encontrará a continuación.

Si hay un error, el servicio puede responder como se indica a continuación:

| Código | Error          | Descripción
| ---- | -------------- | -----------------------------------
| 403  | Prohibido      | La URL no es válida.
| 500  | Error interno | Se produjo un error en el servicio.

 Una vez que se establece la conexión, el servidor apaga el WebSocket cuando se apaga el WebSocket remitente, o cuando el estado es el siguiente:

| Estado de WS | Descripción                                                                     |
| --------- | ------------------------------------------------------------------------------- |
| 1001      | El cliente remitente cierra la conexión.                                    |
| 1001      | La ruta de acceso de Conexión híbrida se ha eliminado o deshabilitado.                        |
| 1008      | El token de seguridad ha expirado, por lo que se ha infringido la directiva de autorización. |
| 1011      | Se ha producido un error en el servicio.                                            |

##### <a name="rejecting-the-socket"></a>Rechazo del socket

 Para rechazar el socket después de inspeccionar el mensaje `accept`, se necesita un protocolo de enlace similar para que el código y la descripción del estado que comunican el motivo del rechazo puedan enviarse al remitente.

 La opción de diseño de protocolo que presentamos en este documento es un protocolo de enlace WebSocket (que está pensado para que termine en un estado de error definido). De esta forma, las implementaciones del cliente del agente de escucha pueden seguir dependiendo de un cliente WebSocket y no hace falta utilizar otro cliente HTTP básico.

 Para rechazar el socket, el cliente toma el URI de dirección del mensaje `accept` y le anexa dos parámetros de cadena de consulta como se muestra a continuación:

| Parámetro                   | Obligatorio | Descripción                              |
| ----------------------- | -------- | ---------------------------------------- |
| sb-hc-statusCode        | Sí      | Código de estado HTTP numérico.                |
| sb-hc-statusDescription | Sí      | Motivo del rechazo en lenguaje natural. |

El URI resultante se utiliza luego para establecer una conexión WebSocket.

Cuando se completa correctamente, este protocolo de enlace genera un error de forma intencionada con el código HTTP 410, ya que no se ha establecido ningún WebSocket. Si algo falla, los siguientes códigos describen el error:

| Código | Error          | Descripción                          |
| ---- | -------------- | ------------------------------------ |
| 403  | Prohibido      | La URL no es válida.                |
| 500  | Error interno | Se ha producido un error en el servicio. |

#### <a name="request-message"></a>Mensaje de solicitud

El servicio envía el mensaje `request` al agente de escucha a través del canal de control. El mismo mensaje también se envía a través del WebSocket de encuentro una vez establecido.

El mensaje `request` consta de dos partes: un encabezado y uno o varios marcos de cuerpo binarios.
Si no hay ningún cuerpo, se omiten los marcos de cuerpo. El indicador que dice si un cuerpo está presente es la propiedad booleana `body` en el mensaje de solicitud.

En una solicitud con un cuerpo de solicitud, la estructura puede ser similar a la siguiente:

``` text
----- Web Socket text frame ----
{
    "request" :
    {
        "body" : true,
        ...
    }
}
----- Web Socket binary frame ----
FEFEFEFEFEFEFEFEFEFEF...
----- Web Socket binary frame ----
FEFEFEFEFEFEFEFEFEFEF...
----- Web Socket binary frame -FIN
FEFEFEFEFEFEFEFEFEFEF...
----------------------------------
```

El agente de escucha DEBE administrar el cuerpo de la solicitud dividido entre varios marcos binarios (consulte sobre los [fragmentos de WebSocket](https://tools.ietf.org/html/rfc6455#section-5.4)).
La solicitud finaliza cuando se ha recibido un marco binario con la marca FIN establecida.

En una solicitud sin cuerpo, solo hay un marco de texto.

``` text
----- Web Socket text frame ----
{
    "request" :
    {
        "body" : false,
        ...
    }
}
----------------------------------
```

El contenido JSON de `request` es este:

* **address**: cadena de URI. Se trata de la dirección de encuentro que se usa con esta solicitud. Si la solicitud entrante es mayor de 64 kB, el resto de este mensaje se deja vacío y el cliente DEBE iniciar un protocolo de enlace de encuentro equivalente a la operación `accept` que se describe a continuación. El servicio coloca entonces el objeto `request` en el socket web establecido. Si cabe la posibilidad de que la respuesta exceda los 64 kB, el agente de escucha DEBE también iniciar un protocolo de enlace de encuentro y transferir luego la respuesta a través del socket web establecido.
* **id**: cadena. Identificador único de esta solicitud.
* **requestHeaders** : este objeto contiene todos los encabezados HTTP que el remitente ha suministrado al punto de conexión, a excepción de la información de autorización, como se explicó [anteriormente](#request-operation), y los encabezados estrictamente relacionados con la conexión a la puerta de enlace. En concreto, todos los encabezados definidos o reservados en [RFC7230](https://tools.ietf.org/html/rfc7230), excepto `Via`, se quitan y no se reenvían:

  * `Connection` (RFC7230, sección 6.1)
  * `Content-Length` (RFC7230, sección 3.3.2)
  * `Host` (RFC7230, sección 5.4)
  * `TE` (RFC7230, sección 4.3)
  * `Trailer` (RFC7230, sección 4.4)
  * `Transfer-Encoding` (RFC7230, sección 3.3.1)
  * `Upgrade` (RFC7230, sección 6.7)
  * `Close` (RFC7230, sección 8.1)

* **requestTarget**: cadena. Esta propiedad contiene el ["Destino de solicitud" (RFC7230, sección 5.3)](https://tools.ietf.org/html/rfc7230#section-5.3) de la solicitud. Esto incluye la parte de la cadena de consulta que se ha quitado de TODOS los parámetros con el prefijo `sb-hc-`.
* **method**: cadena. Este es el método de la solicitud, según [RFC7231, sección 4](https://tools.ietf.org/html/rfc7231#section-4). El método `CONNECT` NO DEBE usarse.
* **body**: booleano. Indica si le siguen uno o varios marcos de cuerpo binarios.

``` JSON
{
    "request" : {
        "address" : "wss://dc-node.servicebus.windows.net:443/$hc/{path}?...",
        "id" : "42c34cb5-7a04-4d40-a19f-bdc66441e736",
        "requestTarget" : "/abc/def?myarg=value&otherarg=...",
        "method" : "GET",
        "requestHeaders" : {
            "Host" : "...",
            "Content-Type" : "...",
            "User-Agent" : "..."
        },
        "body" : true
     }
}
```

##### <a name="responding-to-requests"></a>Respuesta a las solicitudes

El receptor DEBE responder. Los errores repetidos para responder a las solicitudes mientras se mantiene la conexión podrían dar lugar a que el agente de escucha se incluyera en la lista de no permitidos.

Las respuestas se pueden enviar en cualquier orden, pero cada solicitud se debe responder en 60 segundos o la entrega se notificará como si no hubiera pasado. La fecha límite de 60 segundos se cuenta hasta que el servicio recibe el marco `response`. Una respuesta en curso con varios marcos binarios no se puede volver inactiva durante más de 60 segundos, o se termina.

Si la solicitud se recibe a través del canal de control, la respuesta se DEBE enviar en el canal de control desde donde se recibió la solicitud o se DEBE enviar a través de un canal de encuentro.

La respuesta es un objeto JSON llamado "response". Las reglas de administración del contenido del cuerpo son exactamente iguales que las del mensaje `request` y se basan en la propiedad `body`.

* **requestId**. REQUERIDO. El valor de propiedad `id` del mensaje `request` al que se responde.
* **statusCode**: número. REQUERIDO. Un código de estado HTTP numérico que indica el resultado de la notificación. Todos los códigos de estado de [RFC7231, sección 6](https://tools.ietf.org/html/rfc7231#section-6) están permitidos, excepto [502 "Puerta de enlace no válida"](https://tools.ietf.org/html/rfc7231#section-6.6.3) y [504 "Tiempo de espera agotado para la puerta de enlace"](https://tools.ietf.org/html/rfc7231#section-6.6.5).
* **statusDescription**. OPCIONAL. Frase de motivo de código de estado HTTP según [RFC7230, sección 3.1.2](https://tools.ietf.org/html/rfc7230#section-3.1.2)
* **responseHeaders**: encabezados HTTP que se establecerán en una respuesta HTTP externa.
  Al igual que con el objeto `request`, los encabezados definidos de RFC7230 NO SE DEBEN usar.
* **body**: booleano. Indica si le siguen marcos de cuerpo binario.

``` text
----- Web Socket text frame ----
{
    "response" : {
        "requestId" : "42c34cb5-7a04-4d40-a19f-bdc66441e736",
        "statusCode" : "200",
        "responseHeaders" : {
            "Content-Type" : "application/json",
            "Content-Encoding" : "gzip"
        }
         "body" : true
     }
}
----- Web Socket binary frame -FIN
{ "hey" : "mydata" }
----------------------------------
```

##### <a name="responding-via-rendezvous"></a>Respuesta mediante el punto de encuentro

Las respuestas que superen los 64 kB se deben entregar mediante un socket de encuentro. Además, si la solicitud supera los 64 kB y el objeto `request` solo contiene el campo de dirección, se debe establecer un socket de encuentro para obtener `request`. Cuando se haya establecido un socket de encuentro, las respuestas a las solicitudes del cliente respectivo y a las posteriores desde ese cliente respectivo se DEBEN enviar a través del socket de encuentro mientras persista.

La URL `address` del objeto `request` debe usarse tal cual para establecer el socket de encuentro, pero es necesario que contenga los parámetros siguientes:

| Parámetro      | Obligatorio | Descripción
| -------------- | -------- | -------------------------------------------------------------------
| `sb-hc-action` | Sí      | Para aceptar un socket, el parámetro tiene que ser `sb-hc-action=request`

Si hay un error, el servicio puede responder como se indica a continuación:

| Código | Error           | Descripción
| ---- | --------------- | -----------------------------------
| 400  | Solicitud no válida | Acción no reconocida o dirección URL no válida.
| 403  | Prohibido       | La dirección URL ha caducado.
| 500  | Error interno  | Se produjo un error en el servicio.

 Una vez que se establece la conexión, el servidor apaga el WebSocket cuando se apaga el socket HTTP del cliente, o cuando el estado es el siguiente:

| Estado de WS | Descripción                                                                     |
| --------- | ------------------------------------------------------------------------------- |
| 1001      | El cliente remitente cierra la conexión.                                    |
| 1001      | La ruta de acceso de Conexión híbrida se ha eliminado o deshabilitado.                        |
| 1008      | El token de seguridad ha expirado, por lo que se ha infringido la directiva de autorización. |
| 1011      | Se ha producido un error en el servicio.                                            |


#### <a name="listener-token-renewal"></a>Renovación del token del agente de escucha

Cuando el token del agente de escucha está a punto de expirar, puede reemplazarse con el envío de un mensaje en un marco de texto al servicio a través del canal de control establecido. El mensaje contiene un objeto JSON llamado `renewToken`, que define las siguientes propiedades:

* **token**: un token válido de acceso compartido de Service Bus y con codificación URL para el espacio de nombres o la conexión híbrida que concede el derecho de **escucha**.

```json
{
  "renewToken": {
    "token":
      "SharedAccessSignature sr=http%3a%2f%2fcontoso.servicebus.windows.net%2fhyco%2f&amp;sig=XXXXXXXXXX%3d&amp;se=1471633754&amp;skn=SasKeyName"
  }
}
```

Si se produce un error en la validación del token, se deniega el acceso y el servicio en la nube cierra el WebSocket del canal de control con un error. En caso contrario, no se produce ninguna respuesta.

| Estado de WS | Descripción                                                                     |
| --------- | ------------------------------------------------------------------------------- |
| 1008      | El token de seguridad ha expirado, por lo que se ha infringido la directiva de autorización. |

### <a name="web-socket-connect-protocol"></a>Protocolo de conexión del socket web

El protocolo del remitente es idéntico a la forma en que se establece un agente de escucha.
El objetivo es dotar de la máxima transparencia al WebSocket de un extremo a otro. La dirección a la que conectarse es la misma que la del agente de escucha, pero el objeto "action" es diferente y el token necesita otro permiso:

```
wss://{namespace-address}/$hc/{path}?sb-hc-action=...&sb-hc-id=...&sbc-hc-token=...
```

_namespace-address_ es el nombre de dominio completo del espacio de nombres de Azure Relay que hospeda la conexión híbrida, normalmente del formulario `{myname}.servicebus.windows.net`.

La solicitud puede contener más encabezados HTTP arbitrarios, incluidos los definidos por la aplicación. Todos los encabezados especificados se transmiten al agente de escucha y pueden encontrarse en el objeto `connectHeader` del mensaje de control **accept**.

Estas son las opciones de los parámetros de cadena de consulta:

| Parámetro          | ¿Necesario? | Descripción
| -------------- | --------- | -------------------------- |
| `sb-hc-action` | Sí       | En el caso del rol de remitente, el parámetro debe ser `sb-hc-action=connect`.
| `{path}`       | Sí       | (vea el párrafo siguiente)
| `sb-hc-token`  | Sí\*     | El agente de escucha necesita proporcionar un token de acceso compartido de Service Bus válido y codificado para URL al espacio de nombres o a la conexión híbrida que concede el derecho de **envío**.
| `sb-hc-id`     | No        | Un id. opcional que permite realizar el seguimiento de diagnóstico completo y que puede usarlo el agente de escucha durante el protocolo de enlace de aceptación.

 `{path}` es la ruta de acceso del espacio de nombres codificado para la dirección URL de la conexión híbrida preconfigurada en la que se registrará este agente de escucha. La expresión `path` se puede ampliar con un sufijo y una expresión de cadena de consulta para comunicarse de nuevo. Si la conexión híbrida se registra en la ruta de acceso `hyco`, la expresión `path` puede ser `hyco/suffix?param=value&...`, seguida de los parámetros de cadena de consulta que se definen aquí. Una expresión completa puede ser como la siguiente:

```
wss://{namespace-address}/$hc/hyco/suffix?param=value&sb-hc-action=...[&sb-hc-id=...&]sbc-hc-token=...
```

La expresión `path` se pasa al agente de escucha del identificador URI de dirección que se incluye en el mensaje de control "accept".

Si se produce un error en la conexión de WebSocket porque la ruta de acceso de la conexión híbrida no está registrada, porque que falta un token, o no es válido, o por cualquier otro error, se utilizará el modelo de comentarios de estado normal de HTTP 1.1. La descripción del estado contiene un identificador de seguimiento de errores que se pueda comunicar al equipo de soporte técnico de Azure:

| Código | Error          | Descripción
| ---- | -------------- | -------------------------------------------------------------------
| 404  | No encontrado      | La ruta de acceso de la conexión híbrida no es válida o el formato de la URL base no es correcto.
| 401  | No autorizado   | Falta el token de seguridad, tiene una estructura incorrecta o no es válido.
| 403  | Prohibido      | El token de seguridad no es válido para esta ruta de acceso y para esta acción.
| 500  | Error interno | Se ha producido un error en el servicio.

Si el servicio cierra intencionadamente la conexión de WebSocket después de haberse configurado inicialmente, el motivo se comunica mediante un código de error adecuado de protocolo WebSocket, junto con un mensaje de error descriptivo, que también incluye un identificador de seguimiento.

| Estado de WS | Descripción
| --------- | ------------------------------------------------------------------------------- 
| 1000      | El agente de escucha cierra el socket.
| 1001      | La ruta de acceso de Conexión híbrida se ha eliminado o deshabilitado.
| 1008      | El token de seguridad ha expirado, por lo que se ha infringido la directiva de autorización.
| 1011      | Se ha producido un error en el servicio.

### <a name="http-request-protocol"></a>Protocolo de solicitud HTTP

El protocolo de solicitud HTTP permite solicitudes HTTP arbitrarias, excepto las actualizaciones de protocolo.
Las solicitudes HTTP se señalan en la dirección de tiempo de ejecución normal de la entidad, sin el infijo $hc que se usa con los clientes de WebSocket de conexiones híbridas.

```
https://{namespace-address}/{path}?sbc-hc-token=...
```

_namespace-address_ es el nombre de dominio completo del espacio de nombres de Azure Relay que hospeda la conexión híbrida, normalmente del formulario `{myname}.servicebus.windows.net`.

La solicitud puede contener más encabezados HTTP arbitrarios, incluidos los definidos por la aplicación. Todos los encabezados proporcionados, excepto los definidos directamente en RFC7230 (consulte [Mensaje de solicitud](#request-message)) fluyen al cliente de escucha y se pueden encontrar en el objeto `requestHeader` del mensaje de **solicitud**.

Estas son las opciones de los parámetros de cadena de consulta:

| Parámetro          | ¿Necesario? | Descripción
| -------------- | --------- | ---------------- |
| `sb-hc-token`  | Sí\*     | El agente de escucha necesita proporcionar un token de acceso compartido de Service Bus válido y codificado para URL al espacio de nombres o a la conexión híbrida que concede el derecho de **envío**.

El token también se transporta en `ServiceBusAuthorization` o en el encabezado HTTP `Authorization`. El token se puede omitir si la conexión híbrida está configurada para permitir solicitudes anónimas.

Dado que el servicio actúa eficazmente como proxy, aunque no sea de verdad un proxy HTTP, agrega un encabezado `Via` o anota el encabezado `Via` existente compatible con [RFC7230, sección 5.7.1](https://tools.ietf.org/html/rfc7230#section-5.7.1).
El servicio agrega el nombre de host del espacio de nombres de retransmisión a `Via`.

| Código | Message  | Descripción                    |
| ---- | -------- | ------------------------------ |
| 200  | Aceptar       | Al menos un agente de escucha atiende la solicitud.  |
| 202  | Accepted | Al menos un agente de escucha ha aceptado la solicitud. |

Si hay un error, el servicio puede responder como se indica a continuación: Si la respuesta se origina desde el servicio o desde el agente de escucha, se puede identificar a través de la presencia del encabezado `Via`. Si el encabezado está presente, la respuesta procede del agente de escucha.

| Código | Error           | Descripción
| ---- | --------------- |--------- |
| 404  | No encontrado       | La ruta de acceso de la conexión híbrida no es válida o el formato de la URL base no es correcto.
| 401  | No autorizado    | Falta el token de seguridad, tiene una estructura incorrecta o no es válido.
| 403  | Prohibido       | El token de seguridad no es válido para esta ruta de acceso y para esta acción.
| 500  | Error interno  | Se ha producido un error en el servicio.
| 503  | Puerta de enlace incorrecta     | No se pudo enrutar la solicitud a ningún agente de escucha.
| 504  | Tiempo de espera de puerta de enlace | La solicitud se enrutó a un agente de escucha, pero el agente de escucha no confirmó la recepción en el tiempo requerido.

## <a name="next-steps"></a>Pasos siguientes

* [Preguntas más frecuentes acerca de Relay](relay-faq.md)
* [Creación de un espacio de nombres](relay-create-namespace-portal.md)
* [Introducción a .NET](relay-hybrid-connections-dotnet-get-started.md)
* [Introducción a Node](relay-hybrid-connections-node-get-started.md)
