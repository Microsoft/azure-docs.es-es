---
title: Conexión de la aplicación Angular a la API de Azure Cosmos DB para MongoDB mediante Mongoose
description: En este tutorial se describe cómo crear una aplicación Node.js con Angular y Express para administrar los datos almacenados en Cosmos DB. En esta parte, utilice Mongoose para conectarse a Azure Cosmos DB.
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18, devx-track-javascript
ms.reviewer: sngun
ms.openlocfilehash: 59acb2e851be69a9bb928d9b3aae73d8136493e0
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89440922"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---use-mongoose-to-connect-to-cosmos-db"></a>Creación de una aplicación de Angular con API de Azure Cosmos DB para MongoDB: uso de Mongoose para conectarse a Cosmos DB

En este tutorial de varias partes se muestra cómo crear una aplicación Node.js con Express y Angular y conectarla a la [cuenta de Cosmos configurada con la API de Cosmos DB para MongoDB](mongodb-introduction.md). En este artículo se describe la parte 5 del tutorial, que se basa en la [parte 4](tutorial-develop-mongodb-nodejs-part4.md).

En esta parte del tutorial, va a realizar lo siguiente:

> [!div class="checklist"]
> * Utilizar Mongoose para conectarse a Cosmos DB.
> * Obtener la cadena de conexión de Cosmos DB.
> * Crear el modelo de Hero.
> * Crear servicio Hero para obtener datos de Hero.
> * Ejecute localmente la aplicación.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* Antes de empezar este tutorial, complete los pasos de la [parte 4](tutorial-develop-mongodb-nodejs-part4.md).

* Este tutorial requiere que se ejecute localmente la CLI de Azure. Debe tener instalada la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para encontrar la versión. Si tiene que instalar o actualizar la CLI de Azure, consulte [Instalación de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).

* Este tutorial le guía paso a paso por el procedimiento necesario para compilar la aplicación. Si desea descargar el proyecto finalizado, puede obtener la aplicación completa en el [repositorio de angular-cosmosdb](https://github.com/Azure-Samples/angular-cosmosdb) en GitHub.

## <a name="use-mongoose-to-connect"></a>Uso de Mongoose para conectarse

Mongoose es una biblioteca de modelado de datos de objeto (ODM) para MongoDB y Node.js. Puede utilizar Mongoose para conectarse a la cuenta de Azure Cosmos DB. Siga estos pasos para instalar Mongoose y conectarse a Azure Cosmos DB:

1. Instale el módulo mongoose npm, que es una API que permite comunicarse con MongoDB.

    ```bash
    npm i mongoose --save
    ```

1. En la carpeta **server**, cree un archivo denominado **mongo.js**. Va a agregar los detalles de conexión de la cuenta de Azure Cosmos DB a este archivo.

1. Copie el código siguiente en el archivo **mongo.js**. Este código proporciona la funcionalidad siguiente:

   * Requiere Mongoose.
   * Invalida la promesa de Mongo para utilizar la promesa básica que se integra en ES6/ES2015 y versiones posteriores.
   * Llama a un archivo env que le permite configurar ciertos elementos en función de si se encuentra en fase de ensayo, producción o desarrollo. Creará ese archivo en la siguiente sección.
   * Incluye la cadena de conexión de MongoDB, que se establece en el archivo env.
   * Crea una función de conexión que llama a Mongoose.

     ```javascript
     const mongoose = require('mongoose');
     /**
     * Set to Node.js native promises
     * Per https://mongoosejs.com/docs/promises.html
     */
     mongoose.Promise = global.Promise;

     const env = require('./env/environment');

     // eslint-disable-next-line max-len
     const mongoUri = `mongodb://${env.accountName}:${env.key}@${env.accountName}.documents.azure.com:${env.port}/${env.databaseName}?ssl=true`;

     function connect() {
     mongoose.set('debug', true);
     return mongoose.connect(mongoUri, { useMongoClient: true });
     }

     module.exports = {
     connect,
     mongoose
     };
     ```
    
1. En el panel Explorador, en **server**, cree una carpeta denominada **entorno**. En la carpeta **environment**, cree un archivo denominado **environment.js**.

1. En el archivo mongo.js, hay que incluir valores para los parámetros `dbName`, `key` y `cosmosPort`. Copie el código siguiente en el archivo **environment.js**:

    ```javascript
    // TODO: replace if yours are different
    module.exports = {
      accountName: 'your-cosmosdb-account-name-goes-here',
      databaseName: 'admin', 
      key: 'your-key-goes-here',
      port: 10255
    };
    ```

## <a name="get-the-connection-string"></a>Obtención de la cadena de conexión

Para conectar la aplicación a Azure Cosmos DB, tiene que actualizar las opciones de configuración de la aplicación. Siga estos pasos para actualizar la configuración: 

1. En Azure Portal, obtenga el número de puerto, nombre de la cuenta de Azure Cosmos DB y los valores de clave principal de la cuenta de Azure Cosmos DB.

1. En el archivo **environment.js**, cambie el valor de `port` a 10255. 

    ```javascript
    const port = 10255;
    ```

1. En el archivo **environment.js**, cambie el valor de `accountName` por el nombre de la cuenta de Azure Cosmos DB que creó en la [parte 4](tutorial-develop-mongodb-nodejs-part4.md) del tutorial. 

1. Recupere la clave principal de la cuenta de Azure Cosmos DB mediante el siguiente comando de la CLI en la ventana del terminal: 

    ```azurecli-interactive
    az cosmosdb list-keys --name <cosmosdb-name> -g myResourceGroup
    ```    
    
    \<cosmosdb-name> es el nombre de la cuenta de Azure Cosmos DB que creó en la [cuarta parte](tutorial-develop-mongodb-nodejs-part4.md) del tutorial.

1. Copie la clave principal en el archivo **environment.js** como valor de `key`.

La aplicación tiene ahora toda la información necesaria para conectarse a Azure Cosmos DB. 

## <a name="create-a-hero-model"></a>Creación de un modelo de Hero

Luego tiene que definir el esquema de los datos que se van a almacenar en Azure Cosmos DB definiendo un archivo de modelo. Siga estos pasos para crear un _modelo de Hero_ que define el esquema de los datos:

1. En el panel Explorador, cree un archivo denominado **hero.model.js** en la carpeta **server**.

1. Copie el código siguiente en el archivo **hero.model.js**. Este código proporciona la funcionalidad siguiente:

   * Requiere Mongoose.
   * Crea un nuevo esquema con un identificador, un nombre y un mensaje.
   * Crea un modelo con el esquema.
   * Exporta el modelo. 
   * Asigne a la colección el nombre **Heroes** (en lugar de **Heros**, que es el nombre predeterminado de la colección según las reglas de nomenclatura de plurales de Mongoose).

   ```javascript
   const mongoose = require('mongoose');

   const Schema = mongoose.Schema;

   const heroSchema = new Schema(
     {
       id: { type: Number, required: true, unique: true },
       name: String,
       saying: String
     },
     {
       collection: 'Heroes'
     }
   );

   const Hero = mongoose.model('Hero', heroSchema);

   module.exports = Hero;
   ```

## <a name="create-a-hero-service"></a>Creación de un servicio Hero

Cuando haya creado el modelo de Hero, tendrá que definir un servicio para leer los datos y realizar operaciones de lista, creación, eliminación y actualización. Siga estos pasos para crear un _servicio Hero_ que consulta los datos de Azure Cosmos DB:

1. En el panel Explorador, cree un archivo denominado **hero.service.js** en la carpeta **server**.

1. Copie el código siguiente en el archivo **hero.service.js**. Este código proporciona la funcionalidad siguiente:

   * Obtiene el modelo que se ha creado.
   * Se conecta a la base de datos.
   * Se crea una variable `docquery` que utiliza el método `hero.find` para definir una consulta que devuelve todos los héroes.
   * Se ejecuta una consulta con la función `docquery.exec` y una promesa para obtener una lista de todos los héroes, en la que el estado de respuesta es 200. 
   * Si el estado es 500, devuelve el mensaje de error.
   * Dado que usamos módulos, se obtiene los héroes. 

   ```javascript
   const Hero = require('./hero.model');

   require('./mongo').connect();

   function getHeroes() {
     const docquery = Hero.find({});
     docquery
       .exec()
       .then(heroes => {
         res.status(200).json(heroes);
       })
       .catch(error => {
         res.status(500).send(error);
         return;
       });
   }

   module.exports = {
     getHeroes
   };
   ```

## <a name="configure-routes"></a>Configuración de rutas

Luego tiene que configurar rutas para controlar las direcciones URL para la obtención, creación, lectura y eliminación de solicitudes. Los métodos de enrutamiento especifican funciones de devolución de llamada (también denominadas _funciones de controlador_). Estas funciones se llaman cuando la aplicación recibe una solicitud para el punto de conexión especificado y el método HTTP. Siga estos pasos para agregar el servicio Hero y definir las rutas:

1. En Visual Studio Code, en el archivo **routes.js**, convierta en comentario la función `res.send` que envía los datos de ejemplo de Hero. Agregue una línea para llamar a la función `heroService.getHeroes` en su lugar.

    ```javascript
    router.get('/heroes', (req, res) => {
      heroService.getHeroes(req, res);
    //  res.send(200, [
    //      {"id": 10, "name": "Starlord", "saying": "oh yeah"}
    //  ])
    });
    ```

1. En el archivo **routes.js**, use `require` para el servicio Hero:

    ```javascript
    const heroService = require('./hero.service'); 
    ```

1. En el archivo **hero.service.js**, actualice la función `getHeroes` para tomar los parámetros `req` y `res`, tal y como se indica a continuación:

    ```javascript
    function getHeroes(req, res) {
    ```

Dediquemos un minuto a revisar y repasar el código anterior. En primer lugar, entramos en el archivo index.js, que configura el servidor de nodo. Tenga en cuenta que también configura y define las rutas. El archivo routes.js se comunica entonces con el servicio Hero y le indica que obtenga las funciones, como **getHeroes**, y pase la solicitud y la respuesta. El archivo hero.service.js obtiene el modelo y se conecta a Mongo. Luego ejecuta **getHeroes** cuando se llama y devuelve una respuesta de 200. 

## <a name="run-the-app"></a>Ejecución la aplicación

Siga luego estos pasos para ejecutar la aplicación:

1. En Visual Studio Code, guarde todos los cambios. A la izquierda, seleccione el botón **Depurar** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part5/debug-button.png":::y **Iniciar depuración** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part5/start-debugging-button.png":::.

1. Ahora cambie al explorador. Abra las **Herramientas de desarrollo** y la **pestaña Red**. Vaya a `http://localhost:3000` y verá la aplicación.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part5/azure-cosmos-db-heroes-app.png" alt-text="Nueva cuenta de Azure Cosmos DB en Azure Portal":::

No hay ningún héroe almacenado aún en la aplicación. En la siguiente parte de este tutorial, vamos a agregar funcionalidad de colocación, inserción y eliminación. Luego se pueden agregar, actualizar y eliminar héroes desde la interfaz de usuario con conexiones de Mongoose a nuestra base de datos de Azure Cosmos. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos, podrá eliminar el grupo de recursos, la cuenta de Azure Cosmos DB y todos los recursos relacionados. Siga estos pasos para eliminar el grupo de recursos:

 1. Vaya al grupo de recursos donde creó la cuenta de Azure Cosmos DB.
 1. Seleccione **Eliminar grupo de recursos**.
 1. Confirme el nombre del grupo de recursos que quiere eliminar y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Continúe con la parte 6 del tutorial para agregar las funciones Post, Put y Delete a la aplicación:

> [!div class="nextstepaction"]
> [Parte 6: Incorporación de las funciones Post, Put y Delete a la aplicación](tutorial-develop-mongodb-nodejs-part6.md)
