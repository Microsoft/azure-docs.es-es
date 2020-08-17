---
title: Tutorial de Node.js para la API de SQL para Azure Cosmos DB
description: Un tutorial de Node.js en el que se muestra cómo conectar con Azure Cosmos DB y realizar consultas en él mediante SQL API
author: deborahc
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 04/20/2020
ms.author: dech
ms.custom: devx-track-javascript
ms.openlocfilehash: e86f79a286c10c45b2586dd1aa4d3aed035fc450
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421865"
---
# <a name="tutorial-build-a-nodejs-console-app-with-the-javascript-sdk-to-manage-azure-cosmos-db-sql-api-data"></a>Tutorial: Creación de una aplicación de consola de Node.js con el SDK de JavaScript para administrar los datos de SQL API de Azure Cosmos DB

> [!div class="op_single_selector"]
> * [.NET](sql-api-get-started.md)
> * [Java](sql-api-java-get-started.md)
> * [Async Java](sql-api-async-java-get-started.md)
> * [Node.js](sql-api-nodejs-get-started.md)
> 

Como desarrollador, puede que tenga aplicaciones que usan datos de documentos NoSQL. Puede usar una cuenta de SQL API en Azure Cosmos DB para almacenar datos de documentos y acceder a dichos datos. En este tutorial se muestra cómo crear una aplicación de consola Node.js para crear recursos de Azure Cosmos DB y consultarlos.

En este tutorial, aprenderá lo siguiente:

> [!div class="checklist"]
> * Crear una cuenta de Azure Cosmos DB y conectarse a ella.
> * Configurar la aplicación.
> * Crear una base de datos.
> * Cree un contenedor.
> * Agregar elementos al contenedor.
> * Realizar operaciones básicas sobre los elementos, el contenedor y la base de datos.

## <a name="prerequisites"></a>Requisitos previos 

Asegúrese de que dispone de los siguientes recursos:

* Una cuenta de Azure activa. Si no tiene una, puede registrarse para obtener una [prueba gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/). 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Node.js](https://nodejs.org/) versión 6.0.0 o superior.

## <a name="create-azure-cosmos-db-account"></a>Creación de una cuenta de Azure Cosmos DB

Vamos a crear una cuenta de Azure Cosmos DB. Si ya tiene una cuenta que desea usar, puede ir directamente a [Configuración de la aplicación de Node.js](#SetupNode). Si usa el emulador de Azure Cosmos DB, siga los pasos que se indican en [Emulador de Azure Cosmos DB](local-emulator.md) para configurar el emulador y vaya directamente a [Configuración de la aplicación de Node.js](#SetupNode). 

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

## <a name="set-up-your-nodejs-application"></a><a id="SetupNode"></a>Configuración de la aplicación de Node.js

Antes de empezar a escribir código para compilar la aplicación, puede compilar el marco de la aplicación. Ejecute estos pasos para configurar la aplicación de Node.js que tiene el código del marco:

1. Abra su terminal favorito.
2. Busque la carpeta o el directorio donde desea guardar la aplicación de Node.js.
3. Cree archivos de JavaScript vacíos con los siguientes comandos:

   * Windows:
     * `fsutil file createnew app.js 0`
     * `fsutil file createnew config.js 0`
     * `md data`
     * `fsutil file createnew data\databaseContext.js 0`

   * Linux/OS X:
     * `touch app.js`
     * `touch config.js`
     * `mkdir data`
     * `touch data/databaseContext.js`

4. Cree e inicialice un archivo `package.json`. Use el comando siguiente:
   * ```npm init -y```

5. Instale el módulo @azure/cosmos mediante npm. Use el comando siguiente:
   * ```npm install @azure/cosmos --save```

## <a name="set-your-apps-configurations"></a><a id="Config"></a>Configuración de las opciones de la aplicación

Ahora que existe la aplicación, se debe asegurar de que se puede comunicar con Azure Cosmos DB. Si actualiza ciertas opciones de configuración, tal como se muestra en los pasos siguiente, puede establecer que la aplicación se comunique con Azure Cosmos DB:

1. Abra el archivo *config.js* en el editor de texto que prefiera.

1. Copie y pegue el siguiente fragmento de código en el archivo *config.js* y establezca las propiedades `endpoint` y `key` en el URI del punto de conexión de Azure Cosmos DB y la clave principal. La base de datos y los nombres de contenedor se establecen en **Tareas** y **Elementos**. La clave de partición que usará para esta aplicación es **/category**.

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/config.js":::

   Puede encontrar los detalles de la clave y el punto de conexión en el panel **Claves** de [Azure Portal](https://portal.azure.com).

   :::image type="content" source="media/sql-api-nodejs-get-started/node-js-tutorial-keys.png" alt-text="Captura de pantalla de obtención de claves en Azure Portal":::

El SDK de JavaScript usa los términos genéricos *contenedor* y *elemento*. Un contenedor puede ser una colección, un grafo o una tabla. Un elemento puede ser un documento, un vértice o borde, o una fila, y es el contenido que hay en un contenedor. En el fragmento de código anterior, el código `module.exports = config;` se usa para exportar el objeto de configuración, de modo que pueda hacer referencia a él en el archivo *app.js*.

## <a name="create-a-database-and-a-container"></a>Creación de una base de datos y un contenedor

1. Abra el archivo *databaseContext.js* en el editor de texto que prefiera.

1. Copie y pegue el código siguiente en el archivo *databaseContext.js*. Este código define una función que crea la base de datos "Tasks", "Items" y el contenedor si aún no existen en su cuenta de Azure Cosmos:

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/data/databaseContext.js" id="createDatabaseAndContainer":::

   Una base de datos es el contenedor lógico de elementos con particiones en contenedores. Para crear una base de datos, se pueden usar las funciones `createIfNotExists` o create de la clase **Databases**. Un contenedor se compone de elementos que, en el caso de la API de SQL, son documentos JSON. Para crear un contenedor, se pueden usar las funciones `createIfNotExists` o create de la clase **Containers**. Después de crear un contenedor, puede almacenar y consultar los datos.

   > [!WARNING]
   > La creación de un contenedor tiene implicaciones en los precios. Visite la [página de precios](https://azure.microsoft.com/pricing/details/cosmos-db/) para saber qué esperar.

## <a name="import-the-configuration"></a>Importar la configuración

1. Abra el archivo *app.js* en el editor de texto que prefiera.

1. Copie y pegue el código siguiente para importar el módulo `@azure/cosmos`, la configuración y el valor de databaseContext que definió en los pasos anteriores. 

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="ImportConfiguration":::

## <a name="connect-to-the-azure-cosmos-account"></a>Conexión a una cuenta de Azure Cosmos

En el archivo *app.js*, copie y pegue el código siguiente para usar el punto de conexión y la clave guardados anteriormente a fin de crear un nuevo objeto CosmosClient.

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="CreateClientObjectDatabaseContainer":::

> [!Note]
> Si se conecta al **emulador de Cosmos DB**, deshabilite la comprobación de TLS para el proceso de los nodos:
>   ```javascript
>   process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
>   const client = new CosmosClient({ endpoint, key });
>   ```

Ahora que tiene el código necesario para inicializar el cliente de Azure Cosmos DB, se explicará cómo usar los recursos de este servicio.

## <a name="query-items"></a><a id="QueryItem"></a>Elementos de consulta

Azure Cosmos DB admite consultas enriquecidas en los elementos JSON que se almacenan en cada contenedor. En el código de ejemplo siguiente se muestra una consulta que se puede ejecutar con los elementos del contenedor. Puede consultar los elementos mediante la función de consulta de la clase `Items`. Agregue el código siguiente al archivo *app.js* para consultar los elementos de su cuenta de Azure Cosmos:

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="QueryItems":::

## <a name="create-an-item"></a><a id="CreateItem"></a>Creación de un elemento

Para crear un elemento, se puede usar la función create de la clase `Items`. Cuando se usa SQL API, los elementos se proyectan como documentos, que son contenido JSON (arbitrario) definido por el usuario. En este tutorial, creará un nuevo elemento en la base de datos Tasks.

1. En el archivo app.js, proporcione la definición del elemento:

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="DefineNewItem":::

1. Agregue el código siguiente para crear el elemento definido previamente:

   :::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="CreateItem":::

## <a name="update-an-item"></a><a id="ReplaceItem"></a>

Azure Cosmos DB admite el reemplazo del contenido de los elementos. Copie y pegue el código siguiente en el archivo *app.js*. Este código obtiene un elemento del contenedor y actualiza el campo *isComplete* a true.

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="UpdateItem":::

## <a name="delete-an-item"></a><a id="DeleteItem"></a>Eliminación de un elemento

Azure Cosmos DB admite la eliminación de elementos JSON. En el código siguiente se muestra cómo obtener un elemento por su identificador y eliminarlo. Copie y pegue el código siguiente en el archivo *app.js*:

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js" id="DeleteItem":::

## <a name="run-your-nodejs-application"></a><a id="Run"></a>Ejecución de la aplicación de Node.js

En conjunto, el código debe ser similar a este:

:::code language="javascript" source="~/cosmosdb-nodejs-get-started/app.js":::

En el terminal, busque el archivo ```app.js``` y ejecute el comando:

```bash 
node app.js
```

Ahora debería ver la salida de la aplicación GetStarted. La salida debe coincidir con el texto del ejemplo siguiente.

```
Created database:
Tasks

Created container:
Items

Querying container: Items
1 - Pick up apples and strawberries.

Created new item: 3 - Complete Cosmos DB Node.js Quickstart ⚡

Updated item: 3 - Complete Cosmos DB Node.js Quickstart ⚡
Updated isComplete to true

Deleted item with id: 3
```

## <a name="get-the-complete-nodejs-tutorial-solution"></a><a id="GetSolution"></a>Obtención de la solución completa del tutorial de Node.js

Si no dispuso de tiempo para completar los pasos de este tutorial, o simplemente desea descargar el código, puede obtenerlo de [GitHub](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started ).

Para ejecutar la solución de introducción que contiene todo el código de este artículo, necesitará lo siguiente:

* Una [cuenta de Azure Cosmos DB][create-account].
* La solución [Getting Started](https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-getting-started) disponible en GitHub.

Instale las dependencias del proyecto mediante npm. Use el comando siguiente:

* ```npm install``` 

Luego, en el archivo ```config.js```, actualice los valores config.endpoint y config.key como describe en [Paso 3: Configuración de las opciones de la aplicación](#Config).  

Luego, en el terminal, busque el archivo ```app.js``` y ejecute el comando:  

```bash  
node app.js 
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite estos recursos, podrá eliminar el grupo de recursos, la cuenta de Azure Cosmos DB y todos los recursos relacionados. Para hacerlo, seleccione el grupo de recursos que usó en la cuenta de Azure Cosmos DB, seleccione **Eliminar** y confirme el nombre del grupo de recursos que se va a eliminar.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Supervisión de una cuenta de Azure Cosmos DB](monitor-accounts.md)

[create-account]: create-sql-api-dotnet.md#create-account
