---
title: 'Creación de un conjunto de aptitudes en una canalización de búsqueda cognitiva: Azure Search'
description: Defina la extracción de datos, el procesamiento de lenguaje natural o los pasos de análisis de imagen para enriquecer y extraer información estructurada de los datos para utilizarla en Azure Search.
manager: pablocas
author: luiscabrer
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: 010880304c5011bd5a767650aa0ae6b6103bdde2
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540860"
---
# <a name="how-to-create-a-skillset-in-an-enrichment-pipeline"></a>Creación de un conjunto de aptitudes en una canalización de enriquecimiento

La búsqueda cognitiva extrae y enriquece datos para que se puedan buscar en Azure Search. A los pasos de extracción y enriquecimiento los denominamos *aptitudes cognitivas*, combinadas en un *conjunto de aptitudes* al que se hace referencia durante la indexación. Puede usar un conjunto de habilidades [habilidades integrados](cognitive-search-predefined-skills.md) o habilidades personalizadas (consulte [ejemplo: crear una aptitud personalizada](cognitive-search-create-custom-skill-example.md) para obtener más información).

En este artículo aprenderá a crear una canalización de enriquecimiento para las aptitudes que desee utilizar. Se adjunta un conjunto de aptitudes a un [indexador](search-indexer-overview.md) de Azure Search. Una parte del diseño de canalización, que se trata en este artículo, es la construcción del propio conjunto de aptitudes. 

> [!NOTE]
> Otra parte del diseño de canalización es la especificación de un indexador, que se trata en el [siguiente paso](#next-step). Una definición de indexador incluye una referencia al conjunto de aptitudes, además de las asignaciones de campos que se usan para conectar las entradas a las salidas en el índice de destino.

Puntos clave para recordar:

+ Solo puede tener un conjunto de aptitudes por indexador.
+ Un conjunto de aptitudes debe tener al menos una aptitud.
+ Puede crear varias aptitudes del mismo tipo (por ejemplo, variantes de una aptitud de análisis de imagen).

## <a name="begin-with-the-end-in-mind"></a>Inicio pensando en el final

Un paso inicial recomendado consiste en decidir qué datos se extraen de los datos sin procesar y cómo desea usar esos datos en una solución de búsqueda. La creación de una ilustración de la canalización de enriquecimiento completa puede ayudarle a identificar los pasos necesarios.

Supongamos que está interesado en el procesamiento de un conjunto de comentarios de los analistas financieros. Por cada archivo, desea extraer los nombres de las compañías y la opinión general de los comentarios. También le podría interesar escribir un enriquecedor personalizado que utilice el servicio Entity Search de Bing para buscar información adicional acerca de la compañía, como a qué tipo de negocio se dedica dicha compañía. Básicamente, desea extraer información similar a la siguiente, indexada para cada documento:

| texto de registro | compañías | opinión | descripciones de las compañías |
|--------|-----|-----|-----|
|registro de ejemplo| ["Microsoft", "LinkedIn"] | 0,99 | ["Microsoft Corporation es una compañía de tecnología multinacional estadounidense...", "LinkedIn es una red social orientada a las empresas y al empleo..."]

El siguiente diagrama muestra una canalización de enriquecimiento hipotética:

![Una canalización de enriquecimiento hipotética](media/cognitive-search-defining-skillset/sample-skillset.png "A hypothetical enrichment pipeline")


Una vez que tenga una idea clara de lo que desea en la canalización, puede expresar el conjunto de aptitudes que proporciona estos pasos. Funcionalmente, el conjunto de aptitudes se expresa al cargar la definición de indexador en Azure Search. Para más información sobre cómo cargar el indexador, consulte la [documentación del indexador](https://docs.microsoft.com/rest/api/searchservice/create-indexer).


En el diagrama, el paso de *descifrado de documentos* se realiza automáticamente. Básicamente, Azure Search sabe cómo abrir archivos conocidos y crea un campo *content* que contiene el texto extraído de cada documento. Los cuadros blancos son enriquecedores integrados y el cuadro "Entity Search de Bing" con puntos representa un enriquecedor personalizado que está creando. Como se muestra, el conjunto de aptitudes contiene tres aptitudes.

## <a name="skillset-definition-in-rest"></a>Definición del conjunto de aptitudes en REST

Un conjunto de aptitudes se define como una matriz de aptitudes. Cada aptitud define el origen de sus entradas y el nombre de las salidas generadas. Mediante la [API REST de creación de conjuntos de aptitudes](https://docs.microsoft.com/rest/api/searchservice/create-skillset), puede definir un conjunto de aptitudes que se corresponde con el diagrama anterior: 

```http
PUT https://[servicename].search.windows.net/skillsets/[skillset name]?api-version=2019-05-06
api-key: [admin key]
Content-Type: application/json
```

```json
{
  "description": 
  "Extract sentiment from financial records, extract company names, and then find additional information about each company mentioned.",
  "skills":
  [
    {
      "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
      "context": "/document",
      "categories": [ "Organization" ],
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
      "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "score",
          "targetName": "mySentiment"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
     "description": "Calls an Azure function, which in turn calls Bing Entity Search",
      "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/InvokeTextAnalyticsV3?code=foo",
      "httpHeaders": {
          "Ocp-Apim-Subscription-Key": "foobar"
      },
      "context": "/document/organizations/*",
      "inputs": [
        {
          "name": "query",
          "source": "/document/organizations/*"
        }
      ],
      "outputs": [
        {
          "name": "description",
          "targetName": "companyDescription"
        }
      ]
    }
  ]
}
```

## <a name="create-a-skillset"></a>Creación de un conjunto de aptitudes

Al crear un conjunto de aptitudes, puede proporcionar una descripción para que el conjunto de aptitudes se autodocumente. Una descripción es opcional, pero resulta útil para realizar el seguimiento de lo que hace un conjunto de aptitudes. Puesto que el conjunto de aptitudes es un documento JSON, que no se permiten comentarios; debe usar un elemento `description` para esto.

```json
{
  "description": 
  "This is our first skill set, it extracts sentiment from financial records, extract company names, and then finds additional information about each company mentioned.",
  ...
}
```

La siguiente parte del conjunto de aptitudes es una matriz de aptitudes. Cada aptitud se puede considerar como un tipo primitivo de enriquecimiento. Cada aptitud realiza una tarea pequeña en esta canalización de enriquecimiento. Cada una de ellas toma una entrada (o un conjunto de entradas) y devuelve algunas salidas. Las secciones siguientes se centran en cómo especificar habilidades integrados y personalizados, encadenar conocimientos a través de referencias de entrada y salidas. Las entradas pueden proceder de datos de origen o de otra aptitud. Las salidas se pueden asignar a un campo en un índice de búsqueda o utilizarse como entrada para una aptitud de bajada.

## <a name="add-built-in-skills"></a>Agregue las aptitudes integrados

Echemos un vistazo a la primera aptitud, que está integrado [habilidad de reconocimiento de entidades](cognitive-search-skill-entity-recognition.md):

```json
    {
      "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
      "context": "/document",
      "categories": [ "Organization" ],
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
    }
```

* Cada aptitud integrada tiene `odata.type`, `input`, y `output` propiedades. Las propiedades específicas de la aptitud proporcionan información adicional aplicable a dicha aptitud. Para el reconocimiento de entidades, `categories` es una entidad entre un conjunto fijo de tipos de entidad que el modelo previamente entrenado puede reconocer.

* Cada aptitud debe tener un ```"context"```. El contexto representa el nivel en el que tienen lugar las operaciones. En la aptitud anterior, el contexto es el documento completo, lo que significa que la habilidad de reconocimiento de entidades se llama una vez por documento. Las salidas también se producen en ese nivel. Más específicamente, ```"organizations"``` se generan como un miembro de ```"/document"```. En las aptitudes de bajada, puede hacer referencia a esta información recién creada como ```"/document/organizations"```.  Si el campo ```"context"``` no se establece explícitamente, el contexto predeterminado es el documento.

* La aptitud tiene una entrada denominada "text", con una entrada de origen establecida en ```"/document/content"```. La aptitud (reconocimiento de entidades) que opera en el *contenido* campo de cada documento, que es un campo estándar creados por el indexador de blobs de Azure. 

* La aptitud tiene una salida denominada ```"organizations"```. Salidas existen únicamente durante el procesamiento. Para encadenar esta salida a la entrada de una aptitud de bajada, haga referencia a la salida como ```"/document/organizations"```.

* Para un documento determinado, el valor de ```"/document/organizations"``` es una matriz de las organizaciones extraídas del texto. Por ejemplo:

  ```json
  ["Microsoft", "LinkedIn"]
  ```

Algunas situaciones requieren que se haga referencia a cada elemento de una matriz por separado. Por ejemplo, imagine que desea pasar cada elemento de ```"/document/organizations"``` por separado a otra aptitud (por ejemplo, el enriquecedor de Entity Search de Bing personalizado). Puede hacer referencia a cada elemento de la matriz agregando un asterisco a la ruta de acceso: ```"/document/organizations/*"``` 

La segunda aptitud para la extracción de opiniones sigue el mismo patrón que el primer enriquecedor. Toma ```"/document/content"``` como entrada y devuelve una puntuación de opinión para cada instancia del contenido. Puesto que no estableció el campo ```"context"``` explícitamente, la salida (mySentiment) ahora es un elemento secundario de ```"/document"```.

```json
    {
      "@odata.type": "#Microsoft.Skills.Text.SentimentSkill",
      "inputs": [
        {
          "name": "text",
          "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "score",
          "targetName": "mySentiment"
        }
      ]
    },
```

## <a name="add-a-custom-skill"></a>Incorporación de una aptitud personalizada

Recupere la estructura del enriquecedor de Entity Search de Bing personalizado:

```json
    {
      "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
     "description": "This skill calls an Azure function, which in turn calls Bing Entity Search",
      "uri": "https://indexer-e2e-webskill.azurewebsites.net/api/InvokeTextAnalyticsV3?code=foo",
      "httpHeaders": {
          "Ocp-Apim-Subscription-Key": "foobar"
      }
      "context": "/document/organizations/*",
      "inputs": [
        {
          "name": "query",
          "source": "/document/organizations/*"
        }
      ],
      "outputs": [
        {
          "name": "description",
          "targetName": "companyDescription"
        }
      ]
    }
```

Esta definición es una [aptitud personalizada](cognitive-search-custom-skill-web-api.md) que llama a una API web como parte del proceso de enriquecimiento. Para cada organización identificada por el reconocimiento de entidades, este aptitud llama a una API web para buscar la descripción de la organización. La orquestación de cuándo llamar a la API web y cómo pasar la información recibida se controla internamente mediante el motor de enriquecimiento. Sin embargo, se debe proporcionar la inicialización necesaria para llamar a esta API personalizada en JSON (por ejemplo, el identificador uri, httpHeaders y las entradas esperadas). Para obtener instrucciones para crear una API web personalizada para la canalización de enriquecimiento, consulte [How to define a custom interface](cognitive-search-custom-skill-interface.md) (Definición de una interfaz personalizada).

Tenga en cuenta que el campo "context" está establecido en ```"/document/organizations/*"``` con un asterisco, lo que significa que se llama al paso de enriquecimiento *para cada* organización bajo ```"/document/organizations"```. 

La salida, en este caso una descripción de la compañía, se genera para cada organización identificada. Cuando se hace referencia a la descripción en un paso de bajada (por ejemplo, en la extracción de frases clave), se usaría la ruta de acceso ```"/document/organizations/*/description"``` para hacerlo. 

## <a name="add-structure"></a>Agregue estructura

El conjunto de aptitudes genera información estructurada a partir de datos no estructurados. Considere el ejemplo siguiente:

*"En el cuarto trimestre, Microsoft registró unos ingresos por valor de 1100 millones de dólares de LinkedIn, la compañía de red social que compró el año pasado. La adquisición permite a Microsoft combinar las funcionalidades de LinkedIn con su CRM y funcionalidades de Office. Los accionistas están satisfechos con el progreso hasta ahora.".*

Una salida probable sería una estructura generada similar a la siguiente ilustración:

![Estructura de salida de ejemplo](media/cognitive-search-defining-skillset/enriched-doc.png "Sample output structure")

Hasta ahora, ha sido esta estructura solo para uso interno, sólo en memoria y sólo se utilizan en los índices de búsqueda de Azure. La adición de un almacén de información proporciona una manera de guardar enriquecimientos de forma para su uso fuera de la búsqueda.

## <a name="add-a-knowledge-store"></a>Agregar un almacén de conocimiento

[Almacén de conocimiento](knowledge-store-concept-intro.md) es una característica de vista previa de Azure Search para guardar el documento enriquecido. Un almacén de conocimiento que cree, respaldado por una cuenta de almacenamiento de Azure, es el repositorio donde llega los datos enriquecidos. 

Se agrega una definición de la tienda de conocimiento a un conjunto de habilidades. Para ver un tutorial de todo el proceso, consulte [cómo empezar a trabajar con el almacén de conocimiento](knowledge-store-howto.md).

```json
"knowledgeStore": {
  "storageConnectionString": "<an Azure storage connection string>",
  "projections" : [
    {
      "tables": [ ]
    },
    {
      "objects": [
        {
          "storageContainer": "containername",
          "source": "/document/EnrichedShape/",
          "key": "/document/Id"
        }
      ]
    }
  ]
}
```

Puede elegir guardar los documentos enriquecidos como tablas con relaciones jerárquicas conservadas o como documentos JSON en blob storage. Salida de cualquiera de las habilidades en el conjunto de habilidades puede proceder como entrada para la proyección. Si desea para proyectar los datos en una determinada forma, la actualización [aptitud conformador](cognitive-search-skill-shaper.md) ahora se puede modelar tipos complejos para su uso. 

<a name="next-step"></a>

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con la canalización de enriquecimiento y los conjuntos de aptitudes, continúe con [How to reference annotations in a skillset](cognitive-search-concept-annotations-syntax.md) (Referencia a las anotaciones en un conjunto de aptitudes) o [How to map outputs to fields in an index](cognitive-search-output-field-mapping.md) (Asignación de salidas a campos en un índice). 
