---
title: Comprender la compatibilidad con MQTT del Servicio de aprovisionamiento de dispositivos de Azure IoT | Microsoft Docs
description: 'Guía para desarrolladores: compatibilidad con dispositivos que se conectan al punto de conexión de Azure IoT Hub Device Provisioning Service (DPS) mediante el protocolo MQTT.'
author: rajeevmv
ms.service: iot-dps
services: iot-dps
ms.topic: conceptual
ms.date: 10/16/2019
ms.author: ravokkar
ms.custom:
- amqp
- mqtt
ms.openlocfilehash: 7683f5d60c5d788707e2f89774cee42e7820db87
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87924213"
---
# <a name="communicate-with-your-dps-using-the-mqtt-protocol"></a>Comunicación con su DPS mediante el protocolo MQTT

DPS permite que los dispositivos se comuniquen con el punto final del dispositivo DPS mediante:

* El protocolo [MQTT v3.1.1](https://mqtt.org/) en el puerto 8883
* El protocolo [MQTT v3.1.1](http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/os/mqtt-v3.1.1-os.html#_Toc398718127) sobre WebSocket en el puerto 443.

DPS no es un agente MQTT completo y no admite todos los comportamientos especificados en la norma MQTT v3.1.1. En este artículo se describe la manera en que los dispositivos pueden utilizar comportamientos admitidos de MQTT para comunicarse con DPS.

Toda comunicación de los dispositivos con DPS se debe proteger mediante TLS/SSL. Por lo tanto, DPS no admite conexiones no seguras a través del puerto 1883.

 > [!NOTE] 
 > Actualmente, DPS no admite dispositivos que utilizan el [mecanismo de certificación](https://docs.microsoft.com/azure/iot-dps/concepts-device#attestation-mechanism) TPM a través del protocolo MQTT.

## <a name="connecting-to-dps"></a>Conexión a DPS

Un dispositivo puede usar el protocolo MQTT para conectarse a un centro de DPS mediante cualquiera de las opciones siguientes.

* Bibliotecas en el [SDK de aprovisionamiento de Azure IoT](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-sdks#microsoft-azure-provisioning-sdks).
* El protocolo MQTT directamente.

## <a name="using-the-mqtt-protocol-directly-as-a-device"></a>Uso del protocolo MQTT directamente (como un dispositivo)

Si un dispositivo no puede usar los SDK de dispositivo, tendrá la posibilidad de conectarse a los puntos de conexión públicos del dispositivo mediante el protocolo MQTT en el puerto 8883. En el paquete **CONNECT** el dispositivo debe usar los siguientes valores:

* Para el campo **ClientId**, use **registrationId**.

* Para el campo **Nombre de usuario** use `{idScope}/registrations/{registration_id}/api-version=2019-03-31`, donde `{idScope}` es el [idScope](https://docs.microsoft.com/azure/iot-dps/concepts-device#id-scope) del DPS.

* Para el campo **Contraseña** , use un token SAS. El formato del token de SAS es el mismo que para los protocolos HTTPS y AMQP:

  `SharedAccessSignature sr={URL-encoded-resourceURI}&sig={signature-string}&se={expiry}&skn=registration` El resourceURI debe tener el formato `{idScope}/registrations/{registration_id}`. El nombre de la directiva debe ser `registration`.

  > [!NOTE]
  > Las contraseñas de token de SAS no son necesarias si utiliza la autenticación de certificados X.509.

  Para obtener más información sobre cómo generar tokens SAS, consulte la sección de tokens de seguridad de [Control de acceso a DPS](how-to-control-access.md#security-tokens).

A continuación se muestra una lista de comportamientos específicos de la implementación de DPS:

 * DPS no admite la funcionalidad del indicador **CleanSession** que se establece en **0**.

 * Cuando una aplicación de dispositivo se suscribe a un tema con **QoS 2**, DPS concede el nivel de QoS máximo 1 en el paquete **SUBACK**. Después de eso, DPS envía mensajes al dispositivo con QoS 1.

## <a name="tlsssl-configuration"></a>Configuración de TLS/SSL

Para usar el protocolo MQTT directamente, el cliente *debe* conectarse mediante TLS 1.2. Si intenta omitir este paso, se producirá un error con errores de conexión.


## <a name="registering-a-device"></a>Registro de un dispositivo

Para registrar un dispositivo mediante DPS, un dispositivo debe suscribirse mediante `$dps/registrations/res/#` como un **filtro de tema**. El comodín de varios niveles `#` en el filtro de tema solo se utiliza para permitir que el dispositivo reciba propiedades adicionales en el nombre del tema. DPS no permite el uso de los caracteres comodín `#` o `?` para el filtrado de subtemas. Puesto que DPS no es un agente de mensajería de publicación-suscripción de propósito general, solo admite los filtros de tema y los nombres de tema documentados.

El dispositivo debe publicar un mensaje de registro en DPS mediante `$dps/registrations/PUT/iotdps-register/?$rid={request_id}` como **Nombre del tema**. La carga útil debe contener el objeto [Registro de dispositivo](https://docs.microsoft.com/rest/api/iot-dps/runtimeregistration/registerdevice#deviceregistration) en formato JSON.
En un escenario exitoso, el dispositivo recibirá una respuesta sobre el nombre del tema `$dps/registrations/res/202/?$rid={request_id}&retry-after=x` donde x es el valor de reintento en segundos. La carga útil de la respuesta contendrá el objeto [RegistrationOperationStatus](https://docs.microsoft.com/rest/api/iot-dps/runtimeregistration/registerdevice#registrationoperationstatus) en formato JSON.

## <a name="polling-for-registration-operation-status"></a>Sondeo para el estado de la operación de registro

El dispositivo debe sondear el servicio periódicamente para recibir el resultado de la operación de registro del dispositivo. Suponiendo que el dispositivo ya se ha suscrito al tema `$dps/registrations/res/#` como se indicó anteriormente, puede publicar un mensaje Get operationstatus en el nombre del tema `$dps/registrations/GET/iotdps-get-operationstatus/?$rid={request_id}&operationId={operationId}`. El identificador de operación de este mensaje debe ser el valor recibido en el mensaje de respuesta RegistrationOperationStatus en el paso anterior. En el caso de que se realice correctamente, el servicio responderá en el tema `$dps/registrations/res/200/?$rid={request_id}`. La carga útil de la respuesta contendrá el objeto RegistrationOperationStatus. El dispositivo debe seguir sondeando el servicio si el código de respuesta es 202 después de un retraso igual al período de reintento. La operación de registro del dispositivo es exitosa si el servicio devuelve un código de estado 200.

## <a name="connecting-over-websocket"></a>Conexión a través de Websocket
Al conectarse a través de Websocket, especifique el subprotocolo como `mqtt`. Siga [RFC 6455](https://tools.ietf.org/html/rfc6455).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el protocolo MQTT, consulte la [documentación de MQTT](https://mqtt.org/).

Para explorar aún más las funcionalidades de DPS, consulte:

* [Acerca de IoT DPS](about-iot-dps.md)
