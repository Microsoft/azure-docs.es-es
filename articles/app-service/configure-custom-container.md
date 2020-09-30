---
title: Configuración de un contenedor Linux personalizado
description: Aprenda a configurar un contenedor de Linux personalizado en Azure App Service. En este artículo se muestran las tareas de configuración más comunes.
ms.topic: article
ms.date: 03/28/2019
ms.openlocfilehash: 9a27abe5457cf8adf2963db545c629134ae53709
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89566983"
---
# <a name="configure-a-custom-linux-container-for-azure-app-service"></a>Configuración de un contenedor de Linux personalizado para Azure App Service

En este artículo se muestra cómo configurar un contenedor de Linux personalizado para que se ejecute en Azure App Service.

Esta guía incluye conceptos clave e instrucciones para la creación de contenedores de aplicaciones de Linux en App Service. Si nunca ha usado Azure App Service, siga primero el [inicio rápido de contenedores personalizados](quickstart-custom-container.md?pivots=container-linux) y el [tutorial](tutorial-custom-container.md?pivots=container-linux). También hay un [inicio rápido para aplicaciones de varios contenedores](quickstart-multi-container.md) y un [tutorial](tutorial-multi-container-app.md).

## <a name="configure-port-number"></a>Configuración del número de puerto

De forma predeterminada, App Service da por hecho que el contenedor personalizado escucha en el puerto 80. El servidor web de la imagen personalizada puede utilizar un puerto distinto al 80. Para indicar a Azure el puerto que usa el contenedor personalizado se utiliza el valor de la aplicación `WEBSITES_PORT`. La página de GitHub que contiene el [ejemplo de Python de este tutorial](https://github.com/Azure-Samples/docker-django-webapp-linux) muestra que es preciso establecer `WEBSITES_PORT` en _8000_. Puede establecerlo ejecutando el comando [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) en Cloud Shell. Por ejemplo:

```azurecli-interactive
az webapp config appsettings set --resource-group <resource-group-name> --name <app-name> --settings WEBSITES_PORT=8000
```

## <a name="configure-environment-variables"></a>Configuración de las variables de entorno

El contenedor personalizado puede usar variables de entorno que se deben proporcionar de forma externa. Puede pasarlos ejecutando el comando [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) en Cloud Shell. Por ejemplo:

```azurecli-interactive
az webapp config appsettings set --resource-group <resource-group-name> --name <app-name> --settings WORDPRESS_DB_HOST="myownserver.mysql.database.azure.com"
```

Este método funciona tanto para aplicaciones de contenedor único o de varios contenedores, en los que las variables de entorno se especifican en el archivo *docker-compose.yml*.

## <a name="use-persistent-shared-storage"></a>Uso de almacenamiento compartido persistente

Puede usar el directorio */home* en el sistema de archivos de la aplicación para conservar archivos entre reinicios y compartirlos entre instancias. El directorio `/home` de la aplicación se proporciona para permitir a la aplicación de contenedor el acceso al almacenamiento persistente.

Si se deshabilita el almacenamiento persistente, no se conservarán las escrituras en el directorio `/home` entre reinicios de aplicación ni entre varias instancias. La única excepción es el directorio `/home/LogFiles`, que se usa para almacenar los registros de Docker y del contenedor. Cuando se habilita el almacenamiento persistente, se conservan todas las escrituras en el directorio `/home` y todas las instancias de una aplicación escalada horizontalmente podrán acceder a ellas.

De forma predeterminada, el almacenamiento persistente está *habilitado* y la configuración no está expuesta en la configuración de la aplicación. Para deshabilitarlo, establezca el valor de la aplicación `WEBSITES_ENABLE_APP_SERVICE_STORAGE` mediante la ejecución del comando [`az webapp config appsettings set`](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) en Cloud Shell. Por ejemplo:

```azurecli-interactive
az webapp config appsettings set --resource-group <resource-group-name> --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=false
```

> [!NOTE]
> También puede [configurar su propio almacenamiento persistente](configure-connect-to-azure-storage.md?pivots=platform-linux).

## <a name="enable-ssh"></a>Habilite SSH

SSH habilita la comunicación segura entre un contenedor y un cliente. Para que un contenedor personalizado admita SSH, deberá agregarlo en el propio archivo de Dockerfile.

> [!TIP]
> Todos los contenedores de Linux integrados han agregado las instrucciones de SSH en sus repositorios de imágenes. Puede seguir las instrucciones siguientes con el [repositorio de Node.js 10.14](https://github.com/Azure-App-Service/node/blob/master/10.14) para ver cómo se habilita allí.

- Use la instrucción [RUN](https://docs.docker.com/engine/reference/builder/#run) para instalar el servidor SSH y establecer la contraseña para la cuenta raíz en `"Docker!"`. Por ejemplo, para una imagen basada en [Alpine Linux](https://hub.docker.com/_/alpine), necesita los siguientes comandos:

    ```Dockerfile
    RUN apk add openssh \
         && echo "root:Docker!" | chpasswd 
    ```

    Esta configuración no permite realizar conexiones externas al contenedor. SSH solo está disponible en `https://<app-name>.scm.azurewebsites.net` y después de autenticarse con las credenciales de publicación.

- Agregue [este archivo sshd_config](https://github.com/Azure-App-Service/node/blob/master/10.14/sshd_config) al repositorio de imágenes y use la instrucción [COPY](https://docs.docker.com/engine/reference/builder/#copy) para copiar el archivo en el directorio */etc/ssh/* . Para más información acerca de los archivos *sshd_config*, consulte la [documentación de OpenBSD](https://man.openbsd.org/sshd_config).

    ```Dockerfile
    COPY sshd_config /etc/ssh/
    ```

    > [!NOTE]
    > El archivo *sshd_config* debe incluir los elementos siguientes:
    > - `Ciphers` debe incluir al menos un elemento de esta lista: `aes128-cbc,3des-cbc,aes256-cbc`.
    > - `MACs` debe incluir al menos un elemento de esta lista: `hmac-sha1,hmac-sha1-96`.

- Use la instrucción [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose) para abrir el puerto 2222 en el contenedor. Aunque se conozca la contraseña raíz, no se puede acceder al puerto 2222 desde Internet. Solo se puede acceder mediante los contenedores que se encuentren en el puente de una red privada virtual.

    ```Dockerfile
    EXPOSE 80 2222
    ```

- En el script de inicio del contenedor, inicie el servidor SSH.

    ```bash
    /usr/sbin/sshd
    ```

    Para obtener un ejemplo, consulte cómo el [contenedor de Node.js 10.14](https://github.com/Azure-App-Service/node/blob/master/10.14/startup/init_container.sh) predeterminado inicia el servidor SSH.

## <a name="access-diagnostic-logs"></a>Acceso a los registros de diagnóstico

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

## <a name="change-the-docker-image-of-a-custom-container"></a>Cambio de la imagen de Docker de un contenedor personalizado

Para sustituir la imagen actual de una aplicación de contenedor personalizada en Docker por otra imagen, utilice el siguiente comando:

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <docker-hub-repo>/<image>
```

## <a name="use-an-image-from-a-private-registry"></a>Uso de una imagen de un registro privado

Para utilizar una imagen de un registro privado, como Azure Container Registry, ejecute el siguiente comando:

```azurecli-interactive
az webapp config container set --name <app-name> --resource-group <group-name> --docker-custom-image-name <image-name> --docker-registry-server-url <private-repo-url> --docker-registry-server-user <username> --docker-registry-server-password <password>
```

En *\<username>* y *\<password>* , especifique las credenciales de inicio de sesión de su cuenta de registro privada.

## <a name="configure-multi-container-apps"></a>Configuración de aplicaciones de varios contenedores

- [Uso del almacenamiento persistente en Docker Compose](#use-persistent-storage-in-docker-compose)
- [Limitaciones de la versión preliminar](#preview-limitations)
- [Opciones de Docker Compose](#docker-compose-options)

### <a name="use-persistent-storage-in-docker-compose"></a>Uso del almacenamiento persistente en Docker Compose

Las aplicaciones de varios contenedores como WordPress necesitan almacenamiento persistente para funcionar correctamente. Para habilitarla, la configuración de Docker Compose debe apuntar a una ubicación de almacenamiento *fuera* del contenedor. Las ubicaciones de almacenamiento dentro del contenedor no conservan los cambios más allá del reinicio de la aplicación.

Habilite el almacenamiento persistente, establezca el valor de la aplicación `WEBSITES_ENABLE_APP_SERVICE_STORAGE` mediante el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) en Cloud Shell.

```azurecli-interactive
az webapp config appsettings set --resource-group <resource-group-name> --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

En el archivo *docker-compose.yml*, asigne la opción `volumes` a `${WEBAPP_STORAGE_HOME}`. 

`WEBAPP_STORAGE_HOME` es una variable de entorno en App Service que se asigna al almacenamiento persistente para la aplicación. Por ejemplo:

```yaml
wordpress:
  image: wordpress:latest
  volumes:
  - ${WEBAPP_STORAGE_HOME}/site/wwwroot:/var/www/html
  - ${WEBAPP_STORAGE_HOME}/phpmyadmin:/var/www/phpmyadmin
  - ${WEBAPP_STORAGE_HOME}/LogFiles:/var/log
```

### <a name="preview-limitations"></a>Limitaciones de vista previa

La aplicación de varios contenedores está actualmente en versión preliminar. Las siguientes características de la plataforma de App Service no se admiten:

- Autenticación/autorización
- Identidades administradas
- CORS

### <a name="docker-compose-options"></a>Opciones de Docker Compose

Las listas siguientes muestran opciones de configuración admitidas y no admitidas de Docker Compose:

#### <a name="supported-options"></a>Opciones admitidas

- command
- entrypoint
- Environment
- imagen
- ports
- restart
- services
- volumes

#### <a name="unsupported-options"></a>Opciones no admitidas

- build (no permitido)
- depends_on (omitido)
- networks (omitido)
- secrets (omitido)
- puertos que no sean el 80 y 8080 (omitido)

> [!NOTE]
> Cualquier otra opción que no se mencione de forma explícita, se omitirá en la versión preliminar pública.

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Implementar desde el repositorio de contenedor privado](tutorial-custom-container.md?pivots=container-linux)

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de WordPress con varios contenedores](tutorial-multi-container-app.md)
