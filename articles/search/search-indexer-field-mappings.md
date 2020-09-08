---
title: Asignaciones de campos en indexadores
titleSuffix: Azure Cognitive Search
description: Configure asignaciones de campos en un indexador para tener en cuenta las diferencias en los nombres de campo y las representaciones de datos.
manager: nitinme
author: mattmsft
ms.author: magottei
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/11/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: fe4d42fd74b4efd67a01f32611bd170862ec84d0
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89007135"
---
# <a name="field-mappings-and-transformations-using-azure-cognitive-search-indexers"></a>Transformaciones y asignaciones de campos mediante indexadores de Azure Cognitive Search

![Fases del indexador](./media/search-indexer-field-mappings/indexer-stages-field-mappings.png "fases del indexador")

Al usar indexadores de Azure Cognitive Search, habrá ocasiones en que observará que los datos de entrada no coinciden demasiado con el esquema del índice de destino. En esos casos, puede usar **asignaciones de campos** para modificar los datos durante el proceso de indexación.

Algunas situaciones donde las asignaciones de campos son útiles:

* El origen de datos tiene un campo llamado `_id`, pero Azure Cognitive Search no permite los nombres de campo que empiezan por un carácter de subrayado. Una asignación de campo permite cambiar el nombre de un campo de forma eficaz.
* Desea rellenar varios campos en el índice con datos del mismo origen de datos. Por ejemplo, puede querer aplicar diferentes analizadores a esos campos.
* Desea rellenar un campo de índice con los datos de más de un origen de datos, y cada origen de datos usa nombres de campo diferentes.
* Necesita codificar o descodificar sus datos con Base64. Las asignaciones de campos admiten varias **funciones de asignación**, incluidas las funciones de codificación y descodificación Base64.

> [!NOTE]
> Las asignaciones de campos en los indexadores son una manera sencilla de asignar campos de datos a campos de índice, con cierta capacidad para la conversión de datos ligeros. Los datos más complejos pueden requerir un procesamiento previo para transformarlos en un formato que favorezca la indexación. Una opción que se puede considerar es [Azure Data Factory](../data-factory/index.yml).

## <a name="set-up-field-mappings"></a>Configuración de asignaciones de campos

Una asignación de campos consta de tres partes:

1. `sourceFieldName`, que representa un campo de su origen de datos. Esta propiedad es obligatoria.
2. `targetFieldName`opcional, que representa un campo de su índice de búsqueda. Si se omite, se usa el mismo nombre que en el origen de datos.
3. `mappingFunction`opcional, que puede transformar sus datos con una de las diversas funciones predefinidas. Se puede aplicar en las asignaciones de campos de entrada y de salida. La lista completa de funciones se encuentra [a continuación](#mappingFunctions).

Las asignaciones de campos se agregan a la matriz `fieldMappings` de la definición del indexador.

> [!NOTE]
> Si no se agregan asignaciones de campo, los indizadores asumen que los campos de origen de datos deben asignarse a campos de índice con el mismo nombre. Al agregar una asignación de campos, se quitan estas asignaciones de campos predeterminadas para el campo de origen y de destino. Algunos indizadores, como [el indizador de Blob Storage](search-howto-indexing-azure-blob-storage.md), agregan asignaciones de campos predeterminadas para el campo clave del índice.

## <a name="map-fields-using-the-rest-api"></a>Asignación de campos usando la API de REST

Puede agregar asignaciones de campos al crear un indexador con la solicitud API para [crear un indexador](/rest/api/searchservice/create-Indexer). Puede administrar las asignaciones de campos de un indexador existente con la solicitud API para [actualizar un indexador](/rest/api/searchservice/update-indexer).

Por ejemplo, aquí se describe cómo asignar un campo de origen a un campo de destino con un nombre diferente:

```JSON

PUT https://[service name].search.windows.net/indexers/myindexer?api-version=[api-version]
Content-Type: application/json
api-key: [admin key]
{
    "dataSourceName" : "mydatasource",
    "targetIndexName" : "myindex",
    "fieldMappings" : [ { "sourceFieldName" : "_id", "targetFieldName" : "id" } ]
}
```

Se puede hacer referencia a un campo de origen en varias asignaciones de campos. El ejemplo siguiente muestra cómo "bifurcar" un campo, copiando el mismo campo de origen en dos campos de índice distintos:

```JSON

"fieldMappings" : [
    { "sourceFieldName" : "text", "targetFieldName" : "textStandardEnglishAnalyzer" },
    { "sourceFieldName" : "text", "targetFieldName" : "textSoundexAnalyzer" }
]
```

> [!NOTE]
> Azure Cognitive Search usa una comparación que no distingue mayúsculas de minúsculas para resolver los nombres de campo y función de las asignaciones de campos. Esto es práctico (no es necesario que el uso de mayúsculas y minúsculas sea correcto en todo momento), pero se traduce en que su índice u origen de datos no puede tener campos que difieran únicamente en mayúsculas y minúsculas.  
>
>

## <a name="map-fields-using-the-net-sdk"></a>Asignación de campos usando el SDK de .NET

Las asignaciones de campos en el SDK de .NET se definen con la clase [FieldMapping](/dotnet/api/microsoft.azure.search.models.fieldmapping), que tiene las propiedades `SourceFieldName` y `TargetFieldName` y una referencia a `MappingFunction` opcional.

Se pueden especificar asignaciones de campos al crear el indexador o posteriormente estableciendo directamente la propiedad `Indexer.FieldMappings`.

En el siguiente ejemplo de C# se establecen las asignaciones de campo al crear un indexador.

```csharp
  List<FieldMapping> map = new List<FieldMapping> {
    // removes a leading underscore from a field name
    new FieldMapping("_custId", "custId"),
    // URL-encodes a field for use as the index key
    new FieldMapping("docPath", "docId", FieldMappingFunction.Base64Encode() )
  };

  Indexer sqlIndexer = new Indexer(
    name: "azure-sql-indexer",
    dataSourceName: sqlDataSource.Name,
    targetIndexName: index.Name,
    fieldMappings: map,
    schedule: new IndexingSchedule(TimeSpan.FromDays(1)));

  await searchService.Indexers.CreateOrUpdateAsync(indexer);
```

<a name="mappingFunctions"></a>

## <a name="field-mapping-functions"></a>Funciones de asignación de campos

Una función de asignación de campo transforma el contenido de un campo antes de almacenarlo en el índice. Actualmente, se admiten las siguientes funciones de asignación:

* [base64Encode](#base64EncodeFunction)
* [base64Decode](#base64DecodeFunction)
* [extractTokenAtPosition](#extractTokenAtPositionFunction)
* [jsonArrayToStringCollection](#jsonArrayToStringCollectionFunction)
* [urlEncode](#urlEncodeFunction)
* [urlDecode](#urlDecodeFunction)

<a name="base64EncodeFunction"></a>

### <a name="base64encode-function"></a>Función base64Encode

Realiza una codificación Base64 *segura para direcciones URL* de la cadena de entrada. Asume que la entrada presenta una codificación UTF-8.

#### <a name="example---document-key-lookup"></a>Ejemplo: búsqueda de clave de documento

Solo pueden aparecer caracteres seguros para direcciones URL en una clave de documento de Azure Cognitive Search (porque los clientes deben poder enviar el documento con la [API de búsqueda](/rest/api/searchservice/lookup-document)). Si el campo de origen de la clave contiene caracteres de dirección URL no seguros, puede usar la función `base64Encode` para convertirlo en el momento de la indexación. Sin embargo, una clave de documento (tanto antes como después de la conversión) no puede tener más de 1024 caracteres.

Al recuperar la clave codificada en el tiempo de búsqueda, puede usar la función `base64Decode` para obtener el valor de clave original y usarlo para recuperar el documento de origen.

```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "SourceKey",
    "targetFieldName" : "IndexKey",
    "mappingFunction" : {
      "name" : "base64Encode",
      "parameters" : { "useHttpServerUtilityUrlTokenEncode" : false }
    }
  }]
 ```

#### <a name="example---preserve-original-values"></a>Ejemplo: conservación de los valores originales

El [indizador de Blob Storage](search-howto-indexing-azure-blob-storage.md) agrega automáticamente una asignación de campos de `metadata_storage_path`, el URI del blob, al campo clave del índice si no se especifica ninguna asignación de campos. Este valor está codificado en Base64, por lo que es seguro usarlo como una clave de documento de Azure Cognitive Search. En el ejemplo siguiente, se muestra cómo asignar simultáneamente una versión codificada en Base64 con *seguridad de direcciones URL* de un campo `metadata_storage_path` a `index_key` y conservar el valor original en un campo `metadata_storage_path`:

```JSON

"fieldMappings": [
  {
    "sourceFieldName": "metadata_storage_path",
    "targetFieldName": "metadata_storage_path"
  },
  {
    "sourceFieldName": "metadata_storage_path",
    "targetFieldName": "index_key",
    "mappingFunction": {
       "name": "base64Encode"
    }
  }
]
```

Si no incluye una propiedad de parámetros de la función de asignación, el valor `{"useHttpServerUtilityUrlTokenEncode" : true}` se establece como valor predeterminado.

Azure Cognitive Search admite dos codificaciones Base64 distintas. Debe usar los mismos parámetros al codificar y descodificar el mismo campo. Para más información, vea [Opciones de codificación Base64](#base64details) para decidir qué parámetros usar.

<a name="base64DecodeFunction"></a>

### <a name="base64decode-function"></a>Función Base64Decode

Realiza una descodificación Base64 de la cadena de entrada. Se da por hecho que la entrada es una cadena con codificación Base64 *segura para direcciones URL*.

#### <a name="example---decode-blob-metadata-or-urls"></a>Ejemplo: descodificar direcciones URL o metadatos de blob

El origen de datos podría contener cadenas con codificación Base64, como cadenas de metadatos de blob o direcciones URL web, que quiera incluir en búsquedas como texto sin formato. Puede usar la función `base64Decode` para volver a convertir los datos codificados en cadenas normales al rellenar el índice de búsqueda.

```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "Base64EncodedMetadata",
    "targetFieldName" : "SearchableMetadata",
    "mappingFunction" : { 
      "name" : "base64Decode", 
      "parameters" : { "useHttpServerUtilityUrlTokenDecode" : false }
    }
  }]
```

Si no incluye una propiedad de parámetros, el valor `{"useHttpServerUtilityUrlTokenEncode" : true}` se establece como valor predeterminado.

Azure Cognitive Search admite dos codificaciones Base64 distintas. Debe usar los mismos parámetros al codificar y descodificar el mismo campo. Para más información, vea [Opciones de codificación Base64](#base64details) para decidir qué parámetros usar.

<a name="base64details"></a>

#### <a name="base64-encoding-options"></a>Opciones de codificación Base64

Azure Cognitive Search admite la codificación Base64 de seguridad de direcciones URL y la codificación Base64 normal. Una cadena codificada con Base64 durante la indexación se debe descodificar más adelante con las mismas opciones de codificación o, de lo contrario, el resultado no coincidirá con el original.

Si los parámetros `useHttpServerUtilityUrlTokenEncode` o `useHttpServerUtilityUrlTokenDecode` para codificar y descodificar respectivamente se establecen en `true`, `base64Encode` se comporta como [HttpServerUtility.UrlTokenEncode](/dotnet/api/system.web.httpserverutility.urltokenencode?view=netframework-4.8) y `base64Decode` se comporta como [HttpServerUtility.UrlTokenDecode](/dotnet/api/system.web.httpserverutility.urltokendecode?view=netframework-4.8).

> [!WARNING]
> Si se utiliza `base64Encode` para generar valores de clave, `useHttpServerUtilityUrlTokenEncode` debe establecerse en true. Solo se puede usar la codificación Base64 de seguridad de direcciones URL para los valores de clave. Consulte [Reglas de nomenclatura &#40;Azure Cognitive Search&#41;](/rest/api/searchservice/naming-rules) para obtener el conjunto completo de restricciones sobre los caracteres de los valores de clave.

Las bibliotecas .NET de Azure Cognitive Search asumen .NET Framework completo, que proporciona codificación integrada. Las opciones `useHttpServerUtilityUrlTokenEncode` y `useHttpServerUtilityUrlTokenDecode` aprovechan esta funcionalidad integrada. Si usa .NET Core u otro marco, se recomienda establecer esas opciones en `false` y llamar directamente a las funciones de codificación y descodificación de su marco de trabajo.

La tabla siguiente compara diferentes codificaciones Base64 de la cadena `00>00?00`. Para determinar el procesamiento adicional necesario (si existe) para las funciones de Base64, aplique la función de codificación de bibliotecas en la cadena `00>00?00` y compare el resultado con el resultado esperado `MDA-MDA_MDA`.

| Encoding | Salida de codificación Base64 | Procesamiento adicional después de la codificación de bibliotecas | Procesamiento adicional antes de la descodificación de bibliotecas |
| --- | --- | --- | --- |
| Base64 con espaciado interno | `MDA+MDA/MDA=` | Use caracteres seguros para direcciones URL y quite el espaciado interno | Use caracteres estándar de Base64 y agregue espaciado interno |
| Base64 sin espaciado interno | `MDA+MDA/MDA` | Use caracteres seguros para direcciones URL | Use caracteres estándar de Base64 |
| Base64 de seguridad de direcciones URL con espaciado interno | `MDA-MDA_MDA=` | Quite el espaciado interno | Agregue el espaciado interno |
| Base64 de seguridad de direcciones URL sin espaciado interno | `MDA-MDA_MDA` | None | None |

<a name="extractTokenAtPositionFunction"></a>

### <a name="extracttokenatposition-function"></a>Función extractTokenAtPosition

Divide un campo de cadena con el delimitador especificado y elige el token en la posición especificada de la división resultante.

Esta función utiliza los siguientes parámetros:

* `delimiter`: una cadena para su uso como separador al dividir la cadena de entrada.
* `position`: una posición de base cero entera del token que se va a elegir tras dividirse la cadena de entrada.

Por ejemplo, si la entrada es `Jane Doe`, `delimiter` es `" "`(espacio) y `position` es 0, el resultado es `Jane`; si `position` es 1, el resultado es `Doe`. Si la posición hace referencia a un token que no existe, se devolverá un error.

#### <a name="example---extract-a-name"></a>Ejemplo: extraer un nombre

Su origen de datos contiene un campo `PersonName` y desea indexarlo como dos campos `FirstName` y `LastName` independientes. Puede usar esta función para dividir la entrada con el carácter de espacio como delimitador.

```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "PersonName",
    "targetFieldName" : "FirstName",
    "mappingFunction" : { "name" : "extractTokenAtPosition", "parameters" : { "delimiter" : " ", "position" : 0 } }
  },
  {
    "sourceFieldName" : "PersonName",
    "targetFieldName" : "LastName",
    "mappingFunction" : { "name" : "extractTokenAtPosition", "parameters" : { "delimiter" : " ", "position" : 1 } }
  }]
```

<a name="jsonArrayToStringCollectionFunction"></a>

### <a name="jsonarraytostringcollection-function"></a>Función jsonArrayToStringCollection

Transforma una cadena con formato de una matriz JSON de cadenas en una matriz de cadenas que puede usarse para rellenar un campo `Collection(Edm.String)` del índice.

Por ejemplo, si la cadena de entrada es `["red", "white", "blue"]`, el campo de destino de tipo `Collection(Edm.String)` se rellenará con los tres valores `red`, `white` y `blue`. En el caso de los valores de entrada que no pueden analizarse como matrices de cadenas JSON, se devolverá un error.

#### <a name="example---populate-collection-from-relational-data"></a>Ejemplo: rellenar la colección de datos relacionales

Azure SQL Database no tiene un tipo de datos integrado que se asigne de forma natural a los campos `Collection(Edm.String)` de Azure Cognitive Search. Para rellenar los campos de colección de cadenas, puede preprocesar los datos de origen como una matriz de cadenas JSON y, luego, usar la función de asignación `jsonArrayToStringCollection`.

```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "tags", 
    "mappingFunction" : { "name" : "jsonArrayToStringCollection" }
  }]
```

<a name="urlEncodeFunction"></a>

### <a name="urlencode-function"></a>Función urlEncode

Esta función se puede utilizar para codificar una cadena de modo que sea "segura para la dirección URL". Cuando se usa con una cadena que contiene caracteres no permitidos en una dirección URL, esta función convertirá esos caracteres "no seguros" en equivalentes de carácter-entidad. Esta función utiliza el formato de codificación UTF-8.

#### <a name="example---document-key-lookup"></a>Ejemplo: búsqueda de clave de documento

La función `urlEncode` se puede usar como alternativa a la función `base64Encode`, si solo se van a convertir los caracteres no seguros de la dirección URL y se mantienen los otros caracteres tal cual.

Por ejemplo, si la cadena de entrada es `<hello>`, el campo de destino de tipo `(Edm.String)` se rellenará con el valor `%3chello%3e`.

Al recuperar la clave codificada en el tiempo de búsqueda, puede usar la función `urlDecode` para obtener el valor de clave original y usarlo para recuperar el documento de origen.

```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "SourceKey",
    "targetFieldName" : "IndexKey",
    "mappingFunction" : {
      "name" : "urlEncode"
    }
  }]
 ```

 <a name="urlDecodeFunction"></a>

 ### <a name="urldecode-function"></a>Función urlDecode

 Esta función convierte una cadena con codificación URL en una cadena descodificada mediante el formato de codificación UTF-8.

 ### <a name="example---decode-blob-metadata"></a>Ejemplo: descodificar metadatos de blob

 Algunos clientes de almacenamiento de Azure codifican automáticamente los metadatos de blobs si contienen caracteres que no son ASCII. Sin embargo, si quiere que dichos metadatos se puedan buscar (como texto sin formato), puede usar la función `urlDecode` para volver a convertir los datos codificados en cadenas "normales" al rellenar su índice de búsqueda.

 ```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "UrlEncodedMetadata",
    "targetFieldName" : "SearchableMetadata",
    "mappingFunction" : {
      "name" : "urlDecode"
    }
  }]
 ```
 
 <a name="fixedLengthEncodeFunction"></a>
 
 ### <a name="fixedlengthencode-function"></a>Función fixedLengthEncode
 
 Esta función convierte una cadena de cualquier longitud en una cadena de longitud fija.
 
 ### <a name="example---map-document-keys-that-are-too-long"></a>Ejemplo: asignar claves de documento demasiado largas
 
Si se producen errores que indican que la clave del documento tiene más de 1024 caracteres, esta función se puede aplicar para reducir la longitud de la clave del documento.

 ```JSON

"fieldMappings" : [
  {
    "sourceFieldName" : "metadata_storage_path",
    "targetFieldName" : "your key field",
    "mappingFunction" : {
      "name" : "fixedLengthEncode"
    }
  }]
 ```