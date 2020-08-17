---
title: Comandos de la extensión de MongoDB para administrar los datos en la API de Azure Cosmos DB para MongoDB
description: En este artículo se describen los comandos de la extensión de MongoDB para administrar los datos almacenados en la API de Azure Cosmos DB para MongoDB.
author: LuisBosquez
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/28/2020
ms.author: lbosq
ms.custom: devx-track-javascript
ms.openlocfilehash: 7b0ac1e301705b24d706638deb3ee0a15d49c87b
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87415098"
---
# <a name="use-mongodb-extension-commands-to-manage-data-stored-in-azure-cosmos-dbs-api-for-mongodb"></a>Usa comandos de la extensión de MongoDB para administrar los datos almacenados en la API de Azure Cosmos DB para MongoDB 

El siguiente documento contiene los comandos de acción personalizada que son específicos de la API de Azure Cosmos DB para MongoDB. Estos comandos se pueden usar para crear y obtener recursos de base de datos específicos del [modelo de capacidad de Azure Cosmos DB](databases-containers-items.md).

Mediante las API de Azure Cosmos DB para MongoDB, puede disfrutar de las ventajas de Cosmos DB, como distribución global, particionamiento automático, alta disponibilidad, garantías de latencia, cifrado en reposo automático, copias de seguridad, y muchas más, a la vez que conserva su inversión en la aplicación de MongoDB. Puede comunicarse con la API de Azure Cosmos DB para MongoDB con cualquiera de los [controladores de cliente de MongoDB](https://docs.mongodb.org/ecosystem/drivers) de código abierto. La API de Azure Cosmos DB para MongoDB permite usar los controladores de cliente existentes mediante la adhesión al [protocolo de conexión de MongoDB](https://docs.mongodb.org/manual/reference/mongodb-wire-protocol).

## <a name="mongodb-protocol-support"></a>Compatibilidad de protocolo para MongoDB

La API de Azure Cosmos DB para MongoDB es compatible de manera predeterminada con la versión 3.2 y 3.6 del servidor de MongoDB. Consulte la [sintaxis y las características admitidas](mongodb-feature-support.md) para obtener más detalles. 

Los siguientes comandos de extensión proporcionan la capacidad de crear y modificar recursos específicos de Azure Cosmos DB a través de solicitudes de base de datos:

* [Crear una base de datos](#create-database)
* [Actualizar una base de datos](#update-database)
* [Obtener una base de datos](#get-database)
* [Crear una colección](#create-collection)
* [Actualizar una colección](#update-collection)
* [Obtener una colección](#get-collection)

## <a name="create-database"></a><a id="create-database"></a> Crear una base de datos

El comando de extensión de creación de base de datos crea una nueva base de datos de MongoDB. El nombre de la base de datos se puede usar desde el contexto de base de datos establecido por el comando `use database`. En la siguiente tabla se describen los parámetros del comando:

|**Campo**|**Tipo** |**Descripción** |
|---------|---------|---------|
| `customAction`   |  `string`  |   Nombre del comando personalizado, debe ser "CreateDatabase".      |
| `offerThroughput` | `int`  | Rendimiento aprovisionado que establece en la base de datos. Este parámetro es opcional. |
| `autoScaleSettings` | `Object` | Se requiere para el [modo de escalabilidad automática](provision-throughput-autoscale.md). Este objeto contiene la configuración asociada al modo de capacidad de escalabilidad automática. Puede configurar el valor de `maxThroughput`, que describe la mayor cantidad de unidades de solicitud en las que se aumentará la colección de forma dinámica. |

### <a name="output"></a>Output

Si el comando se ejecuta correctamente, devolverá la siguiente respuesta:

```javascript
{ "ok" : 1 }
```

Consulte la [salida predeterminada](#default-output) del comando personalizado para los parámetros en la salida.

### <a name="examples"></a>Ejemplos

#### <a name="create-a-database"></a>Crear una base de datos

Para crear una base de datos denominada `"test"` que use todos los valores predeterminados, use el siguiente comando:

```javascript
use test
db.runCommand({customAction: "CreateDatabase"});
```

Este comando crea una base de datos sin rendimiento en el nivel de base de datos. Esto significa que las colecciones de esta base de datos tendrán que especificar la cantidad de rendimiento que se debe usar.

#### <a name="create-a-database-with-throughput"></a>Creación de una base de datos con rendimiento

Para crear una base de datos denominada `"test"` y especificar un rendimiento aprovisionado en el [nivel de base de datos](set-throughput.md#set-throughput-on-a-database) de 1000 RU/s, use el siguiente comando:

```javascript
use test
db.runCommand({customAction: "CreateDatabase", offerThroughput: 1000 });
```

Se creará una base de datos y se establecerá un rendimiento en ella. Todas las colecciones de esta base de datos compartirán el rendimiento establecido, a menos que las colecciones se creen con [un nivel de rendimiento específico](set-throughput.md#set-throughput-on-a-database-and-a-container).

#### <a name="create-a-database-with-autoscale-throughput"></a>Creación de una base de datos con rendimiento de escalabilidad automática

Para crear una base de datos denominada `"test"` y especificar un rendimiento máximo de escalabilidad automática de 20 000 RU/s en el [nivel de base de datos](set-throughput.md#set-throughput-on-a-database), use el siguiente comando:

```javascript
use test
db.runCommand({customAction: "CreateDatabase", autoScaleSettings: { maxThroughput: 20000 } });
```

## <a name="update-database"></a><a id="update-database"></a> Actualizar una base de datos

El comando de extensión de actualización de la base de datos actualiza las propiedades asociadas con la base de datos especificada. En la siguiente tabla se describen los parámetros del comando:

|**Campo**|**Tipo** |**Descripción** |
|---------|---------|---------|
| `customAction`    |    `string`     |   Nombre del comando personalizado. Debe ser "UpdateDatabase".      |
|  `offerThroughput`   |  `int`       |     Nuevo rendimiento aprovisionado que desea establecer en la base de datos si esta utiliza un [rendimiento en el nivel de base de datos](set-throughput.md#set-throughput-on-a-database)  |
| `autoScaleSettings` | `Object` | Se requiere para el [modo de escalabilidad automática](provision-throughput-autoscale.md). Este objeto contiene la configuración asociada al modo de capacidad de escalabilidad automática. Puede configurar el valor de `maxThroughput`, que describe la mayor cantidad de unidades de solicitud en las que se aumentará la base de datos de forma dinámica. |

Este comando usa la base de datos especificada en el contexto de la sesión. Esta es la base de datos usada en el comando `use <database>`. En este momento, no se puede cambiar el nombre de la base de datos con este comando.

### <a name="output"></a>Output

Si el comando se ejecuta correctamente, devolverá la siguiente respuesta:

```javascript
{ "ok" : 1 }
```

Consulte la [salida predeterminada](#default-output) del comando personalizado para los parámetros en la salida.

### <a name="examples"></a>Ejemplos

#### <a name="update-the-provisioned-throughput-associated-with-a-database"></a>Actualización del rendimiento aprovisionado asociado con una base de datos

Para actualizar el rendimiento aprovisionado de una base de datos con nombre `"test"` a 1200 RU/s, use el siguiente comando:

```javascript
use test
db.runCommand({customAction: "UpdateDatabase", offerThroughput: 1200 });
```

#### <a name="update-the-autoscale-throughput-associated-with-a-database"></a>Actualización del rendimiento de escalabilidad automática asociado con una base de datos

Para actualizar el rendimiento aprovisionado de una base de datos con el nombre `"test"` a 20 000 RU/s, o para transformarla en un [nivel de rendimiento de escalabilidad automática](provision-throughput-autoscale.md), use el siguiente comando:

```javascript
use test
db.runCommand({customAction: "UpdateDatabase", autoScaleSettings: { maxThroughput: 20000 } });
```


## <a name="get-database"></a><a id="get-database"></a> Obtener una base de datos

El comando de extensión de obtención de una base de datos devuelve el objeto de base de datos. El nombre de la base de datos se usa desde el contexto de la base de datos donde se ejecuta el comando.

```javascript
{
  customAction: "GetDatabase"
}
```

En la siguiente tabla se describen los parámetros del comando:


|**Campo**|**Tipo** |**Descripción** |
|---------|---------|---------|
|  `customAction`   |   `string`      |   Nombre del comando personalizado. Debe ser "GetDatabase"|
        
### <a name="output"></a>Output

Si el comando se ejecuta correctamente, la respuesta contiene un documento con los siguientes campos:

|**Campo**|**Tipo** |**Descripción** |
|---------|---------|---------|
|  `ok`   |   `int`     |   Estado de la respuesta. 1 == correcto. 0 == erróneo.      |
| `database`    |    `string`        |   Nombre de la base de datos.      |
|   `provisionedThroughput`  |    `int`      |    Rendimiento aprovisionado que desea establecer en la base de datos si esta utiliza un [rendimiento manual en el nivel de base de datos](set-throughput.md#set-throughput-on-a-database)     |
| `autoScaleSettings` | `Object` | Este objeto contiene los parámetros de capacidad asociados a la base de datos si usa el [modo de escalabilidad automática](provision-throughput-autoscale.md). El valor de `maxThroughput` describe la mayor cantidad de unidades de solicitud en las que se aumentará la base de datos de forma dinámica. |

Si no se puede ejecutar el comando, se devuelve una respuesta de comando personalizado de forma predeterminada. Consulte la [salida predeterminada](#default-output) del comando personalizado para los parámetros en la salida.

### <a name="examples"></a>Ejemplos

#### <a name="get-the-database"></a>Obtención de la base de datos

Con el fin de crear el objeto de base de datos para una base de datos denominada `"test"`, use el comando siguiente:

```javascript
use test
db.runCommand({customAction: "GetDatabase"});
```

Si la base de datos no tiene ningún rendimiento asociado, el resultado sería:

```javascript
{ "database" : "test", "ok" : 1 }
```

Si la base de datos tiene un [rendimiento manual en el nivel de base datos](set-throughput.md#set-throughput-on-a-database) asociado a ella, la salida mostraría los valores de `provisionedThroughput`:

```javascript
{ "database" : "test", "provisionedThroughput" : 20000, "ok" : 1 }
```

Si la base de datos tiene un [rendimiento de escalabilidad automática en el nivel de base de datos](provision-throughput-autoscale.md) asociado a ella, la salida mostraría `provisionedThroughput`, que describe la cantidad mínima de RU/s para la base de datos, y el objeto `autoScaleSettings`, incluido `maxThroughput`, que describe el valor de RU/s máximo para la base de datos.

```javascript
{
        "database" : "test",
        "provisionedThroughput" : 2000,
        "autoScaleSettings" : {
                "maxThroughput" : 20000
        },
        "ok" : 1
}
```

## <a name="create-collection"></a><a id="create-collection"></a> Crear una colección

El comando de extensión de creación de colección crea una nueva colección de MongoDB. El nombre de la base de datos se usa desde el contexto de base de datos establecido por el comando `use database`. El formato del comando CreateCollection es el siguiente:

```javascript
{
  customAction: "CreateCollection",
  collection: "<Collection Name>",
  shardKey: "<Shard key path>",
  offerThroughput: (int), // Amount of throughput allocated to a specific collection

}
```

En la siguiente tabla se describen los parámetros del comando:

| **Campo** | **Tipo** | **Obligatorio** | **Descripción** |
|---------|---------|---------|---------|
| `customAction` | `string` | Obligatorio | Nombre del comando personalizado. Debe ser "CreateCollection".|
| `collection` | `string` | Obligatorio | Nombre de la colección. No se permiten caracteres especiales ni espacios.|
| `offerThroughput` | `int` | Opcional | Rendimiento aprovisionado para establecer en la base de datos. Si no se proporciona este parámetro, el valor predeterminado será el mínimo, 400 RU/s. *Para especificar el rendimiento más allá de 10 000 RU/s, se requiere el parámetro `shardKey`.|
| `shardKey` | `string` | Se requiere para las colecciones de gran rendimiento | La ruta de acceso a la clave de partición para la colección con particiones. Este parámetro es necesario si establece más de 10 000 RU/s en `offerThroughput`.  Si se especifica, todos los documentos insertados requerirán este valor y esta clave. |
| `autoScaleSettings` | `Object` | Se requiere para el [modo de escalabilidad automática](provision-throughput-autoscale.md). | Este objeto contiene la configuración asociada al modo de capacidad de escalabilidad automática. Puede configurar el valor de `maxThroughput`, que describe la mayor cantidad de unidades de solicitud en las que se aumentará la colección de forma dinámica. |

### <a name="output"></a>Output

Devuelve una respuesta predeterminada del comando personalizado. Consulte la [salida predeterminada](#default-output) del comando personalizado para los parámetros en la salida.

### <a name="examples"></a>Ejemplos

#### <a name="create-a-collection-with-the-minimum-configuration"></a>Creación de una colección con la configuración mínima

Para crear una colección con el nombre `"testCollection"` y los valores predeterminados, use el siguiente comando: 

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection"});
```

Se generará una colección fija, sin particiones, con 400 RU/s y un índice en el campo `_id` creado automáticamente. Este tipo de configuración también se aplicará al crear colecciones a través de la función `insert()`. Por ejemplo: 

```javascript
use test
db.newCollection.insert({});
```

#### <a name="create-a-unsharded-collection"></a>Creación de una colección sin particiones

Para crear una colección sin particiones con el nombre `"testCollection"` y el rendimiento aprovisionado de 1000 RU/s, use el siguiente comando: 

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection", offerThroughput: 1000});
``` 

Puede crear una colección de hasta 10 000 RU/s como `offerThroughput` sin necesidad de especificar una clave de partición. En el caso de colecciones con mayor rendimiento, consulte la sección siguiente.

#### <a name="create-a-sharded-collection"></a>Creación de una colección con particiones

Para crear una colección con particiones con el nombre `"testCollection"` y el rendimiento aprovisionado de 11 000 RU/s, y una propiedad `shardkey` "a.b", use el siguiente comando:

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection", offerThroughput: 11000, shardKey: "a.b" });
```

Este comando requiere ahora el parámetro `shardKey`, ya que se han especificado más de 10 000 RU/s en `offerThroughput`.

#### <a name="create-an-unsharded-autoscale-collection"></a>Creación de una colección de escalabilidad automática sin particiones

Para crear una colección sin particiones denominada `'testCollection'` que use la [capacidad de rendimiento de escalabilidad automática](provision-throughput-autoscale.md) establecida en 4000 RU/s, use el siguiente comando:

```javascript
use test
db.runCommand({ 
    customAction: "CreateCollection", collection: "testCollection", 
    autoScaleSettings:{
      maxThroughput: 4000
    } 
});
```

En el valor de `autoScaleSettings.maxThroughput` puede especificar un intervalo de 4000 RU/s a 10 000 RU/s sin una clave de partición. Para un mayor rendimiento de escalabilidad automática, debe especificar el parámetro `shardKey`.

#### <a name="create-a-sharded-autoscale-collection"></a>Creación de una colección de escalabilidad automática con particiones

Para crear una colección con particiones denominada `'testCollection'` con una clave de partición denominada `'a.b'` y que use la [capacidad de rendimiento de escalabilidad automática](provision-throughput-autoscale.md) establecida en 20 000 RU/s, use el siguiente comando:

```javascript
use test
db.runCommand({customAction: "CreateCollection", collection: "testCollection", shardKey: "a.b", autoScaleSettings: { maxThroughput: 20000 }});
```

## <a name="update-collection"></a><a id="update-collection"></a> Actualizar una colección

El comando de extensión de actualización de la colección actualiza las propiedades asociadas con la colección especificada.

```javascript
{
  customAction: "UpdateCollection",
  collection: "<Name of the collection that you want to update>",
  offerThroughput: (int) // New throughput that will be set to the collection
}
```

En la siguiente tabla se describen los parámetros del comando:

|**Campo**|**Tipo** |**Descripción** |
|---------|---------|---------|
|  `customAction`   |   `string`      |   Nombre del comando personalizado. Debe ser "UpdateCollection".      |
|  `collection`   |   `string`      |   Nombre de la colección.       |
| `offerThroughput` | `int` |   Rendimiento aprovisionado para establecer en la colección.|
| `autoScaleSettings` | `Object` | Se requiere para el [modo de escalabilidad automática](provision-throughput-autoscale.md). Este objeto contiene la configuración asociada al modo de capacidad de escalabilidad automática. El valor de `maxThroughput` describe la mayor cantidad de unidades de solicitud en las que se aumentará la colección de forma dinámica. |

## <a name="output"></a>Output

Devuelve una respuesta predeterminada del comando personalizado. Consulte la [salida predeterminada](#default-output) del comando personalizado para los parámetros en la salida.

### <a name="examples"></a>Ejemplos

#### <a name="update-the-provisioned-throughput-associated-with-a-collection"></a>Actualización del rendimiento aprovisionado asociado con una colección

Para actualizar el rendimiento aprovisionado de una colección con nombre `"testCollection"` a 1200 RU/s, use el siguiente comando:

```javascript
use test
db.runCommand({customAction: "UpdateCollection", collection: "testCollection", offerThroughput: 1200 });
```

## <a name="get-collection"></a><a id="get-collection"></a> Obtener una colección

El comando personalizado de obtención de una colección devuelve el objeto de colección.

```javascript
{
  customAction: "GetCollection",
  collection: "<Name of the collection>"
}
```

En la siguiente tabla se describen los parámetros del comando:


|**Campo**|**Tipo** |**Descripción** |
|---------|---------|---------|
| `customAction`    |   `string`      |   Nombre del comando personalizado. Debe ser "GetCollection".      |
| `collection`    |    `string`     |    Nombre de la colección.     |

### <a name="output"></a>Output

Si el comando se ejecuta correctamente, la respuesta contiene un documento con los siguientes campos:


|**Campo**|**Tipo** |**Descripción** |
|---------|---------|---------|
|  `ok`   |    `int`     |   Estado de la respuesta. 1 == correcto. 0 == erróneo.      |
| `database`    |    `string`     |   Nombre de la base de datos.      |
| `collection`    |    `string`     |    Nombre de la colección.     |
|  `shardKeyDefinition`   |   `document`      |  Documento de especificaciones de índice que se usa como clave de partición. Se trata de un parámetro opcional de respuesta.       |
|  `provisionedThroughput`   |   `int`      |    Rendimiento aprovisionado para establecer en la colección. Se trata de un parámetro opcional de respuesta.     |
| `autoScaleSettings` | `Object` | Este objeto contiene los parámetros de capacidad asociados a la base de datos si usa el [modo de escalabilidad automática](provision-throughput-autoscale.md). El valor de `maxThroughput` describe la mayor cantidad de unidades de solicitud en las que se aumentará la colección de forma dinámica. |

Si no se puede ejecutar el comando, se devuelve una respuesta de comando personalizado de forma predeterminada. Consulte la [salida predeterminada](#default-output) del comando personalizado para los parámetros en la salida.

### <a name="examples"></a>Ejemplos

#### <a name="get-the-collection"></a>Obtención de la colección

Para obtener el objeto de colección para una colección denominada `"testCollection"`, use el comando siguiente:

```javascript
use test
db.runCommand({customAction: "GetCollection", collection: "testCollection"});
```

Si la colección tiene una capacidad de rendimiento asociada, incluirá el valor de `provisionedThroughput` y el resultado sería:

```javascript
{
        "database" : "test",
        "collection" : "testCollection",
        "provisionedThroughput" : 400,
        "ok" : 1
}
```

Si la colección tiene un rendimiento de escalabilidad automática asociado, incluirá el objeto `autoScaleSettings` con el parámetro `maxThroughput`, que define el rendimiento máximo al que se aumentará la colección de forma dinámica. Además, incluirá el valor de `provisionedThroughput`, que define el rendimiento mínimo que al que se reducirá la colección de forma dinámica si no hay solicitudes en la colección: 

```javascript
{
        "database" : "test",
        "collection" : "testCollection",
        "provisionedThroughput" : 1000,
        "autoScaleSettings" : {
            "maxThroughput" : 10000
        },
        "ok" : 1
}
```

Si la colección está compartiendo [rendimiento en el nivel de base de datos](set-throughput.md#set-throughput-on-a-database), ya sea en el modo de escalabilidad automática o manual, el resultado sería:

```javascript
{ "database" : "test", "collection" : "testCollection", "ok" : 1 }
```

```javascript
{
        "database" : "test",
        "provisionedThroughput" : 2000,
        "autoScaleSettings" : {
            "maxThroughput" : 20000
        },
        "ok" : 1
}
```


## <a name="default-output-of-a-custom-command"></a><a id="default-output"></a> Salida predeterminada de un comando personalizado

Si no se especifica, la respuesta personalizada contiene un documento con los siguientes campos:

|**Campo**|**Tipo** |**Descripción** |
|---------|---------|---------|
|  `ok`   |    `int`     |   Estado de la respuesta. 1 == correcto. 0 == erróneo.      |
| `code`    |   `int`      |   Solo se devuelve cuando el comando es erróneo (es decir, ok == 0). Contiene el código de error de MongoDB. Se trata de un parámetro opcional de respuesta.      |
|  `errMsg`   |  `string`      |    Solo se devuelve cuando el comando es erróneo (es decir, ok == 0). Contiene un mensaje de error descriptivo. Se trata de un parámetro opcional de respuesta.      |

Por ejemplo:

```javascript
{ "ok" : 1 }
```

## <a name="next-steps"></a>Pasos siguientes

Luego, puede obtener información sobre los siguientes conceptos de Azure Cosmos DB: 

* [Indexación en Azure Cosmos DB](../cosmos-db/index-policy.md)
* [Expiración automática de los datos de Azure Cosmos DB con período de vida](../cosmos-db/time-to-live.md)
