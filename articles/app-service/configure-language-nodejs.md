---
title: Configuración de aplicaciones de Node.js de Windows
description: Aprenda a configurar una aplicación de Node.js en las instancias nativas de Windows de App Service. En este artículo se muestran las tareas de configuración más comunes.
ms.custom: devx-track-javascript
ms.devlang: nodejs
ms.topic: article
ms.date: 06/02/2020
ms.openlocfilehash: 0fc6ed5cb090653e381d82f484d355a514520c62
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87170912"
---
# <a name="configure-a-windows-nodejs-app-for-azure-app-service"></a>Configuración de una aplicación de Node.js para Azure App Service

Las aplicaciones de Node.js deben implementarse con todas las dependencias de NPM. El motor de implementación de App Service ejecuta `npm install --production` automáticamente al implementar un [repositorio de Git](deploy-local-git.md) o un [paquete Zip](deploy-zip.md) con la automatización de compilaciones habilitada. Sin embargo, si implementa los archivos con [FTP/S](deploy-ftp.md), deberá cargar manualmente los paquetes necesarios. Para obtener información sobre las aplicaciones de Linux, consulte [Configuración de una aplicación de PHP en Linux para Azure App Service](containers/configure-language-nodejs.md).

Esta guía incluye conceptos clave e instrucciones para los desarrolladores de Node.js que realizan implementaciones en App Service. Si nunca ha usado Azure App Service, siga primero la [guía de inicio rápido de Node.js](app-service-web-get-started-nodejs.md) y el [tutorial de Node.js con MongoDB](app-service-web-tutorial-nodejs-mongodb-app.md).

## <a name="show-nodejs-version"></a>Mostrar la versión de Node.js

Para mostrar la versión actual de Node.js, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --query "[?name=='WEBSITE_NODE_DEFAULT_VERSION'].value"
```

Para mostrar todas las versiones compatibles de Node.js, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes | grep node
```

## <a name="set-nodejs-version"></a>Establecer la versión de Node.js

Para establecer la aplicación en una [versión compatible de Node.js](#show-nodejs-version), ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com) para configurar `WEBSITE_NODE_DEFAULT_VERSION` en una versión compatible:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings WEBSITE_NODE_DEFAULT_VERSION="10.15"
```

Esta configuración especifica la versión de Node.js que usa, tanto en tiempo de ejecución como durante la restauración automatizada de paquetes durante la automatización de compilaciones de App Service.

> [!NOTE]
> Debe establecer la versión de Node.js en el archivo `package.json` del proyecto. El motor de implementación se ejecuta en un proceso independiente que contiene todas las versiones compatibles de Node.js.

## <a name="access-environment-variables"></a>Acceso a variables de entorno

En App Service, puede [establecer la configuración de la aplicación](configure-common.md) fuera del código de la aplicación. Luego puede acceder a ellas mediante el patrón estándar de Node.js. Por ejemplo, para acceder a una configuración de una aplicación denominada `NODE_ENV`, use el código siguiente:

```javascript
process.env.NODE_ENV
```

## <a name="run-gruntbowergulp"></a>Ejecutar Grunt, Bower o Gulp

De manera predeterminada, la automatización de compilaciones de App Service ejecuta `npm install --production` cuando reconoce que una aplicación de Node.js se implementa a través de Git (o implementación zip con la automatización de compilaciones habilitada). Si la aplicación requiere alguna de las herramientas de automatización más populares, como Grunt, Bower o Gulp, deberá suministrar un [script de implementación personalizado](https://github.com/projectkudu/kudu/wiki/Custom-Deployment-Script) para ejecutarla.

Para habilitar que el repositorio ejecute estas herramientas, deberá agregarlas a las dependencias en el archivo *package.json.* Por ejemplo:

```json
"dependencies": {
  "bower": "^1.7.9",
  "grunt": "^1.0.1",
  "gulp": "^3.9.1",
  ...
}
```

Desde una ventana de terminal local, cambie el directorio a la raíz del repositorio y ejecute los siguientes comandos:

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

## <a name="detect-https-session"></a>Detección de sesión de HTTPS

En App Service, la [terminación de SSL](https://wikipedia.org/wiki/TLS_termination_proxy) se produce en los equilibradores de carga de red, por lo que todas las solicitudes HTTPS llegan a su aplicación en forma de solicitudes HTTP sin cifrar. Si su aplicación lógica necesita comprobar si las solicitudes de usuario están cifradas, inspeccione el encabezado `X-Forwarded-Proto`.

Los marcos web más usados le permiten acceder a la información de `X-Forwarded-*` en el patrón de aplicación estándar. En [Express](https://expressjs.com/), puede usar [trust proxy](https://expressjs.com/guide/behind-proxies.html). Por ejemplo:

```javascript
app.set('trust proxy', 1)
...
if (req.secure) {
  // Do something when HTTPS is used
}
```

## <a name="access-diagnostic-logs"></a>Acceso a los registros de diagnóstico

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

## <a name="troubleshooting"></a>Solución de problemas

Cuando una aplicación de Node.js en funcionamiento se comporta de manera diferente en App Service o genera errores, intente lo siguiente:

- [Acceso a la secuencia de registros](#access-diagnostic-logs).
- Pruebe la aplicación localmente en modo de producción. App Service ejecuta las aplicaciones de Node.js en el modo de producción, por lo que deberá asegurarse de que el proyecto funciona según lo previsto en modo de producción localmente. Por ejemplo:
    - En función de su archivo *package.json*, pueden instalarse distintos paquetes para el modo de producción (`dependencies` frente a `devDependencies`).
    - Algunas plataformas web pueden implementar archivos estáticos de forma diferente en modo de producción.
    - Algunas plataformas web pueden usar scripts de inicio personalizados cuando se ejecutan en modo de producción.
- Ejecute la aplicación en App Service en el modo de desarrollo. Por ejemplo, en [MEAN.js](https://meanjs.org/), puede establecer la aplicación en modo de desarrollo en tiempo de ejecución [estableciendo la configuración de aplicación `NODE_ENV`](configure-common.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación Node.js con MongoDB](app-service-web-tutorial-nodejs-mongodb-app.md)

