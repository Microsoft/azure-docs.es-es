---
title: Configuración de una aplicación de PHP
description: Aprenda a configurar una aplicación de PHP en las instancias nativas de Windows, o en un contenedor de PHP pregenerado, en Azure App Service. En este artículo se muestran las tareas de configuración más comunes.
ms.devlang: php
ms.topic: article
ms.date: 06/02/2020
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: afac8273b5729bcf5470be471145214426dc7dab
ms.sourcegitcommit: 814778c54b59169c5899199aeaa59158ab67cf44
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/13/2020
ms.locfileid: "90055306"
---
# <a name="configure-a-php-app-for-azure-app-service"></a>Configuración de una aplicación de PHP para Azure App Service

En esta guía, se explica cómo se configuran las aplicaciones web de PHP, los back-ends para dispositivos móviles y las aplicaciones de API de Azure App Service.

Esta guía incluye conceptos clave e instrucciones para los desarrolladores de PHP que implementan aplicaciones en App Service. Si nunca ha usado Azure App Service, siga primero la [guía de inicio rápido de PHP](quickstart-php.md) y el [tutorial de PHP con MySQL](tutorial-php-mysql-app.md).

## <a name="show-php-version"></a>Visualización de la versión de PHP

::: zone pivot="platform-windows"  

Para mostrar la versión actual de PHP, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query phpVersion
```

Para mostrar todas las versiones compatibles de PHP, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes | grep php
```

::: zone-end

::: zone pivot="platform-linux"

Para mostrar la versión actual de PHP, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
```

Para mostrar todas las versiones compatibles de PHP, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes --linux | grep PHP
```

::: zone-end

## <a name="set-php-version"></a>Establecer la versión de PHP

::: zone pivot="platform-windows"  

Ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com) para establecer la versión 7.4 de PHP:

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --php-version 7.4
```

::: zone-end

::: zone pivot="platform-linux"

Ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com) para establecer la versión 7.2 de PHP:

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --linux-fx-version "PHP|7.2"
```

::: zone-end

::: zone pivot="platform-windows"  

## <a name="run-composer"></a>Ejecución de Composer

Si quiere que App Service ejecute [Composer](https://getcomposer.org/) en el momento de la implementación, la manera más fácil es incluir Composer en el repositorio.

En una ventana de terminal local, cambie el directorio a la raíz del repositorio y siga las instrucciones que se indican en [Descarga de Composer](https://getcomposer.org/download/) para descargar *composer.phar* a la raíz del directorio.

Ejecute los siguientes comandos (necesita tener instalado [npm](https://www.npmjs.com/get-npm)):

```bash
npm install kuduscript -g
kuduscript --node --scriptType bash --suppressPrompt
```

La raíz del repositorio ahora tiene dos archivos adicionales: *.deployment* y *deploy.sh*.

Abra *deploy.sh* y busque la sección `Deployment`, que tiene el siguiente aspecto:

```bash
##################################################################################################################################
# Deployment
# ----------
```

Agregue la sección de código que necesita para ejecutar la herramienta necesaria *al final* de la sección `Deployment`:

```bash
# 4. Use composer
echo "$DEPLOYMENT_TARGET"
if [ -e "$DEPLOYMENT_TARGET/composer.json" ]; then
  echo "Found composer.json"
  pushd "$DEPLOYMENT_TARGET"
  php composer.phar install $COMPOSER_ARGS
  exitWithMessageOnError "Composer install failed"
  popd
fi
```

Confirme todos los cambios e implemente el código mediante Git o realice la implementación desde un archivo ZIP con la automatización de compilaciones habilitada. Composer ahora debería estar funcionando como parte de la automatización de la implementación.

## <a name="run-gruntbowergulp"></a>Ejecutar Grunt, Bower o Gulp

Si quiere que App Service ejecute herramientas de automatización populares en el momento de la implementación, como Grunt, Bower o Gulp, deberá suministrar un [script de implementación personalizado](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script). App Service ejecuta este script cuando se implementa con Git o con la [implementación desde un archivo ZIP](deploy-zip.md) con la automatización de compilaciones habilitada. 

Para habilitar que el repositorio ejecute estas herramientas, deberá agregarlas a las dependencias en el archivo *package.json.* Por ejemplo:

```json
"dependencies": {
  "bower": "^1.7.9",
  "grunt": "^1.0.1",
  "gulp": "^3.9.1",
  ...
}
```

Desde una ventana de terminal local, cambie el directorio a la raíz del repositorio y ejecute los siguientes comandos (debe tener instalado [npm](https://www.npmjs.com/get-npm)):

```bash
npm install kuduscript -g
kuduscript --node --scriptType bash --suppressPrompt
```

La raíz del repositorio ahora tiene dos archivos adicionales: *.deployment* y *deploy.sh*.

Abra *deploy.sh* y busque la sección `Deployment`, que tiene el siguiente aspecto:

```bash
##################################################################################################################################
# Deployment
# ----------
```

En esta sección termina con la ejecución de `npm install --production`. Agregue la sección de código que necesita para ejecutar la herramienta necesaria *al final* de la sección `Deployment`:

- [Bower](#bower)
- [Gulp](#gulp)
- [Grunt](#grunt)

Consulte un [ejemplo en la muestra de MEAN.js](https://github.com/Azure-Samples/meanjs/blob/master/deploy.sh#L112-L135), donde el script de implementación también ejecuta un comando `npm install` personalizado.

### <a name="bower"></a>Bower

Este fragmento de código ejecuta `bower install`.

```bash
if [ -e "$DEPLOYMENT_TARGET/bower.json" ]; then
  cd "$DEPLOYMENT_TARGET"
  eval ./node_modules/.bin/bower install
  exitWithMessageOnError "bower failed"
  cd - > /dev/null
fi
```

### <a name="gulp"></a>Gulp

Este fragmento de código ejecuta `gulp imagemin`.

```bash
if [ -e "$DEPLOYMENT_TARGET/gulpfile.js" ]; then
  cd "$DEPLOYMENT_TARGET"
  eval ./node_modules/.bin/gulp imagemin
  exitWithMessageOnError "gulp failed"
  cd - > /dev/null
fi
```

### <a name="grunt"></a>Grunt

Este fragmento de código ejecuta `grunt`.

```bash
if [ -e "$DEPLOYMENT_TARGET/Gruntfile.js" ]; then
  cd "$DEPLOYMENT_TARGET"
  eval ./node_modules/.bin/grunt
  exitWithMessageOnError "Grunt failed"
  cd - > /dev/null
fi
```

::: zone-end

::: zone pivot="platform-linux"

## <a name="customize-build-automation"></a>Personalización de la automatización de compilaciones

Si implementa la aplicación utilizando paquetes Git o zip con la automatización de compilaciones activada, la automatización de compilaciones de App Service se ejecutará en este orden:

1. Ejecute el script personalizado si lo especifica `PRE_BUILD_SCRIPT_PATH`.
1. Ejecute `php composer.phar install`.
1. Ejecute el script personalizado si lo especifica `POST_BUILD_SCRIPT_PATH`.

`PRE_BUILD_COMMAND` y `POST_BUILD_COMMAND` son variables de entorno que, de forma predeterminada, están vacías. Para ejecutar comandos anteriores a la compilación, defina `PRE_BUILD_COMMAND`. Para ejecutar comandos posteriores a la compilación, defina `POST_BUILD_COMMAND`.

En el ejemplo siguiente se especifican las dos variables para una serie de comandos, separados por comas.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PRE_BUILD_COMMAND="echo foo, scripts/prebuild.sh"
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings POST_BUILD_COMMAND="echo foo, scripts/postbuild.sh"
```

Para obtener más variables de entorno para personalizar la automatización de compilaciones, consulte la [configuración de Oryx](https://github.com/microsoft/Oryx/blob/master/doc/configuration.md).

Para más información sobre cómo se ejecuta App Service y se compilan aplicaciones de PHP en Linux, consulte la [documentación de Oryx sobre cómo se detectan y se compilan las aplicaciones de PHP](https://github.com/microsoft/Oryx/blob/master/doc/runtimes/php.md).

## <a name="customize-start-up"></a>Personalización del inicio

De forma predeterminada, el contenedor PHP integrado ejecuta el servidor Apache. Al iniciarse, ejecuta `apache2ctl -D FOREGROUND"`. Si lo desea, puede ejecutar otro comando al inicio si ejecuta lo siguiente en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "<custom-command>"
```

::: zone-end

## <a name="access-environment-variables"></a>Acceso a variables de entorno

En App Service, puede [establecer la configuración de la aplicación](configure-common.md#configure-app-settings) fuera del código de la aplicación. Luego podrá acceder a ellas mediante el patrón estándar de [getenv()](https://secure.php.net/manual/function.getenv.php). Por ejemplo, para acceder a una configuración de una aplicación denominada `DB_HOST`, use el código siguiente:

```php
getenv("DB_HOST")
```

## <a name="change-site-root"></a>Cambio de la raíz del sitio

::: zone pivot="platform-windows"  

La plataforma web que prefiera puede usar un subdirectorio como la raíz del sitio. Por ejemplo, [Laravel](https://laravel.com/) usa el subdirectorio *public/* como la raíz del sitio.

Para personalizar la raíz del sitio, establezca la ruta de acceso de la aplicación virtual para la aplicación mediante el comando [`az resource update`](/cli/azure/resource#az-resource-update). En el ejemplo siguiente se establece la raíz del sitio en el subdirectorio *public/* del repositorio. 

```azurecli-interactive
az resource update --name web --resource-group <group-name> --namespace Microsoft.Web --resource-type config --parent sites/<app-name> --set properties.virtualApplications[0].physicalPath="site\wwwroot\public" --api-version 2015-06-01
```

De manera predeterminada, Azure App Service apunta la ruta de acceso de la aplicación virtual raíz ( _/_ ) al directorio raíz de los archivos de la aplicación implementada (_sites\wwwroot_).

::: zone-end

::: zone pivot="platform-linux"

La plataforma web que prefiera puede usar un subdirectorio como la raíz del sitio. Por ejemplo, [Laravel](https://laravel.com/) usa el subdirectorio `public/` como la raíz del sitio.

La imagen de PHP predeterminada para App Service utiliza Apache y no le permite personalizar la raíz del sitio para la aplicación. Para solucionar esta limitación, agregue un archivo *.htaccess* a la raíz del repositorio con el siguiente contenido:

```
<IfModule mod_rewrite.c>
    RewriteEngine on
    RewriteCond %{REQUEST_URI} ^(.*)
    RewriteRule ^(.*)$ /public/$1 [NC,L,QSA]
</IfModule>
```

Si prefiere no usar la reescritura de *.htaccess*, puede implementar la aplicación Laravel con una [imagen personalizada de Docker](quickstart-custom-container.md) en su lugar.

::: zone-end

## <a name="detect-https-session"></a>Detección de sesión de HTTPS

En App Service, la [terminación de SSL](https://wikipedia.org/wiki/TLS_termination_proxy) se produce en los equilibradores de carga de red, por lo que todas las solicitudes HTTPS llegan a su aplicación en forma de solicitudes HTTP sin cifrar. Si su aplicación lógica necesita comprobar si las solicitudes de usuario están cifradas, inspeccione el encabezado `X-Forwarded-Proto`.

```php
if (isset($_SERVER['HTTP_X_FORWARDED_PROTO']) && $_SERVER['HTTP_X_FORWARDED_PROTO'] === 'https') {
// Do something when HTTPS is used
}
```

Los marcos web más usados le permiten acceder a la información de `X-Forwarded-*` en el patrón de aplicación estándar. En [CodeIgniter](https://codeigniter.com/), [is_https()](https://github.com/bcit-ci/CodeIgniter/blob/master/system/core/Common.php#L338-L365) comprueba el valor de `X_FORWARDED_PROTO` de forma predeterminada.

## <a name="customize-phpini-settings"></a>Personalización de la configuración de php.ini

Si necesita hacer cambios en su instalación de PHP, puede cambiar cualquiera de las [directivas php.ini](https://www.php.net/manual/ini.list.php) siguiendo estos pasos.

> [!NOTE]
> La mejor forma de ver la versión de PHP y la configuración de *php.ini* actual consiste en llamar a [phpinfo()](https://php.net/manual/function.phpinfo.php) en la aplicación.
>

### <a name="customize-non-php_ini_system-directives"></a><a name="Customize-non-PHP_INI_SYSTEM directives"></a>Personalización de las directivas que no son de PHP_INI_SYSTEM

::: zone pivot="platform-windows"  

Para personalizar las directivas PHP_INI_USER, PHP_INI_PERDIR y PHP_INI_ALL (consulte las [directivas de php.ini](https://www.php.net/manual/ini.list.php)), agregue un archivo `.user.ini` al directorio raíz de la aplicación.

Agregue la configuración al archivo `.user.ini` usando la misma sintaxis que usaría en un archivo `php.ini`. Por ejemplo, si quiere desactivar la configuración `display_errors` y establecer la configuración `upload_max_filesize` en 10 M, el archivo `.user.ini` contendría el texto siguiente:

```
 ; Example Settings
 display_errors=On
 upload_max_filesize=10M

 ; Write errors to d:\home\LogFiles\php_errors.log
 ; log_errors=On
```

Vuelva a implementar la aplicación con los cambios y reiníciela.

Como alternativa al uso del archivo `.user.ini`, puede usar [ini_set()](https://www.php.net/manual/function.ini-set.php) en la aplicación para personalizar estas directivas que no son de PHP_INI_SYSTEM.

::: zone-end

::: zone pivot="platform-linux"

Para personalizar las directivas PHP_INI_USER, PHP_INI_PERDIR y PHP_INI_ALL (consulte las [directivas de php.ini](https://www.php.net/manual/ini.list.php)), agregue un archivo *.htaccess* al directorio raíz de la aplicación.

En el archivo *.htaccess*, agregue las directivas mediante la sintaxis `php_value <directive-name> <value>`. Por ejemplo:

```
php_value upload_max_filesize 1000M
php_value post_max_size 2000M
php_value memory_limit 3000M
php_value max_execution_time 180
php_value max_input_time 180
php_value display_errors On
php_value upload_max_filesize 10M
```

Vuelva a implementar la aplicación con los cambios y reiníciela. Si la implementa con Kudu (por ejemplo, mediante [Git](deploy-local-git.md)), se reinicia automáticamente después de la implementación.

Como alternativa al uso de *.htaccess*, puede usar [ini_set()](https://www.php.net/manual/function.ini-set.php) en la aplicación para personalizar estas directivas que no son de PHP_INI_SYSTEM.

::: zone-end

### <a name="customize-php_ini_system-directives"></a><a name="customize-php_ini_system-directives"></a>Personalización de las directivas de PHP_INI_SYSTEM

::: zone pivot="platform-windows"  

Para personalizar las directivas de PHP_INI_SYSTEM (consulte [directivas de php.ini](https://www.php.net/manual/ini.list.php)), no puede usar el enfoque *.htaccess*. App Service proporciona un mecanismo diferente mediante la configuración de la aplicación `PHP_INI_SCAN_DIR`.

En primer lugar, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com) para agregar una configuración de aplicación llamada `PHP_INI_SCAN_DIR`:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PHP_INI_SCAN_DIR="d:\home\site\ini"
```

Vaya a la consola de Kudu (`https://<app-name>.scm.azurewebsites.net/DebugConsole`) y navegue a `d:\home\site`.

Cree un directorio en `d:\home\site` llamado `ini`, después cree un archivo *.ini* en el directorio `d:\home\site\ini` (por ejemplo, *settings.ini)* con las directivas que quiera personalizar. Utilice la misma sintaxis que usaría en un archivo *php.ini*. 

Por ejemplo, para cambiar el valor de [expose_php](https://php.net/manual/ini.core.php#ini.expose-php), ejecute los siguientes comandos:

```bash
cd /home/site
mkdir ini
echo "expose_php = Off" >> ini/setting.ini
```

Para que los cambios surtan efecto, reinicie la aplicación.

::: zone-end

::: zone pivot="platform-linux"

Para personalizar las directivas de PHP_INI_SYSTEM (consulte [directivas de php.ini](https://www.php.net/manual/ini.list.php)), no puede usar el enfoque *.htaccess*. App Service proporciona un mecanismo diferente mediante la configuración de la aplicación `PHP_INI_SCAN_DIR`.

En primer lugar, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com) para agregar una configuración de aplicación llamada `PHP_INI_SCAN_DIR`:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PHP_INI_SCAN_DIR="/usr/local/etc/php/conf.d:/home/site/ini"
```

`/usr/local/etc/php/conf.d` es el directorio predeterminado donde se encuentra *php.ini*. `/home/site/ini` es el directorio personalizado en el que agregará un archivo personalizado *.ini*. Separe los valores con un `:`.

Vaya a la sesión SSH web con el contenedor Linux (`https://<app-name>.scm.azurewebsites.net/webssh/host`).

Cree un directorio en `/home/site` llamado `ini`, después cree un archivo *.ini* en el directorio `/home/site/ini` (por ejemplo, *settings.ini)* con las directivas que quiera personalizar. Utilice la misma sintaxis que usaría en un archivo *php.ini*. 

> [!TIP]
> En los contenedores Linux incorporados en App Service, */home* se utiliza como almacenamiento compartido persistente. 
>

Por ejemplo, para cambiar el valor de [expose_php](https://php.net/manual/ini.core.php#ini.expose-php), ejecute los siguientes comandos:

```bash
cd /home/site
mkdir ini
echo "expose_php = Off" >> ini/setting.ini
```

Para que los cambios surtan efecto, reinicie la aplicación.

::: zone-end

## <a name="enable-php-extensions"></a>Habilitación de las extensiones de PHP

::: zone pivot="platform-windows"  

Las instalaciones de PHP integradas contienen las extensiones más usadas. Puede habilitar extensiones adicionales en la misma forma que [personaliza las directivas de php.ini](#customize-php_ini_system-directives).

> [!NOTE]
> La mejor forma de ver la versión de PHP y la configuración de *php.ini* actual consiste en llamar a [phpinfo()](https://php.net/manual/function.phpinfo.php) en la aplicación.
>

Para habilitar extensiones adicionales, siga los pasos que se detallan a continuación:

Agregue un directorio `bin` al directorio raíz de la aplicación y coloque los archivos de extensión `.dll` en él (por ejemplo, *mongodb.dll*). Asegúrese de que las extensiones sean compatibles con la versión de PHP en Azure y que sean compatibles con VC9 y no seguras para subprocesos (nts).

Implemente los cambios

Siga los pasos que se indican en [Personalización de las directivas de PHP_INI_SYSTEM](#customize-php_ini_system-directives), agregue las extensiones al archivo *custom.ini* con las directivas [extension](https://www.php.net/manual/ini.core.php#ini.extension) o [zend_extension](https://www.php.net/manual/ini.core.php#ini.zend-extension).

```
extension=d:\home\site\wwwroot\bin\mongodb.dll
zend_extension=d:\home\site\wwwroot\bin\xdebug.dll
```

Para que los cambios surtan efecto, reinicie la aplicación.

::: zone-end

::: zone pivot="platform-linux"

Las instalaciones de PHP integradas contienen las extensiones más usadas. Puede habilitar extensiones adicionales en la misma forma que [personaliza las directivas de php.ini](#customize-php_ini_system-directives).

> [!NOTE]
> La mejor forma de ver la versión de PHP y la configuración de *php.ini* actual consiste en llamar a [phpinfo()](https://php.net/manual/function.phpinfo.php) en la aplicación.
>

Para habilitar extensiones adicionales, siga los pasos que se detallan a continuación:

Agregue un directorio `bin` al directorio raíz de la aplicación y coloque los archivos de extensión `.so` en él (por ejemplo, *mongodb.so*). Asegúrese de que las extensiones sean compatibles con la versión de PHP en Azure y que sean compatibles con VC9 y no seguras para subprocesos (nts).

Implemente los cambios

Siga los pasos que se indican en [Personalización de las directivas de PHP_INI_SYSTEM](#customize-php_ini_system-directives), agregue las extensiones al archivo *custom.ini* con las directivas [extension](https://www.php.net/manual/ini.core.php#ini.extension) o [zend_extension](https://www.php.net/manual/ini.core.php#ini.zend-extension).

```ini
extension=/home/site/wwwroot/bin/mongodb.so
zend_extension=/home/site/wwwroot/bin/xdebug.so
```

Para que los cambios surtan efecto, reinicie la aplicación.

::: zone-end

## <a name="access-diagnostic-logs"></a>Acceso a los registros de diagnóstico

::: zone pivot="platform-windows"  

Use la utilidad [error_log()](https://php.net/manual/function.error-log.php) estándar para que los registros de diagnóstico se muestren en Azure App Service.

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

::: zone-end

::: zone pivot="platform-linux"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

::: zone-end

## <a name="troubleshooting"></a>Solución de problemas

Cuando una aplicación de PHP en funcionamiento se comporta de manera diferente en App Service o genera errores, intente lo siguiente:

- [Acceso a la secuencia de registros](#access-diagnostic-logs).
- Pruebe la aplicación localmente en modo de producción. App Service ejecuta la aplicación en el modo de producción, por lo que deberá asegurarse de que el proyecto funciona según lo previsto en modo de producción localmente. Por ejemplo:
    - En función de su archivo *composer.json*, pueden instalarse distintos paquetes para el modo de producción (`require` frente a `require-dev`).
    - Algunas plataformas web pueden implementar archivos estáticos de forma diferente en modo de producción.
    - Algunas plataformas web pueden usar scripts de inicio personalizados cuando se ejecutan en modo de producción.
- Ejecute la aplicación en App Service en el modo de depuración. Por ejemplo, en [Laravel](https://meanjs.org/), puede configurar la aplicación para que genere mensajes de depuración en producción al [configurar el ajuste de la aplicación `APP_DEBUG` en `true`](configure-common.md#configure-app-settings).

::: zone pivot="platform-linux"

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de PHP con MySQL](tutorial-php-mysql-app.md)

::: zone pivot="platform-linux"

> [!div class="nextstepaction"]
> [P+F sobre App Service en Linux](faq-app-service-linux.md)

::: zone-end

