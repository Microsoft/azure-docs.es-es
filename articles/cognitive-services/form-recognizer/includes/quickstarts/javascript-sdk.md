---
title: 'Inicio rápido: Biblioteca cliente de Form Recognizer para JavaScript'
description: En este inicio rápido, empezará a trabajar con la biblioteca cliente de Form Recognizer para JavaScript.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 08/21/2020
ms.author: pafarley
ms.custom: devx-track-javascript, devx-track-csharp
ms.openlocfilehash: cc72fa9e05dd25ef2d63f126f0fbb45841cb799c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934365"
---
> [!IMPORTANT]
> * Actualmente, el SDK de Form Recognizer tiene como destino la versión v2.0 del servicio Form Recognizer.
> * Por motivos de simplicidad, en el código de este artículo se usan métodos sincrónicos y almacenamiento de credenciales no protegidas. Para más información, consulte la siguiente documentación de referencia. 

[Documentación de referencia](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/formrecognizer/ai-form-recognizer/) | [Paquete (npm)](https://www.npmjs.com/package/@azure/ai-form-recognizer) | [Ejemplos](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/formrecognizer/ai-form-recognizer/samples)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Un blob de Azure Storage que contenga un conjunto de datos de entrenamiento. Consulte [Creación de un conjunto de datos de aprendizaje para un modelo personalizado](../../build-training-data-set.md) para ver sugerencias y opciones para reunir el conjunto de datos de aprendizaje. En este inicio rápido puede usar los archivos de la carpeta **Entrenar** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451).
* La versión actual de [Node.js](https://nodejs.org/)
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="cree un recurso de Form Recognizer"  target="_blank">create a Form Recognizer resource <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Form Recognizer API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

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
const { FormRecognizerClient, FormTrainingClient, AzureKeyCredential } = require("@azure/ai-form-recognizer");
const fs = require("fs");
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Instale el paquete NPM `ai-form-recognizer`:

```console
npm install @azure/ai-form-recognizer
```

el archivo `package.json` de la aplicación se actualizará con las dependencias.

## <a name="object-model"></a>Modelo de objetos 

Con Form Recognizer, puede crear dos tipos de cliente diferentes. El primero, `FormRecognizerClient`, se utiliza para consultar el servicio con los campos de formulario y el contenido reconocidos. El segundo, `FormTrainingClient`, se usa para crear y administrar los modelos personalizados que puede usar para mejorar el reconocimiento. 

### <a name="formrecognizerclient"></a>FormRecognizerClient
`FormRecognizerClient` proporciona operaciones para:

 * El reconocimiento de los campos de formulario y el contenido mediante modelos personalizados entrenados para reconocer los formularios personalizados. Estos valores se devuelven en una colección de objetos `RecognizedForm`.
 * El reconocimiento del contenido de los formularios, incluidas tablas, líneas y palabras, sin necesidad de entrenar un modelo. El contenido del formulario se devuelve en una colección de objetos `FormPage`.
 * El reconocimiento de los campos comunes de las recepciones mediante un modelo de recepción entrenado previamente en el servicio Form Recognizer. Estos campos y metadatos se devuelven en una colección de `RecognizedReceipt`.

### <a name="formtrainingclient"></a>FormTrainingClient
`FormTrainingClient` proporciona operaciones para:

* El entrenamiento de modelos personalizados para reconocer todos los campos y los valores que se encuentren en los formularios personalizados. Se devuelve un `CustomFormModel` que indica los tipos de formulario que el modelo reconocerá y los campos que se extraerán para cada tipo de formulario. Consulte la [documentación del servicio sobre el entrenamiento de modelos sin etiquetas](#train-a-model-without-labels) si desea obtener una explicación más detallada de la creación de un conjunto de datos de entrenamiento.
* El entrenamiento de modelos personalizados para reconocer los campos y los valores concretos que especifique mediante la etiqueta de los formularios personalizados. Se devuelve un `CustomFormModel` que indica los campos que el modelo va a extraer, así como la precisión estimada para cada campo. Consulte la [documentación del servicio sobre el entrenamiento de modelos con etiqueta](#train-a-model-with-labels) para obtener una explicación más detallada de cómo aplicar etiquetas a un conjunto de datos de entrenamiento.
* La administración de los modelos creados en una cuenta.
* La copia de un modelo personalizado entre recursos de Form Recognizer.

Tenga en cuenta que los modelos también se pueden entrenar mediante una interfaz gráfica de usuario, como la [herramienta de etiquetado de Form Recognizer](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/quickstarts/label-tool).

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas con la biblioteca cliente de Form Recognizer para JavaScript:

* [Autenticar el cliente](#authenticate-the-client)
* [Reconocer el contenido del formulario](#recognize-form-content)
* [Reconocer recibos](#recognize-receipts)
* [Entrenar un modelo personalizado](#train-a-custom-model)
* [Analizar formularios con un modelo personalizado](#analyze-forms-with-a-custom-model)
* [Administrar modelos personalizados](#manage-your-custom-models)

## <a name="authenticate-the-client"></a>Autenticar el cliente

A continuación, cree variables para el punto de conexión y la clave de Azure del recurso. 

```javascript
// You will need to set these environment variables or edit the following values
const endpoint = "<paste-your-form-recognizer-endpoint-here>";
const apiKey = "<paste-your-form-recognizer-key-here>";
```

Luego, autentique un objeto de cliente mediante las variables de suscripción que ha definido. Usará un objeto `AzureKeyCredential`, de modo que, si es necesario, pueda actualizar la clave de API sin crear otros objetos de cliente. También creará un objeto de cliente de entrenamiento.

```javascript
const trainingClient = new FormTrainingClient(endpoint, new AzureKeyCredential(apiKey));
const client = new FormRecognizerClient(endpoint, new AzureKeyCredential(apiKey));
```

## <a name="assets-for-testing"></a>Recursos para pruebas

Los fragmentos de código de esta guía usan formularios remotos con acceso mediante direcciones URL. En cambio, si quiere procesar documentos de formulario locales, consulte los métodos relacionados en la [documentación de referencia](https://docs.microsoft.com/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer) y los [ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples).

También tendrá que agregar referencias a las direcciones URL de los datos de entrenamiento y prueba.
* Para recuperar la dirección URL de SAS de los datos de entrenamiento del modelo personalizado, abra el Explorador de Microsoft Azure Storage, haga clic con el botón derecho en el contenedor y seleccione **Obtener firma de acceso compartido**. Asegúrese de que los permisos de **lectura** y **enumeración** están marcados y haga clic en **Create** (Crear). A continuación, copie el valor en la sección **URL**. Debe tener el formato `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.
* Use las imágenes de envío y de recepción que se incluyen en los ejemplos siguientes (también disponibles en [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms)). O bien, puede seguir los pasos anteriores para obtener la dirección URL de SAS de un documento individual en el almacenamiento de blobs. 

> [!NOTE]
> Los fragmentos de código de esta guía usan formularios remotos con acceso mediante direcciones URL. En cambio, si quiere procesar documentos de formulario locales, consulte los métodos relacionados en la [documentación de referencia](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/).

## <a name="recognize-form-content"></a>Reconocimiento del contenido del formulario

Puede usar Form Recognizer para reconocer tablas, líneas y palabras de los documentos sin necesidad de entrenar un modelo. Para reconocer el contenido de un archivo en un URI determinado, use el método `beginRecognizeContentFromUrl`.

```javascript
async function recognizeContent() {
    const formUrl = "https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/tests/sample_forms/forms/Invoice_1.pdf";
    const poller = await client.beginRecognizeContentFromUrl(formUrl);
    const pages = await poller.pollUntilDone();

    if (!pages || pages.length === 0) {
        throw new Error("Expecting non-empty list of pages!");
    }

    for (const page of pages) {
        console.log(
            `Page ${page.pageNumber}: width ${page.width} and height ${page.height} with unit ${page.unit}`
        );
        for (const table of page.tables) {
            for (const cell of table.cells) {
                console.log(`cell [${cell.rowIndex},${cell.columnIndex}] has text ${cell.text}`);
            }
        }
    }
}

recognizeContent().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

### <a name="output"></a>Output

```console
Page 1: width 8.5 and height 11 with unit inch
cell [0,0] has text Invoice Number
cell [0,1] has text Invoice Date
cell [0,2] has text Invoice Due Date
cell [0,3] has text Charges
cell [0,5] has text VAT ID
cell [1,0] has text 34278587
cell [1,1] has text 6/18/2017
cell [1,2] has text 6/24/2017
cell [1,3] has text $56,651.49
cell [1,5] has text PT
```

## <a name="recognize-receipts"></a>Reconocimiento de recibos

En esta sección se muestra cómo reconocer y extraer campos comunes de recibos de EE. UU. mediante un modelo de recibos entrenado previamente.

Para reconocer los recibos a partir de un URI, use el método `beginRecognizeReceiptsFromUrl`. En el código siguiente se procesa un recibo en el URI especificado y se imprimen los campos y valores principales en la consola.

```javascript
async function recognizeReceipt() {
    receiptUrl = "https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/tests/sample_forms/receipt/contoso-receipt.png";
    const client = new FormRecognizerClient(endpoint, new AzureKeyCredential(apiKey));
    const poller = await client.beginRecognizeReceiptsFromUrl(receiptUrl, {
        onProgress: (state) => { console.log(`status: ${state.status}`); }
    });

    const receipts = await poller.pollUntilDone();

    if (!receipts || receipts.length <= 0) {
        throw new Error("Expecting at lease one receipt in analysis result");
    }

    const receipt = receipts[0];
    console.log("First receipt:");
    const receiptTypeField = receipt.fields["ReceiptType"];
    if (receiptTypeField.valueType === "string") {
        console.log(`  Receipt Type: '${receiptTypeField.value || "<missing>"}', with confidence of ${receiptTypeField.confidence}`);
    }
    const merchantNameField = receipt.fields["MerchantName"];
    if (merchantNameField.valueType === "string") {
        console.log(`  Merchant Name: '${merchantNameField.value || "<missing>"}', with confidence of ${merchantNameField.confidence}`);
    }
    const transactionDate = receipt.fields["TransactionDate"];
    if (transactionDate.valueType === "date") {
        console.log(`  Transaction Date: '${transactionDate.value || "<missing>"}', with confidence of ${transactionDate.confidence}`);
    }
    const itemsField = receipt.fields["Items"];
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
    const totalField = receipt.fields["Total"];
    if (totalField.valueType === "number") {
        console.log(`  Total: '${totalField.value || "<missing>"}', with confidence of ${totalField.confidence}`);
    }
}

recognizeReceipt().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

### <a name="output"></a>Resultados

```console
status: notStarted
status: running
status: succeeded
First receipt:
  Receipt Type: 'Itemized', with confidence of 0.659
  Merchant Name: 'Contoso Contoso', with confidence of 0.516
  Transaction Date: 'Sun Jun 09 2019 17:00:00 GMT-0700 (Pacific Daylight Time)', with confidence of 0.985
    Item Name: '8GB RAM (Black)', with confidence of 0.916
    Item Name: 'SurfacePen', with confidence of 0.858
  Total: '1203.39', with confidence of 0.774
```

## <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

En esta sección se muestra cómo entrenar un modelo con sus propios datos. Un modelo entrenado puede generar datos estructurados que incluyan las relaciones clave-valor del documento de formulario original. Después de entrenar el modelo, puede probarlo, volver a entrenarlo y finalmente usarlo para extraer datos de forma confiable de más formularios en función de las propias necesidades.

> [!NOTE]
> También puede entrenar modelos con una interfaz gráfica de usuario, como la [herramienta de etiquetado de ejemplo de Form Recognizer](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Entrenamiento de un modelo sin etiquetas

Entrene modelos personalizados para reconocer todos los campos y valores que se encuentran en los formularios personalizados sin etiquetar manualmente los documentos de entrenamiento.

La función siguiente entrena un modelo en un conjunto de documentos determinado e imprime el estado del modelo en la consola. 

```javascript
async function trainModel() {

    const containerSasUrl = "<SAS-URL-of-your-form-folder-in-blob-storage>";
    const trainingClient = new FormTrainingClient(endpoint, new AzureKeyCredential(apiKey));

    const poller = await trainingClient.beginTraining(containerSasUrl, false, {
        onProgress: (state) => { console.log(`training status: ${state.status}`); }
    });
    const model = await poller.pollUntilDone();

    if (!model) {
        throw new Error("Expecting valid training result!");
    }

    console.log(`Model ID: ${model.modelId}`);
    console.log(`Status: ${model.status}`);
    console.log(`Training started on: ${model.trainingStartedOn}`);
    console.log(`Training completed on: ${model.trainingCompletedOn}`);

    if (model.submodels) {
        for (const submodel of model.submodels) {
            // since the training data is unlabeled, we are unable to return the accuracy of this model
            console.log("We have recognized the following fields");
            for (const key in submodel.fields) {
                const field = submodel.fields[key];
                console.log(`The model found field '${field.name}'`);
            }
        }
    }
    // Training document information
    if (model.trainingDocuments) {
        for (const doc of model.trainingDocuments) {
            console.log(`Document name: ${doc.name}`);
            console.log(`Document status: ${doc.status}`);
            console.log(`Document page count: ${doc.pageCount}`);
            console.log(`Document errors: ${doc.errors}`);
        }
    }
}

trainModel().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

### <a name="output"></a>Resultados

Esta es la salida de un modelo entrenado con los datos de entrenamiento disponibles en el [SDK de Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training). Esta salida de ejemplo se ha truncado para facilitar la lectura.

```console
training status: creating
training status: ready
Model ID: 9d893595-1690-4cf2-a4b1-fbac0fb11909
Status: ready
Training started on: Thu Aug 20 2020 20:27:26 GMT-0700 (Pacific Daylight Time)
Training completed on: Thu Aug 20 2020 20:27:37 GMT-0700 (Pacific Daylight Time)
We have recognized the following fields
The model found field 'field-0'
The model found field 'field-1'
The model found field 'field-2'
The model found field 'field-3'
The model found field 'field-4'
The model found field 'field-5'
The model found field 'field-6'
The model found field 'field-7'
...
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: 
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: 
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: 
...
```

### <a name="train-a-model-with-labels"></a>Entrenamiento de un modelo con etiquetas

También puede entrenar modelos personalizados mediante el etiquetado manual de los documentos de entrenamiento. En algunos escenarios, el entrenamiento con etiquetas conduce a un mejor rendimiento. Para realizar el entrenamiento con etiquetas, debe disponer de archivos de información con etiquetas especiales (`\<filename\>.pdf.labels.json`) en el contenedor de almacenamiento de blobs junto con los documentos para el entrenamiento. La [herramienta de etiquetado de ejemplo de Form Recognizer](../../quickstarts/label-tool.md) proporciona una interfaz de usuario para ayudarle a crear estos archivos de etiqueta. Cuando los tenga, puede llamar al método `beginTraining` con el parámetro `uselabels` establecido en `true`.

```javascript
async function trainModelLabels() {

    const containerSasUrl = "<SAS-URL-of-your-form-folder-in-blob-storage>";
    const trainingClient = new FormTrainingClient(endpoint, new AzureKeyCredential(apiKey));

    const poller = await trainingClient.beginTraining(containerSasUrl, true, {
        onProgress: (state) => { console.log(`training status: ${state.status}`); }
    });
    const model = await poller.pollUntilDone();

    if (!model) {
        throw new Error("Expecting valid training result!");
    }

    console.log(`Model ID: ${model.modelId}`);
    console.log(`Status: ${model.status}`);
    console.log(`Training started on: ${model.trainingStartedOn}`);
    console.log(`Training completed on: ${model.trainingCompletedOn}`);

    if (model.submodels) {
        for (const submodel of model.submodels) {
            // since the training data is unlabeled, we are unable to return the accuracy of this model
            console.log("We have recognized the following fields");
            for (const key in submodel.fields) {
                const field = submodel.fields[key];
                console.log(`The model found field '${field.name}'`);
            }
        }
    }
    // Training document information
    if (model.trainingDocuments) {
        for (const doc of model.trainingDocuments) {
            console.log(`Document name: ${doc.name}`);
            console.log(`Document status: ${doc.status}`);
            console.log(`Document page count: ${doc.pageCount}`);
            console.log(`Document errors: ${doc.errors}`);
        }
    }
}

trainModelLabels().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

### <a name="output"></a>Output 

Esta es la salida de un modelo entrenado con los datos de entrenamiento disponibles en el [SDK de Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training). Esta salida de ejemplo se ha truncado para facilitar la lectura.

```console
training status: creating
training status: ready
Model ID: 789b1b37-4cc3-4e36-8665-9dde68618072
Status: ready
Training started on: Thu Aug 20 2020 20:30:37 GMT-0700 (Pacific Daylight Time)
Training completed on: Thu Aug 20 2020 20:30:43 GMT-0700 (Pacific Daylight Time)
We have recognized the following fields
The model found field 'CompanyAddress'
The model found field 'CompanyName'
The model found field 'CompanyPhoneNumber'
The model found field 'DatedAs'
...
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: undefined
...
```

## <a name="analyze-forms-with-a-custom-model"></a>Analizar formularios con un modelo personalizado

En esta sección, se muestra cómo extraer información de clave-valor y otro contenido de los tipos de formulario personalizados, con los modelos que ha entrenado con sus propios formularios.

> [!IMPORTANT]
> Para implementar este escenario, debe haber entrenado ya un modelo para que pueda pasar su identificador al método siguiente. Consulte la sección [Entrenamiento de un modelo](#train-a-model-without-labels).

Usará el método `beginRecognizeCustomFormsFromUrl`. El valor devuelto es una colección de objetos `RecognizedForm`, uno para cada página del documento enviado.

```javascript
async function recognizeCustom() {
    // Model ID from when you trained your model.
    const modelId = "<modelId>";
    const formUrl = "https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/tests/sample_forms/forms/Invoice_1.pdf";

    const client = new FormRecognizerClient(endpoint, new AzureKeyCredential(apiKey));
    const poller = await client.beginRecognizeCustomForms(modelId, formUrl, {
        onProgress: (state) => { console.log(`status: ${state.status}`); }
    });
    const forms = await poller.pollUntilDone();

    console.log("Forms:");
    for (const form of forms || []) {
        console.log(`${form.formType}, page range: ${form.pageRange}`);
        console.log("Pages:");
        for (const page of form.pages || []) {
            console.log(`Page number: ${page.pageNumber}`);
            console.log("Tables");
            for (const table of page.tables || []) {
                for (const cell of table.cells) {
                    console.log(`cell (${cell.rowIndex},${cell.columnIndex}) ${cell.text}`);
                }
            }
        }

        console.log("Fields:");
        for (const fieldName in form.fields) {
            // each field is of type FormField
            const field = form.fields[fieldName];
            console.log(
                `Field ${fieldName} has value '${field.value}' with a confidence score of ${field.confidence}`
            );
        }
    }
}

recognizeCustom().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

### <a name="output"></a>Resultados

```console
status: notStarted
status: succeeded
Forms:
custom:form, page range: [object Object]
Pages:
Page number: 1
Tables
cell (0,0) Invoice Number
cell (0,1) Invoice Date
cell (0,2) Invoice Due Date
cell (0,3) Charges
cell (0,5) VAT ID
cell (1,0) 34278587
cell (1,1) 6/18/2017
cell (1,2) 6/24/2017
cell (1,3) $56,651.49
cell (1,5) PT
Fields:
Field Merchant has value 'Invoice For:' with a confidence score of 0.116
Field CompanyPhoneNumber has value '$56,651.49' with a confidence score of 0.249
Field VendorName has value 'Charges' with a confidence score of 0.145
Field CompanyAddress has value '1 Redmond way Suite 6000 Redmond, WA' with a confidence score of 0.258
Field CompanyName has value 'PT' with a confidence score of 0.245
Field Website has value '99243' with a confidence score of 0.114
Field DatedAs has value 'undefined' with a confidence score of undefined
Field Email has value 'undefined' with a confidence score of undefined
Field PhoneNumber has value 'undefined' with a confidence score of undefined
Field PurchaseOrderNumber has value 'undefined' with a confidence score of undefined
Field Quantity has value 'undefined' with a confidence score of undefined
Field Signature has value 'undefined' with a confidence score of undefined
Field Subtotal has value 'undefined' with a confidence score of undefined
Field Tax has value 'undefined' with a confidence score of undefined
Field Total has value 'undefined' with a confidence score of undefined
```

## <a name="manage-your-custom-models"></a>Administración de modelos personalizados

En esta sección se muestra cómo administrar los modelos personalizados almacenados en su cuenta. Como ejemplo, el código siguiente realiza todas las tareas de administración del modelo en una única función. 

### <a name="get-list-of-models-in-account"></a>Obtención de una lista de modelos en la cuenta

El siguiente bloque de código proporciona una lista completa de los modelos disponibles en su cuenta, incluida información sobre cuándo se creó el modelo y su estado actual.

```javascript
async function listModels() {
    const client = new FormTrainingClient(endpoint, new AzureKeyCredential(apiKey));

    // returns an async iteratable iterator that supports paging
    const result = client.listCustomModels();
    let i = 0;
    for await (const modelInfo of result) {
        console.log(`model ${i++}:`);
        console.log(modelInfo);
    }
}

listModels().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

### <a name="output"></a>Resultados

```console
model 0:
{
  modelId: '453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e',
  status: 'invalid',
  trainingStartedOn: 2020-08-20T22:28:52.000Z,
  trainingCompletedOn: 2020-08-20T22:28:53.000Z
}
model 1:
{
  modelId: '628739de-779c-473d-8214-d35c72d3d4f7',
  status: 'ready',
  trainingStartedOn: 2020-08-20T23:16:51.000Z,
  trainingCompletedOn: 2020-08-20T23:16:59.000Z
}
model 2:
{
  modelId: '789b1b37-4cc3-4e36-8665-9dde68618072',
  status: 'ready',
  trainingStartedOn: 2020-08-21T03:30:37.000Z,
  trainingCompletedOn: 2020-08-21T03:30:43.000Z
}
model 3:
{
  modelId: '9d893595-1690-4cf2-a4b1-fbac0fb11909',
  status: 'ready',
  trainingStartedOn: 2020-08-21T03:27:26.000Z,
  trainingCompletedOn: 2020-08-21T03:27:37.000Z
}
```

### <a name="get-list-of-model-ids"></a>Obtención de una lista de identificadores de modelo

Este bloque de código proporciona una lista de modelos e identificadores de modelo.

```javascript
async function listModelIds(){
    const client = new FormTrainingClient(endpoint, new AzureKeyCredential(apiKey));
    // using `iter.next()`
    i = 1;
    let iter = client.listCustomModels();
    let modelItem = await iter.next();
    while (!modelItem.done) {
        console.log(`model ${i++}: ${modelItem.value.modelId}`);
        modelItem = await iter.next();
    }
}
```

### <a name="output"></a>Resultados

```console
model 1: 453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e
model 2: 628739de-779c-473d-8214-d35c72d3d4f7
model 3: 789b1b37-4cc3-4e36-8665-9dde68618072
```

### <a name="get-list-of-model-ids-by-page"></a>Obtención de una lista de identificadores de modelo por página

Este bloque de código proporciona una lista paginada de modelos e identificadores de modelo.

```javascript
async function listModelsByPage(){
    const client = new FormTrainingClient(endpoint, new AzureKeyCredential(apiKey));
    // using `byPage()`
    i = 1;
    for await (const response of client.listCustomModels().byPage()) {
        for (const modelInfo of response.modelList) {
            console.log(`model ${i++}: ${modelInfo.modelId}`);
        }
    }
}

listModelsByPage().catch((err) => {
    console.error("The sample encountered an error:", err);
});
```

### <a name="output"></a>Resultados

```console
model 1: 453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e
model 2: 628739de-779c-473d-8214-d35c72d3d4f7
model 3: 789b1b37-4cc3-4e36-8665-9dde68618072
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

### <a name="output"></a>Output

```console
Model with id 789b1b37-4cc3-4e36-8665-9dde68618072 has been deleted
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Puede ejecutar la aplicación en cualquier momento con cualquier número de funciones que haya leído en esta guía de inicio rápido con este comando:

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

## <a name="see-also"></a>Consulte también

* [¿Qué es Form Recognizer?](../../overview.md)
