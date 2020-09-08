---
title: 'Especificación de un modelo de detección: Face'
titleSuffix: Azure Cognitive Services
description: En este artículo se le mostrará cómo elegir el modelo de detección de caras que usará con la aplicación Face de Azure.
services: cognitive-services
author: yluiu
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 05/16/2019
ms.author: yluiu
ms.custom: devx-track-csharp
ms.openlocfilehash: 3278a1522fe3967dd1de5551e90732e63b7666f2
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88931563"
---
# <a name="specify-a-face-detection-model"></a>Especificación de un modelo de detección de caras

En esta guía se indica cómo especificar un modelo de detección de caras para el servicio Face de Azure.

El servicio Face emplea modelos de aprendizaje automático para realizar operaciones en caras humanas de imágenes. Seguimos mejorando la precisión de nuestros modelos gracias a los comentarios de los clientes y los avances en la investigación, y estas mejoras se ofrecen como actualizaciones del modelo. Los desarrolladores tienen la opción de especificar qué versión del modelo de detección de caras les gustaría usar; pueden elegir el modelo que mejor se adapte a sus casos de uso.

Siga leyendo para obtener información sobre cómo especificar el modelo de detección de caras en ciertas operaciones de caras. El servicio Face usa la detección de caras cada vez que convierte la imagen de una cara en otro tipo de datos.

Si no está seguro de si debe usar el modelo más reciente, vaya a la sección [Evaluación de modelos diferentes](#evaluate-different-models) para evaluar el nuevo modelo y comparar los resultados con el conjunto de datos actual.

## <a name="prerequisites"></a>Prerequisites

Debe estar familiarizado con el concepto de detección de caras de inteligencia artificial. Si no lo está, consulte la guía conceptual de detección de caras o la guía paso a paso:

* [Conceptos relacionados con la detección de caras](../concepts/face-detection.md)
* [Detección de caras en una imagen](HowtoDetectFacesinImage.md)

## <a name="detect-faces-with-specified-model"></a>Detección de caras con el modelo especificado

La detección de caras encuentra las ubicaciones del cuadro delimitador de caras humanas e identifica sus puntos de referencia visuales. Extrae las características de la cara y las almacena para su uso posterior en operaciones de [reconocimiento](../concepts/face-recognition.md).

Al usar la API [Face - Detect], puede asignar la versión del modelo con el parámetro `detectionModel`. Los valores disponibles son:

* `detection_01`
* `detection_02`

Una dirección URL de solicitud para la API de REST [Face - Detect] tendrá este aspecto:

`https://westus.api.cognitive.microsoft.com/face/v1.0/detect[?returnFaceId][&returnFaceLandmarks][&returnFaceAttributes][&recognitionModel][&returnRecognitionModel][&detectionModel]&subscription-key=<Subscription key>`

Si usa la biblioteca cliente, puede asignar el valor de `detectionModel` si pasa una cadena adecuada. Si lo deja sin asignar, la API usará la versión predeterminada del modelo (`detection_01`). Consulte el siguiente ejemplo de código de la biblioteca cliente .NET.

```csharp
string imageUrl = "https://news.microsoft.com/ceo/assets/photos/06_web.jpg";
var faces = await faceClient.Face.DetectWithUrlAsync(imageUrl, false, false, recognitionModel: "recognition_03", detectionModel: "detection_02");
```

## <a name="add-face-to-person-with-specified-model"></a>Adición de una cara a un objeto Person con el modelo especificado

El servicio Face puede extraer datos de los rostros de una imagen y asociarlos a un objeto **Person** con la API [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b). En esta llamada API, puede especificar el modelo de detección igual que en [Face - Detect].

Consulte el siguiente ejemplo de código de la biblioteca cliente .NET.

```csharp
// Create a PersonGroup and add a person with face detected by "detection_02" model
string personGroupId = "mypersongroupid";
await faceClient.PersonGroup.CreateAsync(personGroupId, "My Person Group Name", recognitionModel: "recognition_03");

string personId = (await faceClient.PersonGroupPerson.CreateAsync(personGroupId, "My Person Name")).PersonId;

string imageUrl = "https://news.microsoft.com/ceo/assets/photos/06_web.jpg";
await client.PersonGroupPerson.AddFaceFromUrlAsync(personGroupId, personId, imageUrl, detectionModel: "detection_02");
```

Este código crea un objeto **PersonGroup** con el identificador `mypersongroupid` y le agrega un objeto **Person**. Después, agrega una cara a este objeto **Person** mediante el modelo `detection_02`. Si no se especifica el parámetro *detectionModel*, la API usará el modelo predeterminado (`detection_01`).

> [!NOTE]
> No es necesario usar el mismo modelo de detección para todas las caras en un objeto **Person** ni usar el mismo modelo de detección al detectar caras nuevas para compararlas con un objeto **Person** (en la API [Face - Identify], por ejemplo).

## <a name="add-face-to-facelist-with-specified-model"></a>Adición de una cara a un objeto FaceList con el modelo especificado

También puede especificar un modelo de detección al agregar una cara a un objeto **FaceList** existente. Consulte el siguiente ejemplo de código de la biblioteca cliente .NET.

```csharp
await faceClient.FaceList.CreateAsync(faceListId, "My face collection", recognitionModel: "recognition_03");

string imageUrl = "https://news.microsoft.com/ceo/assets/photos/06_web.jpg";
await client.FaceList.AddFaceFromUrlAsync(faceListId, imageUrl, detectionModel: "detection_02");
```

Este código crea un objeto **FaceList** denominado `My face collection` y le agrega una cara con el modelo `detection_02`. Si no se especifica el parámetro *detectionModel*, la API usará el modelo predeterminado (`detection_01`).

> [!NOTE]
> No es necesario usar el mismo modelo de detección para todas las caras en un objeto **FaceList** ni usar el mismo modelo de detección al detectar caras nuevas para compararlas con un objeto **FaceList**.

## <a name="evaluate-different-models"></a>Evaluación de modelos diferentes

Los diferentes modelos de detección de caras están optimizados para llevar a cabo distintas tareas. En la tabla siguiente encontrará información general sobre las diferencias.

|**detection_01**  |**detection_02**  |
|---------|---------|
|Opción predeterminada para todas las operaciones de detección de caras. | Publicado en mayo de 2019 y disponible de forma opcional en todas las operaciones de detección de caras.
|No está optimizado para caras pequeñas, de perfil o borrosas.  | Precisión mejorada en caras pequeñas, de perfil o borrosas. |
|Devuelve atributos de la cara (posición de la cabeza, edad, emociones, etc.) si se especifican en la llamada de detección. |  No devuelve atributos de la cara.     |
|Devuelve puntos de referencia de la cara si se especifican en la llamada de detección.   | No devuelve puntos de referencia de la cara.  |

La mejor manera de comparar el rendimiento de los modelos `detection_01` y `detection_02` consiste en usarlos en un conjunto de datos de ejemplo. Se recomienda llamar a la API [Face - Detect] con cada modelo de detección en varias imágenes diferentes, sobre todo en imágenes con muchas caras o con caras difíciles de ver. Preste atención al número de caras que devuelve cada modelo.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a especificar el modelo de detección que se debe usar con diferentes tipos de Face API. A continuación, siga un inicio rápido para empezar a usar la detección de caras.

* [SDK de .NET para Face](../Quickstarts/csharp-sdk.md)
* [SDK de Python para Face](../Quickstarts/python-sdk.md)
* [SDK de Go para Face](../Quickstarts/go-sdk.md)

[Face - Detect]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d
[Face - Find Similar]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237
[Face - Identify]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239
[Face - Verify]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a
[PersonGroup - Create]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244
[PersonGroup - Get]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395246
[PersonGroup Person - Add Face]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b
[PersonGroup - Train]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249
[LargePersonGroup - Create]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d
[FaceList - Create]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b
[FaceList - Get]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524c
[FaceList - Add Face]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250
[LargeFaceList - Create]: https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc
