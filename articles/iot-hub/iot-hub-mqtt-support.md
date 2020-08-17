---
title: Información sobre Azure MQTT con IoT Hub | Microsoft Docs
description: 'Guía del desarrollador: compatibilidad con dispositivos que se conectan a un punto de conexión orientado a dispositivos IoT Hub mediante el protocolo MQTT. Incluye información sobre la compatibilidad integrada de MQTT de los SDK de dispositivo IoT de Azure.'
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 10/12/2018
ms.author: robinsh
ms.custom:
- amqp
- mqtt
- 'Role: IoT Device'
- 'Role: Cloud Development'
ms.openlocfilehash: c11de5daacfd0d0b3d12c38064dac704c98ce60b
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924196"
---
# <a name="communicate-with-your-iot-hub-using-the-mqtt-protocol"></a>Comunicación con la instancia de IoT Hub mediante el protocolo MQTT

IoT Hub permite a los dispositivos comunicarse con los puntos de conexión de dispositivos de IoT Hub mediante:

* El protocolo [MQTT v3.1.1](https://mqtt.org/) en el puerto 8883
* El protocolo MQTT v3.1.1 sobre WebSocket en el puerto 443.

IoT Hub no es un agente MQTT completo y no es compatible con todos los comportamientos especificados en el estándar MQTT v3.1.1. En este artículo se describe la manera en que los dispositivos pueden utilizar comportamientos admitidos de MQTT para comunicarse con IoT Hub.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

Toda comunicación de los dispositivos con IoT Hub se debe proteger mediante TLS/SSL. Por lo tanto, IoT Hub no admite conexiones no seguras a través del puerto 1883.

## <a name="connecting-to-iot-hub"></a>Conexión a IoT Hub

Un dispositivo puede usar el protocolo MQTT para conectarse a un centro de IoT mediante cualquiera de las opciones siguientes.

* Bibliotecas en los [SDK de Azure IoT](https://github.com/Azure/azure-iot-sdks).
* El protocolo MQTT directamente.

El puerto MQTT (8883) está bloqueado en muchos entornos de red corporativos y educativos. Si no puede abrir el puerto 8883 en el firewall, se recomienda usar MQTT sobre Web Sockets. MQTT sobre Web Sockets se comunica a través del puerto 443, que casi siempre está abierto en entornos de red. Para obtener información sobre cómo especificar los protocolos MQTT y MQTT sobre Web Sockets cuando use los SDK de IoT de Azure, consulte [Uso de los SDK de dispositivo](#using-the-device-sdks).

## <a name="using-the-device-sdks"></a>Uso de los SDK de dispositivo

Los [SDK de dispositivo](https://github.com/Azure/azure-iot-sdks) compatibles con el protocolo MQTT están disponibles para Java, Node.js, C, C# y Python. Los SDK de dispositivo usan la cadena de conexión de IoT Hub estándar para establecer una conexión a un centro de IoT. Para utilizar el protocolo MQTT, el parámetro de protocolo de cliente debe establecerse en **MQTT**. También puede especificar MQTT sobre Web Sockets en el parámetro de protocolo de cliente. De forma predeterminada, los SDK de dispositivo se conectan a un centro de IoT con la marca **CleanSession** establecida en **0** y usan **QoS 1** para el intercambio de mensajes con el centro de IoT. Aunque es posible configurar **QoS 0** para un intercambio de mensajes más rápido, debe tener en cuenta que la entrega no está garantizada ni confirmada. Por esta razón, **QoS 0** a menudo se denomina "dispare y olvídese".

Cuando un dispositivo se conecta a un centro de IoT, los SDK de dispositivo proporcionan métodos que permiten al dispositivo intercambiar mensajes desde un centro de IoT.

La tabla siguiente contiene vínculos a ejemplos de código para cada idioma admitido y especifica el parámetro que se debe utilizar para establecer una conexión con Azure IoT Hub mediante el protocolo MQTT o MQTT sobre Web Sockets.

| Idioma | Parámetro de protocolo MQTT | Parámetro de protocolo MQTT sobre Web Sockets
| --- | --- | --- |
| [Node.js](https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/simple_sample_device.js) | azure-iot-device-mqtt.Mqtt | azure-iot-device-mqtt.MqttWs |
| [Java](https://github.com/Azure/azure-iot-sdk-java/blob/master/device/iot-device-samples/send-receive-sample/src/main/java/samples/com/microsoft/azure/sdk/iot/SendReceive.java) |[IotHubClientProtocol](https://docs.microsoft.com/java/api/com.microsoft.azure.sdk.iot.device.iothubclientprotocol?view=azure-java-stable).MQTT | IotHubClientProtocol.MQTT_WS |
| [C](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples/iothub_client_sample_mqtt_dm) | [MQTT_Protocol](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/iothubtransportmqtt-h/mqtt-protocol) | [MQTT_WebSocket_Protocol](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/iothubtransportmqtt-websockets-h/mqtt-websocket-protocol) |
| [C#](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/iothub/device/samples) | [TransportType](https://docs.microsoft.com/dotnet/api/microsoft.azure.devices.client.transporttype?view=azure-dotnet).Mqtt | TransportType.Mqtt recurre a MQTT sobre Web Sockets si se produce un error en MQTT. Para especificar solo MQTT sobre Web Sockets, use TransportType.Mqtt_WebSocket_Only. |
| [Python](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples) | Compatible con MQTT de forma predeterminada | Agregue `websockets=True` en la llamada para crear el cliente. |

En el siguiente fragmento se muestra cómo especificar el protocolo MQTT sobre Web Sockets al usar el SDK de Node.js para IoT de Azure:

```javascript
var Client = require('azure-iot-device').Client;
var Protocol = require('azure-iot-device-mqtt').MqttWs;
var client = Client.fromConnectionString(deviceConnectionString, Protocol);
```

En el siguiente fragmento se muestra cómo especificar el protocolo MQTT sobre Web Sockets al usar el SDK de Python para IoT de Azure:

```python
from azure.iot.device.aio import IoTHubDeviceClient
device_client = IoTHubDeviceClient.create_from_connection_string(deviceConnectionString, websockets=True)
```

### <a name="default-keep-alive-timeout"></a>Tiempo de espera de Keep-Alive predeterminado

Para asegurarse de que una conexión cliente/IoT Hub permanece activa, tanto el servicio como el cliente se envían regularmente un ping *Keep-Alive* entre sí. El cliente que usa el SDK de IoT envía un mensaje de Keep-Alive en el intervalo definido en la tabla siguiente:

|Idioma  |Intervalo de mantenimiento de conexión predeterminado  |Configurable  |
|---------|---------|---------|
|Node.js     |   180 Segundos      |     No    |
|Java     |    230 Segundos     |     No    |
|C     | 240 segundos |  [Sí](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/Iothub_sdk_options.md#mqtt-transport)   |
|C#     | 300 segundos |  [Sí](https://github.com/Azure/azure-iot-sdk-csharp/blob/master/iothub/device/src/Transport/Mqtt/MqttTransportSettings.cs#L89)   |
|Python   | 60 segundos |  No   |

Siguiendo la [especificación de MQTT](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718081), el intervalo de ping de Keep-Alive de IoT Hub es 1,5 veces el valor de Keep-Alive del cliente. Sin embargo, IoT Hub limita el tiempo de espera máximo del servidor a 29,45 minutos (1 767 segundos) porque todos los servicios de Azure están enlazados al tiempo de espera de inactividad TCP del equilibrador de carga de Azure, que es de 29,45 minutos. 

Por ejemplo, un dispositivo que usa el SDK de Java envía el ping de Keep-Alive y después pierde la conectividad de red. 230 segundos más tarde, el dispositivo pierde el ping de Keep-Alive porque está sin conexión. Sin embargo, IoT Hub no cierra la conexión de inmediato, espera otros `(230 * 1.5) - 230 = 115` segundos antes de desconectar el dispositivo con el error [404104 DeviceConnectionClosedRemotely](iot-hub-troubleshoot-error-404104-deviceconnectionclosedremotely.md). 

El valor de Keep-Alive máximo del cliente que puede establecer es `1767 / 1.5 = 1177` segundos. Cualquier tráfico restablecerá el valor de Keep-Alive. Por ejemplo, una actualización correcta del token de SAS restablece el valor de Keep-Alive.

### <a name="migrating-a-device-app-from-amqp-to-mqtt"></a>Migración de una aplicación de dispositivo de AMQP a MQTT

Si usa los [SDK de dispositivo](https://github.com/Azure/azure-iot-sdks), el cambio de AMQP a MQTT requiere modificar el parámetro de protocolo en la inicialización del cliente, tal como se indicó anteriormente.

Al hacerlo, asegúrese de comprobar los elementos siguientes:

* AMQP devuelve errores para muchas condiciones, mientras que MQTT finaliza la conexión. Como resultado, la lógica de control de excepciones puede requerir algunos cambios.

* MQTT no admite las operaciones *rechazar* al recibir [mensajes de la nube al dispositivo](iot-hub-devguide-messaging.md). Si la aplicación de back-end tiene que recibir una respuesta de la aplicación del dispositivo, considere el uso de [métodos directos](iot-hub-devguide-direct-methods.md).

* No se admite AMQP en el SDK de Python

## <a name="using-the-mqtt-protocol-directly-as-a-device"></a>Uso del protocolo MQTT directamente (como un dispositivo)

Si un dispositivo no puede usar los SDK de dispositivo, tendrá la posibilidad de conectarse a los puntos de conexión públicos del dispositivo mediante el protocolo MQTT en el puerto 8883. En el paquete **CONNECT** el dispositivo debe usar los siguientes valores:

* Para el campo **ClientId**, use **deviceId**.

* Para el campo **Nombre de usuario** use `{iothubhostname}/{device_id}/?api-version=2018-06-30`, donde `{iothubhostname}` es el CName completo del centro de IoT.

    Por ejemplo, si el nombre de su centro de IoT es **contoso.azure-devices.net** y si el nombre del dispositivo es **MyDevice01**, el campo **Nombre de usuario** completo debe contener:

    `contoso.azure-devices.net/MyDevice01/?api-version=2018-06-30`

* Para el campo **Contraseña** , use un token SAS. El formato del token de SAS es el mismo que para los protocolos HTTPS y AMQP:

  `SharedAccessSignature sig={signature-string}&se={expiry}&sr={URL-encoded-resourceURI}`

  > [!NOTE]
  > Las contraseñas de token de SAS no son necesarias si utiliza la autenticación de certificados X.509. Para obtener más información, consulte [Configuración de la seguridad de X.509 en Azure IoT Hub](iot-hub-security-x509-get-started.md) y siga las instrucciones de código [siguientes](#tlsssl-configuration).

  Para obtener más información sobre cómo generar tokens de SAS, consulte la sección del dispositivo sobre el [uso de tokens de seguridad de IoT Hub](iot-hub-devguide-security.md#use-sas-tokens-in-a-device-app).

  Al realizar la prueba, también puede utilizar la multiplataforma [Azure IoT Tools para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools) o el comando de la extensión CLI [az iot hub generate-sas-token](/cli/azure/ext/azure-iot/iot/hub?view=azure-cli-latest#ext-azure-iot-az-iot-hub-generate-sas-token) para generar rápidamente un token de SAS que puede copiar y pegar en su propio código:

### <a name="for-azure-iot-tools"></a>Para Azure IoT Tools

1. Expanda la pestaña **AZURE IOT HUB DEVICES** (Dispositivos de Azure IoT Hub) en la esquina inferior izquierda de Visual Studio Code.
  
2. Haga clic con el botón derecho en el dispositivo y seleccione **Generate SAS Token for Device** (Generar token de SAS para el dispositivo).
  
3. Establezca la **hora de expiración** y presione "Entrar".
  
4. El token de SAS se crea y se copia en el Portapapeles.

   El token de SAS que ha generado tiene la siguiente estructura:

   `HostName={your hub name}.azure-devices.net;DeviceId=javadevice;SharedAccessSignature=SharedAccessSignature sr={your hub name}.azure-devices.net%2Fdevices%2FMyDevice01%2Fapi-version%3D2016-11-14&sig=vSgHBMUG.....Ntg%3d&se=1456481802`

   La parte de este token que se debe usar como el campo **Password** para conectarse con MQTT es:

   `SharedAccessSignature sr={your hub name}.azure-devices.net%2Fdevices%2FMyDevice01%2Fapi-version%3D2016-11-14&sig=vSgHBMUG.....Ntg%3d&se=1456481802`

Para que MQTT conecte y desconecte paquetes, Azure IoT Hub emite un evento en el canal **Supervisión de operaciones** . Este evento tiene información adicional que puede ayudarle a solucionar problemas de conectividad.

La aplicación para dispositivo puede especificar un mensaje**Will** en el paquete **CONNECT**. La aplicación para dispositivo debe usar `devices/{device_id}/messages/events/` o `devices/{device_id}/messages/events/{property_bag}` como el nombre del tema **Will** para definir los mensajes **Will** que se reenviarán como un mensaje de telemetría. En este caso, si la conexión de red se cierra, pero no se recibió anteriormente un paquete **DISCONNECT** desde el dispositivo, IoT Hub envía el mensaje **Will** que se suministra en el paquete **CONNECT** al canal de telemetría. El canal de telemetría puede ser el punto de conexión **Eventos** predeterminado o un punto de conexión personalizado que se define por el enrutamiento de IoT Hub. El mensaje tiene la propiedad **iothub-MessageType** con un valor de **Will** asignado.

### <a name="an-example-of-c-code-using-mqtt-without-azure-iot-c-sdk"></a>Un ejemplo de código de C que usa MQTT sin el SDK de C de Azure IoT
En este [repositorio](https://github.com/Azure-Samples/IoTMQTTSample), encontrará un par de proyectos de demostración de C/C++ en los que se muestra cómo enviar mensajes de telemetría, recibir eventos con un centro de IOT sin usar el SDK de C de Azure IoT. 

En estos ejemplos se usa la biblioteca Eclipse Mosquitto para enviar mensajes al agente MQTT implementado en IoT Hub.

Este repositorio contiene:

**Para Windows:**

* TelemetryMQTTWin32: contiene código para enviar un mensaje de telemetría a una instancia de Azure IoT Hub compilada y ejecutada en una máquina Windows.

* SubscribeMQTTWin32: contiene código para suscribirse a eventos de un centro de IoT determinado en una máquina Windows.

* DeviceTwinMQTTWin32: contiene código para consultar y suscribirse a los eventos de dispositivo gemelo de un dispositivo en Azure IoT Hub en una máquina Windows.

* PnPMQTTWin32: contiene código para enviar un mensaje de telemetría con funcionalidad de dispositivo IoT Plug & Play en versión preliminar a una instancia de Azure IoT Hub compilada y ejecutada en una máquina Windows. Más información sobre IoT Plug & Play [aquí](https://docs.microsoft.com/azure/iot-pnp/overview-iot-plug-and-play)

**Para Linux:**

* MQTTLinux: contiene código y un script de compilación para ejecutar en Linux (hasta ahora, se han probado WSL, Ubuntu y Raspbian).

* LinuxConsoleVS2019: contiene el mismo código, pero en un proyecto de VS2019 que tiene como destino WSL (subsistema Windows Linux). Este proyecto permite depurar el código que se ejecuta en Linux paso a paso desde Visual Studio.

**Para mosquitto_pub:**

Esta carpeta contiene dos comandos de ejemplo que se usan con la herramienta de utilidad mosquitto_pub que proporciona Mosquitto.org.

* Mosquitto_sendmessage: para enviar un mensaje de texto simple a una instancia de Azure IoT Hub que actúa como dispositivo.

* Mosquitto_subscribe: para ver los eventos que se producen en una instancia de Azure IoT Hub.

## <a name="using-the-mqtt-protocol-directly-as-a-module"></a>Uso del protocolo MQTT directamente (como un módulo)

La conexión a IoT Hub a través de MQTT con una identidad de módulo es similar al dispositivo (se describe [anteriormente](#using-the-mqtt-protocol-directly-as-a-device)), pero debe hacer lo siguiente:

* Establezca el identificador de cliente en `{device_id}/{module_id}`.

* Si la autenticación es mediante el nombre de usuario y la contraseña, establezca el nombre de usuario en `<hubname>.azure-devices.net/{device_id}/{module_id}/?api-version=2018-06-30` y use el token de SAS asociado con la identidad del módulo como contraseña.

* Use `devices/{device_id}/modules/{module_id}/messages/events/` como tema para publicar los datos de telemetría.

* Use `devices/{device_id}/modules/{module_id}/messages/events/` como tema de WILL.

* Los temas de GET y PATCH gemelos son idénticos para los dispositivos y los módulos.

* El tema de estado gemelo es idéntico para los módulos y los dispositivos.

## <a name="tlsssl-configuration"></a>Configuración de TLS/SSL

Para usar el protocolo MQTT directamente, el cliente *debe* conectarse mediante TLS/SSL. Si intenta omitir este paso, se producirá un error con errores de conexión.

Para establecer una conexión TLS, es posible que deba descargar el certificado raíz de DigiCert Baltimore. Este certificado es el que Azure utiliza para proteger la conexión Puede encontrar este certificado en el repositorio [Azure-iot-sdk-c](https://github.com/Azure/azure-iot-sdk-c/blob/master/certs/certs.c). Para obtener más información acerca de estos certificados, vaya al [sitio web de Digicert](https://www.digicert.com/digicert-root-certificates.htm).

Un ejemplo de cómo implementar esto con la versión de Python de la [biblioteca Paho MQTT](https://pypi.python.org/pypi/paho-mqtt) de la Fundación Eclipse podría tener el aspecto siguiente.

En primer lugar, instale la biblioteca Paho desde el entorno de línea de comandos:

```cmd/sh
pip install paho-mqtt
```

A continuación, implemente el cliente en un script de Python. Reemplace los marcadores de posición de la siguiente forma:

* `<local path to digicert.cer>` es la ruta de acceso a un archivo local que contiene el certificado raíz de DigiCert Baltimore. Puede crear este archivo copiando la información de certificado de [certs.c](https://github.com/Azure/azure-iot-sdk-c/blob/master/certs/certs.c) en el SDK de Azure IoT para C. Incluya las líneas `-----BEGIN CERTIFICATE-----` y `-----END CERTIFICATE-----`, quite las marcas `"` al principio y al final de cada línea, y quite los caracteres `\r\n` al final de cada línea.

* `<device id from device registry>` es la identidad de un dispositivo que agregó a IoT Hub.

* `<generated SAS token>` es un token de SAS para el dispositivo que se creó como se describió anteriormente en este artículo.

* `<iot hub name>` el nombre de IoT Hub.

```python
from paho.mqtt import client as mqtt
import ssl

path_to_root_cert = "<local path to digicert.cer file>"
device_id = "<device id from device registry>"
sas_token = "<generated SAS token>"
iot_hub_name = "<iot hub name>"


def on_connect(client, userdata, flags, rc):
    print("Device connected with result code: " + str(rc))


def on_disconnect(client, userdata, rc):
    print("Device disconnected with result code: " + str(rc))


def on_publish(client, userdata, mid):
    print("Device sent message")


client = mqtt.Client(client_id=device_id, protocol=mqtt.MQTTv311)

client.on_connect = on_connect
client.on_disconnect = on_disconnect
client.on_publish = on_publish

client.username_pw_set(username=iot_hub_name+".azure-devices.net/" +
                       device_id + "/?api-version=2018-06-30", password=sas_token)

client.tls_set(ca_certs=path_to_root_cert, certfile=None, keyfile=None,
               cert_reqs=ssl.CERT_REQUIRED, tls_version=ssl.PROTOCOL_TLSv1_2, ciphers=None)
client.tls_insecure_set(False)

client.connect(iot_hub_name+".azure-devices.net", port=8883)

client.publish("devices/" + device_id + "/messages/events/", "{id=123}", qos=1)
client.loop_forever()
```

Para realizar la autenticación con un certificado de dispositivo, actualice el fragmento de código anterior con los cambios siguientes (consulte [Cómo obtener un certificado de entidad de certificación X.509](./iot-hub-x509ca-overview.md#how-to-get-an-x509-ca-certificate) a fin de saber cómo prepararse para la autenticación basada en certificados):

```python
# Create the client as before
# ...

# Set the username but not the password on your client
client.username_pw_set(username=iot_hub_name+".azure-devices.net/" +
                       device_id + "/?api-version=2018-06-30", password=None)

# Set the certificate and key paths on your client
cert_file = "<local path to your certificate file>"
key_file = "<local path to your device key file>"
client.tls_set(ca_certs=path_to_root_cert, certfile=cert_file, keyfile=key_file,
               cert_reqs=ssl.CERT_REQUIRED, tls_version=ssl.PROTOCOL_TLSv1_2, ciphers=None)

# Connect as before
client.connect(iot_hub_name+".azure-devices.net", port=8883)
```

## <a name="sending-device-to-cloud-messages"></a>Envío de mensajes de dispositivo a nube

Después de realizar con éxito una conexión, un dispositivo puede enviar mensajes a un centro IoT mediante `devices/{device_id}/messages/events/` o `devices/{device_id}/messages/events/{property_bag}` como un **Nombre del tema**. El elemento `{property_bag}` permite al dispositivo enviar mensajes con propiedades adicionales en un formato con codificación URL. Por ejemplo:

```text
RFC 2396-encoded(<PropertyName1>)=RFC 2396-encoded(<PropertyValue1>)&RFC 2396-encoded(<PropertyName2>)=RFC 2396-encoded(<PropertyValue2>)…
```

> [!NOTE]
> Este elemento `{property_bag}` usa la misma codificación que se usa para las cadenas de consulta en el protocolo HTTPS.

Esta es una lista de los comportamientos específicos de la implementación de IoT Hub:

* IoT Hub no admite mensajes QoS 2. Si una aplicación de dispositivo publica un mensaje con **QoS 2**, el IoT Hub cierra la conexión de red.

* IoT Hub no retiene los mensajes de conservación. Si un dispositivo envía un mensaje con la marca **RETAIN** establecida en 1, el IoT Hub agrega la propiedad de aplicación **x-opt-retain** al mensaje. En este caso, en lugar de retener el mensaje de conservación, IoT Hub lo pasa a la aplicación de back-end.

* IoT Hub solo admite una conexión MQTT activa por dispositivo. Todas las conexiones MQTT nuevas en nombre del mismo identificador de dispositivo provocarán que IoT Hub cierre la existente.

Para obtener más información, consulte la [guía del desarrollador de mensajería](iot-hub-devguide-messaging.md).

## <a name="receiving-cloud-to-device-messages"></a>Recepción de mensajes de nube a dispositivo

Para recibir mensajes de IoT Hub, un dispositivo debe suscribirse usando `devices/{device_id}/messages/devicebound/#` como un **filtro de tema**. El comodín de varios niveles `#` en el filtro de tema solo se utiliza para permitir que el dispositivo reciba propiedades adicionales en el nombre del tema. IoT Hub no permite el uso de los caracteres comodín `#` o `?` para el filtrado de subtemas. Puesto que IoT Hub no es un agente de mensajería de publicación-suscripción de propósito general, solo admite los filtros de tema y los nombres de tema documentados.

El dispositivo no recibe ningún mensaje desde IoT Hub hasta que se suscribe correctamente al punto de conexión específico del dispositivo, representado por el filtro del tema `devices/{device_id}/messages/devicebound/#`. Después de que se haya establecido una suscripción, el dispositivo recibe solo los mensajes de la nube al dispositivo que se enviaron a este después de la hora de la suscripción. Si el dispositivo se conecta con la marca **CleanSession** establecida en **0**, la suscripción se conserva entre distintas sesiones. En este caso, la próxima vez que el dispositivo se conecte con **CleanSession 0**, este recibirá los mensajes pendientes que se le han enviado mientras estaba desconectado. Si el dispositivo usa la marca **CleanSession** establecida en **1**, no recibe los mensajes de IoT Hub hasta que se suscribe al punto de conexión del dispositivo.

IoT Hub entrega los mensajes con el **Nombre del tema** `devices/{device_id}/messages/devicebound/` o `devices/{device_id}/messages/devicebound/{property_bag}` si hay propiedades de mensaje. `{property_bag}` contiene pares clave-valor con codificación URL de propiedades del mensaje. Las propiedades de la aplicación y del sistema configuradas por el usuario (como **messageId** o **correlationId**) son las únicas que se incluyen en el paquete de propiedades. Los nombres de propiedad del sistema tienen el prefijo **$** ; las propiedades de aplicaciones utilizan el nombre de propiedad original sin prefijo. Para obtener información adicional sobre el formato del contenedor de propiedades, consulte [Envío de mensajes de dispositivo a nube](#sending-device-to-cloud-messages).

En los mensajes de la nube al dispositivo, los valores del contenedor de propiedades se representan como en la tabla siguiente:

| Valor de propiedad | Representación | Descripción |
|----|----|----|
| `null` | `key` | Solo aparece la clave en el contenedor de propiedades |
| cadena vacía | `key=` | Clave seguida de un signo igual sin valor |
| valor que no es nulo ni está vacío | `key=value` | Clave seguida de un signo igual y un valor |

En el ejemplo siguiente se muestra un contenedor de propiedades que contiene tres propiedades de la aplicación: **prop1** con un valor de `null`; **prop2**, que es una cadena vacía (""); y **prop3** que tiene como valor "a string".

```mqtt
/?prop1&prop2=&prop3=a%20string
```

Cuando una aplicación de dispositivo se suscribe a un tema con **QoS 2**, IoT Hub concede el QoS de nivel 1 (el máximo) en el paquete **SUBACK**. Después de eso, IoT Hub envía mensajes al dispositivo con QoS 1.

## <a name="retrieving-a-device-twins-properties"></a>Recuperación de propiedades del dispositivo gemelo

En primer lugar, un dispositivo se suscribe a `$iothub/twin/res/#` para recibir las respuestas de la operación. A continuación, envía un mensaje en blanco al tema `$iothub/twin/GET/?$rid={request id}`, con un valor en **Id. de solicitud**. El servicio envía entonces un mensaje de respuesta que contiene los datos del dispositivo gemelo del tema `$iothub/twin/res/{status}/?$rid={request id}`, con el mismo **identificador de solicitud** que la solicitud.

El identificador de la solicitud puede ser cualquier valor de propiedad válido de mensaje, según la [guía del desarrollador de mensajería de IoT Hub](iot-hub-devguide-messaging.md), y se comprueba que el estado sea un entero.

El cuerpo de la respuesta contiene la sección de propiedades del dispositivo gemelo, como se muestra en la siguiente respuesta de ejemplo:

```json
{
    "desired": {
        "telemetrySendFrequency": "5m",
        "$version": 12
    },
    "reported": {
        "telemetrySendFrequency": "5m",
        "batteryLevel": 55,
        "$version": 123
    }
}
```

Los códigos de estado posibles son:

|Status | Description |
| ----- | ----------- |
| 200 | Correcto |
| 429 | Demasiadas solicitudes (limitadas), según el artículo sobre [limitaciones de IoT Hub](iot-hub-devguide-quotas-throttling.md) |
| 5** | Errores del servidor |

Para obtener más información, consulte la [guía para desarrolladores sobre dispositivos gemelos](iot-hub-devguide-device-twins.md).

## <a name="update-device-twins-reported-properties"></a>Actualización de las propiedades notificadas del dispositivo gemelo

Para actualizar las propiedades notificadas, el dispositivo emite una solicitud a IoT Hub a través de una publicación en un tema designado de MQTT. Después de procesar la solicitud, IoT Hub responde con el estado de éxito o fracaso de la operación de actualización a través de una publicación en otro tema. El dispositivo puede suscribirse a ese tema con el fin de notificarle sobre el resultado de su solicitud de actualización gemela. Para implementar este tipo de interacción entre solicitud y respuesta en MQTT, hemos aprovechado la noción de identificador de solicitud (`$rid`) proporcionada inicialmente por el dispositivo en su solicitud de actualización. Este identificador de solicitud también se incluye en la respuesta de IoT Hub para permitir que el dispositivo correlacione la respuesta a su solicitud previa específica.

La secuencia siguiente describe cómo un dispositivo actualiza las propiedades notificadas en el dispositivo gemelo de IoT Hub:

1. En primer lugar, un dispositivo debe suscribirse al tema `$iothub/twin/res/#` para recibir las respuestas de la operación de IoT Hub.

2. Un dispositivo envía un mensaje que contiene la actualización del dispositivo gemelo al tema `$iothub/twin/PATCH/properties/reported/?$rid={request id}`. Este mensaje incluye un valor de **identificador de solicitud**.

3. Después, el servicio envía un mensaje de respuesta que contiene el nuevo valor de ETag de la colección de propiedades notificadas del tema `$iothub/twin/res/{status}/?$rid={request id}`. Este mensaje de respuesta usa el mismo **identificador de solicitud** que la solicitud.

El cuerpo del mensaje de solicitud contiene un documento JSON, que contiene los nuevos valores de las propiedades notificadas. En el documento JSON, cada miembro actualiza o agrega al miembro correspondiente en el documento del dispositivo gemelo. La configuración de un miembro en `null`, elimina el miembro del objeto contenedor. Por ejemplo:

```json
{
    "telemetrySendFrequency": "35m",
    "batteryLevel": 60
}
```

Los códigos de estado posibles son:

|Status | Descripción |
| ----- | ----------- |
| 204 | Correcto (sin contenido) |
| 400 | Solicitud incorrecta. JSON con formato incorrecto |
| 429 | Demasiadas solicitudes (limitadas), según el artículo sobre [limitaciones de IoT Hub](iot-hub-devguide-quotas-throttling.md) |
| 5** | Errores del servidor |

En el fragmento de código Python siguiente, se muestra el proceso de actualización de propiedades notificadas por el gemelo a través de MQTT (mediante el cliente de Paho MQTT):

```python
from paho.mqtt import client as mqtt

# authenticate the client with IoT Hub (not shown here)

client.subscribe("$iothub/twin/res/#")
rid = "1"
twin_reported_property_patch = "{\"firmware_version\": \"v1.1\"}"
client.publish("$iothub/twin/PATCH/properties/reported/?$rid=" +
               rid, twin_reported_property_patch, qos=0)
```

Cuando se realiza correctamente la operación anterior de actualización de propiedades notificadas por el gemelo, el mensaje de publicación de IoT Hub tiene el siguiente tema: `$iothub/twin/res/204/?$rid=1&$version=6`, donde `204` es el código de estado que indica éxito, `$rid=1` se corresponde con el identificador de solicitud proporcionada por el dispositivo en el código, y `$version` corresponde a la versión de la sección de propiedades notificadas de dispositivos gemelos tras la actualización.

Para obtener más información, consulte la [guía para desarrolladores sobre dispositivos gemelos](iot-hub-devguide-device-twins.md).

## <a name="receiving-desired-properties-update-notifications"></a>Recepción de notificaciones de actualización de las propiedades deseadas

Cuando se conecta un dispositivo, IoT Hub envía notificaciones al tema `$iothub/twin/PATCH/properties/desired/?$version={new version}`, que contiene la actualización realizada por la solución de back-end. Por ejemplo:

```json
{
    "telemetrySendFrequency": "5m",
    "route": null,
    "$version": 8
}
```

Para las actualizaciones de propiedad, los valores `null` significan que se elimina el miembro del objeto JSON. Además, tenga en cuenta que `$version` indica la nueva versión de la sección de propiedades deseadas del gemelo.

> [!IMPORTANT]
> IoT Hub genera notificaciones de cambio solo si los dispositivos están conectados. Asegúrese de implementar el [flujo de reconexión de dispositivos](iot-hub-devguide-device-twins.md#device-reconnection-flow) para mantener las propiedades que quiera sincronizadas entre IoT Hub y la aplicación del dispositivo.

Para obtener más información, consulte la [guía para desarrolladores sobre dispositivos gemelos](iot-hub-devguide-device-twins.md).

## <a name="respond-to-a-direct-method"></a>Respuesta a un método directo

En primer lugar, tiene que suscribirse un dispositivo a `$iothub/methods/POST/#`. IoT Hub envía solicitudes de método al tema `$iothub/methods/POST/{method name}/?$rid={request id}`, con un valor JSON válido o un cuerpo vacío.

Para responder, el dispositivo envía un mensaje con un valor JSON válido o un cuerpo vacío al tema `$iothub/methods/res/{status}/?$rid={request id}`. En este mensaje, el **identificador de solicitud** debe coincidir con el del mensaje de solicitud y el **estado** debe ser un número entero.

Para obtener más información, consulte la [guía para desarrolladores sobre el método directo](iot-hub-devguide-direct-methods.md).

## <a name="additional-considerations"></a>Consideraciones adicionales

Como última consideración, si tiene que personalizar el comportamiento del protocolo MQTT en el lado de la nube, deberá revisar la [puerta de enlace de protocolo de Azure IoT](iot-hub-protocol-gateway.md). Este software permite implementar una puerta de enlace de protocolo personalizado de alto rendimiento que interactúa directamente con IoT Hub. La puerta de enlace de protocolos de IoT de Azure le permite personalizar el protocolo del dispositivo para dar cabida a las implementaciones de MQTT de Brownfield u otros protocolos personalizados. Sin embargo, este enfoque requiere que se ejecute y se ponga en funcionamiento una puerta de enlace de protocolo personalizado.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el protocolo MQTT, consulte la [documentación de MQTT](https://mqtt.org/).

Para más información acerca de planificación de la implementación de IoT Hub, consulte:

* [Catálogo de dispositivos de Azure Certified for IoT](https://catalog.azureiotsolutions.com/)
* [Compatibilidad con protocolos adicionales](iot-hub-protocol-gateway.md)
* [Comparación con Event Hubs](iot-hub-compare-event-hubs.md)
* [Escalado, alta disponibilidad y recuperación ante desastres](iot-hub-scaling.md)

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Guía para desarrolladores de IoT Hub](iot-hub-devguide.md)
* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)
