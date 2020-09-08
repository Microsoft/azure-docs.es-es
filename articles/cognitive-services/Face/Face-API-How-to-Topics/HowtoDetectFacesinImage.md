---
title: 'Detección de caras en una imagen: Face'
titleSuffix: Azure Cognitive Services
description: Esta guía muestra cómo usar la detección de caras para extraer atributos como el sexo, la edad o la postura de una imagen determinada.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 04/18/2019
ms.author: sbowles
ms.custom: devx-track-csharp
ms.openlocfilehash: 231f30f5532d0934ba41e591aa821d56b11d5856
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88928010"
---
# <a name="get-face-detection-data"></a>Obtención de los datos de detección de caras

Esta guía muestra cómo usar la detección de caras para extraer atributos como el sexo, la edad o la postura de una imagen determinada. Los fragmentos de código de esta guía están escritos en C# con la biblioteca cliente Face de Azure Cognitive Services. La misma funcionalidad está disponible mediante la [API REST](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

Esta guía le muestra cómo:

- Obtener las ubicaciones y las dimensiones de las caras de una imagen.
- Obtener las ubicaciones de diversos puntos de referencia faciales, como las pupilas, nariz y boca, de una imagen.
- Adivinar el sexo, edad, emoción y otros atributos de una cara detectada.

## <a name="setup"></a>Configurar

En esta guía se da por supuesto que ya ha construido un objeto [FaceClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient?view=azure-dotnet) llamado `faceClient`, con una clave de suscripción y una dirección URL de punto de conexión de Face. A partir de aquí, puede usar la característica de detección de caras mediante una llamada a [DetectWithUrlAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithurlasync?view=azure-dotnet), que se usa en esta guía o una llamada a [DetectWithStreamAsync](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceoperationsextensions.detectwithstreamasync?view=azure-dotnet). Para obtener instrucciones sobre cómo configurar esta característica, siga uno de los inicios rápidos.

Esta guía se centra en los detalles de la llamada de detección, como los argumentos que se pueden pasar y lo que puede hacer con los datos devueltos. Se recomienda consultar solo las características que necesita. Cada operación tarda más tiempo en completarse.

## <a name="get-basic-face-data"></a>Obtención de los datos básicos de la cara

Para encontrar caras y obtener sus ubicaciones en una imagen, llame al método con el parámetro _returnFaceId_ establecido en **true**. Esta es la configuración predeterminada.

```csharp
IList<DetectedFace> faces = await faceClient.Face.DetectWithUrlAsync(imageUrl, true, false, null);
```

Puede consultar los objetos [DetectedFace](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.detectedface?view=azure-dotnet) devueltos para obtener sus identificadores únicos y un rectángulo que proporciona las coordenadas en píxeles de la cara.

```csharp
foreach (var face in faces)
{
    string id = face.FaceId.ToString();
    FaceRectangle rect = face.FaceRectangle;
}
```

Para obtener información sobre cómo analizar la ubicación y las dimensiones de la cara, consulte [FaceRectangle](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.facerectangle?view=azure-dotnet). Normalmente, este rectángulo contiene los ojos, las cejas, la nariz y la boca. La parte superior de la cabeza, las orejas y la barbilla no están incluidas necesariamente. Para usar el rectángulo facial para recortar una cabeza completa u obtener un retrato intermedio, quizás para una imagen de tipo foto de carnet, puede expandir el rectángulo en cada dirección.

## <a name="get-face-landmarks"></a>Obtención de los puntos de referencia faciales

Los [puntos de referencia faciales](../concepts/face-detection.md#face-landmarks) son un conjunto de puntos fáciles de encontrar en una cara como, por ejemplo, las pupilas o la punta de la nariz. Para obtener los datos del punto de referencia facial, establezca el parámetro _returnFaceLandmarks_ en **true**.

```csharp
IList<DetectedFace> faces = await faceClient.Face.DetectWithUrlAsync(imageUrl, true, true, null);
```

El código siguiente muestra cómo podría recuperar las ubicaciones de la nariz y las pupilas:

```csharp
foreach (var face in faces)
{
    var landmarks = face.FaceLandmarks;

    double noseX = landmarks.NoseTip.X;
    double noseY = landmarks.NoseTip.Y;

    double leftPupilX = landmarks.PupilLeft.X;
    double leftPupilY = landmarks.PupilLeft.Y;

    double rightPupilX = landmarks.PupilRight.X;
    double rightPupilY = landmarks.PupilRight.Y;
}
```

También puede usar los datos de los puntos de referencia faciales para calcular con precisión la dirección de la cara. Por ejemplo, se puede definir la rotación de la cara como un vector desde el centro de la boca hasta el centro de los ojos. El código siguiente calcula este vector:

```csharp
var upperLipBottom = landmarks.UpperLipBottom;
var underLipTop = landmarks.UnderLipTop;

var centerOfMouth = new Point(
    (upperLipBottom.X + underLipTop.X) / 2,
    (upperLipBottom.Y + underLipTop.Y) / 2);

var eyeLeftInner = landmarks.EyeLeftInner;
var eyeRightInner = landmarks.EyeRightInner;

var centerOfTwoEyes = new Point(
    (eyeLeftInner.X + eyeRightInner.X) / 2,
    (eyeLeftInner.Y + eyeRightInner.Y) / 2);

Vector faceDirection = new Vector(
    centerOfTwoEyes.X - centerOfMouth.X,
    centerOfTwoEyes.Y - centerOfMouth.Y);
```

Cuando conoce la dirección de la cara, puede girar el marco rectangular de la cara para alinearlo más correctamente. Para recortar las caras de una imagen, puede girar mediante programación la imagen para que las caras siempre aparezcan verticales.

## <a name="get-face-attributes"></a>Obtención de los atributos de la cara

Además de los rectángulos y los puntos de referencia faciales, la API de detección de caras puede analizar varios atributos conceptuales de una cara. Para obtener una lista completa, consulte la sección conceptual [Atributos de una cara](../concepts/face-detection.md#attributes).

Para analizar los atributos de una cara, establezca el parámetro _returnFaceAttributes_ en una lista de valores [FaceAttributeType Enum](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.faceattributetype?view=azure-dotnet).

```csharp
var requiredFaceAttributes = new FaceAttributeType[] {
    FaceAttributeType.Age,
    FaceAttributeType.Gender,
    FaceAttributeType.Smile,
    FaceAttributeType.FacialHair,
    FaceAttributeType.HeadPose,
    FaceAttributeType.Glasses,
    FaceAttributeType.Emotion
};
var faces = await faceClient.DetectWithUrlAsync(imageUrl, true, false, requiredFaceAttributes);
```

A continuación, puede obtener referencias a los datos devueltos y realizar más operaciones según sus necesidades.

```csharp
foreach (var face in faces)
{
    var attributes = face.FaceAttributes;
    var age = attributes.Age;
    var gender = attributes.Gender;
    var smile = attributes.Smile;
    var facialHair = attributes.FacialHair;
    var headPose = attributes.HeadPose;
    var glasses = attributes.Glasses;
    var emotion = attributes.Emotion;
}
```

Para más información acerca de cada uno de los atributos, consulte la guía conceptual [Detección de caras y atributos](../concepts/face-detection.md).

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido cómo utilizar las diversas funcionalidades de la detección de caras. A continuación, puede integrar estas características en la aplicación siguiendo un tutorial en profundidad.

- [Tutorial: Creación de una aplicación WPF que muestre los datos de las caras de una imagen](../Tutorials/FaceAPIinCSharpTutorial.md)

## <a name="related-topics"></a>Temas relacionados

- [Documentación de referencia (REST)](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)
- [Documentación de referencia (SDK para .NET)](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/faceapi?view=azure-dotnet)