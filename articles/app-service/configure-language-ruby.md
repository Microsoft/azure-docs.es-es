---
title: 'Configuración de aplicaciones de Ruby: Azure App Service'
description: Aprenda a configurar un contenedor de Ruby precompilado para la aplicación. En este artículo se muestran las tareas de configuración más comunes.
ms.topic: quickstart
ms.date: 06/18/2020
ms.reviewer: astay; kraigb
ms.custom: mvc, seodec18
ms.openlocfilehash: c822dbdf9940db7b38d354fa32906c16977df0c0
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88084278"
---
# <a name="configure-a-linux-ruby-app-for-azure-app-service"></a>Configuración de una aplicación de Ruby en Linux para Azure App Service

En este artículo se describe la forma en que [Azure App Service](overview.md) ejecuta las aplicaciones Ruby en un contenedor Linux y cómo se puede personalizar el comportamiento de App Service cuando es necesario. Las aplicaciones de Ruby deben implementarse con todos los archivos [gem](https://rubygems.org/gems) requeridos.

En esta guía se incluyen conceptos clave e instrucciones para los desarrolladores de Ruby que usan un contenedor Linux integrado en App Service. Si nunca ha usado Azure App Service, debe seguir primero el [inicio rápido de Ruby](quickstart-ruby.md) y el [tutorial de Ruby con PostgreSQL](tutorial-ruby-postgres-app.md).

## <a name="show-ruby-version"></a>Consulta de la versión de Ruby

Para mostrar la versión actual de Ruby, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
```

Para mostrar todas las versiones compatibles de Ruby, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes --linux | grep RUBY
```

Para ejecutar una versión no compatible de Ruby, cree su propia imagen de contenedor. Para más información, consulte [Uso de una imagen personalizada de Docker](tutorial-custom-container.md?pivots=container-linux).

## <a name="set-ruby-version"></a>Establecimiento de la versión de Ruby

Ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com) para establecer la versión 2.3 de Ruby:

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --linux-fx-version "RUBY|2.3"
```

> [!NOTE]
> Si ve errores similares al siguiente durante el tiempo de implementación:
> ```
> Your Ruby version is 2.3.3, but your Gemfile specified 2.3.1
> ```
> or
> ```
> rbenv: version `2.3.1' is not installed
> ```
> Significa que la versión de Ruby configurada en el proyecto es distinta de la instalada en el contenedor que se está ejecutando (`2.3.3` en el ejemplo anterior). En el ejemplo anterior, compruebe *Gemfile* y *.ruby-version*, y que la versión de Ruby no está establecida o que es la que está instalada en el contenedor en ejecución (`2.3.3` en el ejemplo anterior).

## <a name="access-environment-variables"></a>Acceso a variables de entorno

En App Service, puede [establecer la configuración de la aplicación](configure-common.md#configure-app-settings) fuera del código de la aplicación. Luego podrá acceder a ellas mediante el patrón estándar [ENV['\<path-name>']](https://ruby-doc.org/core-2.3.3/ENV.html). Por ejemplo, para acceder a una configuración de una aplicación denominada `WEBSITE_SITE_NAME`, use el código siguiente:

```ruby
ENV['WEBSITE_SITE_NAME']
```

## <a name="customize-deployment"></a>Personalización de la implementación

Al implementar un [repositorio de Git](deploy-local-git.md) o un [paquete comprimido](deploy-zip.md) con procesos compilados activados, el motor de implementación (Kudu) ejecuta los siguientes pasos después de la implementación de manera automática y predeterminada:

1. Compruebe si un existe *Gemfile*.
1. Ejecute `bundle clean`. 
1. Ejecute `bundle install --path "vendor/bundle"`.
1. Ejecute `bundle package` para empaquetar los archivos gem en la carpeta vendor/cache.

### <a name="use---without-flag"></a>Uso de la marca --without

Para ejecutar `bundle install` con la marca [--without](https://bundler.io/man/bundle-install.1.html), establezca la [configuración de la aplicación](configure-common.md#configure-app-settings) `BUNDLE_WITHOUT` en una lista de grupos separada por comas. Por ejemplo, el siguiente comando la establece en `development,test`.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings BUNDLE_WITHOUT="development,test"
```

Con esta configuración definida, el motor de implementación ejecuta `bundle install` con `--without $BUNDLE_WITHOUT`.

### <a name="precompile-assets"></a>Precompilación de los recursos

Los pasos posteriores a la implementación no precompilan los recursos de forma predeterminada. Para activar la precompilación de recursos, establezca la [configuración de la aplicación](configure-common.md#configure-app-settings) `ASSETS_PRECOMPILE` en `true`. El comando `bundle exec rake --trace assets:precompile` se ejecutará al final de los pasos posteriores a la implementación. Por ejemplo:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings ASSETS_PRECOMPILE=true
```

Para más información, consulte [Servicio de recursos estáticos](#serve-static-assets).

## <a name="customize-start-up"></a>Personalización del inicio

De forma predeterminada, el contenedor de Ruby inicia el servidor de Rails en la siguiente secuencia (para más información, consulte el [script de inicio](https://github.com/Azure-App-Service/ruby/blob/master/2.3.8/startup.sh)):

1. Genere un valor de [secret_key_base](https://edgeguides.rubyonrails.org/security.html#environmental-security), si aún no existe ninguno. Este valor es necesario para que la aplicación se ejecute en modo de producción.
1. Establezca la variable de entorno `RAILS_ENV` en `production`.
1. Elimine cualquier archivo *.pid* del directorio *tmp/PID* que haya dejado un servidor de Rails que se ejecutó previamente.
1. Compruebe si están instaladas todas las dependencias. Si no, pruebe a instalar los archivos gem desde el directorio *vendor/cache*.
1. Ejecute `rails server -e $RAILS_ENV`.

Puede personalizar el proceso de inicio de las siguientes maneras:

- [Servicio de recursos estáticos](#serve-static-assets)
- [Ejecución en otros modos distintos del de producción](#run-in-non-production-mode)
- [Establecimiento manual de secret_key_base](#set-secret_key_base-manually)

### <a name="serve-static-assets"></a>Servicio de recursos estáticos

El servidor de Rails en el contenedor de Ruby se ejecuta en modo de producción de forma predeterminada y [da por supuesto que los recursos están precompilados y los sirve el servidor web](https://guides.rubyonrails.org/asset_pipeline.html#in-production). Para servir recursos estáticos desde el servidor de Rails, es preciso hacer dos cosas:

- **Precompilar los recursos** - [precompile los recursos estáticos localmente](https://guides.rubyonrails.org/asset_pipeline.html#local-precompilation) e impleméntelos de forma manual. O bien, permita que el motor de implementación lo haga en su lugar (consulte [Precompilación de los recursos](#precompile-assets).
- **Habilitar el servicio de archivos estáticos**: para servir recursos estáticos desde el contenedor de Ruby, [establezca la opción de configuración de la aplicación `RAILS_SERVE_STATIC_FILES`](configure-common.md#configure-app-settings) en `true`. Por ejemplo:

    ```azurecli-interactive
    az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings RAILS_SERVE_STATIC_FILES=true
    ```

### <a name="run-in-non-production-mode"></a>Ejecución en otros modos distintos del de producción

El servidor de Rails se ejecuta en modo de producción de forma predeterminada. Para ejecutarlo en modo de desarrollo, por ejemplo, establezca la [configuración de la aplicación](configure-common.md#configure-app-settings) `RAILS_ENV` en `development`.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings RAILS_ENV="development"
```

Sin embargo, esta configuración por sí sola hace que el servidor de Rails se inicie en modo de desarrollo, que acepta solo solicitudes localhost y no es accesible fuera del contenedor. Para aceptar las solicitudes de cliente remoto, establezca la [configuración de la aplicación](configure-common.md#configure-app-settings) `APP_COMMAND_LINE` en `rails server -b 0.0.0.0`. Esta configuración de aplicación permite ejecutar un comando personalizado en el contenedor de Ruby. Por ejemplo:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings APP_COMMAND_LINE="rails server -b 0.0.0.0"
```

### <a name="set-secret_key_base-manually"></a><a name="set-secret_key_base-manually"></a> Establecimiento manual de secret_key_base

Para utilizar su propio valor de `secret_key_base` en lugar de dejar que App Service genere uno automáticamente, establezca la [configuración de la aplicación](configure-common.md#configure-app-settings) `SECRET_KEY_BASE` en el valor que desee. Por ejemplo:

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings SECRET_KEY_BASE="<key-base-value>"
```

## <a name="access-diagnostic-logs"></a>Acceso a los registros de diagnóstico

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

## <a name="open-ssh-session-in-browser"></a>Abrir sesión SSH en el explorador

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de Rails con PostgreSQL](tutorial-ruby-postgres-app.md)

> [!div class="nextstepaction"]
> [P+F sobre App Service en Linux](faq-app-service-linux.md)
