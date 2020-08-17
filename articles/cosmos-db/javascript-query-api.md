---
title: Uso de la API de consulta integrada de JavaScript en procedimientos almacenados y desencadenadores de Azure Cosmos DB
description: En este artículo se presentan los conceptos de la API de consulta integrada en lenguaje JavaScript para crear desencadenadores y procedimientos almacenados en Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 05/07/2020
ms.author: tisande
ms.reviewer: sngun
ms.custom: devx-track-javascript
ms.openlocfilehash: 56fbcc3950a739c4c9fc3df86468301e2e2ff4d8
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421134"
---
# <a name="javascript-query-api-in-azure-cosmos-db"></a>API de consulta de JavaScript en Azure Cosmos DB

Además de emitir consultas con la API de SQL en Azure Cosmos DB, el [SDK del lado servidor de Cosmos DB](https://azure.github.io/azure-cosmosdb-js-server/) proporciona una interfaz de JavaScript para realizar consultas optimizadas en procedimientos almacenados y desencadenadores de Cosmos DB. No es necesario conocer el lenguaje SQL para utilizar esta interfaz JavaScript. JavaScript Query API permite crear mediante programación consultas pasando las funciones de predicado a una secuencia de llamadas de función, con una sintaxis familiar para los elementos integrados de matriz de ECMAScript5 y conocidas bibliotecas de JavaScript, como Lodash. Las consultas se analizan con el entorno de ejecución de JavaScript para que se ejecuten eficazmente mediante índices de Azure Cosmos DB.

## <a name="supported-javascript-functions"></a>Funciones JavaScript admitidas

| **Function** | **Descripción** |
|---------|---------|
|`chain() ... .value([callback] [, options])`|Inicia una llamada encadenada que debe terminarse con value().|
|`filter(predicateFunction [, options] [, callback])`|Filtra la entrada usando una función de predicado que devuelve True o False para filtrar los documentos de entrada y salida en el conjunto resultante. Esta función es similar a la de una cláusula WHERE de SQL.|
|`flatten([isShallow] [, options] [, callback])`|Combina y reduce las matrices de cada elemento de entrada en una sola matriz. Esta función es similar a la de SelectMany de LINQ.|
|`map(transformationFunction [, options] [, callback])`|Aplica a una proyección dada una función de transformación que asigna cada elemento de entrada a un valor u objeto de JavaScript. Esta función es similar a la de una cláusula SELECT de SQL.|
|`pluck([propertyName] [, options] [, callback])`|Esta función es un acceso directo a una asignación que extrae el valor de una única propiedad de cada elemento de entrada.|
|`sortBy([predicate] [, options] [, callback])`|Genera un nuevo conjunto de documentos al clasificarlos en orden ascendente en la secuencia de documentos de entrada mediante el predicado especificado. Esta función es similar al de una cláusula ORDER BY de SQL.|
|`sortByDescending([predicate] [, options] [, callback])`|Produce un nuevo conjunto de documentos al clasificarlos en orden descendente en la secuencia de documentos de entrada mediante el predicado especificado. Esta función es similar a la de una cláusula ORDER BY x DESC de SQL.|
|`unwind(collectionSelector, [resultSelector], [options], [callback])`|Realiza una autocombinación con matriz interna y agrega los resultados de ambos lados como tuplas a la proyección del resultado. Por ejemplo, al unir un documento de una persona con un documento de mascota, se producirían tuplas [persona,mascota]. Esto es similar a SelectMany de .NET LINK.|

Cuando se incluye dentro del predicado o las funciones selectoras, las siguientes construcciones de JavaScript se optimizan automáticamente para ejecutarse directamente en índices de Azure Cosmos DB:

- Operadores simples: `=` `+` `-` `*` `/` `%` `|` `^` `&` `==` `!=` `===` `!===` `<` `>` `<=` `>=` `||` `&&` `<<` `>>` `>>>!` `~`
- Literales, incluido el literal de objeto: {}
- var, return

Las siguientes construcciones de JavaScript no se optimizan para índices de Azure Cosmos DB:

- Flujo de control (por ejemplo, if, for, while)
- Llamadas a funciones

Para más información, consulte la [documentación de JavaScript del lado servidor de Azure Cosmos DB](https://azure.github.io/azure-cosmosdb-js-server/).

## <a name="sql-to-javascript-cheat-sheet"></a>Hoja de referencia de SQL a JavaScript

En la tabla siguiente se muestran varias consultas SQL con las consultas de JavaScript correspondientes. Como sucede con las consultas SQL, las propiedades (por ejemplo, item.id) distinguen mayúsculas de minúsculas.

> [!NOTE]
> `__` (subrayado doble) es un alias para `getContext().getCollection()` cuando se usa JavaScript Query API.

|**SQL**|**JavaScript Query API**|**Descripción**|
|---|---|---|
|SELECT *<br>FROM docs| __.map(function(doc) { <br>&nbsp;&nbsp;&nbsp;&nbsp;return doc;<br>});|Devuelve resultados de todos los documentos (paginados con el token de continuación) tal y como están.|
|SELECT <br>&nbsp;&nbsp;&nbsp;docs.id,<br>&nbsp;&nbsp;&nbsp;docs.message AS msg,<br>&nbsp;&nbsp;&nbsp;docs.actions <br>FROM docs|__.map(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;return {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;id: doc.id,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;msg: doc.message,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;actions:doc.actions<br>&nbsp;&nbsp;&nbsp;&nbsp;};<br>});|Proyecta el id., el mensaje (con el alias msg) y la acción de todos los documentos.|
|SELECT *<br>FROM docs<br>WHERE<br>&nbsp;&nbsp;&nbsp;docs.id="X998_Y998"|__.filter(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;return doc.id ==="X998_Y998";<br>});|Realiza consultas de los documentos con el predicado: id = "X998_Y998".|
|SELECT *<br>FROM docs<br>WHERE<br>&nbsp;&nbsp;&nbsp;ARRAY_CONTAINS(docs.Tags, 123)|__.filter(function(x) {<br>&nbsp;&nbsp;&nbsp;&nbsp;return x.Tags &amp;& x.Tags.indexOf(123) &amp;gt; -1;<br>});|Realiza consultas de los documentos que tengan una propiedad Tags que sea una matriz que contiene el valor 123.|
|SELECT<br>&nbsp;&nbsp;&nbsp;docs.id,<br>&nbsp;&nbsp;&nbsp;docs.message AS msg<br>FROM docs<br>WHERE<br>&nbsp;&nbsp;&nbsp;docs.id="X998_Y998"|__.chain()<br>&nbsp;&nbsp;&nbsp;&nbsp;.filter(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return doc.id ==="X998_Y998";<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>&nbsp;&nbsp;&nbsp;&nbsp;.map(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;id: doc.id,<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;msg: doc.message<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>.value();|Realiza consultas de los documentos con un predicado, id = "X998_Y998", y, después, proyecta el id. y el mensaje (con el alias msg).|
|SELECT VALUE tag<br>FROM docs<br>JOIN tag IN docs.Tags<br>ORDER BY docs._ts|__.chain()<br>&nbsp;&nbsp;&nbsp;&nbsp;.filter(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  &nbsp;return doc.Tags &amp;& Array.isArray(doc.Tags);<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>&nbsp;&nbsp;&nbsp;&nbsp;.sortBy(function(doc) {<br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return doc._ts;<br>&nbsp;&nbsp;&nbsp;&nbsp;})<br>&nbsp;&nbsp;&nbsp;&nbsp;.pluck("Tags")<br>&nbsp;&nbsp;&nbsp;&nbsp;.flatten()<br>&nbsp;&nbsp;&nbsp;&nbsp;.value()|Filtra los documentos que tienen una propiedad de matriz, Tags, y ordena los documentos resultantes por la propiedad del sistema _ts timestamp; después, proyecta + flattens en la matriz Tags.|

## <a name="next-steps"></a>Pasos siguientes

Aprenda más conceptos y cómo escribir y utilizar procedimientos almacenados, desencadenadores y funciones definidas por el usuario en Azure Cosmos DB:

- [How to write stored procedures and triggers using Javascript Query API in Azure Cosmos DB](how-to-write-javascript-query-api.md) (Escritura de procedimientos almacenados y desencadenadores con Javascript Query API en Azure Cosmos DB)
- [Working with Azure Cosmos DB stored procedures, triggers, and user-defined functions](stored-procedures-triggers-udfs.md) (Trabajo con procedimientos almacenados, desencadenadores y funciones definidas por el usuario en Azure Cosmos DB)
- [How to use stored procedures, triggers, user-defined functions in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md) (Uso procedimientos almacenados, desencadenadores y funciones definidas por el usuario en Azure Cosmos DB)
- [Referencia de la API del servidor JavaScript de Azure Cosmos DB](https://azure.github.io/azure-cosmosdb-js-server)
- [JavaScript ES6 (ECMA 2015)](https://www.ecma-international.org/ecma-262/6.0/)
