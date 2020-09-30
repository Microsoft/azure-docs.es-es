---
title: 'Tutorial: Implementación de una aplicación Django con App Service y el servidor flexible de Azure Database for PostgreSQL (versión preliminar) en una red virtual'
description: Implemente una aplicación Django con App Service y el servidor flexible de Azure Database for PostgreSQL (versión preliminar) en una red virtual
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 09/22/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 3366f39f3aca8ad0114244c122d1003b5e9b91a3
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90929729"
---
# <a name="tutorial-deploy-django-app-with-app-service-and-azure-database-for-postgresql---flexible-server-preview"></a>Tutorial: Implementación de una aplicación Django con App Service y el servidor flexible de Azure Database for PostgreSQL (versión preliminar)

> [!IMPORTANT]
> El servidor flexible de Azure Database for PostgreSQL se encuentra en versión preliminar

En este tutorial, aprenderá a implementar una aplicación de Django en Azure mediante App Services y el servidor flexible de Azure Database for PostgreSQL en una red virtual.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

En este artículo es necesario que ejecute la versión 2.0 de la CLI de Azure, o cualquier versión posterior, de forma local. Para ver la versión instalada, ejecute el comando `az --version`. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

Deberá iniciar sesión en su cuenta mediante el comando [az login](/cli/azure/authenticate-azure-cli?view=interactive-log-in). Anote la propiedad **id** de la salida del comando para el nombre de la suscripción correspondiente.

```azurecli
az login
```

Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione el identificador de suscripción específico en su cuenta mediante el comando [az account set](/cli/azure/account). Sustituya la propiedad **subscription ID** de la salida de **az login** de su suscripción en el marcador de posición de identificador de suscripción.

```azurecli
az account set --subscription <subscription id>
```
## <a name="clone-or-download-the-sample-app"></a>Clonación o descarga de la aplicación de ejemplo

# <a name="git-clone"></a>[Clonación de Git](#tab/clone)

Clone el repositorio de ejemplo:

```terminal
git clone https://github.com/Azure-Samples/djangoapp
```

A continuación, vaya a esa carpeta:

```terminal
cd djangoapp
```

# <a name="download"></a>[Descargar](#tab/download)

Visite [https://github.com/Azure-Samples/djangoapp](https://github.com/Azure-Samples/djangoapp), seleccione **Clonar** y, a continuación, seleccione **Descargar archivo ZIP**.

Descomprima el archivo ZIP en una carpeta denominada *djangoapp*.

A continuación, abra una ventana del terminal en la carpeta *djangoapp*.

---

El ejemplo djangoapp contiene la aplicación en Django de sondeos controlada por datos que se obtiene al seguir [Creación de la primera aplicación Django](https://docs.djangoproject.com/en/2.1/intro/tutorial01/) de la documentación de Django. La aplicación completa se proporciona aquí para su comodidad.

El ejemplo también se ha modificado para ejecutarse en un entorno de producción, como App Service:

- La configuración para producción está en el archivo *azuresite/production.py*. Los detalles de desarrollo están en el archivo *azuresite/settings.py*.
- La aplicación usa la configuración de producción cuando la variable de entorno `DJANGO_ENV` está establecida en "production". Esta variable de entorno se crea más adelante en el tutorial, junto con otras que se usan para la configuración de la base de datos de PostgreSQL.

Estos cambios son específicos de la configuración de Django para ejecutarse en cualquier entorno de producción y no son específicos de App Service. Para obtener más información, consulte la [lista de implementación de Django](https://docs.djangoproject.com/en/2.1/howto/deployment/checklist/).

## <a name="create-a-postgresql-flexible-server-in-a-new-virtual-network"></a>Creación de un servidor flexible de PostgreSQL en una nueva red virtual

Cree un servidor flexible privado y una base de datos dentro de una red virtual (VNet) con el siguiente comando:
```azurecli
# Create Flexible server in a VNET

az postgres flexible-server create --resource-group myresourcegroup --location westus2

```

Este comando realiza las siguientes acciones, que pueden tardar algunos minutos:

- Cree el grupo de recursos si todavía no existe.
- Genera un nombre de servidor, si no se proporciona alguno.
- Crea una nueva red virtual para el nuevo servidor de PostgreSQL. **Anote el nombre de la red virtual y de la subred** que se crearon para el servidor, ya que debe agregar la aplicación web a la misma red virtual.
- Crea un nombre de usuario administrador y una contraseña para el servidor, si no se proporciona alguna. **Tome nota del nombre de usuario y la contraseña** para usarlos en el paso siguiente.
- Cree una base de datos ```postgres``` que se pueda usar para el desarrollo. Puede ejecutar el cliente [**psql** para conectarse a la base de datos](quickstart-create-server-portal.md#connect-to-the-postgresql-database-using-psql) y crear una base de datos diferente.

> [!NOTE]
> Anote la contraseña que se generará si no proporciona alguna. Si olvida la contraseña, tendrá que restablecerla con el comando ``` az postgres flexible-server update```.


## <a name="deploy-the-code-to-azure-app-service"></a>Implementación del código en Azure App Service

En esta sección, creará un host de aplicación en la aplicación de App Service, conectará esta aplicación a la base de datos de Postgres y, a continuación, implementará el código en ese host.


### <a name="create-the-app-service-web-app-in-a-virtual-network"></a>Creación de una aplicación web de App Service en una red virtual

En el terminal, asegúrese de que se encuentra en la raíz del repositorio (`djangoapp`) que contiene el código de la aplicación.

Cree una aplicación de App Service (el proceso de host) con el comando [`az webapp up`](/cli/azure/webapp#az-webapp-up):

```azurecli

# Create a web app
az webapp up --resource-group myresourcegroup --location westus2 --plan DjangoPostgres-tutorial-plan --sku B1 --name <app-name>

# Enable VNET integration for web app.
# Replace <vnet-name> and <subnet-name> with the virtual network and subnet name that the flexible server is using.

az webapp vnet-integration add -g myresourcegroup -n  mywebapp --vnet <vnet-name> --subnet <subnet-name>

# Configure database information as environment variables
# Use the postgres server name , database name , username , password for the database created in the previous steps

az webapp config appsettings set --settings DJANGO_ENV="production" DBHOST="<postgres-server-name>.postgres.database.azure.com" DBNAME="postgres" DBUSER="<username>" DBPASS="<password>"
```
- Para el argumento `--location`, use la misma ubicación que usó para la base de datos en la sección anterior.
- Reemplace *\<app-name>* por un nombre exclusivo en todo Azure (el punto de conexión del servidor es `https://\<app-name>.azurewebsites.net`). Los caracteres permitidos para *\<app-name>* son `A`-`Z`, `0`-`9` y `-`. Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.
- Cree el [plan de App Service](../../app-service/overview-hosting-plans.md) *DjangoPostgres-tutorial-plan* en el plan de tarifa Básico (B1), si no existe. `--plan` y `--sku` son opcionales.
- Cree la aplicación de App Service, si no existe.
- Habilite el registro predeterminado de la aplicación, si aún no está habilitado.
- Cargue el repositorio mediante la implementación del archivo ZIP con la automatización de compilación habilitada.
- El comando **az webapp vnet-integration** agrega la aplicación web en la misma red virtual que el servidor de Postgres.
- El código de la aplicación espera encontrar información de la base de datos en una serie de variables de entorno. Para establecer variables de entorno en App Service, cree "valores de aplicación" mediante el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set).

> [!TIP]
> Muchos comandos de la CLI de Azure almacenan en caché los parámetros comunes, como el nombre del grupo de recursos y el plan de App Service, en el archivo *.azure/config*. Como consecuencia, no es necesario especificar el mismo parámetro con comandos posteriores. Por ejemplo, para volver a implementar la aplicación después de realizar cambios, puede ejecutar `az webapp up` de nuevo sin ningún parámetro.

### <a name="run-django-database-migrations"></a>Ejecución de migraciones de base de datos de Django

Las migraciones de bases de datos de Django aseguran que el esquema de la base de datos de PostgreSQL en Azure coincida con las descritas en el código.

1. Abra una sesión de SSH en el explorador; para ello, vaya a *https://\<app-name>.scm.azurewebsites.net/webssh/host* e inicie sesión con sus credenciales de la cuenta de Azure (no las credenciales del servidor de bases de datos).

1. En la sesión de SSH, ejecute los siguientes comandos (puede pegar los comandos con **CTRL**+**Mayús**+**V**):

    ```bash
    cd site/wwwroot

    # Activate default virtual environment in App Service container
    source /antenv/bin/activate
    # Install packages
    pip install -r requirements.txt
    # Run database migrations
    python manage.py migrate
    # Create the super user (follow prompts)
    python manage.py createsuperuser
    ```

1. El comando `createsuperuser` le pide las credenciales de superusuario. A los efectos de este tutorial, use el nombre de usuario predeterminado `root`, presione **ENTRAR** en la dirección de correo electrónico para que quede en blanco y escriba `postgres1` como contraseña.

### <a name="create-a-poll-question-in-the-app"></a>Creación de una pregunta de sondeo en la aplicación

1. En un explorador, abra la dirección URL *http:\//\<app-name>.azurewebsites.net*. La aplicación debe mostrar el mensaje "No polls are available" (No hay sondeos disponibles) porque todavía no hay sondeos específicos en la base de datos.

1. Vaya a *http:\//\<app-name>.azurewebsites.net/admin*. Inicie sesión con las credenciales de superusuario de la sección anterior (`root` y `postgres1`). En **Sondeos**, seleccione **Agregar** junto a **Preguntas** y cree una pregunta de sondeo con opciones.

1. Navegue nuevamente a *http:\//\<app-name>.azurewebsites.net/* para confirmar que las preguntas ahora se presenten al usuario. Responda a las preguntas como desee para generar algunos datos en la base de datos.

**¡Enhorabuena!** Está ejecutando una aplicación web de Django de Python en Azure App Service para Linux, con una base de datos de Postgres activa.

> [!NOTE]
> App Service detecta un proyecto de Django mediante la búsqueda de un archivo *wsgi.py* en cada subcarpeta, que `manage.py startproject` crea de forma predeterminada. Cuando App Service encuentra ese archivo, carga la aplicación web de Django. Para obtener más información, consulte [Configuración de una imagen de Python integrada](../../app-service/configure-language-python.md).

## <a name="make-code-changes-and-redeploy"></a>Cambios en el código y reimplementación

En esta sección, realizará cambios locales en la aplicación y volverá a implementar el código en App Service. En el proceso, se configura un entorno virtual de Python que admite el trabajo en curso.

### <a name="run-the-app-locally"></a>Ejecución de la aplicación de forma local

Ejecute los siguientes comandos en una ventana de terminal. Asegúrese de seguir las indicaciones al crear el superusuario:

```bash
# Configure the Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install packages
pip install -r requirements.txt
# Run Django migrations
python manage.py migrate
# Create Django superuser (follow prompts)
python manage.py createsuperuser
# Run the dev server
python manage.py runserver
```
Una vez que la aplicación web esté completamente cargada, el servidor de desarrollo de Django proporcionará la dirección URL de la aplicación local en el mensaje "Starting development server at http://127.0.0.1:8000/. Quit the server with CTRL-BREAK" (Iniciando el servidor de desarrollo en https… Salga del servidor con Ctrl+Interrumpir).

:::image type="content" source="./media/tutorial-django-app-service-postgres/django-dev-server-output.png" alt-text="Ejemplo de salida del servidor de desarrollo de Django":::

Pruebe la aplicación localmente con los pasos siguientes:

1. Vaya a *http:\//localhost:8000* en un explorador, que debería mostrar el mensaje No polls are available (No hay sondeos disponibles).

1. Vaya a *http:\//localhost:8000/admin* e inicie sesión con el usuario administrador que creó anteriormente. En **Sondeos**, vuelva a seleccionar **Agregar** junto a **Preguntas** y cree una pregunta de sondeo con opciones.

1. Vaya a *http:\//localhost: 8000* de nuevo y responda la pregunta para probar la aplicación.

1. Detenga el servidor de Django presionando **CTRL**+**C**.

Cuando se ejecute localmente, la aplicación usará una base de datos de Sqlite3 local y no interferirá con la base de datos de producción. También puede usar una base de datos de PostgreSQL local, si lo desea, para simular mejor el entorno de producción.



### <a name="update-the-app"></a>Actualización de la aplicación

En `polls/models.py`, busque la línea que comienza con `choice_text` y cambie el parámetro `max_length` a 100:

```python
# Find this lie of code and set max_length to 100 instead of 200
choice_text = models.CharField(max_length=100)
```

Dado que ha cambiado el modelo de datos, cree una nueva migración de Django y migre la base de datos:

```
python manage.py makemigrations
python manage.py migrate
```

Vuelva a ejecutar el servidor de desarrollo con `python manage.py runserver` y pruebe la aplicación en *http:\//localhost:8000/admin*:

Vuelva a detener el servidor web de Django con **CTRL**+**C**.


### <a name="redeploy-the-code-to-azure"></a>Nueva implementación del código en Azure

Ejecute el comando siguiente en la raíz del repositorio:

```azurecli
az webapp up
```

Este comando usa los parámetros almacenados en la memoria caché del archivo *.azure/config*. Dado que App Service detecta que la aplicación ya existe, simplemente vuelve a implementar el código.



### <a name="rerun-migrations-in-azure"></a>Nueva ejecución de migraciones en Azure

Dado que ha realizado cambios en el modelo de datos, debe volver a ejecutar las migraciones de base de datos en App Service.

Vuelva a abrir una sesión de SSH en el explorador; para ello, vaya a *https://\<app-name>.scm.azurewebsites.net/webssh/host*. Luego, ejecute los siguientes comandos:

```
cd site/wwwroot

# Activate default virtual environment in App Service container
source /antenv/bin/activate
# Run database migrations
python manage.py migrate
```

### <a name="review-app-in-production"></a>Revisión de la aplicación en producción

Vaya a *http:\//\<app-name>.azurewebsites.net* y vuelva a probar la aplicación en producción. (Dado que solo ha cambiado la longitud de un campo de la base de datos, el cambio solo se aprecia si intenta especificar una respuesta más larga al crear una pregunta).

> [!TIP]
> Puede usar [django-storages](https://django-storages.readthedocs.io/en/latest/backends/azure.html) para almacenar recursos multimedia y estáticos en Azure Storage. Puede usar Azure CDN para comprimir en formato GZip los archivos estáticos.


## <a name="manage-your-app-in-the-azure-portal"></a>Administración de una aplicación en Azure Portal

En [Azure Portal](https://portal.azure.com), busque el nombre de la aplicación y seleccione la aplicación en los resultados.

:::image type="content" source="./media/tutorial-django-app-service-postgres/navigate-to-django-app-in-app-services-in-the-azure-portal.png" alt-text="Vaya a la aplicación Python Django en Azure Portal":::

De forma predeterminada, el portal muestra la página **Información general** de la aplicación, que proporciona una vista del rendimiento general. En ella puede también realizar tareas de administración básicas como examinar, detener, reiniciar y eliminar. Las pestañas del lado izquierdo de la página muestran las diferentes páginas de configuración que puede abrir.

:::image type="content" source="./media/tutorial-django-app-service-postgres/manage-django-app-in-app-services-in-the-azure-portal.png" alt-text="Administración de la aplicación Python Django en la página de información general de Azure Portal":::


## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea mantener la aplicación o continuar con el siguiente tutorial, vaya a [Pasos siguientes](#next-steps). De lo contrario, para evitar incurrir en cargos adicionales, puede eliminar la creación del grupo de recursos para este tutorial:

```azurecli
az group delete -g myresourcegroup
```

Este comando usa el nombre del grupo de recursos almacenado en la memoria caché del archivo *.azure/config*. Al eliminar el grupo de recursos, también se desasignan y eliminan todos los recursos que contiene.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo asignar un nombre DNS personalizado a la aplicación:

> [!div class="nextstepaction"]
> [Tutorial: Asignar un nombre DNS personalizado a la aplicación](../../app-service/app-service-web-tutorial-custom-domain.md)

Obtenga información sobre cómo App Service ejecuta una aplicación de Python:

> [!div class="nextstepaction"]
> [Configuración de una aplicación de Python](../../app-service/configure-language-python.md)
