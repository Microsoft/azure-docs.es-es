---
title: Incorporación de etiquetas a gemelos digitales
titleSuffix: Azure Digital Twins
description: Visualización de la implementación de etiquetas en gemelos digitales
author: baanders
ms.author: baanders
ms.date: 7/22/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 59f68909e2f3704fea5c38e3f1535f5996b284ab
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095405"
---
# <a name="add-tags-to-digital-twins"></a>Incorporación de etiquetas a gemelos digitales 

Puede usar el concepto de etiquetas para detallar la identificación y la categorización de los gemelos digitales. En concreto, los usuarios podrían querer replicar etiquetas de sistemas existentes, como las [etiquetas de Haystack](https://project-haystack.org/doc/TagModel), en las instancias de Azure Digital Twins. 

En este documento se describen los patrones que se pueden usar para implementar etiquetas en gemelos digitales.

Las etiquetas se agregan primero como propiedades en el [modelo](concepts-models.md) que describe un gemelo digital. A continuación, esa propiedad se establece en el gemelo durante su creación en función del modelo. Después, las etiquetas se pueden usar en [consultas](concepts-query-language.md) para identificar y filtrar los gemelos.

## <a name="marker-tags"></a>Etiquetas de marcador 

Una **etiqueta de marcador** es una cadena simple que se usa para marcar o categorizar un gemelo digital, como "blue" o "red". Esta cadena es el nombre de la etiqueta y las etiquetas de marcador no tienen valor significativo; significa simplemente que está presente (o ausente). 

### <a name="add-marker-tags-to-model"></a>Incorporación de etiquetas de marcador al modelo 

Las etiquetas de marcador se modelan como asignación [DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) de `string` a `boolean`. Se omite el booleano `mapValue`, ya que la presencia de la etiqueta es lo único que importa. 

Este es un extracto de un modelo gemelo que implementa una etiqueta de marcador como propiedad:

```json
{
  "@type": "Property",
  "name": "tags",
  "schema": {
    "@type": "Map",
    "mapKey": {
      "name": "tagName",
      "schema": "string"
    },
    "mapValue": {
      "name": "tagValue",
      "schema": "boolean"
    }
  }
}
```

### <a name="add-marker-tags-to-digital-twins"></a>Incorporación de etiquetas de marcador a gemelos digitales

Cuando la propiedad `tags` ya forme parte de un modelo de gemelos digitales, configure el valor de esta propiedad para establecer la etiqueta del marcador en el gemelo digital. 

Este es un ejemplo que rellena el marcador `tags` para tres gemelos:

```csharp
entity-01: "tags": { "red": true, "round": true } 
entity-02: "tags": { "blue": true, "round": true } 
entity-03: "tags": { "red": true, "large": true } 
```

### <a name="query-with-marker-tags"></a>Consulta con etiquetas de marcador

Una vez que se han agregado etiquetas a gemelos digitales, estas se pueden usar para filtrar los gemelos en las consultas. 

Esta es una consulta para obtener todos los gemelos etiquetados como "red": 

```sql
select * from digitaltwins where is_defined(tags.red) 
```

También puede combinar etiquetas para consultas más complejas. Esta es una consulta para obtener todos los gemelos etiquetados como "round" y no "red": 

```sql
select * from digitaltwins where not is_defined(tags.red) and is_defined(tags.round) 
```

## <a name="value-tags"></a>Etiquetas de valor 

Una **etiqueta de valor** es un par clave-valor que se utiliza para asignar un valor a cada etiqueta, como `"color": "blue"` o `"color": "red"`. Una vez creada la etiqueta de valor, también se puede usar como etiqueta de marcador; para ello, hay que omitir el valor de la etiqueta. 

### <a name="add-value-tags-to-model"></a>Incorporación de etiquetas de valor al modelo 

Las etiquetas de valor se modelan como asignación [DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) de `string` a `string`. Tanto `mapKey` como `mapValue` tienen significado. 

Este es un extracto de un modelo gemelo que implementa una etiqueta de valor como propiedad:

```json
{
  "@type": "Property",
  "name": "tags",
  "schema": {
    "@type": "Map",
    "mapKey": {
      "name": "tagName",
      "schema": "string"
    },
    "mapValue": {
      "name": "tagValue",
      "schema": "string"
    }
  }
} 
```

### <a name="add-value-tags-to-digital-twins"></a>Incorporación de etiquetas de valor a gemelos digitales

Al igual que con las etiquetas de marcador, puede establecer la etiqueta de valor en un gemelo digital al configurar el valor de esta propiedad `tags` del modelo. Para usar una etiqueta de valor como etiqueta de marcador, puede establecer el campo `tagValue` en el valor de cadena vacía (`""`). 

Este es un ejemplo que rellena el valor `tags` para tres gemelos:

```csharp
entity-01: "tags": { "red": "", "size": "large" } 
entity-02: "tags": { "purple": "", "size": "small" } 
entity-03: "tags": { "red": "", "size": "small" } 
```

Tenga en cuenta que `red` y `purple` se usan como etiquetas de marcador en este ejemplo.

### <a name="query-with-value-tags"></a>Consulta con etiquetas de valor

Al igual que con las etiquetas de marcador, puede usar etiquetas de valor para filtrar gemelos en las consultas. También puede usar etiquetas de valor y etiquetas de marcador juntas.

En el ejemplo anterior, `red` se usa como etiqueta de marcador. Esta es una consulta para obtener todos los gemelos etiquetados como "red": 

```sql
select * from digitaltwins where is_defined(tags.red) 
```

Esta es una consulta para obtener todas las entidades que son "small" (etiqueta de valor) y no "red": 

```sql
select * from digitaltwins where not is_defined(tags.red) and tags.size = 'small' 
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre el diseño y la administración de modelos de gemelos digitales:
* [*Procedimiento: Administración de modelos personalizados*](how-to-manage-model.md)

Más información sobre la consulta del grafo gemelo:
* [*Procedimiento: Consulta del grafo gemelo*](how-to-query-graph.md)