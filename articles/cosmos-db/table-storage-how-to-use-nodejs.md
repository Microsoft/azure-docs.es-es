---
title: Uso de Azure Table Storage y Table API de Azure Cosmos DB desde Node.js
description: Almacene datos estructurados en la nube mediante Azure Table Storage o Table API de Azure Cosmos DB.
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.devlang: nodejs
ms.topic: sample
ms.date: 07/23/2020
author: sakash279
ms.author: akshanka
ms.custom: devx-track-javascript
ms.openlocfilehash: cfcb5645a6284214e233758705537486f32967c6
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88079304"
---
# <a name="how-to-use-azure-table-storage-or-the-azure-cosmos-db-table-api-from-nodejs"></a>Uso de Azure Table Storage o Table API de Azure Cosmos DB desde Node.js

[!INCLUDE [storage-selector-table-include](../../includes/storage-selector-table-include.md)]
[!INCLUDE [storage-table-applies-to-storagetable-and-cosmos](../../includes/storage-table-applies-to-storagetable-and-cosmos.md)]

En este artículo se muestra cómo crear tablas, y almacenar los datos y realizar operaciones CRUD en ellos. Elija usar el servicio Azure Table Storage o Table API de Azure Cosmos DB. Los ejemplos están escritos en Node.js.

## <a name="create-an-azure-service-account"></a>Creación de una cuenta de servicio de Azure

[!INCLUDE [cosmos-db-create-azure-service-account](../../includes/cosmos-db-create-azure-service-account.md)]

**Creación de una cuenta de Azure Storage**

[!INCLUDE [cosmos-db-create-storage-account](../../includes/cosmos-db-create-storage-account.md)]

**Creación de una cuenta de Table API de Azure Cosmos DB**

[!INCLUDE [cosmos-db-create-tableapi-account](../../includes/cosmos-db-create-tableapi-account.md)]

## <a name="configure-your-application-to-access-azure-storage-or-the-azure-cosmos-db-table-api"></a>Configurar la aplicación para acceder a Azure Storage o a Table API de Azure Cosmos DB

Para usar Azure Storage o Azure Cosmos DB necesitará Azure Storage SDK para Node.js, que incluye un conjunto de útiles bibliotecas que se comunican con los servicios REST de Storage.

### <a name="use-node-package-manager-npm-to-install-the-package"></a>Uso del Administrador de paquetes para Node (NPM) para instalar el paquete

1. Use una interfaz de línea de comandos como **PowerShell** (Windows), **Terminal** (Mac) o **Bash** (Unix) y vaya a la carpeta donde creó la aplicación.
2. Escriba **npm install azure-storage** en la ventana de comandos. La salida del comando es similar al ejemplo siguiente.

   ```bash
    azure-storage@0.5.0 node_modules\azure-storage
    +-- extend@1.2.1
    +-- xmlbuilder@0.4.3
    +-- mime@1.2.11
    +-- node-uuid@1.4.3
    +-- validator@3.22.2
    +-- underscore@1.4.4
    +-- readable-stream@1.0.33 (string_decoder@0.10.31, isarray@0.0.1, inherits@2.0.1, core-util-is@1.0.1)
    +-- xml2js@0.2.7 (sax@0.5.2)
    +-- request@2.57.0 (caseless@0.10.0, aws-sign2@0.5.0, forever-agent@0.6.1, stringstream@0.0.4, oauth-sign@0.8.0, tunnel-agent@0.4.1, isstream@0.1.2, json-stringify-safe@5.0.1, bl@0.9.4, combined-stream@1.0.5, qs@3.1.0, mime-types@2.0.14, form-data@0.2.0, http-signature@0.11.0, tough-cookie@2.0.0, hawk@2.3.1, har-validator@1.8.0)
   ```

3. Puede ejecutar manualmente el comando **ls** para comprobar si se ha creado una carpeta **node_modules**. Dentro de dicha carpeta, encontrará el paquete **azure-storage** , que contiene las bibliotecas necesarias para el acceso al almacenamiento.

### <a name="import-the-package"></a>Importación del paquete

Agregue el código siguiente a la parte superior del archivo **server.js** de la aplicación:

```javascript
var azure = require('azure-storage');
```

## <a name="add-your-connection-string"></a>Incorporación de la cadena de conexión

Puede conectarse a la cuenta de almacenamiento de Azure o a la de Table API de Azure Cosmos DB. Obtenga la cadena de conexión en función del tipo de cuenta que esté usando.

### <a name="add-an-azure-storage-connection"></a>Incorporación de una conexión de Azure Storage

El módulo de Azure lee las variables de entorno AZURE_STORAGE_ACCOUNT y AZURE_STORAGE_ACCESS_KEY o AZURE_STORAGE_CONNECTION_STRING para la información necesaria para conectarse a su cuenta de Azure Storage. Si no se configuran estas variables de entorno, debe especificar la información de la cuenta al llamar a `TableService`. Por ejemplo, el código siguiente crea un objeto `TableService`:

```javascript
var tableSvc = azure.createTableService('myaccount', 'myaccesskey');
```

### <a name="add-an-azure-cosmos-db-connection"></a>Adición de una conexión de Azure Cosmos DB

Para agregar una conexión de Azure Cosmos DB, cree un objeto `TableService` y especifique el nombre de la cuenta, la clave principal y el punto de conexión. Puede copiar estos valores desde **Configuración** > **Cadena de conexión** en Azure Portal para la cuenta de Cosmos DB. Por ejemplo:

```javascript
var tableSvc = azure.createTableService('myaccount', 'myprimarykey', 'myendpoint');
```

## <a name="create-a-table"></a>Creación de una tabla

El código siguiente crea un objeto `TableService` y lo usa para crear una tabla.

```javascript
var tableSvc = azure.createTableService();
```

La llamada a `createTableIfNotExists` crea una tabla nueva con el nombre especificado si todavía no existe. El ejemplo siguiente crea una tabla llamada "mytable", si es que no existe todavía:

```javascript
tableSvc.createTableIfNotExists('mytable', function(error, result, response){
  if(!error){
    // Table exists or created
  }
});
```

El `result.created` es `true` si se crea una nueva tabla y `false` si la tabla ya existe. La `response` contiene información sobre la solicitud.

### <a name="apply-filters"></a>Aplicación de filtros

Solo puede aplicar el filtrado opcional a las operaciones realizadas mediante `TableService`. Las operaciones de filtrado pueden incluir registros, reintentos automáticos, etc. Los filtros son objetos que implementan un método con la firma:

```javascript
function handle (requestOptions, next)
```

Después de realizar el preprocesamiento en las opciones de solicitud, el método tiene que llamar a **next** pasando una devolución de llamada con la firma siguiente:

```javascript
function (returnObject, finalCallback, next)
```

En esta devolución de llamada y después de procesar `returnObject` (la respuesta de la solicitud al servidor), la devolución de llamada tiene que invocar a `next`, si existe, para continuar procesando otros filtros, o bien simplemente invocar a `finalCallback` para finalizar la invocación del servicio.

Con el SDK de Azure, se incluyen dos filtros que implementan la lógica de reintento para Node.js, `ExponentialRetryPolicyFilter` y `LinearRetryPolicyFilter`. El siguiente código crea un objeto `TableService` que usa `ExponentialRetryPolicyFilter`:

```javascript
var retryOperations = new azure.ExponentialRetryPolicyFilter();
var tableSvc = azure.createTableService().withFilter(retryOperations);
```

## <a name="add-an-entity-to-a-table"></a>Adición de una entidad a una tabla

Para agregar una entidad, primero cree un objeto que defina las propiedades de la entidad. Todas las entidades tienen que contener un valor para **PartitionKey** y **RowKey**, que son identificadores únicos de la entidad.

* **PartitionKey**: determina la partición en la que se almacena la entidad.
* **RowKey**: identifica de manera única la entidad dentro de la partición.

Tanto **PartitionKey** como **RowKey** tiene que ser valores de cadena. Para obtener más información, consulte [Descripción del modelo de datos del servicio Tabla](https://msdn.microsoft.com/library/azure/dd179338.aspx).

Este es un ejemplo de la definición de una entidad. **dueDate** se define como un tipo de `Edm.DateTime`. La especificación del tipo es opcional y los tipos se deducen si no se especifican.

```javascript
var task = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '1'},
  description: {'_':'take out the trash'},
  dueDate: {'_':new Date(2015, 6, 20), '$':'Edm.DateTime'}
};
```

> [!NOTE]
> Hay también un campo `Timestamp` para cada registro; Azure lo establece cuando se inserta o actualiza una entidad.

También puede usar `entityGenerator` para crear entidades. En el siguiente ejemplo se crea la misma entidad de tarea mediante `entityGenerator`.

```javascript
var entGen = azure.TableUtilities.entityGenerator;
var task = {
  PartitionKey: entGen.String('hometasks'),
  RowKey: entGen.String('1'),
  description: entGen.String('take out the trash'),
  dueDate: entGen.DateTime(new Date(Date.UTC(2015, 6, 20))),
};
```

Para agregar una entidad a la tabla, pase el objeto de la entidad al método `insertEntity`.

```javascript
tableSvc.insertEntity('mytable',task, function (error, result, response) {
  if(!error){
    // Entity inserted
  }
});
```

Si la operación se realiza correctamente, `result` contiene la etiqueta [ETag](https://en.wikipedia.org/wiki/HTTP_ETag) del registro insertado y `response` contiene información sobre la operación.

Respuesta de ejemplo:

```javascript
{ '.metadata': { etag: 'W/"datetime\'2015-02-25T01%3A22%3A22.5Z\'"' } }
```

> [!NOTE]
> De forma predeterminada, el elemento `insertEntity` no devuelve la entidad insertada como parte de la información de `response`. Si tiene pensado realizar otras operaciones en esta entidad o desea almacenar en caché la información, puede ser útil que la devuelva como parte de `result`. Para ello, puede habilitar `echoContent` de la manera siguiente:
>
> `tableSvc.insertEntity('mytable', task, {echoContent: true}, function (error, result, response) {...}`

## <a name="update-an-entity"></a>Actualización de una entidad

Hay varios métodos para actualizar una entidad existente:

* `replaceEntity`: reemplaza una entidad existente para actualizarla.
* `mergeEntity`: combina los valores de propiedad nuevos en la entidad existente para actualizarla.
* `insertOrReplaceEntity`: reemplaza una entidad existente para actualizarla. Si no hay entidades, se insertará una nueva.
* `insertOrMergeEntity`: combina los valores de propiedad nuevos en la entidad existente para actualizarla. Si no hay entidades, se insertará una nueva.

El ejemplo siguiente demuestra cómo actualizar una entidad usando `replaceEntity`:

```javascript
tableSvc.replaceEntity('mytable', updatedTask, function(error, result, response){
  if(!error) {
    // Entity updated
  }
});
```

> [!NOTE]
> De manera predeterminada, al actualizar una entidad no se comprueba si otro proceso ha modificado anteriormente los datos que se actualizan. Para permitir las actualizaciones simultáneas:
>
> 1. Obtenga la etiqueta ETag del objeto que se va a actualizar. Esta se devuelve como parte del valor de `response` para cualquier operación relacionada con entidades y se puede recuperar a través de `response['.metadata'].etag`.
> 2. Al realizar una operación de actualización en una entidad, agregue la información de ETag anteriormente recuperada a la nueva entidad. Por ejemplo:
>
>       entity2['.metadata'].etag = currentEtag;
> 3. Realice la operación de actualización. Si la entidad se modificó desde que recuperara el valor de ETag, como por ejemplo, otra instancia de la aplicación, se devuelve un `error` indicando que la condición de actualización especificada en la solicitud no se ha satisfecho.
>
>

Con `replaceEntity` y `mergeEntity`, si la entidad que se va a actualizar no existe, se produce un error en la operación. Por lo tanto, si desea almacenar una entidad independientemente de si ya existe, use `insertOrReplaceEntity` o `insertOrMergeEntity`.

El `result` de operaciones de actualización correctas contiene la etiqueta **Etag** de la entidad actualizada.

## <a name="work-with-groups-of-entities"></a>Trabajo con grupos de entidades

A veces resulta útil enviar varias operaciones juntas en un lote a fin de garantizar el procesamiento atómico por parte del servidor. Para ello, use la clase `TableBatch` para crear un lote y el método `executeBatch` de `TableService` para realizar las operaciones por lotes.

 El ejemplo siguiente demuestra cómo enviar dos entidades en un lote:

```javascript
var task1 = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '1'},
  description: {'_':'Take out the trash'},
  dueDate: {'_':new Date(2015, 6, 20)}
};
var task2 = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '2'},
  description: {'_':'Wash the dishes'},
  dueDate: {'_':new Date(2015, 6, 20)}
};

var batch = new azure.TableBatch();

batch.insertEntity(task1, {echoContent: true});
batch.insertEntity(task2, {echoContent: true});

tableSvc.executeBatch('mytable', batch, function (error, result, response) {
  if(!error) {
    // Batch completed
  }
});
```

En las operaciones por lotes realizadas correctamente, `result` contiene información de cada operación del lote.

### <a name="work-with-batched-operations"></a>Trabajar con operaciones por lotes

Puede inspeccionar las operaciones agregadas a un lote mediante la visualización de la propiedad `operations`. También se pueden usar los siguientes métodos para trabajar con operaciones.

* **clear**: borra todas las operaciones de un lote.
* **getOperations**: obtiene una operación del lote.
* **hasOperations**: devuelve true si el lote contiene operaciones.
* **removeOperations**: quita una operación.
* **size**: devuelve el número de operaciones del lote.

## <a name="retrieve-an-entity-by-key"></a>Recuperación de una entidad por clave

Para devolver una entidad específica basada en los valores de **PartitionKey** y **RowKey**, use el método **retrieveEntity**.

```javascript
tableSvc.retrieveEntity('mytable', 'hometasks', '1', function(error, result, response){
  if(!error){
    // result contains the entity
  }
});
```

Después de completar esta operación, `result` contiene la entidad.

## <a name="query-a-set-of-entities"></a>Consulta de un conjunto de entidades

Para consultar una tabla, use el objeto **TableQuery** para compilar una expresión de consulta mediante las siguientes cláusulas:

* **select**: son los campos que va a devolver la consulta.
* **where**: la cláusula where.

  * **and**: es una condición where `and`.
  * **or**: es una condición where `or`.
* **top**: es el número de elementos que se obtendrán.

En el siguiente ejemplo se crea una consulta que devuelve los cinco elementos principales con un valor de PartitionKey de "hometasks".

```javascript
var query = new azure.TableQuery()
  .top(5)
  .where('PartitionKey eq ?', 'hometasks');
```

Dado que **select** no se usa, se devuelven todos los campos. Para realizar la consulta en una tabla, use **queryEntities**. En el siguiente ejemplo se usa esta consulta para devolver entidades de 'mytable'.

```javascript
tableSvc.queryEntities('mytable',query, null, function(error, result, response) {
  if(!error) {
    // query was successful
  }
});
```

Si la operación se realiza correctamente, `result.entries` contiene una matriz de entidades que coinciden con la consulta. Si la consulta no puede devolver todas las entidades, `result.continuationToken` no es *null* y se puede usar como el tercer parámetro de **queryEntities** para recuperar más resultados. Para la consulta inicial, use *null* para el tercer parámetro.

### <a name="query-a-subset-of-entity-properties"></a>Consulta de un subconjunto de propiedades de las entidades

Una consulta de tabla puede recuperar solo algunos campos de una entidad.
Esto reduce el ancho de banda y puede mejorar el rendimiento de las consultas, en especial en el caso de entidades de gran tamaño. Use la cláusula **select** y pase los nombres de los campos que se van a devolver. Por ejemplo, la siguiente consulta solo devuelve los campos **description** y **dueDate**.

```javascript
var query = new azure.TableQuery()
  .select(['description', 'dueDate'])
  .top(5)
  .where('PartitionKey eq ?', 'hometasks');
```

## <a name="delete-an-entity"></a>Eliminación de una entidad

Puede eliminar una entidad usando sus claves de partición y fila. En este ejemplo, el objeto **task1** contiene los valores **RowKey** y **PartitionKey** de la entidad que se va a eliminar. A continuación, el objeto pasa al método **deleteEntity** .

```javascript
var task = {
  PartitionKey: {'_':'hometasks'},
  RowKey: {'_': '1'}
};

tableSvc.deleteEntity('mytable', task, function(error, response){
  if(!error) {
    // Entity deleted
  }
});
```

> [!NOTE]
> Cuando elimine elementos, debería considerar el uso de etiquetas ETag para garantizar que otro proceso no haya modificado el elemento. Consulte [Actualización de una entidad](#update-an-entity) para obtener información sobre el uso de etiquetas ETag.
>
>

## <a name="delete-a-table"></a>Eliminar una tabla

El código siguiente elimina una tabla de la cuenta de almacenamiento.

```javascript
tableSvc.deleteTable('mytable', function(error, response){
    if(!error){
        // Table deleted
    }
});
```

Si no está seguro de si existe la tabla, use **deleteTableIfExists**.

## <a name="use-continuation-tokens"></a>Usar tokens de continuación

Cuando consulte tablas para grandes cantidades de resultados, busque tokens de continuación. Puede haber grandes cantidades de datos disponibles para su consulta de los que podría no darse cuenta si no crear para reconocer cuando hay un token de continuación presente.

El objeto **results** que se devuelve al consultar los conjuntos de entidades, establece una propiedad `continuationToken` cuando hay un token de este tipo presente. Entonces, podrá usarlo al realizar una consulta para continuar moviéndose por las entidades de tabla y partición.

Al consultar, puede proporcionar un parámetro `continuationToken` entre la instancia de objeto de consulta y la función de devolución de llamada:

```javascript
var nextContinuationToken = null;
dc.table.queryEntities(tableName,
    query,
    nextContinuationToken,
    function (error, results) {
        if (error) throw error;

        // iterate through results.entries with results

        if (results.continuationToken) {
            nextContinuationToken = results.continuationToken;
        }

    });
```

Si inspecciona el objeto `continuationToken`, encontrará propiedades como `nextPartitionKey`, `nextRowKey` y `targetLocation`, que puede usar para iterar en todos los resultados.

También puede usar `top` junto con `continuationToken` para establecer el tamaño de página.

## <a name="work-with-shared-access-signatures"></a>Trabajo con firmas de acceso compartido

Las firmas de acceso compartido (SAS) constituyen una manera segura de ofrecer acceso granular a las tablas sin proporcionar el nombre o las claves de su cuenta de almacenamiento. Las SAS se usan con frecuencia para proporcionar acceso limitado a sus datos, por ejemplo, para permitir que una aplicación móvil consulte registros.

Una aplicación de confianza, como por ejemplo un servicio basado en la nube, genera una SAS mediante el valor **generateSharedAccessSignature** del elemento **TableService** y se lo proporciona a una aplicación en la que no se confía o en la que se confía parcialmente, como una aplicación móvil. La SAS se genera usando una directiva que describe las fechas de inicio y de finalización durante las cuales la SAS es válida, junto con el nivel de acceso otorgado al titular de la SAS.

En el siguiente ejemplo se genera una nueva directiva de acceso compartido que permitirá al titular de la SAS consultar ('r') la tabla, y que expira 100 minutos después de la hora en que se crea.

```javascript
var startDate = new Date();
var expiryDate = new Date(startDate);
expiryDate.setMinutes(startDate.getMinutes() + 100);
startDate.setMinutes(startDate.getMinutes() - 100);

var sharedAccessPolicy = {
  AccessPolicy: {
    Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
    Start: startDate,
    Expiry: expiryDate
  },
};

var tableSAS = tableSvc.generateSharedAccessSignature('mytable', sharedAccessPolicy);
var host = tableSvc.host;
```

Tenga en cuenta que también debe proporcionar la información del host, puesto que es necesaria cuando el titular de la SAS intenta acceder a la tabla.

La aplicación cliente usa entonces la SAS con **TableServiceWithSAS** para realizar operaciones en la tabla. En el siguiente ejemplo se realiza la conexión a la tabla y se realiza una consulta. Consulte en el artículo [Otorgar acceso limitado a recursos de Azure Storage con firmas de acceso compartido (SAS)](../storage/common/storage-sas-overview.md) el formato de tableSAS.

```javascript
// Note in the following command, host is in the format: `https://<your_storage_account_name>.table.core.windows.net` and the tableSAS is in the format: `sv=2018-03-28&si=saspolicy&tn=mytable&sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D`;

var sharedTableService = azure.createTableServiceWithSas(host, tableSAS);
var query = azure.TableQuery()
  .where('PartitionKey eq ?', 'hometasks');

sharedTableService.queryEntities(query, null, function(error, result, response) {
  if(!error) {
    // result contains the entities
  }
});
```

Como la SAS se generó solo con acceso de consulta, se devuelve un error si intenta insertar, actualizar o eliminar entidades.

### <a name="access-control-lists"></a>Listas de control de acceso

Se puede usar una lista de control de acceso (ACL) para definir la directiva de acceso para una SAS. Esto es útil si desea permitir que varios clientes accedan a la tabla, pero cada uno con directivas de acceso diferentes.

Una ACL se implementa mediante el uso de un conjunto de directivas de acceso, con un Id. asociado a cada directiva. En los siguientes ejemplos se definen dos directivas; una para "user1" y otra para "user2":

```javascript
var sharedAccessPolicy = {
  user1: {
    Permissions: azure.TableUtilities.SharedAccessPermissions.QUERY,
    Start: startDate,
    Expiry: expiryDate
  },
  user2: {
    Permissions: azure.TableUtilities.SharedAccessPermissions.ADD,
    Start: startDate,
    Expiry: expiryDate
  }
};
```

En el siguiente ejemplo se obtiene la ACL actual para la tabla **hometasks** y luego se agregan las nuevas directivas mediante **setTableAcl**. Este enfoque permite lo siguiente:

```javascript
var extend = require('extend');
tableSvc.getTableAcl('hometasks', function(error, result, response) {
if(!error){
    var newSignedIdentifiers = extend(true, result.signedIdentifiers, sharedAccessPolicy);
    tableSvc.setTableAcl('hometasks', newSignedIdentifiers, function(error, result, response){
      if(!error){
        // ACL set
      }
    });
  }
});
```

Después de establecer una ACL, puede crear luego una SAS basada en el identificador de una directiva. En el siguiente ejemplo se crea una nueva SAS para 'user2':

```javascript
tableSAS = tableSvc.generateSharedAccessSignature('hometasks', { Id: 'user2' });
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea los recursos siguientes.

* El [Explorador de Microsoft Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md) es una aplicación independiente y gratuita de Microsoft que permite trabajar visualmente con los datos de Azure Storage en Windows, macOS y Linux.
* Repositorio de [Azure Storage SDK para Node.js](https://github.com/Azure/azure-storage-node) en GitHub.
* [Azure para desarrolladores de Node.js](https://docs.microsoft.com/azure/developer/javascript/)
* [Creación de una aplicación web de Node.js en Azure](../app-service/quickstart-nodejs.md)
* [Compilación e implementación de una aplicación Node.js en un Servicio en la nube de Azure](../cloud-services/cloud-services-nodejs-develop-deploy-app.md) (con Windows PowerShell)
