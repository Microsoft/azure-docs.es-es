---
title: 'Conexión de dispositivos de bajada: Azure IoT Edge | Microsoft Docs'
description: Se describe cómo configurar dispositivos de bajada o dispositivos hoja para su conexión con dispositivos de puerta de enlace Azure IoT Edge.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 06/02/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.custom:
- amqp
- mqtt
- devx-track-javascript
ms.openlocfilehash: b6d2f502eb6bb6bd70e6d76ec88aa332c9ecdc77
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87419876"
---
# <a name="connect-a-downstream-device-to-an-azure-iot-edge-gateway"></a>Conexión de un dispositivo de bajada a una puerta de enlace Azure IoT Edge

En este artículo se proporcionan instrucciones para establecer una conexión de confianza entre dispositivos de bajada y puertas de enlace transparentes IoT Edge. En un escenario de puerta de enlace transparente, uno o varios dispositivos pueden pasar sus mensajes a través de un dispositivo de puerta de enlace único que mantiene la conexión con IoT Hub.

Hay tres pasos generales para configurar una conexión de puerta de enlace transparente correcta. En este artículo se trata el tercer paso:

1. Configure el dispositivo de puerta de enlace como servidor para que los dispositivos de bajada puedan conectarse a él de forma segura. Configure la puerta de enlace para recibir mensajes de los dispositivos de bajada y enrutarlos al destino adecuado. Para más información, consulte [Configuración de un dispositivo IoT Edge para que actúe como puerta de enlace transparente](how-to-create-transparent-gateway.md).
2. Cree una identidad de dispositivo para el dispositivo de bajada para que pueda autenticarse en IoT Hub. Configure el dispositivo de bajada para enviar mensajes a través del dispositivo de puerta de enlace. Para más información, consulte [Autenticación de un dispositivo de bajada en Azure IoT Hub](how-to-authenticate-downstream-device.md).
3. **Conecte el dispositivo de bajada al dispositivo de puerta de enlace y empiece a enviar mensajes.**

Este artículo identifica problemas comunes con las conexiones de los dispositivos de bajada y le guía en la configuración de los dispositivos de bajada, lo que incluye:

* Descripción de los aspectos básicos de la seguridad de la capa de transporte (TLS) y los certificados.
* Descripción del funcionamiento de las bibliotecas de TLS en distintos sistemas operativos y del tratamiento de los certificados en cada sistema operativo.
* Ejemplos de Azure IoT en varios lenguajes que le ayudarán a comenzar.

En este artículo, los términos *puerta de enlace* y *puerta de enlace IoT Edge* hacen referencia a un dispositivo IoT Edge configurado como una puerta de enlace transparente.

## <a name="prerequisites"></a>Requisitos previos

* Debe disponer del archivo de certificado de entidad de certificación raíz que se usó para generar tal certificado del dispositivo en [Configuración de un dispositivo IoT Edge para que actúe como puerta de enlace transparente](how-to-create-transparent-gateway.md) en su dispositivo de bajada. El dispositivo de bajada usa este certificado para validar la identidad del dispositivo de puerta de enlace. Si usó los certificados de demostración, el certificado de entidad de certificado se denomina **azure-iot-test-only.root.ca.cert.pem**.
* Modifique la cadena de conexión que señala al dispositivo de puerta de enlace, como se explica en [Autenticación de un dispositivo de bajada en Azure IoT Hub](how-to-authenticate-downstream-device.md).

## <a name="prepare-a-downstream-device"></a>Preparación de un dispositivo de bajada

Un dispositivo de bajada puede ser cualquier aplicación o plataforma que tenga una identidad creada con el servicio en la nube Azure IoT Hub. En muchos casos, estas aplicaciones utilizan el [SDK de dispositivo IoT de Azure](../iot-hub/iot-hub-devguide-sdks.md). Un dispositivo de bajada podría ser incluso una aplicación que se ejecuta en el propio dispositivo de puerta de enlace IoT Edge. Sin embargo, otro dispositivo IoT Edge no puede ser inferior a una puerta de enlace de IoT Edge.

>[!NOTE]
>Los dispositivos IoT registrados en IOT Hub pueden usar [módulos gemelos](../iot-hub/iot-hub-devguide-module-twins.md) para aislar diferentes procesos, hardware o funciones en un único dispositivo. Las puertas de enlace de IoT Edge admiten conexiones de módulo inferiores mediante la autenticación de clave simétrica, pero no la autenticación de certificado X.509.

Para conectar un dispositivo de bajada a una puerta de enlace IoT Edge, necesita dos cosas:

* Un dispositivo o aplicación que esté configurada con una cadena de conexión de dispositivo de IoT Hub anexada con información para conectarse a la puerta de enlace.

    Este paso se completó en el artículo anterior, [Autenticación de un dispositivo de bajada en Azure IoT Hub](how-to-authenticate-downstream-device.md#retrieve-and-modify-connection-string).

* El dispositivo o la aplicación tienen que confiar en el certificado de la **entidad de certificación raíz** de la puerta de enlace para validar las conexiones de Seguridad de la capa de transporte (TLS) con el dispositivo de puerta de enlace.

    Este paso se explica con detalle en el resto de este artículo. Este paso se puede ser realizar de dos maneras: mediante la instalación del certificado de entidad de certificación en el almacén de certificados del sistema operativo o (para determinados lenguajes) haciendo referencia al certificado dentro de las aplicaciones mediante los SDK de Azure IoT.

## <a name="tls-and-certificate-fundamentals"></a>Aspectos básicos de TLS y los certificados

El desafío de una conexión segura de los dispositivos de bajada a IoT Edge es igual que en cualquier otra comunicación cliente/servidor segura que se produce en Internet. Un cliente y un servidor se comunican de forma segura en Internet mediante la [Seguridad de la capa de transporte (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security). TLS se ha creado mediante construcciones de [Infraestructura de clave pública (PKI)](https://en.wikipedia.org/wiki/Public_key_infrastructure) estándar llamadas certificados. TLS es una especificación bastante compleja que aborda una amplia gama de temas relacionados con la protección de dos puntos de conexión. En esta sección se resumen los conceptos relevantes para que pueda conectar de forma segura los dispositivos a una puerta de enlace IoT Edge.

Cuando un cliente se conecta a un servidor, el servidor presenta una cadena de certificados llamada *cadena de certificados de servidor*. Una cadena de certificados normalmente consta de un certificado de entidad de certificación (CA) raíz, uno o varios certificados de entidad de certificación intermedios y, finalmente, el propio certificado del servidor. Un cliente establece la confianza con un servidor mediante la comprobación criptográfica de la cadena de certificados de servidor completa. Esta validación del cliente de la cadena de certificados de servidor se llama *validación de la cadena del servidor*. El cliente desafía al servidor para demostrar la posesión de la clave privada asociada al certificado de servidor en un proceso denominado *prueba de posesión*. La combinación de la validación de la cadena del servidor y la prueba de posesión se denomina *autenticación de servidor*. Para validar una cadena de certificados de servidor, un cliente necesita una copia del certificado de la entidad de certificación raíz que se usó para crear (o emitir) el certificado del servidor. Normalmente, al conectarse a sitios web, un explorador viene preconfigurado con certificados de entidad de certificación usados habitualmente para que el cliente tenga un proceso de conexión directo.

Cuando un dispositivo se conecta a Azure IoT Hub, el dispositivo es el cliente y el servicio en la nube de IoT Hub es el servidor. El servicio en la nube de IoT Hub está respaldado por un certificado de entidad de certificación raíz llamado **Baltimore CyberTrust Root**, que es ampliamente utilizado y está disponible públicamente. Puesto que el certificado de entidad de certificación de IoT Hub ya está instalado en la mayoría de los dispositivos, muchas implementaciones de TLS (OpenSSL, Schannel, LibreSSL) lo utilizan automáticamente durante la validación de certificados de servidor. Sin embargo, un dispositivo que puede conectarse correctamente a IoT Hub puede tener problemas al intentar conectarse a una puerta de enlace IoT Edge.

Cuando un dispositivo se conecta a una puerta de enlace IoT Edge, el dispositivo de bajada es el cliente y el dispositivo de puerta de enlace es el servidor. Azure IoT Edge permite a los operadores (o usuarios) crear cadenas de certificados de puerta de enlace como consideren oportuno. El operador puede usar un certificado de entidad de certificación pública, como Baltimore o usar un certificado de entidad de certificación de raíz autofirmado (o interno). Los certificados de entidad de certificación pública a menudo tienen un costo asociado, por lo que se utilizan normalmente en escenarios de producción. Los certificados de entidad de certificación autofirmados se prefieren para desarrollo y pruebas. Los artículos de configuración de la puerta de enlace transparente indicados en la introducción usan certificados de entidad de certificación raíz autofirmados.

Cuando se usa un certificado de entidad de certificación raíz autofirmado para una puerta de enlace IoT Edge, este se debe instalar o proporcionar a todos los dispositivos de bajada que intentan conectarse a la puerta de enlace.

![Configuración del certificado de puerta de enlace](./media/how-to-create-transparent-gateway/gateway-setup.png)

Para más información acerca de los certificados de IoT Edge y algunas implicaciones de producción, consulte [Detalles de uso de los certificados de IoT Edge](iot-edge-certs.md).

## <a name="provide-the-root-ca-certificate"></a>Inclusión del certificado de entidad de certificación raíz

Para comprobar los certificados del dispositivo de puerta de enlace, el dispositivo de bajada necesita su propia copia del certificado de entidad de certificación raíz. Si ha usado los scripts proporcionados en el repositorio de Git de IoT Edge para crear certificados de prueba, el certificado de entidad de certificación raíz se denomina **azure-iot-test-only.root.ca.cert.pem**. Si no lo ha hecho ya como parte de los otros pasos de preparación del dispositivo de bajada, mueva este archivo de certificado a cualquier directorio de este dispositivo. Para ello, puede usar un servicio como [Azure Key Vault](https://docs.microsoft.com/azure/key-vault) o una función como [Protocolo de copia segura](https://www.ssh.com/ssh/scp/).

## <a name="install-certificates-in-the-os"></a>Instalación de certificados en el sistema operativo

La instalación del certificado de entidad de certificación raíz en el almacén de certificados del sistema operativo generalmente permite que la mayoría de las aplicaciones lo usen. Hay algunas excepciones, como las aplicaciones de NodeJS, que no utilizan el almacén de certificados del sistema operativo y utilizan en su lugar el almacén de certificados interno del entorno de ejecución de Node. Si no puede instalar el certificado en el nivel de sistema operativo, continúe en [Uso de certificados con los SDK de Azure IoT](#use-certificates-with-azure-iot-sdks).

### <a name="ubuntu"></a>Ubuntu

Los siguientes comandos son un ejemplo de cómo instalar un certificado de entidad de certificación en un host con Ubuntu. En este ejemplo se da por supuesto que usa el certificado **azure-iot-test-only.root.ca.cert.pem** de los artículos de requisitos previos y que ha copiado el certificado en una ubicación del dispositivo de bajada.

```bash
sudo cp <path>/azure-iot-test-only.root.ca.cert.pem /usr/local/share/ca-certificates/azure-iot-test-only.root.ca.cert.pem.crt
sudo update-ca-certificates
```

Verá un mensaje que dice "Updating certificates in /etc/ssl/certs... 1 added, 0 removed; done." (Actualizando certificados en /etc/ssl/certs...1 agregado, 0 quitados, listo)

### <a name="windows"></a>Windows

Los siguientes pasos son un ejemplo de cómo instalar un certificado de entidad de certificación en un host con Windows. En este ejemplo se da por supuesto que usa el certificado **azure-iot-test-only.root.ca.cert.pem** de los artículos de requisitos previos y que ha copiado el certificado en una ubicación del dispositivo de bajada.

Puede instalar certificados con [Import-Certificate](https://docs.microsoft.com/powershell/module/pkiclient/import-certificate?view=win10-ps) de PowerShell como administrador:

```powershell
import-certificate  <file path>\azure-iot-test-only.root.ca.cert.pem -certstorelocation cert:\LocalMachine\root
```

También puede instalar certificados mediante la utilidad **certlm**:

1. En el menú Inicio, busque y seleccione **Administrar certificados de equipo**. Se abre una utilidad llamada **certlm**.
2. Vaya a **Certificados: equipo local** > **Entidades de certificación raíz de confianza**.
3. Haga clic con el botón derecho en **Certificados** y seleccione **Todas las tareas** > **Importar**. Se debe iniciar el asistente para importación de certificados.
4. Siga los pasos tal como se indica e importe el archivo del certificado `<path>/azure-iot-test-only.root.ca.cert.pem`. Cuando se haya completado, verá el mensaje "Importación correcta".

También puede instalar certificados mediante programación con las API de .NET, como se muestra más adelante en el ejemplo de .NET de este artículo.

Normalmente, las aplicaciones utilizan la pila TLS proporcionada por Windows llamada [Schannel](https://docs.microsoft.com/windows/desktop/com/schannel) para conectarse de forma segura mediante TLS. Schannel *requiere* que todos los certificados se instalen en el almacén de certificados de Windows antes de intentar establecer una conexión TLS.

## <a name="use-certificates-with-azure-iot-sdks"></a>Uso de certificados con los SDK de Azure IoT

Esta sección describe cómo conectar los SDK de Azure IoT a un dispositivo IoT Edge con aplicaciones de ejemplo simples. El objetivo de todos los ejemplos es conectar el cliente de dispositivo y enviar mensajes de telemetría a la puerta de enlace, cerrar la conexión y salir.

Debe tener dos cosas listas antes de usar los ejemplos de nivel de aplicación:

* La cadena de conexión de IoT Hub del dispositivo de bajada modificada para que apunte al dispositivo de puerta de enlace y los certificados necesarios para autenticar el dispositivo de bajada en IoT Hub. Para más información, consulte [Autenticación de un dispositivo de bajada en Azure IoT Hub](how-to-authenticate-downstream-device.md).

* La ruta de acceso completa del certificado de entidad de certificación raíz que ha copiado y guardado en algún lugar del dispositivo de bajada.

    Por ejemplo, `<path>/azure-iot-test-only.root.ca.cert.pem`.

### <a name="nodejs"></a>NodeJS

Esta sección proporciona una aplicación de ejemplo para conectar un cliente de dispositivo de NodeJS de Azure IoT a una puerta de enlace IoT Edge. En el caso de las aplicaciones de NodeJS, debe instalar el certificado de entidad de certificación raíz en el nivel de aplicación, tal como se muestra aquí. Las aplicaciones de NodeJS no usan el almacén de certificados del sistema.

1. Puede obtener el ejemplo para **edge_downstream_device.js** desde el [repositorio de ejemplos del SDK de dispositivo IoT de Azure para Node.js](https://github.com/Azure/azure-iot-sdk-node/tree/master/device/samples).
2. Asegúrese de que tiene todos los requisitos previos para ejecutar el ejemplo; para ello, revise el archivo **readme.md**.
3. En el archivo edge_downstream_device.js, actualice las variables **connectionString** y **edge_ca_cert_path**.
4. Consulte la documentación del SDK para obtener instrucciones sobre cómo ejecutar el ejemplo en el dispositivo.

Para entender el ejemplo que se está ejecutando, el siguiente fragmento de código contiene el modo en el que el SDK de cliente lee el archivo de certificado y lo usa para establecer una conexión TLS segura:

```javascript
// Provide the Azure IoT device client via setOptions with the X509
// Edge root CA certificate that was used to setup the Edge runtime
var options = {
    ca : fs.readFileSync(edge_ca_cert_path, 'utf-8'),
};
```

### <a name="net"></a>.NET

Esta sección presenta una aplicación de ejemplo para conectar un cliente de dispositivo de .NET de Azure IoT a una puerta de enlace IoT Edge. Sin embargo, las aplicaciones .NET pueden usar automáticamente los certificados instalados en el almacén de certificados del sistema en los hosts con Linux y Windows.

1. Puede obtener el ejemplo para **EdgeDownstreamDevice** desde la [carpeta de ejemplos de .NET de IoT Edge ](https://github.com/Azure/iotedge/tree/master/samples/dotnet/EdgeDownstreamDevice).
2. Asegúrese de que tiene todos los requisitos previos para ejecutar el ejemplo; para ello, revise el archivo **readme.md**.
3. En el archivo **Propiedades / launchSettings.json**, actualice las variables **DEVICE_CONNECTION_STRING** y **CA_CERTIFICATE_PATH**. Si desea utilizar el certificado instalado en el almacén de certificados de confianza en el sistema host, deje en blanco esta variable.
4. Consulte la documentación del SDK para obtener instrucciones sobre cómo ejecutar el ejemplo en el dispositivo.

Para instalar mediante programación un certificado de confianza en el almacén de certificados mediante una aplicación de .NET, consulte la función **InstallCACert()** del archivo **EdgeDownstreamDevice / Program.cs**. Esta operación es idempotente, por lo que se puede ejecutar varias veces con los mismos valores sin ningún efecto adicional.

### <a name="c"></a>C

Esta sección presenta una aplicación de ejemplo para conectar un cliente de dispositivo de Azure IoT para C a una puerta de enlace IoT Edge. El SDK para C puede funcionar con muchas bibliotecas TLS, incluidas OpenSSL, WolfSSL y Schannel. Para más información, consulte el [SDK de Azure IoT para C](https://github.com/Azure/azure-iot-sdk-c).

1. Puede obtener la aplicación **iotedge_downstream_device_sample** en los [Ejemplos del SDK de dispositivo IoT de Azure para C](https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples).
2. Asegúrese de que tiene todos los requisitos previos para ejecutar el ejemplo; para ello, revise el archivo **readme.md**.
3. En el archivo iotedge_downstream_device_sample.js, actualice las variables **connectionString** y **edge_ca_cert_path**.
4. Consulte la documentación del SDK para obtener instrucciones sobre cómo ejecutar el ejemplo en el dispositivo.

El SDK de dispositivo IoT de Azure para C proporciona una opción para registrar un certificado de entidad de certificación al configurar el cliente. Esta operación no instala el certificado en ningún lugar y en su lugar usa un formato de cadena del certificado en memoria. Al establecer una conexión, se proporciona el certificado guardado a la pila TLS subyacente.

```C
(void)IoTHubDeviceClient_SetOption(device_handle, OPTION_TRUSTED_CERT, cert_string);
```

En hosts con Windows, si no utiliza OpenSSL ni otra biblioteca de TLS, el SDK utiliza de manera predeterminada Schannel. Para que Schannel funcione, se debe instalar el certificado de entidad de certificación raíz de IoT Edge en el almacén de certificados de Windows, no se debe establecer mediante la operación `IoTHubDeviceClient_SetOption`.

### <a name="java"></a>Java

Esta sección presenta una aplicación de ejemplo para conectar un cliente de dispositivo de Java de Azure IoT a una puerta de enlace IoT Edge.

1. Puede obtener el ejemplo **Send-event** en los [Ejemplos del SDK de dispositivo IoT de Azure para Java](https://github.com/Azure/azure-iot-sdk-java/tree/master/device/iot-device-samples).
2. Asegúrese de que tiene todos los requisitos previos para ejecutar el ejemplo; para ello, revise el archivo **readme.md**.
3. Consulte la documentación del SDK para obtener instrucciones sobre cómo ejecutar el ejemplo en el dispositivo.

### <a name="python"></a>Python

Esta sección presenta una aplicación de ejemplo para conectar un cliente de dispositivo de Python de Azure IoT a una puerta de enlace IoT Edge.

1. El ejemplo de **send_message_downstream** se obtiene de los [Ejemplos del SDK de dispositivo IoT de Azure para Python](https://github.com/Azure/azure-iot-sdk-python/tree/master/azure-iot-device/samples/async-edge-scenarios).
2. Establezca las variables de entorno `IOTHUB_DEVICE_CONNECTION_STRING` y `IOTEDGE_ROOT_CA_CERT_PATH` como se especifica en los comentarios del script de Python.
3. Consulte la documentación del SDK para obtener instrucciones adicionales sobre cómo ejecutar el ejemplo en el dispositivo.

## <a name="test-the-gateway-connection"></a>Prueba de la conexión de puerta de enlace

Use este comando de ejemplo para probar que el dispositivo de nivel inferior puede conectarse al dispositivo de puerta de enlace:

```cmd/sh
openssl s_client -connect mygateway.contoso.com:8883 -CAfile <CERTDIR>/certs/azure-iot-test-only.root.ca.cert.pem -showcerts
```

Este comando prueba las conexiones a través de MQTTS (puerto 8883). Si usa un protocolo diferente, ajuste el comando según sea necesario para AMQPS (5671) o HTTPS (433).

La salida de este comando puede ser larga, incluida la información sobre todos los certificados de la cadena. Si la conexión se realiza correctamente, verá una línea como `Verification: OK` o `Verify return code: 0 (ok)`.

![Comprobación de una conexión de puerta de enlace](./media/how-to-connect-downstream-device/verification-ok.png)

## <a name="troubleshoot-the-gateway-connection"></a>Solución de problemas de la conexión de puerta de enlace

Si el dispositivo de hoja tiene una conexión intermitente con su dispositivo de puerta de enlace, intente seguir estos pasos para resolverlo.

1. ¿El nombre de host de la puerta de enlace en la cadena de conexión es el mismo que el valor de nombre de host en el archivo config.yaml de IoT Edge del dispositivo de puerta de enlace?
2. ¿Se puede resolver el nombre de host de la puerta de enlace en una dirección IP? Puede resolver conexiones intermitentes mediante el uso de DNS o bien mediante la incorporación de una entrada de archivo de host en el dispositivo de hoja.
3. ¿Están abiertos los puertos de comunicación en el firewall? Debe ser posible la comunicación basada en el protocolo usado (MQTTS:8883/AMQPS:5671/HTTPS:433) entre el dispositivo de bajada y el dispositivo IoT Edge transparente.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo puede extender IoT Edge las [funcionalidades sin conexión](offline-capabilities.md) en los dispositivos de bajada.
