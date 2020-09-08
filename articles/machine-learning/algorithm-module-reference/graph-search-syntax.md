---
title: Sintaxis de consulta de búsqueda de Graph
titleSuffix: Azure Machine Learning
description: Obtenga información sobre cómo usar la sintaxis de consulta de búsqueda para buscar nodos en el gráfico de la canalización.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: xiaoharper
ms.author: zhanxia
ms.date: 8/24/2020
ms.openlocfilehash: 122da78206ef2055b4867727b174293e74133c05
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88817188"
---
# <a name="graph-search-query-syntax"></a>Sintaxis de consulta de búsqueda de Graph

En este artículo, obtendrá información sobre la sintaxis de las consultas de búsqueda en gráficos en Azure Machine Learning. La característica de búsqueda de Graph permite buscar un nodo por su nombre y sus propiedades. 

 ![Captura de pantalla animada que muestra una experiencia de búsqueda en gráficos de ejemplo](media/search/graph-search.gif)

La búsqueda en gráficos admite la búsqueda de palabras clave de texto completo en los comentarios y el nombre del nodo. También puede filtrar por la propiedad del nodo, como runStatus, Duration o computeTarget. La búsqueda de palabras clave se basa en la consulta de Lucene. Una consulta de búsqueda completa tiene el siguiente aspecto:  

**[consulta de lucene | [consulta por filtro]** 

Puede usar una consulta de Lucene o una consulta por filtro. Para usar ambas opciones, incluya el separador **|** . La sintaxis de la consulta por filtro es más estricta que la consulta de Lucene. Por tanto, si la entrada del cliente se puede analizar con ambas opciones, se aplicará al consulta con filtro.

 

## <a name="lucene-query"></a>Consulta de Lucene

La búsqueda en gráficos usa la consulta simple de Lucene como sintaxis de búsqueda de texto completo en los "comentarios" y el "nombre" del nodo. Se admiten los siguientes operadores de Lucene:

 
- Y/O
- Coincidencia de caracteres comodín con **?** y operadores **\*** .

### <a name="examples"></a>Ejemplos

- Búsqueda simple: `JSON Data`

- Y/O: `JSON AND Validation`

- Varios Y/O: `(JSON AND Validation) OR (TSV AND Training)`

 
- Coincidencia de caracteres comodín: 
    - `machi?e learning`
    -   `mach*ing`
 
>[!NOTE]
> No se puede iniciar una consulta de Lucene con un carácter "*".

##  <a name="filter-query"></a>Consulta con filtro

 
Las consultas con filtro usan el siguiente patrón:
 
    **[key1] [operator1] [value1]; [key2] [operator1] [value2];**

 
Puede usar las siguientes propiedades de nodo como claves:

- runStatus
- compute
- duration
- reuse

También puede usar los siguientes operadores:

- Mayor o igual: `>=`
- Menor o igual: `<=`
- Mayor: `>`
- Menor: `<`
- Igual: `==`
- Contiene: `=`
- No igual: `!=`
- En: `in`

 
 

### <a name="example"></a>Ejemplo

- duration > 100;
- status en { Failed,NotStarted}
- compute en {gpu-cluster}; runStatus en {Completed}

## <a name="technical-notes"></a>Notas técnicas

- La relación entre varios filtros es "Y".
- Si se elige `>=,  >,  <, or  <=`, los valores se convertirán automáticamente al tipo numérico. De lo contrario, se usan tipos de cadena para la comparación.
- En el caso de todos los valores de tipo de cadena, no se distinguen mayúsculas de minúsculas en la comparación.
- El operador "In" espera una colección como valor; la sintaxis de la colección es `{name1, name2, name3}`.
- Se omitirá el espacio entre palabras clave.