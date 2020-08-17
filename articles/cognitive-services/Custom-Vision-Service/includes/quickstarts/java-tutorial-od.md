---
author: areddish
ms.custom: devx-track-java
ms.author: areddish
ms.service: cognitive-services
ms.date: 04/14/2020
ms.openlocfilehash: 383df0d9f3c8fef01d5185be1cf69fe203ba11a2
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87375167"
---
En este artículo se muestra cómo empezar a utilizar el SDK de Custom Vision con Java para crear un modelo de detección de objetos. Después de crearlo, puede agregar regiones etiquetadas, cargar imágenes, entrenar el proyecto, obtener la dirección URL predeterminada del punto de conexión de predicción del proyecto y utilizar el punto de conexión para probar una imagen mediante programación. Utilice este ejemplo como plantilla para crear su propia aplicación de Java.

## <a name="prerequisites"></a>Prerrequisitos

- El IDE de Java que prefiera
- [JDK 7 u 8](https://aka.ms/azure-jdks) instalado.
- [Maven](https://maven.apache.org/) instalado
- [!INCLUDE [create-resources](../../includes/create-resources.md)]

## <a name="get-the-custom-vision-sdk-and-sample-code"></a>Obtención del SDK de Custom Vision y código de ejemplo

Para escribir una aplicación de Java que utiliza Custom Vision, necesitará los paquetes maven de Custom Vision, que están incluidos en el proyecto de ejemplo que descargará, pero puede acceder a ellos individualmente aquí.

Puede encontrar el SDK de Custom Vision en el repositorio central de Maven:
- [SDK de aprendizaje](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-customvision-training)
- [SDK de predicción](https://mvnrepository.com/artifact/com.microsoft.azure.cognitiveservices/azure-cognitiveservices-customvision-prediction)

Clone o descargue el proyecto [Ejemplos del SDK de Cognitive Services para Java](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master). Vaya a la carpeta **Vision/CustomVision/** .

Este proyecto Java crea un nuevo proyecto de detección de objetos de Custom Vision denominado __Sample Java OD Project__, al que se puede acceder en el [sitio web de Custom Vision](https://customvision.ai/). A continuación, carga las imágenes para entrenar y probar un clasificador. En este proyecto, el objetivo del clasificador es determinar si un objeto es un **tenedor** o unas **tijeras**.

[!INCLUDE [get-keys](../../includes/get-keys.md)]

El programa está configurado para hacer referencia a los datos clave como variables de entorno. Vaya a la carpeta **Vision/CustomVision** y escriba los comandos de PowerShell siguientes para establecer las variable de entorno. 

> [!NOTE]
> Si utiliza un sistema operativo distinto de Windows, consulte [Configuración de las variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account?tabs=multiservice%2Cwindows#configure-an-environment-variable-for-authentication) para instrucciones.

```powershell
$env:AZURE_CUSTOMVISION_TRAINING_API_KEY ="<your training api key>"
$env:AZURE_CUSTOMVISION_PREDICTION_API_KEY ="<your prediction api key>"
```

## <a name="understand-the-code"></a>Comprendiendo el código

Cargue el proyecto `Vision/CustomVision` en el IDE de Java y abra el archivo _CustomVisionSamples.java_. Busque el método **runSample** y marque como comentario la llamada al método **ImageClassification_Sample**. Así se ejecuta el escenario de detección de objetos, que no se describe en esta guía. El método **ObjectDetection_Sample** implementa la funcionalidad principal de esta guía de inicio rápido; vaya a su definición e inspeccione el código. 

### <a name="create-a-new-custom-vision-service-project"></a>Creación de un proyecto de Custom Vision Service

Vaya al bloque de código que crea un cliente de aprendizaje y un proyecto de detección de objetos. El proyecto creado se mostrará en el [sitio web de Custom Vision](https://customvision.ai/) que ha visitado anteriormente. Consulte las sobrecargas del método [CreateProject](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.vision.customvision.training.trainings.createproject?view=azure-java-stable#com_microsoft_azure_cognitiveservices_vision_customvision_training_Trainings_createProject_String_CreateProjectOptionalParameter_) para especificar otras opciones al crear el proyecto (explicadas en la guía del portal web [Creación de un detector](../../get-started-build-detector.md)).

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_create_od)]

### <a name="add-tags-to-your-project"></a>Adición de etiquetas al proyecto

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_tags_od)]

### <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

Cuando se etiquetan imágenes en los proyectos de detección de objetos, es preciso especificar la región de cada objeto etiquetado mediante coordenadas normalizadas. Vaya a la definición del mapa `regionMap`. Este código asocia cada una de las imágenes del ejemplo a su región etiquetada.

> [!NOTE]
> Si no tiene una utilidad de hacer clic y arrastrar para marcar las coordenadas de las regiones, puede usar la interfaz de usuario web en [Customvision.ai](https://www.customvision.ai/). En este ejemplo ya se proporcionan las coordenadas.

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_od_mapping)]

Luego, vaya al bloque de código que agrega las imágenes al proyecto. Las imágenes se leen de la carpeta **src/main/resources** carpeta del proyecto y se cargan en el servicio con sus etiquetas adecuadas y las coordenadas de la región.

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_upload_od)]

El fragmento de código anterior usa dos funciones auxiliares que recuperan las imágenes como flujos de recursos y las cargan en el servicio (puede cargar hasta 64 imágenes en un único lote).

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_helpers)]

### <a name="train-the-project-and-publish"></a>Entrenar el proyecto y publicarlo

Este código crea la primera iteración del modelo de predicción y, después, publica dicha iteración en el punto de conexión de la predicción. El nombre que se da a la iteración publicada se puede utilizar para enviar solicitudes de predicción. Una iteración no está disponible en el punto de conexión de la predicción hasta que se publica.

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_train_od)]

### <a name="use-the-prediction-endpoint"></a>Uso del punto de conexión de la predicción

El punto de conexión de la predicción, representado aquí por el objeto `predictor`, es la referencia que utiliza para enviar una imagen al modelo actual y obtener una predicción de clasificación. En este ejemplo, `predictor` se define en otra parte mediante la variable de entorno de la clave de predicción.

[!code-java[](~/cognitive-services-java-sdk-samples/Vision/CustomVision/src/main/java/com/microsoft/azure/cognitiveservices/vision/customvision/samples/CustomVisionSamples.java?name=snippet_prediction_od)]

## <a name="run-the-application"></a>Ejecución de la aplicación

Para compilar y ejecutar la solución con Maven, vaya el directorio del proyecto (**Vision/CustomVision**) en un símbolo del sistema y ejecute el comando run:

```bash
mvn compile exec:java
```

En la salida de la consola podrá ver los resultados de la predicción y del. Luego puede comprobar que la imagen de prueba se ha etiquetado correctamente y que la región de detección es correcta.

[!INCLUDE [clean-od-project](../../includes/clean-od-project.md)]

## <a name="next-steps"></a>Pasos siguientes

Ya ha visto cómo todos los pasos del proceso de detección de objetos se pueden realizar en código. En este ejemplo se ejecuta una sola iteración de entrenamiento, pero a menudo necesitará entrenar y probar el modelo varias veces para hacerlo más preciso. En la guía de aprendizaje siguiente se trata la clasificación de imágenes, pero sus principios son similares a los de la detección de objetos.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](../../test-your-model.md)
