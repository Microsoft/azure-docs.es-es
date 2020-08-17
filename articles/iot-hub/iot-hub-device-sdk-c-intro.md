---
title: SDK de dispositivo IoT de Azure para C | Microsoft Docs
description: Introducción al SDK de dispositivo IoT de Azure para C e información sobre cómo crear aplicaciones para dispositivos que se comunican con un centro de IoT Hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.devlang: c
ms.topic: conceptual
ms.date: 05/17/2019
ms.author: robinsh
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
ms.openlocfilehash: 3acac0c85aa6f3e3725dcf54a646f8ac16250e24
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87324053"
---
# <a name="azure-iot-device-sdk-for-c"></a>SDK de dispositivo IoT de Azure para C

**SDK de dispositivo IoT de Azure para C** es un conjunto de bibliotecas diseñadas para simplificar el proceso de envío y recepción de mensajes desde el servicio **Azure IoT Hub**. Hay distintas variantes del SDK, cada una dirigida a una plataforma concreta, pero este artículo describe el **SDK de dispositivo IoT de Azure para C**.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

El SDK de dispositivo IoT de Azure para C está escrito en ANSI C (C99) para maximizar su portabilidad. Esta característica hace que las bibliotecas sean idóneas para operar en varias plataformas y dispositivos, especialmente si la prioridad es minimizar la superficie de la memoria y del disco.

Existe una amplia gama de plataformas en las que se ha probado el SDK (consulte [Catálogo de dispositivos de Azure Certified for IoT](https://catalog.azureiotsolutions.com/) para obtener más información). Aunque este artículo incluye tutoriales con código de ejemplo que se ejecuta en la plataforma Windows, el código que describe en él es idéntico en todas las plataformas compatibles.

El siguiente vídeo contiene información general sobre el SDK para C de Azure IoT:

>[!VIDEO https://channel9.msdn.com/Shows/Internet-of-Things-Show/Azure-IoT-C-SDK-insights/Player]

En este artículo se presenta la arquitectura del SDK de dispositivo IoT de Azure para C. En él se muestra cómo inicializar la biblioteca de dispositivos, enviar datos a IoT Hub y recibir mensajes de dicho servicio. La información de este artículo debería ser suficiente para comenzar a usar el SDK, pero también se proporcionan referencias a información adicional sobre las bibliotecas.

## <a name="sdk-architecture"></a>Arquitectura del SDK

Puede encontrar el [**SDK de dispositivo IoT de Azure para C**](https://github.com/Azure/azure-iot-sdk-c) en el repositorio de GitHub y ver los detalles de la API en la [referencia de la API de C](https://docs.microsoft.com/azure/iot-hub/iot-c-sdk-ref/).

La versión más reciente de las bibliotecas se puede encontrar en la rama **master** del repositorio:

  ![Captura de pantalla de la rama principal del repositorio](./media/iot-hub-device-sdk-c-intro/RepoMasterBranch.png)

* La implementación básica del SDK está en la carpeta **iothub\_client** que contiene la implementación de la capa inferior de la API del SDK: la biblioteca **IoTHubClient**. La biblioteca **IoTHubClient** contiene las API que implementan la mensajería sin formato tanto para enviar mensajes a IoT Hub como para recibirlos de él. Quien use esta biblioteca es responsable de la implementación de la serialización de mensajes, pero otros detalles de la comunicación con IoT Hub se controlan automáticamente.

* La carpeta **serializer** contiene funciones auxiliares y ejemplos que muestran cómo serializar los datos antes de enviarlos a Azure IoT Hub mediante la biblioteca de cliente. El uso del serializador no es obligatorio y se proporciona como un recurso que puede resultar práctico. Para usar la biblioteca **serializer**, defina un modelo que especifique los datos que se envían a IoT Hub y los mensajes que se esperan recibir de él. Una vez definido el modelo, el SDK proporciona una superficie de API que permite trabajar fácilmente con mensajes del dispositivo a la nube y de la nube al dispositivo sin tener que preocuparse por los detalles de la serialización. La biblioteca depende de otras bibliotecas de código abierto que implementan el transporte mediante protocolos como MQTT y AMQP.

* La biblioteca **IoTHubClient** depende de otras bibliotecas de código abierto:

  * La biblioteca [Azure C shared utility](https://github.com/Azure/azure-c-shared-utility), que proporciona una funcionalidad común para las tareas básicas (como cadenas, manipulación de listas y E/S) necesarias en varios SDK de C relacionados con Azure.

  * La biblioteca [Azure uAMQP](https://github.com/Azure/azure-uamqp-c), que es una implementación del lado cliente de AMQP optimizada para los dispositivos de restricción de recursos.

  * La biblioteca [Azure uMQTT](https://github.com/Azure/azure-umqtt-c), que es una biblioteca de uso general que implementa el protocolo MQTT y que está optimizada para los dispositivos de restricción de recursos.

El uso de estas bibliotecas se entiende más fácilmente si se examina el código de ejemplo. Las siguientes secciones le guiarán a través de varias aplicaciones de ejemplo que se incluyen en el SDK. Este tutorial le permitirá hacerse una idea muy aproximada de las distintas funcionalidades de las capas arquitectónicas del SDK y proporcionará una introducción al funcionamiento de las API.

## <a name="before-you-run-the-samples"></a>Antes de ejecutar los ejemplos

Antes de ejecutar los ejemplos en el SDK de dispositivo IoT de Azure para C, debe [crear una instancia del servicio IoT Hub](iot-hub-create-through-portal.md) en su suscripción de Azure. Después complete las siguientes tareas:

* Preparación del entorno de desarrollo
* Obtención de las credenciales del dispositivo.

### <a name="prepare-your-development-environment"></a>Preparación del entorno de desarrollo

Se proporcionan paquetes para plataformas comunes (como NuGet para Windows o apt_get para Debian y Ubuntu) y los ejemplos usan estos paquetes cuando estén disponibles. En algunos casos, es preciso compilar el SDK para el dispositivo. Si necesita compilar el SDK, consulte [Prepare your development environment](https://github.com/Azure/azure-iot-sdk-c/blob/master/doc/devbox_setup.md) (Preparación de un entorno de desarrollo) en el repositorio de GitHub.

Para obtener el código de la aplicación de ejemplo, descargue una copia del SDK de GitHub. Obtenga su copia del código fuente de la rama **master** del [repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-c).


### <a name="obtain-the-device-credentials"></a>Obtención de las credenciales del dispositivo

Ahora que tiene el código fuente de ejemplo, lo siguiente que debe hacer es obtener un conjunto de credenciales del dispositivo. Para que un dispositivo pueda acceder a un centro de IoT, primero debe agregar el dispositivo al registro de identidad del centro de IoT. Al agregar el dispositivo, se obtiene un conjunto de credenciales del dispositivo que se necesitan para que el dispositivo se pueda conectar al servicio IoT Hub. Las aplicaciones de ejemplo de la siguiente sección esperan que estas credenciales tengan la forma de una **cadena de conexión de dispositivo**.

Hay varias herramientas de código abierto que le ayudan a administrar IoT Hub.

* Una aplicación Windows llamada [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer).

* Una extensión de Visual Studio Code multiplataforma llamada [Azure IoT Tools](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools).

* Una herramienta de la CLI de Python multiplataforma denominada [extensión de IoT para la CLI de Azure](https://github.com/Azure/azure-iot-cli-extension).

En este tutorial se utiliza la herramienta gráfica *device explorer*. Puede usar *Azure IoT Tools para VS Code* si desarrolla en VS Code. También puede usar *extensión de IoT para la CLI de Azure 2.0* si prefiere usar una herramienta de la CLI.

La herramienta device explorer usa las bibliotecas del servicio Azure IoT para realizar varias funciones en IoT Hub, entre las que se incluye la adición de dispositivos. Si usa la herramienta device explorer para agregar un dispositivo, obtiene una cadena de conexión para el dispositivo. Esta cadena de conexión se necesita para ejecutar las aplicaciones de ejemplo.

Si no está familiarizado con la herramienta device explorer, el siguiente procedimiento describe cómo usarla para agregar un dispositivo y obtener la cadena de conexión del dispositivo.

1. Para instalar la herramienta device explorer, consulte [How to use Device Explorer for IoT Hub devices](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/) (Uso de la herramienta device explorer para dispositivos de IoT Hub).

1. Al ejecutar el programa se ve esta interfaz:

   ![Captura de pantalla de Device Explorer Twin](./media/iot-hub-device-sdk-c-intro/DeviceExplorerTwinConfigTab.png)

1. Escriba su **cadena de conexión del IoT Hub** en el primer campo y haga clic en **Update** (Actualizar). Este paso configura la herramienta para que pueda comunicarse con IoT Hub. 

La **Cadena de conexión** puede encontrarse en **Servicio IoT Hub** > **Configuración** > **Directiva de acceso compartido** > **iothubowner**.

1. Cuando se configura la cadena de conexión de IoT Hub, haga clic en la pestaña **Management** (Administración):

   ![Captura de pantalla de Management de Device Explorer Twin](./media/iot-hub-device-sdk-c-intro/DeviceExplorerTwinManagementTab.png)

En esta pestaña se administran los dispositivos registrados en IoT Hub.

1. Para crear un dispositivo, haga clic en el botón **Create** (Crear). Se muestra un cuadro de diálogo con un conjunto de claves (principal y secundaria) previamente rellenadas. Escriba un valor en **Device ID** (Id. de dispositivo) y haga clic en **Create** (Crear).

   ![Captura de pantalla de Create Device](./media/iot-hub-device-sdk-c-intro/CreateDevice.png)

1. Cuando se crea el dispositivo, la lista de dispositivos se actualiza con todos los dispositivos registrados, incluido el que acaba de crear. Si hace clic con el botón derecho en el dispositivo nuevo, verá este menú:

   ![Resultado de hacer clic con el botón derecho en Device Explorer Twin](./media/iot-hub-device-sdk-c-intro/DeviceExplorerTwinManagementTab_RightClick.png)

1. Si elige la opción **Copy connection string for selected device** (Copiar cadena de conexión de dispositivo seleccionado), la cadena de conexión del dispositivo se copia en el Portapapeles. Mantenga una copia de la cadena de conexión de dispositivo, ya que se necesita al ejecutar las aplicaciones de ejemplo que se describen en las secciones siguientes.

Cuando haya completados los pasos anteriores, estará listo para empezar a ejecutar código. La mayoría de los ejemplos tienen una constante en la parte superior del archivo de código fuente principal, que permite especificar una cadena de conexión. Por ejemplo, la línea correspondiente de la aplicación **iothub_client\_samples\_iothub_convenience_sample** aparece de la siguiente manera.

```c
static const char* connectionString = "[device connection string]";
```

## <a name="use-the-iothubclient-library"></a>Uso de la biblioteca IoTHubClient

Dentro de la carpeta **iothub\_client**, en el repositorio [azure-iot-sdks](https://github.com/azure/azure-iot-sdk-c), hay una carpeta **samples** que contiene una aplicación denominada **iothub\_client\_sample\_mqtt**.

La versión de Windows para la aplicación **iothub_client\_samples\_iothub_convenience_sample** incluye la siguiente solución de Visual Studio:

  ![Explorador de soluciones de Visual Studio](./media/iot-hub-device-sdk-c-intro/iothub-client-sample-mqtt.png)

> [!NOTE]
> Si Visual Studio le solicita que redestine el proyecto a la última versión, acepte la solicitud.

Esta solución contiene un proyecto. En esta solución hay cuatro paquetes de NuGet instalados:

* Microsoft.Azure.C.SharedUtility
* Microsoft.Azure.IoTHub.MqttTransport
* Microsoft.Azure.IoTHub.IoTHubClient
* Microsoft.Azure.umqtt

El paquete **Microsoft.Azure.C.SharedUtility** siempre es necesario al trabajar con el SDK. Este ejemplo usa el protocolo MQTT, por lo que es necesario incluir los paquetes **Microsoft.Azure.umqtt** y **Microsoft.Azure.IoTHub.MqttTransport** (hay paquetes equivalentes para AMQP y HTTPS). Como en el ejemplo se usa la biblioteca **IoTHubClient**, se debe incluir también el paquete **Microsoft.Azure.IoTHub.IoTHubClient** en la solución.

La implementación de la aplicación de ejemplo en se encuentra en el archivo de código fuente **iothub_client\_samples\_iothub_convenience_sample**.

En los siguientes pasos se usa esta aplicación de ejemplo para guiarle por los requisitos necesarios para usar la biblioteca **IoTHubClient**.

### <a name="initialize-the-library"></a>Inicialización de la biblioteca

> [!NOTE]
> Antes de empezar a trabajar con las bibliotecas, puede que necesite realizar alguna tarea de inicialización específica de la plataforma. Por ejemplo, si tiene previsto usar AMQP en Linux, debe inicializar la biblioteca OpenSSL. Los ejemplos del [repositorio de GitHub](https://github.com/Azure/azure-iot-sdk-c) llaman a la función **platform\_init** de la utilidad cuando el cliente se inicia y llaman a la función **platform\_deinit** antes de salir. Estas funciones se declaran en el archivo de encabezado platform.h. Examine las definiciones de estas funciones para la plataforma de destino del [repositorio](https://github.com/Azure/azure-iot-sdk-c) para determinar si necesita incluir código de inicialización específico de la plataforma en el cliente.

Para empezar a trabajar con las bibliotecas, primero asigne un identificador de cliente de IoT Hub:

```c
if ((iotHubClientHandle = 
  IoTHubClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol)) == NULL)
{
    (void)printf("ERROR: iotHubClientHandle is NULL!\r\n");
}
else
{
    ...
```

Pase a esta función una copia de la cadena de conexión del dispositivo que obtuvo de la herramienta device explorer. Designe también el protocolo de comunicaciones que va a usar. En este ejemplo se usa MQTT, pero también se pueden usar AMQP y HTTPS.

Cuando tenga un valor **IOTHUB\_CLIENT\_HANDLE** válido, puede empezar a llamar a las API para enviar mensajes a IoT Hub y recibirlos de este.

### <a name="send-messages"></a>Envío de mensajes

La aplicación de ejemplo configura un bucle para enviar mensajes a IoT Hub. El siguiente fragmento de código:

- Crea un mensaje.
- Agrega una propiedad al mensaje.
- Envía un mensaje.

Primero, cree un mensaje:

```c
size_t iterator = 0;
do
{
    if (iterator < MESSAGE_COUNT)
    {
        sprintf_s(msgText, sizeof(msgText), "{\"deviceId\":\"myFirstDevice\",\"windSpeed\":%.2f}", avgWindSpeed + (rand() % 4 + 2));
        if ((messages[iterator].messageHandle = IoTHubMessage_CreateFromByteArray((const unsigned char*)msgText, strlen(msgText))) == NULL)
        {
            (void)printf("ERROR: iotHubMessageHandle is NULL!\r\n");
        }
        else
        {
            messages[iterator].messageTrackingId = iterator;
            MAP_HANDLE propMap = IoTHubMessage_Properties(messages[iterator].messageHandle);
            (void)sprintf_s(propText, sizeof(propText), "PropMsg_%zu", iterator);
            if (Map_AddOrUpdate(propMap, "PropName", propText) != MAP_OK)
            {
                (void)printf("ERROR: Map_AddOrUpdate Failed!\r\n");
            }

            if (IoTHubClient_LL_SendEventAsync(iotHubClientHandle, messages[iterator].messageHandle, SendConfirmationCallback, &messages[iterator]) != IOTHUB_CLIENT_OK)
            {
                (void)printf("ERROR: IoTHubClient_LL_SendEventAsync..........FAILED!\r\n");
            }
            else
            {
                (void)printf("IoTHubClient_LL_SendEventAsync accepted message [%d] for transmission to IoT Hub.\r\n", (int)iterator);
            }
        }
    }
    IoTHubClient_LL_DoWork(iotHubClientHandle);
    ThreadAPI_Sleep(1);

    iterator++;
} while (g_continueRunning);
```

Cada vez que envía un mensaje, especifica una referencia a una función de devolución de llamada que se invoca cuando se envían los datos. En este ejemplo, la función de devolución de llamada de llama **SendConfirmationCallback**. El siguiente fragmento de código muestra dicha función:

```c
static void SendConfirmationCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    EVENT_INSTANCE* eventInstance = (EVENT_INSTANCE*)userContextCallback;
    (void)printf("Confirmation[%d] received for message tracking id = %zu with result = %s\r\n", callbackCounter, eventInstance->messageTrackingId, MU_ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
    /* Some device specific action code goes here... */
    callbackCounter++;
    IoTHubMessage_Destroy(eventInstance->messageHandle);
}
```

Observe la llamada a la función **IoTHubMessage\_Destroy** cuando termine con el mensaje. Esta función libera los recursos asignados al crear el mensaje.

### <a name="receive-messages"></a>Recepción de mensajes

La recepción de mensajes es una operación asincrónica. En primer lugar, se registra la devolución de llamada que se invocará cuando el dispositivo recibe un mensaje:

```c
if (IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, ReceiveMessageCallback, &receiveContext) != IOTHUB_CLIENT_OK)
{
    (void)printf("ERROR: IoTHubClient_LL_SetMessageCallback..........FAILED!\r\n");
}
else
{
    (void)printf("IoTHubClient_LL_SetMessageCallback...successful.\r\n");
    ...
```

El último parámetro es un puntero nulo a lo que quiera. En el ejemplo, es un puntero a un entero, pero podría ser un puntero a una estructura de datos más compleja. Este parámetro permite que la función de devolución de llamada funcione en un estado compartido con el autor de la llamada de esta función.

Cuando el dispositivo recibe un mensaje, se invoca la función de devolución de llamada registrada. Dicha función de devolución de llamada recupera:

* El identificador del mensaje y el identificador de correlación del mensaje.
* El contenido del mensaje.
* Todas las propiedades personalizadas del mensaje.

```c
static IOTHUBMESSAGE_DISPOSITION_RESULT ReceiveMessageCallback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    int* counter = (int*)userContextCallback;
    const char* buffer;
    size_t size;
    MAP_HANDLE mapProperties;
    const char* messageId;
    const char* correlationId;

    // Message properties
    if ((messageId = IoTHubMessage_GetMessageId(message)) == NULL)
    {
        messageId = "<null>";
    }

    if ((correlationId = IoTHubMessage_GetCorrelationId(message)) == NULL)
    {
        correlationId = "<null>";
    }

    // Message content
    if (IoTHubMessage_GetByteArray(message, (const unsigned char**)&buffer, &size) != IOTHUB_MESSAGE_OK)
    {
        (void)printf("unable to retrieve the message data\r\n");
    }
    else
    {
        (void)printf("Received Message [%d]\r\n Message ID: %s\r\n Correlation ID: %s\r\n Data: <<<%.*s>>> & Size=%d\r\n", *counter, messageId, correlationId, (int)size, buffer, (int)size);
        // If we receive the work 'quit' then we stop running
        if (size == (strlen("quit") * sizeof(char)) && memcmp(buffer, "quit", size) == 0)
        {
            g_continueRunning = false;
        }
    }

    // Retrieve properties from the message
    mapProperties = IoTHubMessage_Properties(message);
    if (mapProperties != NULL)
    {
        const char*const* keys;
        const char*const* values;
        size_t propertyCount = 0;
        if (Map_GetInternals(mapProperties, &keys, &values, &propertyCount) == MAP_OK)
        {
            if (propertyCount > 0)
            {
                size_t index;

                printf(" Message Properties:\r\n");
                for (index = 0; index < propertyCount; index++)
                {
                    (void)printf("\tKey: %s Value: %s\r\n", keys[index], values[index]);
                }
                (void)printf("\r\n");
            }
        }
    }

    /* Some device specific action code goes here... */
    (*counter)++;
    return IOTHUBMESSAGE_ACCEPTED;
}
```

Use la función **IoTHubMessage\_GetByteArray** para recuperar el mensaje, que en este ejemplo es una cadena.

### <a name="uninitialize-the-library"></a>Anulación de la inicialización de la biblioteca

Cuando haya terminado de enviar eventos y recibir de mensajes, puede anular la inicialización de la biblioteca de IoT. Para ello, emita la siguiente llamada de función:

```c
IoTHubClient_LL_Destroy(iotHubClientHandle);
```

Esto libera los recursos asignados que previamente ha asignado la función **IoTHubClient\_CreateFromConnectionString**.

Como puede ver, es fácil enviar y recibir mensajes con la biblioteca **IoTHubClient**. La biblioteca controla los detalles de la comunicación con Azure IoT Hub, incluido el protocolo que debe usar (desde la perspectiva del desarrollador, es una opción de configuración simple).

La biblioteca **IoTHubClient** también proporciona un control preciso sobre el modo de serializar los datos que el dispositivo envía a IoT Hub. En algunos casos este nivel de control es una ventaja, pero en otros es un detalle de la implementación por el que no desea preocuparse. En ese caso, puede considerar la posibilidad de usar la biblioteca **serializer**, que se describe en la sección siguiente.

## <a name="use-the-serializer-library"></a>Uso de la biblioteca serializer

Conceptualmente, la biblioteca **serializer** se encuentra encima de la biblioteca **IoTHubClient** en el SDK. Usa la biblioteca **IoTHubClient** para la comunicación subyacente con Azure IoT Hub, pero incorpora funcionalidades de modelado que eliminan la carga de tratar con la serialización de mensajes del desarrollador. Un ejemplo ilustra mejor el funcionamiento de esta biblioteca.

Dentro de la carpeta **serializer**, en el repositorio [azure-iot-sdks](https://github.com/Azure/azure-iot-sdk-c), hay una carpeta llamada **samples** que contiene una aplicación denominada **simplesample\_mqtt**. La versión de Windows de este ejemplo incluye la solución de Visual Studio siguiente:

  ![Solución de Visual Studio para ejemplo de mqtt](./media/iot-hub-device-sdk-c-intro/simplesample_mqtt.png)

> [!NOTE]
> Si Visual Studio le solicita que redestine el proyecto a la última versión, acepte la solicitud.

Al igual que con el ejemplo anterior, esta incluye varios paquetes de NuGet:

* Microsoft.Azure.C.SharedUtility
* Microsoft.Azure.IoTHub.MqttTransport
* Microsoft.Azure.IoTHub.IoTHubClient
* Microsoft.Azure.IoTHub.Serializer
* Microsoft.Azure.umqtt

La mayoría de estos paquetes los ha visto en el ejemplo anterior, pero **Microsoft.Azure.IoTHub.Serializer** es nuevo. Este paquete se requiere cuando se usa la biblioteca **serializer**.

La implementación de la aplicación de ejemplo se encuentra en el archivo **iothub_client\_samples\_iothub_convenience_sample**.

Las secciones siguientes le guiarán por las partes principales de este ejemplo.

### <a name="initialize-the-library"></a>Inicialización de la biblioteca

Para empezar a trabajar con la biblioteca **serializer**, llame a las API de inicialización:

```c
if (serializer_init(NULL) != SERIALIZER_OK)
{
    (void)printf("Failed on serializer_init\r\n");
}
else
{
    IOTHUB_CLIENT_LL_HANDLE iotHubClientHandle = IoTHubClient_LL_CreateFromConnectionString(connectionString, MQTT_Protocol);
    srand((unsigned int)time(NULL));
    int avgWindSpeed = 10;

    if (iotHubClientHandle == NULL)
    {
        (void)printf("Failed on IoTHubClient_LL_Create\r\n");
    }
    else
    {
        ContosoAnemometer* myWeather = CREATE_MODEL_INSTANCE(WeatherStation, ContosoAnemometer);
        if (myWeather == NULL)
        {
            (void)printf("Failed on CREATE_MODEL_INSTANCE\r\n");
        }
        else
        {
...
```

La llamada a la función **serializer\_init** es de las que se realiza una sola vez e inicializa la biblioteca subyacente. Después, se llama a la función **IoTHubClient\_LL\_CreateFromConnectionString**, que es la misma API del ejemplo **IoTHubClient**. Esta llamada establece la cadena de conexión del dispositivo (también es donde se elige el protocolo que se desea usar). En este ejemplo se usa MQTT como transporte, pero se podrían usar AMQP o HTTPS.

Por último, llame a la función **CREATE\_MODEL\_INSTANCE**. **WeatherStation** es el espacio de nombres del modelo y **ContosoAnemometer** es el nombre del modelo. Una vez que se crea la instancia del modelo, se puede usar para empezar a enviar y recibir mensajes. Sin embargo, es importante entender qué es un modelo.

### <a name="define-the-model"></a>Definición del modelo

Un modelo de la biblioteca **serializer** define los mensajes que el dispositivo puede enviar a IoT Hub y los mensajes, denominados *acciones* en el lenguaje de modelado, que puede recibir. Defina un modelo usando un conjunto de macros C como en la aplicación de ejemplo **iothub_client\_samples\_iothub_convenience_sample**:

```c
BEGIN_NAMESPACE(WeatherStation);

DECLARE_MODEL(ContosoAnemometer,
WITH_DATA(ascii_char_ptr, DeviceId),
WITH_DATA(int, WindSpeed),
WITH_ACTION(TurnFanOn),
WITH_ACTION(TurnFanOff),
WITH_ACTION(SetAirResistance, int, Position)
);

END_NAMESPACE(WeatherStation);
```

Ambas macros **BEGIN\_NAMESPACE** y **END\_NAMESPACE** toman el espacio de nombres del modelo como argumento. Se espera que todo lo que haya entre estas macros sea la definición del modelo, o modelos, y las estructuras de datos que usan los modelos.

En este ejemplo, hay un único modelo denominado **ContosoAnemometer**. Este modelo define dos datos que el dispositivo puede enviar a IoT Hub: **DeviceId** y **WindSpeed**. También define tres acciones (mensajes) que el dispositivo puede recibir: **TurnFanOn**, **TurnFanOff** y **SetAirResistance**. Cada elemento de datos tiene un tipo y cada acción tiene un nombre (y, opcionalmente, un conjunto de parámetros).

Los datos y las acciones definidos en el modelo definen una superficie de API que se puede usar para enviar mensajes a IoT Hub y para responder a los mensajes que se envían al dispositivo. El uso de este modelo se entiende mejor con un ejemplo.

### <a name="send-messages"></a>Envío de mensajes

El modelo define los datos que puede enviar a IoT Hub. En este ejemplo, eso significa que uno de los dos elementos de datos se define mediante la macro **WITH_DATA**. Para enviar los valores **DeviceId** y **WindSpeed** a IoT Hub, es preciso seguir varios pasos. El primero es establecer los datos que se desean enviar:

```c
myWeather->DeviceId = "myFirstDevice";
myWeather->WindSpeed = avgWindSpeed + (rand() % 4 + 2);
```

El modelo que definió anteriormente le permite establecer los valores mediante el establecimiento de los miembros de un tipo **struct**. A continuación, serialice el mensaje que desee enviar:

```c
unsigned char* destination;
size_t destinationSize;
if (SERIALIZE(&destination, &destinationSize, myWeather->DeviceId, myWeather->WindSpeed) != CODEFIRST_OK)
{
    (void)printf("Failed to serialize\r\n");
}
else
{
    sendMessage(iotHubClientHandle, destination, destinationSize);
    free(destination);
}
```

Este código serializa el dispositivo a la nube a un búfer (al que hace referencia **destination**). Luego, el código invoca a la función **sendMessage** para enviar el mensaje a IoT Hub:

```c
static void sendMessage(IOTHUB_CLIENT_LL_HANDLE iotHubClientHandle, const unsigned char* buffer, size_t size)
{
    static unsigned int messageTrackingId;
    IOTHUB_MESSAGE_HANDLE messageHandle = IoTHubMessage_CreateFromByteArray(buffer, size);
    if (messageHandle == NULL)
    {
        printf("unable to create a new IoTHubMessage\r\n");
    }
    else
    {
        if (IoTHubClient_LL_SendEventAsync(iotHubClientHandle, messageHandle, sendCallback, (void*)(uintptr_t)messageTrackingId) != IOTHUB_CLIENT_OK)
        {
            printf("failed to hand over the message to IoTHubClient");
        }
        else
        {
            printf("IoTHubClient accepted the message for delivery\r\n");
        }
        IoTHubMessage_Destroy(messageHandle);
    }
    messageTrackingId++;
}
```

Del segundo al último parámetros de **IoTHubClient\_LL\_SendEventAsync** son una referencia a una función de devolución de llamada a la que se llama cuando los datos se envían correctamente. Esta es la función de devolución de llamada del ejemplo:

```c
void sendCallback(IOTHUB_CLIENT_CONFIRMATION_RESULT result, void* userContextCallback)
{
    unsigned int messageTrackingId = (unsigned int)(uintptr_t)userContextCallback;

    (void)printf("Message Id: %u Received.\r\n", messageTrackingId);

    (void)printf("Result Call Back Called! Result is: %s \r\n", MU_ENUM_TO_STRING(IOTHUB_CLIENT_CONFIRMATION_RESULT, result));
}
```

El segundo parámetro es un puntero que lleva al contexto de usuario, el mismo puntero que se pasó a **IoTHubClient\_LL\_SendEventAsync**. En este caso, el contexto es un contador sencillo, pero puede ser lo que desee.

Eso es todo lo necesario para enviar mensajes del dispositivo a la nube. Lo único que queda describir es cómo recibir mensajes.

### <a name="receive-messages"></a>Recepción de mensajes

La recepción de un mensaje es similar a cómo funcionan los mensajes en la biblioteca **IoTHubClient** . En primer lugar, se registra una función de devolución de llamada de mensaje:

```c
if (IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, 
  IoTHubMessage, myWeather) != IOTHUB_CLIENT_OK)
{
    printf("unable to IoTHubClient_SetMessageCallback\r\n");
}
else
{
...
```

Luego, se escribe la función de devolución de llamada que se invoca cuando se recibe un mensaje:

```c
static IOTHUBMESSAGE_DISPOSITION_RESULT IoTHubMessage(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    IOTHUBMESSAGE_DISPOSITION_RESULT result;
    const unsigned char* buffer;
    size_t size;
    if (IoTHubMessage_GetByteArray(message, &buffer, &size) != IOTHUB_MESSAGE_OK)
    {
        printf("unable to IoTHubMessage_GetByteArray\r\n");
        result = IOTHUBMESSAGE_ABANDONED;
    }
    else
    {
        /*buffer is not zero terminated*/
        char* temp = malloc(size + 1);
        if (temp == NULL)
        {
            printf("failed to malloc\r\n");
            result = IOTHUBMESSAGE_ABANDONED;
        }
        else
        {
            (void)memcpy(temp, buffer, size);
            temp[size] = '\0';
            EXECUTE_COMMAND_RESULT executeCommandResult = EXECUTE_COMMAND(userContextCallback, temp);
            result =
                (executeCommandResult == EXECUTE_COMMAND_ERROR) ? IOTHUBMESSAGE_ABANDONED :
                (executeCommandResult == EXECUTE_COMMAND_SUCCESS) ? IOTHUBMESSAGE_ACCEPTED :
                IOTHUBMESSAGE_REJECTED;
            free(temp);
        }
    }
    return result;
}
```

Este código es repetitivo, lo que significa que es el mismo para cualquier solución. Esta función recibe el mensaje y se encarga de enrutarlo a la función adecuada mediante la llamada a **EXECUTE\_COMMAND**. La función a la que se llama en este punto depende de la definición de las acciones del modelo.

Cuando se define una acción en el modelo, es necesario implementar una función a la que se llama cuando el dispositivo recibe el mensaje correspondiente. Por ejemplo, si el modelo define esta acción:

```c
WITH_ACTION(SetAirResistance, int, Position)
```

Defina una función con esta firma:

```c
EXECUTE_COMMAND_RESULT SetAirResistance(ContosoAnemometer* device, int Position)
{
    (void)device;
    (void)printf("Setting Air Resistance Position to %d.\r\n", Position);
    return EXECUTE_COMMAND_SUCCESS;
}
```

Observe que el nombre de la función coincide con el nombre de la acción en el modelo y que los parámetros de la función coinciden con los parámetros especificados para la acción. El primer parámetro siempre es necesario y contiene un puntero a la instancia del modelo.

Cuando el dispositivo recibe un mensaje que coincide con esta firma, se llama a la función correspondiente. Por lo tanto, aparte de tener que incluir el código reutilizable de **IoTHubMessage**, para recibir mensajes solo hay que definir una función sencilla para cada acción definida en el modelo.

### <a name="uninitialize-the-library"></a>Anulación de la inicialización de la biblioteca

Cuando haya terminado el envío de datos y la recepción de mensajes, puede anular la inicialización de la biblioteca de IoT:

```c
...
        DESTROY_MODEL_INSTANCE(myWeather);
    }
    IoTHubClient_LL_Destroy(iotHubClientHandle);
}
serializer_deinit();
```

Cada una de estas tres funciones se corresponde con las tres funciones de inicialización descritas anteriormente. Llamar a estas API garantiza que se liberan los recursos asignados previamente.

## <a name="next-steps"></a>Pasos siguientes

Este artículo contiene los aspectos básicos del uso de las bibliotecas en el **SDK de dispositivo IoT de Azure para C**. Se le ha proporcionado información suficiente para conocer lo que incluye el SDK, su arquitectura y cómo empezar a trabajar con los ejemplos de Windows. El siguiente artículo continúa la descripción del SDK y explica [más información sobre la biblioteca IoTHubClient](iot-hub-device-sdk-c-iothubclient.md).

Para más información acerca del desarrollo para IoT Hub, consulte los [SDK de IoT Hub](iot-hub-devguide-sdks.md).

Para explorar aún más las funcionalidades de IoT Hub, consulte:

* [Implementación de IA en dispositivos perimetrales con Azure IoT Edge](../iot-edge/tutorial-simulate-device-linux.md)
