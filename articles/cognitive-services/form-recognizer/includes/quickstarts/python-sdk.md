---
title: 'Inicio rápido: Biblioteca cliente de Form Recognizer para Python'
description: En este inicio rápido, empezará a trabajar con la biblioteca cliente de Form Recognizer para Python.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: include
ms.date: 08/21/2020
ms.author: pafarley
ms.openlocfilehash: b178a0b347888f22d9a3c0ee88a203e377cb15be
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88864655"
---
> [!IMPORTANT]
> * Actualmente, el SDK de Form Recognizer tiene como destino la versión v2.0 del servicio Form Recognizer.
> * Por motivos de simplicidad, en el código de este artículo se usan métodos sincrónicos y almacenamiento de credenciales no protegidas. Para más información, consulte la siguiente documentación de referencia. 

[Documentación de referencia](https://docs.microsoft.com/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/azure/ai/formrecognizer) | [Paquete (PyPi)](https://pypi.org/project/azure-ai-formrecognizer/) | [Ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* Un blob de Azure Storage que contenga un conjunto de datos de entrenamiento. Consulte [Creación de un conjunto de datos de aprendizaje para un modelo personalizado](../../build-training-data-set.md) para ver sugerencias y opciones para reunir el conjunto de datos de aprendizaje. En este inicio rápido puede usar los archivos de la carpeta **Entrenar** del [conjunto de datos de ejemplo](https://go.microsoft.com/fwlink/?linkid=2090451).
* [Python 2.7, 3.5 o superior](https://www.python.org/)
* Una vez que tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer"  title="cree un recurso de Form Recognizer"  target="_blank">create a Form Recognizer resource <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Form Recognizer API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="object-model"></a>Modelo de objetos 

Con Form Recognizer, puede crear dos tipos de cliente diferentes. El primero, `form_recognizer_client`, se utiliza para consultar el servicio con los campos de formulario y el contenido reconocidos. El segundo, `form_training_client`, se usa para crear y administrar los modelos personalizados que puede usar para mejorar el reconocimiento. 

### <a name="formrecognizerclient"></a>FormRecognizerClient
`form_recognizer_client` proporciona operaciones para:

 * El reconocimiento de los campos de formulario y el contenido mediante modelos personalizados entrenados para reconocer los formularios personalizados. 
 * El reconocimiento del contenido de los formularios, incluidas tablas, líneas y palabras, sin necesidad de entrenar un modelo. 
 * El reconocimiento de los campos comunes de las recepciones mediante un modelo de recepción entrenado previamente en el servicio Form Recognizer.

### <a name="formtrainingclient"></a>FormTrainingClient
`form_training_client` proporciona operaciones para:

* El entrenamiento de modelos personalizados para reconocer todos los campos y los valores que se encuentren en los formularios personalizados. Consulte la [documentación del servicio sobre el entrenamiento de modelos sin etiquetas](#train-a-model-without-labels) si desea obtener una explicación más detallada de la creación de un conjunto de datos de entrenamiento.
* El entrenamiento de modelos personalizados para reconocer los campos y los valores concretos que especifique mediante la etiqueta de los formularios personalizados. Consulte la [documentación del servicio sobre el entrenamiento de modelos con etiqueta](#train-a-model-with-labels) para obtener una explicación más detallada de cómo aplicar etiquetas a un conjunto de datos de entrenamiento.
* La administración de los modelos creados en una cuenta.
* La copia de un modelo personalizado entre recursos de Form Recognizer.

Tenga en cuenta que los modelos también se pueden entrenar mediante una interfaz gráfica de usuario, como la [herramienta de etiquetado de Form Recognizer](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/quickstarts/label-tool).

## <a name="setting-up"></a>Instalación

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Después de instalar Python, puede instalar la versión más reciente de la biblioteca de cliente de Form Recognizer con:

```console
pip install azure-ai-formrecognizer
```

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Cree una aplicación de Python en el IDE o editor que prefiera. A continuación, importe las bibliotecas siguientes. Tenga en cuenta que vamos a importar las bibliotecas necesarias tanto para el entrenamiento como para el reconocimiento de formularios.

```python
import os
from azure.core.exceptions import ResourceNotFoundError
from azure.ai.formrecognizer import FormRecognizerClient
from azure.ai.formrecognizer import FormTrainingClient
from azure.core.credentials import AzureKeyCredential
```

Cree variables para el punto de conexión y la clave de Azure del recurso. 

```python
endpoint = "<paste-your-form-recognizer-endpoint-here>"
key = "<paste-your-form-recognizer-key-here>"
```

## <a name="code-examples"></a>Ejemplos de código

En estos fragmentos de código, se muestra cómo realizar las siguientes tareas con la biblioteca cliente de Form Recognizer para Python:

* [Autenticar el cliente](#authenticate-the-client)
* [Reconocer el contenido del formulario](#recognize-form-content)
* [Reconocer recibos](#recognize-receipts)
* [Entrenar un modelo personalizado](#train-a-custom-model)
* [Analizar formularios con un modelo personalizado](#analyze-forms-with-a-custom-model)
* [Administrar modelos personalizados](#manage-your-custom-models)


## <a name="authenticate-the-client"></a>Autenticar el cliente

Aquí, autenticará dos objetos de cliente mediante las variables de suscripción que definió anteriormente. Usará un objeto `AzureKeyCredential`, de modo que, si es necesario, pueda actualizar la clave de API sin crear otros objetos de cliente.

```python
form_recognizer_client = FormRecognizerClient(endpoint, AzureKeyCredential(key))
form_training_client = FormTrainingClient(endpoint, AzureKeyCredential(key))
```

## <a name="assets-for-testing"></a>Recursos para pruebas

Los fragmentos de código de esta guía usan formularios remotos con acceso mediante direcciones URL. En cambio, si quiere procesar documentos de formulario locales, consulte los métodos relacionados en la [documentación de referencia](https://docs.microsoft.com/python/api/azure-ai-formrecognizer/azure.ai.formrecognizer) y los [ejemplos](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples).

También tendrá que agregar referencias a las direcciones URL de los datos de entrenamiento y prueba.
* Para recuperar la dirección URL de SAS de los datos de entrenamiento del modelo personalizado, abra el Explorador de Microsoft Azure Storage, haga clic con el botón derecho en el contenedor y seleccione **Obtener firma de acceso compartido**. Asegúrese de que los permisos de **lectura** y **enumeración** están marcados y haga clic en **Create** (Crear). A continuación, copie el valor en la sección **URL**. Debe tener el formato `https://<storage account>.blob.core.windows.net/<container name>?<SAS value>`.
* Use las imágenes de envío y de recepción que se incluyen en los ejemplos siguientes (también disponibles en [GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms)). O bien, puede seguir los pasos anteriores para obtener la dirección URL de SAS de un documento individual en el almacenamiento de blobs. 

> [!NOTE]
> Los fragmentos de código de esta guía usan formularios remotos con acceso mediante direcciones URL. En cambio, si quiere procesar documentos de formulario locales, consulte los métodos relacionados en la [documentación de referencia](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/).

## <a name="recognize-form-content"></a>Reconocimiento del contenido del formulario

Puede usar Form Recognizer para reconocer tablas, líneas y palabras de los documentos sin necesidad de entrenar un modelo.

Para reconocer el contenido de un archivo en una dirección URL determinada, use el método `begin_recognize_content`. El valor devuelto es una colección de objetos `FormPage`, uno para cada página del documento enviado. El código siguiente recorre en iteración estos objetos e imprime los pares clave-valor y los datos de tabla extraídos.

```Python
formUrl = "https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/tests/sample_forms/forms/Invoice_1.pdf"

poller = form_recognizer_client.begin_recognize_content_from_url(formUrl)
page = poller.result()

table = page[0].tables[0] # page 1, table 1
print("Table found on page {}:".format(table.page_number))
for cell in table.cells:
    print("Cell text: {}".format(cell.text))
    print("Location: {}".format(cell.bounding_box))
    print("Confidence score: {}\n".format(cell.confidence))
```

### <a name="output"></a>Output

```console
Table found on page 1:
Cell text: Invoice Number
Location: [Point(x=0.5075, y=2.8088), Point(x=1.9061, y=2.8088), Point(x=1.9061, y=3.3219), Point(x=0.5075, y=3.3219)]
Confidence score: 1.0

Cell text: Invoice Date
Location: [Point(x=1.9061, y=2.8088), Point(x=3.3074, y=2.8088), Point(x=3.3074, y=3.3219), Point(x=1.9061, y=3.3219)]
Confidence score: 1.0

Cell text: Invoice Due Date
Location: [Point(x=3.3074, y=2.8088), Point(x=4.7074, y=2.8088), Point(x=4.7074, y=3.3219), Point(x=3.3074, y=3.3219)]
Confidence score: 1.0

Cell text: Charges
Location: [Point(x=4.7074, y=2.8088), Point(x=5.386, y=2.8088), Point(x=5.386, y=3.3219), Point(x=4.7074, y=3.3219)]
Confidence score: 1.0

...

```

## <a name="recognize-receipts"></a>Reconocimiento de recibos

En esta sección se muestra cómo reconocer y extraer campos comunes de recibos de EE. UU. mediante un modelo de recibos entrenado previamente. Para reconocer los recibos a partir de una dirección URL, use el método `begin_recognize_receipts_from_url`. 

```python
receiptUrl = "https://raw.githubusercontent.com/Azure/azure-sdk-for-python/master/sdk/formrecognizer/azure-ai-formrecognizer/tests/sample_forms/receipt/contoso-receipt.png"

poller = form_recognizer_client.begin_recognize_receipts_from_url(receiptUrl)
result = poller.result()

for receipt in result:
    for name, field in receipt.fields.items():
        if name == "Items":
            print("Receipt Items:")
            for idx, items in enumerate(field.value):
                print("...Item #{}".format(idx + 1))
                for item_name, item in items.value.items():
                    print("......{}: {} has confidence {}".format(item_name, item.value, item.confidence))
        else:
            print("{}: {} has confidence {}".format(name, field.value, field.confidence))
```

### <a name="output"></a>Resultados

```console
ReceiptType: Itemized has confidence 0.659
MerchantName: Contoso Contoso has confidence 0.516
MerchantAddress: 123 Main Street Redmond, WA 98052 has confidence 0.986
MerchantPhoneNumber: None has confidence 0.99
TransactionDate: 2019-06-10 has confidence 0.985
TransactionTime: 13:59:00 has confidence 0.968
Receipt Items:
...Item #1
......Name: 8GB RAM (Black) has confidence 0.916
......TotalPrice: 999.0 has confidence 0.559
...Item #2
......Quantity: None has confidence 0.858
......Name: SurfacePen has confidence 0.858
......TotalPrice: 99.99 has confidence 0.386
Subtotal: 1098.99 has confidence 0.964
Tax: 104.4 has confidence 0.713
Total: 1203.39 has confidence 0.774
```

## <a name="train-a-custom-model"></a>Entrenamiento de un modelo personalizado

En esta sección se muestra cómo entrenar un modelo con sus propios datos. Un modelo entrenado puede generar datos estructurados que incluyan las relaciones clave-valor del documento de formulario original. Después de entrenar el modelo, puede probarlo, volver a entrenarlo y finalmente usarlo para extraer datos de forma confiable de más formularios en función de las propias necesidades.

> [!NOTE]
> También puede entrenar modelos con una interfaz gráfica de usuario, como la [herramienta de etiquetado de ejemplo de Form Recognizer](../../quickstarts/label-tool.md).

### <a name="train-a-model-without-labels"></a>Entrenamiento de un modelo sin etiquetas

Entrene modelos personalizados para reconocer todos los campos y valores que se encuentran en los formularios personalizados sin etiquetar manualmente los documentos de entrenamiento.

El código siguiente usa el cliente de entrenamiento con la función `begin_training` para entrenar un modelo en un conjunto de documentos determinado. El objeto devuelto `CustomFormModel` contiene información sobre los tipos de formulario que el modelo puede reconocer y los campos que puede extraer de cada tipo de formulario. El siguiente bloque de código imprime esta información en la consola.

```python
# To train a model you need an Azure Storage account.
# Use the SAS URL to access your training files.
trainingDataUrl = "<SAS-URL-of-your-form-folder-in-blob-storage>"

poller = form_training_client.begin_training(trainingDataUrl, use_training_labels=False)
model = poller.result()

print("Model ID: {}".format(model.model_id))
print("Status: {}".format(model.status))
print("Training started on: {}".format(model.training_started_on))
print("Training completed on: {}".format(model.training_completed_on))

print("\nRecognized fields:")
for submodel in model.submodels:
    print(
        "The submodel with form type '{}' has recognized the following fields: {}".format(
            submodel.form_type,
            ", ".join(
                [
                    field.label if field.label else name
                    for name, field in submodel.fields.items()
                ]
            ),
        )
    )

# Training result information
for doc in model.training_documents:
    print("Document name: {}".format(doc.name))
    print("Document status: {}".format(doc.status))
    print("Document page count: {}".format(doc.page_count))
    print("Document errors: {}".format(doc.errors))
```

### <a name="output"></a>Resultados

Esta es la salida para un modelo entrenado con los datos de entrenamiento disponibles en el [SDK de Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training).

```console
Model ID: 628739de-779c-473d-8214-d35c72d3d4f7
Status: ready
Training started on: 2020-08-20 23:16:51+00:00
Training completed on: 2020-08-20 23:16:59+00:00

Recognized fields:
The submodel with form type 'form-0' has recognized the following fields: Additional Notes:, Address:, Company Name:, Company Phone:, Dated As:, Details, Email:, Hero Limited, Name:, Phone:, Purchase Order, Purchase Order #:, Quantity, SUBTOTAL, Seattle, WA 93849 Phone:, Shipped From, Shipped To, TAX, TOTAL, Total, Unit Price, Vendor Name:, Website:
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_4.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_5.jpg
Document status: succeeded
Document page count: 1
Document errors: []
```

### <a name="train-a-model-with-labels"></a>Entrenamiento de un modelo con etiquetas

También puede entrenar modelos personalizados mediante el etiquetado manual de los documentos de entrenamiento. En algunos escenarios, el entrenamiento con etiquetas conduce a un mejor rendimiento. El valor de `CustomFormModel` devuelto indica los campos que el modelo puede extraer, junto con su precisión estimada en cada campo. El siguiente bloque de código imprime esta información en la consola.

> [!IMPORTANT]
> Para realizar el entrenamiento con etiquetas, debe disponer de archivos de información con etiquetas especiales (`\<filename\>.pdf.labels.json`) en el contenedor de almacenamiento de blobs junto con los documentos para el entrenamiento. La [herramienta de etiquetado de ejemplo de Form Recognizer](../../quickstarts/label-tool.md) proporciona una interfaz de usuario para ayudarle a crear estos archivos de etiqueta. Cuando los tenga, puede llamar a la función `begin_training` con el parámetro *use_training_labels* establecido en `true`.

```python
# To train a model you need an Azure Storage account.
# Use the SAS URL to access your training files.
trainingDataUrl = "<SAS-URL-of-your-form-folder-in-blob-storage>"

poller = form_training_client.begin_training(trainingDataUrl, use_training_labels=True)
model = poller.result()

print("Model ID: {}".format(model.model_id))
print("Status: {}".format(model.status))
print("Training started on: {}".format(model.training_started_on))
print("Training completed on: {}".format(model.training_completed_on))

print("\nRecognized fields:")
for submodel in model.submodels:
    print(
        "The submodel with form type '{}' has recognized the following fields: {}".format(
            submodel.form_type,
            ", ".join(
                [
                    field.label if field.label else name
                    for name, field in submodel.fields.items()
                ]
            ),
        )
    )

# Training result information
for doc in model.training_documents:
    print("Document name: {}".format(doc.name))
    print("Document status: {}".format(doc.status))
    print("Document page count: {}".format(doc.page_count))
    print("Document errors: {}".format(doc.errors))
```

### <a name="output"></a>Output

Esta es la salida para un modelo entrenado con los datos de entrenamiento disponibles en el [SDK de Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/formrecognizer/azure-ai-formrecognizer/samples/sample_forms/training).

```console
Model ID: ae636292-0b14-4e26-81a7-a0bfcbaf7c91

Status: ready
Training started on: 2020-08-20 23:20:56+00:00
Training completed on: 2020-08-20 23:20:57+00:00

Recognized fields:
The submodel with form type 'form-ae636292-0b14-4e26-81a7-a0bfcbaf7c91' has recognized the following fields: CompanyAddress, CompanyName, CompanyPhoneNumber, DatedAs, Email, Merchant, PhoneNumber, PurchaseOrderNumber, Quantity, Signature, Subtotal, Tax, Total, VendorName, Website
Document name: Form_1.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_2.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_3.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_4.jpg
Document status: succeeded
Document page count: 1
Document errors: []
Document name: Form_5.jpg
Document status: succeeded
Document page count: 1
Document errors: []
```

## <a name="analyze-forms-with-a-custom-model"></a>Analizar formularios con un modelo personalizado

En esta sección, se muestra cómo extraer información de clave-valor y otro contenido de los tipos de formulario personalizados, con los modelos que ha entrenado con sus propios formularios.

> [!IMPORTANT]
> Para implementar este escenario, debe haber entrenado ya un modelo para que pueda pasar su identificador al método siguiente. Consulte la sección [Entrenamiento de un modelo](#train-a-model-without-labels).

Usará el método `begin_recognize_custom_forms_from_url`. El valor devuelto es una colección de objetos `RecognizedForm`, uno para cada página del documento enviado. El código siguiente imprime los resultados del análisis en la consola. Imprime cada campo reconocido y el valor correspondiente, junto con una puntuación de confianza.

```python
# Model ID from when you trained your model.
model_id = "<your custom model id>"

poller = form_recognizer_client.begin_recognize_custom_forms_from_url(
    model_id=model_id, form_url=formUrl)
result = poller.result()

for recognized_form in result:
    print("Form type: {}".format(recognized_form.form_type))
    for name, field in recognized_form.fields.items():
        print("Field '{}' has label '{}' with value '{}' and a confidence score of {}".format(
            name,
            field.label_data.text if field.label_data else name,
            field.value,
            field.confidence
        ))
```

### <a name="output"></a>Resultados

Con el modelo del ejemplo anterior, se proporciona la siguiente salida.

```console
Form type: form-ae636292-0b14-4e26-81a7-a0bfcbaf7c91
Field 'Merchant' has label 'Merchant' with value 'Invoice For:' and a confidence score of 0.116
Field 'CompanyAddress' has label 'CompanyAddress' with value '1 Redmond way Suite 6000 Redmond, WA' and a confidence score of 0.258
Field 'Website' has label 'Website' with value '99243' and a confidence score of 0.114
Field 'VendorName' has label 'VendorName' with value 'Charges' and a confidence score of 0.145
Field 'CompanyPhoneNumber' has label 'CompanyPhoneNumber' with value '$56,651.49' and a confidence score of 0.249
Field 'CompanyName' has label 'CompanyName' with value 'PT' and a confidence score of 0.245
Field 'DatedAs' has label 'DatedAs' with value 'None' and a confidence score of None
Field 'Email' has label 'Email' with value 'None' and a confidence score of None
Field 'PhoneNumber' has label 'PhoneNumber' with value 'None' and a confidence score of None
Field 'PurchaseOrderNumber' has label 'PurchaseOrderNumber' with value 'None' and a confidence score of None
Field 'Quantity' has label 'Quantity' with value 'None' and a confidence score of None
Field 'Signature' has label 'Signature' with value 'None' and a confidence score of None
Field 'Subtotal' has label 'Subtotal' with value 'None' and a confidence score of None
Field 'Tax' has label 'Tax' with value 'None' and a confidence score of None
Field 'Total' has label 'Total' with value 'None' and a confidence score of None
```

## <a name="manage-your-custom-models"></a>Administración de modelos personalizados

En esta sección se muestra cómo administrar los modelos personalizados almacenados en su cuenta. 

### <a name="check-the-number-of-models-in-the-formrecognizer-resource-account"></a>Comprobación del número de modelos de la cuenta de recursos de FormRecognizer

El siguiente bloque de código comprueba cuántos modelos se han guardado en la cuenta de Form Recognizer y compara esta cifra con el límite de la cuenta.

```python
account_properties = form_training_client.get_account_properties()
print("Our account has {} custom models, and we can have at most {} custom models".format(
    account_properties.custom_model_count, account_properties.custom_model_limit
))
```

### <a name="output"></a>Resultados

```console
Our account has 5 custom models, and we can have at most 5000 custom models
```

### <a name="list-the-models-currently-stored-in-the-resource-account"></a>Enumeración de los modelos almacenados actualmente en la cuenta de recursos

El siguiente bloque de código enumera los modelos actuales de la cuenta e imprime los detalles en la consola. También se guarda una referencia al primer modelo.

```python
# Next, we get a paged list of all of our custom models
custom_models = form_training_client.list_custom_models()

print("We have models with the following ids:")

# Let's pull out the first model
first_model = next(custom_models)
print(first_model.model_id)
for model in custom_models:
    print(model.model_id)
```

### <a name="output"></a>Resultados

Esta es una salida de ejemplo para la cuenta de prueba.

```console
We have models with the following ids:
453cc2e6-e3eb-4e9f-aab6-e1ac7b87e09e
628739de-779c-473d-8214-d35c72d3d4f7
ae636292-0b14-4e26-81a7-a0bfcbaf7c91
b4b5df77-8538-4ffb-a996-f67158ecd305
c6309148-6b64-4fef-aea0-d39521452699
```

### <a name="get-a-specific-model-using-the-models-id"></a>Obtención de un modelo específico mediante el identificador del modelo

El siguiente bloque de código usa el identificador de modelo guardado en la sección anterior y lo emplea para recuperar detalles sobre el modelo.

```python
model_id = "<model_id from the Train a Model sample>"

custom_model = form_training_client.get_custom_model(model_id=model_id)
print("Model ID: {}".format(custom_model.model_id))
print("Status: {}".format(custom_model.status))
print("Training started on: {}".format(custom_model.training_started_on))
print("Training completed on: {}".format(custom_model.training_completed_on))
```

### <a name="output"></a>Resultados

Esta es la salida de ejemplo para el modelo personalizado creado en el ejemplo anterior.

```console
Model ID: ae636292-0b14-4e26-81a7-a0bfcbaf7c91
Status: ready
Training started on: 2020-08-20 23:20:56+00:00
Training completed on: 2020-08-20 23:20:57+00:00
```

### <a name="delete-a-model-from-the-resource-account"></a>Eliminación de un modelo de la cuenta de recursos

También puede eliminar un modelo de su cuenta haciendo referencia a su identificador. Este código elimina el modelo usado en la sección anterior.

```python
form_training_client.delete_model(model_id=custom_model.model_id)

try:
    form_training_client.get_custom_model(model_id=custom_model.model_id)
except ResourceNotFoundError:
    print("Successfully deleted model with id {}".format(custom_model.model_id))
```

## <a name="run-the-application"></a>Ejecución de la aplicación

Puede ejecutar la aplicación en cualquier momento con cualquier número de funciones que haya leído en esta guía de inicio rápido con este comando:

```console
python quickstart-file.py
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si quiere limpiar y eliminar una suscripción a Cognitive Services, puede eliminar el recurso o grupo de recursos. Al eliminar el grupo de recursos, también se elimina cualquier otro recurso que esté asociado a él.

* [Portal](../../../cognitive-services-apis-create-account.md#clean-up-resources)
* [CLI de Azure](../../../cognitive-services-apis-create-account-cli.md#clean-up-resources)

## <a name="troubleshooting"></a>Solución de problemas

### <a name="general"></a>General

La biblioteca cliente de Form Recognizer producirá excepciones definidas en [Azure Core](https://aka.ms/azsdk-python-azure-core).

## <a name="logging"></a>Registro

Esta biblioteca usa la [biblioteca de registro estándar](https://docs.python.org/3/library/logging.html) para el registro. La información básica sobre las sesiones HTTP (direcciones URL, encabezados, etc.) se registra en el nivel de información.

El registro detallado del nivel de depuración, incluidos los cuerpos de solicitud y respuesta, y los encabezados no redactados, se puede habilitar en un cliente con el argumento de palabra clave `logging_enable`:

```python
import sys
import logging
from azure.ai.formrecognizer import FormRecognizerClient
from azure.core.credentials import AzureKeyCredential

# Create a logger for the 'azure' SDK
logger = logging.getLogger('azure')
logger.setLevel(logging.DEBUG)

# Configure a console output
handler = logging.StreamHandler(stream=sys.stdout)
logger.addHandler(handler)

endpoint = "https://<my-custom-subdomain>.cognitiveservices.azure.com/"
credential = AzureKeyCredential("<api_key>")

# This client will log detailed information about its HTTP sessions, at DEBUG level
form_recognizer_client = FormRecognizerClient(endpoint, credential, logging_enable=True)
```

Igualmente, `logging_enable` puede habilitar el registro detallado de una sola operación, aunque no esté habilitado para el cliente:

```python
poller = form_recognizer_client.begin_recognize_receipts(receipt, logging_enable=True)
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, se ha usado la biblioteca cliente de Python para Form Recognizer para entrenar modelos y analizar formularios de maneras diferentes. A continuación, obtenga sugerencias para crear un mejor conjunto de datos de entrenamiento y generar modelos más precisos.

> [!div class="nextstepaction"]
> [Creación de un conjunto de datos de aprendizaje](../../build-training-data-set.md)

## <a name="see-also"></a>Consulte también

* [¿Qué es Form Recognizer?](../../overview.md)
