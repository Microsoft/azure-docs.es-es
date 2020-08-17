---
title: Incorporación de funciones CRUD a una aplicación Angular con la API de Azure Cosmos DB para MongoDB
description: Parte 6 de la serie de tutoriales en la que se explica cómo crear una aplicación de MongoDB con Angular y Node en Azure Cosmos DB utilizando las mismas API que para MongoDB
author: johnpapa
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 12/26/2018
ms.author: jopapa
ms.custom: seodec18, devx-track-javascript
ms.reviewer: sngun
ms.openlocfilehash: a5d697465f811f468fb0047284126deae0c455dd
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87424314"
---
# <a name="create-an-angular-app-with-azure-cosmos-dbs-api-for-mongodb---add-crud-functions-to-the-app"></a>Creación de una aplicación de Angular con la API de Azure Cosmos DB para MongoDB: Incorporación de funciones CRUD a la aplicación

En este tutorial de varias partes se muestra cómo crear una nueva aplicación escrita en Node.js con Express y Angular y cómo conectarla a la [cuenta de Cosmos configurada con la API de Cosmos DB para MongoDB](mongodb-introduction.md). La parte 6 del tutorial se basa en la [parte 5](tutorial-develop-mongodb-nodejs-part5.md) y aborda las tareas siguientes:

> [!div class="checklist"]
> * Creación de las funciones Post, Put y Delete para el servicio Hero
> * Ejecución la aplicación

> [!VIDEO https://www.youtube.com/embed/Y5mdAlFGZjc]

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar esta parte del tutorial, asegúrese de que ha completado los pasos descritos en la [parte 5](tutorial-develop-mongodb-nodejs-part5.md) del tutorial.

> [!TIP]
> Este tutorial le guía por los pasos necesarios para crear la aplicación paso a paso. Si desea descargar el proyecto finalizado, puede obtener la aplicación completa en el [repositorio de angular-cosmosdb](https://github.com/Azure-Samples/angular-cosmosdb) en GitHub.

## <a name="add-a-post-function-to-the-hero-service"></a>Incorporación de una función Post al servicio Hero

1. En Visual Studio Code, abra **routes.js** y **hero.service.js** al mismo tiempo; para ello presione el botón **Dividir editor** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part6/split-editor-button.png":::.

    Observe que la línea 7 de routes.js está llamando a la función `getHeroes` en la línea 5 de **hero.service.js**.  Necesitamos crear este mismo emparejamiento para las funciones post, put y delete. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/routes-heroservicejs.png" alt-text="routes.js y hero.service.js en Visual Studio Code":::
    
    Empecemos por codificar el servicio Hero. 

2. Copie el código siguiente en **hero.service.js** después de la función `getHeroes` y antes de `module.exports`. Este código:  
   * Usa el modelo de Hero para registrar un nuevo componente Hero.
   * Comprueba las respuestas para ver si hay un error y devuelve el valor de estado 500.

   ```javascript
   function postHero(req, res) {
     const originalHero = { uid: req.body.uid, name: req.body.name, saying: req.body.saying };
     const hero = new Hero(originalHero);
     hero.save(error => {
       if (checkServerError(res, error)) return;
       res.status(201).json(hero);
       console.log('Hero created successfully!');
     });
   }

   function checkServerError(res, error) {
     if (error) {
       res.status(500).send(error);
       return error;
     }
   }
   ```

3. En **hero.service.js**, actualice `module.exports` para incluir la nueva función `postHero`. 

    ```javascript
    module.exports = {
      getHeroes,
      postHero
    };
    ```

4. En **routes.js**, agregue un enrutador para la función `post` después del enrutador `get`. Este enrutador publica los héroes de uno en uno. Al estructurar el archivo de enrutador de esta forma, se muestran limpiamente todos los puntos de conexión de la API disponibles y se deja el trabajo real para el archivo **hero.service.js**.

    ```javascript
    router.post('/hero', (req, res) => {
      heroService.postHero(req, res);
    });
    ```

5. Ejecute la aplicación para comprobar que todo funcionó. En Visual Studio Code, guarde todos los cambios, seleccione el botón **Depurar** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part6/debug-button.png"::: de la izquierda e **Iniciar depuración** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part6/start-debugging-button.png":::.

6. Ahora vuelva atrás en el explorador de Internet y abra la pestaña Red de las herramientas de desarrollador; en la mayoría de los equipos, debe presionar F12. Vaya a `http://localhost:3000` para ver las llamadas realizadas a través de la red.

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/add-new-hero.png" alt-text="Pestaña Red de Chrome que muestra la actividad de red":::

7. Agregue un nuevo héroe; para ello, seleccione el botón **Add New Hero** (Agregar nuevo héroe). Escriba el identificador "999", el nombre "Fred" y el mensaje "Hello"; y haga clic en **Guardar**. En la pestaña Red debería ver que ha enviado una solicitud POST para el nuevo héroe. 

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/post-new-hero.png" alt-text="Pestaña Red de Chrome que muestra la actividad de red para las funciones Get y Post":::

    Ahora volvamos y agregue las funciones Put y Delete a la aplicación.

## <a name="add-the-put-and-delete-functions"></a>Incorporación de las funciones Put y Delete

1. En **routes.js**, agregue los enrutadores `put` y `delete` tras el enrutador post.

    ```javascript
    router.put('/hero/:uid', (req, res) => {
      heroService.putHero(req, res);
    });

    router.delete('/hero/:uid', (req, res) => {
      heroService.deleteHero(req, res);
    });
    ```

2. Copie el código siguiente en **hero.service.js** después de la función `checkServerError`. Este código:
   * Crea las funciones `put` y `delete`
   * Comprueba si se ha encontrado el héroe
   * Realiza el control de errores 

   ```javascript
   function putHero(req, res) {
     const originalHero = {
       uid: parseInt(req.params.uid, 10),
       name: req.body.name,
       saying: req.body.saying
     };
     Hero.findOne({ uid: originalHero.uid }, (error, hero) => {
       if (checkServerError(res, error)) return;
       if (!checkFound(res, hero)) return;

      hero.name = originalHero.name;
       hero.saying = originalHero.saying;
       hero.save(error => {
         if (checkServerError(res, error)) return;
         res.status(200).json(hero);
         console.log('Hero updated successfully!');
       });
     });
   }

   function deleteHero(req, res) {
     const uid = parseInt(req.params.uid, 10);
     Hero.findOneAndRemove({ uid: uid })
       .then(hero => {
         if (!checkFound(res, hero)) return;
         res.status(200).json(hero);
         console.log('Hero deleted successfully!');
       })
       .catch(error => {
         if (checkServerError(res, error)) return;
       });
   }

   function checkFound(res, hero) {
     if (!hero) {
       res.status(404).send('Hero not found.');
       return;
     }
     return hero;
   }
   ```

3. En **hero.service.js**, exporte los nuevos módulos:

   ```javascript
    module.exports = {
      getHeroes,
      postHero,
      putHero,
      deleteHero
    };
    ```

4. Ahora que hemos actualizado el código, seleccione el botón **Reiniciar** :::image type="icon" source="./media/tutorial-develop-mongodb-nodejs-part6/restart-debugger-button.png"::: en Visual Studio Code.

5. Actualice la página en el explorador de Internet y seleccione el botón **Add New Hero** (Agregar nuevo héroe). Agregue un nuevo héroe con el identificador "9", el nombre "Starlord" y el mensaje "Hello". Haga clic en el botón **Guardar** para guardar el nuevo héroe.

6. Ahora seleccione el héroe **Starlord** y cambie el mensaje de "Hi" a "Bye"; después, seleccione el botón **Guardar**. 

    Ahora puede seleccionar el identificador en la pestaña Red para mostrar la carga. En la carga puede ver que el mensaje ahora está establecido en "Bye".

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/put-hero-function.png" alt-text="Aplicación Heroes y pestaña Red que muestra la carga"::: 

    También puede eliminar uno de los héroes de la interfaz de usuario y ver el tiempo que se tarda en completar la operación de eliminación. Para probarlo, seleccione el botón "Eliminar" del héroe llamado "Fred".

    :::image type="content" source="./media/tutorial-develop-mongodb-nodejs-part6/times.png" alt-text="Aplicación Heroes y la pestaña Red que muestra el tiempo para completar las funciones"::: 

    Si actualiza la página, la pestaña Red muestra el tiempo que se tarda en obtener los héroes. Aunque estos tiempos son cortos, en gran medida ello depende de dónde estén ubicados los datos en el mundo y de su capacidad para replicarlos geográficamente en un área cerca de los usuarios. Puede encontrar más información acerca de la replicación geográfica en el tutorial siguiente, que estará disponible próximamente.

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha hecho lo siguiente:

> [!div class="checklist"]
> * Agregó las funciones Post, Put y Delete a la aplicación 

Compruebe pronto si hay otros vídeos en esta serie de tutoriales.

