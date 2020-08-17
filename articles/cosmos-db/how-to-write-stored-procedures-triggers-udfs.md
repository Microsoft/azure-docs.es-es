---
title: Escribir procedimientos almacenados, desencadenadores y funciones definidas por el usuario en Azure Cosmos DB
description: Aprenda a definir procedimientos almacenados, desencadenadores y funciones definidas por el usuario (UDF) en Azure Cosmos DB.
author: timsander1
ms.service: cosmos-db
ms.topic: how-to
ms.date: 06/16/2020
ms.author: tisande
ms.custom: devx-track-javascript
ms.openlocfilehash: 17cde20d69fbf7d135fe48c9e285ccf348b4b58c
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420165"
---
# <a name="how-to-write-stored-procedures-triggers-and-user-defined-functions-in-azure-cosmos-db"></a>Escritura de procedimientos almacenados, desencadenadores y funciones definidas por el usuario (UDF) en Azure Cosmos DB

Azure Cosmos DB ofrece una ejecución transaccional integrada del lenguaje de JavaScript que le permite escribir **procedimientos almacenados**, **desencadenadores** y **funciones definidas por el usuario (UDF)** . Cuando se usa la API de SQL en Azure Cosmos DB, puede definir los procedimientos almacenados, desencadenadores y UDF en el lenguaje JavaScript. Puede escribir su lógica en JavaScript y ejecutarla dentro del motor de base de datos. Puede crear y ejecutar desencadenadores, procedimientos almacenados y UDF mediante [Azure Portal](https://portal.azure.com/), la [API de consulta integrada del lenguaje JavaScript en Azure Cosmos DB](javascript-query-api.md) y los [SDK del cliente de la API de SQL de Cosmos DB](sql-api-dotnet-samples.md). 

Para llamar a un procedimiento almacenado, desencadenador o función definida por el usuario, deberá registrarlo. Para obtener más información, consulte [How to work with stored procedures, triggers, user-defined functions in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md) (Trabajo con procedimientos almacenados, desencadenadores y funciones definidas por el usuario en Azure Cosmos DB).

> [!NOTE]
> Para los contenedores con particiones, al ejecutar un procedimiento almacenado, se debe proporcionar un valor de clave de partición en las opciones de solicitud. Los procedimientos almacenados siempre se limitan a una clave de partición. Los elementos que tienen un valor de clave de partición diferente no estarán visibles para el procedimiento almacenado. Esto también se aplica a los desencadenadores.
> [!Tip]
> Cosmos admite la implementación de contenedores con procedimientos almacenados, desencadenadores y funciones definidas por el usuario. Para más información, consulte [Creación de un contenedor de Azure Cosmos DB con funcionalidad del lado servidor.](manage-sql-with-resource-manager.md#create-sproc)

## <a name="how-to-write-stored-procedures"></a><a id="stored-procedures"></a>Escritura de procedimientos almacenados

Los procedimientos almacenados se escriben con JavaScript y pueden crear, actualizar, leer, consultar y eliminar elementos dentro de un contenedor de Azure Cosmos. Los procedimientos almacenados se registran por colección y pueden funcionar en cualquier documento o dato adjunto presente en esa colección.

Este es un sencillo procedimiento almacenado que devuelve una respuesta "Hola mundo".

```javascript
var helloWorldStoredProc = {
    id: "helloWorld",
    serverScript: function () {
        var context = getContext();
        var response = context.getResponse();

        response.setBody("Hello, World");
    }
}
```

El objeto de contexto proporciona acceso a todas las operaciones que se pueden realizar en Azure Cosmos DB, así como acceso a los objetos de solicitud y respuesta. En este caso, usará el objeto de respuesta para establecer el cuerpo de la respuesta que se devolverá al cliente.

Una vez escrito, el procedimiento almacenado debe registrarse con una colección. Para obtener más información, consulte el artículo [Uso de procedimientos almacenados de Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md#stored-procedures).

### <a name="create-an-item-using-stored-procedure"></a><a id="create-an-item"></a>Creación de un elemento con el procedimiento almacenado

Cuando se crea un elemento utilizando el procedimiento almacenado, el elemento se inserta en el contenedor de Azure Cosmos y se devuelve un identificador para el elemento recién creado. La creación de un elemento es una operación asincrónica y depende de las funciones de devolución de llamada de JavaScript. La función de devolución de llamada tiene dos parámetros, uno para el objeto de error en caso de que la operación no se complete y otro para un valor devuelto; en este caso, el objeto creado. Dentro de la devolución de llamada, puede controlar la excepción o lanzar un error. En caso de que no se proporcione una devolución de llamada y haya un error, el sistema en tiempo de ejecución de Azure Cosmos DB producirá un error.

El procedimiento almacenado también incluye un parámetro para establecer la descripción, que es un valor booleano. Cuando el parámetro se establece en true y falta la descripción, el procedimiento almacenado iniciará una excepción. En caso contrario, el resto del procedimiento almacenado continúa ejecutándose.

El siguiente procedimiento almacenado de ejemplo toma una matriz de nuevos elementos de Azure Cosmos como entrada, la inserta en el contenedor de Azure Cosmos y devuelve el número de elementos insertados. En este ejemplo, estamos aprovechando el ejemplo ToDoList del [inicio rápido de la API de SQL en .NET](create-sql-api-dotnet.md).

```javascript
function createToDoItems(items) {
    var collection = getContext().getCollection();
    var collectionLink = collection.getSelfLink();
    var count = 0;

    if (!items) throw new Error("The array is undefined or null.");

    var numItems = items.length;

    if (numItems == 0) {
        getContext().getResponse().setBody(0);
        return;
    }

    tryCreate(items[count], callback);

    function tryCreate(item, callback) {
        var options = { disableAutomaticIdGeneration: false };

        var isAccepted = collection.createDocument(collectionLink, item, options, callback);

        if (!isAccepted) getContext().getResponse().setBody(count);
    }

    function callback(err, item, options) {
        if (err) throw err;
        count++;
        if (count >= numItems) {
            getContext().getResponse().setBody(count);
        } else {
            tryCreate(items[count], callback);
        }
    }
}
```

### <a name="arrays-as-input-parameters-for-stored-procedures"></a>Matrices como parámetros de entrada para procedimientos almacenados

Al definir un procedimiento almacenado mediante Azure Portal, los parámetros de entrada siempre se envían como una cadena para el procedimiento almacenado. Incluso si pasa una matriz de cadenas como entrada, la matriz se convierte en cadena y se envía al procedimiento almacenado. Para solucionar este problema, puede definir una función en el procedimiento almacenado para analizar la cadena como una matriz. El código siguiente muestra cómo analizar un parámetro de entrada de cadena como una matriz:

```javascript
function sample(arr) {
    if (typeof arr === "string") arr = JSON.parse(arr);

    arr.forEach(function(a) {
        // do something here
        console.log(a);
    });
}
```

### <a name="transactions-within-stored-procedures"></a><a id="transactions"></a>Transacciones en procedimientos almacenados

Puede implementar las transacciones en los elementos dentro de un contenedor mediante el uso de un procedimiento almacenado. En el ejemplo siguiente se utilizan las transacciones de la aplicación de un juego de fútbol de fantasía para intercambiar jugadores entre dos equipos en una sola operación. El procedimiento almacenado intenta leer los dos elementos de Azure Cosmos, donde cada uno de ellos corresponde al identificador del jugador que se ha pasado como un argumento. Si se encuentran ambos jugadores, el procedimiento almacenado actualiza los elementos intercambiando sus equipos. Si se produce cualquier error durante el proceso, el procedimiento almacenado lanza una excepción de JavaScript que de forma implícita cancela la transacción.

```javascript
// JavaScript source code
function tradePlayers(playerId1, playerId2) {
    var context = getContext();
    var container = context.getCollection();
    var response = context.getResponse();

    var player1Document, player2Document;

    // query for players
    var filterQuery =
    {
        'query' : 'SELECT * FROM Players p where p.id = @playerId1',
        'parameters' : [{'name':'@playerId1', 'value':playerId1}] 
    };

    var accept = container.queryDocuments(container.getSelfLink(), filterQuery, {},
        function (err, items, responseOptions) {
            if (err) throw new Error("Error" + err.message);

            if (items.length != 1) throw "Unable to find both names";
            player1Item = items[0];

            var filterQuery2 =
            {
                'query' : 'SELECT * FROM Players p where p.id = @playerId2',
                'parameters' : [{'name':'@playerId2', 'value':playerId2}]
            };
            var accept2 = container.queryDocuments(container.getSelfLink(), filterQuery2, {},
                function (err2, items2, responseOptions2) {
                    if (err2) throw new Error("Error" + err2.message);
                    if (items2.length != 1) throw "Unable to find both names";
                    player2Item = items2[0];
                    swapTeams(player1Item, player2Item);
                    return;
                });
            if (!accept2) throw "Unable to read player details, abort ";
        });

    if (!accept) throw "Unable to read player details, abort ";

    // swap the two players’ teams
    function swapTeams(player1, player2) {
        var player2NewTeam = player1.team;
        player1.team = player2.team;
        player2.team = player2NewTeam;

        var accept = container.replaceDocument(player1._self, player1,
            function (err, itemReplaced) {
                if (err) throw "Unable to update player 1, abort ";

                var accept2 = container.replaceDocument(player2._self, player2,
                    function (err2, itemReplaced2) {
                        if (err) throw "Unable to update player 2, abort"
                    });

                if (!accept2) throw "Unable to update player 2, abort";
            });

        if (!accept) throw "Unable to update player 1, abort";
    }
}
```

### <a name="bounded-execution-within-stored-procedures"></a><a id="bounded-execution"></a>Ejecución enlazada dentro de procedimientos almacenados

El siguiente es un ejemplo de un procedimiento almacenado que importa elementos de forma masiva en un contenedor de Azure Cosmos. El procedimiento almacenado controla la ejecución enlazada comprobando el valor de devolución booleano de `createDocument` y, a continuación, utiliza el recuento de elementos insertados en cada invocación del procedimiento almacenado para realizar un seguimiento y reanudar el progreso en todos los lotes.

```javascript
function bulkImport(items) {
    var container = getContext().getCollection();
    var containerLink = container.getSelfLink();

    // The count of imported items, also used as current item index.
    var count = 0;

    // Validate input.
    if (!items) throw new Error("The array is undefined or null.");

    var itemsLength = items.length;
    if (itemsLength == 0) {
        getContext().getResponse().setBody(0);
    }

    // Call the create API to create an item.
    tryCreate(items[count], callback);

    // Note that there are 2 exit conditions:
    // 1) The createDocument request was not accepted.
    //    In this case the callback will not be called, we just call setBody and we are done.
    // 2) The callback was called items.length times.
    //    In this case all items were created and we don’t need to call tryCreate anymore. Just call setBody and we are done.
    function tryCreate(item, callback) {
        var isAccepted = container.createDocument(containerLink, item, callback);

        // If the request was accepted, callback will be called.
        // Otherwise report current count back to the client,
        // which will call the script again with remaining set of items.
        if (!isAccepted) getContext().getResponse().setBody(count);
    }

    // This is called when container.createDocument is done in order to process the result.
    function callback(err, item, options) {
        if (err) throw err;

        // One more item has been inserted, increment the count.
        count++;

        if (count >= itemsLength) {
            // If we created all items, we are done. Just set the response.
            getContext().getResponse().setBody(count);
        } else {
            // Create next document.
            tryCreate(items[count], callback);
        }
    }
}
```

### <a name="async-await-with-stored-procedures"></a><a id="async-promises"></a>Característica async/await con procedimientos almacenados

A continuación se proporciona un procedimiento almacenado de ejemplo que usa async-await con promesas mediante una función auxiliar. El procedimiento almacenado consulta un elemento y lo reemplaza.

```javascript
function async_sample() {
    const ERROR_CODE = {
        NotAccepted: 429
    };

    const asyncHelper = {
        queryDocuments(sqlQuery, options) {
            return new Promise((resolve, reject) => {
                const isAccepted = __.queryDocuments(__.getSelfLink(), sqlQuery, options, (err, feed, options) => {
                    if (err) reject(err);
                    resolve({ feed, options });
                });
                if (!isAccepted) reject(new Error(ERROR_CODE.NotAccepted, "replaceDocument was not accepted."));
            });
        },

        replaceDocument(doc) {
            return new Promise((resolve, reject) => {
                const isAccepted = __.replaceDocument(doc._self, doc, (err, result, options) => {
                    if (err) reject(err);
                    resolve({ result, options });
                });
                if (!isAccepted) reject(new Error(ERROR_CODE.NotAccepted, "replaceDocument was not accepted."));
            });
        }
    };

    async function main() {
        let continuation;
        do {
            let { feed, options } = await asyncHelper.queryDocuments("SELECT * from c", { continuation });

            for (let doc of feed) {
                doc.newProp = 1;
                await asyncHelper.replaceDocument(doc);
            }

            continuation = options.continuation;
        } while (continuation);
    }

    main().catch(err => getContext().abort(err));
}
```

## <a name="how-to-write-triggers"></a><a id="triggers"></a>Escritura de desencadenadores

Azure Cosmos DB admite desencadenadores previos y posteriores. Los desencadenadores previos se ejecutan antes de modificar un elemento de la base de datos, y los desencadenadores posteriores se ejecutan después de modificar un elemento de la base de datos.

### <a name="pre-triggers"></a><a id="pre-triggers"></a>Desencadenadores previos

En el ejemplo siguiente se muestra cómo se puede utilizar un desencadenador previo para validar las propiedades de un elemento de Azure Cosmos que se está creando. En este ejemplo, estamos aprovechando el ejemplo ToDoList del [inicio rápido de la API de SQL en .NET](create-sql-api-dotnet.md) para agregar una propiedad de marca de tiempo a un elemento recién agregado, en caso de que no tenga ninguna.

```javascript
function validateToDoItemTimestamp() {
    var context = getContext();
    var request = context.getRequest();

    // item to be created in the current operation
    var itemToCreate = request.getBody();

    // validate properties
    if (!("timestamp" in itemToCreate)) {
        var ts = new Date();
        itemToCreate["timestamp"] = ts.getTime();
    }

    // update the item that will be created
    request.setBody(itemToCreate);
}
```

Los desencadenadores previos no pueden tener parámetros de entrada. El objeto solicitado en el desencadenador se utiliza para manipular el mensaje de solicitud asociado con la operación. En el ejemplo anterior, el desencadenador previo se ejecuta al crear un elemento de Azure Cosmos, y el cuerpo del mensaje de la solicitud contiene el elemento que se creará en formato JSON.

Cuando se registran los desencadenadores, puede especificar las operaciones que se pueden ejecutar con ellos. Este desencadenador se debería crear con un valor `TriggerOperation` de `TriggerOperation.Create`, lo que significa que no se permite usar el desencadenador en una operación de reemplazo, como se muestra en el código siguiente.

Para obtener ejemplos de cómo registrar y llamar a un desencadenador previo, vea los artículos sobre [desencadenadores previos](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) y [desencadenadores posteriores](how-to-use-stored-procedures-triggers-udfs.md#post-triggers). 

### <a name="post-triggers"></a><a id="post-triggers"></a>Desencadenadores posteriores

En el ejemplo siguiente se muestra un desencadenador posterior. Este desencadenador consulta el elemento de metadatos y lo actualiza con detalles del elemento recién creado.


```javascript
function updateMetadata() {
var context = getContext();
var container = context.getCollection();
var response = context.getResponse();

// item that was created
var createdItem = response.getBody();

// query for metadata document
var filterQuery = 'SELECT * FROM root r WHERE r.id = "_metadata"';
var accept = container.queryDocuments(container.getSelfLink(), filterQuery,
    updateMetadataCallback);
if(!accept) throw "Unable to update metadata, abort";

function updateMetadataCallback(err, items, responseOptions) {
    if(err) throw new Error("Error" + err.message);
        if(items.length != 1) throw 'Unable to find metadata document';

        var metadataItem = items[0];

        // update metadata
        metadataItem.createdItems += 1;
        metadataItem.createdNames += " " + createdItem.id;
        var accept = container.replaceDocument(metadataItem._self,
            metadataItem, function(err, itemReplaced) {
                    if(err) throw "Unable to update metadata, abort";
            });
        if(!accept) throw "Unable to update metadata, abort";
        return;
}
```

Es importante tener en cuenta la ejecución transaccional de los desencadenadores en Azure Cosmos DB. El desencadenador posterior se ejecuta como parte de la misma transacción para el propio elemento subyacente. Una excepción durante la ejecución del desencadenador posterior producirá un error de toda la transacción. Todo lo que esté confirmado se revertirá y se devolverá una excepción.

Para obtener ejemplos de cómo registrar y llamar a un desencadenador previo, vea los artículos sobre [desencadenadores previos](how-to-use-stored-procedures-triggers-udfs.md#pre-triggers) y [desencadenadores posteriores](how-to-use-stored-procedures-triggers-udfs.md#post-triggers). 

## <a name="how-to-write-user-defined-functions"></a><a id="udfs"></a>Escritura de funciones definidas por el usuario

En el ejemplo siguiente se crea una UDF para calcular los impuestos sobre la renta para diferentes niveles de renta. A continuación, se usaría esta función definida por el usuario dentro de una consulta. Para los fines de este ejemplo, suponga hay un contenedor denominado "Ingresos" con las siguientes propiedades:

```json
{
   "name": "Satya Nadella",
   "country": "USA",
   "income": 70000
}
```

Lo siguiente es una definición de función para calcular los impuestos sobre la renta para diferentes niveles de renta:

```javascript
function tax(income) {

        if(income == undefined)
            throw 'no input';

        if (income < 1000)
            return income * 0.1;
        else if (income < 10000)
            return income * 0.2;
        else
            return income * 0.4;
    }
```

Para obtener ejemplos de cómo registrar y utilizar una función definida por el usuario, consulte el artículo [How to use user-defined functions in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md#udfs) (Trabajo con funciones definidas por el usuario en Azure Cosmos DB).

## <a name="logging"></a>Registro 

Al utilizar procedimientos almacenados, desencadenadores o funciones que haya definido el usuario, puede registrar los pasos con el comando `console.log()`. Este comando concentrará una cadena para la depuración cuando `EnableScriptLogging` esté establecido en "true" tal como se muestra en el ejemplo siguiente:

```javascript
var response = await client.ExecuteStoredProcedureAsync(
document.SelfLink,
new RequestOptions { EnableScriptLogging = true } );
Console.WriteLine(response.ScriptLog);
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda más conceptos y cómo escribir o utilizar procedimientos almacenados, desencadenadores y funciones definidas por el usuario en Azure Cosmos DB:

* [How to register and use stored procedures, triggers, and user-defined functions in Azure Cosmos DB](how-to-use-stored-procedures-triggers-udfs.md) (Registro y uso de procedimientos almacenados, desencadenadores y funciones definidas por el usuario en Azure Cosmos DB)

* [How to write stored procedures and triggers using Javascript Query API in Azure Cosmos DB](how-to-write-javascript-query-api.md) (Escritura de procedimientos almacenados y desencadenadores con la API de consulta de Javascript en Azure Cosmos DB)

* [Working with Azure Cosmos DB stored procedures, triggers, and user-defined functions in Azure Cosmos DB](stored-procedures-triggers-udfs.md) (Trabajo con procedimientos almacenados, desencadenadores y funciones definidas por el usuario de Azure Cosmos DB en Azure Cosmos DB)

* [Working with JavaScript language integrated query API in Azure Cosmos DB](javascript-query-api.md) (Trabajo con la API de consulta integrada del lenguaje JavaScript en Azure Cosmos DB)
