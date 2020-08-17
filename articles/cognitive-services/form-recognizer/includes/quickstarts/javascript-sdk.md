---
title: 'Inicio rápido: Biblioteca cliente de Form Recognizer para JavaScript'
description: En este inicio rápido, empezará a trabajar con la biblioteca cliente de Form Recognizer para JavaScript.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 05/08/2020
ms.author: pafarley
ms.custom: devx-track-javascript
ms.openlocfilehash: a8b49beb463b4e9e19ee98baaef911e6dff15631
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405471"
---
[Documentación de referencia](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/formrecognizer/ai-form-recognizer/) | [Paquete (npm)](https://www.npmjs.com/package/@azure/ai-form-recognizer) | [Ejemplos](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
* Un blob de Azure Storage que contenga un conjunto de datos de entrenamiento. Consulte [Creación de un conjunto de datos de aprendizaje para un modelo personalizado](../../build-training-data-set.md) para ver sugerencias y opciones para reunir el conjunto de datos de aprendizaje. En este inicio rápido puede usar los archivos de la carpeta **Entrenar** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451).
* La versión actual de [Node.js](https://nodejs.org/)

## <a name="setting-up"></a>Instalación

### <a name="create-a-form-recognizer-azure-resource"></a>Creación de un recurso de Azure de Form Recognizer

[!INCLUDE [create resource](../create-resource.md)]

### <a name="create-environment-variables"></a>Creación de variables de entorno

[!INCLUDE [environment-variables](../environment-variables.md)]

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```

Ejecute el comando `npm init` para crear una aplicación de nodo con un archivo `package.json`. 

```console
npm init
```

Cree un archivo llamado `index.js`, ábralo e importe las bibliotecas siguientes:

```javascript
const { FormRecognizerClient, AzureKeyCredential } = require("@azure/ai-form-recognizer");
const fs = require("fs");
```

Cargue también el archivo de variable de entorno.

```javascript
// Load the .env file if it exists
require("dotenv").config();
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Instale los paquetes `ai-form-recognizer` y `dotenv` de NPM:

```console
npm install @azure/ai-form-recognizer dotenv
```

el archivo `package.json` de la aplicación se actualizará con las dependencias.


<!-- 
    Object model
-->

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas con la biblioteca cliente de Form Recognizer para JavaScript:

* [Autenticar el cliente](#authenticate-the-client)
* [Reconocer el contenido del formulario](#recognize-form-content)
* [Reconocer recibos](#recognize-receipts)
* [Entrenar un modelo personalizado](#train-a-custom-model)
* [Analizar formularios con un modelo personalizado](#analyze-forms-with-a-custom-model)
* [Administrar modelos personalizados](#manage-your-custom-models)

## <a name="authenticate-the-client"></a>Autenticar el cliente

En la función `main`, cree variables para el punto de conexión y la clave de Azure del recurso. Si ha creado la variable de entorno después de haber iniciado la aplicación, tendrá que cerrar y volver a abrir el editor, el IDE o el shell para acceder a esta variable.

```javascript
// You will need to set these environment variables or edit the following values
const endpoint = process.env["FORM_RECOGNIZER_ENDPOINT"] || "<cognitive services endpoint>";
const apiKey = process.env["FORM_RECOGNIZER_KEY"] || "<api key>";
```

Luego, autentique un objeto de cliente mediante las variables de suscripción que ha definido. Usará un objeto **AzureKeyCredential**, de modo que, si es necesario, pueda actualizar la clave de API sin crear otros objetos de cliente. También creará un objeto de cliente de entrenamiento.

```javascript
const trainingClient = new FormTrainingClient(endpoint, new AzureKeyCredential(apiKey));

const client = new FormRecognizerClient(endpoint, new AzureKeyCredential(apiKey));
```

### <a name="call-client-specific-functions"></a>Llamada a funciones específicas del cliente

El siguiente bloque de código de `main` usa los objetos de cliente para llamar a funciones de cada una de las tareas principales del SDK de Form Recognizer. Definirá estas funciones más adelante.

También tendrá que agregar referencias a las direcciones URL de los datos de entrenamiento y prueba.
* Para recuperar la dirección URL de SAS de los datos de entrenamiento del modelo personalizado, abra el Explorador de Microsoft Azure Storage, haga clic con el botón derecho en el contenedor y seleccione **Obtener firma de acceso compartido**. Asegúrese de que los permisos de **lectura** y **enumeración** están marcados y haga clic en **Create** (Crear). A continuación, copie el valor en la sección **URL**. Debe tener el formato `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.
* Para obtener una dirección URL de un formulario para prueba, puede usar los pasos anteriores para obtener la dirección URL de SAS de un documento del almacenamiento de blobs. También puede tomar la dirección URL de un documento situado en otro lugar.
* Use el método anterior para obtener también la dirección URL de una imagen de recibo, o utilice la dirección URL de la imagen de ejemplo proporcionada.

> [!NOTE]
> Los fragmentos de código de esta guía usan formularios remotos con acceso mediante direcciones URL. En cambio, si quiere procesar documentos de formulario locales, consulte los métodos relacionados en la [documentación de referencia](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/).


```javascript
const trainingDataUrl = "<url/path to the labeled training documents>";
const formUrl = "<SAS-URL-of-a-form-in-blob-storage>";
const receiptUrl = "https://docs.microsoft.com/azure/cognitive-services/form-recognizer/media/contoso-allinone.jpg";


// Call Form Recognizer scenarios:
await GetContent(recognizerClient, formUrl);
await AnalyzeReceipt(recognizerClient, receiptUrl);
modelId = await TrainModel(trainingClient, trainingDataUrl);
await AnalyzePdfForm(recognizerClient, modelId, formUrl);
await ManageModels(trainingClient, trainingDataUrl);
```

## <a name="recognize-form-content"></a>Reconocimiento del contenido del formulario

Puede usar Form Recognizer para reconocer tablas, líneas y palabras de los documentos sin necesidad de entrenar un modelo. Para reconocer el contenido de un archivo en un URI determinado, use el método **beginRecognizeContentFromUrl**.

```javascript
async function GetContent( recognizerClient, invoiceUri)
{
    const poller = await client.beginRecognizeContentFromUrl(invoiceUri);
    await poller.pollUntilDone();
    const response = poller.getResult();
```

El valor devuelto es una colección de objetos **FormPage**, uno para cada página del documento enviado. El código siguiente recorre en iteración estos objetos e imprime los pares clave-valor y los datos de tabla extraídos.

```javascript
    for (const page of response.pages) {
    console.log(
        `Page ${page.pageNumber}: width ${page.width} and height ${page.height} with unit ${page.unit}`
    );
        for (const table of page.tables) {
            for (const row of table.rows) {
                for (const cell of row.cells) {
                    console.log(`cell [${cell.rowIndex},${cell.columnIndex}] has text ${cell.text}`);
                }
            }
        }
    }
}
```

## <a name="recognize-receipts"></a>Reconocimiento de recibos

En esta sección se muestra cómo reconocer y extraer campos comunes de recibos de EE. UU. mediante un modelo de recibos entrenado previamente.

Para reconocer los recibos a partir de un URI, use el método **beginRecognizeReceiptsFromUrl**. El valor devuelto es una colección de objetos **RecognizedReceipt**, uno para cada página del documento enviado. En el código siguiente se procesa un recibo en el URI especificado y se imprimen los campos y valores principales en la consola.

```javascript
async function AnalyzeReceipt( client, receiptUri)
{
    const poller = await client.beginRecognizeReceiptsFromUrl(url, {
        includeTextDetails: true,
        onProgress: (state) => {
            console.log(`analyzing status: ${state.status}`);
        }
    });
    await poller.pollUntilDone();
    const response = poller.getResult();


    const receipt = receipts[0];
    console.log("First receipt:");
    // For supported fields recognized by the service, please refer to https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/GetAnalyzeReceiptResult.
    const receiptTypeField = receipt.recognizedForm.fields["ReceiptType"];
    if (receiptTypeField.valueType === "string") {
        console.log(`  Receipt Type: '${receiptTypeField.value || "<missing>"}', with confidence of ${receiptTypeField.confidence}`);
    }
    const merchantNameField = receipt.recognizedForm.fields["MerchantName"];
    if (merchantNameField.valueType === "string") {
        console.log(`  Merchant Name: '${merchantNameField.value || "<missing>"}', with confidence of ${merchantNameField.confidence}`);
    }
    const transactionDate = receipt.recognizedForm.fields["TransactionDate"];
    if (transactionDate.valueType === "date") {
        console.log(`  Transaction Date: '${transactionDate.value || "<missing>"}', with confidence of ${transactionDate.confidence}`);
    }
```

El siguiente bloque de código recorre en iteración los elementos detectados en el recibo e imprime los detalles en la consola.

```javascript
    const itemsField = receipt.recognizedForm.fields["Items"];
    if (itemsField.valueType === "array") {
        for (const itemField of itemsField.value || []) {
            if (itemField.valueType === "object") {
                const itemNameField = itemField.value["Name"];
                if (itemNameField.valueType === "string") {
                    console.log(`    Item Name: '${itemNameField.value || "<missing>"}', with confidence of ${itemNameField.confidence}`);
                }
            }
        }
    }
}
```


## <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

En esta sección se muestra cómo entrenar un modelo con sus propios datos. Un modelo entrenado puede generar datos estructurados que incluyan las relaciones clave-valor del documento de formulario original. Después de entrenar el modelo, puede probarlo, volver a entrenarlo y finalmente usarlo para extraer datos de forma confiable de más formularios en función de las propias necesidades.

> [!NOTE]
> También puede entrenar modelos con una interfaz gráfica de usuario, como la [herramienta de etiquetado de ejemplo de Form Recognizer](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Entrenamiento de un modelo sin etiquetas

Entrene modelos personalizados para reconocer todos los campos y valores que se encuentran en los formularios personalizados sin etiquetar manualmente los documentos de entrenamiento.

La función siguiente entrena un modelo en un conjunto de documentos determinado e imprime el estado del modelo en la consola. 

```javascript
async function TrainModel(trainingClient, trainingDataUrl)
{
    const poller = await trainingClient.beginTraining(trainingDataUrl, false, {
        onProgress: (state) => {
            console.log(`training status: ${state.status}`);
        }
    });
    await poller.pollUntilDone();
    const response = poller.getResult();
    
    if (!response) {
        throw new Error("Expecting valid response!");
    }
    
    console.log(`Model ID: ${response.modelId}`);
    console.log(`Status: ${response.status}`);
    console.log(`Created on: ${response.requestedOn}`);
    console.log(`Last modified: ${response.completedOn}`);
```

El objeto devuelto **CustomFormModel** contiene información sobre los tipos de formulario que el modelo puede reconocer y los campos que puede extraer de cada tipo de formulario. El siguiente bloque de código imprime esta información en la consola.

```javascript
    if (response.submodels) {
        for (const submodel of response.submodels) {
            // since the training data is unlabeled, we are unable to return the accuracy of this model
            console.log("We have recognized the following fields");
            for (const key in submodel.fields) {
                const field = submodel.fields[key];
                console.log(`The model found field '${field.name}'`);
            }
        }
    }
```

Por último, este método devuelve el identificador único del modelo.

```csharp
    return response.modelId;
}
```

### <a name="train-a-model-with-labels"></a>Entrenamiento de un modelo con etiquetas

También puede entrenar modelos personalizados mediante el etiquetado manual de los documentos de entrenamiento. En algunos escenarios, el entrenamiento con etiquetas conduce a un mejor rendimiento. Para realizar el entrenamiento con etiquetas, debe disponer de archivos de información de etiquetas especiales ( *\<filename\>.pdf.labels.json*) en el contenedor de almacenamiento de blobs junto con los documentos de entrenamiento. La [herramienta de etiquetado de ejemplo de Form Recognizer](../../quickstarts/label-tool.md) proporciona una interfaz de usuario para ayudarle a crear estos archivos de etiqueta. Cuando los tenga, puede llamar al método **beginTraining** con el parámetro *uselabels* establecido en `true`.

```javascript
async function TrainModelWithLabelsAsync(
    const poller = await trainingClient.beginTraining(trainingDataUrl, true, {
        onProgress: (state) => {
            console.log(`training status: ${state.status}`);
        }
    });
    await poller.pollUntilDone();
    const model = poller.getResult();
    
    return model.modelId;
}
```

## <a name="analyze-forms-with-a-custom-model"></a>Analizar formularios con un modelo personalizado

En esta sección, se muestra cómo extraer información de clave-valor y otro contenido de los tipos de formulario personalizados, con los modelos que ha entrenado con sus propios formularios.

> [!IMPORTANT]
> Para implementar este escenario, debe haber entrenado ya un modelo para que pueda pasar su identificador al método siguiente. Consulte la sección [Entrenamiento de un modelo](#train-a-model-without-labels).

Usará el método **beginRecognizeCustomFormsFromUrl**. El valor devuelto es una colección de objetos **RecognizedForm**, uno para cada página del documento enviado.

```javascript
// Analyze PDF form document at an accessible URL
async function AnalyzePdfForm(client, modelId, formUrl)
{    
    const poller = await client.beginRecognizeCustomFormsFromUrl(modelId, formUrl, {
        onProgress: (state) => {
            console.log(`status: ${state.status}`);
        }
    });
    await poller.pollUntilDone();
    const response = poller.getResult();
```

El código siguiente imprime los resultados del análisis en la consola. Imprime cada campo reconocido y el valor correspondiente, junto con una puntuación de confianza.

```javascript
    console.log("Fields:");
    for (const fieldName in form.fields) {
        // each field is of type FormField
        const field = form.fields[fieldName];
        console.log(
            `Field ${fieldName} has value '${field.value}' with a confidence score of ${field.confidence}`
        );
    }
}
```

## <a name="manage-your-custom-models"></a>Administración de modelos personalizados

En esta sección se muestra cómo administrar los modelos personalizados almacenados en su cuenta. Como ejemplo, el código siguiente realiza todas las tareas de administración del modelo en una única función. Comience por copiar la signatura de función siguiente:

```javascript
async function ManageModels(trainingClient, trainingFileUrl)
{
```

### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Comprobación del número de modelos de la cuenta de recursos de FormRecognizer

El siguiente bloque de código comprueba cuántos modelos se han guardado en la cuenta de Form Recognizer y compara esta cifra con el límite de la cuenta.

```csharp
    // First, we see how many custom models we have, and what our limit is
    const accountProperties = await trainingClient.getAccountProperties();
    console.log(
        `Our account has ${accountProperties.customModelCount} custom models, and we can have at most ${accountProperties.customModelLimit} custom models`
    );
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Enumeración de los modelos almacenados actualmente en la cuenta de recursos

El siguiente bloque de código enumera los modelos actuales de la cuenta e imprime los detalles en la consola. También se guarda una referencia al primer modelo.

```javascript
    // Next, we get a paged async iterator of all of our custom models
    const result = trainingClient.listCustomModels();

    // We could print out information about first ten models
    // and save the first model id for later use
    let i = 0;
    let firstModel;
    for await (const model of result) {
        console.log(`model ${i++}:`);
        console.log(model);
        if (i === 1) {
            firstModel = model;
        }
        if (i > 10) {
            break;
        }
    }
```

### <a name="get-a-specific-model-using-the-models-id"></a>Obtención de un modelo específico mediante el identificador del modelo

El siguiente bloque de código usa el identificador de modelo guardado en la sección anterior y lo emplea para recuperar detalles sobre el modelo.

```csharp
    // Now we'll get the first custom model in the paged list
    const model = await trainingClient.getModel(firstModel.modelId);
    console.log(`Model Id: ${model.modelId}`);
    console.log(`Status: ${model.status}`);
    console.log("Documents used in training: [");
    for (const doc of model.trainingDocuments || []) {
        console.log(`  ${doc.documentName}`);
    }
    console.log("]");
```

### <a name="delete-a-model-from-the-resource-account"></a>Eliminación de un modelo de la cuenta de recursos

También puede eliminar un modelo de su cuenta haciendo referencia a su identificador. Este código elimina el modelo usado en la sección anterior.

```javascript
    await client.deleteModel(firstModel.modelId);
    try {
        const deleted = await trainingClient.deleteModel(firstModel.modelId);
        console.log(deleted);
    } catch (err) {
        // Expected
        console.log(`Model with id ${firstModel.modelId} has been deleted`);
    }
}
```


## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el comando `node` en el archivo de inicio rápido.

```console
node index.js
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Solución de problemas

### <a name="enable-logs"></a>Habilitamiento de registros
Puede establecer la siguiente variable de entorno para ver los registros de depuración cuando se usa esta biblioteca.

```
export DEBUG=azure*
```

Para obtener instrucciones más detalladas sobre cómo habilitar los registros, vea los documentos del paquete [@azure/logger](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/core/logger).


## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se ha usado la biblioteca cliente de Python para Form Recognizer para entrenar modelos y analizar formularios de maneras diferentes. A continuación, obtenga sugerencias para crear un mejor conjunto de datos de entrenamiento y generar modelos más precisos.

> [!div class="nextstepaction"]
> [Creación de un conjunto de datos de aprendizaje](../../build-training-data-set.md)

* [¿Qué es Form Recognizer?](../../overview.md)
* El código de ejemplo de esta guía (y más) se puede encontrar en [GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples).