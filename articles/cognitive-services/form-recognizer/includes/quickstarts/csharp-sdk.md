---
title: 'Inicio rápido: Biblioteca cliente de Form Recognizer para .NET'
description: En este inicio rápido, empezará a trabajar con la biblioteca cliente de Form Recognizer para .NET.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 05/06/2020
ms.author: pafarley
ms.openlocfilehash: 8ff353c5386f7ad5f30144ca50740c751b81ffc5
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87910671"
---
[Documentación de referencia](https://docs.microsoft.com/dotnet/api/overview/azure/ai.formrecognizer-readme-pre) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/src) | [Paquete (NuGet)](https://www.nuget.org/packages/Azure.AI.FormRecognizer) | [Ejemplos](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/).
* Un blob de Azure Storage que contenga un conjunto de datos de entrenamiento. Consulte [Creación de un conjunto de datos de aprendizaje para un modelo personalizado](../../build-training-data-set.md) para ver sugerencias y opciones para reunir el conjunto de datos de aprendizaje. En este inicio rápido puede usar los archivos de la carpeta **Entrenar** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451).
* La versión actual de [.NET Core](https://dotnet.microsoft.com/download/dotnet-core).

## <a name="setting-up"></a>Instalación

### <a name="create-a-form-recognizer-azure-resource"></a>Creación de un recurso de Azure de Form Recognizer

[!INCLUDE [create resource](../create-resource.md)]

### <a name="create-environment-variables"></a>Creación de variables de entorno

[!INCLUDE [environment-variables](../environment-variables.md)]

### <a name="create-a-new-c-application"></a>Creación de una aplicación de C#

En una ventana de consola (por ejemplo, cmd, PowerShell o Bash), use el comando `dotnet new` para crear una nueva aplicación de consola con el nombre `formrecognizer-quickstart`. Este comando crea un sencillo proyecto "Hola mundo" de C# con un solo archivo de origen: _Program.cs_. 

```console
dotnet new console -n formrecognizer-quickstart
```

Cambie el directorio a la carpeta de aplicaciones recién creada. A continuación, compile la aplicación con:

```console
dotnet build
```

La salida de la compilación no debe contener advertencias ni errores. 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

En el directorio del proyecto, abra el archivo *Program.cs* en el editor o IDE que prefiera. Agregue las siguientes directivas `using`:

```csharp
using Azure.AI.FormRecognizer;
using Azure.AI.FormRecognizer.Models;
using Azure.AI.FormRecognizer.Training;

using System;
using System.IO;
using System.Threading.Tasks;
```

Después, agregue el código siguiente en el método **Main** de la aplicación. Definirá esta tarea asincrónica más adelante. 

```csharp
static void Main(string[] args)
{
    var t1 = RunFormRecognizerClient();

    Task.WaitAll(t1);
}
```

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Dentro del directorio de aplicaciones, instale la biblioteca cliente de Form Recognizer para .NET con el siguiente comando:

```console
dotnet add package Azure.AI.FormRecognizer --version 1.0.0-preview.3
```

Si usa el IDE de Visual Studio, la biblioteca cliente estará disponible como un paquete de NuGet descargable.


<!-- Objet model TBD -->



## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas con la biblioteca cliente de Form Recognizer para .NET:

* [Autenticar el cliente](#authenticate-the-client)
* [Reconocer el contenido del formulario](#recognize-form-content)
* [Reconocer recibos](#recognize-receipts)
* [Entrenar un modelo personalizado](#train-a-custom-model)
* [Analizar formularios con un modelo personalizado](#analyze-forms-with-a-custom-model)
* [Administrar modelos personalizados](#manage-your-custom-models)


## <a name="authenticate-the-client"></a>Autenticar el cliente

Debajo del método `Main`, defina la tarea a la que se hace referencia en `Main`. Aquí, autenticará dos objetos de cliente mediante las variables de suscripción que definió anteriormente. Usará un objeto **AzureKeyCredential**, de modo que, si es necesario, pueda actualizar la clave de API sin crear otros objetos de cliente.

```csharp
static async Task RunFormRecognizerClient()
{ 
    string endpoint = Environment.GetEnvironmentVariable(
        "FORM_RECOGNIZER_ENDPOINT");
    string apiKey = Environment.GetEnvironmentVariable(
        "FORM_RECOGNIZER_KEY");
    var credential = new AzureKeyCredential(apiKey);
    
    var trainingClient = new FormTrainingClient(new Uri(endpoint), credential);
    var recognizerClient = new FormRecognizerClient(new Uri(endpoint), credential);
```

### <a name="call-client-specific-methods"></a>Llamada a los métodos específicos del cliente

El siguiente bloque de código usa los objetos cliente para llamar a los métodos de cada una de las tareas principales del SDK de Form Recognizer. Más adelante, definirá estos métodos.

También tendrá que agregar referencias a las direcciones URL de los datos de entrenamiento y prueba. 
* Para recuperar la dirección URL de SAS de los datos de entrenamiento del modelo personalizado, abra el Explorador de Microsoft Azure Storage, haga clic con el botón derecho en el contenedor y seleccione **Obtener firma de acceso compartido**. Asegúrese de que los permisos de **lectura** y **enumeración** están marcados y haga clic en **Create** (Crear). A continuación, copie el valor en la sección **URL**. Debe tener el formato `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.
* Para obtener una dirección URL de un formulario para prueba, puede usar los pasos anteriores para obtener la dirección URL de SAS de un documento del almacenamiento de blobs. También puede tomar la dirección URL de un documento situado en otro lugar.
* Use el método anterior para obtener también la dirección URL de una imagen de recibo, o utilice la dirección URL de la imagen de ejemplo proporcionada.

> [!NOTE]
> Los fragmentos de código de esta guía usan formularios remotos con acceso mediante direcciones URL. En cambio, si quiere procesar documentos de formulario locales, consulte los métodos relacionados en la [documentación de referencia](https://docs.microsoft.com/dotnet/api/overview/azure/ai.formrecognizer-readme-pre).

```csharp
    string trainingDataUrl = "<SAS-URL-of-your-form-folder-in-blob-storage>";
    string formUrl = "<SAS-URL-of-a-form-in-blob-storage>";
    string receiptUrl = "https://docs.microsoft.com/azure/cognitive-services/form-recognizer/media"
    + "/contoso-allinone.jpg";

    // Call Form Recognizer scenarios:
    Console.WriteLine("Get form content...");
    await GetContent(recognizerClient, formUrl);

    Console.WriteLine("Analyze receipt...");
    await AnalyzeReceipt(recognizerClient, receiptUrl);

    Console.WriteLine("Train Model with training data...");
    Guid modelId = await TrainModel(trainingClient, trainingDataUrl);

    Console.WriteLine("Analyze PDF form...");
    await AnalyzePdfForm(recognizerClient, modelId, formUrl);

    Console.WriteLine("Manage models...");
    await ManageModels(trainingClient, trainingDataUrl) ;
}
```

## <a name="recognize-form-content"></a>Reconocimiento del contenido del formulario

Puede usar Form Recognizer para reconocer tablas, líneas y palabras de los documentos sin necesidad de entrenar un modelo.

Para reconocer el contenido de un archivo en un URI determinado, use el método **StartRecognizeContentFromUri**.

```csharp
private static async Task GetContent(
    FormRecognizerClient recognizerClient, string invoiceUri)
{
    Response<FormPageCollection> formPages = await recognizerClient
        .StartRecognizeContentFromUri(new Uri(invoiceUri))
        .WaitForCompletionAsync();
```

El valor devuelto es una colección de objetos **FormPage**, uno para cada página del documento enviado. El código siguiente recorre en iteración estos objetos e imprime los pares clave-valor y los datos de tabla extraídos.

```csharp
    foreach (FormPage page in formPages.Value)
    {
        Console.WriteLine($"Form Page {page.PageNumber} has {page.Lines.Count}" + 
            $" lines.");
    
        for (int i = 0; i < page.Lines.Count; i++)
        {
            FormLine line = page.Lines[i];
            Console.WriteLine($"    Line {i} has {line.Words.Count}" + 
                $" word{(line.Words.Count > 1 ? "s" : "")}," +
                $" and text: '{line.Text}'.");
        }
    
        for (int i = 0; i < page.Tables.Count; i++)
        {
            FormTable table = page.Tables[i];
            Console.WriteLine($"Table {i} has {table.RowCount} rows and" +
                $" {table.ColumnCount} columns.");
            foreach (FormTableCell cell in table.Cells)
            {
                Console.WriteLine($"    Cell ({cell.RowIndex}, {cell.ColumnIndex})" +
                    $" contains text: '{cell.Text}'.");
            }
        }
    }
}
```

## <a name="recognize-receipts"></a>Reconocimiento de recibos

En esta sección se muestra cómo reconocer y extraer campos comunes de recibos de EE. UU. mediante un modelo de recibos entrenado previamente.

Para reconocer los recibos a partir de un URI, use el método **StartRecognizeReceiptsFromUri**. El valor devuelto es una colección de objetos **RecognizedReceipt**, uno para cada página del documento enviado. En el código siguiente se procesa un recibo en el URI especificado y se imprimen los campos y valores principales en la consola.

```csharp
private static async Task AnalyzeReceipt(
    FormRecognizerClient recognizerClient, string receiptUri)
{
    RecognizedReceiptCollection receipts = await recognizerClient.StartRecognizeReceiptsFromUri(new Uri(receiptUri))
    .WaitForCompletionAsync();

    foreach (RecognizedReceipt receipt in receipts)
    {
    FormField merchantNameField;
    if (receipt.RecognizedForm.Fields.TryGetValue("MerchantName", out merchantNameField))
    {
        if (merchantNameField.Value.Type == FieldValueType.String)
        {
            string merchantName = merchantNameField.Value.AsString();

            Console.WriteLine($"Merchant Name: '{merchantName}', with confidence {merchantNameField.Confidence}");
        }
    }

    FormField transactionDateField;
    if (receipt.RecognizedForm.Fields.TryGetValue("TransactionDate", out transactionDateField))
    {
        if (transactionDateField.Value.Type == FieldValueType.Date)
        {
            DateTime transactionDate = transactionDateField.Value.AsDate();

            Console.WriteLine($"Transaction Date: '{transactionDate}', with confidence {transactionDateField.Confidence}");
        }
    }
```

El siguiente bloque de código recorre en iteración los elementos detectados en el recibo e imprime los detalles en la consola.

```csharp
    FormField itemsField;
    if (receipt.RecognizedForm.Fields.TryGetValue("Items", out itemsField))
    {
        if (itemsField.Value.Type == FieldValueType.List)
        {
            foreach (FormField itemField in itemsField.Value.AsList())
            {
                Console.WriteLine("Item:");

                if (itemField.Value.Type == FieldValueType.Dictionary)
                {
                    IReadOnlyDictionary<string, FormField> itemFields = itemField.Value.AsDictionary();

                    FormField itemNameField;
                    if (itemFields.TryGetValue("Name", out itemNameField))
                    {
                        if (itemNameField.Value.Type == FieldValueType.String)
                        {
                            string itemName = itemNameField.Value.AsString();

                            Console.WriteLine($"    Name: '{itemName}', with confidence {itemNameField.Confidence}");
                        }
                    }

                    FormField itemTotalPriceField;
                    if (itemFields.TryGetValue("TotalPrice", out itemTotalPriceField))
                    {
                        if (itemTotalPriceField.Value.Type == FieldValueType.Float)
                        {
                            float itemTotalPrice = itemTotalPriceField.Value.AsFloat();

                            Console.WriteLine($"    Total Price: '{itemTotalPrice}', with confidence {itemTotalPriceField.Confidence}");
                        }
                    }
                }
            }
        }
    }
```

Por último, el último bloque de código imprime el valor total del recibo.

```csharp
    FormField totalField;
    if (receipt.RecognizedForm.Fields.TryGetValue("Total", out totalField))
    {
        if (totalField.Value.Type == FieldValueType.Float)
        {
            float total = totalField.Value.AsFloat();

            Console.WriteLine($"Total: '{total}', with confidence '{totalField.Confidence}'");
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

El método siguiente entrena un modelo en un conjunto de documentos dado e imprime el estado del modelo en la consola. 

```csharp
private static async Task<Guid> TrainModel(
    FormRecognizerClient trainingClient, string trainingDataUrl)
{
    CustomFormModel model = await trainingClient
        .StartTrainingAsync(new Uri(trainingFileUrl), useTrainingLabels: false).WaitForCompletionAsync();
    
    Console.WriteLine($"Custom Model Info:");
    Console.WriteLine($"    Model Id: {model.ModelId}");
    Console.WriteLine($"    Model Status: {model.Status}");
    Console.WriteLine($"    Requested on: {model.RequestedOn}");
    Console.WriteLine($"    Completed on: {model.CompletedOn}");
```

El objeto devuelto **CustomFormModel** contiene información sobre los tipos de formulario que el modelo puede reconocer y los campos que puede extraer de cada tipo de formulario. El siguiente bloque de código imprime esta información en la consola.

```csharp
    foreach (CustomFormSubmodel submodel in model.Submodels)
    {
        Console.WriteLine($"Submodel Form Type: {submodel.FormType}");
        foreach (CustomFormModelField field in submodel.Fields.Values)
        {
            Console.Write($"    FieldName: {field.Name}");
            if (field.Label != null)
            {
                Console.Write($", FieldLabel: {field.Label}");
            }
            Console.WriteLine("");
        }
    }
```

Por último, este método devuelve el identificador único del modelo.

```csharp
    return model.ModelId;
}
```

### <a name="train-a-model-with-labels"></a>Entrenamiento de un modelo con etiquetas

También puede entrenar modelos personalizados mediante el etiquetado manual de los documentos de entrenamiento. En algunos escenarios, el entrenamiento con etiquetas conduce a un mejor rendimiento. Para realizar el entrenamiento con etiquetas, debe disponer de archivos de información de etiquetas especiales ( *\<filename\>.pdf.labels.json*) en el contenedor de almacenamiento de blobs junto con los documentos de entrenamiento. La [herramienta de etiquetado de ejemplo de Form Recognizer](../../quickstarts/label-tool.md) proporciona una interfaz de usuario para ayudarle a crear estos archivos de etiqueta. Cuando los tenga, puede llamar al método **StartTrainingAsync** con el parámetro *uselabels* establecido en `true`.

```csharp
private static async Task<Guid> TrainModelWithLabelsAsync(
    FormRecognizerClient trainingClient, string trainingDataUrl)
{
    CustomFormModel model = await trainingClient
    .StartTrainingAsync(new Uri(trainingFileUrl), useTrainingLabels: true).WaitForCompletionAsync();
    
    Console.WriteLine($"Custom Model Info:");
    Console.WriteLine($"    Model Id: {model.ModelId}");
    Console.WriteLine($"    Model Status: {model.Status}");
    Console.WriteLine($"    Requested on: {model.RequestedOn}");
    Console.WriteLine($"    Completed on: {model.CompletedOn}");
```

El valor de **CustomFormModel** devuelto indica los campos que el modelo puede extraer, junto con su precisión estimada en cada campo. El siguiente bloque de código imprime esta información en la consola.

```csharp
    foreach (CustomFormSubmodel submodel in model.Submodels)
    {
        Console.WriteLine($"Submodel Form Type: {submodel.FormType}");
        foreach (CustomFormModelField field in submodel.Fields.Values)
        {
            Console.Write($"    FieldName: {field.Name}");
            if (field.Accuracy != null)
            {
                Console.Write($", Accuracy: {field.Accuracy}");
            }
            Console.WriteLine("");
        }
    }
    return model.ModelId;
}
```

## <a name="analyze-forms-with-a-custom-model"></a>Analizar formularios con un modelo personalizado

En esta sección, se muestra cómo extraer información de clave-valor y otro contenido de los tipos de formulario personalizados, con los modelos que ha entrenado con sus propios formularios.

> [!IMPORTANT]
> Para implementar este escenario, debe haber entrenado ya un modelo para que pueda pasar su identificador al método siguiente. Consulte la sección [Entrenamiento de un modelo](#train-a-model-without-labels).

Usará el método **StartRecognizeCustomFormsFromUri**. El valor devuelto es una colección de objetos **RecognizedForm**: uno para cada página del documento enviado.

```csharp
// Analyze PDF form data
private static async Task AnalyzePdfForm(
    FormRecognizerClient recognizerClient, Guid modelId, string formUrl)
{    
    Response<IReadOnlyList<RecognizedForm>> forms = await recognizerClient
        .StartRecognizeCustomFormsFromUri(modelId.ToString(), new Uri(formUrl))
        .WaitForCompletionAsync();
```

El código siguiente imprime los resultados del análisis en la consola. Imprime cada campo reconocido y el valor correspondiente, junto con una puntuación de confianza.

```csharp
    foreach (RecognizedForm form in forms.Value)
    {
        Console.WriteLine($"Form of type: {form.FormType}");
        foreach (FormField field in form.Fields.Values)
        {
            Console.WriteLine($"Field '{field.Name}: ");
    
            if (field.LabelText != null)
            {
                Console.WriteLine($"    Label: '{field.LabelText.Text}");
            }
    
            Console.WriteLine($"    Value: '{field.ValueText.Text}");
            Console.WriteLine($"    Confidence: '{field.Confidence}");
        }
    }
}
```

## <a name="manage-your-custom-models"></a>Administración de modelos personalizados

En esta sección se muestra cómo administrar los modelos personalizados almacenados en su cuenta. Como ejemplo, en el código siguiente se realizan todas las tareas de administración del modelo en un único método. Comience por copiar la firma del método siguiente:

```csharp
private static async Task ManageModels(
    FormRecognizerClient trainingClient, string trainingFileUrl)
{
```

### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Comprobación del número de modelos de la cuenta de recursos de FormRecognizer

El siguiente bloque de código comprueba cuántos modelos se han guardado en la cuenta de Form Recognizer y compara esta cifra con el límite de la cuenta.

```csharp
    // Check number of models in the FormRecognizer account, 
    // and the maximum number of models that can be stored.
    AccountProperties accountProperties = trainingClient.GetAccountProperties();
    Console.WriteLine($"Account has {accountProperties.CustomModelCount} models.");
    Console.WriteLine($"It can have at most {accountProperties.CustomModelLimit}" +
        $" models.");
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Enumeración de los modelos almacenados actualmente en la cuenta de recursos

El siguiente bloque de código enumera los modelos actuales de la cuenta e imprime los detalles en la consola.

```csharp
    // List the first ten or fewer models currently stored in the account.
    Pageable<CustomFormModelInfo> models = trainingClient.GetModelInfos();
    
    foreach (CustomFormModelInfo modelInfo in models.Take(10))
    {
        Console.WriteLine($"Custom Model Info:");
        Console.WriteLine($"    Model Id: {modelInfo.ModelId}");
        Console.WriteLine($"    Model Status: {modelInfo.Status}");
        Console.WriteLine($"    Created On: {modelInfo.CreatedOn}");
        Console.WriteLine($"    Last Modified: {modelInfo.LastModified}");
    }
```

### <a name="get-a-specific-model-using-the-models-id"></a>Obtención de un modelo específico mediante el identificador del modelo

El siguiente bloque de código entrena un nuevo modelo (al igual que en la sección [Entrenamiento de un modelo](#train-a-model-without-labels)) y, luego, recupera una segunda referencia a él mediante su identificador.

```csharp
    // Create a new model to store in the account
    CustomFormModel model = await trainingClient.StartTrainingAsync(
        new Uri(trainingFileUrl)).WaitForCompletionAsync();
    
    // Get the model that was just created
    CustomFormModel modelCopy = trainingClient.GetCustomModel(model.ModelId);
    
    Console.WriteLine($"Custom Model {modelCopy.ModelId} recognizes the following" +
        " form types:");
    
    foreach (CustomFormSubModel subModel in modelCopy.Models)
    {
        Console.WriteLine($"SubModel Form Type: {subModel.FormType}");
        foreach (CustomFormModelField field in subModel.Fields.Values)
        {
            Console.Write($"    FieldName: {field.Name}");
            if (field.Label != null)
            {
                Console.Write($", FieldLabel: {field.Label}");
            }
            Console.WriteLine("");
        }
    }
```

### <a name="delete-a-model-from-the-resource-account"></a>Eliminación de un modelo de la cuenta de recursos

También puede eliminar un modelo de su cuenta haciendo referencia a su identificador.

```csharp
    // Delete the model from the account.
    trainingClient.DeleteModel(model.ModelId);
}
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación desde el directorio de la aplicación con el comando `dotnet run`.

```dotnet
dotnet run
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Solución de problemas

Al interactuar con la biblioteca cliente de Form Recognizer de Cognitive Services mediante el SDK de .NET, los errores devueltos por el servicio producirán una excepción `RequestFailedException`. Incluirán el mismo código de estado HTTP que se habría devuelto con una solicitud de API REST.

Por ejemplo, si envía la imagen de un recibo con un URI no válido, se devuelve un error `400`, que indica "solicitud incorrecta".

```csharp Snippet:FormRecognizerBadRequest
try
{
    RecognizedReceiptCollection receipts = await client.StartRecognizeReceiptsFromUri(new Uri(receiptUri)).WaitForCompletionAsync();

}
catch (RequestFailedException e)
{
    Console.WriteLine(e.ToString());
}
```

Observará que se registra información adicional, como el identificador de la solicitud del cliente de la operación.

```
Message:
    Azure.RequestFailedException: Service request failed.
    Status: 400 (Bad Request)

Content:
    {"error":{"code":"FailedToDownloadImage","innerError":
    {"requestId":"8ca04feb-86db-4552-857c-fde903251518"},
    "message":"Failed to download image from input URL."}}

Headers:
    Transfer-Encoding: chunked
    x-envoy-upstream-service-time: REDACTED
    apim-request-id: REDACTED
    Strict-Transport-Security: REDACTED
    X-Content-Type-Options: REDACTED
    Date: Mon, 20 Apr 2020 22:48:35 GMT
    Content-Type: application/json; charset=utf-8
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se ha usado la biblioteca cliente de .NET para Form Recognizer para entrenar modelos y analizar formularios de maneras diferentes. A continuación, obtenga sugerencias para crear un mejor conjunto de datos de entrenamiento y generar modelos más precisos.

> [!div class="nextstepaction"]
> [Creación de un conjunto de datos de aprendizaje](../../build-training-data-set.md)

* [¿Qué es Form Recognizer?](../../overview.md)
* El código de ejemplo de esta guía (y más) se puede encontrar en [GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/formrecognizer/Azure.AI.FormRecognizer/samples/README.md).
