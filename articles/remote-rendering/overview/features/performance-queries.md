---
title: Consultas de rendimiento del lado del servidor
description: Cómo realizar consultas de rendimiento del lado servidor mediante llamadas API
author: florianborn71
ms.author: flborn
ms.date: 02/10/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: c7212157bb7ef541ac1eb1753f46ea6ad434c2ca
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89013374"
---
# <a name="server-side-performance-queries"></a>Consultas de rendimiento del lado del servidor

Un buen rendimiento de la representación en el servidor es fundamental para unas velocidades de fotogramas estables y una buena experiencia del usuario. Es importante supervisar atentamente las características del rendimiento en el servidor y optimizarlo cuando sea necesario. Los datos del rendimiento se pueden consultar mediante funciones de API dedicadas.

Lo que más afecta al rendimiento de la representación son los datos de entrada del modelo. Puede ajustar los datos de entrada tal como se describe en [Configuración de la conversión de modelos](../../how-tos/conversion/configure-model-conversion.md).

También puede producirse un cuello de botella en el rendimiento de la aplicación del lado cliente. Para un análisis exhaustivo del rendimiento del lado cliente, se recomienda realizar un [:::no-loc text="performance trace":::](../../how-tos/performance-tracing.md).

## <a name="clientserver-timeline"></a>Escala de tiempo de cliente o servidor

Antes de entrar en detalles con respecto a los distintos valores de latencia, merece la pena echar un vistazo a los puntos de sincronización entre el cliente y el servidor en la escala de tiempo:

![Escala de tiempo de canalización](./media/server-client-timeline.png)

En la ilustración se muestra el proceso:

* El cliente inicia una *estimación de la posición* en una velocidad de fotogramas constante de 60 Hz (cada 16,6 ms).
* Después, el servidor inicia la representación, basándose en la posición.
* El servidor devuelve la imagen de vídeo codificada.
* El cliente descodifica la imagen, realiza algunas tareas de CPU y GPU en ella y muestra la imagen.

## <a name="frame-statistics-queries"></a>Consultas de estadísticas de fotogramas

Las estadísticas de fotogramas ofrecen información de alto nivel, como la latencia, para el último fotograma. Los datos proporcionados en la estructura de `FrameStatistics` se miden en el lado cliente, por lo que la API es una llamada sincrónica:

```cs
void QueryFrameData(AzureSession session)
{
    FrameStatistics frameStatistics;
    if (session.GraphicsBinding.GetLastFrameStatistics(out frameStatistics) == Result.Success)
    {
        // do something with the result
    }
}
```

```cpp
void QueryFrameData(ApiHandle<AzureSession> session)
{
    FrameStatistics frameStatistics;
    if (*session->GetGraphicsBinding()->GetLastFrameStatistics(&frameStatistics) == Result::Success)
    {
        // do something with the result
    }
}
```

El objeto `FrameStatistics` recuperado contiene los siguientes miembros:

| Member | Explicación |
|:-|:-|
| latencyPoseToReceive | La latencia de la estimación de la posición de la cámara en el dispositivo cliente hasta que un fotograma de servidor en esta posición esté totalmente disponible para la aplicación cliente. Este valor incluye los ciclos de ida y vuelta de red, el tiempo de representación del servidor, la descodificación de vídeo y la compensación por vibración. Consulte el **intervalo 1 en la ilustración anterior.**|
| latencyReceiveToPresent | Latencia de la disponibilidad de un fotograma remoto recibido hasta que la aplicación cliente llama a PresentFrame en la CPU. |
| latencyPresentToDisplay  | Latencia desde la presentación de un fotograma en la CPU hasta que la pantalla se enciende. Este valor incluye el tiempo de GPU del cliente, cualquier almacenamiento en búfer de fotogramas realizado por el sistema operativo, la reproyección de hardware y el tiempo de examen de la visualización dependiente del dispositivo. Consulte el **intervalo 2 en la ilustración anterior.**|
| timeSinceLastPresent | El tiempo entre las llamadas posteriores a PresentFrame en la CPU. Unos valores mayores que la duración de la visualización (por ejemplo 16,6 ms en un dispositivo cliente de 60 Hz) indican problemas causados por la aplicación cliente que no finalizan la carga de trabajo de la CPU a tiempo. Consulte el **intervalo 3 de la ilustración anterior.**|
| videoFramesReceived | Número de fotogramas recibidos del servidor en el último segundo. |
| videoFrameReusedCount | Número de fotogramas recibidos en el último segundo que se usaron en el dispositivo más de una vez. Unos valores distintos de cero indican que los fotogramas han tenido que reutilizarse y reproyectarse debido a la vibración de la red o a un tiempo excesivo de representación del servidor. |
| videoFramesSkipped | Número de fotogramas recibidos en el último segundo que se descodificaron, pero que no se mostraron en la pantalla porque llegó un fotograma más reciente. Unos valores distintos de cero indican que la vibración de la red ha provocado que varios fotogramas se retrasaran y que llegaran juntos al dispositivo cliente en una ráfaga. |
| videoFramesDiscarded | Es muy similar a **videoFramesSkipped**, pero el motivo para su descarte es que un fotograma ha llegado tan tarde que ya ni siquiera se puede correlacionar con ninguna posición pendiente. Si esto sucede, hay una contención de red grave.|
| videoFrameMinDelta | Cantidad mínima de tiempo entre dos fotogramas consecutivos que llegan durante el último segundo. Junto con videoFrameMaxDelta, este intervalo proporciona una indicación de vibración causada por el códec de vídeo o de red. |
| videoFrameMaxDelta | Cantidad máxima de tiempo entre dos fotogramas consecutivos que llegan durante el último segundo. Junto con videoFrameMinDelta, este intervalo proporciona una indicación de la vibración causada por el códec de vídeo o de red. |

La suma de todos los valores de latencia suele ser mucho mayor que el tiempo del fotograma disponible a 60 Hz. Esto es correcto, porque varios fotogramas se han lanzado en paralelo y las nuevas solicitudes de fotogramas se inician a la velocidad de fotogramas deseada, tal como se muestra en la ilustración. Sin embargo, si la latencia se vuelve demasiado grande, afecta a la calidad de la [reproyección de la fase más tardía](../../overview/features/late-stage-reprojection.md) y puede poner en peligro la experiencia general.

`videoFramesReceived`, `videoFrameReusedCount` y `videoFramesDiscarded` se pueden usar para medir el rendimiento de la red y del servidor. Si el valor de `videoFramesReceived` es bajo y el de `videoFrameReusedCount` es alto, puede indicar una congestión de la red o un rendimiento deficiente del servidor. Un valor de `videoFramesDiscarded` alto también indica congestión de la red.

Por último,`timeSinceLastPresent`, `videoFrameMinDelta` y `videoFrameMaxDelta` ofrecen una idea de la varianza de los fotogramas de vídeo entrantes y las llamadas presentes locales. Una varianza alta implica una velocidad de fotogramas inestable.

Ninguno de los valores anteriores proporciona una indicación clara de la latencia de red pura (las flechas rojas de la ilustración), ya que el tiempo exacto en el que el servidor está ocupado con la representación debe restarse del valor del ciclo de ida y vuelta `latencyPoseToReceive`. La parte del lado servidor de la latencia total es información que no está disponible para el cliente. Sin embargo, en el siguiente párrafo se explica cómo se aproxima este valor a través de la entrada adicional del servidor y se expone a través del valor de `networkLatency`.

## <a name="performance-assessment-queries"></a>Consultas de valoración del rendimiento

Las *consultas de valoración del rendimiento* proporcionan información más detallada sobre la carga de trabajo de la CPU y GPU en el servidor. Dado que los datos se solicitan desde el servidor, la consulta de una instantánea de rendimiento sigue el patrón asincrónico habitual:

```cs
PerformanceAssessmentAsync _assessmentQuery = null;

void QueryPerformanceAssessment(AzureSession session)
{
    _assessmentQuery = session.Actions.QueryServerPerformanceAssessmentAsync();
    _assessmentQuery.Completed += (PerformanceAssessmentAsync res) =>
    {
        // do something with the result:
        PerformanceAssessment result = res.Result;
        // ...

        _assessmentQuery = null;
    };
}
```

```cpp
void QueryPerformanceAssessment(ApiHandle<AzureSession> session)
{
    ApiHandle<PerformanceAssessmentAsync> assessmentQuery = *session->Actions()->QueryServerPerformanceAssessmentAsync();
    assessmentQuery->Completed([] (ApiHandle<PerformanceAssessmentAsync> res)
    {
        // do something with the result:
        PerformanceAssessment result = res->GetResult();

        // ...

    });
}
```

Al contrario que el objeto `FrameStatistics`, el objeto `PerformanceAssessment` contiene información del servidor:

| Member | Explicación |
|:-|:-|
| timeCPU | Tiempo medio de CPU de servidor por fotograma en milisegundos |
| timeGPU | Tiempo medio de GPU de servidor por fotograma en milisegundos |
| utilizationCPU | Utilización total de CPU del servidor en porcentaje |
| utilizationGPU | Utilización total de GPU del servidor en porcentaje |
| memoryCPU | Uso total de la memoria principal del servidor en porcentaje |
| memoryGPU | Uso total de la memoria de vídeo dedicada en porcentaje de GPU del servidor |
| networkLatency | La latencia media aproximada de la red del ciclo de ida y vuelta en milisegundos. En la ilustración anterior, se corresponde a la suma de las flechas rojas. El valor se calcula mediante la resta del tiempo de representación real del servidor del valor de `latencyPoseToReceive` de `FrameStatistics`. Aunque esta aproximación no es precisa, proporciona alguna indicación de la latencia de red, aislada de los valores de latencia calculados en el cliente. |
| polygonsRendered | Número de triángulos representados en un fotograma. Este número también incluye los triángulos que se seleccionan más adelante durante la representación. Esto significa que este número no varía demasiado en las distintas posiciones de la cámara, pero el rendimiento puede variar drásticamente en función de la tasa de selección del triángulo.|

Para ayudarle a evaluar los valores, cada parte incluye una clasificación de calidad como **Excelente**, **Bueno**, **Mediocre** o **Deficiente**.
Esta métrica de evaluación proporciona una indicación aproximada del estado del servidor, pero no debe considerarse absoluta. Por ejemplo, supongamos que ve una puntuación "Mediocre" para el tiempo de GPU. Se considera mediocre porque se acerca al límite del presupuesto general del tiempo de los fotogramas. Sin embargo, en su caso, podría ser un buen valor, ya que se está representando un modelo complejo.

## <a name="statistics-debug-output"></a>Salida de depuración de estadísticas

La clase `ARRServiceStats` es una clase de C# que se encapsula en torno a las consultas de estadísticas de fotogramas y de evaluación del rendimiento y proporciona una práctica funcionalidad para devolver estadísticas como valores agregados o como una cadena pregenerada. El código siguiente es la forma más fácil de mostrar las estadísticas del lado servidor en la aplicación cliente.

```cs
ARRServiceStats _stats = null;

void OnConnect()
{
    _stats = new ARRServiceStats();
}

void OnDisconnect()
{
    _stats = null;
}

void Update()
{
    if (_stats != null)
    {
        // update once a frame to retrieve new information and build average values
        _stats.Update(Service.CurrentActiveSession);

        // retrieve a string with relevant stats information
        InfoLabel.text = _stats.GetStatsString();
    }
}
```

El código anterior rellena la etiqueta de texto con el texto siguiente:

![Salida de la cadena ArrServiceStats](./media/arr-service-stats.png)

La API `GetStatsString` da formato a una cadena de todos los valores, pero cada valor único también se puede consultar mediante programación desde la instancia de `ARRServiceStats`.

También hay variantes de los miembros, que agregan los valores a lo largo del tiempo. Vea los miembros con sufijo `*Avg`, `*Max` o `*Total`. El miembro `FramesUsedForAverage` indica el número de fotogramas que se han utilizado para esta agregación.

## <a name="next-steps"></a>Pasos siguientes

* [Creación de seguimientos del rendimiento en el lado servidor](../../how-tos/performance-tracing.md)
* [Configuración de la conversión de modelos](../../how-tos/conversion/configure-model-conversion.md)
