---
title: Aptitud cognitiva para la extracción de frases clave
titleSuffix: Azure Cognitive Search
description: Evalúa el texto no estructurado y, en cada registro, devuelve una lista de frases clave en un canal de enriquecimiento con inteligencia artificial de Búsqueda cognitiva de Azure.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: 145d8265412c4efcaa2afad97feb23528a36191d
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936068"
---
#   <a name="key-phrase-extraction-cognitive-skill"></a>Aptitud cognitiva para la extracción de frases clave

La aptitud **Extracción de frases clave** evalúa el texto no estructurado y, en cada registro, devuelve una lista de frases clave. Esta aptitud utiliza los modelos de aprendizaje automático proporcionados por [Text Analytics](../cognitive-services/text-analytics/overview.md) en Cognitive Services.

Esta capacidad es útil si necesita identificar rápidamente los principales puntos de conversación en el registro. Por ejemplo, dado el texto de entrada "La comida estaba deliciosa y el personal era maravilloso", el servicio devuelve "comida" y "personal maravilloso".

> [!NOTE]
> A medida que expanda el ámbito aumentando la frecuencia de procesamiento, agregando más documentos o agregando más algoritmos de IA, tendrá que [asociar un recurso facturable de Cognitive Services](cognitive-search-attach-cognitive-services.md). Los cargos se acumulan cuando se llama a las API de Cognitive Services y por la extracción de imágenes como parte de la fase de descifrado de documentos de Azure Cognitive Search. No hay ningún cargo por la extracción de texto de documentos.
>
> La ejecución de aptitudes integradas se cobra según los [precios de pago por uso de Cognitive Services](https://azure.microsoft.com/pricing/details/cognitive-services/) existentes. Los precios de la extracción de imágenes se describen en la [página de precios de Búsqueda cognitiva de Azure](https://azure.microsoft.com/pricing/details/search/).


## <a name="odatatype"></a>@odata.type  
Microsoft.Skills.Text.KeyPhraseExtractionSkill 

## <a name="data-limits"></a>Límites de datos
El tamaño máximo de un registro debe tener menos de 50 000 caracteres según la medición de [`String.Length`](/dotnet/api/system.string.length). Si tiene que dividir los datos antes de enviarlos al extractor de frases clave, puede usar la [aptitud de división de texto](cognitive-search-skill-textsplit.md).

## <a name="skill-parameters"></a>Parámetros de la aptitud

Los parámetros distinguen mayúsculas de minúsculas.

| Entradas                | Descripción |
|---------------------|-------------|
| `defaultLanguageCode` | (Opcional) Es el código de idioma que se aplicará a los documentos que no especifiquen el lenguaje de forma explícita.  Si no se especifica el código de idioma predeterminado, se usará el inglés (en) como código de idioma predeterminado. <br/> Vea [Full list of supported languages](../cognitive-services/text-analytics/language-support.md) (Lista completa de idiomas admitidos). |
| `maxKeyPhraseCount`   | (Opcional) Es el número máximo de frases clave para producir. |

## <a name="skill-inputs"></a>Entradas de la aptitud

| Entrada  | Descripción |
|--------------------|-------------|
| `text` | Texto que se va a analizar.|
| `languageCode`    |  Cadena que indica el idioma de los registros. Si no se especifica este parámetro, el código de idioma predeterminado se utilizará para analizar los registros. <br/>Vea [Full list of supported languages](../cognitive-services/text-analytics/language-support.md) (Lista completa de idiomas admitidos).|

## <a name="skill-outputs"></a>Salidas de la aptitud

| Output     | Descripción |
|--------------------|-------------|
| `keyPhrases` | Una lista de frases clave extraídas del texto de entrada. Las frases clave se devuelven por orden de importancia. |


##  <a name="sample-definition"></a>Definición de ejemplo

Considere un registro de SQL que tiene los siguientes campos:

```json
{
    "content": "Glaciers are huge rivers of ice that ooze their way over land, powered by gravity and their own sheer weight. They accumulate ice from snowfall and lose it through melting. As global temperatures have risen, many of the world’s glaciers have already started to shrink and retreat. Continued warming could see many iconic landscapes – from the Canadian Rockies to the Mount Everest region of the Himalayas – lose almost all their glaciers by the end of the century.",
    "language": "en"
}
```

Después, la definición de habilidad puede tener el siguiente aspecto:

```json
 {
    "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
    "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      },
      {
        "name": "languageCode",
        "source": "/document/language" 
      }
    ],
    "outputs": [
      {
        "name": "keyPhrases",
        "targetName": "myKeyPhrases"
      }
    ]
  }
```

##  <a name="sample-output"></a>Salida de ejemplo

En el ejemplo anterior, el resultado de su aptitud se escribirá en un nuevo nodo en el árbol enriquecido denominado "document/myKeyPhrases", ya que es el `targetName` que se ha especificado. Si no especifica un `targetName`, sería "document/keyPhrases".

#### <a name="documentmykeyphrases"></a>document/myKeyPhrases 
```json
            [
              "world’s glaciers", 
              "huge rivers of ice", 
              "Canadian Rockies", 
              "iconic landscapes",
              "Mount Everest region",
              "Continued warming"
            ]
```

Puede usar "document/myKeyPhrases" como entrada en otras aptitudes o como origen de una [asignación de campos de salida](cognitive-search-output-field-mapping.md).

## <a name="errors-and-warnings"></a>Errores y advertencias
Si proporciona un código de idioma no admitido, se generará un error y no se extraerán las frases clave.
Si el texto está vacío, se creará una advertencia.
Si el texto tiene más de 50 000 caracteres, solo se analizarán los primeros 50 000 caracteres y se emitirá una advertencia.

## <a name="see-also"></a>Consulte también

+ [Aptitudes integradas](cognitive-search-predefined-skills.md)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Definición de asignaciones de campos de salida](cognitive-search-output-field-mapping.md)