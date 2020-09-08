---
title: 'Análisis de vídeos casi en tiempo real: Computer Vision'
titleSuffix: Azure Cognitive Services
description: Aprenda a realizar análisis casi en tiempo real de fotogramas procedentes de una secuencia de vídeo en directo mediante la API Computer Vision.
services: cognitive-services
author: KellyDF
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: sample
ms.date: 09/09/2019
ms.author: kefre
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: c86b06ff46f1ddc8d22d2ab7ec4bc8620a8c862f
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933178"
---
# <a name="analyze-videos-in-near-real-time"></a>Análisis de vídeo casi en tiempo real

En este artículo se muestra cómo realizar análisis casi en tiempo real de fotogramas procedentes de una secuencia de vídeo en directo mediante la API Computer Vision. Los elementos básicos de este análisis son:

- Obtención de fotogramas desde un origen de vídeo.
- Selección de los fotogramas que se van a analizar.
- Envío de estos fotogramas a la API.
- Consumo de todos los resultados del análisis que se devuelven de la llamada API.

Los ejemplos de este artículo están escritos en C#. Para acceder al código, vaya a la página [Ejemplo de análisis de fotogramas de vídeo](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) en GitHub.

## <a name="approaches-to-running-near-real-time-analysis"></a>Enfoques para ejecutar análisis casi en tiempo real

Se pueden usar varios enfoques para solucionar el problema de la ejecución de análisis casi en tiempo real en secuencias de vídeo. En este artículo se describen tres de ellos, de menos a más sofisticado.

### <a name="design-an-infinite-loop"></a>Diseño de un bucle infinito

El diseño más sencillo para el análisis casi en tiempo real es un bucle infinito. En cada iteración de este bucle, se obtiene un fotograma, se analiza y, después, se consume el resultado:

```csharp
while (true)
{
    Frame f = GrabFrame();
    if (ShouldAnalyze(f))
    {
        AnalysisResult r = await Analyze(f);
        ConsumeResult(r);
    }
}
```

Si su análisis tuviera que consistir en un algoritmo de cliente ligero, este enfoque sería adecuado. Sin embargo, cuando el análisis se produce en la nube, la latencia resultante implica que una llamada API podría tardar varios segundos. Durante este tiempo no se capturan imágenes y el subproceso básicamente no realiza ninguna acción. La velocidad de fotogramas máxima está limitada por la latencia de las llamadas API.

### <a name="allow-the-api-calls-to-run-in-parallel"></a>Permitir la ejecución en paralelo de las llamadas API

Aunque se puede usar un bucle simple de un único subproceso en un algoritmo de cliente ligero, no se ajusta bien a la latencia de una llamada API en la nube. La solución a este problema es permitir que la llamada API de ejecución prolongada se ejecute en paralelo con la captura de los fotogramas. En C#, esto se puede hacer mediante un paralelismo basado en tareas. Por ejemplo, puede ejecutar el siguiente código:

```csharp
while (true)
{
    Frame f = GrabFrame();
    if (ShouldAnalyze(f))
    {
        var t = Task.Run(async () =>
        {
            AnalysisResult r = await Analyze(f);
            ConsumeResult(r);
        }
    }
}
```

Con este enfoque, cada análisis se inicia en una tarea independiente. La tarea puede ejecutarse en segundo plano mientras se siguen capturando nuevos fotogramas. El enfoque evita el bloqueo del subproceso principal mientras se espera la devolución de una llamada API. Sin embargo, el enfoque puede presentar ciertas desventajas:
* Desaparecen algunas de las garantías que proporcionaba la versión simple. En otras palabras, se pueden producir varias llamadas API en paralelo y los resultados pueden devolverse en el orden equivocado. 
* También podría suceder que varios subprocesos especificaran la función ConsumeResult() al mismo tiempo, lo que podría ser peligroso si la función no es segura para subprocesos. 
* Por último, este código simple no realiza un seguimiento de las tareas que se crean, por lo que las excepciones desaparecen de manera silenciosa. Por consiguiente, es preciso agregar un subproceso "consumidor" que realice un seguimiento de las tareas del análisis, genere excepciones, termine tareas de larga duración y garantice que los resultados se consumen en el orden correcto y de uno en uno.

### <a name="design-a-producer-consumer-system"></a>Diseño de un sistema productor-consumidor

Para el enfoque final, el diseño de un sistema "productor-consumidor", se crea un subproceso de productor que es similar al bucle infinito mencionado anteriormente. Sin embargo, en lugar de consumir los resultados del análisis en cuanto están disponibles, el productor simplemente coloca las tareas en una cola para hacer un seguimiento de ellas.

```csharp
// Queue that will contain the API call tasks.
var taskQueue = new BlockingCollection<Task<ResultWrapper>>();

// Producer thread.
while (true)
{
    // Grab a frame.
    Frame f = GrabFrame();

    // Decide whether to analyze the frame.
    if (ShouldAnalyze(f))
    {
        // Start a task that will run in parallel with this thread.
        var analysisTask = Task.Run(async () =>
        {
            // Put the frame, and the result/exception into a wrapper object.
            var output = new ResultWrapper(f);
            try
            {
                output.Analysis = await Analyze(f);
            }
            catch (Exception e)
            {
                output.Exception = e;
            }
            return output;
        }

        // Push the task onto the queue.
        taskQueue.Add(analysisTask);
    }
}
```

También se crea un subproceso de consumidor, que saca las tareas de la cola, espera a que finalicen y muestra el resultado, o bien genera la excepción que se ha iniciado. El uso de la cola permite garantizar que los resultados se consumen uno a uno, en el orden correcto y sin limitar la velocidad de fotogramas máxima del sistema.

```csharp
// Consumer thread.
while (true)
{
    // Get the oldest task.
    Task<ResultWrapper> analysisTask = taskQueue.Take();
 
    // Wait until the task is completed.
    var output = await analysisTask;

    // Consume the exception or result.
    if (output.Exception != null)
    {
        throw output.Exception;
    }
    else
    {
        ConsumeResult(output.Analysis);
    }
}
```

## <a name="implement-the-solution"></a>Implementación de la solución

### <a name="get-started-quickly"></a>Introducción rápida

Para ayudarle a poner en marcha su aplicación lo antes posible, hemos implementado el sistema que se describe en la sección anterior. Se pretende que tenga la flexibilidad suficiente para albergar muchos escenarios, aunque es fácil de usar. Para acceder al código, vaya a la página [Ejemplo de análisis de fotogramas de vídeo](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) en GitHub.

La biblioteca contiene la clase `FrameGrabber`, que implementa el sistema productor-consumidor que se ha descrito anteriormente para procesar los fotogramas de vídeo de una cámara web. Los usuarios pueden especificar la forma exacta de la llamada API y la clase usa eventos para que el código de la llamada sepa cuándo se adquiere un nuevo fotograma o cuándo hay disponible un nuevo resultado del análisis.

Para ilustrar algunas de las posibilidades, se proporcionan dos aplicaciones de ejemplo que utilizan la biblioteca. 

La primera es una aplicación de consola sencilla que captura fotogramas de la cámara web predeterminada y los envía al servicio Face para la detección de caras. En el código siguiente se reproduce una versión simplificada de la aplicación:

```csharp
using System;
using System.Linq;
using Microsoft.Azure.CognitiveServices.Vision.Face;
using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
using VideoFrameAnalyzer;

namespace BasicConsoleSample
{
    internal class Program
    {
        const string ApiKey = "<your API key>";
        const string Endpoint = "https://<your API region>.api.cognitive.microsoft.com";

        private static async Task Main(string[] args)
        {
            // Create grabber.
            FrameGrabber<DetectedFace[]> grabber = new FrameGrabber<DetectedFace[]>();

            // Create Face Client.
            FaceClient faceClient = new FaceClient(new ApiKeyServiceClientCredentials(ApiKey))
            {
                Endpoint = Endpoint
            };

            // Set up a listener for when we acquire a new frame.
            grabber.NewFrameProvided += (s, e) =>
            {
                Console.WriteLine($"New frame acquired at {e.Frame.Metadata.Timestamp}");
            };

            // Set up a Face API call.
            grabber.AnalysisFunction = async frame =>
            {
                Console.WriteLine($"Submitting frame acquired at {frame.Metadata.Timestamp}");
                // Encode image and submit to Face service.
                return (await faceClient.Face.DetectWithStreamAsync(frame.Image.ToMemoryStream(".jpg"))).ToArray();
            };

            // Set up a listener for when we receive a new result from an API call.
            grabber.NewResultAvailable += (s, e) =>
            {
                if (e.TimedOut)
                    Console.WriteLine("API call timed out.");
                else if (e.Exception != null)
                    Console.WriteLine("API call threw an exception.");
                else
                    Console.WriteLine($"New result received for frame acquired at {e.Frame.Metadata.Timestamp}. {e.Analysis.Length} faces detected");
            };

            // Tell grabber when to call the API.
            // See also TriggerAnalysisOnPredicate
            grabber.TriggerAnalysisOnInterval(TimeSpan.FromMilliseconds(3000));

            // Start running in the background.
            await grabber.StartProcessingCameraAsync();

            // Wait for key press to stop.
            Console.WriteLine("Press any key to stop...");
            Console.ReadKey();

            // Stop, blocking until done.
            await grabber.StopProcessingAsync();
        }
    }
}
```

La segunda aplicación de ejemplo es algo más interesante. Permite elegir la API a la que se llama en los fotogramas de vídeo. A la izquierda, la aplicación muestra una vista previa del vídeo en directo. A la derecha, superpone el resultado de la API más reciente en el fotograma correspondiente.

En la mayoría de los modos, hay retraso visible entre el vídeo en directo de la izquierda y el análisis que se ve a la derecha. Este retraso es el tiempo que tarda en realizarse la llamada API. Una excepción es el modo "EmotionsWithClientFaceDetect", que realiza la detección de caras localmente en el equipo cliente mediante OpenCV antes de enviar las imágenes a Azure Cognitive Services. 

Con este enfoque, puede visualizar la cara detectada de inmediato. Luego, puede actualizar las emociones, después de que se devuelva la llamada API. Esto demuestra la posibilidad de un enfoque "híbrido", es decir, se puede realizar un procesamiento simple en el cliente y, después, pueden usarse Cognitive Services APIs para intensificar este procesamiento con un análisis más avanzado si es necesario.

![La aplicación LiveCameraSample muestra una imagen con etiquetas](../../Video/Images/FramebyFrame.jpg)

### <a name="integrate-the-samples-into-your-codebase"></a>Integración de los ejemplos en el código base

Para empezar con este ejemplo, realice estas acciones:

1. Cree una [cuenta de Azure](https://azure.microsoft.com/free/cognitive-services/). Si ya tiene una, puede pasar a la siguiente sección.
2. Cree recursos para Computer Vision y Face en Azure Portal para obtener la clave y el punto de conexión. Asegúrese de seleccionar el nivel gratis (F0) durante la instalación.
   - [Computer Vision](https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision)
   - [Face](https://portal.azure.com/#create/Microsoft.CognitiveServicesFace) Después de implementar los recursos, haga clic en **Ir al recurso** para recopilar la clave y el punto de conexión de cada recurso. 
3. Clone el repositorio [Cognitive-Samples-VideoFrameAnalysis](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/) de GitHub.
4. Abra el ejemplo en Visual Studio 2015, o cualquier versión posterior, y después, compile y ejecute las aplicaciones de ejemplo:
    - Para BasicConsoleSample, la clave de Face está codificada de forma rígida directamente en [BasicConsoleSample/Program.cs](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/blob/master/Windows/BasicConsoleSample/Program.cs).
    - En el caso de LiveCameraSample, escriba las claves en el panel **Configuración** de la aplicación. Las claves se mantienen de una sesión a otra como datos de usuario.

Cuando esté listo para integrar los ejemplos, haga referencia a la biblioteca VideoFrameAnalyzer desde sus propios proyectos.

Las funcionalidades de comprensión de imágenes, voces, vídeo y texto de VideoFrameAnalyzer utilizan Azure Cognitive Services. Microsoft recibirá las imágenes, el audio, el vídeo y los demás datos que cargue (a través de esta aplicación) y puede usarlos para mejorar el servicio. Pedimos su ayuda para proteger a las personas cuyos datos envíe la aplicación a Azure Cognitive Services.

## <a name="summary"></a>Resumen

En este artículo, ha aprendido a ejecutar análisis casi en tiempo real de secuencias de vídeo en directo mediante los servicios Face y Computer Vision. También ha aprendido a usar el código de ejemplo para empezar.

No dude en realizar comentarios y sugerencias en el [repositorio de GitHub](https://github.com/Microsoft/Cognitive-Samples-VideoFrameAnalysis/). Para proporcionar comentarios de API más amplios, visite nuestro [sitio de UserVoice](https://cognitive.uservoice.com/).

