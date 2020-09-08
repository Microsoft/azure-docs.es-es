---
title: 'Migración de datos de caras entre distintas suscripciones: Face'
titleSuffix: Azure Cognitive Services
description: En esta guía se muestra cómo migrar los datos de caras almacenados de una suscripción de Face a otra.
services: cognitive-services
author: nitinme
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: nitinme
ms.custom: devx-track-csharp
ms.openlocfilehash: 3d52cd57d48a03c5dab8cb8098cd212e76ca76a1
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929081"
---
# <a name="migrate-your-face-data-to-a-different-face-subscription"></a>Migración de los datos de caras a una suscripción de Face distinta

En esta guía se muestra cómo mover datos de caras, como un objeto PersonGroup guardado con caras, a una suscripción de Face de Azure Cognitive Services distinta. Para mover los datos, use la característica Instantánea. Esto le evita tener que crear y entrenar repetidamente un objeto PersonGroup o FaceList cuando mueva o expanda las operaciones. Por ejemplo, quizás haya creado un objeto PersonGroup con una suscripción gratuita y ahora quiera migrarlo a la suscripción de pago. O bien, puede que deba sincronizar datos de caras entre regiones para una operación empresarial a gran escala.

Esta misma estrategia de migración también se aplica a los objetos LargePersonGroup y LargeFaceList. Si no está familiarizado con los conceptos que aparecen en esta guía, consulte sus definiciones en la guía [Conceptos del reconocimiento facial](../concepts/face-recognition.md). Esta guía usa la biblioteca cliente .NET de Face con C#.

## <a name="prerequisites"></a>Requisitos previos

Necesita los siguientes elementos:

- Dos claves de suscripción de Face (una con los datos existentes y otra a la cual migrar). Para suscribirse al servicio Face y obtener la clave, siga las instrucciones de [Creación de una cuenta de Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account).
- Cadena de identificador de suscripción de Face que se corresponde con la suscripción de destino. Para encontrarla, seleccione **Información general** en Azure Portal. 
- Cualquier edición de [Visual Studio 2015 o 2017](https://www.visualstudio.com/downloads/).

## <a name="create-the-visual-studio-project"></a>Creación del proyecto de Visual Studio

Esta guía utiliza una aplicación de consola sencilla para ejecutar la migración de datos de caras. Para realizar una implementación completa, consulte el [ejemplo de instantánea de Face](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample) en GitHub.

1. En Visual Studio, cree un nuevo proyecto de Aplicación de consola (.NET Framework). Asígnele el nombre **FaceApiSnapshotSample**.
1. Obtenga los paquetes NuGet requeridos. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes de NuGet**. Seleccione la pestaña **Examinar** y, luego, **Incluir versión previa**. Busque e instale el paquete siguiente:
    - [Microsoft.Azure.CognitiveServices.Vision.Face 2.3.0-preview](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.2.0-preview)

## <a name="create-face-clients"></a>Creación de clientes de Face

En el método **Main** de *Program.cs*, cree dos instancias de [FaceClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient?view=azure-dotnet) para las suscripciones de origen y destino. En este ejemplo, usamos una suscripción de Face de la región de Este de Asia como origen y una suscripción de Oeste de EE. UU. como destino. Este ejemplo muestra cómo migrar datos de una región de Azure a otra. 

[!INCLUDE [subdomains-note](../../../../includes/cognitive-services-custom-subdomains-note.md)]

```csharp
var FaceClientEastAsia = new FaceClient(new ApiKeyServiceClientCredentials("<East Asia Subscription Key>"))
    {
        Endpoint = "https://southeastasia.api.cognitive.microsoft.com/>"
    };

var FaceClientWestUS = new FaceClient(new ApiKeyServiceClientCredentials("<West US Subscription Key>"))
    {
        Endpoint = "https://westus.api.cognitive.microsoft.com/"
    };
```

Rellene los valores de las claves de suscripción y las direcciones URL de los puntos de conexión para las suscripciones de origen y destino.


## <a name="prepare-a-persongroup-for-migration"></a>Preparación de un objeto PersonGroup para la migración

Necesita el identificador del objeto PersonGroup en la suscripción de origen para migrarlo a la suscripción de destino. Use el método [PersonGroupOperationsExtensions.ListAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperationsextensions.listasync?view=azure-dotnet) recuperar una lista de los objetos PersonGroup. Obtenga luego la propiedad [PersonGroup.PersonGroupId](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.persongroup.persongroupid?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Vision_Face_Models_PersonGroup_PersonGroupId). Este proceso parecerá distinto en función de los objetos PersonGroup que tenga. En esta guía, el identificador del objeto PersonGroup de origen se almacena en `personGroupId`.

> [!NOTE]
> El [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample) crea y entrena un nuevo objeto PersonGroup para migrar. En la mayoría de los casos, ya debe tener un objeto PersonGroup para utilizarlo.

## <a name="take-a-snapshot-of-a-persongroup"></a>Instantánea de un objeto PersonGroup

Una instantánea es el almacenamiento remoto temporal de ciertos tipos de datos de Face. Funciona como una especie de Portapapeles para copiar datos de una suscripción a otra. En primer lugar, tome una instantánea de los datos en la suscripción de origen. Después, la aplica a un nuevo objeto de datos en la suscripción de destino.

Utilice la instancia de FaceClient de la suscripción de origen para tomar una instantánea de PersonGroup. Use [TakeAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.snapshotoperationsextensions.takeasync?view=azure-dotnet) con el identificador de PersonGroup y el identificador de la suscripción de destino. Si tiene varias suscripciones de destino, agréguelas como entradas de matriz en el tercer parámetro.

```csharp
var takeSnapshotResult = await FaceClientEastAsia.Snapshot.TakeAsync(
    SnapshotObjectType.PersonGroup,
    personGroupId,
    new[] { "<Azure West US Subscription ID>" /* Put other IDs here, if multiple target subscriptions wanted */ });
```

> [!NOTE]
> El proceso de tomar y aplicar instantáneas no interrumpe ninguna llamada normal al objeto PersonGroup o FaceList de origen o de destino. No realice llamadas simultáneas que cambien el objeto de origen, como [llamadas de administración de FaceList](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.facelistoperations?view=azure-dotnet) o la llamada de [entrenamiento de PersonGroup](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperations?view=azure-dotnet), por ejemplo. La operación de instantánea se puede ejecutar antes o después de esas operaciones o puede provocar errores.

## <a name="retrieve-the-snapshot-id"></a>Recuperación del identificador de la instantánea

El método utilizado para tomar instantáneas es asincrónico, por lo que debe esperar a su finalización. No se pueden cancelar operaciones de instantáneas. En este código, el método `WaitForOperation` supervisa la llamada asincrónica. Controla el estado cada 100 ms. Una vez finalizada la operación, recupere el identificador de operación mediante el análisis del campo `OperationLocation`. 

```csharp
var takeOperationId = Guid.Parse(takeSnapshotResult.OperationLocation.Split('/')[2]);
var operationStatus = await WaitForOperation(FaceClientEastAsia, takeOperationId);
```

Un valor `OperationLocation` típico tiene este aspecto:

```csharp
"/operations/a63a3bdd-a1db-4d05-87b8-dbad6850062a"
```

El método auxiliar `WaitForOperation` es el siguiente:

```csharp
/// <summary>
/// Waits for the take/apply operation to complete and returns the final operation status.
/// </summary>
/// <returns>The final operation status.</returns>
private static async Task<OperationStatus> WaitForOperation(IFaceClient client, Guid operationId)
{
    OperationStatus operationStatus = null;
    do
    {
        if (operationStatus != null)
        {
            Thread.Sleep(TimeSpan.FromMilliseconds(100));
        }

        // Get the status of the operation.
        operationStatus = await client.Snapshot.GetOperationStatusAsync(operationId);

        Console.WriteLine($"Operation Status: {operationStatus.Status}");
    }
    while (operationStatus.Status != OperationStatusType.Succeeded
            && operationStatus.Status != OperationStatusType.Failed);

    return operationStatus;
}
```

Cuando el estado de la operación muestra `Succeeded`, obtenga el identificador de la instantánea analizando el campo `ResourceLocation` de la instancia OperationStatus devuelta.

```csharp
var snapshotId = Guid.Parse(operationStatus.ResourceLocation.Split('/')[2]);
```

Un valor `resourceLocation` típico tiene este aspecto:

```csharp
"/snapshots/e58b3f08-1e8b-4165-81df-aa9858f233dc"
```

## <a name="apply-a-snapshot-to-a-target-subscription"></a>Aplicación de la instantánea en una suscripción de destino

A continuación, cree el nuevo objeto PersonGroup en la suscripción de destino mediante un identificador generado aleatoriamente. Luego, use la instancia FaceClient de la suscripción de destino para aplicar la instantánea en este objeto PersonGroup. Pase el identificador de la instantánea y el nuevo identificador de PersonGroup.

```csharp
var newPersonGroupId = Guid.NewGuid().ToString();
var applySnapshotResult = await FaceClientWestUS.Snapshot.ApplyAsync(snapshotId, newPersonGroupId);
```


> [!NOTE]
> Un objeto de instantánea solo es válido durante 48 horas. Solo tome una instantánea si pretende usarla para migrar datos poco después.

Una solicitud de aplicación de instantánea devuelve otro identificador de operación. Para obtener este identificador, analice el campo `OperationLocation` de la instancia applySnapshotResult devuelta. 

```csharp
var applyOperationId = Guid.Parse(applySnapshotResult.OperationLocation.Split('/')[2]);
```

El proceso de aplicación de la instantánea también es asincrónico, por lo que use de nuevo `WaitForOperation` para esperar a que finalice.

```csharp
operationStatus = await WaitForOperation(FaceClientWestUS, applyOperationId);
```

## <a name="test-the-data-migration"></a>Prueba de la migración de datos

Una vez que se aplique la instantánea, el nuevo objeto PersonGroup de la suscripción de destino se rellena con los datos de caras originales. De forma predeterminada, también se copian los resultados del entrenamiento. El nuevo objeto PersonGroup está listo para las llamadas de identificación de caras sin necesidad de reentrenamiento.

Para probar la migración de datos, ejecute estas operaciones y compare los resultados que imprimen en la consola:

```csharp
await DisplayPersonGroup(FaceClientEastAsia, personGroupId);
await IdentifyInPersonGroup(FaceClientEastAsia, personGroupId);

await DisplayPersonGroup(FaceClientWestUS, newPersonGroupId);
// No need to retrain the person group before identification,
// training results are copied by snapshot as well.
await IdentifyInPersonGroup(FaceClientWestUS, newPersonGroupId);
```

Use estos métodos auxiliares:

```csharp
private static async Task DisplayPersonGroup(IFaceClient client, string personGroupId)
{
    var personGroup = await client.PersonGroup.GetAsync(personGroupId);
    Console.WriteLine("Person Group:");
    Console.WriteLine(JsonConvert.SerializeObject(personGroup));

    // List persons.
    var persons = await client.PersonGroupPerson.ListAsync(personGroupId);

    foreach (var person in persons)
    {
        Console.WriteLine(JsonConvert.SerializeObject(person));
    }

    Console.WriteLine();
}
```

```csharp
private static async Task IdentifyInPersonGroup(IFaceClient client, string personGroupId)
{
    using (var fileStream = new FileStream("data\\PersonGroup\\Daughter\\Daughter1.jpg", FileMode.Open, FileAccess.Read))
    {
        var detectedFaces = await client.Face.DetectWithStreamAsync(fileStream);

        var result = await client.Face.IdentifyAsync(detectedFaces.Select(face => face.FaceId.Value).ToList(), personGroupId);
        Console.WriteLine("Test identify against PersonGroup");
        Console.WriteLine(JsonConvert.SerializeObject(result));
        Console.WriteLine();
    }
}
```

Ahora puede usar el nuevo objeto PersonGroup en la suscripción de destino. 

Para actualizar el objeto PersonGroup de destino de nuevo en el futuro, cree un nuevo objeto PersonGroup para recibir la instantánea. Para ello, siga los pasos de esta guía. Una instantánea solo se puede aplicar una sola vez a un objeto PersonGroup único.

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de terminar de migrar datos de caras, elimine manualmente el objeto de instantánea.

```csharp
await FaceClientEastAsia.Snapshot.DeleteAsync(snapshotId);
```

## <a name="next-steps"></a>Pasos siguientes

A continuación, consulte la documentación de referencia de la API relevante, explore una aplicación de ejemplo que utilice la característica Instantánea o siga una guía de procedimientos para comenzar a utilizar las otras operaciones de API mencionadas aquí:

- [Documentación de referencia de Instantánea (SDK de .NET)](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.snapshotoperations?view=azure-dotnet)
- [Ejemplo de instantánea de Face](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample)
- [Agregar caras](how-to-add-faces.md)
- [Detección de caras en una imagen](HowtoDetectFacesinImage.md)
- [Identificación de caras en imágenes](HowtoIdentifyFacesinImage.md)
