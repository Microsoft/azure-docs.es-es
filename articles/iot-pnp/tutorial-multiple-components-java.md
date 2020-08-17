---
title: Conexión del código del dispositivo del componente de Java de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub | Microsoft Docs
description: Compile y ejecute el código de dispositivo de Java de ejemplo de la versión preliminar de IoT Plug and Play que usa varios componentes y se conecta a un centro de IoT. Use la herramienta Azure IoT Explorer para ver la información enviada por el dispositivo al centro.
author: ericmitt
ms.author: ericmitt
ms.date: 07/14/2020
ms.topic: tutorial
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 1d16d8c54939c4f659b6a1530e2d360b957a09ad
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87351940"
---
# <a name="tutorial-connect-a-sample-iot-plug-and-play-preview-multiple-component-device-application-to-iot-hub-java"></a>Tutorial: Conexión de una aplicación de dispositivo de varios componentes de ejemplo de la versión preliminar de IoT Plug and Play a IoT Hub (Java)

[!INCLUDE [iot-pnp-tutorials-device-selector.md](../../includes/iot-pnp-tutorials-device-selector.md)]

En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con varios componentes, cómo conectarla al centro de IoT y cómo usar la CLI de Azure para ver los datos de telemetría que envía. La aplicación de ejemplo se escribe en Java y se incluye en el SDK de dispositivo IoT de Azure para Java. Un generador de soluciones puede usar la CLI de Azure para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

En este tutorial se muestra cómo compilar una aplicación de dispositivo IoT Plug and Play de ejemplo con componentes e interfaz raíz, cómo conectarla a un centro de IoT y cómo usar la herramienta Azure IoT Explorer para ver la información que envía al centro. La aplicación de ejemplo se escribe en Java y se incluye en el SDK de dispositivo IoT de Azure para Java. Un generador de soluciones puede usar la herramienta Azure IoT Explorer para comprender las funcionalidades de cualquier dispositivo IoT Plug and Play sin necesidad de ver nada de código del dispositivo.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial en Windows, instale el siguiente software en el entorno Windows local:

* Java SE Development Kit 8. En el artículo [Soporte técnico de Java a largo plazo para Azure y Azure Stack](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable), en la sección **Soporte técnico a largo plazo**, seleccione **Java 8**.
* [Apache Maven 3](https://maven.apache.org/download.cgi).

### <a name="azure-iot-explorer"></a>Azure IoT Explorer

Para interactuar con el dispositivo de ejemplo en la segunda parte de esta guía de inicio rápido, use la herramienta **Azure IoT Explorer**. [Descargue e instale la versión de Azure IoT Explorer más reciente](./howto-use-iot-explorer.md) para su sistema operativo.

[!INCLUDE [iot-pnp-prepare-iot-hub.md](../../includes/iot-pnp-prepare-iot-hub.md)]

Ejecute el siguiente comando para obtener la _cadena de conexión del centro de IoT_ para su centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub show-connection-string --hub-name <YourIoTHubName> --output table
```

> [!TIP]
> También puede usar la herramienta Azure IoT Explorer para buscar la cadena de conexión del centro de IoT.

Ejecute el siguiente comando para obtener la _cadena de conexión del dispositivo_ que ha agregado al centro. Tome nota de esta cadena de conexión, ya que la usará más adelante en este inicio rápido:

```azurecli-interactive
az iot hub device-identity show-connection-string --hub-name <YourIoTHubName> --device-id <YourDeviceID> --output table
```

[!INCLUDE [iot-pnp-download-models.md](../../includes/iot-pnp-download-models.md)]

## <a name="download-the-code"></a>Descargar el código

En este tutorial, se prepara un entorno de desarrollo que se puede usar para clonar y compilar el SDK del dispositivo de Azure IoT Hub para Java.

Abra un símbolo del sistema en el directorio que prefiera. Ejecute el siguiente comando para clonar el repositorio de GitHub de las [bibliotecas y los SDK de Azure IoT para Java](https://github.com/Azure/azure-iot-sdk-java) en esta ubicación:

```cmd
git clone https://github.com/Azure/azure-iot-sdk-java.git
```

Esta operación puede tardar varios minutos en completarse.

## <a name="build-the-code"></a>Compilación del código

En Windows, vaya a la carpeta raíz del repositorio del SDK de Java clonado. A continuación, vaya a la carpeta *\device\iot-device-samples\pnp-device-sample\temerature-controller-device-sample*.

Ejecute el siguiente comando para compilar la aplicación de ejemplo:

```java
mvn clean package
```

## <a name="run-the-device-sample"></a>Ejecución del ejemplo de dispositivo

Cree una variable de entorno denominada **IOTHUB_DEVICE_CONNECTION_STRING** para almacenar la cadena de conexión del dispositivo que anotó anteriormente.

Para ejecutar la aplicación de ejemplo, utilice el comando siguiente:

```java
mvn exec:java -Dexec.mainClass="samples.com.microsoft.azure.sdk.iot.device.TemperatureController"
```

El dispositivo ya está listo para recibir comandos y actualizaciones de propiedades, y ha empezado a enviar datos de telemetría al centro. Siga ejecutando el ejemplo a medida que complete los pasos siguientes.

## <a name="use-azure-iot-explorer-to-validate-the-code"></a>Uso de Azure IoT Explorer para validar el código

Una vez iniciado el ejemplo de cliente de dispositivo, compruebe que funciona con la herramienta Azure IoT Explorer.

[!INCLUDE [iot-pnp-iot-explorer.md](../../includes/iot-pnp-iot-explorer.md)]

## <a name="review-the-code"></a>Revisión del código

En este ejemplo se implementa un dispositivo controlador de temperatura IoT Plug and Play. El modelo que implementa este ejemplo usa [varios componentes](concepts-components.md). El [archivo de modelo del lenguaje de definición de gemelos digitales (DTDL) del dispositivo de temperatura](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/samples/TemperatureController.json) define los datos de telemetría, las propiedades y los comandos que implementa el dispositivo.

El código del dispositivo usa la clase `DeviceClient` estándar para conectarse al centro de IoT. El dispositivo envía el identificador de modelo del modelo DTDL que implementa en la solicitud de conexión. Un dispositivo que envía un identificador de modelo es un dispositivo IoT Plug and Play:

```java
private static void initializeDeviceClient() throws URISyntaxException, IOException {
    ClientOptions options = new ClientOptions();
    options.setModelId(MODEL_ID);
    deviceClient = new DeviceClient(deviceConnectionString, protocol, options);

    deviceClient.registerConnectionStatusChangeCallback((status, statusChangeReason, throwable, callbackContext) -> {
        log.debug("Connection status change registered: status={}, reason={}", status, statusChangeReason);

        if (throwable != null) {
            log.debug("The connection status change was caused by the following Throwable: {}", throwable.getMessage());
            throwable.printStackTrace();
        }
    }, deviceClient);

    deviceClient.open();
}
```

El identificador del modelo se almacena en el código tal y como se muestra en el siguiente fragmento de código:

```java
private static final String MODEL_ID = "dtmi:com:example:Thermostat;1";
```

Una vez que el dispositivo se conecta al centro de IoT, el código registra los controladores de comandos.

```java
deviceClient.subscribeToDeviceMethod(new MethodCallback(), null, new MethodIotHubEventCallback(), null);
```

Existen controladores independientes para las actualizaciones de propiedades deseadas en los dos componentes de termostato:

```java
deviceClient.startDeviceTwin(new TwinIotHubEventCallback(), null, new GenericPropertyUpdateCallback(), null);
Map<Property, Pair<TwinPropertyCallBack, Object>> desiredPropertyUpdateCallback = Stream.of(
        new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
                new Property(THERMOSTAT_1, null),
                new Pair<>(new TargetTemperatureUpdateCallback(), THERMOSTAT_1)),
        new AbstractMap.SimpleEntry<Property, Pair<TwinPropertyCallBack, Object>>(
                new Property(THERMOSTAT_2, null),
                new Pair<>(new TargetTemperatureUpdateCallback(), THERMOSTAT_2))
).collect(Collectors.toMap(AbstractMap.SimpleEntry::getKey, AbstractMap.SimpleEntry::getValue));

deviceClient.subscribeToTwinDesiredProperties(desiredPropertyUpdateCallback);
```

El código de ejemplo envía los datos de telemetría de cada componente de termostato:

```java
sendTemperatureReading(THERMOSTAT_1);
sendTemperatureReading(THERMOSTAT_2);
```

El método `sendTemperatureReading` usa la clase `PnpHhelper` para crear mensajes para cada componente:

```java
Message message = PnpHelper.createIotHubMessageUtf8(telemetryName, currentTemperature, componentName);
```

La clase `PnpHelper` contiene otros métodos de ejemplo que se pueden usar con un modelo de varios componentes.

Use la herramienta Azure IoT Explorer para ver los datos de telemetría y las propiedades de los dos componentes de termostato:

:::image type="content" source="media/tutorial-multiple-components-java/multiple-component.png" alt-text="Dispositivo de varios componentes en Azure IoT Explorer":::

También puede usar la herramienta Azure IoT Explorer para llamar a comandos en cualquiera de los dos componentes de termostato o en la interfaz raíz.

[!INCLUDE [iot-pnp-clean-resources.md](../../includes/iot-pnp-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a conectar un dispositivo IoT Plug and Play con componentes a un centro de IoT. Para obtener más información sobre los modelos de dispositivo IoT Plug and Play, consulte:

> [!div class="nextstepaction"]
> [Guía para desarrolladores de modelado de IoT Plug and Play (versión preliminar)](concepts-developer-guide.md)
