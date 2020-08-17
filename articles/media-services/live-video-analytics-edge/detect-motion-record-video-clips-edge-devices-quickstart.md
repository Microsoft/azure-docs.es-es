---
title: 'Detección de movimiento y grabación de vídeo en dispositivos perimetrales: Azure'
description: En este inicio rápido se muestra cómo usar Live Video Analytics en IoT Edge para analizar la fuente de vídeo en directo desde una cámara IP (simulada), detectar si hay algún movimiento y, en caso afirmativo, grabar un clip de vídeo MP4 en el sistema de archivos local del dispositivo perimetral.
ms.topic: quickstart
ms.date: 04/27/2020
ms.openlocfilehash: 14dcc7b298244a1d53a9b820c641ea87c4f9a016
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091868"
---
# <a name="quickstart-detect-motion-and-record-video-on-edge-devices"></a>Inicio rápido: Detección de movimiento y grabación de vídeo en dispositivos perimetrales
 
En este inicio rápido se muestra cómo usar Live Video Analytics en IoT Edge para analizar la fuente de vídeo en directo desde una cámara IP (simulada). Muestra cómo detectar si existe algún movimiento y, en caso afirmativo, graba un clip de vídeo MP4 en el sistema de archivos local del dispositivo perimetral. El inicio rápido usa una máquina virtual de Azure como dispositivo IoT Edge y emplea también una secuencia de vídeo en directo simulada. 

Este artículo se basa en un ejemplo de código escrito en C#. Se basa en el inicio rápido [Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md). 

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F), en caso de que aún no lo haya hecho.
* [Visual Studio Code](https://code.visualstudio.com/) con las siguientes extensiones:
    * [Herramientas de Azure IoT](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
    * [C#](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)
* [SDK de .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1).
* Si no ha completado el inicio rápido [Detección de movimiento y emisión de eventos](detect-motion-emit-events-quickstart.md), siga estos pasos:
     1. [Configuración de los recursos de Azure](detect-motion-emit-events-quickstart.md#set-up-azure-resources)
     1. [Configuración del entorno de desarrollo](detect-motion-emit-events-quickstart.md#set-up-your-development-environment)
     1. [Generación e implementación del manifiesto de implementación de IoT Edge](detect-motion-emit-events-quickstart.md#generate-and-deploy-the-deployment-manifest)
     1. [Preparación para supervisar eventos](detect-motion-emit-events-quickstart.md#prepare-to-monitor-events)

> [!TIP]
> Al instalar Azure IoT Tools, es posible que se le pida que instale Docker. Si lo desea, ignore esta petición.

## <a name="review-the-sample-video"></a>Revisión del vídeo de ejemplo
A medida que configura los recursos de Azure de este inicio rápido, se copiará un vídeo (corto) de un área de aparcamiento en la máquina virtual Linux en Azure que se usa como dispositivo IoT Edge. Este archivo de vídeo se usará para simular un streaming en vivo para este tutorial.

Abra una aplicación como [VLC Media Player](https://www.videolan.org/vlc/), seleccione Ctrl + N y pegue [este vínculo](https://lvamedia.blob.core.windows.net/public/lots_015.mkv) en el vídeo sobre el área de aparcamiento para iniciar la reproducción. Hacia la marca de 5 segundos, un coche blanco se desplaza por el aparcamiento.

Realice los pasos que se indican a continuación para usar Live Video Analytics en IoT Edge para detectar el movimiento del coche y grabar un clip de vídeo que empiece en torno a la marca de los 5 segundos.

## <a name="overview"></a>Información general

![Información general](./media/quickstarts/overview-qs4.png)

En el diagrama anterior se muestra cómo fluyen las señales en este inicio rápido. [Un módulo perimetral](https://github.com/Azure/live-video-analytics/tree/master/utilities/rtspsim-live555) simula una cámara IP que hospeda un servidor de protocolo RTSP. Un nodo de [origen RTSP](media-graph-concept.md#rtsp-source) extrae la fuente de vídeo de este servidor y envía fotogramas de vídeo al nodo del [procesador de detección del movimiento](media-graph-concept.md#motion-detection-processor). El origen RTSP envía los mismos fotogramas de vídeo a un nodo de [procesador de la puerta de señales](media-graph-concept.md#signal-gate-processor), que permanece cerrado hasta que un evento lo desencadena.

Cuando el procesador de detección del movimiento detecta movimiento en el vídeo, envía un evento al nodo de procesador de la puerta de señales y lo desencadena. La puerta se abre durante el tiempo configurado y envía fotogramas de vídeo al nodo del [receptor de archivos](media-graph-concept.md#file-sink). Este nodo del receptor registra el vídeo como un archivo MP4 en el sistema de archivos local del dispositivo perimetral. El archivo se guarda en la ubicación configurada.

En este inicio rápido realizará lo siguiente:

1. Creará e implementará el grafo de elementos multimedia.
1. Interpretará los resultados.
1. Limpieza de recursos.

## <a name="examine-and-edit-the-sample-files"></a>Examen y edición de los archivos de ejemplo
Como parte de los requisitos previos de este inicio rápido, ha descargado el código de ejemplo en una carpeta. Siga estos pasos para examinar y editar el código de ejemplo.

1. En Visual Studio Code, vaya a *src/edge*. Puede ver el archivo *.env* y algunos archivos de plantilla de implementación.

    La plantilla de implementación hace referencia al manifiesto de implementación del dispositivo perimetral, en el que las variables se utilizan para algunas propiedades. El archivo *.env* incluye los valores de esas variables.
1. Vaya a la carpeta *src/cloud-to-device-console-app*. Aquí verá el archivo *appsettings.json* y algunos otros archivos:
    * ***c2d-console-app.csproj***: archivo de proyecto de Visual Studio Code.
    * ***operations.json***: la lista de operaciones que desea que ejecute el programa.
    * ***Program.cs***: el código del programa de ejemplo. Este código:

        * Carga la configuración de la aplicación.
        * Invoca los métodos directos que expone el módulo Live Video Analytics en IoT Edge. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus [métodos directos](direct-methods.md). 
        * Se pone en pausa para que pueda examinar la salida del programa en la ventana **TERMINAL** y los eventos generados por el módulo en la ventana **SALIDA**.
        * Invoca los métodos directos para limpiar los recursos.

1. Edite el archivo *operations.json*:
    * Cambie el vínculo a la topología del grafo:

        `"topologyUrl" : "https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/evr-motion-files/topology.json"`
    * En `GraphInstanceSet`, edite el nombre de la topología del grafo para que coincida con el valor del vínculo anterior:
    
      `"topologyName" : "EVRToFilesOnMotionDetection"`

    * Edite la dirección URL de RTSP para que apunte al archivo de vídeo:

        `"value": "rtsp://rtspsim:554/media/lots_015.mkv"`

    * En `GraphTopologyDelete`, edite el nombre:

        `"name": "EVRToFilesOnMotionDetection"`

## <a name="review---check-the-modules-status"></a>Revisión: Comprobación del estado de los módulos

En el paso [Generación e implementación del manifiesto de implementación de IoT Edge](detect-motion-emit-events-quickstart.md#generate-and-deploy-the-deployment-manifest), en Visual Studio Code, expanda el nodo **Iva-sample-device** en **AZURE IOT HUB** (en la sección situada en la parte inferior izquierda). Debería ver los siguientes módulos implementados:

* El módulo de Live Video Analytics, denominado **lvaEdge**.
* El módulo **rtspsim**, que simula un servidor RTSP, que actúa como el origen de una fuente de vídeo en directo.

  ![Módulos](./media/quickstarts/lva-sample-device-node.png)


## <a name="review---prepare-for-monitoring-events"></a>Revisión: Preparación de la supervisión de eventos
Asegúrese de que haya completado los pasos de [preparación de la supervisión de eventos](detect-motion-emit-events-quickstart.md#prepare-to-monitor-events).

![Iniciar la supervisión del punto de conexión de eventos integrado](./media/quickstarts/start-monitoring-iothub-events.png)

## <a name="run-the-sample-program"></a>Ejecución del programa de ejemplo

1. Para iniciar una sesión de depuración, seleccione la tecla F5. La ventana **TERMINAL** muestra algunos mensajes.
1. El código de *operations.json* llama a los métodos directos `GraphTopologyList` y `GraphInstanceList`. Si ha limpiado los recursos después de los inicios rápidos anteriores, este proceso devolverá listas vacías y, a continuación, se pausará. Seleccione la tecla Entrar.

    ```
    --------------------------------------------------------------------------
    Executing operation GraphTopologyList
    -----------------------  Request: GraphTopologyList  --------------------------------------------------
    {
      "@apiVersion": "1.0"
    }
    ---------------  Response: GraphTopologyList - Status: 200  ---------------
    {
      "value": []
    }
    --------------------------------------------------------------------------
    Executing operation WaitForInput
    Press Enter to continue
    ```

    La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:

     * Una llamada a `GraphTopologySet` que utiliza `topologyUrl`. 
     * Una llamada a `GraphInstanceSet` que usa el cuerpo siguiente:

         ```
         {
           "@apiVersion": "1.0",
           "name": "Sample-Graph",
           "properties": {
             "topologyName": "EVRToFilesOnMotionDetection",
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
1. La salida de la ventana **TERMINAL** se pone en pausa en `Press Enter to continue`. No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana **SALIDA** de Visual Studio Code. Verá los mensajes que el módulo Live Video Analytics en IoT Edge envía al centro de IoT. En la siguiente sección de este inicio rápido se analizan estos mensajes.

1. El grafo multimedia continúa ejecutándose e imprimiendo los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener el grafo multimedia, vuelva a la ventana **TERMINAL** y seleccione Entrar. 

    La siguiente serie de llamadas limpia los recursos:
     * Una llamada a `GraphInstanceDeactivate` desactiva la instancia del grafo.
     * Una llamada a `GraphInstanceDelete` elimina la instancia.
     * Una llamada a `GraphTopologyDelete` elimina la topología.
     * Una llamada final a `GraphTopologyList` muestra que la lista está ahora vacía.

## <a name="interpret-results"></a>Interpretación de los resultados 
Cuando se ejecuta el grafo multimedia, los resultados del nodo del procesador de detección del movimiento pasan a través del nodo de receptor de IoT Hub al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** de Visual Studio Code contienen una sección `body` y una sección `applicationProperties`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, se definen las propiedades de la aplicación y el contenido del cuerpo mediante el módulo Live Video Analytics.

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se crean instancias del grafo multimedia, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente.

```
[IoTHubMonitor] [05:37:21 AM] Message received from [lva-sample-device/lvaEdge]:
{  
"body": {
"sdp": "SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 xxx.xxx.xxx.xxx\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets={SPS}\r\na=control:track1\r\n"  
},  
"applicationProperties": {  
    "dataVersion": "1.0",  
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",  
    "subject": "/graphInstances/Sample-Graph-1/sources/rtspSource",  
    "eventType": "Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished",  
    "eventTime": "2020-05-21T05:37:21.398Z",
    }  
}
```

En la salida anterior: 

* El mensaje es un evento de diagnóstico `MediaSessionEstablished`. Indica que el nodo de origen RTSP (el sujeto) ha establecido una conexión con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* En `applicationProperties`, `subject` hace referencia al nodo de la topología del grafo de elementos multimedia desde el que se generó el mensaje. En este caso, el mensaje se origina en el nodo de origen RTSP.
* En `applicationProperties`, `eventType` indica que este evento es un evento de diagnóstico.
* El valor `eventTime` es la hora en que se produjo el evento.
* La sección `body` contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="recordingstarted-event"></a>Evento RecordingStarted

Cuando se detecta movimiento, se activa el nodo de procesador de la puerta de señales y el nodo de receptor de archivos del grafo multimedia empieza a escribir un archivo MP4. El nodo del receptor de archivos envía un evento operativo. El `type` se establece en `motion` para indicar que se trata de un resultado del procesador de detección del movimiento. El valor `eventTime` es la hora UTC en la que se produjo el movimiento. Para más información sobre este proceso, consulte la sección de [Introducción](#overview) de este inicio rápido.

A continuación se muestra un ejemplo de este mensaje:

```
[IoTHubMonitor] [05:37:27 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "filePath",
    "outputLocation": "/var/media/sampleFilesFromEVR-filesinkOutput-20200521T053726Z.mp4"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",  
    "subject": "/graphInstances/Sample-Graph-1/sinks/fileSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingStarted",
    "eventTime": "2020-05-21T05:37:27.713Z",
    "dataVersion": "1.0"
  }
}
```

En el mensaje anterior: 

* En `applicationProperties`, `subject` hace referencia al nodo del grafo multimedia desde el que se generó el mensaje. En este caso, el mensaje se origina en el nodo del receptor de archivos.
* En `applicationProperties`, `eventType` indica que este evento es operativo.
* El valor `eventTime` es la hora en que se produjo el evento. Esta hora es entre 5 y 6 segundos posterior a `MediaSessionEstablished` y posterior al inicio del flujo de vídeo. Esta hora se corresponde a la marca de 5 o 6 segundos cuando el [coche empezó a moverse](#review-the-sample-video) por el aparcamiento.
* La sección `body` contiene datos sobre el evento operativo. En este caso, los datos incluyen `outputType` y `outputLocation`.
* La variable `outputType` indica que esta información es sobre la ruta de acceso del archivo.
* El valor `outputLocation` es la ubicación del archivo MP4 en el módulo de Edge.

### <a name="recordingstopped-and-recordingavailable-events"></a>Eventos RecordingStopped y RecordingAvailable

Si examina las propiedades del nodo de procesador de la puerta de señales en la [topología del grafo](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-motion-files/topology.json) puede ver que los tiempos de activación se han establecido en 5 segundos. Por tanto, unos 5 segundos después de que se reciba el evento `RecordingStarted`, obtendrá:

* Un evento `RecordingStopped`, que indica que la grabación se ha detenido.
* Un evento `RecordingAvailable`, que indica que ahora se puede usar el archivo MP4 para su visualización.

Normalmente, los dos eventos se emiten con unos segundos de diferencia.

## <a name="play-the-mp4-clip"></a>Reproducción del clip de MP4

Los archivos MP4 se escriben en un directorio del dispositivo perimetral que configuró en el archivo *.env* mediante la clave OUTPUT_VIDEO_FOLDER_ON_DEVICE. Si ha utilizado el valor predeterminado, los resultados estarán en la carpeta */home/lvaadmin/samples/output/* .

Para reproducir el clip de MP4:

1. Vaya al grupo de recursos, busque la máquina virtual y conéctese mediante Azure Bastion.

    ![Resource group](./media/quickstarts/resource-group.png)
    
    ![máquina virtual](./media/quickstarts/virtual-machine.png)

1. Inicie sesión con las credenciales que se generaron al [configurar los recursos de Azure](detect-motion-emit-events-quickstart.md#set-up-azure-resources). 
1. En el símbolo del sistema, vaya al directorio correspondiente. La ubicación predeterminada es */home/lvaadmin/samples/output*. Debería ver los archivos MP4 en el directorio.

    ![Output](./media/quickstarts/samples-output.png) 

1. Use [Secure Copy (SCP)](../../virtual-machines/linux/copy-files-to-linux-vm-using-scp.md) para copiar los archivos en la máquina local. 
1. Reproduzca los archivos mediante [VLC Media Player](https://www.videolan.org/vlc/) o cualquier otro reproductor MP4.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si su intención es probar los demás inicios rápidos, conserve los recursos creados. En caso contrario, en Azure Portal, vaya a los grupos de recursos, seleccione el grupo de recursos en el que ejecutó este inicio rápido y, a continuación, elimine todos los recursos.

## <a name="next-steps"></a>Pasos siguientes

* Siga las indicaciones del inicio rápido [Ejecución de Live Video Analytics con su propio modelo](use-your-model-quickstart.md) para aplicar la inteligencia artificial a las fuentes de vídeo en directo.
* Revise los desafíos adicionales para los usuarios avanzados:

    * Use una [cámara IP](https://en.wikipedia.org/wiki/IP_camera) compatible con RTSP en lugar de utilizar el simulador RTSP. Puede encontrar cámaras IP compatibles con RTSP en la página de [productos compatibles con ONVIF](https://www.onvif.org/conformant-products). Busque dispositivos que cumplan con los perfiles G, S o T.
    * Use un dispositivo Linux AMD64 o x64 en lugar de usar una máquina virtual Linux en Azure. El dispositivo debe estar en la misma red que la cámara IP. Siga las instrucciones de [Instalación del entorno de ejecución de Azure IoT Edge en Linux](../../iot-edge/how-to-install-iot-edge-linux.md). A continuación, siga las instrucciones que se encuentran en [Implementación del primer módulo IoT Edge en un dispositivo virtual Linux](../../iot-edge/quickstart-linux.md) para registrar el dispositivo con Azure IoT Hub.
