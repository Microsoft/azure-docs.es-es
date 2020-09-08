---
title: 'Tutorial: Aplicación Node.js con MongoDB'
description: Aprenda a poner en funcionamiento una aplicación de Node.js en Azure, con conexión a una base de datos de MongoDB en Azure (Cosmos DB). MEAN.js se utiliza en el tutorial.
ms.assetid: 0b4d7d0e-e984-49a1-a57a-3c0caa955f0e
ms.devlang: nodejs
ms.topic: tutorial
ms.date: 06/16/2020
ms.custom: mvc, cli-validate, seodec18, devx-track-javascript
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: a409af7468daf2ac9de731bcdba5e8f355422ddd
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88082526"
---
# <a name="tutorial-build-a-nodejs-and-mongodb-app-in-azure"></a>Tutorial: Compilación de una aplicación Node.js y MongoDB en Azure

::: zone pivot="platform-windows"  

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático. En este tutorial se muestra cómo crear una aplicación de Node.js en App Service en Windows y conectarla a una base de datos de MongoDB. Cuando haya terminado, tendrá una aplicación MEAN (MongoDB, Express, AngularJS y Node.js) que se ejecuta en [Azure App Service](overview.md). Por sencillez, la aplicación de ejemplo usa el [marco web MEAN.js ](https://meanjs.org/).

::: zone-end

::: zone pivot="platform-linux"


[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación automática de revisiones con el sistema operativo Linux. En este tutorial se muestra cómo crear una aplicación de Node.js en App Service en Linux, conectarla localmente a una base de datos de MongoDB y luego implementarla en una base de datos en la API de Azure Cosmos DB para MongoDB. Cuando haya terminado, tendrá una aplicación MEAN (MongoDB, Express, AngularJS y Node.js) que se ejecuta en App Service en Linux. Por sencillez, la aplicación de ejemplo usa el [marco web MEAN.js ](https://meanjs.org/).

::: zone-end

![Aplicación MEAN.js que se ejecuta en Azure App Service](./media/tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Temas que se abordarán:

> [!div class="checklist"]
> * Crear una base de datos MongoDB en Azure
> * Conectar una aplicación Node.js a MongoDB
> * Implementar la aplicación en Azure
> * Actualizar el modelo de datos y volver a implementar la aplicación
> * Transmitir registros de diagnóstico desde Azure
> * Administrar la aplicación en Azure Portal

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

1. [Instalación de Git](https://git-scm.com/)
2. [Instalación de Node.js y NPM](https://nodejs.org/)
3. [Instale Bower](https://bower.io/) (necesario para [MEAN.js](https://meanjs.org/docs/0.5.x/#getting-started))
4. [Instale Gulp.js](https://gulpjs.com/) (necesario para [MEAN.js](https://meanjs.org/docs/0.5.x/#getting-started))
5. [Descarga y ejecución de MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/) 

## <a name="test-local-mongodb"></a>Prueba de la base de datos MongoDB local

Abra la ventana de terminal y use `cd` para cambiar al directorio `bin` de la instalación de MongoDB. Esta ventana de terminal se puede usar para ejecutar todos los comandos de este tutorial.

Ejecute `mongo` en el terminal para conectarse a su servidor local de MongoDB.

```bash
mongo
```

Si la conexión se realiza correctamente, la base de datos de MongoDB estará ya funcionando. En caso contrario, asegúrese de que la base de datos de MongoDB local se inicia con los pasos descritos en [Install MongoDB Community Edition](https://docs.mongodb.com/manual/administration/install-community/) (Instalación de MongoDB Community Edition). Con frecuencia, MongoDB ya estará instalado, pero aun así deberá iniciarlo ejecutando `mongod`. 

Cuando haya terminado de probar la base de datos de MongoDB, escriba `Ctrl+C` en el terminal. 

## <a name="create-local-nodejs-app"></a>Creación de una aplicación local Node.js

En este paso, configurará el proyecto local de Node.js.

### <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

En la ventana del terminal, use `cd` para cambiar a un directorio de trabajo.  

Ejecute el comando siguiente para clonar el repositorio de ejemplo. 

```bash
git clone https://github.com/Azure-Samples/meanjs.git
```

Este repositorio de ejemplo contiene una copia del [repositorio MEAN.js](https://github.com/meanjs/mean). Se ha modificado para ejecutarse App Service (para más información, consulte el [archivo LÉAME](https://github.com/Azure-Samples/meanjs/blob/master/README.md) del repositorio MEAN.js).

### <a name="run-the-application"></a>Ejecución de la aplicación

Instale los comandos siguientes para instalar los paquetes necesarios e inicie la aplicación.

```bash
cd meanjs
npm install
npm start
```

Ignore la advertencia de config.domain. Cuando la aplicación se haya cargado completamente, verá algo similar al siguiente mensaje:

<pre>
--
MEAN.JS: entorno de desarrollo

Entorno: servidor de desarrollo:          http://0.0.0.0:3000 Base de datos: mongodb://localhost/mean-dev Versión de la aplicación:     0.5.0 MEAN.JS versión: 0.5.0 --
</pre>

Vaya a `http://localhost:3000` en un explorador. Haga clic en **Registrarse** en el menú superior y cree un usuario de prueba. 

La aplicación de ejemplo MEAN.js almacena datos de usuario en la base de datos. Si crea un usuario e inicia sesión con él correctamente, la aplicación escribe datos en la base de datos de MongoDB local.

![MEAN.js se conecta correctamente a MongoDB](./media/tutorial-nodejs-mongodb-app/mongodb-connect-success.png)

Seleccione **Admin (Administrador) > Manage Articles (Administrar artículos)** para agregar algunos artículos.

Para detener Node.js en cualquier momento, presione `Ctrl+C` en el terminal. 

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-production-mongodb"></a>Creación de una base de datos de MongoDB de producción

En este paso, creará una base de datos MongoDB en Azure. Cuando la aplicación se implementa en Azure, utiliza esta base de datos en la nube.

Para MongoDB, en este tutorial se usa [Azure Cosmos DB](/azure/documentdb/). Cosmos DB admite las conexiones del cliente de MongoDB.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)] 

### <a name="create-a-cosmos-db-account"></a>Creación de una cuenta de Cosmos DB

> [!NOTE]
> Hay un costo por la creación de las bases de datos de Azure Cosmos DB en este tutorial en su propia suscripción de Azure. Para usar una cuenta gratuita de Azure Cosmos DB durante siete días, puede usar la experiencia [Pruebe gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/). Simplemente haga clic en el botón **Crear** en el icono de MongoDB para crear una base de datos gratuita de MongoDB en Azure. Una vez creada la base de datos, vaya a **Cadena de conexión** en el portal y recupere la cadena de conexión de Azure Cosmos DB para su uso posterior en el tutorial.
>

En Cloud Shell, cree una cuenta de Cosmos DB con el comando [`az cosmosdb create`](/cli/azure/cosmosdb?view=azure-cli-latest#az-cosmosdb-create).

En el siguiente comando, sustituya el marcador de posición *\<cosmosdb-name>* por un nombre único de base de datos de Cosmos DB. Este nombre se usa como parte del punto de conexión de Cosmos DB, `https://<cosmosdb-name>.documents.azure.com/`, por lo que el nombre debe ser único en todas las cuentas de Cosmos DB de Azure. El nombre debe contener solo letras minúsculas, números y el carácter de guion (-), y debe tener una longitud de entre 3 y 50 caracteres.

```azurecli-interactive
az cosmosdb create --name <cosmosdb-name> --resource-group myResourceGroup --kind MongoDB
```

El parámetro *--kind MongoDB* habilita las conexiones de cliente de MongoDB.

Cuando se crea la cuenta de Cosmos DB, la CLI de Azure muestra información similar a la del ejemplo siguiente:

<pre>
{
  "consistencyPolicy":
  {
    "defaultConsistencyLevel": "Session",
    "maxIntervalInSeconds": 5,
    "maxStalenessPrefix": 100
  },
  "databaseAccountOfferType": "Standard",
  "documentEndpoint": "https://&lt;cosmosdb-name&gt;.documents.azure.com:443/",
  "failoverPolicies": 
  ...
  &lt; Output truncated for readability &gt;
}
</pre>

## <a name="connect-app-to-production-mongodb"></a>Conexión de la aplicación a la base de datos MongoDB de producción

En este paso, conectará la aplicación de ejemplo MEAN.js a la base de datos Cosmos DB que acaba de crear mediante una cadena de conexión de MongoDB. 

### <a name="retrieve-the-database-key"></a>Recuperación de la clave de base de datos

Para conectarse a la base de datos Cosmos DB, necesita la clave de base de datos. En Cloud Shell, use el comando [`az cosmosdb list-keys`](/cli/azure/cosmosdb?view=azure-cli-latest#az-cosmosdb-list-keys) para recuperar la clave principal.

```azurecli-interactive
az cosmosdb list-keys --name <cosmosdb-name> --resource-group myResourceGroup
```

La CLI de Azure muestra información similar a la del ejemplo siguiente:

<pre>
{
  "primaryMasterKey": "RS4CmUwzGRASJPMoc0kiEvdnKmxyRILC9BWisAYh3Hq4zBYKr0XQiSE4pqx3UchBeO4QRCzUt1i7w0rOkitoJw==",
  "primaryReadonlyMasterKey": "HvitsjIYz8TwRmIuPEUAALRwqgKOzJUjW22wPL2U8zoMVhGvregBkBk9LdMTxqBgDETSq7obbwZtdeFY7hElTg==",
  "secondaryMasterKey": "Lu9aeZTiXU4PjuuyGBbvS1N9IRG3oegIrIh95U6VOstf9bJiiIpw3IfwSUgQWSEYM3VeEyrhHJ4rn3Ci0vuFqA==",
  "secondaryReadonlyMasterKey": "LpsCicpVZqHRy7qbMgrzbRKjbYCwCKPQRl0QpgReAOxMcggTvxJFA94fTi0oQ7xtxpftTJcXkjTirQ0pT7QFrQ=="
}
</pre>

Copie el valor de `primaryMasterKey`. Esta información la necesitará en el siguiente paso.

<a name="devconfig"></a>
### <a name="configure-the-connection-string-in-your-nodejs-application"></a>Configuración de la cadena de conexión en la aplicación Node.js

En la carpeta _config/env/_ del repositorio MEAN.js local, cree un archivo llamado _local-production.js_. _.gitignore_ ya está configurado para mantener este archivo fuera del repositorio. 

Copie en él el código siguiente. Asegúrese de reemplazar los dos marcadores de posición *\<cosmosdb-name>* por el nombre de la base de datos de Cosmos DB y el marcador de posición *\<primary-master-key>* por la clave que copió en el paso anterior.

```javascript
module.exports = {
  db: {
    uri: 'mongodb://<cosmosdb-name>:<primary-master-key>@<cosmosdb-name>.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false'
  }
};
```

La opción `ssl=true` se requiere porque [Cosmos DB requiere SSL](../cosmos-db/connect-mongodb-account.md#connection-string-requirements). 

Guarde los cambios.

### <a name="test-the-application-in-production-mode"></a>Prueba de la aplicación en modo de producción 

En una ventana de terminal local, ejecute el siguiente comando para minimizar y agrupar scripts para el entorno de producción. Este proceso genera los archivos que necesita dicho entorno.

```bash
gulp prod
```

En la ventana de terminal local, ejecute el siguiente comando para usar la cadena de conexión que configuró en _config/env/local-production.js_. Omita el error de certificado y la advertencia de config.domain.

```bash
# Bash
NODE_ENV=production node server.js

# Windows PowerShell
$env:NODE_ENV = "production" 
node server.js
```

`NODE_ENV=production` establece la variable de entorno que indica a Node.js que se ejecute en el entorno de producción.  `node server.js` inicia el servidor de Node.js con `server.js` en la raíz del repositorio. Así es como se carga la aplicación de Node.js en Azure. 

Cuando se cargue la aplicación, compruebe que se ejecuta en el entorno de producción:

<pre>
--
MEAN.JS

Entorno:        producción Servidor:          http://0.0.0.0:8443 Database:        mongodb://&lt; cosmosdb-name&gt;:&lt; primary-master-key&gt;@&lt; cosmosdb-name&gt;.documents.azure.com:10250/mean?ssl=true&sslverifycertificate=false Versión de la aplicación:     0.5.0 MEAN.JS versión: 0.5.0
</pre>

Vaya a `http://localhost:8443` en un explorador. Haga clic en **Registrarse** en el menú superior y cree un usuario de prueba. Si crea un usuario e inicia sesión con él correctamente, la aplicación escribe datos en la base de datos de Cosmos DB en Azure. 

En el terminal, detenga Node.js escribiendo `Ctrl+C`. 

## <a name="deploy-app-to-azure"></a>Implementación de la aplicación en Azure

En este paso, implementará la aplicación Node.js conectada a MongoDB en Azure App Service.

### <a name="configure-a-deployment-user"></a>Configuración de un usuario de implementación

[!INCLUDE [Configure deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

::: zone pivot="platform-windows"  

[!INCLUDE [Create app service plan no h](../../includes/app-service-web-create-app-service-plan-no-h.md)]

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-linux-no-h.md)]

::: zone-end

<a name="create"></a>
### <a name="create-a-web-app"></a>Creación de una aplicación web

::: zone pivot="platform-windows"  

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-nodejs-no-h.md)] 

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-nodejs-linux-no-h.md)] 

::: zone-end

### <a name="configure-an-environment-variable"></a>Configuración de una variable de entorno

De forma predeterminada, el proyecto MEAN.js mantiene _config/env/local-production.js_ fuera del repositorio de Git. Por lo tanto, para su aplicación de Azure, use la configuración de aplicación para definir la cadena de conexión de MongoDB.

Para establecer la configuración de la aplicación, utilice el comando [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) en Cloud Shell. 

En el ejemplo siguiente se realiza una configuración de aplicación `MONGODB_URI` en la aplicación de Azure. De nuevo, reemplace los marcadores *\<app-name>* , *\<cosmosdb-name>* y *\<primary-master-key>* .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings MONGODB_URI="mongodb://<cosmosdb-name>:<primary-master-key>@<cosmosdb-name>.documents.azure.com:10250/mean?ssl=true"
```

En el código de Node.js, [acceda a la configuración de la aplicación](configure-language-nodejs.md#access-environment-variables) con `process.env.MONGODB_URI`, igual que accedería a cualquier variable de entorno. 

En el repositorio MEAN.js local, abra _config/env/production.js_ (no _config/env/local-production.js_), que tiene la configuración específica para el entorno de producción. La aplicación MEAN.js predeterminada ya está configurada para usar la variable de entorno `MONGODB_URI` que ha creado.

```javascript
db: {
  uri: ... || process.env.MONGODB_URI || ...,
  ...
},
```

### <a name="push-to-azure-from-git"></a>Inserción en Azure desde Git

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Counting objects: 5, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (5/5), done.
Writing objects: 100% (5/5), 489 bytes | 0 bytes/s, done.
Total 5 (delta 3), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '6c7c716eee'.
remote: Running custom deployment command...
remote: Running deployment command...
remote: Handling node.js deployment.
.
.
.
remote: Deployment successful.
To https://&lt;app-name&gt;.scm.azurewebsites.net/&lt;app-name&gt;.git
 * [new branch]      master -> master
</pre>

Puede observar que el proceso de implementación ejecuta [Gulp](https://gulpjs.com/) después de `npm install`. App Service no ejecuta tareas Gulp o Grunt durante la implementación, así que este repositorio de ejemplo tiene dos archivos adicionales en su directorio raíz para permitirlo: 

- _.deployment_: este archivo indica a App Service que ejecute `bash deploy.sh` como script de implementación personalizado.
- _deploy.sh_: se trata del script de implementación personalizado. Si revisa el archivo, verá que se ejecuta `gulp prod` después de `npm install` y `bower install`. 

Puede usar este enfoque para agregar cualquier paso a la implementación basada en Git. Si reinicia la aplicación de Azure en cualquier momento, App Service no vuelve a ejecutar estas tareas de automatización. Para más información, consulte [Ejecute Bower y Grunt/Gulp](configure-language-nodejs.md#run-gruntbowergulp).

### <a name="browse-to-the-azure-app"></a>Navegación hasta la aplicación de Azure 

Vaya a la aplicación implementada mediante el explorador web. 

```bash 
http://<app-name>.azurewebsites.net 
``` 

Haga clic en **Registrarse** en el menú superior y cree un usuario ficticio. 

Si se realiza correctamente y la aplicación inicia automáticamente la sesión del usuario creado, la aplicación MEAN.js de Azure tiene conectividad con la base de datos de MongoDB (Cosmos DB). 

![Aplicación MEAN.js que se ejecuta en Azure App Service](./media/tutorial-nodejs-mongodb-app/meanjs-in-azure.png)

Seleccione **Admin (Administrador) > Manage Articles (Administrar artículos)** para agregar algunos artículos. 

**¡Enhorabuena!** Está ejecutando una aplicación Node.js orientada a datos en Azure App Service.

## <a name="update-data-model-and-redeploy"></a>Actualización del modelo de datos y nueva implementación

En este paso, se cambia el modelo de datos de `article` y se publica el cambio en Azure.

### <a name="update-the-data-model"></a>Actualización del modelo de datos

En el repositorio MEAN.js local, abra _modules/articles/server/models/article.server.model.js_.

En `ArticleSchema`, agregue un tipo `String` llamado `comment`. Cuando haya finalizado, su código de esquema tendrá este aspecto:

```javascript
const ArticleSchema = new Schema({
  ...,
  user: {
    type: Schema.ObjectId,
    ref: 'User'
  },
  comment: {
    type: String,
    default: '',
    trim: true
  }
});
```

### <a name="update-the-articles-code"></a>Actualización del código de artículos

Actualice el resto del código de `articles` para usar `comment`.

Hay cinco archivos que es necesario modificar: el controlador del servidor y las cuatro vistas del cliente. 

Abra _modules/articles/server/controllers/articles.server.controller.js_.

En la función `update`, agregue una asignación para `article.comment`. El código siguiente muestra la función `update` completada:

```javascript
exports.update = function (req, res) {
  let article = req.article;

  article.title = req.body.title;
  article.content = req.body.content;
  article.comment = req.body.comment;

  ...
};
```

Abra _modules/articles/client/views/view-article.client.view.html_.

Justo antes de la etiqueta `</section>` de cierre, agregue la línea siguiente para mostrar `comment` junto con el resto de los datos de artículo:

```html
<p class="lead" ng-bind="vm.article.comment"></p>
```

Abra _modules/articles/client/views/list-articles.client.view.html_.

Justo antes de la etiqueta `</a>` de cierre, agregue la línea siguiente para mostrar `comment` junto con el resto de los datos de artículo:

```html
<p class="list-group-item-text" ng-bind="article.comment"></p>
```

Abra _modules/articles/client/views/admin/list-articles.client.view.html_.

Dentro del elemento `<div class="list-group">` e inmediatamente encima de la etiqueta `</a>` de cierre, agregue la siguiente línea para mostrar `comment` junto con el resto de los datos de artículo:

```html
<p class="list-group-item-text" data-ng-bind="article.comment"></p>
```

Abra _modules/articles/client/views/admin/form-article.client.view.html_.

Busque el elemento `<div class="form-group">` que contiene el botón de envío, que es similar a este:

```html
<div class="form-group">
  <button type="submit" class="btn btn-default">{{vm.article._id ? 'Update' : 'Create'}}</button>
</div>
```

Inmediatamente encima de esta etiqueta, agregue otro elemento `<div class="form-group">` que permita a los usuarios editar el campo `comment`. El nuevo elemento debe ser como este:

```html
<div class="form-group">
  <label class="control-label" for="comment">Comment</label>
  <textarea name="comment" data-ng-model="vm.article.comment" id="comment" class="form-control" cols="30" rows="10" placeholder="Comment"></textarea>
</div>
```

### <a name="test-your-changes-locally"></a>Prueba de los cambios localmente

Guarde todos los cambios.

En la ventana de terminal local, pruebe de nuevo los cambios en el modo de producción.

```bash
# Bash
gulp prod
NODE_ENV=production node server.js

# Windows PowerShell
gulp prod
$env:NODE_ENV = "production" 
node server.js
```

Vaya a `http://localhost:8443` en un explorador y asegúrese de que ha iniciado sesión.

Seleccione **Admin (Administrador) > Manage Articles (Administrar artículos)** y luego haga clic en el botón **+** para agregar un artículo.

Ahora verá el nuevo cuadro de texto `Comment`.

![Campo de comentario agregado a los artículos](./media/tutorial-nodejs-mongodb-app/added-comment-field.png)

En el terminal, detenga Node.js escribiendo `Ctrl+C`. 

### <a name="publish-changes-to-azure"></a>Publicación de los cambios en Azure

En la ventana del terminal local, confirme los cambios en Git e inserte los cambios de código en Azure.

```bash
git commit -am "added article comment"
git push azure master
```

Una vez que `git push` esté completo, vaya a la aplicación de Azure y pruebe la nueva funcionalidad.

![Modelo y cambios en la base de datos publicados en Azure](media/tutorial-nodejs-mongodb-app/added-comment-field-published.png)

Si agregó anteriormente artículos, aún puede verlos. Los datos existentes en Cosmos DB no se pierden. Además, se actualiza el esquema de datos y los datos existentes permanecen sin cambios.

## <a name="stream-diagnostic-logs"></a>Transmisión de registros de diagnóstico 

::: zone pivot="platform-windows"  

Mientras se ejecuta la aplicación de Node.js en Azure App Service, los registros de la consola se canalizan a su terminal. De este modo, puede obtener los mismos mensajes de diagnóstico para ayudarle a depurar errores de la aplicación.

Para iniciar la transmisión del registro, use el comando [`az webapp log tail`](/cli/azure/webapp/log?view=azure-cli-latest#az-webapp-log-tail) en Cloud Shell.

```azurecli-interactive
az webapp log tail --name <app-name> --resource-group myResourceGroup
``` 

Cuando se inicie la secuencia de registro, actualice la aplicación de Azure en el explorador para obtener algún tráfico web. Ahora verá los registros de la consola canalizados a su terminal.

Para detener las secuencias de registro en cualquier momento, escriba `Ctrl+C`. 

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

::: zone-end

## <a name="manage-your-azure-app"></a>Administración de la aplicación de Azure

Vaya a [Azure Portal](https://portal.azure.com) para ver la aplicación que ha creado.

En el menú izquierdo, haga clic en **App Services** y, luego, en el nombre de la aplicación de Azure.

![Navegación en el portal a la aplicación de Azure](./media/tutorial-nodejs-mongodb-app/access-portal.png)

De manera predeterminada, el portal muestra la página **Información general** de la aplicación. Esta página proporciona una visión del funcionamiento de la aplicación. En este caso, también puede realizar tareas de administración básicas como examinar, detener, iniciar, reiniciar y eliminar. Las pestañas del lado izquierdo de la página muestran las diferentes páginas de configuración que puede abrir.

![Página de App Service en Azure Portal](./media/tutorial-nodejs-mongodb-app/web-app-blade.png)

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>Pasos siguientes

¿Qué ha aprendido?

> [!div class="checklist"]
> * Crear una base de datos MongoDB en Azure
> * Conectar una aplicación Node.js a MongoDB
> * Implementar la aplicación en Azure
> * Actualizar el modelo de datos y volver a implementar la aplicación
> * Transmitir registros desde Azure a un terminal
> * Administrar la aplicación en Azure Portal

Pase al siguiente tutorial para aprender cómo asignar un nombre DNS personalizado a la aplicación.

> [!div class="nextstepaction"] 
> [Asignación de un nombre DNS personalizado existente a Azure App Service](app-service-web-tutorial-custom-domain.md)

O bien, eche un vistazo a otros recursos:

> [!div class="nextstepaction"]
> [Configuración de la aplicación Node.js](configure-language-nodejs.md)
