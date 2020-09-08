---
title: 'Tutorial: Implementación de una aplicación de Django de Python con Postgres'
description: Cree una aplicación web de Python con una base de datos de PostgreSQL e impleméntela en Azure. En el tutorial se usa el marco de Django, y la aplicación se hospeda en Azure App Service en Linux.
ms.devlang: python
ms.topic: tutorial
ms.date: 07/22/2020
ms.custom:
- mvc
- seodec18
- seo-python-october2019
- cli-validate
- devx-track-python
- devx-track-azurecli
ms.openlocfilehash: 368a87d1054e4a5ad12fa1e8c78bcde39f76ee63
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88959415"
---
# <a name="tutorial-deploy-a-django-web-app-with-postgresql-in-azure-app-service"></a>Tutorial: Implementación de una aplicación web de Django de Python con PostgreSQL en Azure App Service

En este tutorial se muestra cómo implementar una aplicación web de [Django](https://www.djangoproject.com/) de Python en [Azure App Service](overview.md) y cómo conectarla a una base de datos de Azure Database for Postgres. App Service proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático.

En este tutorial, se usa la CLI de Azure para completar las siguientes tareas:

> [!div class="checklist"]
> * Configurar el entorno inicial con Python y la CLI de Azure
> * Crear una base de datos de Azure Database for PostgreSQL
> * Implementar código en Azure App Service y conectarse a Postgres
> * Actualizar el código y volver a implementarlo
> * Visualización de los registros de diagnóstico
> * Administrar la aplicación web en Azure Portal

También puede usar la [versión de Azure Portal de este tutorial](/azure/developer/python/tutorial-python-postgresql-app-portal).


## <a name="set-up-your-initial-environment"></a>Configuración del entorno inicial

1. Disponga de una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
1. Instale <a href="https://www.python.org/downloads/" target="_blank">Python 3.6 o una versión posterior</a>.
1. Instale la <a href="/cli/azure/install-azure-cli" target="_blank">CLI de Azure</a> 2.0.80 o posterior, con la que se ejecutan comandos en cualquier shell para aprovisionar y configurar los recursos de Azure.

Abra una ventana del terminal y compruebe que la versión de Python es la 3.6 o superior:

# <a name="bash"></a>[Bash](#tab/bash)

```bash
python3 --version
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```cmd
py -3 --version
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
py -3 --version
```

---

Compruebe que la versión de la CLI de Azure es la 2.0.80 o posterior:

```azurecli
az --version
```

Después, inicie sesión en Azure mediante la CLI:

```azurecli
az login
```

Este comando abre un explorador que recopila las credenciales. Cuando el comando finaliza, muestra una salida JSON que contiene información sobre las suscripciones.

Una vez que haya iniciado sesión, puede ejecutar comandos de Azure con la CLI de Azure para trabajar con los recursos de su suscripción.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

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

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

## <a name="create-postgres-database-in-azure"></a>Creación de una base de datos de Postgres en Azure

<!-- > [!NOTE]
> Before you create an Azure Database for PostgreSQL server, check which [compute generation](../postgresql/concepts-pricing-tiers.md#compute-generations-and-vcores) is available in your region. -->

Instale la extensión `db-up` para la CLI de Azure:

```azurecli
az extension add --name db-up
```

Si no se reconoce el comando `az`, asegúrese de que tiene instalada la CLI de Azure tal y como se describe en [Configuración del entorno inicial](#set-up-your-initial-environment).

A continuación, cree la base de datos de Postgres en Azure con el comando [`az postgres up`](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up):

```azurecli
az postgres up --resource-group DjangoPostgres-tutorial-rg --location westus2 --sku-name B_Gen5_1 --server-name <postgre-server-name> --database-name pollsdb --admin-user <admin-username> --admin-password <admin-password> --ssl-enforcement Enabled
```

- Reemplace *\<postgres-server-name>* por un nombre que sea exclusivo en todo Azure (el punto de conexión del servidor es `https://\<postgres-server-name>.postgres.database.azure.com`). Un buen patrón es usar una combinación del nombre de la empresa y otro valor exclusivo.
- Para *\<admin-username>* y *\<admin-password>* , especifique las credenciales para crear un usuario administrador de este servidor Postgres.
- El [plan de tarifa](../postgresql/concepts-pricing-tiers.md) B_Gen5_1 (Básico, Gen5 y 1 núcleo) que se usa aquí es el menos costoso. En el caso de las bases de datos de producción, omita el argumento `--sku-name` para usar el nivel GP_Gen5_2 (De uso general, gen. 5, 2 núcleos) en su lugar.

Este comando realiza las siguientes acciones, que pueden tardar algunos minutos:

- Crea un [grupo de recursos](../azure-resource-manager/management/overview.md#terminology) denominado `DjangoPostgres-tutorial-rg`, si aún no existe.
- Crea un servidor de Postgres.
- Crea una cuenta de administrador predeterminada con un nombre de usuario y una contraseña exclusivos. (Para especificar sus propias credenciales, use los argumentos `--admin-user` y `--admin-password` con el comando `az postgres up`).
- Crea una base de datos `pollsdb`.
- Habilita el acceso desde la dirección IP local.
- Habilita el acceso desde servicios de Azure.
- Crea un usuario de base de datos con acceso a la base de datos `pollsdb`.

Puede realizar todos los pasos por separado con otros comandos `az postgres` y `psql`, pero `az postgres up` realiza todos los pasos juntos.

Cuando el comando se completa, genera un objeto JSON que contiene cadenas de conexión diferentes para la base de datos, junto con la dirección URL del servidor, un nombre de usuario generado (como "joyfulKoala@msdocs-djangodb-12345") y una contraseña de GUID. Copie el nombre de usuario y la contraseña en un archivo de texto temporal, ya que los necesitará más adelante en este tutorial.

<!-- not all locations support az postgres up -->
> [!TIP]
> `-l <location-name>`, se puede establecer en cualquiera de las [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/). Puede obtener las regiones disponibles para su suscripción con el comando [`az account list-locations`](/cli/azure/account#az-account-list-locations). En el caso de las aplicaciones de producción, coloque la base de datos y la aplicación en la misma ubicación.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

## <a name="deploy-the-code-to-azure-app-service"></a>Implementación del código en Azure App Service

En esta sección, creará un host de aplicación en la aplicación de App Service, conectará esta aplicación a la base de datos de Postgres y, a continuación, implementará el código en ese host.

### <a name="create-the-app-service-app"></a>Creación de la aplicación de App Service

En el terminal, asegúrese de que se encuentra en la raíz del repositorio (`djangoapp`) que contiene el código de la aplicación.

Cree una aplicación de App Service (el proceso de host) con el comando [`az webapp up`](/cli/azure/webapp#az-webapp-up):

```azurecli
az webapp up --resource-group DjangoPostgres-tutorial-rg --location westus2 --plan DjangoPostgres-tutorial-plan --sku B1 --name <app-name>
```
<!-- without --sku creates PremiumV2 plan -->

- Para el argumento `--location`, use la misma ubicación que usó para la base de datos en la sección anterior.
- Reemplace *\<app-name>* por un nombre exclusivo en todo Azure (el punto de conexión del servidor es `https://\<app-name>.azurewebsites.net`). Los caracteres permitidos para *\<app-name>* son `A`-`Z`, `0`-`9` y `-`. Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.

Este comando realiza las siguientes acciones, que pueden tardar algunos minutos:

<!-- - Create the resource group if it doesn't exist. `--resource-group` is optional. -->
<!-- No it doesn't. az webapp up doesn't respect --resource-group -->
- Cree el [grupo de recursos](../azure-resource-manager/management/overview.md#terminology) si todavía no existe. (En este comando se usa el mismo grupo de recursos en el que se creó la base de datos anterior).
- Cree el [plan de App Service](overview-hosting-plans.md) *DjangoPostgres-tutorial-plan* en el plan de tarifa Básico (B1), si no existe. `--plan` y `--sku` son opcionales.
- Cree la aplicación de App Service, si no existe.
- Habilite el registro predeterminado de la aplicación, si aún no está habilitado.
- Cargue el repositorio mediante la implementación del archivo ZIP con la automatización de compilación habilitada.

Tras una implementación correcta, el comando genera la salida JSON como en el ejemplo siguiente:

![Ejemplo de la salida del comando az webapp up](./media/tutorial-python-postgresql-app/az-webapp-up-output.png)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

> [!TIP]
> Muchos comandos de la CLI de Azure almacenan en caché los parámetros comunes, como el nombre del grupo de recursos y el plan de App Service, en el archivo *.azure/config*. Como consecuencia, no es necesario especificar el mismo parámetro con comandos posteriores. Por ejemplo, para volver a implementar la aplicación después de realizar cambios, puede ejecutar `az webapp up` de nuevo sin ningún parámetro. Sin embargo, los comandos que proceden de las extensiones de la CLI, como `az postgres up`, no usan la memoria caché en este momento, por lo que es necesario especificar el grupo de recursos y la ubicación aquí con `az webapp up`.

> [!NOTE]
> Si intenta visitar la dirección URL de la aplicación en este momento, se produce el error "DisallowedHost at /". Este error se produce porque todavía no ha configurado la aplicación para usar la configuración de producción descrita anteriormente, lo que se hace en la sección siguiente.

### <a name="configure-environment-variables-to-connect-the-database"></a>Configuración de las variables de entorno para conectarse a la base de datos

Con el código ahora implementado en App Service, el paso siguiente consiste en conectar la aplicación a la base de datos de Postgres en Azure.

El código de la aplicación espera encontrar información de la base de datos en una serie de variables de entorno. Para establecer variables de entorno en App Service, cree "valores de aplicación" mediante el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az-webapp-config-appsettings-set).

```azurecli
az webapp config appsettings set --settings DJANGO_ENV="production" DBHOST="<postgres-server-name>.postgres.database.azure.com" DBNAME="pollsdb" DBUSER="<username>" DBPASS="<password>"
```

- Reemplace *\<postgres-server-name>* por el nombre que usó anteriormente con el comando `az postgres up`.
- Reemplace *\<username>* y *\<password>* por las credenciales que el comando también ha generado automáticamente.
- El grupo de recursos y el nombre de la aplicación se extraen de los valores almacenados en caché del archivo *.azure/config*.
- El comando crea valores denominados `DJANGO_ENV`, `DBHOST`, `DBNAME`, `DBUSER` y `DBPASS`, según lo esperado por el código de la aplicación.
- En el código de Python, tiene acceso a esta configuración como variables de entorno con instrucciones como `os.environ.get('DJANGO_ENV')`. Para obtener más información, consulte [Acceso a variables de entorno](configure-language-python.md#access-environment-variables).

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

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
    
1. El comando `createsuperuser` le pide las credenciales de superusuario. A los efectos de este tutorial, use el nombre de usuario predeterminado `root`, presione **ENTRAR** en la dirección de correo electrónico para que quede en blanco y escriba `Pollsdb1` como contraseña.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)
    
### <a name="create-a-poll-question-in-the-app"></a>Creación de una pregunta de sondeo en la aplicación

1. En un explorador, abra la dirección URL *http:\//\<app-name>.azurewebsites.net*. La aplicación debe mostrar el mensaje "No polls are available" (No hay sondeos disponibles) porque todavía no hay sondeos específicos en la base de datos.

1. Vaya a *http:\//\<app-name>.azurewebsites.net/admin*. Inicie sesión con las credenciales de superusuario de la sección anterior (`root` y `Pollsdb1`). En **Sondeos**, seleccione **Agregar** junto a **Preguntas** y cree una pregunta de sondeo con opciones.

1. Navegue nuevamente a *http:\//\<app-name>.azurewebsites.net/* para confirmar que las preguntas ahora se presenten al usuario. Responda a las preguntas como desee para generar algunos datos en la base de datos.

**¡Enhorabuena!** Está ejecutando una aplicación web de Django de Python en Azure App Service para Linux, con una base de datos de Postgres activa.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

> [!NOTE]
> App Service detecta un proyecto de Django mediante la búsqueda de un archivo *wsgi.py* en cada subcarpeta, que `manage.py startproject` crea de forma predeterminada. Cuando App Service encuentra ese archivo, carga la aplicación web de Django. Para obtener más información, consulte [Configuración de una imagen de Python integrada](configure-language-python.md).

## <a name="make-code-changes-and-redeploy"></a>Cambios en el código y reimplementación

En esta sección, realizará cambios locales en la aplicación y volverá a implementar el código en App Service. En el proceso, se configura un entorno virtual de Python que admite el trabajo en curso.

### <a name="run-the-app-locally"></a>Ejecución de la aplicación de forma local

Ejecute los siguientes comandos en una ventana de terminal. Asegúrese de seguir las indicaciones al crear el superusuario:

# <a name="bash"></a>[bash](#tab/bash)

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

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
# Configure the Python virtual environment
py -3 -m venv venv
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force
venv\scripts\activate

# Install packages
pip install -r requirements.txt
# Run Django migrations
python manage.py migrate
# Create Django superuser (follow prompts)
python manage.py createsuperuser
# Run the dev server
python manage.py runserver
```

# <a name="cmd"></a>[CMD](#tab/cmd)

```cmd
:: Configure the Python virtual environment
py -3 -m venv venv
venv\scripts\activate

:: Install packages
pip install -r requirements.txt
:: Run Django migrations
python manage.py migrate
:: Create Django superuser (follow prompts)
python manage.py createsuperuser
:: Run the dev server
python manage.py runserver
```
---

Una vez que la aplicación web esté completamente cargada, el servidor de desarrollo de Django proporcionará la dirección URL de la aplicación local en el mensaje "Starting development server at http://127.0.0.1:8000/. Quit the server with CTRL-BREAK" (Iniciando el servidor de desarrollo en https… Salga del servidor con Ctrl+Interrumpir).

![Ejemplo de salida del servidor de desarrollo de Django](./media/tutorial-python-postgresql-app/django-dev-server-output.png)

Pruebe la aplicación localmente con los pasos siguientes:

1. Vaya a *http:\//localhost:8000* en un explorador, que debería mostrar el mensaje No polls are available (No hay sondeos disponibles). 

1. Vaya a *http:\//localhost:8000/admin* e inicie sesión con el usuario administrador que creó anteriormente. En **Sondeos**, vuelva a seleccionar **Agregar** junto a **Preguntas** y cree una pregunta de sondeo con opciones. 

1. Vaya a *http:\//localhost: 8000* de nuevo y responda la pregunta para probar la aplicación. 

1. Detenga el servidor de Django presionando **CTRL**+**C**.

Cuando se ejecute localmente, la aplicación usará una base de datos de Sqlite3 local y no interferirá con la base de datos de producción. También puede usar una base de datos de PostgreSQL local, si lo desea, para simular mejor el entorno de producción.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

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

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

### <a name="redeploy-the-code-to-azure"></a>Nueva implementación del código en Azure

Ejecute el comando siguiente en la raíz del repositorio:

```azurecli
az webapp up
```

Este comando usa los parámetros almacenados en la memoria caché del archivo *.azure/config*. Dado que App Service detecta que la aplicación ya existe, simplemente vuelve a implementar el código.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

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

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

### <a name="review-app-in-production"></a>Revisión de la aplicación en producción

Vaya a *http:\//\<app-name>.azurewebsites.net* y vuelva a probar la aplicación en producción. (Dado que solo ha cambiado la longitud de un campo de la base de datos, el cambio solo se aprecia si intenta especificar una respuesta más larga al crear una pregunta).

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

## <a name="stream-diagnostic-logs"></a>Transmisión de registros de diagnóstico

Puede acceder a los registros de la consola generados desde dentro del contenedor que hospeda la aplicación en Azure.

Ejecute el siguiente comando de la CLI de Azure para ver la secuencia de registro. Este comando usa los parámetros almacenados en la memoria caché del archivo *.azure/config*.

```azurecli
az webapp log tail
```

Si no ve los registros de la consola de inmediato, vuelve a comprobarlo en 30 segundos.

Para detener el streaming de registro en cualquier momento, escriba **Ctrl**+**C**.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

> [!NOTE]
> También puede inspeccionar los archivos de registro desde el explorador en `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.
>
> `az webapp up` activa el registro predeterminado. Por motivos de rendimiento, este registro se desactiva después de un tiempo, pero vuelve a activarse cada vez que se vuelve a ejecutar `az webapp up`. Para activarlo manualmente, ejecute el siguiente comando:
>
> ```azurecli
> az webapp log config --docker-container-logging filesystem
> ```

## <a name="manage-your-app-in-the-azure-portal"></a>Administración de una aplicación en Azure Portal

En [Azure Portal](https://portal.azure.com), busque el nombre de la aplicación y seleccione la aplicación en los resultados.

![Vaya a la aplicación Python Django en Azure Portal](./media/tutorial-python-postgresql-app/navigate-to-django-app-in-app-services-in-the-azure-portal.png)

De forma predeterminada, el portal muestra la página **Información general** de la aplicación, que proporciona una vista del rendimiento general. En ella puede también realizar tareas de administración básicas como examinar, detener, reiniciar y eliminar. Las pestañas del lado izquierdo de la página muestran las diferentes páginas de configuración que puede abrir.

![Administración de la aplicación Python Django en la página de información general de Azure Portal](./media/tutorial-python-postgresql-app/manage-django-app-in-app-services-in-the-azure-portal.png)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea mantener la aplicación o continuar con el siguiente tutorial, vaya a [Pasos siguientes](#next-steps). De lo contrario, para evitar incurrir en cargos adicionales, puede eliminar la creación del grupo de recursos para este tutorial:

```azurecli
az group delete
```

Este comando usa el nombre del grupo de recursos almacenado en la memoria caché del archivo *.azure/config*. Al eliminar el grupo de recursos, también se desasignan y eliminan todos los recursos que contiene.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/DjangoCLITutorialHelp)

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo asignar un nombre DNS personalizado a la aplicación:

> [!div class="nextstepaction"]
> [Tutorial: Asignar un nombre DNS personalizado a la aplicación](app-service-web-tutorial-custom-domain.md)

Obtenga información sobre cómo App Service ejecuta una aplicación de Python:

> [!div class="nextstepaction"]
> [Configuración de una aplicación de Python](configure-language-python.md)