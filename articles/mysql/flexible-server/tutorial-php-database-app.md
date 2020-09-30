---
title: 'Tutorial: Creación de una aplicación PHP (Laravel) con el servidor flexible de Azure Database for MySQL'
description: En este tutorial se explica cómo crear una aplicación PHP con un servidor flexible.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: tutorial
ms.devlang: php
ms.date: 9/21/2020
ms.custom: mvc
ms.openlocfilehash: 1bad9a7da6f0604f910ce1095b734043be8cf3c3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90929765"
---
# <a name="tutorial-build-a-php-laravel-and-mysql-flexible-server-preview-app-in-azure-app-service"></a>Tutorial: Creación de una aplicación de servidor flexible (versión preliminar) de MySQL y PHP (Laravel) en Azure App Service


:::image type="content" source="media/tutorial-php-database-app/complete-checkbox-published.png" alt-text="Aplicación web PHP en Azure con un servidor flexible":::

[Azure App Service](https://docs.microsoft.com/azure/app-service/overview) proporciona un servicio de hospedaje web muy escalable y con aplicación automática de revisiones con el sistema operativo Linux. En este tutorial, se muestra cómo crear una aplicación PHP en Azure y conectarla a una base de datos MySQL. Cuando haya terminado, tendrá una aplicación de [Laravel](https://laravel.com/) que se ejecuta en Azure App Service en Linux.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Configurar una aplicación PHP (Laravel) con MySQL local
> * Crear un servidor flexible (versión preliminar) de MySQL
> * Conectar una aplicación PHP a un servidor flexible (versión preliminar) de MySQL
> * Implementar la aplicación en Azure App Service
> * Actualizar el modelo de datos y volver a implementar la aplicación
> * Administrar la aplicación en Azure Portal

Si no tiene una [suscripción a Azure](https://docs.microsoft.com/azure/guides/developer/azure-developer-guide#understanding-accounts-subscriptions-and-billing), cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

1. [Instalación de Git](https://git-scm.com/)
2. [Instalación de PHP 5.6.4, o cualquier versión posterior](https://php.net/downloads.php)
3. [Instalación de Composer](https://getcomposer.org/doc/00-intro.md)
4. Habilite las siguientes extensiones de PHP que Laravel necesita: OpenSSL, PDO-MySQL, Mbstring, Tokenizer, XML
5. [Instalación e inicio de MySQL](https://dev.mysql.com/doc/refman/5.7/en/installing.html)

## <a name="prepare-local-mysql"></a>Preparación de MySQL local

En este paso, creará una base de datos en el servidor MySQL local para usarlo en este tutorial.

### <a name="connect-to-local-mysql-server"></a>Conexión a un servidor MySQL local

En una ventana de terminal, conéctese al servidor MySQL local. Esta ventana de terminal se puede usar para ejecutar todos los comandos de este tutorial.

```bash
mysql -u root -p
```

Si se le pide una contraseña, escriba la de la cuenta `root`. Si no recuerda la contraseña de la cuenta raíz, consulte [MySQL: How to Reset the Root Password](https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html) (MySQL: Instrucciones de restablecimiento de la contraseña de la cuenta raíz).

Si el comando se ejecuta correctamente, significa que el servidor MySQL está en ejecución. De lo contrario, siga los [pasos posteriores a la instalación de MySQL](https://dev.mysql.com/doc/refman/5.7/en/postinstallation.html) para asegurarse de iniciar el servidor local de MySQL.

### <a name="create-a-database-locally"></a>Creación de una base de datos local

En el símbolo del sistema de `mysql`, cree una base de datos.

```sql
CREATE DATABASE sampledb;
```

Escriba `quit` para salir de la conexión del servidor.

```sql
quit
```

<a name="step2"></a>

## <a name="create-a-php-app-locally"></a>Creación de una aplicación PHP local
En este paso, obtendrá una aplicación Laravel de ejemplo, configurará la conexión a base de datos correspondiente y la ejecutará de forma local.

### <a name="clone-the-sample"></a>Clonación del ejemplo

En la ventana de terminal, vaya a un directorio vacío en el que pueda clonar la aplicación de ejemplo.  Ejecute el comando siguiente para clonar el repositorio de ejemplo.

```bash
git clone https://github.com/Azure-Samples/laravel-tasks
```

`cd` en el directorio clonado.
Instale los paquetes requeridos.

```bash
cd laravel-tasks
composer install
```

### <a name="configure-mysql-connection"></a>Configuración de la conexión a MySQL

En la raíz del repositorio, cree un archivo llamado *.env*. Copie las siguientes variables en el archivo *.env*. Reemplace el marcador de posición _&lt;root_password>_ por la contraseña del usuario raíz de MySQL.

```txt
APP_ENV=local
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_DATABASE=sampledb
DB_USERNAME=root
DB_PASSWORD=<root_password>
```

Para obtener información sobre la forma en que Laravel usa el archivo _.env_, consulte [Laravel Environment Configuration](https://laravel.com/docs/5.4/configuration#environment-configuration) (Configuración del entorno de Laravel).

### <a name="run-the-sample-locally"></a>Ejecución local del código

Ejecute las [migraciones de la base de datos Laravel](https://laravel.com/docs/5.4/migrations) para crear las tablas que necesita la aplicación. Para ver qué tablas se crean en las migraciones, mire en el directorio _database/migrations_ del repositorio de Git.

```bash
php artisan migrate
```

Genere una nueva clave de aplicación Laravel.

```bash
php artisan key:generate
```

Ejecute la aplicación.

```bash
php artisan serve
```

Vaya a `http://localhost:8000` en un explorador. Agregue algunas tareas a la página.

:::image type="content" source="media/tutorial-php-database-app/mysql-connect-success.png" alt-text="Conexión correcta de PHP a MySQL":::

Para detener PHP, escriba `Ctrl + C` en el terminal.

## <a name="create-a-mysql-flexible-server-preview"></a>Crear un servidor flexible (versión preliminar) de MySQL
En este paso, creará una base de datos MySQL en el [servidor flexible (versión preliminar) de Azure Database for MySQL](/azure/mysql), que se encuentra en versión preliminar pública. Posteriormente, configurará la aplicación PHP para que se conecte a esta base de datos. En [Azure Cloud Shell](https://docs.microsoft.com/azure/cloud-shell/overview), cree un servidor mediante el comando [`az flexible-server create`](/cli/azure/mysql/server#az-mysql-flexible-server-create).

```azurecli-interactive
az mysql flexible-server create  --resource-group myResourceGroup --public-access <IP-Address>
```

> [!IMPORTANT]
> - Tome nota del **nombre del servidor** y la **cadena de conexión** para usarlos en el paso siguiente para conectarse y ejecutar la migración de datos de Laravel.
> - En el argumento **IP-Address**, proporcione la dirección IP del equipo cliente. El servidor se bloquea cuando se crea y es necesario que permita el acceso a su equipo cliente para administrar el servidor de forma local.

### <a name="configure-server-firewall-to-allow-web-app-to-connect-to-the-server"></a>Configuración del firewall del servidor para permitir que la aplicación web se conecte al servidor

En Cloud Shell, cree una regla de firewall para que el servidor MySQL permita conexiones de clientes con el comando az mysql server firewall-rule create. Cuando la dirección IP inicial y la dirección IP final se establecen en ```0.0.0.0```, el firewall solo se habilita para otros servicios de Azure que no tienen una dirección IP estática para conectarse al servidor.

```azurecli
az mysql flexible-server firewall-rule create --name allanyAzureIPs --server <mysql-server-name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

### <a name="connect-to-production-mysql-server-locally"></a>Conexión al servidor MySQL de producción local

En la ventana del terminal local, conéctese al servidor MySQL de Azure. Use el valor que especificó anteriormente para ```<admin-user>``` y ```<mysql-server-name>```. Cuando se le solicite una contraseña, utilice la contraseña que especificó al crear la base de datos en Azure.

```bash
mysql -u <admin-user> -h <mysql-server-name>.mysql.database.azure.com -P 3306 -p
```

### <a name="create-a-production-database"></a>Creación de una base de datos de producción

En el símbolo del sistema de `mysql`, cree una base de datos.

```sql
CREATE DATABASE sampledb;
```

### <a name="create-a-user-with-permissions"></a>Creación de un usuario con permisos

Cree un usuario de base de datos denominado _phpappuser_ y concédale todos los privilegios de la base de datos `sampledb`. Para simplificar el tutorial, use _MySQLAzure2020_ como contraseña.

```sql
CREATE USER 'phpappuser' IDENTIFIED BY 'MySQLAzure2020';
GRANT ALL PRIVILEGES ON sampledb.* TO 'phpappuser';
```

Escriba `quit` para salir de la conexión del servidor.

```sql
quit
```

## <a name="connect-app-to-mysql-flexible-server"></a>Conexión de una aplicación PHP a un servidor flexible de MySQL

En este paso, conectará la aplicación PHP a la base de datos MySQL que creó en Azure Database for MySQL.

<a name="devconfig"></a>

### <a name="configure-the-database-connection"></a>Configuración de la conexión de base de datos

En la raíz del repositorio, cree un archivo _.env.production_ y copie en él las siguientes variables. Reemplace el marcador de posición _&lt;mysql-server-name>_ en *DB_HOST* y *DB_USERNAME*.

```
APP_ENV=production
APP_DEBUG=true
APP_KEY=

DB_CONNECTION=mysql
DB_HOST=<mysql-server-name>.mysql.database.azure.com
DB_DATABASE=sampledb
DB_USERNAME=phpappuser
DB_PASSWORD=MySQLAzure2017
MYSQL_SSL=true
```

Guarde los cambios.

> [!TIP]
> Para proteger la información de conexión de MySQL, este archivo ya se ha excluido del repositorio de Git (vea _.gitignore_ en la raíz del repositorio). Más adelante, aprenderá a configurar variables de entorno en App Service para conectarse a la base de datos en Azure Database for MySQL. Con las variables de entorno, no es preciso el archivo *.env* en App Service.
>

### <a name="configure-tlsssl-certificate"></a>Configuración del certificado TLS/SSL

De manera predeterminada, el servidor flexible de MySQL aplica las conexiones TLS de los clientes. Para conectarse a la base de datos MySQL en Azure, debe usar el certificado [ _.pem_ proporcionado por el servidor flexible de Azure Database for MySQL](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem). Descargue [este certificado](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem)) y colóquelo en la carpeta **ssl** de la copia local del repositorio de la aplicación de ejemplo.

Abra _config/database.php_ y agregue los parámetros `sslmode` y `options` a `connections.mysql`, como se muestra en el código siguiente.

```php
'mysql' => [
    ...
    'sslmode' => env('DB_SSLMODE', 'prefer'),
    'options' => (env('MYSQL_SSL') && extension_loaded('pdo_mysql')) ? [
        PDO::MYSQL_ATTR_SSL_KEY    => '/ssl/DigiCertGlobalRootCA.crt.pem',
    ] : []
],
```

### <a name="test-the-application-locally"></a>Prueba de la aplicación de forma local

Ejecute migraciones de base de datos Laravel con _.env.production_ como archivo de entorno para crear las tablas de la base de datos MySQL en Azure Database for MySQL. Recuerde que _. env.production_ tiene la información de conexión a su base de datos MySQL de Azure.

```bash
php artisan migrate --env=production --force
```

El archivo _.env.production_ aún no cuenta con una clave de aplicación válida. Genere una nueva para él en el terminal.

```bash
php artisan key:generate --env=production --force
```

Ejecute la aplicación de ejemplo con _.env.production_ como archivo de entorno.

```bash
php artisan serve --env=production
```

Vaya a `http://localhost:8000`. Si la página se carga sin errores, significa que la aplicación PHP se está conectado a la base de datos MySQL de Azure.

Agregue algunas tareas a la página.

:::image type="content" source="media/tutorial-php-database-app/mysql-connect-success.png" alt-text="Conexión correcta de PHP a Azure Database for MySQL":::

Para detener PHP, escriba `Ctrl + C` en el terminal.

### <a name="commit-your-changes"></a>Confirmación de los cambios

Ejecute los siguientes comandos de Git para confirmar los cambios:

```bash
git add .
git commit -m "database.php updates"
```

La aplicación está lista para implementarse.

## <a name="deploy-to-azure"></a>Implementar en Azure

En este paso se implementará la aplicación PHP conectada a MySQL en Azure App Service.

### <a name="configure-a-deployment-user"></a>Configuración de un usuario de implementación

Se puede implementar FTP y Git local en una aplicación web de Azure mediante un usuario de implementación. Una vez configurado este usuario de implementación, podrá usarlo en todas las implementaciones de Azure. El nombre de usuario y la contraseña en el nivel de cuenta son diferentes de las credenciales de suscripción de Azure.

Para configurar el usuario de implementación, ejecute el comando [az webapp deployment user set](https://docs.microsoft.com/cli/azure/webapp/deployment/user#az-webapp-deployment-user-set) en Azure Cloud Shell. Reemplace _&lt;username>_ y _&lt;password>_ por su nombre de usuario y contraseña de implementación.

El nombre de usuario debe ser único dentro de Azure y no debe contener el símbolo "\@" para las inserciones de Git local.
La contraseña debe tener al menos ocho caracteres y dos de los tres elementos siguientes: letras, números y símbolos.

```bash
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku F1 --is-linux
```

La salida JSON muestra la contraseña como NULL. Si recibe un error de "conflicto". Detalles: error 409, cambie el nombre de usuario. Si recibe un error de "solicitud incorrecta". Detalles: error 400, use una contraseña más segura. **Anote el nombre de usuario y la contraseña que se usarán para implementar las aplicaciones web.**

### <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

En Cloud Shell, cree un plan de App Service en el grupo de recursos con el comando [az appservice plan create](https://docs.microsoft.com/cli/azure/appservice/plan#az-appservice-plan-create). En el ejemplo siguiente se crea un plan de App Service denominado myAppServicePlan en el plan de tarifa Gratis (--sku F1) en un contenedor Linux (--is-linux).

az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku F1 --is-linux

<a name="create"></a>

### <a name="create-a-web-app"></a>Creación de una aplicación web

Cree una [aplicación web](https://docs.microsoft.com/azure/app-service/overview#app-service-on-linux) en el plan de App Service myAppServicePlan.

En Cloud Shell, puede usar el comando [az webapp create](https://docs.microsoft.com/cli/azure/webapp#az-webapp-create). En el siguiente ejemplo, reemplace _&lt;app-name>_ por un nombre único global de aplicación (los caracteres válidos son `a-z`, `0-9` y `-`). El tiempo de ejecución se establece en `PHP|7.0`. Para ver todos los runtime admitidos, ejecute [az webapp list-runtimes --linux](https://docs.microsoft.com/cli/azure/webapp#az-webapp-list-runtimes).

```bash
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --runtime "PHP|7.3" --deployment-local-git
```

Cuando se haya creado la aplicación web, la CLI de Azure mostrará información similar a la del ejemplo siguiente:

```
Local git is configured with url of 'https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git'
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app-name>.azurewebsites.net",
  "deploymentLocalGitUrl": "https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

Ha creado una nueva aplicación web vacía, con la implementación de Git habilitada.

> [!NOTE]
> La dirección URL de Git remoto se muestra en la propiedad deploymentLocalGitUrl, con el formato https://<username>@<app-name>.scm.azurewebsites.net/<app-name>.git. Guarde esta dirección URL, ya que la necesitará más adelante.

### <a name="configure-database-settings"></a>Configuración de la base de datos

En App Service, las variables de entorno se establecen como _valores de aplicación_ mediante el comando [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set).

El siguiente comando permite configurar los valores de aplicación `DB_HOST`, `DB_DATABASE`, `DB_USERNAME` y `DB_PASSWORD`. Reemplace los marcadores de posición _&lt;app-name>_ y _&lt;mysql-server-name>_ .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings DB_HOST="<mysql-server-name>.mysql.database.azure.com" DB_DATABASE="sampledb" DB_USERNAME="phpappuser" DB_PASSWORD="MySQLAzure2017" MYSQL_SSL="true"
```

Para acceder a la configuración puede usar el método [getenv](https://www.php.net/manual/en/function.getenv.php) de PHP. El código de Laravel usa un contenedor [env ](https://laravel.com/docs/5.4/helpers#method-env) sobre el elemento `getenv` de PHP. Por ejemplo, la configuración de MySQL de _config/database.php_ es como el código siguiente:

```php
'mysql' => [
    'driver'    => 'mysql',
    'host'      => env('DB_HOST', 'localhost'),
    'database'  => env('DB_DATABASE', 'forge'),
    'username'  => env('DB_USERNAME', 'forge'),
    'password'  => env('DB_PASSWORD', ''),
    ...
],
```

### <a name="configure-laravel-environment-variables"></a>Configuración de las variables de entorno de Laravel

Laravel necesita una clave de aplicación en App Service. Puede configurarlo con valores de aplicación.

En la ventana del terminal local, use `php artisan` para generar una clave de aplicación sin guardarla en _.env_.

```bash
php artisan key:generate --show
```

En Cloud Shell, establezca la clave de aplicación en la aplicación App Service, para lo que hay que usar el comando [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set). Reemplace los marcadores de posición _&lt;app-name>_ y _&lt;outputofphpartisankey:generate>_ .

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group myResourceGroup --settings APP_KEY="<output_of_php_artisan_key:generate>" APP_DEBUG="true"
```

`APP_DEBUG="true"` indica a Laravel que devuelva información de depuración si la aplicación implementada encuentra errores. Al ejecutar una aplicación de producción, establézcala en `false`, que es más seguro.

### <a name="set-the-virtual-application-path"></a>Establecimiento de la ruta de acceso de la aplicación virtual

El [ciclo de vida de la aplicación de Laravel](https://laravel.com/docs/5.4/lifecycle) comienza en el directorio _public_ en lugar de en el directorio raíz de la aplicación. La imagen de Docker PHP predeterminada para App Service utiliza Apache y no le permite personalizar `DocumentRoot` para Laravel. Sin embargo, puede usar `.htaccess` para volver a escribir todas las solicitudes para que apunten a _/public_ en lugar de al directorio raíz. En la raíz del repositorio, ya se ha agregado `.htaccess` con este fin. Con él, la aplicación Laravel está preparada para implementarse.

Para más información, consulte [Change site root](https://docs.microsoft.com/azure/app-service/configure-language-php?pivots=platform-linux#change-site-root) (Cambio de la raíz del sitio).

### <a name="push-to-azure-from-git"></a>Inserción en Azure desde Git

En la ventana del terminal local, agregue una instancia remota de Azure al repositorio de Git local. Reemplace _&lt;deploymentLocalGitUrl-from-create-step>_ por la dirección URL del repositorio de Git remoto que guardó en [Creación de una aplicación web](#create-a-web-app).

```bash
git remote add azure <deploymentLocalGitUrl-from-create-step>
```

Realice la insercion en la instancia remota de Azure para implementar la aplicación con el comando siguiente. Cuando el Administrador de credenciales de Git le solicite las credenciales, asegúrese de que especifica las que creó en **Configuración de un usuario de implementación**, no las que se usan para iniciar sesión en Azure Portal.

```bash
git push azure master
```

Este comando puede tardar varios minutos en ejecutarse. Durante la ejecución, muestra información similar a la del ejemplo siguiente:

<pre>
Counting objects: 3, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 291 bytes | 0 bytes/s, done.
Total 3 (delta 2), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id 'a5e076db9c'.
remote: Running custom deployment command...
remote: Running deployment command...
...
&lt; Output has been truncated for readability &gt;
</pre>

### <a name="browse-to-the-azure-app"></a>Navegación hasta la aplicación de Azure

Vaya a `http://<app-name>.azurewebsites.net` y agregue algunas tareas a la lista.

:::image type="content" source="media/tutorial-php-database-app/php-mysql-in-azure.png" alt-text="Aplicación web PHP en Azure":::

Ya está ejecutando una aplicación PHP orientada a datos en Azure App Service.

## <a name="update-model-locally-and-redeploy"></a>Actualización local del modelo y nueva implementación

En este paso, se realiza un cambio sencillo en el modelo de datos `task` y en la aplicación web y, después, se publica la actualización en Azure.

Para el escenario de las tareas, modifique la aplicación de forma que pueda marcar una tarea como completada.

### <a name="add-a-column"></a>Adición de una columna

En la ventana del terminal local, vaya a la raíz del repositorio de Git.

Generar una migración de base de datos nueva para la tabla `tasks`:

```bash
php artisan make:migration add_complete_column --table=tasks
```

Este comando muestra el nombre del archivo de migración generado. Busque este archivo en _database/migrations_ y ábralo.

Reemplace el método `up` por el código siguiente:

```php
public function up()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->boolean('complete')->default(False);
    });
}
```

El código anterior agrega una columna booleana a la tabla `tasks` denominada `complete`.

Reemplace el método `down` por el siguiente código para la acción de reversión:

```php
public function down()
{
    Schema::table('tasks', function (Blueprint $table) {
        $table->dropColumn('complete');
    });
}
```

En la ventana del terminal local, ejecute las migraciones de base de datos Laravel para realizar el cambio en la base de datos local.

```bash
php artisan migrate
```

En función de la [convención de nomenclatura de Laravel](https://laravel.com/docs/5.4/eloquent#defining-models), el modelo `Task` (consulte _app/Task.php_) se asigna a la tabla `tasks` de manera predeterminada.

### <a name="update-application-logic"></a>Actualización de la lógica de aplicación

Abra el archivo *routes/web.php*. La aplicación define aquí tanto sus rutas como su lógica de negocios.

Al final del archivo, agregue una ruta con el siguiente código:

```php
/**
 * Toggle Task completeness
 */
Route::post('/task/{id}', function ($id) {
    error_log('INFO: post /task/'.$id);
    $task = Task::findOrFail($id);

    $task->complete = !$task->complete;
    $task->save();

    return redirect('/');
});
```

El código anterior lleva a cabo una sencilla actualización en el modelo de datos, para lo que alterna el valor de `complete`.

### <a name="update-the-view"></a>Actualización de la vista

Abra el archivo *resources/views/tasks.blade.php*. Busque la etiqueta de apertura `<tr>` y reemplácela por:

```html
<tr class="{{ $task->complete ? 'success' : 'active' }}" >
```

El código anterior cambia el color de la fila en función de si la tarea se ha completado.

En la siguiente línea, tiene el siguiente código:

```html
<td class="table-text"><div>{{ $task->name }}</div></td>
```

Reemplace esta línea al completo por el siguiente código:

```html
<td>
    <form action="{{ url('task/'.$task->id) }}" method="POST">
        {{ csrf_field() }}

        <button type="submit" class="btn btn-xs">
            <i class="fa {{$task->complete ? 'fa-check-square-o' : 'fa-square-o'}}"></i>
        </button>
        {{ $task->name }}
    </form>
</td>
```

El código anterior agrega el botón de envío que hace referencia a la ruta que definió previamente.

### <a name="test-the-changes-locally"></a>Prueba local de los cambios

En la ventana del terminal local, ejecute el servidor de desarrollo desde el directorio raíz del repositorio de Git.

```bash
php artisan serve
```

Para ver cómo cambia el estado de la tarea, navegue hasta `http://localhost:8000` y active la casilla.

:::image type="content" source="media/tutorial-php-database-app/complete-checkbox.png" alt-text="Casilla agregada a tarea":::

Para detener PHP, escriba `Ctrl + C` en el terminal.

### <a name="publish-changes-to-azure"></a>Publicación de los cambios en Azure

En la ventana del terminal local, ejecute las migraciones de base de datos Laravel con la cadena de conexión de producción para realizar el cambio en la base de datos de Azure.

```bash
php artisan migrate --env=production --force
```

Confirme todos los cambios en Git y, después, inserte los cambios en el código en Azure.

```bash
git add .
git commit -m "added complete checkbox"
git push azure master
```

Una vez que `git push` esté completo, vaya a la aplicación de Azure y pruebe la nueva funcionalidad.

:::image type="content" source="media/tutorial-php-database-app/complete-checkbox-published.png" alt-text="Modelo y cambios en la base de datos publicados en Azure":::

Si ha agregado tareas, estas se conservarán en la base de datos. Las actualizaciones que efectúe en el esquema de datos dejan los datos existentes intactos.

## <a name="clean-up-resources"></a>Limpieza de recursos
En los pasos anteriores, creó recursos de Azure en un grupo de recursos. Si prevé que no necesitará estos recursos en el futuro, elimine el grupo de recursos ejecutando el siguiente comando en Cloud Shell:

```bash
az group delete --name myResourceGroup
```

<a name="next"></a>

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administración de recursos en Azure Portal](https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resources-portal) <br/>
> [!div class="nextstepaction"]
> [Administración del servidor](how-to-manage-server-cli.md)
