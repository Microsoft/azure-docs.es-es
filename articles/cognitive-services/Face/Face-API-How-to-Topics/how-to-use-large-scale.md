---
title: 'Ejemplo: Usar la característica a gran escala: Face'
titleSuffix: Azure Cognitive Services
description: Esta guía es un artículo sobre cómo escalar verticalmente a partir de los objetos existentes PersonGroup y FaceList a los objetos LargePersonGroup y LargeFaceList.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: sample
ms.date: 05/01/2019
ms.author: sbowles
ms.custom: devx-track-csharp
ms.openlocfilehash: c4667b810e4d3d5f13e13572e2420880da5a8a0b
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88931631"
---
# <a name="example-use-the-large-scale-feature"></a>Ejemplo: Usar la característica a gran escala

Esta guía es un artículo avanzado sobre cómo escalar verticalmente a partir de los objetos existentes PersonGroup y FaceList a los objetos LargePersonGroup y LargeFaceList, respectivamente. En esta guía se muestra el proceso de migración. Se presuponen conocimientos básicos de los objetos PersonGroup y FaceList, de la operación [Train](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ae2d16ac60f11b48b5aa4) y de las funciones de reconocimiento facial. Para más información sobre estos temas, consulte la guía conceptual [Reconocimiento facial](../concepts/face-recognition.md).

LargePersonGroup y LargeFaceList se conocen colectivamente como operaciones a gran escala. LargePersonGroup puede contener hasta un millón de personas, cada una de las cuales con un máximo de 248 caras. LargeFaceList puede contener hasta un millón de caras. Las operaciones a gran escala son similares a las de PersonGroup y FaceList convencionales, pero tienen algunas diferencias debido a la arquitectura nueva. 

Los ejemplos se escriben en C# mediante el uso de la biblioteca cliente de Face de Azure Cognitive Services.

> [!NOTE]
> Para habilitar la búsqueda de caras para Identification y FindSimilar a gran escala, incorpore una operación Train para preprocesar LargeFaceList y LargePersonGroup. El tiempo de entrenamiento varía de unos segundos a una media hora de acuerdo con la capacidad real. Durante el período de entrenamiento, es posible realizar las operaciones Identification y FindSimilar si antes se realizó correctamente un entrenamiento. El inconveniente es que las personas y caras nuevas agregadas no aparecen en el resultado hasta que se complete una nueva migración posterior al entrenamiento a gran escala.

## <a name="step-1-initialize-the-client-object"></a>Paso 1: Inicialización del objeto de cliente

Cuando usa la biblioteca cliente de Face, la clave de suscripción y el punto de conexión de la suscripción se pasan a través del constructor de la clase FaceClient. Por ejemplo:

```csharp
string SubscriptionKey = "<Subscription Key>";
// Use your own subscription endpoint corresponding to the subscription key.
string SubscriptionEndpoint = "https://westus.api.cognitive.microsoft.com";
private readonly IFaceClient faceClient = new FaceClient(
            new ApiKeyServiceClientCredentials(subscriptionKey),
            new System.Net.Http.DelegatingHandler[] { });
faceClient.Endpoint = SubscriptionEndpoint
```

Para obtener la clave de suscripción con su punto de conexión correspondiente, vaya a Azure Marketplace desde Azure Portal.
Para más información, consulte [Suscripciones](https://azure.microsoft.com/services/cognitive-services/directory/vision/).

## <a name="step-2-code-migration"></a>Paso 2: Migración del código

Esta sección se centra en cómo migrar la implementación de PersonGroup o FaceList a LargePersonGroup o LargeFaceList. Aunque LargePersonGroup o LargeFaceList difieren de PersonGroup o FaceList en el diseño y la implementación interna, las interfaces de la API son similares por motivos de compatibilidad con versiones anteriores.

No se admite la migración de datos. En su lugar, puede volver a crear LargePersonGroup o LargeFaceList.

### <a name="migrate-a-persongroup-to-a-largepersongroup"></a>Migración de PersonGroup a LargePersonGroup

Migrar de PersonGroup a LargePersonGroup es una operación sencilla. Comparten exactamente las mismas operaciones de nivel de grupo.

Para la implementación de PersonGroup (o una implementación relacionada con las personas), solo es necesario cambiar las rutas de acceso de las API o la clase/módulo del SDK a LargePersonGroup y LargePersonGroup Person.

Agregue todas las caras y las personas de PersonGroup al nuevo LargePersonGroup. Para más información, consulte [Agregar caras](how-to-add-faces.md).

### <a name="migrate-a-facelist-to-a-largefacelist"></a>Migración de FaceList y LargeFaceList

| API de FaceList | API de LargeFaceList |
|:---:|:---:|
| Crear | Crear |
| Eliminar | Eliminar |
| Obtener | Obtener |
| List | List |
| Actualizar | Actualizar |
| - | Train |
| - | Get Training Status |

La tabla anterior es una comparación de las operaciones en el nivel de lista entre FaceList y LargeFaceList. Tal y como se muestra, LargeFaceList incluye nuevas operaciones (Train y Get Training Status) en comparación con FaceList. Entrenar LargeFaceList es una condición previa de la operación [FindSimilar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237). El entrenamiento no es necesario para FaceList. El fragmento de código siguiente es una función auxiliar para esperar el entrenamiento de LargeFaceList:

```csharp
/// <summary>
/// Helper function to train LargeFaceList and wait for finish.
/// </summary>
/// <remarks>
/// The time interval can be adjusted considering the following factors:
/// - The training time which depends on the capacity of the LargeFaceList.
/// - The acceptable latency for getting the training status.
/// - The call frequency and cost.
///
/// Estimated training time for LargeFaceList in different scale:
/// -     1,000 faces cost about  1 to  2 seconds.
/// -    10,000 faces cost about  5 to 10 seconds.
/// -   100,000 faces cost about  1 to  2 minutes.
/// - 1,000,000 faces cost about 10 to 30 minutes.
/// </remarks>
/// <param name="largeFaceListId">The Id of the LargeFaceList for training.</param>
/// <param name="timeIntervalInMilliseconds">The time interval for getting training status in milliseconds.</param>
/// <returns>A task of waiting for LargeFaceList training finish.</returns>
private static async Task TrainLargeFaceList(
    string largeFaceListId,
    int timeIntervalInMilliseconds = 1000)
{
    // Trigger a train call.
    await FaceClient.LargeTrainLargeFaceListAsync(largeFaceListId);

    // Wait for training finish.
    while (true)
    {
        Task.Delay(timeIntervalInMilliseconds).Wait();
        var status = await faceClient.LargeFaceList.TrainAsync(largeFaceListId);

        if (status.Status == Status.Running)
        {
            continue;
        }
        else if (status.Status == Status.Succeeded)
        {
            break;
        }
        else
        {
            throw new Exception("The train operation is failed!");
        }
    }
}
```

Anteriormente, un uso típico de FaceList con caras agregadas y FindSimilar tenía este aspecto:

```csharp
// Create a FaceList.
const string FaceListId = "myfacelistid_001";
const string FaceListName = "MyFaceListDisplayName";
const string ImageDir = @"/path/to/FaceList/images";
faceClient.FaceList.CreateAsync(FaceListId, FaceListName).Wait();

// Add Faces to the FaceList.
Parallel.ForEach(
    Directory.GetFiles(ImageDir, "*.jpg"),
    async imagePath =>
        {
            using (Stream stream = File.OpenRead(imagePath))
            {
                await faceClient.FaceList.AddFaceFromStreamAsync(FaceListId, stream);
            }
        });

// Perform FindSimilar.
const string QueryImagePath = @"/path/to/query/image";
var results = new List<SimilarPersistedFace[]>();
using (Stream stream = File.OpenRead(QueryImagePath))
{
    var faces = faceClient.Face.DetectWithStreamAsync(stream).Result;
    foreach (var face in faces)
    {
        results.Add(await faceClient.Face.FindSimilarAsync(face.FaceId, FaceListId, 20));
    }
}
```

En la migración a LargeFaceList, se convierte en lo siguiente:

```csharp
// Create a LargeFaceList.
const string LargeFaceListId = "mylargefacelistid_001";
const string LargeFaceListName = "MyLargeFaceListDisplayName";
const string ImageDir = @"/path/to/FaceList/images";
faceClient.LargeFaceList.CreateAsync(LargeFaceListId, LargeFaceListName).Wait();

// Add Faces to the LargeFaceList.
Parallel.ForEach(
    Directory.GetFiles(ImageDir, "*.jpg"),
    async imagePath =>
        {
            using (Stream stream = File.OpenRead(imagePath))
            {
                await faceClient.LargeFaceList.AddFaceFromStreamAsync(LargeFaceListId, stream);
            }
        });

// Train() is newly added operation for LargeFaceList.
// Must call it before FindSimilarAsync() to ensure the newly added faces searchable.
await TrainLargeFaceList(LargeFaceListId);

// Perform FindSimilar.
const string QueryImagePath = @"/path/to/query/image";
var results = new List<SimilarPersistedFace[]>();
using (Stream stream = File.OpenRead(QueryImagePath))
{
    var faces = faceClient.Face.DetectWithStreamAsync(stream).Result;
    foreach (var face in faces)
    {
        results.Add(await faceClient.Face.FindSimilarAsync(face.FaceId, largeFaceListId: LargeFaceListId));
    }
}
```

Tal como se mostró anteriormente, la administración de datos y la parte FindSimilar son prácticamente iguales. La única excepción es que debe realizarse una operación Train de procesamiento previo totalmente nueva en LargeFaceList para que FindSimilar funcione.

## <a name="step-3-train-suggestions"></a>Paso 3: Sugerencias de entrenamiento

Aunque la operación Train acelera las operaciones [FindSimilar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) e [Identification](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239), el tiempo de entrenamiento se ve afectado, especialmente cuando se aplica a gran escala. En la tabla siguiente se indica el tiempo de entrenamiento estimado de las diferentes escalas.

| Escala para caras o personas | Tiempo de entrenamiento estimado |
|:---:|:---:|
| 1,000 | 1 a 2 segundos |
| 10 000 | 5 a 10 segundos |
| 100 000 | 1 a 2 minutos |
| 1 000 000 | 10 a 30 minutos |

Para aprovechar más la característica a gran escala, se recomiendan las estrategias siguientes.

### <a name="step-31-customize-time-interval"></a>Paso 3.1: Personalización del intervalo de tiempo

Como se muestra en `TrainLargeFaceList()`, hay un intervalo de tiempo en milisegundos para retrasar el proceso de comprobación del estado del entrenamiento infinito. Cuando LargeFaceList tiene más caras, el uso de un intervalo mayor reduce el número de llamadas y el costo. Personalice el intervalo de tiempo según la capacidad esperada de LargeFaceList.

La misma estrategia se aplica también a LargePersonGroup. Por ejemplo, al entrenar un LargePersonGroup con un millón de personas, `timeIntervalInMilliseconds` puede ser 60 000, que es un intervalo de un minuto.

### <a name="step-32-small-scale-buffer"></a>Paso 3.2: Búfer a pequeña escala

Las búsquedas de personas o caras en LargePersonGroup o LargeFaceList solo se pueden realizar después del entrenamiento. En un escenario dinámico, se agregan constantemente nuevas personas o caras que deben poder buscarse de inmediato, aunque el entrenamiento podría tardar más tiempo del deseado. 

Para mitigar este problema, use una colección LargePersonGroup o LargeFaceList a pequeña escala adicional como búfer solo para las entradas recién agregadas. Este búfer tarda menos tiempo en el entrenamiento porque es más pequeño. La funcionalidad de búsqueda inmediata en este búfer temporal debería funcionar. Use este búfer en combinación con el entrenamiento en la colección LargePersonGroup o LargeFaceList maestra mediante la ejecución del entrenamiento maestro en un intervalo más escaso. Algunos ejemplos son en medio de la noche y a diario.

Flujo de trabajo de ejemplo:

1. Cree un LargePersonGroup o un LargeFaceList maestro, es decir, una colección maestra. Cree un LargePersonGroup o un LargeFaceList de búfer, es decir, una colección búfer. La colección búfer es solo para las personas o caras recién agregadas.
1. Agregue las nuevas personas o caras a la colección maestra y a la colección búfer.
1. Entrene solo la colección búfer a intervalos más cortos para garantizar que las entradas recién agregadas surtan efecto.
1. Llame a Identification o FindSimilar en la colección maestra y la colección búfer. Combine los resultados.
1. Cuando el tamaño de la colección búfer aumenta hasta un umbral o en un tiempo de inactividad del sistema, cree una nueva colección búfer. Desencadene la operación Train en la colección maestra.
1. Elimine la colección búfer anterior una vez que finalice la operación Train de la colección maestra.

### <a name="step-33-standalone-training"></a>Paso 3.3: Entrenamiento independiente

Si es aceptable una latencia relativamente larga, no es necesario desencadenar la operación Train justo después de agregar datos nuevos. En su lugar, la operación de entrenamiento se puede separar de la lógica principal y desencadenar periódicamente. Esta estrategia es adecuada para escenarios dinámicos con una latencia aceptable. Se puede aplicar a escenarios estáticos para reducir aún más la frecuencia de la operación Train.

Supongamos que hay una función `TrainLargePersonGroup` similar a `TrainLargeFaceList`. Una implementación típica del entrenamiento independiente en LargePersonGroup mediante la invocación de la clase [`Timer`](https://msdn.microsoft.com/library/system.timers.timer(v=vs.110).aspx) en `System.Timers` es:

```csharp
private static void Main()
{
    // Create a LargePersonGroup.
    const string LargePersonGroupId = "mylargepersongroupid_001";
    const string LargePersonGroupName = "MyLargePersonGroupDisplayName";
    faceClient.LargePersonGroup.CreateAsync(LargePersonGroupId, LargePersonGroupName).Wait();

    // Set up standalone training at regular intervals.
    const int TimeIntervalForStatus = 1000 * 60; // 1-minute interval for getting training status.
    const double TimeIntervalForTrain = 1000 * 60 * 60; // 1-hour interval for training.
    var trainTimer = new Timer(TimeIntervalForTrain);
    trainTimer.Elapsed += (sender, args) => TrainTimerOnElapsed(LargePersonGroupId, TimeIntervalForStatus);
    trainTimer.AutoReset = true;
    trainTimer.Enabled = true;

    // Other operations like creating persons, adding faces, and identification, except for Train.
    // ...
}

private static void TrainTimerOnElapsed(string largePersonGroupId, int timeIntervalInMilliseconds)
{
    TrainLargePersonGroup(largePersonGroupId, timeIntervalInMilliseconds).Wait();
}
```

Para más información sobre implementaciones relacionadas con la identificación y la administración de datos, consulte [Agregar caras](how-to-add-faces.md) e [Identificación de caras en imágenes](HowtoIdentifyFacesinImage.md).

## <a name="summary"></a>Resumen

En esta guía, aprendió a migrar el código existente de PersonGroup o FaceList, no los datos, a LargePersonGroup o LargeFaceList:

- LargePersonGroup y LargeFaceList funcionan de manera similar a PersonGroup o FaceList, excepto en que LargeFaceList requiere la operación Train.
- Adopte la estrategia de entrenamiento adecuada para la actualización de datos dinámicos en conjuntos de datos a gran escala.

## <a name="next-steps"></a>Pasos siguientes

Siga una guía de procedimientos para obtener información sobre cómo agregar caras a un elemento PersonGroup o ejecutar la operación Identify en un elemento PersonGroup.

- [Agregar caras](how-to-add-faces.md)
- [Identificación de caras en imágenes](HowtoIdentifyFacesinImage.md)
