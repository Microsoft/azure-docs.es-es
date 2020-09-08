---
title: 'Tutorial: Python y AI a través de blobs de Azure'
titleSuffix: Azure Cognitive Search
description: Recorra un ejemplo de extracción de texto y de procesamiento de lenguaje natural por el contenido del almacenamiento de blobs con las API REST de Azure Cognitive Search y el cuaderno de Jupyter Python.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.devlang: python
ms.topic: tutorial
ms.date: 06/12/2020
ms.custom: devx-track-python
ms.openlocfilehash: 39891b69cdb8e7f392657514d255f5f85b3eba60
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936034"
---
# <a name="tutorial-use-python-and-ai-to-generate-searchable-content-from-azure-blobs"></a>Tutorial: Uso de Python y AI para generar contenido que permita búsquedas desde blobs de Azure

Si tiene texto no estructurado o imágenes en Azure Blob Storage, una [canalización de enriquecimiento de inteligencia artificial](cognitive-search-concept-intro.md) puede extraer la información y crear contenido útil en escenarios de búsqueda de texto completo o minería de conocimiento. Aunque una canalización puede procesar los archivos de imagen, este tutorial de Python se centra en el texto y se aplica la detección de idioma y el procesamiento del lenguaje natural para crear campos que se puedan aprovechar en las consultas, las facetas y los filtros.

En este tutorial se usa Python y las [API REST de Search](/rest/api/searchservice/) para realizar las siguientes tareas:

> [!div class="checklist"]
> * Comience con documentos completos (texto no estructurado) como PDF, HTML, DOCX y PPTX en Azure Blob Storage.
> * Defina una canalización que extraiga texto, detecte el idioma, reconozca entidades y detecte frases clave.
> * Defina un índice para almacenar la salida (contenido sin procesar, además de pares nombre-valor generados por canalización).
> * Ejecute la canalización para iniciar transformaciones y análisis, así como para crear y cargar el índice.
> * Explore los resultados mediante la búsqueda de texto completo y una sintaxis de consulta enriquecida.

Si no tiene una suscripción a Azure, abra una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

+ [Almacenamiento de Azure](https://azure.microsoft.com/services/storage/)
+ [Anaconda 3.7](https://www.anaconda.com/distribution/#download-section)
+ [Creación](search-create-service-portal.md) o [búsqueda de un servicio de búsqueda existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Puede usar un servicio gratuito para este tutorial. Un servicio de búsqueda gratuito tiene una limitación de tres índices, tres indexadores y tres orígenes de datos. En este tutorial se crea uno de cada uno. Antes de empezar, asegúrese de que haya espacio en el servicio para aceptar los nuevos recursos.

## <a name="download-files"></a>Descarga de archivos

1. Abra esta [carpeta de OneDrive](https://1drv.ms/f/s!As7Oy81M_gVPa-LCb5lC_3hbS-4) y, en la esquina superior izquierda, haga clic en **Descargar** para copiar los archivos en el equipo. 

1. Haga clic con el botón derecho en el archivo ZIP y seleccione **Extraer todo**. Hay 14 archivos de varios tipos. Para este ejercicio, usará 7.

## <a name="1---create-services"></a>1: Creación de servicios

En este tutorial se usa Azure Cognitive Search en la indexación y las consultas, Cognitive Services en el back-end para el enriquecimiento de inteligencia artificial y Azure Blob Storage en la provisión de datos. Este tutorial permanece por debajo de la asignación gratuita de 20 transacciones por indexador al día en Cognitive Services, por lo que los únicos servicios que tiene que crear son la búsqueda y el almacenamiento.

Si es posible, cree los dos en la misma región y grupo de recursos para la proximidad y la capacidad de administración. En la práctica, la cuenta de Azure Storage puede estar en cualquier región.

### <a name="start-with-azure-storage"></a>Comienzo con Azure Storage

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y haga clic en **+ Crear un recurso**.

1. Busque *cuenta de almacenamiento* y seleccione la oferta de Cuenta de almacenamiento de Microsoft.

   ![Creación de una cuenta de almacenamiento](media/cognitive-search-tutorial-blob/storage-account.png "Crear cuenta de almacenamiento")

1. En la pestaña Datos básicos, se necesitan los siguientes elementos. Acepte los valores predeterminados para todo lo demás.

   + **Grupo de recursos**. Seleccione un grupo existente o cree uno nuevo, pero use el mismo grupo para todos los servicios, con el fin de que pueda administrarlos colectivamente.

   + **Nombre de cuenta de almacenamiento**. Si cree que puede tener varios recursos del mismo tipo, use el nombre para diferenciarlos por tipo y región, por ejemplo *blobstoragewestus*. 

   + **Ubicación**. Si es posible, elija la misma ubicación que se usa para Azure Cognitive Search y Cognitive Services. Una ubicación única anula los cargos de ancho de banda.

   + **Tipo de cuenta**. Elija el valor predeterminado, *StorageV2 (uso general v2)* .

1. Haga clic en **Revisar y crear** para crear el servicio.

1. Una vez creado, haga clic en **Go to the resource** (Ir al recurso) para abrir la página de información general.

1. Haga clic en el servicio **Blobs**.

1. Haga clic en **+ Contenedor** para crear un contenedor y asígnele el nombre *cog-search-demo*.

1. Seleccione *cog-search-demo* y haga clic en **Cargar** para abrir la carpeta en la que guardó los archivos de descarga. Seleccione todos los archivos que no sean de imagen. Debería tener 7 archivos. Haga clic en **Aceptar** para empezar a cargar.

   ![Carga de archivos de ejemplo](media/cognitive-search-tutorial-blob/sample-files.png "Carga de archivos de ejemplo")

1. Antes de salir de Azure Storage, obtenga una cadena de conexión para poder formular una conexión en Azure Cognitive Search. 

   1. Vuelva a la página de información general de la cuenta de almacenamiento (hemos usado *blobstragewestus* como ejemplo). 
   
   1. En el panel de navegación izquierdo, seleccione **Claves de acceso** y copie una de las cadenas de conexión. 

   La cadena de conexión es una dirección URL parecida a la del ejemplo siguiente:

      ```http
      DefaultEndpointsProtocol=https;AccountName=<storageaccountname>;AccountKey=<your account key>;EndpointSuffix=core.windows.net
      ```

1. Guarde la cadena de conexión en el Bloc de notas. La necesitará más adelante, al configurar la conexión del origen de datos.

### <a name="cognitive-services"></a>Cognitive Services

El enriquecimiento de inteligencia artificial está respaldado por Cognitive Services, lo que incluye Text Analytics y Computer Vision para el procesamiento de imágenes y del lenguaje natural. Si su objetivo fuera trabajar con un prototipo o un proyecto reales, en este punto aprovisionaría Cognitive Services (en la misma región que Azure Cognitive Search) para poder conectarlo con las operaciones de indexación.

Dado que en este tutorial solo se usan siete transacciones, puede omitir el aprovisionamiento de recursos, dado que Azure Cognitive Search puede conectarse a Cognitive Services y proporcionar 20 transacciones gratuitas por cada ejecución del indexador. Con la asignación gratuita es suficiente. En el caso de proyectos mayores, planee aprovisionamiento de Cognitive Services en el nivel S0 de pago por uso. Para más información, consulte [Asociación de Cognitive Services](cognitive-search-attach-cognitive-services.md).

### <a name="azure-cognitive-search"></a>Azure Cognitive Search

El tercer componente es Azure Cognitive Search, que se puede [crear en el portal](search-create-service-portal.md). Puede usar el nivel Gratis para realizar este tutorial. 

Al igual que con Azure Blob Storage dedique un momento a recopilar la clave de acceso. Además, cuando empiece a estructurar las solicitudes, deberá proporcionar el punto de conexión y la clave de la API de administración que se usan para autenticar cada solicitud.

### <a name="get-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Obtención de una clave de API de administración y una dirección URL para Azure Cognitive Search

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y, en la página **Información general** del servicio de búsqueda, obtenga el nombre del servicio de búsqueda. Para confirmar el nombre del servicio, revise la dirección URL del punto de conexión. Si la dirección URL del punto de conexión fuera `https://mydemo.search.windows.net`, el nombre del servicio sería `mydemo`.

2. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

   Obtenga también la clave de consulta. Es una práctica recomendada emitir solicitudes de consulta con acceso de solo lectura.

   ![Obtención del nombre del servicio y las claves de consulta y administrador](media/search-get-started-nodejs/service-name-and-keys.png)

Todas las solicitudes enviadas al servicio necesitan una clave de API en el encabezado. Una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que la envía y el servicio que se encarga de ella.

## <a name="2---start-a-notebook"></a>2\. Inicio de un cuaderno

Siga las instrucciones para crear el cuaderno o descargue un cuaderno terminado en el [repositorio Azure-Search-python-samples](https://github.com/Azure-Samples/azure-search-python-samples/tree/master/Tutorial-AI-Enrichment).

Utilice el navegador de Anaconda para iniciar Jupyter Notebook y crear un nuevo cuaderno de Python 3.

En el cuaderno, ejecute este script para cargar las bibliotecas que usa para trabajar con JSON y formular las solicitudes HTTP.

```python
import json
import requests
from pprint import pprint
```

En el mismo cuaderno, defina los nombres para el origen de datos, índice, indexador y conjunto de aptitudes. Ejecute este script para configurar los nombres para este tutorial.

```python
# Define the names for the data source, skillset, index and indexer
datasource_name = "cogsrch-py-datasource"
skillset_name = "cogsrch-py-skillset"
index_name = "cogsrch-py-index"
indexer_name = "cogsrch-py-indexer"
```

En el siguiente script, reemplace los marcadores de posición para el servicio de búsqueda (YOUR-SEARCH-SERVICE-NAME) y clave de API de administración (YOUR-ADMIN-API-KEY) y, a continuación, ejecútelo para configurar el punto de conexión del servicio de búsqueda.

```python
# Setup the endpoint
endpoint = 'https://<YOUR-SEARCH-SERVICE-NAME>.search.windows.net/'
headers = {'Content-Type': 'application/json',
           'api-key': '<YOUR-ADMIN-API-KEY>'}
params = {
    'api-version': '2020-06-30'
}
```

## <a name="3---create-the-pipeline"></a>3: Creación de la canalización

En Azure Cognitive Search, el procesamiento de la inteligencia artificial se produce durante la indexación (o la ingesta de datos). En esta parte del tutorial se crean cuatro objetos: origen de datos, definición de índice, conjunto de aptitudes e indexador. 

### <a name="step-1-create-a-data-source"></a>Paso 1: Creación de un origen de datos

Un [objeto de origen de datos](/rest/api/searchservice/create-data-source) proporciona la cadena de conexión al contenedor de blobs que contiene los archivos.

En el siguiente script, reemplace el marcador de posición YOUR-BLOB-RESOURCE-CONNECTION-STRING con la cadena de conexión para el blob que creó en el paso anterior. Reemplace el texto del marcador de posición del contenedor. A continuación, ejecute el script para crear un origen de datos denominado `cogsrch-py-datasource`.

```python
# Create a data source
datasourceConnectionString = "<YOUR-BLOB-RESOURCE-CONNECTION-STRING>"
datasource_payload = {
    "name": datasource_name,
    "description": "Demo files to demonstrate cognitive search capabilities.",
    "type": "azureblob",
    "credentials": {
        "connectionString": datasourceConnectionString
    },
    "container": {
        "name": "<YOUR-BLOB-CONTAINER-NAME>"
    }
}
r = requests.put(endpoint + "/datasources/" + datasource_name,
                 data=json.dumps(datasource_payload), headers=headers, params=params)
print(r.status_code)
```

La solicitud debe devolver un código de estado 201 que confirme el éxito de la operación.

En Azure Portal, en la página del panel de información del servicio de búsqueda, compruebe que cogsrch-py-datasource aparece en la lista **Orígenes de datos**. Haga clic en **Actualizar**  para actualizar la página.

![Icono de orígenes de datos en el portal](./media/cognitive-search-tutorial-blob-python/py-data-source-tile.png "Icono de orígenes de datos en el portal")

### <a name="step-2-create-a-skillset"></a>Paso 2: Creación de un conjunto de aptitudes

En este paso, definirá un conjunto de pasos de enriquecimiento para aplicar a los datos. Llamará a cada paso de enriquecimiento una *aptitud* y al conjunto de pasos de enriquecimiento, un *conjunto de aptitudes*. Este tutorial usa un conjunto de [aptitudes cognitivas integradas](cognitive-search-predefined-skills.md):

+ [Reconocimiento de entidades](cognitive-search-skill-entity-recognition.md) para extraer los nombres de las organizaciones del contenido del contenedor de blobs.

+ [Detección de idioma](cognitive-search-skill-language-detection.md) para identificar el idioma del contenido.

+ [División de texto](cognitive-search-skill-textsplit.md) para dividir contenido de gran tamaño en fragmentos más pequeños antes de llamar a la aptitud de extracción de frases clave. La extracción de frases clave acepta entradas de 50 000 caracteres o menos. Algunos de los archivos de ejemplo deben dividirse para no superar este límite.

+ [Extracción de frases clave](cognitive-search-skill-keyphrases.md) para extraer las frases clave principales. 

Ejecute el siguiente script para crear un conjunto de aptitudes denominado `cogsrch-py-skillset`.

```python
# Create a skillset
skillset_payload = {
    "name": skillset_name,
    "description":
    "Extract entities, detect language and extract key-phrases",
    "skills":
    [
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "categories": ["Organization"],
            "defaultLanguageCode": "en",
            "inputs": [
                {
                    "name": "text", 
                    "source": "/document/content"
                }
            ],
            "outputs": [
                {
                    "name": "organizations", 
                    "targetName": "organizations"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
            "inputs": [
                {
                    "name": "text", 
                    "source": "/document/content"
                }
            ],
            "outputs": [
                {
                    "name": "languageCode",
                    "targetName": "languageCode"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.SplitSkill",
            "textSplitMode": "pages",
            "maximumPageLength": 4000,
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/content"
                },
                {
                    "name": "languageCode",
                    "source": "/document/languageCode"
                }
            ],
            "outputs": [
                {
                    "name": "textItems",
                    "targetName": "pages"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
            "context": "/document/pages/*",
            "inputs": [
                {
                    "name": "text", 
                    "source": "/document/pages/*"
                },
                {
                    "name": "languageCode", 
                    "source": "/document/languageCode"
                }
            ],
            "outputs": [
                {
                    "name": "keyPhrases",
                    "targetName": "keyPhrases"
                }
            ]
        }
    ]
}

r = requests.put(endpoint + "/skillsets/" + skillset_name,
                 data=json.dumps(skillset_payload), headers=headers, params=params)
print(r.status_code)
```

La solicitud debe devolver un código de estado 201 que confirme el éxito de la operación.

La aptitud de extracción de frases clave se aplica a cada página. Al establecer el contexto en `"document/pages/*"`, se ejecuta este enriquecedor para cada miembro de la matriz de documentos o páginas (para cada página del documento).

Cada aptitud se ejecuta en el contenido del documento. Durante el procesamiento, Azure Cognitive Search extrae cada documento para leer el contenido de diferentes formatos de archivo. El texto encontrado en el archivo de origen se coloca en un campo `content`, uno para cada documento. Por lo tanto, establezca la entrada como `"/document/content"`.

A continuación se muestra una representación gráfica del conjunto de aptitudes.

![Descripción de un conjunto de aptitudes](media/cognitive-search-tutorial-blob/skillset.png "Descripción de un conjunto de aptitudes")

Las salidas se pueden asignar a un índice, usar como entrada para una aptitud de bajada, o ambas cosas como sucede con el código de idioma. En el índice, un código de idioma es útil para el filtrado. Como entrada, el código de idioma se usa en las aptitudes de análisis de texto para informar a las reglas lingüísticas de la separación de palabras.

Para obtener más información sobre los conceptos básicos del conjunto de aptitudes, consulte el tema sobre la [definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md).

### <a name="step-3-create-an-index"></a>Paso 3: Creación de un índice

En esta sección, se define el esquema de índice. Para ello, se especifican los campos que se incluirán en el índice que permite búsquedas y se establecen los atributos de búsqueda de cada campo. Los campos tienen un tipo y pueden tener atributos que determinen cómo se utiliza el campo (si permite búsquedas, se puede ordenar, etc.). Los nombres de campos de un índice no tienen que coincidir exactamente con los nombres de campo del origen. En un paso posterior, agregará asignaciones de campos en un indexador para conectar los campos de origen y destino. Para este paso, defina el índice con convenciones de nomenclatura de campos adecuadas para la aplicación de búsqueda.

En este ejercicio se utilizan los siguientes campos y tipos de campos:

| field-names: | id         | content   | languageCode | keyPhrases         | organizations     |
|--------------|----------|-------|----------|--------------------|-------------------|
| field-types: | Edm.String|Edm.String| Edm.String| List<Edm.String>  | List<Edm.String>  |

Ejecute este script para crear el índice denominado `cogsrch-py-index`.

```python
# Create an index
index_payload = {
    "name": index_name,
    "fields": [
        {
            "name": "id",
            "type": "Edm.String",
            "key": "true",
            "searchable": "true",
            "filterable": "false",
            "facetable": "false",
            "sortable": "true"
        },
        {
            "name": "content",
            "type": "Edm.String",
            "sortable": "false",
            "searchable": "true",
            "filterable": "false",
            "facetable": "false"
        },
        {
            "name": "languageCode",
            "type": "Edm.String",
            "searchable": "true",
            "filterable": "false",
            "facetable": "false"
        },
        {
            "name": "keyPhrases",
            "type": "Collection(Edm.String)",
            "searchable": "true",
            "filterable": "false",
            "facetable": "false"
        },
        {
            "name": "organizations",
            "type": "Collection(Edm.String)",
            "searchable": "true",
            "sortable": "false",
            "filterable": "false",
            "facetable": "false"
        }
    ]
}

r = requests.put(endpoint + "/indexes/" + index_name,
                 data=json.dumps(index_payload), headers=headers, params=params)
print(r.status_code)
```

La solicitud debe devolver un código de estado 201 que confirme el éxito de la operación.

Para más información acerca de cómo definir un índice, consulte [Creación de un índice (API REST de Azure Cognitive Search)](/rest/api/searchservice/create-index).

### <a name="step-4-create-and-run-an-indexer"></a>Paso 4: Creación y ejecución de un indexador

Un [indexador](/rest/api/searchservice/create-indexer) rige la canalización. Los tres componentes que ha creado hasta ahora (origen de datos, conjunto de aptitudes e índice) son entradas para un indexador. La creación del indexador en Azure Cognitive Search es el evento que pone toda la canalización en movimiento. 

Para unir estos objetos en un indexador, tiene que definir las asignaciones de campos.

+ Los elementos `"fieldMappings"` se procesan antes que el conjunto de aptitudes, de forma que se asignan los campos de origen del origen de datos a los campos de destino de un índice. Si los tipos y nombres de campo son los mismos en ambos extremos, no se requiere ninguna asignación.

+ Los elementos `"outputFieldMappings"` se procesan después que el conjunto de aptitudes, y hacen referencia a elementos `"sourceFieldNames"` que no existen hasta que el descifrado o el enriquecimiento de documentos los crean. El elemento `"targetFieldName"` es un campo de un índice.

Además de enlazar las entradas a las salidas, también puede usar las asignaciones de campos para aplanar las estructuras de datos. Para más información, consulte [Asignación de campos enriquecidos a un índice de búsqueda](cognitive-search-output-field-mapping.md).

Ejecute este script para crear un indexador denominado `cogsrch-py-indexer`.

```python
# Create an indexer
indexer_payload = {
    "name": indexer_name,
    "dataSourceName": datasource_name,
    "targetIndexName": index_name,
    "skillsetName": skillset_name,
    "fieldMappings": [
        {
            "sourceFieldName": "metadata_storage_path",
            "targetFieldName": "id",
            "mappingFunction":
            {"name": "base64Encode"}
        },
        {
            "sourceFieldName": "content",
            "targetFieldName": "content"
        }
    ],
    "outputFieldMappings":
    [
        {
            "sourceFieldName": "/document/organizations",
            "targetFieldName": "organizations"
        },
        {
            "sourceFieldName": "/document/pages/*/keyPhrases/*",
            "targetFieldName": "keyPhrases"
        },
        {
            "sourceFieldName": "/document/languageCode",
            "targetFieldName": "languageCode"
        }
    ],
    "parameters":
    {
        "maxFailedItems": -1,
        "maxFailedItemsPerBatch": -1,
        "configuration":
        {
            "dataToExtract": "contentAndMetadata",
            "imageAction": "generateNormalizedImages"
        }
    }
}

r = requests.put(endpoint + "/indexers/" + indexer_name,
                 data=json.dumps(indexer_payload), headers=headers, params=params)
print(r.status_code)
```

La solicitud debe devolver un código de estado 201 pronto, sin embargo, el proceso puede tardar varios minutos en completarse. Aunque el conjunto de datos es pequeño, las aptitudes analíticos, como el análisis de imágenes, son intensivas y tardarán tiempo.

Puede [supervisar el estado del indexador](#check-indexer-status) para determinar cuándo se ejecuta el indexador o finaliza.

> [!TIP]
> La creación de un indexador invoca la canalización. Si hay problemas para acceder a los datos, las entradas y salidas de asignación o el orden de las operaciones, se muestran en esta fase. Para volver a ejecutar la canalización con los cambios de código o script, tendrá que quitar primero los objetos. Para más información, consulte [Restablecer y volver a ejecutar](#reset).

#### <a name="about-the-request-body"></a>Acerca del cuerpo de la solicitud

El script establece `"maxFailedItems"` en -1, que indica al motor de indexación que ignore los errores durante la importación de datos. Esto es útil porque hay muy pocos documentos en el origen de datos de demostración. Para un origen de datos mayor, debería establecer un valor mayor que 0.

Observe también la instrucción `"dataToExtract":"contentAndMetadata"` en los parámetros de configuración. Esta instrucción indica al indexador que extraiga el contenido de diferentes formatos de archivo, y los metadatos relacionados con cada archivo.

Una vez extraído el contenido, puede establecer `imageAction` para que se extraiga el texto de las imágenes que se encuentran en el origen de datos. La configuración de `"imageAction":"generateNormalizedImages"`, en combinación con OCR Skill y Text Merge Skill, le dice al indexador que extraiga texto de las imágenes (por ejemplo, la palabra "stop" de una señal de Stop de tráfico) y lo inserte como parte del campo de contenido. Este comportamiento se aplica a las imágenes insertadas en los documentos (piense en una imagen de un archivo PDF), así como a las imágenes que se encuentran en el origen de datos, como un archivo JPG.

<a name="check-indexer-status"></a>

## <a name="4---monitor-indexing"></a>4: Supervisión de la indexación

Una vez definido el indexador, se ejecuta automáticamente cuando se envía la solicitud. Dependiendo de las aptitudes cognitivas definidas, la indexación puede tardar más de lo esperado. Para averiguar si el procesamiento del indexador está completo, ejecute el siguiente script.

```python
# Get indexer status
r = requests.get(endpoint + "/indexers/" + indexer_name +
                 "/status", headers=headers, params=params)
pprint(json.dumps(r.json(), indent=1))
```

En la respuesta, supervise los valores `"status"` y `"endTime"` en `"lastResult"`. Ejecute el script de forma periódica para comprobar el estado. Cuando haya finalizado el indexador el estado se establecerá en "success", se especificará un "endTime", y la respuesta incluirá cualquier error o advertencia que se produjera durante el enriquecimiento.

![Se crea el indexador](./media/cognitive-search-tutorial-blob-python/py-indexer-is-created.png "Se crea el indexador")

Las advertencias son comunes con algunas combinaciones de aptitudes y archivos de origen y no siempre indican un problema. Muchas advertencias son benignas. Por ejemplo, si indexa un archivo JPEG que no tenga texto, se verá la advertencia de esta captura de pantalla.

![Advertencia de ejemplo del indexador](./media/cognitive-search-tutorial-blob-python/py-indexer-warning-example.png "Advertencia de ejemplo del indexador")

## <a name="5---search"></a>5: Búsqueda

Una vez finalizada la indexación, ejecute consultas que devuelvan el contenido de los campos individuales. De forma predeterminada, Azure Cognitive Search devuelve los primeros 50 resultados. Los datos de ejemplo tiene un tamaño reducido, por lo que la configuración predeterminada funciona bien. Sin embargo, si trabaja con conjuntos de datos más grandes, deberá incluir parámetros en la cadena de consulta para que se devuelvan más resultados. Para ver las instrucciones, consulte [Cómo paginar los resultados de la búsqueda en Azure Cognitive Search](search-pagination-page-layout.md).

Como paso de comprobación, obtenga la definición del índice que muestra todos los campos.

```python
# Query the service for the index definition
r = requests.get(endpoint + "/indexes/" + index_name,
                 headers=headers, params=params)
pprint(json.dumps(r.json(), indent=1))
```

Los resultados deben tener un aspecto similar al del siguiente ejemplo. La captura de pantalla muestra solo una parte de la respuesta.

![Índice de consulta para todos los campos](./media/cognitive-search-tutorial-blob-python/py-query-index-for-fields.png "Índice de consulta para todos los campos")

La salida es el esquema de índice, con el nombre, el tipo y los atributos de cada campo.

Envíe una segunda consulta para `"*"` a fin de devolver todo el contenido de un campo único, como `organizations`.

```python
# Query the index to return the contents of organizations
r = requests.get(endpoint + "/indexes/" + index_name +
                 "/docs?&search=*&$select=organizations", headers=headers, params=params)
pprint(json.dumps(r.json(), indent=1))
```

Los resultados deben tener un aspecto similar al del siguiente ejemplo. La captura de pantalla muestra solo una parte de la respuesta.

![Índice de consulta para los contenidos de las organizaciones](./media/cognitive-search-tutorial-blob-python/py-query-index-for-organizations.png "Consulta del índice para devolver los contenidos de las organizaciones")

Repita el procedimiento con los campos adicionales: `content`, `languageCode`, `keyPhrases` y `organizations` en este ejercicio. Puede devolver varios campos a través de `$select` con una lista delimitada por comas.

Puede usar GET o POST, según la longitud y la complejidad de la cadena de consulta. Para más información, consulte la [Realización de una consulta al índice de Azure Search con la API de REST](/rest/api/searchservice/search-documents).

<a name="reset"></a>

## <a name="reset-and-rerun"></a>Restablecer y volver a ejecutar

En las primeras etapas experimentales de desarrollo, el enfoque más práctico para las iteraciones del diseño es eliminar los objetos de Azure Cognitive Search y permitir que el código vuelva a generarlos. Los nombres de los recursos son únicos. La eliminación de un objeto permite volver a crearlo con el mismo nombre.

También puede usar el portal para eliminar los índices, los indexadores, los orígenes de datos y los conjuntos de aptitudes. Al eliminar el indexador, puede opcionalmente, de forma selectiva elimina el índice, aptitudes y datos de origen al mismo tiempo.

![Eliminación de los objetos de búsqueda](./media/cognitive-search-tutorial-blob-python/py-delete-indexer-delete-all.png "Eliminación de objetos de búsqueda en el portal")

También puede eliminarlos mediante un script. El script siguiente muestra cómo eliminar un conjunto de aptitudes. 

```python
# delete the skillset
r = requests.delete(endpoint + "/skillsets/" + skillset_name,
                    headers=headers, params=params)
pprint(json.dumps(r.json(), indent=1))
```

Se devuelve el código de estado 204 si la eliminación se realiza correctamente.

## <a name="takeaways"></a>Puntos clave

Este tutorial muestra los pasos básicos para la creación de una canalización de indexación enriquecida a través de la creación de componentes: un origen de datos, un conjunto de aptitudes, un índice y un indexador.

Las [aptitudes integradas](cognitive-search-predefined-skills.md) se presentan, junto con las definiciones del conjunto de aptitudes y una forma de encadenamiento de aptitudes, mediante entradas y salidas. También aprendió que `outputFieldMappings` en la definición del indexador es necesario para enrutar los valores enriquecidos de la canalización a un índice que permita búsquedas en un servicio Azure Cognitive Search.

Por último, ha aprendido cómo probar los resultados y restablecer el sistema para otras iteraciones. Ha aprendido que emitir consultas en el índice devuelve la salida creada por la canalización de indexación enriquecida. En esta versión, existe un mecanismo para ver las construcciones internas (documentos enriquecidos creados por el sistema). También ha aprendido cómo comprobar el estado del indexador y qué objetos se tienen que eliminar antes de volver a ejecutar una canalización.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, al final de un proyecto, es recomendable eliminar los recursos que ya no necesite. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede buscar y administrar los recursos en el portal, mediante el vínculo Todos los recursos o Grupos de recursos en el panel de navegación izquierdo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con todos los objetos de una canalización de enriquecimiento de AI, echemos un vistazo más de cerca a las definiciones de actitudes y a los conocimientos individuales.

> [!div class="nextstepaction"]
> [Creación de un conjunto de aptitudes](cognitive-search-defining-skillset.md)