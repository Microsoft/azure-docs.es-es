---
title: Aptitud de búsqueda de entidades personalizadas de Cognitive Search
titleSuffix: Azure Cognitive Search
description: Extraiga diferentes entidades personalizadas del texto de una canalización de búsqueda cognitiva de Azure Cognitive Search. Esta aptitud está actualmente en versión preliminar pública.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: 5511551f240fe4fdd2f2aa3bc8a3a2615505f35f
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936119"
---
#     <a name="custom-entity-lookup-cognitive-skill-preview"></a>Aptitud cognitiva de búsqueda de entidades personalizadas (versión preliminar)

> [!IMPORTANT] 
> Esta aptitud está actualmente en versión preliminar pública. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Actualmente no hay compatibilidad con el portal ni con el SDK de .NET.

La aptitud de **búsqueda de entidades personalizadas** busca texto de una lista de palabras y frases personalizada definida por el usuario. Con esta lista, se etiquetan todos los documentos con entidades coincidentes. La aptitud también admite un grado de coincidencia aproximada que se puede aplicar para buscar coincidencias similares pero no exactas.  

Esta aptitud no está enlazada a una API de Cognitive Services y se puede usar de forma gratuita durante el período de versión preliminar. No obstante, debe [asociar un recurso de Cognitive Services](./cognitive-search-attach-cognitive-services.md) para invalidar el límite de enriquecimiento diario. El límite diario se aplica al acceso gratuito a Cognitive Services cuando se accede a él mediante Azure Cognitive Search.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.CustomEntityLookupSkill 

## <a name="data-limits"></a>Límites de datos
+ El tamaño máximo de registro de entrada admitido es 256 MB. Si necesita desglosar los datos antes de enviarlos a la aptitud de búsqueda de entidades personalizadas, puede usar la [aptitud de división de texto](cognitive-search-skill-textsplit.md).
+ La tabla de definiciones de entidades máxima que se admite es de 10 MB si se proporciona mediante el parámetro *entitiesDefitionUri*. 
+ Si las entidades se definen en línea, con el parámetro *inlineEntitiesDefinition*, el tamaño máximo admitido es de 10 KB.

## <a name="skill-parameters"></a>Parámetros de la aptitud

Los parámetros distinguen mayúsculas de minúsculas.

| Nombre de parámetro     | Descripción |
|--------------------|-------------|
| `entitiesDefinitionUri`    | Ruta de acceso a un archivo JSON o CSV que contiene todo el texto de destino con el que debe coincidir. Esta definición de entidad se lee al principio de una ejecución del indexador; las actualizaciones en medio de la ejecución de este archivo no se realizarán hasta posteriores ejecuciones. Esta configuración debe ser accesible mediante HTTPS. Consulte [Formato de definición de entidad personalizada](#custom-entity-definition-format) que aparece más adelante para ver el esquema CSV o JSON esperado.|
|`inlineEntitiesDefinition` | Definiciones de entidades JSON insertadas. Este parámetro reemplaza al parámetro entitiesDefinitionUri si existe. No se pueden proporcionar más de 10 KB de configuración en línea. Consulte [Definición de entidad personalizada](#custom-entity-definition-format) que aparece más adelante para ver el esquema JSON esperado. |
|`defaultLanguageCode` |    (Opcional) Código de idioma del texto de entrada que se utiliza para dividir en tokens y definir el texto de entrada. Se admiten los siguientes idiomas: `da, de, en, es, fi, fr, it, ko, pt`. El idioma predeterminado es inglés (`en`). Si se pasa un formato “código de idioma-código de país”, solo se usa la parte “código de idioma” del formato.  |


## <a name="skill-inputs"></a>Entradas de la aptitud

| Nombre de entrada      | Descripción                   |
|---------------|-------------------------------|
| `text`          | Texto que se analizará.          |
| `languageCode`    | Opcional. El valor predeterminado es `"en"`.  |


## <a name="skill-outputs"></a>Salidas de la aptitud


| Nombre de salida      | Descripción                   |
|---------------|-------------------------------|
| `entities` | Matriz de objetos que contiene información sobre las coincidencias encontradas y los metadatos relacionados. Cada una de las entidades identificadas puede contener los campos siguientes:  <ul> <li> *name*: La entidad de nivel superior identificada. La entidad representa el formulario "normalizado". </li> <li> *id*:  Un identificador único para la entidad tal y como lo define el usuario en el "formato de definición de entidad personalizada".</li> <li> *description*: Descripción de la entidad tal y como la define el usuario en el "formato de definición de entidad personalizada". </li> <li> *type:* Tipo de entidad tal y como la define el usuario en el "formato de definición de entidad personalizada".</li> <li> *subtype:* Subtipo de entidad tal y como la define el usuario en el "formato de definición de entidad personalizada".</li>  <li> *matches*: Colección que describe cada una de las coincidencias para esa entidad en el texto de origen. Cada coincidencia tendrá los siguientes miembros: </li> <ul> <li> *text*: Coincidencia de texto sin formato del documento de origen. </li> <li> *offset*: La ubicación en la que se encontró la coincidencia en el texto. </li> <li> *length*:  La longitud del texto coincidente. </li> <li> *matchDistance*: El número de caracteres que diferencia a esta coincidencia del nombre o alias de la entidad original.  </li> </ul> </ul>
  |

## <a name="custom-entity-definition-format"></a>Formato de definición de entidad personalizada

Hay tres formas diferentes de proporcionar la lista de entidades personalizadas a la aptitud de búsqueda de entidades personalizadas. Puede proporcionar la lista en un archivo .CSV, un archivo .JSON o como una definición insertada como parte de la definición de la aptitud.  

Si el archivo de definición es un archivo CSV o JSON, la ruta de acceso del archivo se debe proporcionar como parte del parámetro *entitiesDefitionUri*. En este caso, el archivo se descarga una vez al principio de cada ejecución del indexador. El archivo debe ser accesible siempre que se pretenda ejecutar el indexador. Además, el archivo debe estar codificado como UTF-8.

Si la definición se proporciona insertada, debe proporcionarse como insertada como contenido del parámetro *inlineEntitiesDefinition* de la aptitud. 

### <a name="csv-format"></a>Formato CSV

Puede proporcionar la definición de las entidades personalizadas que se van a buscar en un archivo de valores separados por comas (CSV); para ello, especifique la ruta de acceso al archivo y configúrelo en el parámetro *entitiesDefinitionUri* de la aptitud. La ruta de acceso debe estar en una ubicación https. El archivo de definición puede tener un tamaño máximo de 10 MB.

El formato CSV es sencillo. Cada línea representa una entidad única, como se muestra a continuación:

```
Bill Gates, BillG, William H. Gates
Microsoft, MSFT
Satya Nadella 
```

En este caso, hay tres entidades que se pueden devolver como entidades encontradas (Bill Gates, Satya Nadella, Microsoft), pero se identificarán si alguno de los términos de la línea (alias) coincide con el texto. Por ejemplo, si se encuentra la cadena "William H. Gates" en un documento, se devolverá una coincidencia para la entidad "Bill Gates".

### <a name="json-format"></a>Formato JSON

También puede proporcionar la definición de las entidades personalizadas que desea buscar mediante un archivo JSON. El formato JSON proporciona un poco más de flexibilidad, ya que permite definir reglas de coincidencia por término. Por ejemplo, puede especificar la distancia de las coincidencias aproximadas (distancia Damerau-Levenshtein) para cada término o si la coincidencia debe distinguir entre mayúsculas y minúsculas o no. 

 Al igual que con los archivos CSV, debe proporcionar la ruta de acceso al archivo JSON y configurarlo en el parámetro *entitiesDefitionUri* de la aptitud. La ruta de acceso debe estar en una ubicación https. El archivo de definición puede tener un tamaño máximo de 10 MB.

La definición de lista de entidades personalizadas JSON más básica puede ser una lista de entidades que deben coincidir:

```json
[ 
    { 
        "name" : "Bill Gates"
    }, 
    { 
        "name" : "Microsoft"
    }, 
    { 
        "name" : "Satya Nadella"
    }
]
```

Un ejemplo más complejo de una definición JSON puede proporcionar opcionalmente el identificador, la descripción, el tipo y el subtipo de cada entidad, así como otros *alias*. Si se encuentra una coincidencia con un término de alias, la entidad se devolverá también:

```json
[ 
    { 
        "name" : "Bill Gates",
        "description" : "Microsoft founder." ,
        "aliases" : [ 
            { "text" : "William H. Gates", "caseSensitive" : false },
            { "text" : "BillG", "caseSensitive" : true }
        ]
    }, 
    { 
        "name" : "Xbox One", 
        "type": "Harware",
        "subtype" : "Gaming Device",
        "id" : "4e36bf9d-5550-4396-8647-8e43d7564a76",
        "description" : "The Xbox One product"
    }, 
    { 
        "name" : "LinkedIn" , 
        "description" : "The LinkedIn company", 
        "id" : "differentIdentifyingScheme123", 
        "fuzzyEditDistance" : 0 
    }, 
    { 
        "name" : "Microsoft" , 
        "description" : "Microsoft Corporation", 
        "id" : "differentIdentifyingScheme987", 
        "defaultCaseSensitive" : false, 
        "defaultFuzzyEditDistance" : 1, 
        "aliases" : [ 
            { "text" : "MSFT", "caseSensitive" : true }
        ]
    } 
] 
```

En las tablas siguientes se describe más detalladamente los distintos parámetros de configuración que puede establecer al definir las entidades que deben coincidir:

|  Nombre del campo  |        Descripción  |
|--------------|----------------------|
| `name` | El descriptor de entidad de nivel superior. Las coincidencias en la salida de la aptitud se agruparán por este nombre y este debe representar la forma "normalizada" del texto que se encuentra.  |
| `description`  | (Opcional) Este campo se puede usar como paso a través de los metadatos personalizados acerca de los textos coincidentes. El valor de este campo aparecerá con cada coincidencia de su entidad en la salida de la aptitud. |
| `type` | (Opcional) Este campo se puede usar como paso a través de los metadatos personalizados acerca de los textos coincidentes. El valor de este campo aparecerá con cada coincidencia de su entidad en la salida de la aptitud. |
| `subtype` | (Opcional) Este campo se puede usar como paso a través de los metadatos personalizados acerca de los textos coincidentes. El valor de este campo aparecerá con cada coincidencia de su entidad en la salida de la aptitud. |
| `id` | (Opcional) Este campo se puede usar como paso a través de los metadatos personalizados acerca de los textos coincidentes. El valor de este campo aparecerá con cada coincidencia de su entidad en la salida de la aptitud. |
| `caseSensitive` | (Opcional) El valor predeterminado es "false". Valor booleano que indica si las comparaciones con el nombre de la entidad deben distinguir entre mayúsculas y minúsculas. Las coincidencias de "Microsoft" de ejemplo que no distinguen mayúsculas de minúsculas podrían ser: microsoft, microSoft, MICROSOFT |
| `fuzzyEditDistance` | (Opcional) El valor predeterminado es 0. El valor máximo es 5. Indica el número aceptable de caracteres diferentes que seguirán constituyendo una coincidencia con el nombre de la entidad. Se devuelve el valor mínimo de coincidencia aproximada posible para cualquier coincidencia determinada.  Por ejemplo, si la distancia de edición se establece en 3, "Windows 10" todavía coincidiría con "Windows", "Windows10" y "Windows 7". <br/> Si la distinción entre mayúsculas y minúsculas está establecida en false, las diferencias entre mayúsculas y minúsculas NO cuentan para la tolerancia con la coincidencia aproximada. En caso contrario, sí lo hacen. |
| `defaultCaseSensitive` | (Opcional) Cambia el valor predeterminado de distinción entre mayúsculas y minúsculas para esta entidad. Se utiliza para cambiar el valor predeterminado de todos los valores caseSensitive de los alias. |
| `defaultFuzzyEditDistance` | (Opcional) Cambia el valor predeterminado de la distancia de edición de la coincidencia aproximada para esta entidad. Se puede utilizar para cambiar el valor predeterminado de todos los valores fuzzyEditDistance de los alias. |
| `aliases` | (Opcional) Matriz de objetos complejos que se pueden usar para especificar ortografías o sinónimos alternativos al nombre de la entidad raíz. |

| Propiedades de alias | Descripción |
|------------------|-------------|
| `text`  | Ortografía o representación alternativa de algún nombre de entidad de destino.  |
| `caseSensitive` | (Opcional) Actúa igual que el parámetro "caseSensitive" de la entidad raíz anterior, pero solo se aplica a este alias. |
| `fuzzyEditDistance` | (Opcional) Actúa igual que el parámetro "fuzzyEditDistance" de la entidad raíz anterior, pero solo se aplica a este alias. |


### <a name="inline-format"></a>Formato en línea

En algunos casos, puede que resulte más conveniente proporcionar la lista de entidades personalizadas para que coincidan directamente en línea con la definición de la aptitud. En ese caso, puede usar un formato JSON similar al descrito anteriormente, pero inserto en la definición de la aptitud.
Solo las configuraciones que tienen un tamaño inferior a 10 KB (tamaño serializado) se pueden definir en línea. 

##    <a name="sample-definition"></a>Definición de ejemplo

A continuación se muestra un ejemplo de definición de aptitud con un formato en línea:

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.CustomEntityLookupSkill",
    "context": "/document",
    "inlineEntitiesDefinition": 
    [
      { 
        "name" : "Bill Gates",
        "description" : "Microsoft founder." ,
        "aliases" : [ 
            { "text" : "William H. Gates", "caseSensitive" : false },
            { "text" : "BillG", "caseSensitive" : true }
        ]
      }, 
      { 
        "name" : "Xbox One", 
        "type": "Hardware",
        "subtype" : "Gaming Device",
        "id" : "4e36bf9d-5550-4396-8647-8e43d7564a76",
        "description" : "The Xbox One product"
      }
    ],    
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "entities",
        "targetName": "matchedEntities"
      }
    ]
  }
```
Si decide proporcionar un puntero al archivo de definición de entidades, también puede ver a continuación una definición de aptitud de ejemplo con el formato `entitiesDefinitionUri`:

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.CustomEntityLookupSkill",
    "context": "/document",
    "entitiesDefinitionUri": "https://myblobhost.net/keyWordsConfig.csv",    
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "entities",
        "targetName": "matchedEntities"
      }
    ]
  }

```

##    <a name="sample-input"></a>Entrada de ejemplo

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "text": "The company, Microsoft, was founded by Bill Gates. Microsoft's gaming console is called Xbox",
             "languageCode": "en"
           }
      }
    ]
}
```

##    <a name="sample-output"></a>Salida de ejemplo

```json
  { 
    "values" : 
    [ 
      { 
        "recordId": "1", 
        "data" : { 
          "entities": [
            { 
              "name" : "Microsoft", 
              "description" : "This document refers to Microsoft the company", 
              "id" : "differentIdentifyingScheme987", 
              "matches" : [ 
                { 
                  "text" : "microsoft", 
                  "offset" : 13, 
                  "length" : 9, 
                  "matchDistance" : 0 
                }, 
                { 
                  "text" : "Microsoft",
                  "offset" : 49, 
                  "length" : 9, 
                  "matchDistance" : 0
                }
              ] 
            },
            { 
              "name" : "Bill Gates",
              "description" : "William Henry Gates III, founder of Microsoft.", 
              "matches" : [
                { 
                  "text" : "Bill Gates",
                  "offset" : 37, 
                  "length" : 10,
                  "matchDistance" : 0 
                }
              ]
            }
          ] 
        } 
      } 
    ] 
  } 
```

## <a name="errors-and-warnings"></a>Errores y advertencias

### <a name="warning-reached-maximum-capacity-for-matches-skipping-all-further-duplicate-matches"></a>Advertencia: Se alcanzó la capacidad máxima de coincidencias; se omitirán todas las coincidencias duplicadas.

Esta advertencia se emitirá si el número de coincidencias detectadas es mayor que el máximo permitido. En este caso, se dejarán de incluir coincidencias duplicadas. Si esto no le resulta aceptable, envíe una [incidencia de soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support) para que podamos ayudarle con su caso de uso individual.

## <a name="see-also"></a>Consulte también

+ [Aptitudes integradas](cognitive-search-predefined-skills.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Aptitud de reconocimiento de entidades (para buscar entidades conocidas)](cognitive-search-skill-entity-recognition.md)