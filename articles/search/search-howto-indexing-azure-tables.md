---
title: Búsqueda en el contenido de almacenamiento de tablas de Azure
titleSuffix: Azure Cognitive Search
description: Aprenda a indexar datos almacenados en Azure Table Storage con un indizador de Azure Cognitive Search.
manager: nitinme
author: mgottein
ms.author: magottei
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/11/2020
ms.openlocfilehash: 26be48e7968345863799191539bd668ea6d9a4a2
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929574"
---
# <a name="how-to-index-tables-from-azure-table-storage-with-azure-cognitive-search"></a>Indexación de documentos en Azure Blob Storage con Azure Cognitive Search

En este artículo se muestra cómo usar Azure Cognitive Search para indexar datos almacenados en Azure Table Storage.

## <a name="set-up-azure-table-storage-indexing"></a>Configuración de la indexación de Azure Table Storage

Puede configurar un indexador de Azure Table Storage mediante estos recursos:

* [Azure Portal](https://ms.portal.azure.com)
* [API REST](/rest/api/searchservice/Indexer-operations) de Azure Cognitive Search
* [SDK para .NET](/dotnet/api/overview/azure/search) de Azure Cognitive Search

En este caso, demostramos el flujo mediante la API de REST. 

### <a name="step-1-create-a-datasource"></a>Paso 1: Crear un origen de datos

Un origen de datos especifica los datos que se indexan, las credenciales necesarias para acceder a ellos y las directivas que permiten que Azure Cognitive Search identifique cambios en los datos de forma eficiente.

Para realizar la indexación de tablas, el origen de datos debe tener las siguientes propiedades:

- **name** es el nombre único del origen de datos dentro del servicio de búsqueda.
- **type** debe ser `azuretable`.
- El parámetro **credentials** contiene la cadena de conexión de la cuenta de almacenamiento. Para más información, consulte la sección [Formas de especificar las credenciales](#Credentials).
- **container** establece el nombre de la tabla y una consulta opcional.
    - Especifique el nombre de la tabla con el parámetro `name`.
    - De manera opcional, especifique una consulta con el parámetro `query`. 

> [!IMPORTANT] 
> Siempre que sea posible, utilice un filtro en PartitionKey para mejorar el rendimiento. Cualquier otra consulta realiza un examen de toda la tabla, lo que produce un rendimiento deficiente de las tablas grandes. Consulte la sección [Consideraciones sobre rendimiento](#Performance).


Para crear un origen de datos:

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "table-datasource",
        "type" : "azuretable",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-table", "query" : "PartitionKey eq '123'" }
    }   
```

Para más información sobre la API de creación de origen de datos, consulte [Create Datasource](/rest/api/searchservice/create-data-source) (Creación de origen de datos).

<a name="Credentials"></a>
#### <a name="ways-to-specify-credentials"></a>Formas de especificar las credenciales ####

Puede proporcionar las credenciales para la tabla de una de estas maneras: 

- **Cadena de conexión de la cuenta de almacenamiento de acceso completo**: `DefaultEndpointsProtocol=https;AccountName=<your storage account>;AccountKey=<your account key>` Para obtener la cadena de conexión de Azure Portal, vaya a la **hoja de la cuenta de almacenamiento** > **Configuración** > **Claves** (para las cuentas de almacenamiento del modelo clásico) o **Configuración** > **Claves de acceso** (para las cuentas de almacenamiento de Azure Resource Manager).
- **Cadena de conexión de la firma de acceso compartido de la cuenta de almacenamiento**: `TableEndpoint=https://<your account>.table.core.windows.net/;SharedAccessSignature=?sv=2016-05-31&sig=<the signature>&spr=https&se=<the validity end time>&srt=co&ss=t&sp=rl` La firma de acceso compartido debe tener permisos de enumeración y lectura sobre los contenedores (en este caso, tablas) y objetos (filas de tabla).
-  **Firma de acceso compartido de tabla**: `ContainerSharedAccessUri=https://<your storage account>.table.core.windows.net/<table name>?tn=<table name>&sv=2016-05-31&sig=<the signature>&se=<the validity end time>&sp=r` La firma de acceso compartido debería tener permisos de consulta (lectura) en la tabla.

Para más información sobre las firmas de acceso compartido de almacenamiento, consulte [Uso de firmas de acceso compartido (SAS)](../storage/common/storage-sas-overview.md).

> [!NOTE]
> Si usa credenciales de firma de acceso compartido, deberá actualizar las credenciales del origen de datos periódicamente con firmas renovadas para evitar que expiren. Si las credenciales de firma de acceso compartido expiran, el indexador produce un error con un mensaje parecido a este: "Las credenciales proporcionadas en la cadena de conexión no son válidas o han expirado".  

### <a name="step-2-create-an-index"></a>Paso 2: Creación de un índice
El índice especifica los campos de un documento, los atributos y otras construcciones que conforman la experiencia de búsqueda.

Para crear un índice:

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "key", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "SomeColumnInMyTable", "type": "Edm.String", "searchable": true }
          ]
    }
```

Para más información sobre la creación de índices, consulte [Create Index](/rest/api/searchservice/create-index) (Creación de un índice).

### <a name="step-3-create-an-indexer"></a>Paso 3: Creación de un indexador
Un indexador conecta un origen de datos con un índice de búsqueda de destino y proporciona una programación para automatizar la actualización de datos. 

Después de crear el origen de datos y el índice, ya puede crear el indexador:

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "table-indexer",
      "dataSourceName" : "table-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" }
    }
```

Este indexador se ejecuta cada dos horas. (El intervalo de programación se establece en "PT2H"). Para ejecutar un indizador cada 30 minutos, establézcalo en PT30M. El intervalo más breve que se admite es de cinco minutos. La programación es opcional: si se omite, el indexador solo se ejecuta una vez cuando se crea. Sin embargo, puede ejecutarlo a petición en cualquier momento.   

Para más información sobre la API de creación de indexador, consulte [Create Indexer](/rest/api/searchservice/create-indexer) (Creación de un indexador).

Para más información sobre cómo definir las programaciones del indexador, consulte [Programación de indexadores para Azure Cognitive Search](search-howto-schedule-indexers.md).

## <a name="deal-with-different-field-names"></a>Trabajo con distintos nombres de campos
A veces, los nombres de los campos de un índice existente no coincidirán con los nombres de las propiedades de la tabla. Puede usar asignaciones de campos para asignar los nombres de propiedad procedentes de la tabla a los nombres de campo del índice de búsqueda. Para aprender más sobre las asignaciones de campos, consulte [Las asignaciones de campos de indizador de Azure Cognitive Search salvan las diferencias entre los orígenes de datos y los índices de búsqueda](search-indexer-field-mappings.md).

## <a name="handle-document-keys"></a>Trabajo con claves de documento
En Azure Cognitive Search, la clave del documento identifica de forma exclusiva a un documento. Cada índice de búsqueda debe tener exactamente un campo clave de tipo `Edm.String`. El campo de clave es necesario para cada documento que se agrega al índice (de hecho, es el único campo obligatorio).

Como las filas de tablas tienen una clave compuesta, Azure Cognitive Search genera un campo sintético llamado `Key`, que es una concatenación de valores de clave de partición y clave de fila. Por ejemplo, si el valor PartitionKey de una fila es `PK1` y el valor RowKey es `RK1`, el valor del campo `Key` es `PK1RK1`.

> [!NOTE]
> El valor `Key` puede contener caracteres no válidos en claves de documentos, como guiones. Para tratar con caracteres no válidos, use la [función de asignación de campos](search-indexer-field-mappings.md#base64EncodeFunction)`base64Encode`. Si lo hace, recuerde utilizar también la codificación Base64 de seguridad de direcciones URL al pasar las claves de documento en las llamadas de la API como búsqueda.
>
>

## <a name="incremental-indexing-and-deletion-detection"></a>Indexación incremental y detección de eliminación
Cuando configure un indizador de tablas para que se ejecute según una programación, vuelva a indexar solo las filas nuevas o actualizadas, según lo determine el valor `Timestamp` de una fila. No tiene que especificar ninguna directiva de detección de cambios. La indexación incremental se habilita automáticamente.

Para indicar que determinados documentos se deben quitar del índice, puede usar una estrategia de eliminación temporal. En lugar de eliminar una fila, agregue una propiedad para indicar que se elimina y configure una directiva de detección de eliminaciones temporales en el origen de datos. Por ejemplo, la siguiente directiva considera que una fila se ha eliminado si tiene una propiedad `IsDeleted` con el valor `"true"`:

```http
    PUT https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
        "name" : "my-table-datasource",
        "type" : "azuretable",
        "credentials" : { "connectionString" : "<your storage connection string>" },
        "container" : { "name" : "table name", "query" : "<query>" },
        "dataDeletionDetectionPolicy" : { "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy", "softDeleteColumnName" : "IsDeleted", "softDeleteMarkerValue" : "true" }
    }   
```

<a name="Performance"></a>
## <a name="performance-considerations"></a>Consideraciones de rendimiento

De forma predeterminada, Azure Cognitive Search usa el siguiente filtro de consulta: `Timestamp >= HighWaterMarkValue`. Como las tablas de Azure no tienen un índice secundario en el campo `Timestamp`, este tipo de consulta requiere un examen de toda la tabla y, por tanto, es lento para tablas grandes.


Aquí hay dos posibles enfoques para mejorar el rendimiento de la indexación de tablas. Ambos se basan en el uso de particiones de tabla: 

- Si los datos se pueden dividir de forma natural en varios intervalos de partición, cree un origen de datos y un indexador correspondiente para cada intervalo de partición. Cada indexador tiene que procesar solo un intervalo de partición concreto, lo que mejora el rendimiento de las consultas. Si los datos que hay que indexar tienen un número pequeño de particiones fijas, aún mejor: cada indexador solo realiza el examen de una partición. Por ejemplo, para crear un origen de datos para procesar un intervalo de partición con las claves de `000` a `100`, utilice una consulta como esta: 
    ```
    "container" : { "name" : "my-table", "query" : "PartitionKey ge '000' and PartitionKey lt '100' " }
    ```

- Si los datos se dividen por tiempo (por ejemplo, se crea una nueva partición cada día o semana), considere el siguiente enfoque: 
    - Utilice una consulta con el formato: `(PartitionKey ge <TimeStamp>) and (other filters)`. 
    - Supervise el progreso del indexador mediante la [API de obtención del estado del indexador](/rest/api/searchservice/get-indexer-status) y actualice periódicamente la condición `<TimeStamp>` de la consulta según el valor alto correcto de la marca de agua más reciente. 
    - Con este enfoque, si tiene que desencadenar un reindexado completo, tendrá que restablecer la consulta del origen de datos, además de restablecer el indexador. 


## <a name="help-us-make-azure-cognitive-search-better"></a>Ayúdenos a mejorar Azure Cognitive Search
Si tiene solicitudes o ideas para mejorar las características, envíelas en el [sitio de UserVoice](https://feedback.azure.com/forums/263029-azure-search/).