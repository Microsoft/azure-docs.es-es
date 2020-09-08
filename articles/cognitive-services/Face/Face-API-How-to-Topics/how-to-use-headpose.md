---
title: Uso del atributo HeadPose
titleSuffix: Azure Cognitive Services
description: Aprenda a usar el atributo HeadPose para girar automáticamente el rectángulo de cara o detectar gestos de cabeza en una fuente de vídeo.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: sample
ms.date: 05/29/2019
ms.author: pafarley
ms.custom: devx-track-csharp
ms.openlocfilehash: fe9cc44542e97880b076d871dde82f5a02a58957
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88928979"
---
# <a name="use-the-headpose-attribute"></a>Uso del atributo HeadPose

En esta guía verá cómo puede usar el atributo HeadPose de una cara detectada para habilitar algunos escenarios clave.

## <a name="rotate-the-face-rectangle"></a>Giro del rectángulo de cara

El rectángulo facial, que se devuelve con cada rostro detectado, marca la ubicación y el tamaño de la cara en la imagen. De forma predeterminada, el rectángulo siempre se alinea con la imagen (sus lados son vertical y horizontal); esto puede resultar ineficaz para enmarcar caras en ángulo. En casos en los que desea recortar mediante programación las caras de una imagen, es mejor ser capaz de girar el rectángulo de recorte.

La aplicación de ejemplo [Cognitive Services Face WPF](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/Cognitive-Services-Face-WPF) usa el atributo HeadPose para girar los rectángulos de cara detectados.

### <a name="explore-the-sample-code"></a>Exploración del código de ejemplo

El atributo HeadPose permite girar mediante programación el rectángulo de cara. Si especifica este atributo cuando se detectan caras (consulte el artículo sobre [cómo detectar caras](HowtoDetectFacesinImage.md)), podrá realizar consultas en él más adelante. El siguiente método de la aplicación [Cognitive Services Face WPF](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/Cognitive-Services-Face-WPF) recibe una lista de objetos **DetectedFace** y devuelve una lista de objetos **[Face](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/app-samples/Cognitive-Services-Face-WPF/Sample-WPF/Controls/Face.cs)** . Aquí **Face** es una clase personalizada que almacena datos de cara, incluidas las coordenadas del rectángulo actualizado. Se calculan nuevos valores para **top**, **left**, **width** y **height** y un nuevo campo **FaceAngle** especifica el giro.

```csharp
/// <summary>
/// Calculate the rendering face rectangle
/// </summary>
/// <param name="faces">Detected face from service</param>
/// <param name="maxSize">Image rendering size</param>
/// <param name="imageInfo">Image width and height</param>
/// <returns>Face structure for rendering</returns>
public static IEnumerable<Face> CalculateFaceRectangleForRendering(IList<DetectedFace> faces, int maxSize, Tuple<int, int> imageInfo)
{
    var imageWidth = imageInfo.Item1;
    var imageHeight = imageInfo.Item2;
    var ratio = (float)imageWidth / imageHeight;
    int uiWidth = 0;
    int uiHeight = 0;
    if (ratio > 1.0)
    {
        uiWidth = maxSize;
        uiHeight = (int)(maxSize / ratio);
    }
    else
    {
        uiHeight = maxSize;
        uiWidth = (int)(ratio * uiHeight);
    }

    var uiXOffset = (maxSize - uiWidth) / 2;
    var uiYOffset = (maxSize - uiHeight) / 2;
    var scale = (float)uiWidth / imageWidth;

    foreach (var face in faces)
    {
        var left = (int)(face.FaceRectangle.Left * scale + uiXOffset);
        var top = (int)(face.FaceRectangle.Top * scale + uiYOffset);

        // Angle of face rectangles, default value is 0 (not rotated).
        double faceAngle = 0;

        // If head pose attributes have been obtained, re-calculate the left & top (X & Y) positions.
        if (face.FaceAttributes?.HeadPose != null)
        {
            // Head pose's roll value acts directly as the face angle.
            faceAngle = face.FaceAttributes.HeadPose.Roll;
            var angleToPi = Math.Abs((faceAngle / 180) * Math.PI);

            // _____       | / \ |
            // |____|  =>  |/   /|
            //             | \ / |
            // Re-calculate the face rectangle's left & top (X & Y) positions.
            var newLeft = face.FaceRectangle.Left +
                face.FaceRectangle.Width / 2 -
                (face.FaceRectangle.Width * Math.Sin(angleToPi) + face.FaceRectangle.Height * Math.Cos(angleToPi)) / 2;

            var newTop = face.FaceRectangle.Top +
                face.FaceRectangle.Height / 2 -
                (face.FaceRectangle.Height * Math.Sin(angleToPi) + face.FaceRectangle.Width * Math.Cos(angleToPi)) / 2;

            left = (int)(newLeft * scale + uiXOffset);
            top = (int)(newTop * scale + uiYOffset);
        }

        yield return new Face()
        {
            FaceId = face.FaceId?.ToString(),
            Left = left,
            Top = top,
            OriginalLeft = (int)(face.FaceRectangle.Left * scale + uiXOffset),
            OriginalTop = (int)(face.FaceRectangle.Top * scale + uiYOffset),
            Height = (int)(face.FaceRectangle.Height * scale),
            Width = (int)(face.FaceRectangle.Width * scale),
            FaceAngle = faceAngle,
        };
    }
}
```

### <a name="display-the-updated-rectangle"></a>Presentación del rectángulo actualizado

A partir de aquí, puede usar los objetos **Face** devueltos en la pantalla. Las siguientes líneas de [FaceDetectionPage.xaml](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/blob/master/app-samples/Cognitive-Services-Face-WPF/Sample-WPF/Controls/FaceDetectionPage.xaml) muestran cómo se representa el nuevo rectángulo a partir de estos datos:

```xaml
 <DataTemplate>
    <Rectangle Width="{Binding Width}" Height="{Binding Height}" Stroke="#FF26B8F4" StrokeThickness="1">
        <Rectangle.LayoutTransform>
            <RotateTransform Angle="{Binding FaceAngle}"/>
        </Rectangle.LayoutTransform>
    </Rectangle>
</DataTemplate>
```

## <a name="detect-head-gestures"></a>Detección de gestos de la cabeza

Puede detectar gestos de la cabeza como asentir o sacudir la cabeza mediante el seguimiento de cambios de HeadPose en tiempo real. Puede usar esta característica como un detector de vida personalizado.

La detección de vida es la tarea mediante la cual se determina si un sujeto es una persona real y no una imagen o una representación de vídeo. Un detector de gestos de la cabeza puede servir como ayuda para comprobar la vida, en especial por oposición a la representación de una imagen de una persona.

> [!CAUTION]
> Para detectar gestos de la cabeza en tiempo real, deberá llamar a la API Face a una velocidad alta (más de una vez por segundo). Si tiene una suscripción de nivel gratuito (f0), no podrá hacerlo. Si tiene una suscripción de nivel de pago, asegúrese de que ha calculado los costos de realizar llamadas rápidas a la API para la detección de gestos de cara.

Consulte un ejemplo práctico de detección de gestos de cabeza en el [ejemplo de HeadPose para Face](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceAPIHeadPoseSample), en GitHub.

## <a name="next-steps"></a>Pasos siguientes

Consulte la aplicación [Cognitive Services Face WPF](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/Cognitive-Services-Face-WPF) en GitHub para ver un ejemplo práctico de rectángulos de cara girados. O bien, consulte la aplicación del [ejemplo de HeadPose para Face](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples), que realiza el seguimiento del atributo HeadPose en tiempo real para detectar movimientos de cabeza.