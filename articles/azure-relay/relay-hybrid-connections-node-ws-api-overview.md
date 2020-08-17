---
title: Introducción a las API de Node para Azure Relay | Microsoft Docs
description: En este artículo se proporciona información general sobre la API de Node.js para el servicio Azure Relay. También se muestra cómo usar el paquete de Node hyco-ws.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 67d818a95d63b3097f81f799a287fb4c48a1cfb7
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386255"
---
# <a name="relay-hybrid-connections-node-api-overview"></a>Introducción a las API de Node para conexiones híbridas de Relay

## <a name="overview"></a>Información general

El paquete de Node [`hyco-ws`](https://www.npmjs.com/package/hyco-ws) para Conexiones híbridas de Azure Relay es una ampliación del paquete NPM ["ws"](https://www.npmjs.com/package/ws) en el que se basa. Este paquete vuelve a exportar todas las exportaciones de ese paquete base y agrega otras nuevas que habilitan la integración con la característica Conexiones híbridas del servicio Azure Relay. 

Las aplicaciones existentes con `require('ws')` pueden usar este paquete con `require('hyco-ws')` en su lugar, lo que también hace posibles escenarios híbridos en los que una aplicación puede escuchar conexiones WebSocket localmente desde "dentro del firewall", así como por medio de Conexiones híbridas, de forma simultánea.
  
## <a name="documentation"></a>Documentación

Las API están [documentadas en el paquete principal "ws"](https://github.com/websockets/ws/blob/master/doc/ws.md). En este artículo se describe la diferencia de este paquete con esa línea de base. 

Las diferencias principales entre el paquete base y "hyco-ws" es que este último agrega una nueva clase de servidor, exportada por medio de `require('hyco-ws').RelayedServer`, y varios métodos auxiliares.

### <a name="package-helper-methods"></a>Métodos auxiliares del paquete

Hay varios métodos de utilidad disponibles en la exportación de paquete, a los que puede hacer referencia como sigue:

```JavaScript
const WebSocket = require('hyco-ws');

var listenUri = WebSocket.createRelayListenUri('namespace.servicebus.windows.net', 'path');
listenUri = WebSocket.appendRelayToken(listenUri, 'ruleName', '...key...')
...

```

Los métodos auxiliares son para utilizarse con este paquete, pero también los puede usar un servidor de Node para permitir que los clientes web o de dispositivo creen agentes de escucha o remitentes. El servidor utiliza estos métodos pasándoles URI que insertan tokens de corta duración. Estos URI también se pueden usar con pilas WebSocket comunes que no admiten que se establezcan encabezados HTTP para el protocolo de enlace WebSocket. Se admite la inserción de tokens de autorización en el URI principalmente para los escenarios de uso con biblioteca externa. 

#### <a name="createrelaylistenuri"></a>createRelayListenUri

```JavaScript
var uri = createRelayListenUri([namespaceName], [path], [[token]], [[id]])
```

Crea un URI válido para el agente de escucha de conexión híbrida de Azure Relay para el espacio de nombres y la ruta de acceso dados. Este URI se puede usar con la versión de retransmisión de la clase WebSocketServer.

- `namespaceName` (se requiere): el nombre de dominio completo del espacio de nombres de Azure Relay que se va a usar.
- `path` (se requiere): el nombre de una conexión híbrida de Azure Relay existente en dicho espacio de nombres.
- `token` (opcional): un token de acceso de Relay emitido anteriormente que se inserta en el identificador URI del agente de escucha (consulte el ejemplo siguiente).
- `id` (opcional): un identificador de seguimiento que habilita el seguimiento los diagnósticos de un extremo a otro de las solicitudes.

El valor `token` es opcional y solo debe usarse cuando no sea posible enviar encabezados HTTP junto con el protocolo de enlace WebSocket, como sucede con la pila WebSocket de W3C.                  


#### <a name="createrelaysenduri"></a>createRelaySendUri
 
```JavaScript
var uri = createRelaySendUri([namespaceName], [path], [[token]], [[id]])
```

Crea un URI válido para envío de conexión híbrida de Azure Relay para el espacio de nombres y la ruta de acceso dados. Este URI se puede utilizar con cualquier cliente de WebSocket.

- `namespaceName` (se requiere): el nombre de dominio completo del espacio de nombres de Azure Relay que se va a usar.
- `path` (se requiere): el nombre de una conexión híbrida de Azure Relay existente en dicho espacio de nombres.
- `token` (opcional): un token de acceso de Relay emitido anteriormente que se inserta en el identificador URI de envío (consulte el ejemplo siguiente).
- `id` (opcional): un identificador de seguimiento que habilita el seguimiento los diagnósticos de un extremo a otro de las solicitudes.

El valor `token` es opcional y solo debe usarse cuando no sea posible enviar encabezados HTTP junto con el protocolo de enlace WebSocket, como sucede con la pila WebSocket de W3C.                   


#### <a name="createrelaytoken"></a>createRelayToken 

```JavaScript
var token = createRelayToken([uri], [ruleName], [key], [[expirationSeconds]])
```

Crea un token de firma de acceso compartido (SAS) de Azure Relay para el identificador URI de destino, la regla de SAS y la clave de regla de SAS dados que es válido durante el número de segundos determinado o durante una hora, a partir del instante actual si se omite el argumento de expiración.

- `uri` (se requiere): el identificador URI para el que va a emitir el token. El identificador URI se normaliza para usar el esquema HTTP y se quita la información de la cadena de consulta.
- `ruleName` (se requiere): nombre de regla de SAS de la entidad representada por el identificador URI dado o del espacio de nombres representado por la parte de host del identificador URI.
- `key` (se requiere): clave válida para la regla SAS. 
- `expirationSeconds` (opcional): número de segundos que faltan para que expire el token generado. Si no se especifica, el valor predeterminado es 1 hora (3600).

El token emitido confiere los derechos asociados a la regla de SAS especificada para la duración dada.

#### <a name="appendrelaytoken"></a>appendRelayToken

```JavaScript
var uri = appendRelayToken([uri], [ruleName], [key], [[expirationSeconds]])
```

Este método es funcionalmente equivalente al método `createRelayToken` que se documentó anteriormente, pero devuelve el token anexado correctamente al identificador URI de entrada.

### <a name="class-wsrelayedserver"></a>Clase ws.RelayedServer

La clase `hycows.RelayedServer` es una alternativa a la clase `ws.Server` que no escucha en la red local, sino que delega la escucha en el servicio Azure Relay.

Ambas clases son en su mayor parte compatibles con los contratos, lo que significa que cualquier aplicación existente que use la clase `ws.Server` puede cambiarse fácilmente para que use la versión retransmitida. Las diferencias principales residen en el constructor y en las opciones disponibles.

#### <a name="constructor"></a>Constructor  

```JavaScript 
var ws = require('hyco-ws');
var server = ws.RelayedServer;

var wss = new server(
    {
        server: ws.createRelayListenUri(ns, path),
        token: function() { return ws.createRelayToken('http://' + ns, keyrule, key); }
    });
```

El constructor `RelayedServer` admite un conjunto de argumentos diferente del de `Server`, ya que ni es un agente de escucha independiente ni puede insertarse en un marco de trabajo del agente de escucha HTTP existente. Además, hay menos opciones disponibles porque la administración de WebSocket se delega en gran medida en el servicio Relay.

Argumentos del constructor:

- `server` (se requiere): el identificador URI completo de un nombre de conexión híbrida en la que se escucha, y que normalmente se construye con el método auxiliar WebSocket.createRelayListenUri().
- `token` (se requiere): este argumento contiene una cadena de token emitida anteriormente o una función de devolución de llamada a la que se puede llamar para obtener este tipo de cadena de token. Se prefiere la opción de devolución de llamada, puesto que permite la renovación del token.

#### <a name="events"></a>Eventos

Las instancias de `RelayedServer` emiten tres eventos que permiten controlar las solicitudes entrantes, establecer conexiones y detectar condiciones de error. Debe suscribirse al evento `connect` para controlar los mensajes. 

##### <a name="headers"></a>headers

```JavaScript 
function(headers)
```

El evento `headers` se genera inmediatamente antes de que se acepte una conexión entrante, lo que permite que se modifiquen los encabezados que se envían al cliente. 

##### <a name="connection"></a>conexión

```JavaScript
function(socket)
```

Se emite cuando se acepta una nueva conexión de WebSocket. El objeto es de tipo `ws.WebSocket`, igual que con el paquete base.


##### <a name="error"></a>error

```JavaScript
function(error)
```

Si el servidor subyacente emite un error, se reenvía aquí.  

#### <a name="helpers"></a>Asistentes

Para simplificar el inicio de un servidor de retransmisión y la suscripción inmediata a las conexiones entrantes, el paquete expone una función auxiliar sencilla, que también se utiliza en los ejemplos, como se indica a continuación:

##### <a name="createrelayedlistener"></a>createRelayedListener

```JavaScript
var WebSocket = require('hyco-ws');

var wss = WebSocket.createRelayedServer(
    {
        server: WebSocket.createRelayListenUri(ns, path),
        token: function() { return WebSocket.createRelayToken('http://' + ns, keyrule, key); }
    }, 
    function (ws) {
        console.log('connection accepted');
        ws.onmessage = function (event) {
            console.log(JSON.parse(event.data));
        };
        ws.on('close', function () {
            console.log('connection closed');
        });       
});
``` 

##### <a name="createrelayedserver"></a>createRelayedServer

```javascript
var server = createRelayedServer([options], [connectCallback] )
```

Este método llama al constructor para crear una instancia de RelayedServer y después suscribe la devolución de llamada proporcionada al evento "connection".
 
##### <a name="relayedconnect"></a>relayedConnect

Con solo reflejar el asistente `createRelayedServer` en la función, `relayedConnect` crea una conexión de cliente y se suscribe al evento &quot;open&quot; en el socket resultante.

```JavaScript
var uri = WebSocket.createRelaySendUri(ns, path);
WebSocket.relayedConnect(
    uri,
    WebSocket.createRelayToken(uri, keyrule, key),
    function (socket) {
        ...
    }
);
```

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre Azure Relay, visite estos vínculos:
* [¿Qué es Relay de Azure?](relay-what-is-it.md)
* [Available Relay APIs ](relay-api-overview.md) (API de Relay disponibles)
