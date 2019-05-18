---
title: 'Aptitud de búsqueda cognitiva de reconocimiento de entidades con nombre: Azure Search'
description: Extraiga entidades con nombre de persona, ubicación y organización del texto en una canalización de búsqueda cognitiva de Azure Search.
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
ms.openlocfilehash: e6a7d4bb7452ff8f6b3c2536a5aa100a15a6ec78
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65539289"
---
#    <a name="named-entity-recognition-cognitive-skill"></a>Aptitud cognitiva Reconocimiento de entidades con nombre

La habilidad **Reconocimiento de entidades con nombre** extrae entidades con nombre del texto. Las entidades disponibles incluyen los tipos `person`, `location` y `organization`.

> [!IMPORTANT]
> Reconocimiento de entidades con nombre ya no está disponible la aptitud reemplazado por [Microsoft.Skills.Text.EntityRecognitionSkill](cognitive-search-skill-entity-recognition.md). Soporte técnico se detiene en el 15 de febrero de 2019 y la API se ha quitado del producto en el 2 de mayo de 2019. Siga las recomendaciones de [en desuso habilidades de búsqueda cognitiva](cognitive-search-skill-deprecated.md) para migrar a una aptitud admitida.

> [!NOTE]
> A medida que expanda el ámbito aumentando la frecuencia de procesamiento, agregando más documentos o agregando más algoritmos de IA, tendrá que [asociar un recurso facturable de Cognitive Services](cognitive-search-attach-cognitive-services.md). Los cargos se acumulan cuando se llama a las API de Cognitive Services y para la extracción de imágenes como parte de la fase de descifrado de documentos en Azure Search. No hay ningún cargo por la extracción de texto de documentos.
>
> La ejecución de aptitudes integradas se cobra según los [precios de pago por uso de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/) existentes. Los precios de la extracción de imágenes se describen en la [página de precios de Azure Search](https://go.microsoft.com/fwlink/?linkid=2042400).


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.NamedEntityRecognitionSkill

## <a name="data-limits"></a>Límites de datos
El tamaño máximo de un registro debe tener 50 000 caracteres según lo que mida `String.Length`. Si tiene que dividir los datos antes de enviarlos al extractor de frases clave, puede usar la [aptitud de división de texto](cognitive-search-skill-textsplit.md).

## <a name="skill-parameters"></a>Parámetros de las aptitudes

Los parámetros distinguen mayúsculas de minúsculas.

| Nombre de parámetro     | DESCRIPCIÓN |
|--------------------|-------------|
| categorías    | Matriz de categorías que se deben extraer.  Tipos de categorías posibles: `"Person"`, `"Location"` y `"Organization"`. Si no se proporciona ninguna categoría, se devuelven todos los tipos.|
|defaultLanguageCode |  Código de idioma del texto de entrada. Se admiten los siguientes idiomas: `de, en, es, fr, it`|
| minimumPrecision  | Número comprendido entre 0 y 1. Si la precisión es inferior a este valor, no se devuelve la entidad. El valor predeterminado es 0.|

## <a name="skill-inputs"></a>Entradas de la aptitud

| Indicar nombre      | DESCRIPCIÓN                   |
|---------------|-------------------------------|
| languageCode  | Opcional. El valor predeterminado es `"en"`.  |
| text          | Texto que se analizará.          |

## <a name="skill-outputs"></a>Salidas de la aptitud

| Nombre de salida     | DESCRIPCIÓN                   |
|---------------|-------------------------------|
| persons      | Una matriz de cadenas donde cada cadena representa el nombre de una persona. |
| ubicaciones  | Una matriz de cadenas donde cada cadena representa una ubicación. |
| organizations  | Una matriz de cadenas donde cada cadena representa una organización. |
| entities | Una matriz de tipos complejos. Cada tipo complejo incluye los siguientes campos: <ul><li>categoría (`"person"`, `"organization"` o `"location"`)</li> <li>valor (el nombre de entidad real)</li><li>desplazamiento (la ubicación donde se encontró en el texto)</li><li>confianza (un valor entre 0 y 1 que representa esa confianza de que el valor es una entidad real)</li></ul> |

##  <a name="sample-definition"></a>Definición de ejemplo

```json
  {
    "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
    "categories": [ "Person", "Location", "Organization"],
    "defaultLanguageCode": "en",
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
      }
    ]
  }
```
##  <a name="sample-input"></a>Entrada de ejemplo

```json
{
    "values": [
      {
        "recordId": "1",
        "data":
           {
             "text": "This is the loan application for Joe Romero, he is a Microsoft employee who was born in Chile and then moved to Australia… Ana Smith is provided as a reference.",
             "languageCode": "en"
           }
      }
    ]
}
```

##  <a name="sample-output"></a>Salida de ejemplo

```json
{
  "values": [
    {
      "recordId": "1",
      "data" : 
      {
        "persons": [ "Joe Romero", "Ana Smith"],
        "locations": ["Chile", "Australia"],
        "organizations":["Microsoft"],
        "entities":  
        [
          {
            "category":"person",
            "value": "Joe Romero",
            "offset": 33,
            "confidence": 0.87
          },
          {
            "category":"person",
            "value": "Ana Smith",
            "offset": 124,
            "confidence": 0.87
          },
          {
            "category":"location",
            "value": "Chile",
            "offset": 88,
            "confidence": 0.99
          },
          {
            "category":"location",
            "value": "Australia",
            "offset": 112,
            "confidence": 0.99
          },
          {
            "category":"organization",
            "value": "Microsoft",
            "offset": 54,
            "confidence": 0.99
          }
        ]
      }
    }
  ]
}
```


## <a name="error-cases"></a>Casos de error
Si no se admite el código de idioma del documento, se devuelve un error y no se extrae ninguna entidad.

## <a name="see-also"></a>Vea también

+ [Aptitudes predefinidas](cognitive-search-predefined-skills.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Aptitud cognitiva Reconocimiento de entidades con nombre](cognitive-search-skill-entity-recognition.md)
