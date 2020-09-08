---
title: 'Ejemplo: Identificación de caras en imágenes: Face'
titleSuffix: Azure Cognitive Services
description: En esta guía se muestra cómo identificar caras desconocidas con los objetos PersonGroup, que se crean a partir de personas conocidas de antemano.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: sample
ms.date: 07/02/2020
ms.author: sbowles
ms.custom: devx-track-csharp
ms.openlocfilehash: bae84d93d15abe8804a430dacd2f2cddf8a9aed9
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919172"
---
# <a name="example-identify-faces-in-images"></a>Ejemplo: Identificación de caras en imágenes

En esta guía se muestra cómo identificar caras desconocidas con los objetos PersonGroup, que se crean a partir de personas conocidas de antemano. Los ejemplos se escriben en C# mediante el uso de la biblioteca cliente de Face de Azure Cognitive Services.

En este ejemplo se muestra:

- Cómo crear un elemento PersonGroup. Este elemento PersonGroup contiene una lista de personas conocidas.
- Cómo asignar caras a cada persona. Estas caras se usan como base de referencia para identificar personas. Le recomendamos usar vistas frontales claras de caras. Un ejemplo es un identificador de fotografía. Un buen conjunto de fotos incluye caras de la misma persona en distintas posturas, colores de ropa o peinados.

## <a name="prerequisites"></a>Requisitos previos
* La versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Creación de un recurso de Face"  target="_blank">cree un recurso de Face <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez implementado, haga clic en **Ir al recurso** y copie la clave.
* Algunas fotografías con las caras de las personas identificadas. [Descargue fotos de ejemplo](https://github.com/Microsoft/Cognitive-Face-Windows/tree/master/Data) para Anna, Bill y Clare.
* Una serie de fotos de prueba. Las fotografías pueden contener o no las caras de las personas identificadas. Use algunas imágenes del vínculo de fotografías de ejemplo.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

Cree una nueva aplicación de consola de .NET Core en el IDE o editor que prefiera. 

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `face-identify`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: *Program.cs*. 

```dotnetcli
dotnet new console -n face-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. Para compilar la aplicación:

```dotnetcli
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores. 

```output
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

## <a name="step-1-authorize-the-api-call"></a>Paso 1: Autorización de la llamada a la API

Cada llamada a Face API requiere una clave de suscripción. Esta clave puede pasarse a través de un parámetro de cadena de consulta o especificarse en la cabecera de la solicitud. Cuando usa una biblioteca cliente, la clave de suscripción se pasa mediante el constructor de la clase **FaceClient**. Agregue el siguiente código al método **Main** de la clase *Program.cs*.
 
```csharp 
private readonly IFaceClient faceClient = new FaceClient(
    new ApiKeyServiceClientCredentials("<subscription key>"),
    new System.Net.Http.DelegatingHandler[] { });
```

## <a name="step-2-create-the-persongroup"></a>Paso 2: Creación del elemento PersonGroup

En este paso, un **PersonGroup** denominado "MyFriends" contiene a Anna, Bill y Clare. Cada persona tiene varias caras registrados. Las caras se deben detectar en las imágenes. Después de todos estos pasos, tiene un elemento **PersonGroup** similar al de la siguiente imagen:

![MyFriends](../Images/group.image.1.jpg)

### <a name="step-21-define-people-for-the-persongroup"></a>Paso 2.1: Definir personas para el elemento PersonGroup
Una persona es una unidad básica de identificación. Una persona puede tener una o más caras conocidas registradas. Un elemento **PersonGroup** es una colección de personas. Cada persona se define dentro de un elemento **PersonGroup** determinado. La identificación se hace en un elemento **PersonGroup**. La tarea consiste en crear un elemento **PersonGroup** y, después, crear personas en él, como Anna, Bill y Clare.

En primer lugar, cree un elemento **PersonGroup** mediante la API [PersonGroup - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244). La API de la biblioteca cliente correspondiente es el método **CreatePersonGroupAsync** para la clase **FaceClient**. El identificador de grupo que está especificado para crear el grupo es único para cada suscripción. También puede obtener, actualizar o eliminar elementos **PersonGroup** mediante otras API **PersonGroup**. 

Una vez que se define un grupo, puede definir las personas dentro de él mediante [PersonGroup Person - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) API. El método de la biblioteca cliente es **CreatePersonAsync**. Puede agregar una cara a cada persona una vez creada.

```csharp 
// Create an empty PersonGroup
string personGroupId = "myfriends";
await faceClient.PersonGroup.CreateAsync(personGroupId, "My Friends");
 
// Define Anna
CreatePersonResult friend1 = await faceClient.PersonGroupPerson.CreateAsync(
    // Id of the PersonGroup that the person belonged to
    personGroupId,    
    // Name of the person
    "Anna"            
);
 
// Define Bill and Clare in the same way
```
### <a name="step-22-detect-faces-and-register-them-to-the-correct-person"></a><a name="step2-2"></a> Paso 2.2: Detectar caras y registrarlas en la persona correcta
La detección se realiza enviando una petición web "POST" a [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) API con el archivo de imagen en el cuerpo de la solicitud HTTP. Cuando se usa la biblioteca cliente, la detección de caras se realiza mediante uno de los métodos DetectAsync de la clase FaceClient.

Para cada cara detectada llame a [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) para agregarla a la persona correcta.

El siguiente código demuestra el proceso de cómo detectar una cara en una imagen y agregarla a una persona:

```csharp 
// Directory contains image files of Anna
const string friend1ImageDir = @"D:\Pictures\MyFriends\Anna\";
 
foreach (string imagePath in Directory.GetFiles(friend1ImageDir, "*.jpg"))
{
    using (Stream s = File.OpenRead(imagePath))
    {
        // Detect faces in the image and add to Anna
        await faceClient.PersonGroupPerson.AddFaceFromStreamAsync(
            personGroupId, friend1.PersonId, s);
    }
}
// Do the same for Bill and Clare
``` 
Si la imagen contiene más de una cara, solo se agrega la cara más grande. Puede agregar otras caras a la persona. Pase una cadena con el formato "targetFace = left, top, width, height" al parámetro de consulta targetFace de [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) API. También puede usar el parámetro opcional targetFace para que el método AddPersonFaceAsync agregue otras caras. Cada cara agregada a la persona tiene un identificador de cara persistente único. Puede usar este identificador en [PersonGroup Person – Delete Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523e) y en [Face – Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239).

## <a name="step-3-train-the-persongroup"></a>Paso 3: Entrenamiento del elemento PersonGroup

Es necesario entrenar el elemento **PersonGroup** para poder realizar una identificación con él. El elemento **PersonGroup** se debe volver a entrenar después de agregar o quitar a una persona, o si se edita su cara registrada. El entrenamiento se realiza mediante [PersonGroup – Train](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249) API. Al usar la biblioteca cliente, es una llamada al método **TrainPersonGroupAsync**:
 
```csharp 
await faceClient.PersonGroup.TrainAsync(personGroupId);
```
 
El entrenamiento es un proceso asincrónico. Es posible que no se haya terminado incluso después de que se haya devuelto el método **TrainPersonGroupAsync**. Quizás necesite consultar el estado del entrenamiento. Use la API [PersonGroup - Get Training Status](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395247) o el método **GetPersonGroupTrainingStatusAsync** de la biblioteca cliente. El código siguiente muestra una lógica sencilla para esperar a que se complete el entrenamiento del elemento **PersonGroup**:
 
```csharp 
TrainingStatus trainingStatus = null;
while(true)
{
    trainingStatus = await faceClient.PersonGroup.GetTrainingStatusAsync(personGroupId);
 
    if (trainingStatus.Status != TrainingStatusType.Running)
    {
        break;
    }
 
    await Task.Delay(1000);
} 
``` 

## <a name="step-4-identify-a-face-against-a-defined-persongroup"></a>Paso 4: Identificación de una cara con un elemento PersonGroup definido

Cuando el servicio de Face realiza las identificaciones, calcula la similitud de una cara de prueba entre todas las caras dentro de un grupo. Devuelve las personas más comparables con la cara de prueba. Este proceso se realiza con [Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) API o con el método IdentifyAsync de la biblioteca cliente.

La cara de prueba se debe detectar con los pasos anteriores. Luego, el identificador de cara se pasa a la API de identificación como segundo argumento. Es posible identificar varios id. de caras a la vez. El resultado contiene todos los resultados identificados. De manera predeterminada, el proceso de identificación devuelve solo una persona que coincida mejor con la cara de la prueba. Si lo prefiere, especifique el parámetro opcional _maxNumOfCandidatesReturned_ para que el proceso de identificación devuelva más candidatos.

El código siguiente muestra el proceso de identificación:

```csharp 
string testImageFile = @"D:\Pictures\test_img1.jpg";

using (Stream s = File.OpenRead(testImageFile))
{
    var faces = await faceClient.Face.DetectWithStreamAsync(s);
    var faceIds = faces.Select(face => face.FaceId.Value).ToArray();
 
    var results = await faceClient.Face.IdentifyAsync(faceIds, personGroupId);
    foreach (var identifyResult in results)
    {
        Console.WriteLine("Result of face: {0}", identifyResult.FaceId);
        if (identifyResult.Candidates.Length == 0)
        {
            Console.WriteLine("No one identified");
        }
        else
        {
            // Get top 1 among all candidates returned
            var candidateId = identifyResult.Candidates[0].PersonId;
            var person = await faceClient.PersonGroupPerson.GetAsync(personGroupId, candidateId);
            Console.WriteLine("Identified as {0}", person.Name);
        }
    }
}
``` 

Una vez que complete los pasos, intente identificar las distintas caras. Vea si las caras de Anna, Bill o Clare se pueden identificar correctamente según las imágenes cargadas para realizar la detección de caras. Consulte los siguientes ejemplos:

![Identificación de distintas caras](../Images/identificationResult.1.jpg )

## <a name="step-5-request-for-large-scale"></a>Paso 5: Solicitud de gran escala

Un elemento **PersonGroup** puede contener hasta 10 000 personas según la limitación de diseño anterior. Para más información sobre escenarios de hasta un millón de escalas, consulte [Uso de la característica a gran escala](how-to-use-large-scale.md).

## <a name="summary"></a>Resumen

En esta guía, aprendió el proceso de creación de un elemento **PersonGroup** y a identificar una persona. Se explicaron y mostraron las siguientes características:

- Detección de caras con [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d) API.
- Creación de un elemento PersonGroups con [PersonGroup - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) API.
- Creación de personas con [PersonGroup Person - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) API.
- Entrenamiento de un elemento PersonGroup con [PersonGroup – Train](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249) API.
- Identificación de caras desconocidas en el elemento PersonGroup con [Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) API.

## <a name="related-topics"></a>Temas relacionados

- [Face recognition concepts](../concepts/face-recognition.md) (Conceptos del reconocimiento facial)
- [Detección de caras en una imagen](HowtoDetectFacesinImage.md)
- [Agregar caras](how-to-add-faces.md)
- [Uso de la característica a gran escala](how-to-use-large-scale.md)
