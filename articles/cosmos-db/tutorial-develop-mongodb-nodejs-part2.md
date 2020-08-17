---
title: Creación de una aplicación Node.js Express con la API de Azure Cosmos DB para MongoDB (segunda parte)
description: Parte 2 de la serie de tutoriales en la que se explican cómo crear una aplicación de MongoDB con Angular y Node en Azure Cosmos DB utilizando las mismas API que para MongoDB.
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18
ms.reviewer: sngun
ms.openlocfilehash: a796ae32033dbc09b1f5b888ded9ea5e1f4c457e
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86523492"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---create-a-nodejs-express-app"></a>Creación de una aplicación de Angular con la API de Azure Cosmos DB para MongoDB: Creación de una aplicación de Node.js Express

En este tutorial de varias partes se muestra cómo crear una nueva aplicación escrita en Node.js con Express y Angular y cómo conectarla a la [cuenta de Cosmos configurada con la API de Cosmos DB para MongoDB](mongodb-introduction.md).

La parte 2 del tutorial se basa en [la introducción](tutorial-develop-mongodb-nodejs.md) y aborda las tareas siguientes:

> [!div class="checklist"]
> * Instalación de la CLI de Angular y TypeScript
> * Creación de un nuevo proyecto mediante Angular
> * Creación de la aplicación mediante la plataforma Express
> * Prueba de la aplicación en Postman

## <a name="video-walkthrough"></a>Tutorial en vídeo

> [!VIDEO https://www.youtube.com/embed/lIwJIYcGSUg]

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar esta parte del tutorial, asegúrese de que ha visto el [vídeo de introducción](tutorial-develop-mongodb-nodejs.md).

En este tutorial también se necesita: 
* [Node.js](https://nodejs.org/) versión 8.4.0 o superior.
* [Postman](https://www.getpostman.com/)
* [Visual Studio Code](https://code.visualstudio.com/) o su editor favorito de código.

> [!TIP]
> Este tutorial le guía por los pasos necesarios para crear la aplicación paso a paso. Si desea descargar el proyecto finalizado, puede obtener la aplicación completa en el [repositorio de angular-cosmosdb](https://github.com/Azure-Samples/angular-cosmosdb) en GitHub.

## <a name="install-the-angular-cli-and-typescript"></a>Instalación de la CLI de Angular y TypeScript

1. Abra una ventana del símbolo del sistema de Windows o de Terminal de Mac e instale la CLI de Angular.

    ```bash
    npm install -g @angular/cli
    ```

2. Instale TypeScript escribiendo el comando siguiente en el símbolo del sistema. 

    ```bash
    npm install -g typescript
    ```

## <a name="use-the-angular-cli-to-create-a-new-project"></a>Uso de la CLI de Angular para crear un nuevo proyecto

1. En el símbolo del sistema, cambie a la carpeta donde desea crear el proyecto nuevo, a continuación, ejecute el siguiente comando. Este comando crea una nueva carpeta y un proyecto llamado angular-cosmosdb e instala los componentes de Angular requeridos para una nueva aplicación. Utiliza la configuración mínima (--minimal) y especifica que el proyecto utiliza Sass (una sintaxis similar a CSS con la marca --style scss).

    ```bash
    ng new angular-cosmosdb --minimal --style scss
    ```

2. Una vez finalizado el comando, cambie los directorios a la carpeta src/client.

    ```bash
    cd angular-cosmosdb
    ```

3. A continuación, abra la carpeta en Visual Studio Code.

    ```bash
    code .
    ```

## <a name="build-the-app-using-the-express-framework"></a>Creación de aplicación mediante la plataforma Express

1. En Visual Studio Code, en el panel **Explorer**, haga clic con el botón derecho en la carpeta **src**, haga clic en **Nueva carpeta** y asigne el nombre *server* a la nueva carpeta.

2. En el panel **Explorer**, haga clic con el botón derecho en la carpeta **server**, haga clic en **Nuevo archivo** y asigne el nombre *index.js* al nuevo archivo.

3. En el símbolo del sistema, utilice el siguiente comando para instalar el analizador de cuerpo. Esto ayuda a la aplicación a analizar los datos JSON que se pasan a través de las API.

    ```bash
    npm i express body-parser --save
    ```

4. En Visual Studio Code, copie el código siguiente en el archivo index.js. Este código:
    * Hace referencia a Express
    * Extrae el analizador de cuerpo para leer datos JSON en el cuerpo de las solicitudes
    * Usa una característica integrada llamada path
    * Establece variables raíz para facilitar la búsqueda de la ubicación del código
    * Configura un puerto
    * Inicia Express
    * Indica a la aplicación cómo utilizar el middleware que se va a usar para atender el servidor
    * Sirve todo lo que se encuentra en la carpeta dist, que será el contenido estático
    * Sirve la aplicación y sirve index.html para cualquier solicitud GET que no se encuentre en el servidor (para vínculos profundos)
    * Inicia el servidor con app.listen
    * Utiliza una función de flecha para registrar que el puerto está activo
    
   ```node
   const express = require('express');
   const bodyParser = require('body-parser');
   const path = require('path');
   const routes = require('./routes');

   const root = './';
   const port = process.env.PORT || '3000';
   const app = express();

   app.use(bodyParser.json());
   app.use(bodyParser.urlencoded({ extended: false }));
   app.use(express.static(path.join(root, 'dist/angular-cosmosdb')));
   app.use('/api', routes);
   app.get('*', (req, res) => {
     res.sendFile('dist/angular-cosmosdb/index.html', {root});
   });

   app.listen(port, () => console.log(`API running on localhost:${port}`));
   ```

5. En Visual Studio Code, en el panel **Explorer**, haga clic con el botón derecho en la carpeta **server** y, a continuación, haga clic en **Nuevo archivo**. Asigne el nombre *routes.js* al archivo nuevo. 

6. Copie el código siguiente en **routes.js**. Este código:
   * Hace referencia el enrutador de Express
   * Obtiene los héroes
   * Devuelve el contenido JSON para un hérore definido

   ```node
   const express = require('express');
   const router = express.Router();

   router.get('/heroes', (req, res) => {
    res.send(200, [
       {"id": 10, "name": "Starlord", "saying": "oh yeah"}
    ])
   });

   module.exports=router;
   ```

7. Guarde todos los archivos modificados. 

8. En Visual Studio Code, haga clic en el botón **Depurar** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/debug-button.png"::: y haga clic en el botón de engranaje :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/gear-button.png":::. El nuevo archivo launch.json se abre en Visual Studio Code. 

8. En la línea 11 del archivo launch.json, cambie `"${workspaceFolder}\\server"` por `"program": "${workspaceRoot}/src/server/index.js"` y guarde el archivo.

9. Haga clic en el botón **Iniciar depuración** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part2/start-debugging-button.png"::: para ejecutar la aplicación.

    La aplicación se debe ejecutar sin errores.

## <a name="use-postman-to-test-the-app"></a>Uso de Postman para probar la aplicación

1. Ahora, abra Postman y escriba `http://localhost:3000/api/heroes` en el cuadro GET. 

2. Haga clic en el botón **Enviar** y obtenga la respuesta JSON de la aplicación. 

    Esta respuesta muestra que la aplicación está activa y se está ejecutando localmente. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part2/azure-cosmos-db-postman.png" alt-text="Postman muestra la solicitud y la respuesta":::


## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha hecho lo siguiente:

> [!div class="checklist"]
> * Ha creado un proyecto de Node.js mediante la CLI de Angular
> * Ha probado la aplicación con Postman

Puede continuar con la siguiente parte del tutorial para crear la interfaz de usuario.

> [!div class="nextstepaction"]
> [Creación de la interfaz de usuario con Angular](tutorial-develop-mongodb-nodejs-part3.md)
