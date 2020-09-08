---
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-services
ms.date: 08/17/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: ba09deda155ac446693a7e1037390a3f1fd2700f
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88919391"
---
En este artículo se proporciona información y un ejemplo de código que pueden ayudarle a empezar a utilizar la biblioteca cliente de Custom Vision con C# para crear un modelo de clasificación de imágenes. Después de crearlo, puede agregar etiquetas, cargar imágenes, entrenar el proyecto, obtener la dirección URL predeterminada del punto de conexión de predicción del proyecto y utilizar el punto de conexión para probar una imagen mediante programación. Utilice este ejemplo como plantilla para crear su propia aplicación de .NET. Si desea seguir el proceso de creación y uso de un modelo de clasificación _sin_ código, consulte la [guía basada en explorador](../../getting-started-build-a-classifier.md) en su lugar.

## <a name="prerequisites"></a>Prerequisites

- Cualquier edición de [Visual Studio 2015 o 2017](https://www.visualstudio.com/downloads/).
- [!INCLUDE [create-resources](../../includes/create-resources.md)]

## <a name="get-the-custom-vision-client-library-and-sample-code"></a>Obtención de la biblioteca cliente y el código de ejemplo de Custom Vision

Para escribir una aplicación de .NET que utiliza Custom Vision, necesitará los paquetes NuGet de Custom Vision, que están incluidos en el proyecto de ejemplo que descargará, pero puede acceder a ellos individualmente aquí.

- [Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training/)
- [Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction/)

Clone o descargue el proyecto [Ejemplos de .NET de Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples). Vaya a la carpeta **CustomVision/ImageClassification** carpeta y abra _ImageClassification.csproj_ en Visual Studio.

Este proyecto de Visual Studio crea un nuevo proyecto de Custom Vision denominado __My New Project__, al que se puede acceder en el [sitio web de Custom Vision](https://customvision.ai/). A continuación, carga las imágenes para entrenar y probar un clasificador. En este proyecto, el clasificador pretende determinar si un árbol es un __abeto__ o un __cerezo japonés__.

[!INCLUDE [get-keys](../../includes/get-keys.md)]

## <a name="understand-the-code"></a>Comprendiendo el código

Abra el archivo _Program.cs_ e inspeccione el código. [Cree variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para las claves de entrenamiento y predicción denominadas `CUSTOM_VISION_TRAINING_KEY` y `CUSTOM_VISION_PREDICTION_KEY`, respectivamente. El script buscará estas variables.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ImageClassification/Program.cs?name=snippet_keys)]

Además, obtenga la dirección URL del punto de conexión de la página de configuración del sitio web de Custom Vision. Guárdela en una variable de entorno llamada `CUSTOM_VISION_ENDPOINT`. El script guarda una referencia a ella en la raíz de la clase.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ImageClassification/Program.cs?name=snippet_endpoint)]

Las siguientes líneas de código ejecutan la funcionalidad principal del proyecto.

### <a name="create-a-new-custom-vision-service-project"></a>Creación de un proyecto de Custom Vision Service

El proyecto creado se mostrará en el [sitio web de Custom Vision](https://customvision.ai/) que ha visitado anteriormente. Consulte el método [CreateProject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.customvision.training.customvisiontrainingclientextensions.createproject?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Vision_CustomVision_Training_CustomVisionTrainingClientExtensions_CreateProject_Microsoft_Azure_CognitiveServices_Vision_CustomVision_Training_ICustomVisionTrainingClient_System_String_System_String_System_Nullable_System_Guid__System_String_System_Collections_Generic_IList_System_String__) para especificar otras opciones al crear el proyecto (explicadas en la guía del portal web [Compilación de un clasificador](../../getting-started-build-a-classifier.md)).   

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ImageClassification/Program.cs?name=snippet_create)]

### <a name="create-tags-in-the-project"></a>Creación de etiquetas en el proyecto

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ImageClassification/Program.cs?name=snippet_tags)]

### <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

Se incluyen las imágenes de este proyecto. Se hace referencia a ellas en el método **LoadImagesFromDisk** de _Program.cs_. Puede cargar hasta 64 imágenes en un único lote.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ImageClassification/Program.cs?name=snippet_upload)]

### <a name="train-the-classifier-and-publish"></a>Entrenar el clasificador y publicarlo

Este código crea la primera iteración del modelo de predicción y, después, publica dicha iteración en el punto de conexión de la predicción. Puede usar el nombre de la iteración para enviar solicitudes de predicción. Una iteración no está disponible en el punto de conexión de predicción hasta que se publica.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ImageClassification/Program.cs?name=snippet_train)]

### <a name="set-the-prediction-endpoint"></a>Establecimiento del punto de conexión de la predicción

El punto de conexión de la predicción es la referencia que puede utilizar para enviar una imagen al modelo actual y obtener una predicción de clasificación.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ImageClassification/Program.cs?name=snippet_prediction_endpoint)]

### <a name="submit-an-image-to-the-default-prediction-endpoint"></a>Envío de una imagen al punto de conexión de la predicción predeterminado

En este script, se carga la imagen de prueba en el método **LoadImagesFromDisk** y la salida de la predicción del modelo se va a mostrar en la consola. El valor de la variable `publishedModelName` debe corresponder al valor de "Published as" (Publicado como) que se encuentra en la pestaña **Performance** (Rendimiento) del portal de Custom Vision. 

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ImageClassification/Program.cs?name=snippet_prediction)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Cuando se ejecuta la aplicación, se debería abrir una ventana de consola y escribir la salida siguiente:

```console
Creating new project:
        Uploading images
        Training
Done!

Making a prediction:
        Hemlock: 95.0%
        Japanese Cherry: 0.0%
```

Luego puede comprobar que la imagen de prueba (que se encuentra en **imágenes/Test/** ) se ha etiquetado correctamente. Para salir de la aplicación, presione cualquier tecla. También puede volver al [sitio web de Custom Vision](https://customvision.ai) y ver el estado actual del proyecto recién creado.

[!INCLUDE [clean-ic-project](../../includes/clean-ic-project.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora ha visto cómo realizar cada paso del proceso de clasificación de imágenes en el código. En este ejemplo se ejecuta una sola iteración de entrenamiento, pero a menudo necesitará entrenar y probar su modelo varias veces para hacerlo más preciso.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](../../test-your-model.md)
