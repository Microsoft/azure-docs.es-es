---
title: Concepto de grafo multimedia en Azure
description: Un grafo multimedia le permite definir dónde se debe capturar el elemento multimedia, cómo se debe procesar y dónde se deben entregar los resultados. En este artículo se ofrece una descripción detallada del concepto de grafo multimedia.
ms.topic: conceptual
ms.date: 05/01/2020
ms.openlocfilehash: 6be741ee38cc8f1980fe9aa96883f9aacc1be8e2
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89048436"
---
# <a name="media-graph"></a>Grafo multimedia

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura

* [Introducción a Análisis de vídeos en vivo en IoT Edge](overview.md)
* [Terminología de Análisis de vídeos en vivo en IoT Edge](terminology.md)

## <a name="overview"></a>Información general

Un grafo multimedia le permite definir dónde se debe capturar el elemento multimedia, cómo se debe procesar y dónde se deben entregar los resultados. Para ello, conecte componentes o nodos de la manera deseada. En el diagrama siguiente se muestra una representación gráfica de un grafo multimedia.  

![Una representación gráfica de un grafo multimedia](./media/media-graph/overview.png)

Análisis de vídeos en vivo en IoT Edge admite diferentes tipos de orígenes, procesadores y receptores.

* Los **nodos de origen** permiten capturar elementos multimedia en el grafo multimedia. Los elementos multimedia en este contexto, conceptualmente, pueden ser una secuencia de audio, una secuencia de vídeo, un flujo de datos o una secuencia que tiene audio, vídeo o datos combinados en un solo flujo.
* Los **nodos de procesador** permiten el procesamiento de elementos multimedia dentro del grafo multimedia.
* Los **nodos receptores** permiten entregar los resultados de procesamiento a servicios y aplicaciones fuera del grafo multimedia.

## <a name="media-graph-topologies-and-instances"></a>Topologías e instancias de los grafos multimedia 

El Análisis de vídeos en vivo en IoT Edge permite administrar grafos multimedia con dos conceptos: "topología de grafo" e "instancia de grafo". Una topología de grafo permite definir el plano técnico de un grafo, con parámetros como marcadores de posición para los valores. La topología define qué nodos se usan en el grafo multimedia y cómo se conectan dentro de dicho grafo. Por ejemplo, si desea grabar la fuente desde una cámara, necesitaría un grafo con un nodo de origen que reciba vídeo y un nodo receptor que escriba el vídeo.

Los valores de los parámetros de la topología se especifican al crear instancias de grafo que hacen referencia a la topología. Esto permite crear varias instancias que hacen referencia a la misma topología, pero con valores diferentes para los parámetros especificados en ella. En el ejemplo anterior, podría haber usado parámetros para representar la dirección IP de la cámara y el nombre del vídeo grabado. Puede crear muchas instancias de grafos con esa topología: una instancia para cada cámara de un edificio, a ser posible, cada una con una dirección IP y un nombre específicos.

## <a name="media-graph-states"></a>Estados de los grafos multimedia  

El ciclo de vida de las topologías de grafos y las instancias de grafos se muestra en el siguiente diagrama de estado.

![Ciclo de vida de la topología de grafo e instancia de grafo](./media/media-graph/graph-topology-lifecycle.svg)

Comience por [crear una topología de grafo](direct-methods.md#graphtopologyset). A continuación, para cada fuente de vídeo en directo que desee procesar con esta topología, [cree una instancia de grafo](direct-methods.md#graphinstanceset). 

La instancia de grafo estará en el estado `Inactive` (inactivo).

Cuando esté listo para enviar la fuente de vídeo en directo a la instancia de grafo, [actívela](direct-methods.md#graphinstanceactivate). La instancia de grafo pasará brevemente a través de un estado `Activating` de transición y, si se realiza correctamente, entrará en un estado `Active`. En el estado `Active`, se procesarán los elementos multimedia (si la instancia de grafo recibe datos de entrada).

> [!NOTE]
>  Una instancia de grafo puede estar activa sin que los datos fluyan a través de ella (por ejemplo, la cámara se queda sin conexión).
> La suscripción de Azure se facturará cuando la instancia de grafo esté en estado activo.

Puede repetir el proceso de creación y activación de otras instancias de grafo para la misma topología, si tiene otras fuentes de vídeo en directo para procesar.

Cuando haya terminado de procesar la fuente de vídeo en directo, puede [desactivar](direct-methods.md#graphinstancedeactivate) la instancia de grafo. La instancia de grafo pasará brevemente a través de un estado `Deactivating` de transición, vaciará todos los datos que tenga y, a continuación, volverá al estado `Inactive`.

Solo puede [eliminar](direct-methods.md#graphinstancedelete) una instancia de grafo cuando esté en el estado `Inactive`.

Una vez eliminadas todas las instancias de grafo que hacen referencia a una topología de grafo específica, puede [eliminar la topología de grafo](direct-methods.md#graphtopologydelete).


## <a name="sources-processors-and-sinks"></a>Orígenes, procesadores y receptores  

Análisis de vídeos en vivo en IoT Edge admite los siguientes tipos de nodos dentro de un grafo multimedia:

### <a name="sources"></a>Orígenes 

#### <a name="rtsp-source"></a>Origen RTSP 

Un nodo de origen RTSP permite ingerir elementos multimedia desde un servidor [RTSP] (https://tools.ietf.org/html/rfc2326 ). La vigilancia y las cámaras basadas en IP transmiten sus datos en un protocolo denominado RTSP (protocolo de transmisión en tiempo real) que es diferente de otros tipos de dispositivos, como teléfonos y cámaras de vídeo. Este protocolo se usa para establecer y controlar las sesiones multimedia entre un servidor (la cámara) y un cliente. El nodo de origen RTSP en un grafo multimedia actúa como un cliente y puede establecer una sesión con un servidor RTSP. Muchos dispositivos, como la mayoría de las [cámaras IP](https://en.wikipedia.org/wiki/IP_camera) tienen un servidor RTSP integrado. [ONVIF](https://www.onvif.org/) exigirá que se admita RTSP en la definición de los dispositivos compatibles con los [perfiles G, S y T](https://www.onvif.org/wp-content/uploads/2019/12/ONVIF_Profile_Feature_overview_v2-3.pdf). El nodo de origen RTSP requiere que especifique una dirección URL de RTSP, junto con las credenciales para habilitar una conexión autenticada.

#### <a name="iot-hub-message-source"></a>Origen de mensajes de IoT Hub 

Al igual que otros [módulos de IoT Edge](../../iot-edge/iot-edge-glossary.md#iot-edge-module), el módulo Análisis de vídeos en vivo en IoT Edge puede recibir mensajes a través del [centro de IoT Edge](../../iot-edge/iot-edge-glossary.md#iot-edge-hub). Estos mensajes se pueden enviar desde otros módulos o aplicaciones que se ejecutan en el dispositivo perimetral o desde la nube. Estos mensajes se entregan (enrutan) a una [entrada con nombre](../../iot-edge/module-composition.md#sink) en el módulo. Un nodo de origen de mensajes de IoT Hub permite que dichos mensajes lleguen a un grafo multimedia. Estos mensajes o señales se pueden usar internamente en el grafo multimedia, normalmente para activar las puertas de señales (vea [puertas de señales](#signal-gate-processor) a continuación). 

Por ejemplo, puede tener un módulo de IoT Edge que genera un mensaje cuando se abre una puerta. El mensaje de ese módulo se puede enrutar al centro de IoT Edge, desde donde se puede enrutar al origen de los mensajes del centro de IoT de un grafo multimedia. Dentro del grafo multimedia, el origen de los mensajes del centro de IoT puede pasar el evento a un procesador de la puerta de señales, que puede activar la grabación del vídeo desde un origen RTSP en un archivo. 

### <a name="processors"></a>Procesadores  

#### <a name="motion-detection-processor"></a>Procesador de detección de movimiento 

El nodo del procesador de detección de movimiento permite detectar movimiento en vídeos en directo. Examina los fotogramas de vídeo entrantes y determina si hay movimiento en el vídeo. Si se detecta movimiento, pasa los fotogramas de vídeo al componente de nivel inferior y emite un evento. El nodo del procesador de detección de movimiento (junto con otros nodos) se puede usar para desencadenar la grabación del vídeo entrante cuando se detecta movimiento.

#### <a name="frame-rate-filter-processor"></a>Procesador de filtros de velocidad de fotogramas  

El nodo del procesador de filtros de velocidad de fotogramas permite muestrear los fotogramas de la secuencia de vídeo entrante a una velocidad especificada. Esto permite reducir el número de fotogramas que se envían a los componentes de nivel inferior (como el nodo del procesador de extensiones HTTP) para su posterior procesamiento.

#### <a name="http-extension-processor"></a>Procesador de extensiones HTTP

El nodo del procesador de extensiones HTTP permite conectar su propio módulo de IoT Edge a un grafo multimedia. Este nodo adopta los fotogramas de vídeo descodificados como entrada y los retransmite a un punto de conexión de HTTP REST expuesto por el módulo. Este nodo tiene la capacidad de autenticarse con el punto de conexión de REST, si es necesario. Además, el nodo tiene un formateador de imagen integrado para escalar y codificar los fotogramas de vídeo antes de retransmitirlos al punto de conexión de REST. El escalador tiene opciones para conservar, rellenar o ajustar la relación de aspecto de la imagen. El codificador de imágenes admite los formatos JPEG, PNG o BMP.

#### <a name="grpc-extension-processor"></a>Procesador de extensiones gRPC

Este nodo procesador de extensiones gRPC toma los fotogramas de vídeo descodificados como entrada y los retransmite a un punto de conexión [gRPC](terminology.md#grpc) expuesto por el módulo. Además, el nodo tiene un formateador de imagen integrado para escalar y codificar los fotogramas de vídeo antes de retransmitirlos al punto de conexión gRPC. El escalador tiene opciones para conservar, rellenar o ajustar la relación de aspecto de la imagen. El codificador de imágenes admite los formatos jpeg, png o bmp.

#### <a name="signal-gate-processor"></a>Procesador de la puerta de señales  

El nodo del procesador de la puerta de señales permite reenviar de manera condicional elementos multimedia de un nodo a otro. También actúa como un búfer, lo que permite sincronizar elementos multimedia y eventos. Un caso de uso típico consiste en insertar un nodo del procesador de la puerta de señales entre el nodo de origen RTSP y el nodo receptor de recursos, con el uso de la salida de un nodo del procesador de detección de movimiento para desencadenar la puerta. Con este tipo de grafo multimedia, solo se graba el vídeo cuando se detecta movimiento.

### <a name="sinks"></a>Receptores  

#### <a name="asset-sink"></a>Receptor de recursos  

Un nodo receptor de recursos permite escribir datos de elementos multimedia (vídeo o audio) en un recurso de Azure Media Services. Solo puede haber un nodo receptor de recursos en un grafo multimedia. Consulte la sección de [recursos](terminology.md#asset) para obtener más información sobre los recursos y su rol en la grabación y la reproducción de elementos multimedia. También puede ver el artículo sobre la [grabación continua de vídeo](continuous-video-recording-concept.md) para obtener más información sobre cómo se usan las propiedades de este nodo.

#### <a name="file-sink"></a>Receptor de archivos  

El nodo receptor de archivos permite escribir datos de elementos multimedia (vídeo o audio) en una ubicación del sistema de archivos local del dispositivo IoT Edge. Solo puede haber un nodo receptor de archivos en un grafo multimedia y debe ser de un nivel inferior a un nodo del procesador de la puerta de señales. Esto limita la duración de los archivos de salida a los valores especificados en las propiedades del nodo del procesador de la puerta de señales.

#### <a name="iot-hub-message-sink"></a>Receptor de mensajes de IoT Hub  

Un nodo receptor de mensajes de IoT Hub permite publicar eventos en el centro de IoT Edge. Después, el centro de IoT Edge puede enrutar los datos a otros módulos o aplicaciones del dispositivo perimetral o a IoT Hub en la nube (según las rutas especificadas en el manifiesto de implementación). El nodo receptor de mensajes de IoT Hub puede aceptar eventos de procesadores de nivel superior como un nodo del procesador de detección de movimiento o de un servicio de inferencia externo a través de un nodo de procesador de extensiones HTTP.

## <a name="rules-on-the-use-of-nodes"></a>Reglas sobre el uso de los nodos

Consulte las [limitaciones de las topologías de grafo](quotas-limitations.md#limitations-on-graph-topologies-at-preview) para obtener más reglas sobre cómo se pueden usar diferentes nodos dentro de un grafo multimedia.

## <a name="scenarios"></a>Escenarios

Mediante una combinación de los orígenes, procesadores y receptores definidos anteriormente, puede crear grafos multimedia para diversos escenarios que implican el análisis de vídeos en vivo. Los escenarios de ejemplo son:

* [Grabación continua de vídeo](continuous-video-recording-concept.md)
* [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)
* [Análisis de vídeos en vivo sin grabación de vídeo](analyze-live-video-concept.md)

## <a name="next-steps"></a>Pasos siguientes

Para ver cómo puede ejecutar la detección de movimiento en una fuente de vídeo en directo, vea [Inicio rápido: Ejecución de Análisis de vídeos en vivo con su propio modelo](use-your-model-quickstart.md).
