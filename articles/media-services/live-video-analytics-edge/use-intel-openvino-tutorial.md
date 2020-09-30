---
title: Análisis de vídeo en directo mediante la extensión OpenVINO™ Model Server – AI de Intel
description: En este tutorial, se usará un servidor de modelos de IA proporcionado por Intel para analizar la fuente de vídeo en directo desde una cámara IP (simulada).
ms.topic: tutorial
ms.date: 09/08/2020
titleSuffix: Azure
ms.openlocfilehash: e620da1a4f0b7f782d478314fb0e2e83ab9a124a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90906627"
---
# <a name="tutorial-analyze-live-video-by-using-openvino-model-server--ai-extension-from-intel"></a>Tutorial: Análisis de vídeo en directo mediante la extensión OpenVINO™ Model Server – AI de Intel 

En este tutorial se muestra cómo usar la extensión OpenVINO™ Model Server – AI de Intel para analizar una fuente de vídeo en directo desde una cámara IP (simulada). Verá cómo este servidor de inferencia le proporciona acceso a modelos para detectar objetos (una persona, un vehículo o una bicicleta), y a un modelo para clasificar vehículos. Un subconjunto de los fotogramas de la fuente de vídeo en directo se envía a este servidor de inferencia y los resultados se envían al centro de IoT Edge.

En este tutorial se usa una máquina virtual de Azure como dispositivo IoT Edge y se emplea una secuencia de vídeo en directo simulada. Se basa en el código de ejemplo escrito en C# y se basa en [Inicio rápido: Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md).

> [!NOTE]
> En este tutorial se requiere el uso de una máquina x86-64 como dispositivo perimetral.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure que incluya una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), en caso de que aún no lo haya hecho.
* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    * [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
    * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* Si no realizó el inicio rápido [Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md), asegúrese de llevar a cabo los pasos para [configurar los recursos de Azure](detect-motion-emit-events-quickstart.md#set-up-azure-resources).

> [!TIP]
> Al instalar Azure IoT Tools, es posible que se le pida que instale Docker. Puede omitir el aviso.

## <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo

Cuando se configuran los recursos de Azure, se copia un vídeo corto de un aparcamiento en la máquina virtual Linux en Azure que se va a usar como dispositivo IoT Edge. En este inicio rápido se usa el archivo de vídeo para simular una secuencia en directo.

Abra una aplicación como [VLC Media Player](https://www.videolan.org/vlc/). Seleccione Ctrl + N y, después, pegue un vínculo al [vídeo](https://lvamedia.blob.core.windows.net/public/lots_015.mkv) para iniciar la reproducción. Se verá el metraje de los vehículos de un aparcamiento, la mayoría de ellos estacionados, y uno en movimiento.

En este inicio rápido, usará Live Video Analytics en IoT Edge junto con la extensión OpenVINO™ Model Server – AI de Intel para detectar objetos (por ejemplo, vehículos) o para clasificarlos. Los eventos de inferencia resultantes se publicarán en el centro de IoT Edge.

## <a name="overview"></a>Información general

> [!div class="mx-imgBorder"]
> :::image type="content" source="./media/use-intel-openvino-tutorial/http-extension-with-vino.svg" alt-text="Información general":::

En este diagrama se muestra cómo fluyen las señales en este inicio rápido. Un [módulo perimetral](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen de RTSP](media-graph-concept.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de filtros de velocidad de fotogramas](media-graph-concept.md#frame-rate-filter-processor), que limita la velocidad de los fotogramas de la secuencia de vídeo que llega al nodo del [procesador de extensión HTTP](media-graph-concept.md#http-extension-processor). 

El nodo de extensión HTTP desempeña el rol de un proxy. Convierte los fotogramas de vídeo en el tipo de imagen especificado. Luego, la imagen se retransmite a través de REST a otro módulo perimetral que ejecuta modelos de IA detrás de un punto de conexión HTTP. En este ejemplo, ese módulo perimetral es la extensión OpenVINO™ Model Server – AI de Intel. El nodo del procesador de extensión HTTP recopila los resultados de la detección y publica los eventos en el nodo del [receptor de IoT Hub](media-graph-concept.md#iot-hub-message-sink), que posteriormente los envía a [IoT Edge Hub](../../iot-edge/iot-edge-glossary.md#iot-edge-hub).

En este tutorial va a:

1. Crear e implementar el grafo de elementos multimedia, y lo modificará.
1. Interpretará los resultados.
1. Limpieza de recursos.

## <a name="about-openvino-model-server--ai-extension-from-intel"></a>Acerca de la extensión OpenVINO™ Model Server – AI de Intel

La distribución del [kit de herramientas de OpenVINO™](https://software.intel.com/content/www/us/en/develop/tools/openvino-toolkit.html) de Intel® (la inferencia visual abierta y la optimización de la red neuronal) es un kit de software gratuito que ayuda a los desarrolladores y a los científicos de datos a acelerar las cargas de trabajo de visión artificial, a simplificar la inferencia y las implementaciones de aprendizaje profundo y a permitir la ejecución sencilla y heterogénea en las plataformas de Intel® del perímetro a la nube. Incluye Deep Learning Deployment Toolkit de Intel® con optimizador de modelos y motor de inferencia, así como el repositorio [Open Model Zoo](https://github.com/openvinotoolkit/open_model_zoo) que incluye más de 40 modelos preentrenados optimizados.

Con el fin de crear soluciones de análisis de vídeo en directo complejas y de alto rendimiento, el módulo Live Video Analytics en IoT Edge se debe emparejar con un potente motor de inferencia que pueda aprovechar la escala en el perímetro. En este tutorial, se envían solicitudes de inferencia a la [extensión OpenVINO™ Model Server – AI de Intel](https://aka.ms/lva-intel-ovms), un módulo perimetral diseñado para funcionar con Live Video Analytics en IoT Edge. Este módulo de servidor de inferencia contiene la solución OpenVINO™ Model Server (OVMS), un servidor de inferencia que incluye el kit de herramientas de OpenVINO™; este está perfectamente optimizado para cargas de trabajo de Computer Vision y se ha desarrollado para las arquitecturas de Intel. Se ha agregado una extensión a OVMS para facilitar el intercambio de fotogramas de vídeo y resultados de inferencia entre el servidor de inferencia y el módulo de Live Video Analytics en IoT Edge, lo que le permite ejecutar cualquier modelo compatible con el kit de herramientas de OpenVINO™ (puede personalizar el módulo de servidor de inferencia al modificar [este código](https://github.com/openvinotoolkit/model_server/tree/master/extras/ams_wrapper)). Además, se puede seleccionar entre una amplia variedad de mecanismos de aceleración proporcionados por hardware de Intel®. Entre ellos figuran las CPU (Atom, Core, Xeon), los FPGA, y las VPU.

En la versión inicial de este servidor de inferencia, se tiene acceso a los siguientes [modelos](https://github.com/openvinotoolkit/model_server/tree/master/extras/ams_models):

- Detección de vehículos (dirección URL de inferencia: http://{module-name}: 4000/vehicleDetection)
- Detección de personas/vehículos/bicicletas (dirección URL de inferencia: http://{module-name}: 4000/personVehicleBikeDetection)
- Clasificación de vehículos (dirección URL de inferencia: http://{module-name}: 4000/vehicleClassification)
- Detección de caras (dirección URL de inferencia: http://{module-name}: 4000/faceDetection)

> [!NOTE]
> Al descargar y usar el módulo de Edge: la extensión OpenVINO™ Model Server – AI de Intel, y el software incluido, acepte los términos y condiciones del [contrato de licencia](https://www.intel.com/content/www/us/en/legal/terms-of-use.html).
> Intel se compromete a respetar los derechos humanos y a evitar ser cómplice de la vulneración de tales derechos. Consulte los [principios de los derechos humanos globales de Intel](https://www.intel.com/content/www/us/en/policy/policy-human-rights.html). Los productos y el software de Intel están concebidos únicamente para usarse en aplicaciones que no infrinjan los derechos humanos reconocidos internacionalmente, ni contribuyan a tal infracción.

## <a name="create-and-deploy-the-media-graph"></a>Creación e implementación del grafo de elementos multimedia

### <a name="examine-and-edit-the-sample-files"></a>Examen y edición de los archivos de ejemplo

Como parte de los requisitos previos, ha descargado el código de ejemplo en una carpeta. Siga estos pasos para examinar y editar los archivos de ejemplo.

1. En Visual Studio Code, vaya a *src/edge*. Puede ver el archivo *.env* y algunos archivos de plantilla de implementación.

    La plantilla de implementación hace referencia al manifiesto de implementación del dispositivo perimetral. Incluye algunos valores de marcador de posición. El archivo *.env* incluye los valores de esas variables.

1. Vaya a la carpeta *src/cloud-to-device-console-app*. Aquí verá tanto su archivo *appsettings.json* como otros archivos:

    * ***c2d-console-app.csproj***: el archivo de proyecto de Visual Studio Code.
    * ***operations.json***: una lista de las operaciones que desea que ejecute el programa.
    * ***Program.cs***: el código del programa de ejemplo. Este código:

        * Carga la configuración de la aplicación.
        * Invoca los métodos directos que expone el módulo Live Video Analytics en IoT Edge. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus [métodos directos](direct-methods.md).
        * Se pone en pausa para que pueda examinar la salida del programa en la ventana **TERMINAL** y los eventos generados por el módulo en la ventana **SALIDA**.
        * Invoca los métodos directos para limpiar los recursos.


1. Edite el archivo *operations.json*:
    * Cambie el vínculo a la topología del grafo:

        `"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtensionOpenVINO/topology.json"`

    * En `GraphInstanceSet`, edite el nombre de la topología del grafo para que coincida con el valor del vínculo anterior:

      `"topologyName" : "InferencingWithOpenVINO"`

    * En `GraphTopologyDelete`, edite el nombre:

      `"name": "InferencingWithOpenVINO"`

### <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Generación e implementación del manifiesto de implementación de IoT Edge

1. Haga clic con el botón derecho en el archivo *src/edge/deployment.openvino.template.json* y seleccione **Generación de un manifiesto de implementación de IoT Edge**.

    ![Generación de un manifiesto de implementación de IoT Edge](./media/use-intel-openvino-tutorial/generate-deployment-manifest.png)  

    El archivo de manifiesto *deployment.yolov3.amd64.json* se crea en la carpeta *src/edge/config*.

1. Si ha completado el inicio rápido [Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md), omita este paso. 

    Si no lo ha hecho, cerca del panel **AZURE IOT HUB**, en la esquina inferior izquierda, seleccione el icono **Más acciones** y, después, seleccione **Set IoT Hub Connection String** (Establecer cadena de conexión de IoT Hub). La cadena se puede copiar del archivo *appsettings.json*. O bien, para asegurarse de que ha configurado el centro de IoT adecuado en Visual Studio Code, use el [comando de selección de IoT Hub](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub).
    
    ![Establecimiento de la cadena de conexión de IoT Hub](./media/quickstarts/set-iotconnection-string.png)

1. Haga clic con el botón derecho en *src/edge/config/deployment.openvino.amd64.json* y seleccione **Crear una implementación para un dispositivo individual**. 

    ![Crear una implementación para un dispositivo individual](./media/use-intel-openvino-tutorial/deploy-manifest.png)

1. Cuando se le pida que seleccione un dispositivo IoT Hub, seleccione **lva-sample-device**.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice Azure IoT Hub. El dispositivo perimetral ahora muestra los siguientes módulos implementados:

    * El módulo de Live Video Analytics, denominado **lvaEdge**.
    * El módulo **rtspsim**, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo.
    * El módulo **openvino**, que es el módulo de la extensión OpenVINO™ Model Server – AI de Intel 

### <a name="prepare-to-monitor-events"></a>Preparación para supervisar eventos

Haga clic con el botón derecho en el dispositivo de Live Video Analytics y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión de eventos integrado). Este paso es necesario para supervisar los eventos de IoT Hub en la ventana **SALIDA** de Visual Studio Code. 

![Iniciar supervisión](./media/quickstarts/start-monitoring-iothub-events.png) 

### <a name="run-the-sample-program-to-detect-vehicles"></a>Ejecución del programa de ejemplo para detectar vehículos
Si abre la [topología del grafo](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtensionOpenVINO/topology.json) de este tutorial en un explorador, verá que el valor de `inferencingUrl` se ha establecido en `http://openvino:4000/vehicleDetection`, lo que significa que el servidor de inferencia devolverá resultados después de detectar vehículos, si los hay, en el vídeo en directo.

1. Para iniciar una sesión de depuración, seleccione la tecla F5. Verá mensajes impresos en la ventana **TERMINAL**.
1. El código de *operations.json* comienza con llamadas a los métodos directos `GraphTopologyList` y `GraphInstanceList`. Si ha limpiado los recursos tras haber completado los inicios rápidos anteriores, este proceso devolverá listas vacías y, después, se pausará. Para continuar, seleccione la tecla Entrar.

    La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:

     * Una llamada a `GraphTopologySet` que utiliza la instancia anterior de `topologyUrl`.
     * Una llamada a `GraphInstanceSet` que usa el cuerpo siguiente:

         ```
         {
           "@apiVersion": "1.0",
           "name": "Sample-Graph-1",
           "properties": {
             "topologyName": "InferencingWithOpenVINO",
             "description": "Sample graph description",
             "parameters": [
               {
                 "name": "rtspUrl",
                 "value": "rtsp://rtspsim:554/media/lots_015.mkv"
               },
               {
                 "name": "rtspUserName",
                 "value": "testuser"
               },
               {
                 "name": "rtspPassword",
                 "value": "testpassword"
               }
             ]
           }
         }
         ```

     * Una llamada a `GraphInstanceActivate` que inicia la instancia del grafo y el flujo de vídeo.
     * Una segunda llamada a `GraphInstanceList` que muestra que la instancia del grafo está en ejecución.
1. La salida de la ventana **TERMINAL** se pone en pausa tras el mensaje `Press Enter to continue`. No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana **SALIDA** de Visual Studio Code. Verá los mensajes que el módulo Live Video Analytics en IoT Edge envía al centro de IoT. En la siguiente sección de este inicio rápido se analizan estos mensajes.
1. El grafo multimedia continúa ejecutándose e imprimiendo los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener el grafo multimedia, vuelva a la ventana **TERMINAL** y seleccione Entrar. 

    La siguiente serie de llamadas limpia los recursos:
      * Una llamada a `GraphInstanceDeactivate` desactiva la instancia del grafo.
      * Una llamada a `GraphInstanceDelete` elimina la instancia.
      * Una llamada a `GraphTopologyDelete` elimina la topología.
      * Una llamada final a `GraphTopologyList` muestra que la lista está vacía.

## <a name="interpret-results"></a>Interpretación de los resultados

Cuando se ejecuta el grafo multimedia, los resultados del nodo del procesador de extensión HTTP atraviesan el nodo de receptor de IoT Hub y llegan al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** contienen una sección `body` y una sección `applicationProperties`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, se definen las propiedades de la aplicación y el contenido del cuerpo mediante el módulo Live Video Analytics. 

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se crean instancias del grafo multimedia, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente. El tipo de evento es `Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished`.

```
[IoTHubMonitor] [9:42:18 AM] Message received from [lvaedgesample/lvaEdge]:
{
  "body": {
    "sdp": "SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 nnn.nn.0.6\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets=Z00AKeKQCgC3YC3AQEBpB4kRUA==,aO48gA==\r\na=control:track1\r\n"
  },
  "applicationProperties": {
    "dataVersion": "1.0",
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/sources/rtspSource",
    "eventType": "Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished",
    "eventTime": "2020-07-24T16:42:18.1280000Z"
  }
}
```

En este mensaje, tenga en cuenta estos detalles:

* El mensaje es un evento de diagnóstico. `MediaSessionEstablished` indica que el nodo de origen RTSP (el sujeto) ha conectado con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* En `applicationProperties`, `subject` indica que el mensaje se generó desde el nodo de origen RTSP del grafo multimedia.
* En `applicationProperties`, `eventType` indica que este evento es de diagnóstico.
* `eventTime` indica la hora a la que se produjo el evento.
* `body` contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="inference-event"></a>Evento de inferencia

El nodo procesador de la extensión HTTP recibe resultados de inferencia del módulo de la extensión OpenVINO™ Model Server – AI. Luego, emite los resultados a través del nodo del receptor de IoT Hub como eventos de inferencia. 

En estos eventos, el tipo se establece en `entity` para indicar que es una entidad, como un coche o un camión. El valor `eventTime` es la hora UTC en que se detectó el objeto. 

En el ejemplo siguiente, se detectaron dos vehículos, con un valor de confianza superior a 0,9.

```
[IoTHubMonitor] [9:43:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "inferences": [
      {
        "type": "entity",
        "subtype": "vehicleDetection",
        "entity": {
          "tag": {
            "value": "vehicle",
            "confidence": 0.9951713681221008
          },
          "box": {
            "l": 0.042635321617126465,
            "t": 0.4004564881324768,
            "w": 0.10961548984050751,
            "h": 0.07942074537277222
          }
        }
      },
      {
        "type": "entity",
        "subtype": "vehicleDetection",
        "entity": {
          "tag": {
            "value": "vehicle",
            "confidence": 0.928486168384552
          },
          "box": {
            "l": 0.2506900727748871,
            "t": 0.07512682676315308,
            "w": 0.05470699071884155,
            "h": 0.07408371567726135
          }
        }
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/inferenceClient",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-07-24T16:43:18.1280000Z"
  }
}
```

En los mensajes, tenga en cuenta los siguientes detalles:

* En `applicationProperties`, `subject` hace referencia al nodo de la topología del grafo multimedia desde el que se generó el mensaje. 
* En `applicationProperties`, `eventType` indica que este es un evento de análisis.
* El valor `eventTime` es la hora en que se produjo el evento.
* La sección `body` contiene datos sobre el evento de análisis. En este caso, el evento es un evento de inferencia, por lo que el cuerpo contiene los datos `inferences`.
* La sección `inferences` indica que el valor de `type` es `entity`. En esta sección se incluyen datos adicionales sobre la entidad.

## <a name="run-the-sample-program-to-detect-persons-or-vehicles-or-bikes"></a>Ejecución del programa de ejemplo para detectar personas, vehículos o bicicletas
Para usar un modelo diferente, tendrá que modificar la topología del grafo y también el archivo `operations.json`.

Copie la [topología del grafo](https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/httpExtensionOpenVINO/topology.json) en un archivo local, por ejemplo, `C:\TEMP\topology.json`. Abra esa copia y cambie el valor de `inferencingUrl` por `http://openvino:4000/personVehicleBikeDetection`.

En Visual Studio Code, vaya a la carpeta *src/cloud-to-device-console-app* y abra el archivo `operations.json`. Cambie la línea con `topologyUrl` por:

```
      "topologyFile" : "C:\\TEMP\\topology.json" 
```
Ahora puede repetir los pasos anteriores para volver a ejecutar el programa de ejemplo con la nueva topología. Los resultados de la inferencia serán similares (en el esquema) a los del modelo de detección de vehículos, solo que `subtype` está establecido en `personVehicleBikeDetection`.

## <a name="run-the-sample-program-to-classify-vehicles"></a>Ejecución del programa de ejemplo para clasificar vehículos
En Visual Studio Code, abra la copia local de `topology.json` del paso anterior y cambie el valor de `inferencingUrl` por `http://openvino:4000/vehicleClassification`. Si ha ejecutado el ejemplo anterior para detectar personas, vehículos o bicicletas, no es necesario volver a modificar el archivo `operations.json`.

Ahora puede repetir los pasos anteriores para volver a ejecutar el programa de ejemplo con la nueva topología. A continuación, se muestra un resultado de clasificación de ejemplo.

```
[IoTHubMonitor] [9:44:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "inferences": [
      {
        "type": "classification",
        "subtype": "color",
        "classification": {
          "tag": {
            "value": "black",
            "confidence": 0.9179772138595581
          }
        }
      },
      {
        "type": "classification",
        "subtype": "type",
        "classification": {
          "tag": {
            "value": "truck",
            "confidence": 1
          }
        }
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/inferenceClient",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-07-24T16:44:18.1280000Z"
  }
}
```

## <a name="run-the-sample-program-to-detect-faces"></a>Ejecución del programa de ejemplo para detectar caras
En Visual Studio Code, abra la copia local de `topology.json` del paso anterior y cambie el valor de `inferencingUrl` por `http://openvino:4000/faceDetection`. Si ha ejecutado el ejemplo anterior para detectar personas, vehículos o bicicletas, no es necesario volver a modificar el archivo `operations.json`.

Ahora puede repetir los pasos anteriores para volver a ejecutar el programa de ejemplo con la nueva topología. A continuación, se muestra un resultado de detección de ejemplo (Nota: El vídeo del aparcamiento usado anteriormente no contiene ninguna cara detectable, por lo que se debe usar otro vídeo para probar este modelo).

```
[IoTHubMonitor] [9:54:18 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "inferences": [
      {
        "type": "entity",
        "subtype": "faceDetection",
        "entity": {
          "tag": {
            "value": "face",
            "confidence": 0.9997053742408752
          },
          "box": {
            "l": 0.2559490501880646,
            "t": 0.03403960168361664,
            "w": 0.17685115337371826,
            "h": 0.45835764706134796
          }
        }
      }
    ]
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",
    "subject": "/graphInstances/GRAPHINSTANCENAMEHERE/processors/inferenceClient",
    "eventType": "Microsoft.Media.Graph.Analytics.Inference",
    "eventTime": "2020-07-24T16:54:18.1280000Z"
  }
}
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si su intención es probar otros inicios rápidos o tutoriales, conserve los recursos creados. En caso contrario, vaya a Azure Portal, luego a los grupos de recursos, seleccione el grupo de recursos donde ejecutó este tutorial y elimine todos los recursos.

## <a name="next-steps"></a>Pasos siguientes

Revise los desafíos adicionales para los usuarios avanzados:

* Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) que sea compatible con RTSP, en lugar de utilizar el simulador RTSP. Puede buscar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products/). Busque dispositivos que se ajusten a los perfiles G, S o T.
* Use un dispositivo Linux AMD64 o x64, en lugar de una máquina virtual Linux de Azure. El dispositivo debe estar en la misma red que la cámara IP. Puede seguir las instrucciones que aparecen en [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge-linux.md). Luego, registre el dispositivo en Azure IoT Hub, para lo que debe seguir las instrucciones que se encuentran en [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md).
