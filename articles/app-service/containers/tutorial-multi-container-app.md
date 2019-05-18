---
title: 'Creación de aplicación de varios contenedores en Web App for Containers: Azure App Service'
description: Aprenda a utilizar varios contenedores en Azure con Docker Compose, WordPress y MySQL.
keywords: servicio de aplicación de azure, aplicación web, linux, docker, compose, multicontenedor, varios contenedores, aplicación web para contenedores, contenedor, wordpress, azure db for mysql, base de datos de producción con contenedores
services: app-service
documentationcenter: ''
author: msangapu
manager: jeconnoc
editor: ''
ms.service: app-service
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.date: 04/29/2019
ms.author: msangapu
ms.openlocfilehash: d4a93bbd823db59d688f77a626b1225519fb7b44
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65407615"
---
# <a name="tutorial-create-a-multi-container-preview-app-in-web-app-for-containers"></a>Tutorial: Creación de una aplicación de varios contenedores (versión preliminar) en Web App for Containers

[Web App for Containers](app-service-linux-intro.md) proporciona una manera flexible de utilizar imágenes de Docker. En este tutorial, aprenderá a crear una aplicación de varios contenedor usando WordPress y MySQL. Completará este tutorial en Cloud Shell, pero también puede ejecutar estos comandos localmente con la herramienta de la línea de comandos de la [CLI de Azure](/cli/azure/install-azure-cli) (versión 2.0.32 o posterior).

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Convertir una configuración de Docker Compose para trabajar con Web App for Containers
> * Implementar una aplicación de varios contenedor en Azure
> * Agregar la configuración de la aplicación
> * Utilizar el almacenamiento persistente para los contenedores
> * Conexión a Azure Database for MySQL
> * Solución de errores

[!INCLUDE [Free trial note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, se necesita experiencia con [Docker Compose](https://docs.docker.com/compose/).

## <a name="download-the-sample"></a>Descarga del ejemplo

Para este tutorial, utilice el archivo de Compose de [Docker](https://docs.docker.com/compose/wordpress/#define-the-project), pero tendrá que modificarlo incluyendo Azure Database for MySQL, el almacenamiento persistente y Redis. El archivo de configuración se pueden encontrar en los [ejemplos de Azure](https://github.com/Azure-Samples/multicontainerwordpress).

[!code-yml[Main](../../../azure-app-service-multi-container/docker-compose-wordpress.yml)]

Para ver las opciones de configuración que se admiten, consulte [Opciones de Docker Compose](configure-custom-container.md#docker-compose-options).

En Cloud Shell, cree un directorio de tutorial y luego cambie a él.

```bash
mkdir tutorial

cd tutorial
```

A continuación, ejecute el comando siguiente para clonar el repositorio de la aplicación de ejemplo en el directorio de tutorial. Después, cambie al directorio `multicontainerwordpress`.

```bash
git clone https://github.com/Azure-Samples/multicontainerwordpress

cd multicontainerwordpress
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [resource group intro text](../../../includes/resource-group.md)]

En Cloud Shell, cree un grupo de recursos con el comando [`az group create`](/cli/azure/group?view=azure-cli-latest#az-group-create). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *Centro y Sur de EE. UU.* Para ver todas las ubicaciones de App Service que se admiten en Linux en el nivel **Estándar**, ejecute el comando [`az appservice list-locations --sku S1 --linux-workers-enabled`](/cli/azure/appservice?view=azure-cli-latest#az-appservice-list-locations).

```azurecli-interactive
az group create --name myResourceGroup --location "South Central US"
```

Generalmente se crean el grupo de recursos y los recursos en una región cercana.

Cuando finaliza el comando, una salida de JSON muestra las propiedades del grupo de recursos.

## <a name="create-an-azure-app-service-plan"></a>Crear un plan de Azure App Service

En Cloud Shell, cree un plan de App Service en el grupo de recursos con el comando [`az appservice plan create`](/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create).

<!-- [!INCLUDE [app-service-plan](app-service-plan-linux.md)] -->

En el ejemplo siguiente se crea un plan de App Service denominado `myAppServicePlan` en el plan de tarifa **Estándar** (`--sku S1`) y en un contenedor Linux (`--is-linux`).

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux
```

Cuando se ha creado el plan de App Service, Cloud Shell muestra información similar al ejemplo siguiente:

```json
{
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "South Central US",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "linux",
  "location": "South Central US",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  < JSON data removed for brevity. >
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
}
```

### <a name="docker-compose-with-wordpress-and-mysql-containers"></a>Docker Compose con contenedores de WordPress y MySQL

## <a name="create-a-docker-compose-app"></a>Creación de una aplicación de Docker Compose

En Cloud Shell, cree una [aplicación web](app-service-linux-intro.md) de varios contenedores en el plan de App Service `myAppServicePlan` con el comando [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create). No olvide reemplazar _\<app-name_ por un nombre de aplicación único.

```azurecli-interactive
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

Cuando se haya creado la aplicación web, Cloud Shell mostrará información similar a la del ejemplo siguiente:

```json
{
  "additionalProperties": {},
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app-name>.azurewebsites.net",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

### <a name="browse-to-the-app"></a>Navegación hasta la aplicación

Vaya a la aplicación implementada en (`http://<app-name>.azurewebsites.net`). La aplicación puede tardar unos minutos en cargarse. Si recibe un error, deje pasar unos minutos, y actualice el explorador. Si tiene problemas y desea encontrar soluciones a los mismos, consulte los [registros de contenedor](#find-docker-container-logs).

![Aplicación de varios contenedores de ejemplo en Web App for Containers][1]

**Enhorabuena**, ha creado una aplicación de varios contenedores en Web App for Containers. A continuación va a configurar la aplicación para usar Azure Database for MySQL. No instale aún WordPress.

## <a name="connect-to-production-database"></a>Conexión a la base de datos de producción

No se recomienda utilizar contenedores de base de datos en un entorno de producción. Los contenedores locales no son escalables. En su lugar, deberá usar Azure Database for MySQL que sí se puede escalar.

### <a name="create-an-azure-database-for-mysql-server"></a>Creación de un servidor de Azure Database for MySQL

Cree un servidor de Azure Database for MySQL con el comando [`az mysql server create`](/cli/azure/mysql/server?view=azure-cli-latest#az-mysql-server-create).

En el siguiente comando, reemplace el nombre del servidor MySQL en el lugar en el que vea el marcador de posición _&lt;mysql-server-name>_ (los caracteres válidos son `a-z`, `0-9` y `-`). Este nombre forma parte del nombre de host del servidor MySQL (`<mysql-server-name>.database.windows.net`), por lo que es preciso que sea globalmente único.

```azurecli-interactive
az mysql server create --resource-group myResourceGroup --name <mysql-server-name>  --location "South Central US" --admin-user adminuser --admin-password My5up3rStr0ngPaSw0rd! --sku-name B_Gen4_1 --version 5.7
```

La creación del servidor puede tardar unos minutos en llevarse a cabo. Cuando se haya creado el servidor MySQL, Cloud Shell muestra información similar a la del siguiente ejemplo:

```json
{
  "administratorLogin": "adminuser",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "<mysql-server-name>.database.windows.net",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/<mysql-server-name>",
  "location": "southcentralus",
  "name": "<mysql-server-name>",
  "resourceGroup": "myResourceGroup",
  ...
}
```

### <a name="configure-server-firewall"></a>Configuración del firewall del servidor

Cree una regla de firewall para que el servidor MySQL permita conexiones de cliente con el comando [`az mysql server firewall-rule create`](/cli/azure/mysql/server/firewall-rule?view=azure-cli-latest#az-mysql-server-firewall-rule-create). Cuando tanto la dirección IP de inicio como final están establecidas en 0.0.0.0., el firewall solo se abre para otros recursos de Azure.

```azurecli-interactive
az mysql server firewall-rule create --name allAzureIPs --server <mysql-server-name> --resource-group myResourceGroup --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!TIP]
> Puede ser incluso más restrictivo con su regla de firewall [usando solo las direcciones IP de salida que utiliza su aplicación](../overview-inbound-outbound-ips.md?toc=%2fazure%2fapp-service%2fcontainers%2ftoc.json#find-outbound-ips).
>

### <a name="create-the-wordpress-database"></a>Creación de la base de datos de WordPress

```azurecli-interactive
az mysql db create --resource-group myResourceGroup --server-name <mysql-server-name> --name wordpress
```

Cuando se haya creado la base de datos, Cloud Shell muestra información similar a la del siguiente ejemplo:

```json
{
  "additionalProperties": {},
  "charset": "latin1",
  "collation": "latin1_swedish_ci",
  "id": "/subscriptions/12db1644-4b12-4cab-ba54-8ba2f2822c1f/resourceGroups/myResourceGroup/providers/Microsoft.DBforMySQL/servers/<mysql-server-name>/databases/wordpress",
  "name": "wordpress",
  "resourceGroup": "myResourceGroup",
  "type": "Microsoft.DBforMySQL/servers/databases"
}
```

### <a name="configure-database-variables-in-wordpress"></a>Configuración de las variables de la base de datos de WordPress

Para conectar la aplicación de WordPress a este nuevo servidor MySQL, tendrá que configurar algunas variables de entorno específicas de WordPress, incluida la ruta de acceso de la entidad emisora de certificados SSL definida por `MYSQL_SSL_CA`. El [Baltimore CyberTrust Root](https://www.digicert.com/digicert-root-certificates.htm) de [DigiCert](https://www.digicert.com/) se proporciona en la [imagen personalizada](https://docs.microsoft.com/azure/app-service/containers/tutorial-multi-container-app#use-a-custom-image-for-mysql-ssl-and-other-configurations) a continuación.

Para realizar estos cambios, use el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) en Cloud Shell. La configuración de la aplicación distingue mayúsculas de minúsculas y la separación se realiza mediante espacios.

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WORDPRESS_DB_HOST="<mysql-server-name>.mysql.database.azure.com" WORDPRESS_DB_USER="adminuser@<mysql-server-name>" WORDPRESS_DB_PASSWORD="My5up3rStr0ngPaSw0rd!" WORDPRESS_DB_NAME="wordpress" MYSQL_SSL_CA="BaltimoreCyberTrustroot.crt.pem"
```

Cuando se haya creado la configuración de aplicación, Cloud Shell muestra información similar al ejemplo siguiente:

```json
[
  {
    "name": "WORDPRESS_DB_HOST",
    "slotSetting": false,
    "value": "<mysql-server-name>.mysql.database.azure.com"
  },
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@<mysql-server-name>"
  },
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WORDPRESS_DB_PASSWORD",
    "slotSetting": false,
    "value": "My5up3rStr0ngPaSw0rd!"
  },
  {
    "name": "MYSQL_SSL_CA",
    "slotSetting": false,
    "value": "BaltimoreCyberTrustroot.crt.pem"
  }
]
```

Para más información acerca de las variables de entorno, consulte [Configuración de variables de entorno](configure-custom-container.md#configure-environment-variables).

### <a name="use-a-custom-image-for-mysql-ssl-and-other-configurations"></a>Uso de una imagen personalizada para SSL de MySQL y otras configuraciones

De forma predeterminada, Azure Database for MySQL usa SSL. WordPress requiere configuración adicional para usar SSL con MySQL. La "imagen oficial" de WordPress no proporciona la configuración adicional, pero se ha preparado una [imagen personalizada](https://github.com/Azure-Samples/multicontainerwordpress) para su comodidad. En la práctica, debe agregar los cambios deseados a su propia imagen.

La imagen personalizada se basa en la "imagen oficial" de [WordPress de Docker Hub](https://hub.docker.com/_/wordpress/). Los siguientes cambios se realizaron en esta imagen personalizada para Azure Database for MySQL:

* [Agrega el archivo de certificado de Baltimore Cyber Trust Root para SSL a MySQL.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L61)
* [Usa la configuración de aplicación para el certificado de entidad emisora de certificados de SSL de MySQL en WordPress wp-config.php.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L163)
* [Agrega WordPress "define" para MYSQL_CLIENT_FLAGS necesario para MySQL SSL.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L164)

Los siguientes cambios se realizaron para Redis (para su uso en una sección posterior):

* [Agrega la extensión PHP para Redis v4.0.2.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [Agrega la descompresión necesaria para la extracción de archivos.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [Agrega el complemento Redis Object Cache 1.3.8 WordPress.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [Usa la configuración de aplicación para el nombre de host de Redis en WordPress wp-config.php.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

Para usar la imagen personalizada, deberá actualizar el archivo docker-compose-wordpress.yml. En Cloud Shell, escriba `nano docker-compose-wordpress.yml` para abrir el editor de texto nano. Cambie `image: wordpress` para usar `image: microsoft/multicontainerwordpress`. Ya no necesita el contenedor de la base de datos. Elimine las secciones `db`, `environment`, `depends_on`, y `volumes` del archivo de configuración. El archivo debería tener el aspecto del siguiente código:

```yaml
version: '3.3'

services:
   wordpress:
     image: microsoft/multicontainerwordpress
     ports:
       - "8000:80"
     restart: always
```

Guarde los cambios y salga de nano. Use el comando `^O` para guardar y `^X` para salir.

### <a name="update-app-with-new-configuration"></a>Actualización de la aplicación con una configuración nueva

En Cloud Shell, vuelva a configurar la [aplicación web](app-service-linux-intro.md) de varios contenedores con el comando [az webapp config container set](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set). No olvide reemplazar _\<app-name >_ por el nombre de la aplicación web que creó anteriormente.

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

Cuando se haya vuelto a configurar la aplicación, Cloud Shell muestra información similar a la del ejemplo siguiente:

```json
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICB3b3JkcHJlc3M6CiAgICAgaW1hZ2U6IG1zYW5nYXB1L3dvcmRwcmVzcwogICAgIHBvcnRzOgogICAgICAgLSAiODAwMDo4MCIKICAgICByZXN0YXJ0OiBhbHdheXM="
  }
]
```

### <a name="browse-to-the-app"></a>Navegación hasta la aplicación

Vaya a la aplicación implementada en (`http://<app-name>.azurewebsites.net`). La aplicación está ahora usando Azure Database for MySQL.

![Aplicación de varios contenedores de ejemplo en Web App for Containers][1]

## <a name="add-persistent-storage"></a>Incorporación de almacenamiento persistente

La aplicación de varios contenedores se está ejecutando ahora en Web App for Containers. Sin embargo, si instala WordPress ahora y reinicia la aplicación más adelante, se encontrará con que la instalación de WordPress ha desaparecido. Esto se debe a que la configuración de Docker Compose apunta en la actualidad a una ubicación de almacenamiento dentro de su contenedor. Los archivos instalados en el contenedor no se conservan más allá del reinicio de la aplicación. En esta sección, va a [agregar almacenamiento persistente](configure-custom-container.md#use-persistent-shared-storage) a un contenedor de WordPress.

### <a name="configure-environment-variables"></a>Configuración de las variables de entorno

Para usar el almacenamiento persistente, tendrá que habilitar este valor dentro de App Service. Para realizar este cambio, use el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) en Cloud Shell. La configuración de la aplicación distingue mayúsculas de minúsculas y la separación se realiza mediante espacios.

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WEBSITES_ENABLE_APP_SERVICE_STORAGE=TRUE
```

Cuando se haya creado la configuración de aplicación, Cloud Shell muestra información similar al ejemplo siguiente:

```json
[
  < JSON data removed for brevity. >
  {
    "name": "WORDPRESS_DB_NAME",
    "slotSetting": false,
    "value": "wordpress"
  },
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  }
]
```

### <a name="modify-configuration-file"></a>Modificación del archivo de configuración

En Cloud Shell, escriba `nano docker-compose-wordpress.yml` para abrir el editor de texto nano.

La opción `volumes` asigna el sistema de archivos a un directorio dentro del contenedor. `${WEBAPP_STORAGE_HOME}` es una variable de entorno en App Service que se asigna al almacenamiento persistente para la aplicación. Deberá usar esta variable de entorno en la opción de volúmenes para que los archivos de WordPress se instalen en el almacenamiento persistente en lugar del contenedor. Realice las modificaciones siguientes en el archivo:

En la sección `wordpress`, agregue una opción `volumes` para que tenga un aspecto como el del siguiente código:

```yaml
version: '3.3'

services:
   wordpress:
     image: microsoft/multicontainerwordpress
     volumes:
      - ${WEBAPP_STORAGE_HOME}/site/wwwroot:/var/www/html
     ports:
       - "8000:80"
     restart: always
```

### <a name="update-app-with-new-configuration"></a>Actualización de la aplicación con una configuración nueva

En Cloud Shell, vuelva a configurar la [aplicación web](app-service-linux-intro.md) de varios contenedores con el comando [az webapp config container set](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set). No olvide reemplazar _\<app-name_ por un nombre de aplicación único.

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file docker-compose-wordpress.yml
```

Una vez que el comando se ejecute, muestra un resultado similar al ejemplo siguiente:

```json
[
  {
    "name": "WEBSITES_ENABLE_APP_SERVICE_STORAGE",
    "slotSetting": false,
    "value": "TRUE"
  },
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
```

### <a name="browse-to-the-app"></a>Navegación hasta la aplicación

Vaya a la aplicación implementada en (`http://<app-name>.azurewebsites.net`).

El contenedor de WordPress está utilizando Azure Database for MySQL y almacenamiento persistente.

## <a name="add-redis-container"></a>Incorporación del contenedor de Redis

 La "imagen oficial" de WordPress no incluye las dependencias para Redis. Estas dependencias y la configuración adicional necesaria para utilizar Redis con WordPress se han preparado para usted en esta [imagen personalizada](https://github.com/Azure-Samples/multicontainerwordpress). En la práctica, debe agregar los cambios deseados a su propia imagen.

La imagen personalizada se basa en la "imagen oficial" de [WordPress de Docker Hub](https://hub.docker.com/_/wordpress/). En esta imagen personalizada para Redis se han realizado los siguientes cambios:

* [Agrega la extensión PHP para Redis v4.0.2.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/Dockerfile#L35)
* [Agrega la descompresión necesaria para la extracción de archivos.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L71)
* [Agrega el complemento Redis Object Cache 1.3.8 WordPress.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L74)
* [Usa la configuración de aplicación para el nombre de host de Redis en WordPress wp-config.php.](https://github.com/Azure-Samples/multicontainerwordpress/blob/5669a89e0ee8599285f0e2e6f7e935c16e539b92/docker-entrypoint.sh#L162)

Agregue el contenedor de Redis a la parte inferior del archivo de configuración para que se parezca al siguiente ejemplo:

```yaml
version: '3.3'

services:
   wordpress:
     image: microsoft/multicontainerwordpress
     ports:
       - "8000:80"
     restart: always

   redis:
     image: redis:3-alpine
     restart: always
```

### <a name="configure-environment-variables"></a>Configuración de las variables de entorno

Para usar Redis, tendrá que habilitar este valor, `WP_REDIS_HOST`, dentro de App Service. Se trata de un *valor obligatorio* para que WordPress se comunique con el host de Redis. Para realizar este cambio, use el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings?view=azure-cli-latest#az-webapp-config-appsettings-set) en Cloud Shell. La configuración de la aplicación distingue mayúsculas de minúsculas y la separación se realiza mediante espacios.

```azurecli-interactive
az webapp config appsettings set --resource-group myResourceGroup --name <app-name> --settings WP_REDIS_HOST="redis"
```

Cuando se haya creado la configuración de aplicación, Cloud Shell muestra información similar al ejemplo siguiente:

```json
[
  < JSON data removed for brevity. >
  {
    "name": "WORDPRESS_DB_USER",
    "slotSetting": false,
    "value": "adminuser@<mysql-server-name>"
  },
  {
    "name": "WP_REDIS_HOST",
    "slotSetting": false,
    "value": "redis"
  }
]
```

### <a name="update-app-with-new-configuration"></a>Actualización de la aplicación con una configuración nueva

En Cloud Shell, vuelva a configurar la [aplicación web](app-service-linux-intro.md) de varios contenedores con el comando [az webapp config container set](/cli/azure/webapp/config/container?view=azure-cli-latest#az-webapp-config-container-set). No olvide reemplazar _\<app-name_ por un nombre de aplicación único.

```azurecli-interactive
az webapp config container set --resource-group myResourceGroup --name <app-name> --multicontainer-config-type compose --multicontainer-config-file compose-wordpress.yml
```

Una vez que el comando se ejecute, muestra un resultado similar al ejemplo siguiente:

```json
[
  {
    "name": "DOCKER_CUSTOM_IMAGE_NAME",
    "value": "COMPOSE|dmVyc2lvbjogJzMuMycKCnNlcnZpY2VzOgogICBteXNxbDoKICAgICBpbWFnZTogbXlzcWw6NS43CiAgICAgdm9sdW1lczoKICAgICAgIC0gZGJfZGF0YTovdmFyL2xpYi9teXNxbAogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgTVlTUUxfUk9PVF9QQVNTV09SRDogZXhhbXBsZXBhc3MKCiAgIHdvcmRwcmVzczoKICAgICBkZXBlbmRzX29uOgogICAgICAgLSBteXNxbAogICAgIGltYWdlOiB3b3JkcHJlc3M6bGF0ZXN0CiAgICAgcG9ydHM6CiAgICAgICAtICI4MDAwOjgwIgogICAgIHJlc3RhcnQ6IGFsd2F5cwogICAgIGVudmlyb25tZW50OgogICAgICAgV09SRFBSRVNTX0RCX1BBU1NXT1JEOiBleGFtcGxlcGFzcwp2b2x1bWVzOgogICAgZGJfZGF0YTo="
  }
]
```

### <a name="browse-to-the-app"></a>Navegación hasta la aplicación

Vaya a la aplicación implementada en (`http://<app-name>.azurewebsites.net`).

Complete los pasos e instale WordPress.

### <a name="connect-wordpress-to-redis"></a>Conexión de WordPress a Redis

Inicie sesión en el administrador de WordPress. En el panel de navegación izquierdo, seleccione **Plugins**y, a continuación, **Plugins instalados**.

![Selección Plugins de WordPress][2]

Mostrar todos los complementos aquí

En la página Plugins, busque **Redis Object Cache** y haga clic en **Activar**.

![Activación de Redis][3]

Haga clic en **Configuración**.

![Haga clic en Configuración][4]

Haga clic en el botón **Enable Object Cache**.

![Haga clic en el botón "Enable Object Cache"][5]

WordPress se conecta al servidor de Redis. El **estado** de la conexión aparece en la misma página.

![WordPress se conecta al servidor de Redis. El **estado** de la conexión aparece en la misma página.][6]

**Enhorabuena**, ha conectado WordPress a Redis. La aplicación preparada para el entorno de producción está utilizando ahora **Azure Database for MySQL, el almacenamiento persistente y Redis**. Ahora puede escalar horizontalmente su Plan de App Service para varias instancias.

## <a name="find-docker-container-logs"></a>Búsqueda de registros de Docker Container

Si experimenta problemas utilizando varios contenedores, puede acceder a los registros de contenedor a través de: `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.

Verá un resultado similar al del siguiente ejemplo:

```json
[
   {
      "machineName":"RD00XYZYZE567A",
      "lastUpdated":"2018-05-10T04:11:45Z",
      "size":25125,
      "href":"https://<app-name>.scm.azurewebsites.net/api/vfs/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log",
      "path":"/home/LogFiles/2018_05_10_RD00XYZYZE567A_docker.log"
   }
]
```

Verá un registro para cada contenedor y un registro adicional para el proceso primario. Copie el respectivo valor `href` en el explorador para ver el registro.

[!INCLUDE [Clean-up section](../../../includes/cli-script-clean-up.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial aprendió lo siguiente:
> [!div class="checklist"]
> * Convertir una configuración de Docker Compose para trabajar con Web App for Containers
> * Implementar una aplicación de varios contenedor en Azure
> * Agregar la configuración de la aplicación
> * Utilizar el almacenamiento persistente para los contenedores
> * Conexión a Azure Database for MySQL
> * Solución de errores

Vaya al siguiente tutorial para aprender a asignar un nombre DNS personalizado a una aplicación web.

> [!div class="nextstepaction"]
> [Tutorial: Asignar un nombre DNS personalizado a la aplicación](../app-service-web-tutorial-custom-domain.md)

O bien, eche un vistazo a otros recursos:

> [!div class="nextstepaction"]
> [Configuración de un contenedor personalizado](configure-custom-container.md)

<!--Image references-->
[1]: ./media/tutorial-multi-container-app/azure-multi-container-wordpress-install.png
[2]: ./media/tutorial-multi-container-app/wordpress-plugins.png
[3]: ./media/tutorial-multi-container-app/activate-redis.png
[4]: ./media/tutorial-multi-container-app/redis-settings.png
[5]: ./media/tutorial-multi-container-app/enable-object-cache.png
[6]: ./media/tutorial-multi-container-app/redis-connected.png
