---
title: 'Ejemplo: Agregar caras a un objeto PersonGroup: Face'
titleSuffix: Azure Cognitive Services
description: En esta guía se muestra cómo agregar una gran cantidad de personas y caras a un objeto PersonGroup con el servicio Face de Azure Cognitive Services.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: sample
ms.date: 04/10/2019
ms.author: sbowles
ms.custom: devx-track-csharp
ms.openlocfilehash: 06c353cf5fbd5c2b1058d99e7f91e9fcc3853cb1
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929053"
---
# <a name="add-faces-to-a-persongroup"></a>Agregar caras a un objeto PersonGroup

En esta guía se muestra cómo agregar una gran cantidad de personas y caras a un objeto PersonGroup. La misma estrategia se aplica también a los objetos LargePersonGroup, FaceList y LargeFaceList. Este ejemplo se escribe en C# mediante el uso de la biblioteca cliente de Face para .NET de Azure Cognitive Services.

## <a name="step-1-initialization"></a>Paso 1: Inicialización

El código siguiente declara varias variables e implementa una función auxiliar para programar las solicitudes para agregar caras:

- `PersonCount` es el número total de personas.
- `CallLimitPerSecond` es el número máximo de llamadas por segundo según el nivel de suscripción.
- `_timeStampQueue` es una cola para registrar las marcas de tiempo de solicitud.
- `await WaitCallLimitPerSecondAsync()` espera hasta que se valide para enviar la solicitud siguiente.

```csharp
const int PersonCount = 10000;
const int CallLimitPerSecond = 10;
static Queue<DateTime> _timeStampQueue = new Queue<DateTime>(CallLimitPerSecond);

static async Task WaitCallLimitPerSecondAsync()
{
    Monitor.Enter(_timeStampQueue);
    try
    {
        if (_timeStampQueue.Count >= CallLimitPerSecond)
        {
            TimeSpan timeInterval = DateTime.UtcNow - _timeStampQueue.Peek();
            if (timeInterval < TimeSpan.FromSeconds(1))
            {
                await Task.Delay(TimeSpan.FromSeconds(1) - timeInterval);
            }
            _timeStampQueue.Dequeue();
        }
        _timeStampQueue.Enqueue(DateTime.UtcNow);
    }
    finally
    {
        Monitor.Exit(_timeStampQueue);
    }
}
```

## <a name="step-2-authorize-the-api-call"></a>Paso 2: Autorización de la llamada a la API

Cuando usa una biblioteca cliente, debe pasar la clave de suscripción al constructor de la clase **FaceClient**. Por ejemplo:

```csharp
private readonly IFaceClient faceClient = new FaceClient(
    new ApiKeyServiceClientCredentials("<SubscriptionKey>"),
    new System.Net.Http.DelegatingHandler[] { });
```

Para obtener la clave de suscripción, vaya a Azure Marketplace desde Azure Portal. Para más información, consulte [Suscripciones](https://www.microsoft.com/cognitive-services/sign-up).

## <a name="step-3-create-the-persongroup"></a>Paso 3: Creación del elemento PersonGroup

Se crea un elemento PersonGroup denominado "MyPersonGroup" para guardar las personas.
El tiempo de solicitud se pone en cola en `_timeStampQueue` para garantizar la validación general.

```csharp
const string personGroupId = "mypersongroupid";
const string personGroupName = "MyPersonGroup";
_timeStampQueue.Enqueue(DateTime.UtcNow);
await faceClient.LargePersonGroup.CreateAsync(personGroupId, personGroupName);
```

## <a name="step-4-create-the-persons-for-the-persongroup"></a>Paso 4: Creación de las personas del objeto PersonGroup

Las personas se crean de forma simultánea y se aplica también `await WaitCallLimitPerSecondAsync()` para evitar que se supere el límite de llamadas.

```csharp
Person[] persons = new Person[PersonCount];
Parallel.For(0, PersonCount, async i =>
{
    await WaitCallLimitPerSecondAsync();

    string personName = $"PersonName#{i}";
    persons[i] = await faceClient.PersonGroupPerson.CreateAsync(personGroupId, personName);
});
```

## <a name="step-5-add-faces-to-the-persons"></a>Paso 5: Adición de caras a las personas

Las caras que se agregan a las distintas personas se procesan de manera simultánea. Las caras que se agregan a una persona específica se procesan de manera secuencial.
Una vez más, se invoca `await WaitCallLimitPerSecondAsync()` para garantizar que la frecuencia de solicitud está dentro del ámbito de limitación.

```csharp
Parallel.For(0, PersonCount, async i =>
{
    Guid personId = persons[i].PersonId;
    string personImageDir = @"/path/to/person/i/images";

    foreach (string imagePath in Directory.GetFiles(personImageDir, "*.jpg"))
    {
        await WaitCallLimitPerSecondAsync();

        using (Stream stream = File.OpenRead(imagePath))
        {
            await faceClient.PersonGroupPerson.AddFaceFromStreamAsync(personGroupId, personId, stream);
        }
    }
});
```

## <a name="summary"></a>Resumen

En esta guía, conoció el proceso de creación de un objeto PersonGroup con un gran número de personas y caras. Varios recordatorios:

- Esta estrategia se aplica también a los objetos FaceList y LargePersonGroup.
- La incorporación y eliminación de caras a los distintos objetos FaceList o a las distintas personas de un objeto LargePersonGroup se procesan de manera simultánea.
- La incorporación y eliminación de caras a un objeto FaceList específico o a una persona específica de un objeto LargePersonGroup se realizan de manera secuencial.
- Para simplificar, en esta guía se omite cómo controlar una excepción potencial. Si quiere mejorar la solidez, aplique la directiva de reintentos correspondiente.

Se explicaron y mostraron las siguientes características:

- Creación de un elemento PersonGroups con [PersonGroup - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) API.
- Creación de personas con [PersonGroup Person - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) API.
- Incorporación de caras a personas con [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) API.

## <a name="related-topics"></a>Temas relacionados

- [Identificación de caras en imágenes](HowtoIdentifyFacesinImage.md)
- [Detección de caras en una imagen](HowtoDetectFacesinImage.md)
- [Uso de la característica a gran escala](how-to-use-large-scale.md)
