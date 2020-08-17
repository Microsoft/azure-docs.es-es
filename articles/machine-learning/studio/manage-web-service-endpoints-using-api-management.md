---
title: 'ML Studio (clásico): Administración de servicios web mediante API Management: Azure'
description: Una guía que muestra cómo administrar los servicios web de AzureML mediante la Administración de API. Administre los puntos de conexión de la API REST mediante la definición del acceso del usuario, la limitación de uso y la supervisión del panel.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 11/03/2017
ms.openlocfilehash: 08fcf3540b2f8677f3fbfff49f21f99062d0ca40
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87433260"
---
# <a name="manage-azure-machine-learning-studio-classic-web-services-using-api-management"></a>Administración de servicios web de Azure Machine Learning Studio (clásico) con API Management

**SE APLICA A:**  ![sí](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (clásico)   ![no](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../compare-azure-ml-to-studio-classic.md)


## <a name="overview"></a>Información general
En esta guía, se explica cómo empezar a usar rápidamente API Management para administrar los servicios web de Azure Machine Learning Studio (clásico).

## <a name="what-is-azure-api-management"></a>¿Qué es la Azure API Management?
Azure API Management es un servicio de Azure que le permite administrar los extremos de la API de REST al definir el acceso del usuario, el límite de uso y la supervisión de panel. Consulte el [sitio de Azure API Management](https://azure.microsoft.com/services/api-management/) para más información. Para empezar a trabajar con Azure API Management, consulte [la guía de importación y publicación](/azure/api-management/import-and-publish). Esta otra guía, en la que está basada esta guía, aborda más temas, incluidos las configuraciones de notificación, el nivel de precios, el control de respuestas, la autenticación de los usuarios, la creación de productos, las suscripciones de desarrollador y los paneles de uso.

## <a name="prerequisites"></a>Requisitos previos
Para completar a esta guía, necesita:

* Una cuenta de Azure.
* Una cuenta de AzureML.
* El área de trabajo, el servicio y la api_key para un experimento de Aprendizaje automático de Azure implementado como un servicio web. Para más información sobre cómo crear un experimento de Aprendizaje automático de Azure consulte el [inicio rápido de Studio](create-experiment.md). Para información sobre cómo implementar un experimento de Studio (clásico) como servicio web, vea el [procedimiento de implementación de Studio](deploy-a-machine-learning-web-service.md) sobre cómo implementar un experimento de Aprendizaje automático de Azure como un servicio web. Además, el Apéndice A contiene instrucciones sobre cómo crear y probar un experimento de Aprendizaje automático de Azure sencillo e implementarlo como un servicio web.

## <a name="create-an-api-management-instance"></a>Creación de una instancia de API Management

Puede administrar el servicio web Azure Machine Learning con una instancia de API Management.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **+ Crear un recurso**.
3. En el cuadro Buscar, escriba "API management" y, a continuación, seleccione el recurso "API management".
4. Haga clic en **Crear**.
5. El valor de **Nombre** se usará para crear una dirección URL única (en este ejemplo se utiliza "demoazureml").
6. Seleccione los valores de **Suscripción**, **Grupo de recursos** y **Ubicación** de la instancia de servicio.
7. Especifique el valor de **Nombre de organización** (en este ejemplo se utiliza "demoazureml").
8. Escriba su **correo electrónico de administrador**: esta dirección de correo electrónico se utilizará para las notificaciones desde el sistema de API Management.
9. Haga clic en **Crear**.

El nuevo servicio puede tardar hasta 30 minutos en crearse.

![create-service](./media/manage-web-service-endpoints-using-api-management/create-service.png)


## <a name="create-the-api"></a>Creación de la API
Una vez creada la instancia de servicio, el paso siguiente es crear la API. Una API consta de un conjunto de operaciones que se pueden invocar desde una aplicación cliente. Las operaciones API se realizan con proxy en servicios web existentes. Esta guía crea las API que representan los servicios web de AzureML RRS y BES existentes.

Para crear la API:

1. En Azure Portal, abra la instancia del servicio que creó.
2. En el panel de navegación izquierdo, haga clic en **API**.

   ![api-management-menu](./media/manage-web-service-endpoints-using-api-management/api-management.png)

1. Haga clic en **Agregar API**.
2. Escriba un **nombre de API web** (en este ejemplo se usa "AzureML Demo API").
3. En **Dirección URL de servicio web**, escriba "`https://ussouthcentral.services.azureml.net`".
4. Especifique un **Sufijo de URL de API web". Esto se convertirá en la última parte de la dirección URL que los clientes usarán para enviar solicitudes a la instancia del servicio (en este ejemplo se utiliza "azureml-demo").
5. En **Web API URL scheme** (Esquema de URL de API web), seleccione **HTTPS**.
6. For **Products** (Productos), seleccione **Starter** (Motor de arranque).
7. Haga clic en **Save**(Guardar).


## <a name="add-the-operations"></a>Adición de operaciones

Las operaciones se agregan y se configuran para una API en el portal del publicador. Para acceder al portal para editores, haga clic en **Portal para editores** en Azure Portal, en el servicio API Management, seleccione **API**, **Operaciones** y, finalmente, haga clic en **Agregar operación**.

![add-operation](./media/manage-web-service-endpoints-using-api-management/add-an-operation.png)

Se mostrará la ventana **Nueva operación** y la pestaña **Firma** se seleccionará de forma predeterminada.

## <a name="add-rrs-operation"></a>Adición de una operación de registro de recursos
En primer lugar, cree una operación para el servicio RRS de AzureML:

1. En **Verbo HTTP**, seleccione **POST**.
2. En **Modelo de URL**, escriba "`/workspaces/{workspace}/services/{service}/execute?api-version={apiversion}&details={details}`".
3. Escriba un **nombre para mostrar** (en este ejemplo se usa "RRS Execute").

   ![add-rrs-operation-signature](./media/manage-web-service-endpoints-using-api-management/add-rrs-operation-signature.png)

4. Haga clic en **Respuestas** > **AGREGAR** a la izquierda y seleccione **200 Aceptar**.
5. Haga clic en **Guardar** para guardar esta operación.

   ![add-rrs-operation-response](./media/manage-web-service-endpoints-using-api-management/add-rrs-operation-response.png)

## <a name="add-bes-operations"></a>Adición de operaciones BES

> [!NOTE]
> Aquí no se incluyen las capturas de pantalla de las operaciones BES, ya que son muy similares a las de agregar la operación RRS.

### <a name="submit-but-not-start-a-batch-execution-job"></a>Envío (pero no inicio) de un trabajo de ejecución por lotes

1. Haga clic en **Agregar operación** para agregar la operación BES a la API.
2. En **Verbo HTTP**, seleccione **POST**.
3. En **Modelo de URL**, escriba "`/workspaces/{workspace}/services/{service}/jobs?api-version={apiversion}`".
4. Escriba un **nombre para mostrar** (en este ejemplo se usa "BES Submit").
5. Haga clic en **Respuestas** > **AGREGAR** a la izquierda y seleccione **200 Aceptar**.
6. Haga clic en **Save**(Guardar).

### <a name="start-a-batch-execution-job"></a>Inicio de un trabajo de ejecución por lotes

1. Haga clic en **Agregar operación** para agregar la operación BES a la API.
2. En **Verbo HTTP**, seleccione **POST**.
3. En **Verbo HTTP**, escriba "`/workspaces/{workspace}/services/{service}/jobs/{jobid}/start?api-version={apiversion}`".
4. Escriba un **nombre para mostrar** (en este ejemplo se usa "BES Start").
6. Haga clic en **Respuestas** > **AGREGAR** a la izquierda y seleccione **200 Aceptar**.
7. Haga clic en **Save**(Guardar).

### <a name="get-the-status-or-result-of-a-batch-execution-job"></a>Obtener el estado o el resultado de un trabajo de ejecución por lotes

1. Haga clic en **Agregar operación** para agregar la operación BES a la API.
2. En **Verbo HTTP**, seleccione **GET**.
3. En **Modelo de URL**, escriba "`/workspaces/{workspace}/services/{service}/jobs/{jobid}?api-version={apiversion}`".
4. Escriba un **nombre para mostrar** (en este ejemplo se usa "BES Status").
6. Haga clic en **Respuestas** > **AGREGAR** a la izquierda y seleccione **200 Aceptar**.
7. Haga clic en **Save**(Guardar).

### <a name="delete-a-batch-execution-job"></a>Eliminación de un trabajo de ejecución por lotes

1. Haga clic en **Agregar operación** para agregar la operación BES a la API.
2. En **Verbo HTTP**, seleccione **DELETE**.
3. En **Modelo de URL**, escriba "`/workspaces/{workspace}/services/{service}/jobs/{jobid}?api-version={apiversion}`".
4. Escriba un **nombre para mostrar** (en este ejemplo se usa "BES Delete").
5. Haga clic en **Respuestas** > **AGREGAR** a la izquierda y seleccione **200 Aceptar**.
6. Haga clic en **Save**(Guardar).

## <a name="call-an-operation-from-the-developer-portal"></a>Llamada a una operación desde el portal para desarrolladores

Se puede llamar a las operaciones directamente desde el portal para desarrolladores, lo que proporciona una forma cómoda de ver y probar las operaciones de una API. En este paso, llamará al método **RRS Execute** que se agregó a la **API de demostración de AzureML**. 

1. Haga clic en **Portal para desarrolladores**.

   ![developer-portal](./media/manage-web-service-endpoints-using-api-management/developer-portal.png)

2. Haga clic en **API** en el menú superior y, a continuación, haga clic en **API de demostración de AzureML** para ver las operaciones disponibles.

   ![demoazureml-api](./media/manage-web-service-endpoints-using-api-management/demoazureml-api.png)

3. Seleccione **Ejecución de RRS** para la operación. Haga clic en **Pruébelo**.

   ![try-it](./media/manage-web-service-endpoints-using-api-management/try-it.png)

4. En **Request parameters** (Parámetros de solicitud), especifique su área de trabajo en **workspace** y **service**, respectivamente, escriba "2.0 en **apiversion** y "true" en **details**. Puede encontrar el **área de trabajo** y el **servicio** en el panel del servicio web AzureML (consulte **Prueba del servicio web** en el apéndice A).

   En **Request headers** (Encabezados de solicitud), haga clic en **Add header** (Agregar encabezado) y escriba "Content-Type" y "application/json". Haga clic en **Agregar encabezado** y escriba "Autorización" y "Portador *\<your service API-KEY\>* ". La CLAVE de API se puede encontrar en el panel del servicio web de AzureML (consulte **Prueba del servicio web** en el apéndice A).

   En **Request body** (Cuerpo de la solicitud), escriba `{"Inputs": {"input1": {"ColumnNames": ["Col2"], "Values": [["This is a good day"]]}}, "GlobalParameters": {}}`.

   ![azureml-demo-api](./media/manage-web-service-endpoints-using-api-management/azureml-demo-api.png)

5. Haga clic en **Enviar**.

   ![Enviar](./media/manage-web-service-endpoints-using-api-management/send.png)

Después de invocar una operación, el portal para desarrolladores mostrará el campo **Dirección URL solicitada** en el servicio de back-end, así como los campos **Estado de respuesta**, **Encabezados de respuesta** y **Contenido de respuesta**.

![response-status](./media/manage-web-service-endpoints-using-api-management/response-status.png)

## <a name="appendix-a---creating-and-testing-a-simple-azureml-web-service"></a>Apéndice A: Creación y prueba de un servicio web sencillo de AzureML
### <a name="creating-the-experiment"></a>Creación del experimento
A continuación se muestran los pasos para crear un experimento de Aprendizaje automático de Azure sencillo e implementarlo como un servicio web. El servicio web toma como entrada una columna de texto arbitrario de entrada y devuelve un conjunto de características representadas como números enteros. Por ejemplo:

| Texto | Texto con hash |
| --- | --- |
| Este es un buen día |1 1 2 2 0 2 0 1 |

En primer lugar, mediante el explorador que prefiera, vaya a: [https://studio.azureml.net/](https://studio.azureml.net/) y escriba sus credenciales para iniciar sesión. Después, cree un nuevo experimento en blanco.

![search-experiment-templates](./media/manage-web-service-endpoints-using-api-management/search-experiment-templates.png)

Cambie su nombre a **SimpleFeatureHashingExperiment**. Expanda **Conjuntos de datos guardados** y arrastre **Reseñas de libros de Amazon** al experimento.

![simple-feature-hashing-experiment](./media/manage-web-service-endpoints-using-api-management/simple-feature-hashing-experiment.png)

Expanda **Transformación de datos** y **Manipulación** y arrastre **Seleccionar columnas de conjunto de datos** al experimento. Conecte **Reseñas de libros de Amazon** a **Seleccionar columnas de conjunto de datos**.

![Conexión del módulo de conjunto de datos de las reseñas de libros a un módulo de proyección de columnas](./media/manage-web-service-endpoints-using-api-management/project-columns.png)

Haga clic en **Seleccionar columnas de conjunto de datos** y después haga clic en **Iniciar el selector de columnas** y seleccione **Col2**. Haga clic en la marca de verificación para aplicar estos cambios.

![Selección de columnas mediante nombres de columna](./media/manage-web-service-endpoints-using-api-management/select-columns.png)

Expanda **Análisis de texto** y arrastre **Hash de características** al experimento. Conecte **Seleccionar columnas de conjunto de datos** a **Hash de características**.

![connect-project-columns](./media/manage-web-service-endpoints-using-api-management/connect-project-columns.png)

Escriba **3** en **Tamaño de bits de hash**. Se crearán 8 (23) columnas.

![hashing-bitsize](./media/manage-web-service-endpoints-using-api-management/hashing-bitsize.png)

En este punto, quizá quiera hacer clic en **Ejecutar** para probar el experimento.

![Ejecutar](./media/manage-web-service-endpoints-using-api-management/run.png)

### <a name="create-a-web-service"></a>Creación de un servicio web
Ahora va a crear un servicio web. Expanda **Servicio web** y arrastre **Entrada** al experimento. Conecte **Entrada** a **Hash de características**. Arrastre también **Resultado** al experimento. Conecte **Resultado** a **Hash de características**.

![output-to-feature-hashing](./media/manage-web-service-endpoints-using-api-management/output-to-feature-hashing.png)

Haga clic en **Publicar servicio web**.

![publish-web-service](./media/manage-web-service-endpoints-using-api-management/publish-web-service.png)

Haga clic en **Sí** para publicar el experimento.

![yes-to-publish](./media/manage-web-service-endpoints-using-api-management/yes-to-publish.png)

### <a name="test-the-web-service"></a>Prueba del servicio web
Un servicio web de AzureML consta de los extremos RRS (servicio de solicitud/respuesta) y BES (servicio de ejecución por lotes). RRS sirve para la ejecución sincrónica. BES sirve para la ejecución por lotes asincrónica. Para probar el servicio web con el origen de Python del ejemplo siguiente, puede que necesite descargar e instalar el SDK de Azure para Python (consulte: [Instalación de Python](/azure/developer/python/azure-sdk-install)).

También necesitará el **área de trabajo**, el **servicio** y la **api_key** del experimento para el origen de ejemplo siguiente. Puede encontrar el área de trabajo y el servicio haciendo clic en **Solicitud-respuesta** o en **Ejecución de lotes** del experimento en el panel del servicio web.

![find-workspace-and-service](./media/manage-web-service-endpoints-using-api-management/find-workspace-and-service.png)

Puede encontrar la **api_key** haciendo clic en el experimento del panel del servicio web.

![find-api-key](./media/manage-web-service-endpoints-using-api-management/find-api-key.png)

#### <a name="test-rrs-endpoint"></a>Prueba del extremo RRS
##### <a name="test-button"></a>Botón Probar
Una manera fácil de probar el extremo RRS es hacer clic en **Probar** en el panel del servicio web.

![test](./media/manage-web-service-endpoints-using-api-management/test.png)

Escriba **Este es un buen día** para **col2**. Haga clic en la marca de verificación.

![enter-data](./media/manage-web-service-endpoints-using-api-management/enter-data.png)

Verá algo parecido a lo siguiente:

![sample-output](./media/manage-web-service-endpoints-using-api-management/sample-output.png)

##### <a name="sample-code"></a>Código de ejemplo
Otra forma de probar el RRS es desde el código de cliente. Si hace clic en **Solicitud-respuesta** en el panel y se desplaza hasta la parte inferior, verá el código de ejemplo de C#, Python y R. También verá la sintaxis de la solicitud de RRS, incluidos el URI de solicitud, los encabezados y el cuerpo.

Esta guía muestra un ejemplo de Python en funcionamiento. Necesitará modificarlo con el **área de trabajo**, el **servicio** y la **api_key** del experimento.

```python
import urllib2
import json
workspace = "<REPLACE WITH YOUR EXPERIMENT'S WEB SERVICE WORKSPACE ID>"
service = "<REPLACE WITH YOUR EXPERIMENT'S WEB SERVICE SERVICE ID>"
api_key = "<REPLACE WITH YOUR EXPERIMENT'S WEB SERVICE API KEY>"
data = {
"Inputs": {
    "input1": {
        "ColumnNames": ["Col2"],
        "Values": [ [ "This is a good day" ] ]
    },
},
"GlobalParameters": { }
}
url = "https://ussouthcentral.services.azureml.net/workspaces/" + workspace + "/services/" + service + "/execute?api-version=2.0&details=true"
headers = {'Content-Type':'application/json', 'Authorization':('Bearer '+ api_key)}
body = str.encode(json.dumps(data))
try:
    req = urllib2.Request(url, body, headers)
    response = urllib2.urlopen(req)
    result = response.read()
    print "result:" + result
        except urllib2.HTTPError, error:
    print("The request failed with status code: " + str(error.code))
    print(error.info())
    print(json.loads(error.read()))
```

#### <a name="test-bes-endpoint"></a>Prueba de extremo BES
Haga clic en **Ejecución de lotes** en el panel y desplácese hasta la parte inferior. Verá el código de ejemplo de C#, Python y R. También verá la sintaxis de las solicitudes BES para enviar un trabajo, iniciar un trabajo, obtener el estado o los resultados de un trabajo y eliminar un trabajo.

Esta guía muestra un ejemplo de Python en funcionamiento. Debe modificarlo con el **área de trabajo**, el **servicio** y la **api_key** del experimento. Además, deberá modificar el **nombre de la cuenta de almacenamiento**, la **clave de la cuenta de almacenamiento** y el **nombre de contenedor de almacenamiento**. Por último, deberá modificar la ubicación del **archivo de entrada** y la ubicación del **archivo de salida**.

```python
import urllib2
import json
import time
from azure.storage import *
workspace = "<REPLACE WITH YOUR WORKSPACE ID>"
service = "<REPLACE WITH YOUR SERVICE ID>"
api_key = "<REPLACE WITH THE API KEY FOR YOUR WEB SERVICE>"
storage_account_name = "<REPLACE WITH YOUR AZURE STORAGE ACCOUNT NAME>"
storage_account_key = "<REPLACE WITH YOUR AZURE STORAGE KEY>"
storage_container_name = "<REPLACE WITH YOUR AZURE STORAGE CONTAINER NAME>"
input_file = "<REPLACE WITH THE LOCATION OF YOUR INPUT FILE>" # Example: C:\\mydata.csv
output_file = "<REPLACE WITH THE LOCATION OF YOUR OUTPUT FILE>" # Example: C:\\myresults.csv
input_blob_name = "mydatablob.csv"
output_blob_name = "myresultsblob.csv"
def printHttpError(httpError):
print("The request failed with status code: " + str(httpError.code))
print(httpError.info())
print(json.loads(httpError.read()))
return
def saveBlobToFile(blobUrl, resultsLabel):
print("Reading the result from " + blobUrl)
try:
    response = urllib2.urlopen(blobUrl)
except urllib2.HTTPError, error:
    printHttpError(error)
    return
with open(output_file, "wb+") as f:
    f.write(response.read())
print(resultsLabel + " have been written to the file " + output_file)
return
def processResults(result):
first = True
results = result["Results"]
for outputName in results:
    result_blob_location = results[outputName]
    sas_token = result_blob_location["SasBlobToken"]
    base_url = result_blob_location["BaseLocation"]
    relative_url = result_blob_location["RelativeLocation"]
    print("The results for " + outputName + " are available at the following Azure Storage location:")
    print("BaseLocation: " + base_url)
    print("RelativeLocation: " + relative_url)
    print("SasBlobToken: " + sas_token)
    if (first):
        first = False
        url3 = base_url + relative_url + sas_token
        saveBlobToFile(url3, "The results for " + outputName)
return

def invokeBatchExecutionService():
url = "https://ussouthcentral.services.azureml.net/workspaces/" + workspace +"/services/" + service +"/jobs"
blob_service = BlobService(account_name=storage_account_name, account_key=storage_account_key)
print("Uploading the input to blob storage...")
data_to_upload = open(input_file, "r").read()
blob_service.put_blob(storage_container_name, input_blob_name, data_to_upload, x_ms_blob_type="BlockBlob")
print "Uploaded the input to blob storage"
input_blob_path = "/" + storage_container_name + "/" + input_blob_name
connection_string = "DefaultEndpointsProtocol=https;AccountName=" + storage_account_name + ";AccountKey=" + storage_account_key
payload =  {
    "Input": {
        "ConnectionString": connection_string,
        "RelativeLocation": input_blob_path
    },
    "Outputs": {
        "output1": { "ConnectionString": connection_string, "RelativeLocation": "/" + storage_container_name + "/" + output_blob_name },
    },
    "GlobalParameters": {
    }
}
    body = str.encode(json.dumps(payload))
headers = { "Content-Type":"application/json", "Authorization":("Bearer " + api_key)}
print("Submitting the job...")
# submit the job
req = urllib2.Request(url + "?api-version=2.0", body, headers)
try:
    response = urllib2.urlopen(req)
except urllib2.HTTPError, error:
    printHttpError(error)
    return
result = response.read()
job_id = result[1:-1] # remove the enclosing double-quotes
print("Job ID: " + job_id)
# start the job
print("Starting the job...")
req = urllib2.Request(url + "/" + job_id + "/start?api-version=2.0", "", headers)
try:
    response = urllib2.urlopen(req)
except urllib2.HTTPError, error:
    printHttpError(error)
    return
url2 = url + "/" + job_id + "?api-version=2.0"

while True:
    print("Checking the job status...")
    # If you are using Python 3+, replace urllib2 with urllib.request in the following code
    req = urllib2.Request(url2, headers = { "Authorization":("Bearer " + api_key) })
    try:
        response = urllib2.urlopen(req)
    except urllib2.HTTPError, error:
        printHttpError(error)
        return
    result = json.loads(response.read())
    status = result["StatusCode"]
    print "status:" + status
    if (status == 0 or status == "NotStarted"):
        print("Job " + job_id + " not yet started...")
    elif (status == 1 or status == "Running"):
        print("Job " + job_id + " running...")
    elif (status == 2 or status == "Failed"):
        print("Job " + job_id + " failed!")
        print("Error details: " + result["Details"])
        break
    elif (status == 3 or status == "Cancelled"):
        print("Job " + job_id + " cancelled!")
        break
    elif (status == 4 or status == "Finished"):
        print("Job " + job_id + " finished!")
        processResults(result)
        break
    time.sleep(1) # wait one second
return
invokeBatchExecutionService()
```
