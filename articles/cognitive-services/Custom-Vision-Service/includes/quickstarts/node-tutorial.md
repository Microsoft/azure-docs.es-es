---
author: areddish
ms.author: areddish
ms.service: cognitive-services
ms.date: 08/17/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 2a8937debc38dab4b2d38b56d1c6a9c3edcbe2a7
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88508580"
---
En este artículo se muestra cómo empezar a utilizar la biblioteca cliente de Custom Vision con Node.js para crear un modelo de clasificación de imágenes. Después de crearlo, puede agregar etiquetas, cargar imágenes, entrenar el proyecto, obtener la dirección URL publicada del punto de conexión de predicción del proyecto y utilizar el punto de conexión para probar una imagen mediante programación. Utilice este ejemplo como plantilla para crear su propia aplicación de Node.js. Si desea seguir el proceso de creación y utilizar un modelo de clasificación _sin_ código, consulte la [guía basada en explorador](../../getting-started-build-a-classifier.md) en su lugar.

## <a name="prerequisites"></a>Prerrequisitos

- [Node.js 8](https://www.nodejs.org/en/download/) o versiones posteriores instalado.
- [npm](https://www.npmjs.com/) instalado.
- [!INCLUDE [create-resources](../../includes/create-resources.md)]

## <a name="install-the-custom-vision-client-library"></a>Instalación de la biblioteca cliente de Custom Vision

Para instalar la biblioteca cliente del servicio Custom Vision para Node.js, ejecute el comando siguiente en PowerShell:

```shell
npm install @azure/cognitiveservices-customvision-training
npm install @azure/cognitiveservices-customvision-prediction
```

[!INCLUDE [get-keys](../../includes/get-keys.md)]

[!INCLUDE [node-get-images](../../includes/node-get-images.md)]

## <a name="add-the-code"></a>Adición del código

Cree un archivo llamado *sample.js* en el directorio del proyecto que prefiera.

### <a name="create-the-custom-vision-service-project"></a>Creación del proyecto de Custom Vision Service

Para crear un proyecto de Custom Vision Service, agregue el siguiente código al script. Inserte las claves de suscripción en las definiciones adecuadas y establezca el valor de la ruta de acceso sampleDataRoot en la ruta de la carpeta de imágenes. Asegúrese de que el valor endPoint coincida con los puntos de conexión de entrenamiento y predicción creados en [Customvision.ai](https://www.customvision.ai/). Tenga en cuenta que la diferencia entre la creación de un proyecto de detección de objetos y uno de clasificación de imágenes es el dominio especificado en la llamada de **createProject**.

```javascript
const util = require('util');
const fs = require('fs');
const TrainingApi = require("@azure/cognitiveservices-customvision-training");
const PredictionApi = require("@azure/cognitiveservices-customvision-prediction");
const msRest = require("@azure/ms-rest-js");

const setTimeoutPromise = util.promisify(setTimeout);

const trainingKey = "<your training key>";
const predictionKey = "<your prediction key>";
const predictionResourceId = "<your prediction resource id>";
const sampleDataRoot = "<path to image files>";

const endPoint = "https://<my-resource-name>.cognitiveservices.azure.com/"

const publishIterationName = "classifyModel";

const credentials = new msRest.ApiKeyCredentials({ inHeader: { "Training-key": trainingKey } });
const trainer = new TrainingApi.TrainingAPIClient(credentials, endPoint);

(async () => {
    console.log("Creating project...");
    const sampleProject = await trainer.createProject("Sample Project");
```

### <a name="create-tags-in-the-project"></a>Creación de etiquetas en el proyecto

Para crear etiquetas de clasificación al proyecto, agregue el código siguiente al final de *sample.js*:

```javascript
    const hemlockTag = await trainer.createTag(sampleProject.id, "Hemlock");
    const cherryTag = await trainer.createTag(sampleProject.id, "Japanese Cherry");
```

### <a name="upload-and-tag-images"></a>Carga y etiquetado de imágenes

Para agregar las imágenes de ejemplo al proyecto, inserte el siguiente código después de crear la etiqueta. Este código carga cada imagen con su etiqueta correspondiente. Puede cargar hasta 64 imágenes en un único lote.

> [!NOTE]
> Tendrá que cambiar *sampleDataRoot* por la ruta de acceso a las imágenes en función del lugar en que descargó el proyecto Cognitive Services Node.js SDK Samples antes.

```javascript
    console.log("Adding images...");
    let fileUploadPromises = [];
    
    const hemlockDir = `${sampleDataRoot}/Hemlock`;
    const hemlockFiles = fs.readdirSync(hemlockDir);
    hemlockFiles.forEach(file => {
        fileUploadPromises.push(trainer.createImagesFromData(sampleProject.id, fs.readFileSync(`${hemlockDir}/${file}`), { tagIds: [hemlockTag.id] }));
    });
    
    const cherryDir = `${sampleDataRoot}/Japanese Cherry`;
    const japaneseCherryFiles = fs.readdirSync(cherryDir);
    japaneseCherryFiles.forEach(file => {
        fileUploadPromises.push(trainer.createImagesFromData(sampleProject.id, fs.readFileSync(`${cherryDir}/${file}`), { tagIds: [cherryTag.id] }));
    });
    
    await Promise.all(fileUploadPromises);
```

### <a name="train-the-classifier-and-publish"></a>Entrenar el clasificador y publicarlo

Este código crea la primera iteración del modelo de predicción y, después, publica dicha iteración en el punto de conexión de la predicción. El nombre que se da a la iteración publicada se puede utilizar para enviar solicitudes de predicción. Una iteración no está disponible en el punto de conexión de la predicción hasta que se publica.

```javascript
    console.log("Training...");
    let trainingIteration = await trainer.trainProject(sampleProject.id);
    
    // Wait for training to complete
    console.log("Training started...");
    while (trainingIteration.status == "Training") {
        console.log("Training status: " + trainingIteration.status);
        await setTimeoutPromise(1000, null);
        trainingIteration = await trainer.getIteration(sampleProject.id, trainingIteration.id)
    }
    console.log("Training status: " + trainingIteration.status);
    
    // Publish the iteration to the end point
    await trainer.publishIteration(sampleProject.id, trainingIteration.id, publishIterationName, predictionResourceId);
```

### <a name="get-and-use-the-published-iteration-on-the-prediction-endpoint"></a>Obtener y usar la iteración publicada en el punto de conexión de predicción

Para enviar una imagen al punto de conexión de la predicción y recuperar la predicción, agregue el código siguiente al final del archivo:

```javascript
    const predictor_credentials = new msRest.ApiKeyCredentials({ inHeader: { "Prediction-key": predictionKey } });
    const predictor = new PredictionApi.PredictionAPIClient(predictor_credentials, endPoint);
    const testFile = fs.readFileSync(`${sampleDataRoot}/Test/test_image.jpg`);

    const results = await predictor.classifyImage(sampleProject.id, publishIterationName, testFile);

    // Step 6. Show results
    console.log("Results:");
    results.predictions.forEach(predictedResult => {
        console.log(`\t ${predictedResult.tagName}: ${(predictedResult.probability * 100.0).toFixed(2)}%`);
    });
})()
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute *sample.js*.

```shell
node sample.js
```

La salida de la aplicación debe ser similar al texto siguiente:

```console
Creating project...
Adding images...
Training...
Training started...
Training status: Training
Training status: Training
Training status: Training
Training status: Completed
Results:
         Hemlock: 94.97%
         Japanese Cherry: 0.01%
```

Luego puede comprobar que la imagen de prueba (que se encuentra en **<base_image_url>/Images/Test/** ) se ha etiquetado correctamente. También puede volver al [sitio web de Custom Vision](https://customvision.ai) y ver el estado actual del proyecto recién creado.

[!INCLUDE [clean-ic-project](../../includes/clean-ic-project.md)]

## <a name="next-steps"></a>Pasos siguientes

Ya ha visto cómo todos los pasos del proceso de detección de objetos se pueden realizar en código. En este ejemplo se ejecuta una sola iteración de entrenamiento, pero a menudo necesitará entrenar y probar el modelo varias veces para hacerlo más preciso.

> [!div class="nextstepaction"]
> [Prueba y reentrenamiento del modelo](../../test-your-model.md)
