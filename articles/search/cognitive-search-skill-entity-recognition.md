---
title: Aptitud cognitiva de reconocimiento de entidades
titleSuffix: Azure Cognitive Search
description: Extraiga diferentes tipos de entidades del texto de una canalización de enriquecimiento de Búsqueda cognitiva de Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/17/2020
ms.openlocfilehash: fd35f297e88c37aec39938b0bfd60288e591a62c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936085"
---
#    <a name="entity-recognition-cognitive-skill"></a>Aptitud cognitiva de reconocimiento de entidades

La aptitud **Reconocimiento de entidades** extrae entidades de distintos tipos del texto. Esta aptitud utiliza los modelos de aprendizaje automático proporcionados por [Text Analytics](../cognitive-services/text-analytics/overview.md) en Cognitive Services.

> [!NOTE]
> A medida que expanda el ámbito aumentando la frecuencia de procesamiento, agregando más documentos o agregando más algoritmos de IA, tendrá que [asociar un recurso facturable de Cognitive Services](cognitive-search-attach-cognitive-services.md). Los cargos se acumulan cuando se llama a las API de Cognitive Services y por la extracción de imágenes como parte de la fase de descifrado de documentos de Azure Cognitive Search. No hay ningún cargo por la extracción de texto de documentos.
>
> La ejecución de aptitudes integradas se cobra según los [precios de pago por uso de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/) existentes. Los precios de la extracción de imágenes se describen en la [página de precios de Búsqueda cognitiva de Azure](https://azure.microsoft.com/pricing/details/search/).


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.EntityRecognitionSkill

## <a name="data-limits"></a>Límites de datos
El tamaño máximo de un registro debe tener menos de 50 000 caracteres según la medición de [`String.Length`](/dotnet/api/system.string.length). Si tiene que dividir los datos antes de enviarlos al extractor de frases clave, puede usar la [aptitud de división de texto](cognitive-search-skill-textsplit.md).

## <a name="skill-parameters"></a>Parámetros de la aptitud

Los parámetros distinguen mayúsculas de minúsculas y son opcionales.

| Nombre de parámetro     | Descripción |
|--------------------|-------------|
| `categories`    | Matriz de categorías que se deben extraer.  Tipos de categorías posibles: `"Person"`, `"Location"`, `"Organization"`, `"Quantity"`, `"Datetime"`, `"URL"`, `"Email"`. Si no se proporciona ninguna categoría, se devuelven todos los tipos.|
| `defaultLanguageCode` |    Código de idioma del texto de entrada. Se admiten los siguientes idiomas: `ar, cs, da, de, en, es, fi, fr, hu, it, ja, ko, nl, no, pl, pt-BR, pt-PT, ru, sv, tr, zh-hans`. No todas las categorías de entidad son compatibles con todos los idiomas. Consulte la nota que aparece a continuación.|
| `minimumPrecision` | Un valor entre 0 y 1. Si la puntuación de confianza (en el resultado `namedEntities`) es inferior a este valor, no se devuelve la entidad. El valor predeterminado es 0. |
| `includeTypelessEntities` | Establézcalo en `true` si desea reconocer entidades conocidas que no se ajustan a las categorías actuales. Las entidades reconocidas se devuelven en el campo de salida complejo `entities`. Por ejemplo, "Windows 10" es una entidad conocida (un producto), pero como "Productos" no es una categoría admitida, esta entidad se incluiría en el campo de salida de las entidades. Valor predeterminado: `false` |


## <a name="skill-inputs"></a>Entradas de la aptitud

| Nombre de entrada      | Descripción                   |
|---------------|-------------------------------|
| `languageCode`    | Opcional. El valor predeterminado es `"en"`.  |
| `text`          | Texto que se analizará.          |

## <a name="skill-outputs"></a>Salidas de la aptitud

> [!NOTE]
> No todas las categorías de entidad son compatibles con todos los idiomas. Los tipos de categoría de entidad `"Person"`, `"Location"` y `"Organization"` se admiten para todos los idiomas anteriores. Solo _de_, _en_, _es_, _fr_ y _zh-hans_ admiten la extracción de los tipos `"Quantity"`, `"Datetime"`, `"URL"` y `"Email"`. Para más información, consulte [Compatibilidad de idiomas y regiones para Text Analytics API](../cognitive-services/text-analytics/language-support.md).  

| Nombre de salida      | Descripción                   |
|---------------|-------------------------------|
| `persons`       | Una matriz de cadenas donde cada cadena representa el nombre de una persona. |
| `locations`  | Una matriz de cadenas donde cada cadena representa una ubicación. |
| `organizations`  | Una matriz de cadenas donde cada cadena representa una organización. |
| `quantities`  | Una matriz de cadenas donde cada cadena representa una cantidad. |
| `dateTimes`  | Una matriz de cadenas donde cada cadena representa un valor de fecha y hora (como aparece en el texto). |
| `urls` | Una matriz de cadenas donde cada cadena representa una dirección URL. |
| `emails` | Una matriz de cadenas donde cada cadena representa un correo electrónico. |
| `namedEntities` | Una matriz de tipos complejos, que contiene los siguientes campos: <ul><li>category</li> <li>valor (el nombre de entidad real)</li><li>desplazamiento (la ubicación donde se encontró en el texto)</li><li>confianza (cuanto más alto sea el valor, más real será una entidad)</li></ul> |
| `entities` | Una matriz de tipos complejos que contiene información valiosa acerca de las entidades extraídas del texto, con los siguientes campos: <ul><li> name (el nombre real de la entidad. Esto representa una forma "normalizada")</li><li> wikipediaId</li><li>wikipediaLanguage</li><li>wikipediaUrl (un vínculo a la página de Wikipedia de la entidad)</li><li>bingId</li><li>type (la categoría de la entidad reconocida)</li><li>subType (solo disponible para determinadas categorías; proporciona una vista más pormenorizada del tipo de entidad)</li><li> matches (una colección compleja que contiene)<ul><li>text (el texto sin formato de la entidad)</li><li>offset (la ubicación donde se encontró)</li><li>length (la longitud del texto de la entidad sin formato)</li></ul></li></ul> |

##    <a name="sample-definition"></a>Definición de ejemplo

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
    "categories": [ "Person", "Email"],
    "defaultLanguageCode": "en",
    "includeTypelessEntities": true,
    "minimumPrecision": 0.5,
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      }
    ],
    "outputs": [
      {
        "name": "persons",
        "targetName": "people"
      },
      {
        "name": "emails",
        "targetName": "contact"
      },
      {
        "name": "entities"
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
             "text": "Contoso corporation was founded by John Smith. They can be reached at contact@contoso.com",
             "languageCode": "en"
           }
      }
    ]
}
```

##    <a name="sample-output"></a>Salida de ejemplo

```json
{
  "values": [
    {
      "recordId": "1",
      "data" : 
      {
        "persons": [ "John Smith"],
        "emails":["contact@contoso.com"],
        "namedEntities": 
        [
          {
            "category":"Person",
            "value": "John Smith",
            "offset": 35,
            "confidence": 0.98
          }
        ],
        "entities":  
        [
          {
            "name":"John Smith",
            "wikipediaId": null,
            "wikipediaLanguage": null,
            "wikipediaUrl": null,
            "bingId": null,
            "type": "Person",
            "subType": null,
            "matches": [{
                "text": "John Smith",
                "offset": 35,
                "length": 10
            }]
          },
          {
            "name": "contact@contoso.com",
            "wikipediaId": null,
            "wikipediaLanguage": null,
            "wikipediaUrl": null,
            "bingId": null,
            "type": "Email",
            "subType": null,
            "matches": [
            {
                "text": "contact@contoso.com",
                "offset": 70,
                "length": 19
            }]
          },
          {
            "name": "Contoso",
            "wikipediaId": "Contoso",
            "wikipediaLanguage": "en",
            "wikipediaUrl": "https://en.wikipedia.org/wiki/Contoso",
            "bingId": "349f014e-7a37-e619-0374-787ebb288113",
            "type": null,
            "subType": null,
            "matches": [
            {
                "text": "Contoso",
                "offset": 0,
                "length": 7
            }]
          }
        ]
      }
    }
  ]
}
```

Tenga en cuenta que los desplazamientos devueltos para las entidades en la salida de esta aptitud se devuelven directamente desde la [API de Text Analytics](../cognitive-services/text-analytics/overview.md), lo que significa que si los usa para indexar en la cadena original, debe usar la clase [StringInfo](/dotnet/api/system.globalization.stringinfo?view=netframework-4.8) en .NET para extraer el contenido correcto.  [Se pueden encontrar aquí más detalles.](../cognitive-services/text-analytics/concepts/text-offsets.md)

## <a name="error-cases"></a>Casos de error
Si no se admite el código de idioma del documento, se devuelve un error y no se extrae ninguna entidad.

## <a name="see-also"></a>Consulte también

+ [Aptitudes integradas](cognitive-search-predefined-skills.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)