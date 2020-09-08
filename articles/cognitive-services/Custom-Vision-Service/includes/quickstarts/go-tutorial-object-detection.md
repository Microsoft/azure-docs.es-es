---
author: areddish
ms.author: areddish
ms.service: cognitive-services
ms.date: 08/17/2020
ms.openlocfilehash: a56b95fe4f6b7005e823ebe80fd2e74ed1cf7725
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88511351"
---
En este artículo se proporciona información y código de ejemplo para ayudarle a empezar a usar la biblioteca cliente de Custom Vision con Go para crear un modelo de detección de objetos. Después de crearlo, puede agregar regiones etiquetadas, cargar imágenes, entrenar el proyecto, obtener la dirección URL publicada del punto de conexión de predicción del proyecto y utilizar el punto de conexión para probar una imagen mediante programación. Use este ejemplo como plantilla para crear su propia aplicación de Go.

## <a name="prerequisites"></a>Prerrequisitos

- [Go 1.8+](https://golang.org/doc/install)
- [!INCLUDE [create-resources](../../includes/create-resources.md)]

## <a name="install-the-custom-vision-client-library"></a>Instalación de la biblioteca cliente de Custom Vision

Para instalar la biblioteca cliente del servicio Custom Vision para Go, ejecute el comando siguiente en PowerShell:

```shell
go get -u github.com/Azure/azure-sdk-for-go/...
```

O bien, si usa `dep`, dentro de su repositorio ejecute:
```shell
dep ensure -add github.com/Azure/azure-sdk-for-go
```

[!INCLUDE [get-keys](../../includes/get-keys.md)]

[!INCLUDE [python-get-images](../../includes/python-get-images.md)]

## <a name="add-the-code"></a>Adición del código

Cree un archivo llamado *sample.go* en el directorio de proyecto que prefiera.

### <a name="create-the-custom-vision-service-project"></a>Creación del proyecto de Custom Vision Service

Para crear un proyecto de Custom Vision Service, agregue el siguiente código al script. Inserte las claves de la suscripción en las definiciones pertinentes. Además, obtenga la dirección URL del punto de conexión de la página de configuración del sitio web de Custom Vision.

Consulte el método [CreateProject](https://docs.microsoft.com/java/api/com.microsoft.azure.cognitiveservices.vision.customvision.training.trainings.createproject?view=azure-java-stable#com_microsoft_azure_cognitiveservices_vision_customvision_training_Trainings_createProject_String_CreateProjectOptionalParameter_) para especificar otras opciones al crear el proyecto (explicadas en la guía del portal web [Creación de un detector](../../get-started-build-detector.md)).

```go
import(
    "context"
    "bytes"
    "fmt"
    "io/ioutil"
    "path"
    "log"
    "time"
    "github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v3.0/customvision/training"
    "github.com/Azure/azure-sdk-for-go/services/cognitiveservices/v3.0/customvision/prediction"
)

var (
    training_key string = "<your training key>"
    prediction_key string = "<your prediction key>"
    prediction_resource_id = "<your prediction resource id>"
    endpoint string = "<your endpoint URL>"
    project_name string = "Go Sample OD Project"
    iteration_publish_name = "detectModel"
    sampleDataDirectory = "<path to sample images>"
)

func main() {
    fmt.Println("Creating project...")

    ctx = context.Background()

    trainer := training.New(training_key, endpoint)

    var objectDetectDomain training.Domain
    domains, _ := trainer.GetDomains(ctx)

    for _, domain := range *domains.Value {
        fmt.Println(domain, domain.Type)
        if domain.Type == "ObjectDetection" && *domain.Name == "General" {
            objectDetectDomain = domain
            break
        }
    }
    fmt.Println("Creating project...")
    project, _ := trainer.CreateProject(ctx, project_name, "", objectDetectDomain.ID, "")
```

### <a name="create-tags-in-the-project"></a>Creación de etiquetas en el proyecto

Para crear etiquetas de clasificación para el proyecto, agregue el código siguiente al final de *sample.go*:

```Go
# Make two tags in the new project
forkTag, _ := trainer.CreateTag(ctx, *project.ID, "fork", "A fork", string(training.Regular))
scissorsTag, _ := trainer.CreateTag(ctx, *project.ID, "scissors", "Pair of scissors", string(training.Regular))
```

### <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

Cuando se etiquetan imágenes en los proyectos de detección de objetos, es preciso especificar la región de cada objeto etiquetado mediante coordenadas normalizadas.

> [!NOTE]
> Si no tiene una utilidad de hacer clic y arrastrar para marcar las coordenadas de las regiones, puede usar la interfaz de usuario web en [Customvision.ai](https://www.customvision.ai/). En este ejemplo ya se proporcionan las coordenadas.

Para agregar las imágenes, etiquetas y regiones al proyecto, inserte el siguiente código después de crear la etiqueta. Para este tutorial, las regiones se codificaron de forma rígida en el código. Las regiones especifican el rectángulo delimitador en coordenadas normalizadas y las coordenadas se proporcionan en el siguiente orden: izquierda, superior, ancho y alto.

```Go
forkImageRegions := map[string][4]float64{
    "fork_1.jpg": [4]float64{ 0.145833328, 0.3509314, 0.5894608, 0.238562092 },
    "fork_2.jpg": [4]float64{ 0.294117659, 0.216944471, 0.534313738, 0.5980392 },
    "fork_3.jpg": [4]float64{ 0.09191177, 0.0682516545, 0.757352948, 0.6143791 },
    "fork_4.jpg": [4]float64{ 0.254901975, 0.185898721, 0.5232843, 0.594771266 },
    "fork_5.jpg": [4]float64{ 0.2365196, 0.128709182, 0.5845588, 0.71405226 },
    "fork_6.jpg": [4]float64{ 0.115196079, 0.133611143, 0.676470637, 0.6993464 },
    "fork_7.jpg": [4]float64{ 0.164215669, 0.31008172, 0.767156839, 0.410130739 },
    "fork_8.jpg": [4]float64{ 0.118872553, 0.318251669, 0.817401946, 0.225490168 },
    "fork_9.jpg": [4]float64{ 0.18259804, 0.2136765, 0.6335784, 0.643790841 },
    "fork_10.jpg": [4]float64{ 0.05269608, 0.282303959, 0.8088235, 0.452614367 },
    "fork_11.jpg": [4]float64{ 0.05759804, 0.0894935, 0.9007353, 0.3251634 },
    "fork_12.jpg": [4]float64{ 0.3345588, 0.07315363, 0.375, 0.9150327 },
    "fork_13.jpg": [4]float64{ 0.269607842, 0.194068655, 0.4093137, 0.6732026 },
    "fork_14.jpg": [4]float64{ 0.143382356, 0.218578458, 0.7977941, 0.295751631 },
    "fork_15.jpg": [4]float64{ 0.19240196, 0.0633497, 0.5710784, 0.8398692 },
    "fork_16.jpg": [4]float64{ 0.140931368, 0.480016381, 0.6838235, 0.240196079 },
    "fork_17.jpg": [4]float64{ 0.305147052, 0.2512582, 0.4791667, 0.5408496 },
    "fork_18.jpg": [4]float64{ 0.234068632, 0.445702642, 0.6127451, 0.344771236 },
    "fork_19.jpg": [4]float64{ 0.219362751, 0.141781077, 0.5919118, 0.6683006 },
    "fork_20.jpg": [4]float64{ 0.180147052, 0.239820287, 0.6887255, 0.235294119 },
}

scissorsImageRegions := map[string][4]float64{
    "scissors_1.jpg": [4]float64{ 0.4007353, 0.194068655, 0.259803921, 0.6617647 },
    "scissors_2.jpg": [4]float64{ 0.426470578, 0.185898721, 0.172794119, 0.5539216 },
    "scissors_3.jpg": [4]float64{ 0.289215684, 0.259428144, 0.403186262, 0.421568632 },
    "scissors_4.jpg": [4]float64{ 0.343137264, 0.105833367, 0.332107842, 0.8055556 },
    "scissors_5.jpg": [4]float64{ 0.3125, 0.09766343, 0.435049027, 0.71405226 },
    "scissors_6.jpg": [4]float64{ 0.379901975, 0.24308826, 0.32107842, 0.5718954 },
    "scissors_7.jpg": [4]float64{ 0.341911763, 0.20714055, 0.3137255, 0.6356209 },
    "scissors_8.jpg": [4]float64{ 0.231617644, 0.08459154, 0.504901946, 0.8480392 },
    "scissors_9.jpg": [4]float64{ 0.170343131, 0.332957536, 0.767156839, 0.403594762 },
    "scissors_10.jpg": [4]float64{ 0.204656869, 0.120539248, 0.5245098, 0.743464053 },
    "scissors_11.jpg": [4]float64{ 0.05514706, 0.159754932, 0.799019635, 0.730392158 },
    "scissors_12.jpg": [4]float64{ 0.265931368, 0.169558853, 0.5061275, 0.606209159 },
    "scissors_13.jpg": [4]float64{ 0.241421565, 0.184264734, 0.448529422, 0.6830065 },
    "scissors_14.jpg": [4]float64{ 0.05759804, 0.05027781, 0.75, 0.882352948 },
    "scissors_15.jpg": [4]float64{ 0.191176474, 0.169558853, 0.6936275, 0.6748366 },
    "scissors_16.jpg": [4]float64{ 0.1004902, 0.279036, 0.6911765, 0.477124184 },
    "scissors_17.jpg": [4]float64{ 0.2720588, 0.131977156, 0.4987745, 0.6911765 },
    "scissors_18.jpg": [4]float64{ 0.180147052, 0.112369314, 0.6262255, 0.6666667 },
    "scissors_19.jpg": [4]float64{ 0.333333343, 0.0274019931, 0.443627447, 0.852941155 },
    "scissors_20.jpg": [4]float64{ 0.158088237, 0.04047389, 0.6691176, 0.843137264 },
}
```
Luego, use esta asignación de asociaciones para cargar cada imagen de ejemplo con sus coordenadas de región (puede cargar hasta 64 imágenes en un único lote). Agregue el código siguiente:

> [!NOTE]
> Tendrá que cambiar la ruta de acceso a las imágenes en función de donde haya descargado anteriormente el proyecto Cognitive Services Go SDK Samples.

```Go
// Go through the data table above and create the images
fmt.Println("Adding images...")
var fork_images []training.ImageFileCreateEntry
for file, region := range forkImageRegions {
    imageFile, _ := ioutil.ReadFile(path.Join(sampleDataDirectory, "fork", file))

    imageRegion := training.Region { 
        TagID:forkTag.ID,
        Left:&region[0],
        Top:&region[1],
        Width:&region[2],
        Height:&region[3],
    }

    fork_images = append(fork_images, training.ImageFileCreateEntry {
        Name: &file,
        Contents: &imageFile,
        Regions: &[]training.Region{ imageRegion },
    })
}
    
fork_batch, _ := trainer.CreateImagesFromFiles(ctx, *project.ID, training.ImageFileCreateBatch{ 
    Images: &fork_images,
})

if (!*fork_batch.IsBatchSuccessful) {
    fmt.Println("Batch upload failed.")
}

var scissor_images []training.ImageFileCreateEntry
for file, region := range scissorsImageRegions {
    imageFile, _ := ioutil.ReadFile(path.Join(sampleDataDirectory, "scissors", file))

    imageRegion := training.Region { 
        TagID:scissorsTag.ID,
        Left:&region[0],
        Top:&region[1],
        Width:&region[2],
        Height:&region[3],
    }

    scissor_images = append(scissor_images, training.ImageFileCreateEntry {
        Name: &file,
        Contents: &imageFile,
        Regions: &[]training.Region{ imageRegion },
    })
}
    
scissor_batch, _ := trainer.CreateImagesFromFiles(ctx, *project.ID, training.ImageFileCreateBatch{ 
    Images: &scissor_images,
})
    
if (!*scissor_batch.IsBatchSuccessful) {
    fmt.Println("Batch upload failed.")
}     
```

### <a name="train-the-project-and-publish"></a>Entrenar el proyecto y publicarlo

Este código crea la primera iteración del modelo de predicción y, después, publica dicha iteración en el punto de conexión de la predicción. El nombre que se da a la iteración publicada se puede utilizar para enviar solicitudes de predicción. Una iteración no está disponible en el punto de conexión de predicción hasta que se publica.

```go
iteration, _ := trainer.TrainProject(ctx, *project.ID)
fmt.Println("Training status:", *iteration.Status)
for {
    if *iteration.Status != "Training" {
        break
    }
    time.Sleep(5 * time.Second)
    iteration, _ = trainer.GetIteration(ctx, *project.ID, *iteration.ID)
    fmt.Println("Training status:", *iteration.Status)
}

trainer.PublishIteration(ctx, *project.ID, *iteration.ID, iteration_publish_name, prediction_resource_id))
```

### <a name="get-and-use-the-published-iteration-on-the-prediction-endpoint"></a>Obtener y usar la iteración publicada en el punto de conexión de predicción

Para enviar una imagen al punto de conexión de la predicción y recuperar la predicción, agregue el código siguiente al final del archivo:

```go
    fmt.Println("Predicting...")
    predictor := prediction.New(prediction_key, endpoint)

    testImageData, _ := ioutil.ReadFile(path.Join(sampleDataDirectory, "Test", "test_od_image.jpg"))
    results, _ := predictor.DetectImage(ctx, *project.ID, iteration_publish_name, ioutil.NopCloser(bytes.NewReader(testImageData)), "")

    for _, prediction := range *results.Predictions    {
        boundingBox := *prediction.BoundingBox

        fmt.Printf("\t%s: %.2f%% (%.2f, %.2f, %.2f, %.2f)", 
            *prediction.TagName,
            *prediction.Probability * 100,
            *boundingBox.Left,
            *boundingBox.Top,
            *boundingBox.Width,
            *boundingBox.Height)
        fmt.Println("")
    }
}
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute *sample.go*.

```shell
go run sample.go
```

La salida de la aplicación debe aparecer en la consola. Luego puede comprobar que la imagen de prueba (que se encuentra en **samples/vision/images/Test**) se ha etiquetado correctamente y que la región de detección es correcta.

[!INCLUDE [clean-od-project](../../includes/clean-od-project.md)]

## <a name="next-steps"></a>Pasos siguientes

Ya ha visto cómo todos los pasos del proceso de detección de objetos se pueden realizar en código. En este ejemplo se ejecuta una sola iteración de entrenamiento, pero a menudo necesitará entrenar y probar el modelo varias veces para hacerlo más preciso. En la guía de aprendizaje siguiente se trata la clasificación de imágenes, pero sus principios son similares a los de la detección de objetos.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](../../test-your-model.md)
