---
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-services
ms.date: 08/17/2020
ms.openlocfilehash: 3e4206d00d33020098770600e151f9075d160caa
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88511335"
---
En este artículo se muestra cómo empezar a utilizar el biblioteca cliente de Custom Vision con C# para crear un modelo de detección de objetos. Después de crearlo, puede agregar regiones etiquetadas, cargar imágenes, entrenar el proyecto, obtener la dirección URL predeterminada del punto de conexión de predicción del proyecto y utilizar el punto de conexión para probar una imagen mediante programación. Utilice este ejemplo como plantilla para crear su propia aplicación de .NET. 

## <a name="prerequisites"></a>Prerequisites

- Cualquier edición de [Visual Studio 2015 o 2017](https://www.visualstudio.com/downloads/).
- [!INCLUDE [create-resources](../../includes/create-resources.md)]

## <a name="get-the-custom-vision-client-library-and-sample-code"></a>Obtención de la biblioteca cliente y el código de ejemplo de Custom Vision

Para escribir una aplicación de .NET que utiliza Custom Vision, necesitará los paquetes NuGet de Custom Vision, que están incluidos en el proyecto de ejemplo que descargará, pero puede acceder a ellos individualmente aquí.

- [Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training/)
- [Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction/)

Clone o descargue el proyecto [Ejemplos de .NET de Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples). Vaya a la carpeta **CustomVision/ObjectDetection** y abra _ObjectDetection.csproj_ en Visual Studio.

Este proyecto de Visual Studio crea un nuevo proyecto de Custom Vision denominado __My New Project__, al que se puede acceder en el [sitio web de Custom Vision](https://customvision.ai/). A continuación, carga las imágenes para entrenar y probar un modelo de detección de objetos. En este proyecto, el modelo se entrena para detectar las tenedores y tijeras en las imágenes.

[!INCLUDE [get-keys](../../includes/get-keys.md)]

## <a name="understand-the-code"></a>Comprendiendo el código

Abra el archivo _Program.cs_ e inspeccione el código. [Cree variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) para las claves de entrenamiento y predicción denominadas `CUSTOM_VISION_TRAINING_KEY` y `CUSTOM_VISION_PREDICTION_KEY`, respectivamente. El script buscará estas variables.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_keys)]

Además, obtenga la dirección URL del punto de conexión de la página de configuración del sitio web de Custom Vision. Guárdela en una variable de entorno llamada `CUSTOM_VISION_ENDPOINT`. El script guarda una referencia a ella en la raíz de la clase.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_endpoint)]

### <a name="create-a-new-custom-vision-service-project"></a>Creación de un proyecto de Custom Vision Service

El siguiente fragmento de código siguiente crea un proyecto de detección de objetos. El proyecto creado se mostrará en el [sitio web de Custom Vision](https://customvision.ai/) que ha visitado anteriormente. Consulte el método [CreateProject](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.vision.customvision.training.customvisiontrainingclientextensions.createproject?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Vision_CustomVision_Training_CustomVisionTrainingClientExtensions_CreateProject_Microsoft_Azure_CognitiveServices_Vision_CustomVision_Training_ICustomVisionTrainingClient_System_String_System_String_System_Nullable_System_Guid__System_String_System_Collections_Generic_IList_System_String__) para especificar otras opciones al crear el proyecto (explicadas en la guía del portal web [Creación de un detector](../../get-started-build-detector.md)).  

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_create)]


### <a name="add-tags-to-the-project"></a>Adición de etiquetas al proyecto

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_tags)]

### <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

Cuando se etiquetan imágenes en los proyectos de detección de objetos, es preciso especificar la región de cada objeto etiquetado mediante coordenadas normalizadas. El código siguiente asocia cada una de las imágenes del ejemplo a su región etiquetada.

> [!NOTE]
> Si no tiene una utilidad de hacer clic y arrastrar para marcar las coordenadas de las regiones, puede usar la interfaz de usuario web en [Customvision.ai](https://www.customvision.ai/). En este ejemplo ya se proporcionan las coordenadas.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_upload_regions)]

Luego, esta asignación de asociaciones se usa para cargar cada imagen de ejemplo con sus coordenadas de región. Puede cargar hasta 64 imágenes en un único lote.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_upload)]

En este punto, ha cargado todas las imágenes de ejemplo y etiquetado cada una (**tenedor** o **tijeras**) con un rectángulo de píxeles asociado.

### <a name="train-the-project"></a>Entrenamiento del proyecto

Este código crea la primera iteración de aprendizaje del proyecto.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_train)]

### <a name="publish-the-current-iteration"></a>Publicación de la iteración actual

El nombre que se da a la iteración publicada se puede utilizar para enviar solicitudes de predicción. Una iteración no está disponible en el punto de conexión de predicción hasta que se publica.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_publish)]

### <a name="create-a-prediction-endpoint"></a>Creación de un punto de conexión de la predicción

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_prediction_endpoint)]

### <a name="use-the-prediction-endpoint"></a>Uso del punto de conexión de la predicción

Esta parte del script carga la imagen de prueba, consulta el punto de conexión del modelo y envía los datos de la predicción a la consola.

[!code-csharp[](~/cognitive-services-dotnet-sdk-samples/CustomVision/ObjectDetection/Program.cs?name=snippet_prediction)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Cuando se ejecuta la aplicación, se debería abrir una ventana de consola y escribir la salida siguiente:

```console
Creating new project:
        Training
Done!

Making a prediction:
        fork: 98.2% [ 0.111609578, 0.184719115, 0.6607002, 0.6637112 ]
        scissors: 1.2% [ 0.112389535, 0.119195729, 0.658031344, 0.7023591 ]
```

Luego puede comprobar que la imagen de prueba (que se encuentra en **Images/Test/** ) se ha etiquetado correctamente y que la región de detección es correcta. En este momento puede presionar cualquier tecla para salir de la aplicación.

[!INCLUDE [clean-od-project](../../includes/clean-od-project.md)]

## <a name="next-steps"></a>Pasos siguientes

Ahora ha visto cómo realizar cada paso del proceso de detección de objetos en el código. En este ejemplo se ejecuta una sola iteración de entrenamiento, pero a menudo necesitará entrenar y probar el modelo varias veces para hacerlo más preciso. En la guía siguiente se trata la clasificación de imágenes, pero sus principios son similares a los de la detección de objetos.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](../../test-your-model.md)
