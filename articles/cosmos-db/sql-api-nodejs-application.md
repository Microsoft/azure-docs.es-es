---
title: 'Tutorial: Creación de una aplicación web Node.js mediante el SDK de JavaScript de Azure Cosmos DB para administrar los datos de SQL API'
description: Este tutorial de Node.js explora cómo usar Microsoft Azure Cosmos DB para almacenar datos de una aplicación web Node.js Express hospedada en una característica de Web Apps de Microsoft Azure App Service y acceder a ellos.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 11/05/2019
ms.author: sngun
ms.custom: devx-track-javascript
ms.openlocfilehash: 9554fa63d13c5da465653064d69d7950d91f0300
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421882"
---
# <a name="tutorial-build-a-nodejs-web-app-using-the-javascript-sdk-to-manage-a-sql-api-account-in-azure-cosmos-db"></a>Tutorial: Compilación de una aplicación web de Node.js mediante el SDK de JavaScript para administrar la cuenta de SQL API en Azure Cosmos DB 

> [!div class="op_single_selector"]
> * [.NET](sql-api-dotnet-application.md)
> * [Java](sql-api-java-application.md)
> * [Node.js](sql-api-nodejs-application.md)
> * [Python](sql-api-python-application.md)
> * [Xamarin](mobile-apps-with-xamarin.md)
> 

Como desarrollador, puede que tenga aplicaciones que usan datos de documentos NoSQL. Puede usar una cuenta de SQL API en Azure Cosmos DB para almacenar datos de documentos y acceder a dichos datos. En este tutorial de Node.js se muestra cómo almacenar datos desde una cuenta de SQL API en Azure Cosmos DB mediante una aplicación de Node.js Express hospedada en la característica Web Apps de Microsoft Azure App Service y acceder a ellos. En este tutorial, creará una aplicación basada en web (aplicación de tareas pendientes), que le permite crear, recuperar y completar tareas. Las tareas se almacenan como documentos JSON en Azure Cosmos DB. 

En este tutorial se muestra cómo crear una cuenta de SQL API en Azure Cosmos DB mediante Azure Portal. A continuación se compila y ejecuta una aplicación web que se basa en el SDK de Node.js para crear una base de datos y un contenedor y agregar elementos al contenedor. Este tutorial utiliza la versión 3.0 del SDK de JavaScript.

En este tutorial se describen las tareas siguientes:

> [!div class="checklist"]
> * Creación de una cuenta de Azure Cosmos DB
> * Creación de una aplicación Node.js
> * Conexión de una aplicación a Azure Cosmos DB
> * Ejecución e implementación de la aplicación en Azure

## <a name="prerequisites"></a><a name="_Toc395783176"></a>Requisitos previos

Antes de seguir las instrucciones del presente artículo, asegúrese de tener los siguientes recursos:

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 

  [!INCLUDE [cosmos-db-emulator-docdb-api](../../includes/cosmos-db-emulator-docdb-api.md)]

* [Node.js][Node.js], versión 6.10 o posterior.
* [Express Generator](https://www.expressjs.com/starter/generator.html) (puede instalar Express mediante `npm install express-generator -g`)
* Instale [Git][Git] en la estación de trabajo local.

## <a name="create-an-azure-cosmos-db-account"></a><a name="_Toc395637761"></a>Creación de una cuenta de Azure Cosmos DB
Para comenzar, creemos una cuenta de Azure Cosmos DB. Si ya tiene una cuenta o si usa el Emulador de Azure Cosmos DB para este tutorial, puede ir directamente al [Paso 2: Creación de una aplicación Node.js](#_Toc395783178).

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

[!INCLUDE [cosmos-db-keys](../../includes/cosmos-db-keys.md)]

## <a name="create-a-new-nodejs-application"></a><a name="_Toc395783178"></a>Creación de una aplicación Node.js
Ahora aprendamos a crear un proyecto Node.js básico de Hola mundo usando el marco Express.

1. Abra su terminal favorito como, por ejemplo, el símbolo del sistema de Node.js.

1. Navegue hasta el directorio en el que desea almacenar la nueva aplicación.

1. Use Express Generator para generar una nueva aplicación denominada **todo**.

   ```bash
   express todo
   ```

1. Abra el directorio **todo** e instale las dependencias.

   ```bash
   cd todo
   npm install
   ```

1. Ejecute la nueva aplicación.

   ```bash
   npm start
   ```

1. Para ver la nueva aplicación, vaya a `http://localhost:3000` desde el explorador.
   
   :::image type="content" source="./media/sql-api-nodejs-application/cosmos-db-node-js-express.png" alt-text="Aprendizaje de Node.js: captura de pantalla de la aplicación Hello World en una ventana del explorador":::

   Para detener la aplicación, use CTRL+C en la ventana del terminal y seleccione **y** para finalizar el trabajo por lotes.

## <a name="install-the-required-modules"></a><a name="_Toc395783179"></a>Instalación de los módulos necesarios

El archivo **package.json** es uno de los archivos creados en la raíz del proyecto. Este archivo contiene una lista de módulos adicionales necesarios para una aplicación Node.js. Cuando implemente esta aplicación en Azure, este archivo se usará para determinar qué módulos se deben instalar en Azure para ofrecer respaldo a su aplicación. Instale dos paquetes más para este tutorial.

1. Instale el módulo **\@azure/cosmos** mediante npm. 

   ```bash
   npm install @azure/cosmos
   ```

## <a name="connect-the-nodejs-application-to-azure-cosmos-db"></a><a name="_Toc395783180"></a>Conexión de la aplicación Node.js a Azure Cosmos DB
Ahora que ha completado la instalación y configuración iniciales, a continuación escribirá código que la aplicación de tareas pendientes (Todo) necesita para comunicarse con Azure Cosmos DB.

### <a name="create-the-model"></a>Creación del modelo
1. En la raíz del directorio del proyecto, cree un nuevo directorio con el nombre **models**.  

2. En el directorio **models**, cree un archivo nuevo con el nombre **taskDao.js**. Este archivo contiene el código necesario para crear la base de datos y el contenedor. También define los métodos para leer, actualizar, crear y encontrar tareas en Azure Cosmos DB. 

3. Copie el código siguiente en el archivo **taskDao.js**:

   ```javascript
    // @ts-check
    const CosmosClient = require('@azure/cosmos').CosmosClient
    const debug = require('debug')('todo:taskDao')

    // For simplicity we'll set a constant partition key
    const partitionKey = undefined
    class TaskDao {
      /**
       * Manages reading, adding, and updating Tasks in Cosmos DB
       * @param {CosmosClient} cosmosClient
       * @param {string} databaseId
       * @param {string} containerId
       */
      constructor(cosmosClient, databaseId, containerId) {
        this.client = cosmosClient
        this.databaseId = databaseId
        this.collectionId = containerId

        this.database = null
        this.container = null
      }

      async init() {
        debug('Setting up the database...')
        const dbResponse = await this.client.databases.createIfNotExists({
          id: this.databaseId
        })
        this.database = dbResponse.database
        debug('Setting up the database...done!')
        debug('Setting up the container...')
        const coResponse = await this.database.containers.createIfNotExists({
          id: this.collectionId
        })
        this.container = coResponse.container
        debug('Setting up the container...done!')
      }

      async find(querySpec) {
        debug('Querying for items from the database')
        if (!this.container) {
          throw new Error('Collection is not initialized.')
        }
        const { resources } = await this.container.items.query(querySpec).fetchAll()
        return resources
      }

      async addItem(item) {
        debug('Adding an item to the database')
        item.date = Date.now()
        item.completed = false
        const { resource: doc } = await this.container.items.create(item)
        return doc
      }

      async updateItem(itemId) {
        debug('Update an item in the database')
        const doc = await this.getItem(itemId)
        doc.completed = true

        const { resource: replaced } = await this.container
          .item(itemId, partitionKey)
          .replace(doc)
        return replaced
      }

      async getItem(itemId) {
        debug('Getting an item from the database')
        const { resource } = await this.container.item(itemId, partitionKey).read()
        return resource
      }
    }

    module.exports = TaskDao
   ```
4. Guarde y cierre el archivo **taskDao.js** .  

### <a name="create-the-controller"></a>Crear el controlador

1. En el directorio **routes** del nuevo proyecto, cree un nuevo archivo denominado **tasklist.js**.  

2. Agregue el siguiente código a **tasklist.js**. Este código carga los módulos CosmosClient y async, que utiliza **tasklist.js**. Este código también define la clase **TaskList**, que pasa como una instancia del objeto **TaskDao** que definimos anteriormente:
   
   ```javascript
    const TaskDao = require("../models/TaskDao");
    
    class TaskList {
      /**
       * Handles the various APIs for displaying and managing tasks
       * @param {TaskDao} taskDao
       */
      constructor(taskDao) {
        this.taskDao = taskDao;
      }
      async showTasks(req, res) {
        const querySpec = {
          query: "SELECT * FROM root r WHERE r.completed=@completed",
          parameters: [
            {
              name: "@completed",
              value: false
            }
          ]
        };

        const items = await this.taskDao.find(querySpec);
        res.render("index", {
          title: "My ToDo List ",
          tasks: items
        });
      }

      async addTask(req, res) {
        const item = req.body;

        await this.taskDao.addItem(item);
        res.redirect("/");
      }

      async completeTask(req, res) {
        const completedTasks = Object.keys(req.body);
        const tasks = [];

        completedTasks.forEach(task => {
          tasks.push(this.taskDao.updateItem(task));
        });

        await Promise.all(tasks);

        res.redirect("/");
      }
    }

    module.exports = TaskList;
   ```

3. Guarde y cierre el archivo **tasklist.js** .

### <a name="add-configjs"></a>Agregar config.js

1. En la raíz del directorio del proyecto, cree un nuevo archivo con el nombre **config.js**. 

2. Agregue el código siguiente al archivo **config.js**. Este código define las opciones de configuración y los valores necesarios para nuestra aplicación.
   
   ```javascript
   const config = {};

   config.host = process.env.HOST || "[the endpoint URI of your Azure Cosmos DB account]";
   config.authKey =
     process.env.AUTH_KEY || "[the PRIMARY KEY value of your Azure Cosmos DB account";
   config.databaseId = "ToDoList";
   config.containerId = "Items";

   if (config.host.includes("https://localhost:")) {
     console.log("Local environment detected");
     console.log("WARNING: Disabled checking of self-signed certs. Do not have this code in production.");
     process.env.NODE_TLS_REJECT_UNAUTHORIZED = "0";
     console.log(`Go to http://localhost:${process.env.PORT || '3000'} to try the sample.`);
   }

   module.exports = config;
   ```

3. En el archivo **config.js**, actualice los valores de HOST y AUTH_KEY con los valores de la página Claves de la cuenta de Azure Cosmos DB en [Azure Portal](https://portal.azure.com). 

4. Guarde y cierre el archivo **config.js** .

### <a name="modify-appjs"></a>Modificar app.js

1. En el directorio del proyecto, abra el archivo **app.js** . Este archivo se creó anteriormente, cuando se creó la aplicación web de Express.  

2. Agregue el código siguiente al archivo **app.js**. Este código define el archivo de configuración que se va a utilizar y carga los valores en algunas variables que se utilizarán en las secciones siguientes. 
   
   ```javascript
    const CosmosClient = require('@azure/cosmos').CosmosClient
    const config = require('./config')
    const TaskList = require('./routes/tasklist')
    const TaskDao = require('./models/taskDao')

    const express = require('express')
    const path = require('path')
    const logger = require('morgan')
    const cookieParser = require('cookie-parser')
    const bodyParser = require('body-parser')

    const app = express()

    // view engine setup
    app.set('views', path.join(__dirname, 'views'))
    app.set('view engine', 'jade')

    // uncomment after placing your favicon in /public
    //app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
    app.use(logger('dev'))
    app.use(bodyParser.json())
    app.use(bodyParser.urlencoded({ extended: false }))
    app.use(cookieParser())
    app.use(express.static(path.join(__dirname, 'public')))

    //Todo App:
    const cosmosClient = new CosmosClient({
      endpoint: config.host,
      key: config.authKey
    })
    const taskDao = new TaskDao(cosmosClient, config.databaseId, config.containerId)
    const taskList = new TaskList(taskDao)
    taskDao
      .init(err => {
        console.error(err)
      })
      .catch(err => {
        console.error(err)
        console.error(
          'Shutting down because there was an error settinig up the database.'
        )
        process.exit(1)
      })

    app.get('/', (req, res, next) => taskList.showTasks(req, res).catch(next))
    app.post('/addtask', (req, res, next) => taskList.addTask(req, res).catch(next))
    app.post('/completetask', (req, res, next) =>
      taskList.completeTask(req, res).catch(next)
    )
    app.set('view engine', 'jade')

    // catch 404 and forward to error handler
    app.use(function(req, res, next) {
      const err = new Error('Not Found')
      err.status = 404
      next(err)
    })

    // error handler
    app.use(function(err, req, res, next) {
      // set locals, only providing error in development
      res.locals.message = err.message
      res.locals.error = req.app.get('env') === 'development' ? err : {}

      // render the error page
      res.status(err.status || 500)
      res.render('error')
    })

    module.exports = app
   ```

3. Por último, guarde y cierre el archivo **app.js**.

## <a name="build-a-user-interface"></a><a name="_Toc395783181"></a>Creación de una interfaz de usuario

Ahora vamos a crear la interfaz de usuario para que un usuario pueda interactuar con la aplicación. La aplicación Express que hemos creado en las secciones anteriores utiliza **Jade** como motor de vistas.

1. El archivo **layout.jade** del directorio **views** se utiliza como plantilla global para otros archivos **.jade**. En este paso podrá modificarlo para utilizar Twitter Bootstrap, que es un kit de herramientas que se utiliza para diseñar un sitio web.  

2. Abra el archivo **layout.jade** que se encuentra en la carpeta **views** y reemplace el contenido por el código siguiente:

   ```html
   doctype html
   html
     head
       title= title
       link(rel='stylesheet', href='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/css/bootstrap.min.css')
       link(rel='stylesheet', href='/stylesheets/style.css')
     body
       nav.navbar.navbar-inverse.navbar-fixed-top
         div.navbar-header
           a.navbar-brand(href='#') My Tasks
       block content
       script(src='//ajax.aspnetcdn.com/ajax/jQuery/jquery-1.11.2.min.js')
       script(src='//ajax.aspnetcdn.com/ajax/bootstrap/3.3.2/bootstrap.min.js')
   ```

    Este código indica al motor **Jade** que represente HTML de nuestra aplicación y que cree un **block** llamado **content** donde podemos especificar el diseño de nuestras páginas de contenido. Guarde y cierre este archivo **layout.jade**.

3. Ahora abra el archivo **index.jade**, la vista que utilizará nuestra aplicación, y reemplace el contenido del archivo por el código siguiente:

   ```html
   extends layout
   block content
        h1 #{title}
        br
        
        form(action="/completetask", method="post")
         table.table.table-striped.table-bordered
            tr
              td Name
              td Category
              td Date
              td Complete
            if (typeof tasks === "undefined")
              tr
                td
            else
              each task in tasks
                tr
                  td #{task.name}
                  td #{task.category}
                  - var date  = new Date(task.date);
                  - var day   = date.getDate();
                  - var month = date.getMonth() + 1;
                  - var year  = date.getFullYear();
                  td #{month + "/" + day + "/" + year}
                  td
                   if(task.completed) 
                    input(type="checkbox", name="#{task.id}", value="#{!task.completed}", checked=task.completed)
                   else
                    input(type="checkbox", name="#{task.id}", value="#{!task.completed}", checked=task.completed)
          button.btn.btn-primary(type="submit") Update tasks
        hr
        form.well(action="/addtask", method="post")
          label Item Name:
          input(name="name", type="textbox")
          label Item Category:
          input(name="category", type="textbox")
          br
          button.btn(type="submit") Add item
   ```

Este código amplía el diseño y proporciona contenido para el marcador de posición **content** que hemos visto anteriormente en el archivo **layout.jade**. En este diseño hemos creado dos formularios HTML.

El primer formulario contiene una tabla para nuestros datos y un botón que nos permite actualizar los elementos mediante la publicación en el método **/completeTask** de nuestro controlador.
    
El segundo formulario contiene dos campos de entrada y un botón que nos permite crear un nuevo elemento publicando en el método **/addtask** de nuestro controlador. Eso es todo lo que necesitamos para que funcione la aplicación.

## <a name="run-your-application-locally"></a><a name="_Toc395783181"></a>Ejecución de la aplicación de forma local

Ahora que compiló la aplicación, puede ejecutarla localmente con los pasos siguientes:  

1. Para probar la aplicación en el equipo local, ejecute `npm start` en el terminal para iniciar la aplicación y, a continuación, actualice la página `http://localhost:3000` del explorador. La página debe parecerse a lo que se muestra en la siguiente captura de pantalla:
   
    :::image type="content" source="./media/sql-api-nodejs-application/cosmos-db-node-js-localhost.png" alt-text="Captura de pantalla de la aplicación MyTodo List en una ventana del explorador":::

    > [!TIP]
    > Si recibe un error acerca de la sangría en el archivo layout.jade o el archivo index.jade, asegúrese de que las dos primeras líneas de los dos archivos están justificadas a la izquierda, sin espacios en blanco. Si hay espacios delante de las dos primeras líneas, quítelos, guarde ambos archivos y, a continuación, actualice la ventana del explorador. 

2. Utilice los campos Elemento, Nombre del elemento y Categoría para especificar una nueva tarea y, a continuación, seleccione **Agregar elemento**. Esto crea un documento en Azure Cosmos DB con esas propiedades. 

3. La página debería actualizarse para mostrar el elemento recién creado en la lista ToDo.
   
    :::image type="content" source="./media/sql-api-nodejs-application/cosmos-db-node-js-added-task.png" alt-text="Captura de pantalla de la aplicación con un nuevo elemento en la lista de tareas pendientes":::

4. Para completar una tarea, active la casilla en la columna Completo y, a continuación, seleccione **Actualizar tareas**. Se actualizará el documento que ya haya creado y lo quitará de la vista.

5. Para detener la aplicación, presione CTRL+C en la ventana de terminal y, después, seleccione **Y** para finalizar el trabajo por lotes.

## <a name="deploy-your-application-to-web-apps"></a><a name="_Toc395783182"></a>Implementación de la aplicación en Web Apps

Una vez que la aplicación de ejecuta correctamente de manera local, puede implementarla en Azure siguiendo estos pasos:

1. Habilite un repositorio de Git para la aplicación de Web Apps si todavía no lo ha hecho.

2. Agregue la aplicación de Web Apps como Git remoto.
   
   ```bash
   git remote add azure https://username@your-azure-website.scm.azurewebsites.net:443/your-azure-website.git
   ```

3. Implemente la aplicación llevándola al servidor remoto.
   
   ```bash
   git push azure master
   ```

4. En unos segundos, la aplicación web se publica y se inicia en un explorador.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite estos recursos, podrá eliminar el grupo de recursos, la cuenta de Azure Cosmos DB y todos los recursos relacionados. Para hacerlo, seleccione el grupo de recursos que usó en la cuenta de Azure Cosmos DB, seleccione **Eliminar** y confirme el nombre del grupo de recursos que se va a eliminar.

## <a name="next-steps"></a><a name="_Toc395637775"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de aplicaciones móviles con Xamarin y Azure Cosmos DB](mobile-apps-with-xamarin.md)


[Node.js]: https://nodejs.org/
[Git]: https://git-scm.com/
[GitHub]: https://github.com/Azure-Samples/azure-cosmos-db-sql-api-nodejs-todo-app

