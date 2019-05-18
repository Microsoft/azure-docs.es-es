---
title: 'Aptitud de búsqueda cognitiva Conformador: Azure Search'
description: Extraiga metadatos e información estructurada a partir de datos no estructurados y confórmelos como un tipo complejo en una canalización de enriquecimiento de Azure Search.
services: search
manager: pablocas
author: luiscabrer
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.date: 05/02/2019
ms.author: luisca
ms.custom: seodec2018
ms.openlocfilehash: 058b6c979346d9dcce36940432d0e222e919dba9
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540821"
---
#   <a name="shaper-cognitive-skill"></a>Habilidad cognitiva Conformador

El **conformador** aptitud consolida varias entradas en un [tipo complejo](search-howto-complex-data-types.md) que puede hacer referencia a más adelante en la canalización de enriquecimiento. La aptitud **Conformador** básicamente permite crear una estructura, definir el nombre de los miembros de esa estructura y asignar valores a cada miembro. Algunos ejemplos de campos consolidados útiles en escenarios de búsqueda son la combinación de un nombre y apellido en una única estructura, ciudad y estado en una única estructura o el nombre y fecha de nacimiento en una única estructura para establecer la identidad única.

La versión de API determina la profundidad de forma puede lograr. 

| Versión de API | Dar forma a los comportamientos | 
|-------------|-------------------|
| Versión de 2019-05-06-versión preliminar de la API de REST (no se admite el SDK. NET) | Los objetos complejos, múltiples niveles de profundidad, uno **conformador** definición de habilidades. |
| 2019-05-06 ** (disponible con carácter general), 2017-11-11-Preview| Objetos complejos, un nivel de profundidad. Una forma de varios nivel requiere encadenar juntos varios pasos de modelado.|

Tal como lo proporciona `api-version=2019-05-06-Preview`, el **conformador** aptitud se muestra en [escenario 3](#nested-complex-types) agrega un nuevo opcional *sourceContext* propiedad a la entrada. El *origen* y *sourceContext* propiedades son mutuamente excluyentes. Si la entrada está en el contexto de la aptitud, basta con usar *origen*. Si la entrada está en un *diferentes* contexto que el contexto de habilidades, use el *sourceContext*. El *sourceContext* , deberá definir una entrada anidada con el elemento específico abordado como el origen. 

En la respuesta, para todas las versiones de API, el nombre de salida es siempre "salido". Internamente, la canalización puede asignar un nombre diferente, como "analyzedText" como se muestra en los ejemplos siguientes, pero la **conformador** aptitud propio devuelve "output" en la respuesta. Esto podría ser importante si está depurando documentos enriquecidos y observa la discrepancia de nombre, o si compila una habilidad personalizada y estructura la respuesta por sí mismo.

> [!NOTE]
> El **conformador** aptitud no está enlazada a una API de Cognitive Services y no se le cobrará para usarlo. Sin embargo, debe [adjuntar un recurso de Cognitive Services](cognitive-search-attach-cognitive-services.md) para invalidar la opción del recurso **Gratis**, que tiene un límite de unos pocos enriquecimientos al día.

## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Util.ShaperSkill

## <a name="scenario-1-complex-types"></a>Escenario 1: los tipos complejos

Piense en un escenario en el que quiere crear una estructura denominada *analyzedText* con dos miembros: *text* y *sentiment*, respectivamente. En un índice de Azure Search, se llama a un campo de búsqueda de varias partes un *tipo complejo* y a menudo se crea cuando el origen de datos tiene una estructura compleja correspondiente que se asigna a ella.

Sin embargo, es otro enfoque para la creación de tipos complejos a través de la **conformador** aptitud. Al incluir esta aptitud en un conjunto de habilidades, las operaciones en memoria durante el procesamiento de conjunto de habilidades pueden generar las formas de datos con estructuras anidadas, que pueden asignarse a un tipo complejo en el índice. 

La siguiente definición de la habilidad de ejemplo proporciona al miembro nombres como entrada. 


```json
{
  "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
  "context": "/document/content/phrases/*",
  "inputs": [
    {
      "name": "text",
      "source": "/document/content/phrases/*"
    },
    {
      "name": "sentiment",
      "source": "/document/content/phrases/*/sentiment"
    }
  ],
  "outputs": [
    {
      "name": "output",
      "targetName": "analyzedText"
    }
  ]
}
```

### <a name="sample-index"></a>Índice de ejemplo

Un indizador que se invoca un conjunto de habilidades y un indizador que requiere un índice. Una representación compleja de campo del índice podría ser similar al ejemplo siguiente. 

```json

    "name": "my-index",
    "fields": [
        {   "name": "myId", "type": "Edm.String", "key": true, "filterable": true   },
        {   "name": "analyzedText", "type": "Edm.ComplexType",
            "fields": [{
                    "name": "text",
                    "type": "Edm.String",
                    "filterable": false,
                    "sortable": false,
                    "facetable": false,
                    "searchable": true  },
          {
                    "name": "sentiment",
                    "type": "Edm.Double",
                    "searchable": true,
                    "filterable": true,
                    "sortable": true,
                    "facetable": true
                },
```

### <a name="skill-input"></a>Entrada de conocimientos

Un documento JSON entrante que proporciona la entrada puede usar para este **conformador** aptitud podría ser:

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "text": "this movie is awesome",
                "sentiment": 0.9
            }
        }
    ]
}
```


### <a name="skill-output"></a>Salida de conocimientos

La aptitud *Conformador* genera un nuevo elemento llamado **analyzedText** con los elementos combinados de *text* y *sentiment*. Esta salida se ajusta al esquema de índice. Se importarán e indizada en un índice de Azure Search.

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
            "analyzedText": 
              {
                "text": "this movie is awesome" ,
                "sentiment": 0.9
              }
           }
      }
    ]
}
```

## <a name="scenario-2-input-consolidation"></a>Escenario 2: entrada consolidación

En otro ejemplo, imagine que en distintas fases del procesamiento de la canalización ha extraído el título de un libro y los títulos de capítulos de distintas páginas del libro. Ahora podría crear una única estructura compuesta por estas distintas entradas.

El **conformador** definición de habilidades para este escenario podría ser similar al ejemplo siguiente:

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "context": "/document",
    "inputs": [
        {
            "name": "title",
            "source": "/document/content/title"
        },
        {
            "name": "chapterTitles",
            "source": "/document/content/pages/*/chapterTitles/*/title"
        }
    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "titlesAndChapters"
        }
    ]
}
```

### <a name="skill-output"></a>Salida de conocimientos
En este caso, el **conformador** aplana todos los títulos de capítulos para crear una sola matriz. 

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "titlesAndChapters": {
                    "title": "How to be happy",
                    "chapterTitles": [
                        "Start young",
                        "Laugh often",
                        "Eat, sleep and exercise"
                    ]
                }
            }
        }
    ]
}
```

<a name="nested-complex-types"></a>

## <a name="scenario-3-input-consolidation-from-nested-contexts"></a>Escenario 3: consolidación de entrada de contextos anidados

> [!NOTE]
> Anidar estructuras que admiten la [API de REST versión 2019-05-06-Preview](search-api-preview.md) puede usarse en un [almacén knowledge](knowledge-store-concept-intro.md) o en un índice de Azure Search.

Imagine que tiene el título, capítulos y contenido de un libro y ha ejecutado la entidad frases de reconocimiento y la clave en el contenido y ahora debe agregar los resultados de las habilidades diferentes en una sola forma con el nombre de capítulo, entidades y las frases clave.

El **conformador** definición de habilidades para este escenario podría ser similar al ejemplo siguiente:

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
    "context": "/document",
    "inputs": [
        {
            "name": "title",
            "source": "/document/content/title"
        },
        {
            "name": "chapterTitles",
            "sourceContext": "/document/content/pages/*/chapterTitles/*",
            "inputs": [
              {
                  "name": "title",
                  "source": "/document/content/pages/*/chapterTitles/*/title"
              },
              {
                  "name": "number",
                  "source": "/document/content/pages/*/chapterTitles/*/number"
              }
            ]
        }

    ],
    "outputs": [
        {
            "name": "output",
            "targetName": "titlesAndChapters"
        }
    ]
}
```

### <a name="skill-output"></a>Salida de conocimientos
En este caso, el **conformador** crea un tipo complejo. Esta estructura existe en memoria. Si desea guardarla en un almacén de conocimiento, debe crear una proyección en el conjunto de habilidades que define las características de almacenamiento.

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "titlesAndChapters": {
                    "title": "How to be happy",
                    "chapterTitles": [
                      { "title": "Start young", "number": 1},
                      { "title": "Laugh often", "number": 2},
                      { "title": "Eat, sleep and exercise", "number: 3}
                    ]
                }
            }
        }
    ]
}
```

## <a name="see-also"></a>Vea también

+ [Aptitudes predefinidas](cognitive-search-predefined-skills.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Uso de tipos complejos](search-howto-complex-data-types.md)
+ [Información general de la tienda de conocimiento](knowledge-store-concept-intro.md)
+ [Cómo empezar a trabajar con el almacén de información](knowledge-store-howto.md)