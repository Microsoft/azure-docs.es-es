---
ms.openlocfilehash: 8f9ed14a0bcef346281c38146cbb2d9551633c15
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89421545"
---
### <a name="examine-and-edit-the-sample-files"></a>Examen y edición de los archivos de ejemplo

Como parte de los requisitos previos, ha descargado el código de ejemplo en una carpeta. Siga estos pasos para examinar y editar los archivos de ejemplo.

1. En Visual Studio Code, vaya a *src/edge*. Puede ver el archivo *.env* y algunos archivos de plantilla de implementación.

    El archivo deployment.grpcyolov3icpu.template.json hace referencia al manifiesto de implementación del dispositivo perimetral. Incluye algunos valores de marcador de posición. El archivo .env incluye los valores de esas variables.
1. Vaya a la carpeta *src/cloud-to-device-console-app*. Aquí verá tanto su archivo *appsettings.json* como otros archivos:
    
    * c2d-console-app.csproj: el archivo de proyecto de Visual Studio Code.
    * operations.json: una lista de las operaciones que desea que ejecute el programa.
    * Program.cs: el código del programa de ejemplo. Este código:

        * Carga la configuración de la aplicación.
        * Invoca los métodos directos que expone el módulo Live Video Analytics en IoT Edge. Puede usar el módulo para analizar secuencias de vídeo en directo mediante la invocación de sus métodos directos.
        * Se pone en pausa para que pueda examinar la salida del programa en la ventana **TERMINAL** y los eventos generados por el módulo en la ventana **SALIDA**.
        * Invoca los métodos directos para limpiar los recursos.
1. Edite el archivo *operations.json*:
 
    * Cambie el vínculo a la topología del grafo:
    * `"topologyUrl"` : `"https://raw.githubusercontent.com/Azure/live-video-analytics/master/MediaGraph/topologies/grpcExtension/topology.json"`
    * En GraphInstanceSet, edite el nombre de la topología del grafo para que coincida con el valor del vínculo anterior:
    * `"topologyName"` : `"InferencingWithGrpcExtension"`
    * En GraphTopologyDelete, edite el nombre:
    * `"name"` : `"InferencingWithGrpcExtension"`

> [!NOTE]
> <p>
> <details>
> <summary>Expándalo y consulte cómo se implementa el nodo MediaGraphGrpcExtension en la topología</summary>
> <pre><code>
> {
>   "@type": "#Microsoft.Media.MediaGraphGrpcExtension",
>   "name": "grpcExtension",
>   "endpoint": {
>       "@type": "#Microsoft.Media.MediaGraphUnsecuredEndpoint",
>       "url": "${grpcExtensionAddress}",
>       "credentials": {
>           "@type": "#Microsoft.Media.MediaGraphUsernamePasswordCredentials",
>           "username": "${grpcExtensionUserName}",
>           "password": "${grpcExtensionPassword}"
>       }
>   },
>   "dataTransfer": {
>       "mode": "sharedMemory",
>       "SharedMemorySizeMiB": "5"
>   },
>   "image": {
>       "scale": {
>           "mode": "${imageScaleMode}",
>           "width": "${frameWidth}",
>           "height": "${frameHeight}"
>       },
>       "format": {
>           "@type": "#Microsoft.Media.MediaGraphImageFormatEncoded",
>           "encoding": "${imageEncoding}",
>           "quality": "${imageQuality}"
>       }
>   },
>   "inputs": [
>       {
>           "nodeName": "motionDetection"
>       }
>   ]
> }          
> </code></pre>
> </details>    
> </p>

### <a name="generate-and-deploy-the-iot-edge-deployment-manifest"></a>Generación e implementación del manifiesto de implementación de IoT Edge

1. Haga clic con el botón derecho en el archivo *src/edge/* *deployment.grpcyolov3icpu.template.json* y seleccione **Generate IoT Edge Deployment Manifest** (Generar manifiesto de implementación de IoT Edge).

    ![Generación de un manifiesto de implementación de IoT Edge](../../../media/quickstarts/generate-iot-edge-deployment-manifest-grpc.png)

    El archivo de manifiesto *deployment.grpcyolov3icpu.amd64.json* se crea en la carpeta *src/edge/config*.
1. Si ha completado el inicio rápido [Detección de movimiento y emisión de eventos](../../../detect-motion-emit-events-quickstart.md), omita este paso.

    Si no lo ha hecho, cerca del panel **AZURE IOT HUB**, en la esquina inferior izquierda, seleccione el icono **Más acciones** y, después, seleccione **Set IoT Hub Connection String** (Establecer cadena de conexión de IoT Hub). La cadena se puede copiar del archivo *appsettings.json*. O bien, para asegurarse de que ha configurado el centro de IoT adecuado en Visual Studio Code, use el [comando de selección de IoT Hub](https://github.com/Microsoft/vscode-azure-iot-toolkit/wiki/Select-IoT-Hub).

    ![Cadena de conexión de IoT Hub](../../../media/quickstarts/iot-hub-connection-string-grpc.png)
1. Haga clic con el botón derecho en *src/edge/config/* *deployment.grpcyolov3icpu.amd64.json* y seleccione **Create Deployment for Single Device** (Crear una implementación para un dispositivo individual).

    ![Crear una implementación para un dispositivo individual](../../../media/quickstarts/create-deployment-single-device-grpc.png)
1. Cuando se le pida que seleccione un dispositivo IoT Hub, seleccione **lva-sample-device**.
1. Tras aproximadamente 30 segundos, en la esquina inferior izquierda de la ventana, actualice Azure IoT Hub. El dispositivo perimetral ahora muestra los siguientes módulos implementados:

    * El módulo de Live Video Analytics, llamado **lvaEdge**.
    * El módulo **rtspsim**, que simula un servidor RTSP y actúa como el origen de una fuente de vídeo en directo.

        > [!NOTE]
        > Si usa su propio dispositivo perimetral en lugar del proporcionado por el script de instalación, vaya al dispositivo perimetral y ejecute los siguientes comandos con **derechos de administrador**, para extraer y almacenar el archivo de vídeo de ejemplo que se usa para este inicio rápido:  

        ```
        mkdir /home/lvaadmin/samples
        mkdir /home/lvaadmin/samples/input    
        curl https://lvamedia.blob.core.windows.net/public/camera-300s.mkv > /home/lvaadmin/samples/input/camera-300s.mkv  
        chown -R lvaadmin /home/lvaadmin/samples/  
        ```
    * El módulo **lvaExtension**, que es el modelo de detección de objetos YOLOv3 que usa gRPC como método de comunicación y aplica Computer Vision a las imágenes y devuelve varias clases de tipos de objeto.
    
    ![Extensión lva](../../../media/quickstarts/lvaextension-grpc.png)

### <a name="prepare-to-monitor-events"></a>Preparación para supervisar eventos

Haga clic con el botón derecho en el dispositivo de Live Video Analytics y seleccione **Start Monitoring Built-in Event Endpoint** (Iniciar supervisión del punto de conexión de eventos integrado). Este paso es necesario para supervisar los eventos de IoT Hub en la ventana **SALIDA** de Visual Studio Code.

![Iniciar supervisión](../../../media/quickstarts/start-monitoring-built-event-endpoint-grpc.png)

### <a name="run-the-sample-program"></a>Ejecución del programa de ejemplo

1. Para iniciar una sesión de depuración, seleccione la tecla F5. Verá mensajes impresos en la ventana TERMINAL.
1. El código de *operations.json* comienza con llamadas a los métodos directos `GraphTopologyList` y `GraphInstanceList`. Si ha limpiado los recursos tras haber completado los inicios rápidos anteriores, este proceso devolverá listas vacías y, después, se pausará. Para continuar, seleccione la tecla Entrar.
    
    ```
    -------------------------------Executing operation GraphTopologyList-----------------------  
    Request: GraphTopologyList  --------------------------------------------------
    {
    "@apiVersion": "1.0"
    }
    ---------------  
    Response: GraphTopologyList - Status: 200  ---------------
    {
    "value": []
    }
    --------------------------------------------------------------------------
    Executing operation WaitForInput
    Press Enter to continue
    ```
    
    La ventana **TERMINAL** muestra el siguiente conjunto de llamadas al método directo:
    
    * Una llamada a `GraphTopologySet` que utiliza la instancia anterior de topologyUrl.
    * Una llamada a `GraphInstanceSet` que usa el cuerpo siguiente:
    
    ```
    {
      "@apiVersion": "1.0",
      "name": "Sample-Graph-1",
      "properties": {
        "topologyName": "InferencingWithGrpcExtension",
        "description": "Sample graph description",
        "parameters": [
          {
            "name": "rtspUrl",
            "value": "rtsp://rtspsim:554/media/camera-300s.mkv"
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
1. La salida de la ventana **TERMINAL** se pondrá en pausa con el mensaje Press Enter to continue (Presione Entrar para continuar). No seleccione Entrar todavía. Desplácese hacia arriba para ver las cargas de la respuesta JSON para los métodos directos que ha invocado.
1. Cambie a la ventana **SALIDA** de Visual Studio Code. Verá los mensajes que el módulo Live Video Analytics en IoT Edge envía al centro de IoT. En la siguiente sección de este inicio rápido se analizan estos mensajes.
1. El grafo multimedia continúa ejecutándose e imprimiendo los resultados. El simulador RTSP sigue recorriendo el vídeo de origen. Para detener el grafo multimedia, vuelva a la ventana **TERMINAL** y seleccione Entrar.
1. La siguiente serie de llamadas limpia los recursos:
    
    * Una llamada a `GraphInstanceDeactivate` desactiva la instancia del grafo.
    * Una llamada a `GraphInstanceDelete` elimina la instancia.
    * Una llamada a `GraphTopologyDelete` elimina la topología.
    * Una llamada final a `GraphTopologyList` muestra que la lista está vacía.

