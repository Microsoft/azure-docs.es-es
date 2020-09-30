---
title: 'Inicio rápido: Creación de una aplicación de Python'
description: Para empezar a trabajar con Azure App Service, implemente su primera aplicación Python en un contenedor Linux en App Service.
ms.topic: quickstart
ms.date: 09/08/2020
ms.custom: seo-python-october2019, cli-validate, devx-track-python
ms.openlocfilehash: c64971303a2fefb25a73f0a8e8900c14ef05486c
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90603618"
---
# <a name="quickstart-create-a-python-app-in-azure-app-service-on-linux"></a>Inicio rápido: Creación de una aplicación de Python en Azure App Service en Linux

En este inicio rápido se implementa una aplicación web de Python en [App Service en Linux](overview.md#app-service-on-linux), un servicio de Azure de hospedaje de sitios web muy escalable y con aplicación automática de revisiones. Puede usar la [interfaz de la línea de comandos (CLI) de Azure](/cli/azure/install-azure-cli) local en un equipo Mac, Linux o Windows. La aplicación web que va a configurar usa un nivel de App Service gratuito, por lo que no incurrirá en ningún gasto en el transcurso de este artículo.

> [!TIP]
> Si prefiere implementar aplicaciones mediante un entorno de desarrollo integrado, consulte **[Implementación de aplicaciones de Python en App Service desde Visual Studio Code](/azure/developer/python/tutorial-deploy-app-service-on-linux-01)** .

## <a name="set-up-your-initial-environment"></a>Configuración del entorno inicial

Antes de empezar, debe disponer de lo siguiente:

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

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="clone-the-sample"></a>Clonación del ejemplo

Ejecute el comando siguiente para clonar el repositorio de ejemplo y vaya a la carpeta. ([Instale git](https://git-scm.com/downloads) si aún no lo tiene).

```terminal
git clone https://github.com/Azure-Samples/python-docs-hello-world
cd python-docs-hello-world
```

El código de ejemplo contiene el archivo *application.py*, que indica a App Service que el código contiene una aplicación de Flask. Para más información, consulte [Proceso de inicio del contenedor](configure-language-python.md#container-startup-process).

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="run-the-sample"></a>Ejecución del ejemplo

# <a name="bash"></a>[Bash](#tab/bash)

En primer lugar, cree un entorno virtual e instale las dependencias:

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

A continuación, establezca la variable de entorno `FLASK_APP` en el módulo de entrada de la aplicación y ejecute el servidor de desarrollo de Flask:

```
export FLASK_APP=application.py
flask run
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

En primer lugar, cree un entorno virtual e instale las dependencias:

```powershell
py -3 -m venv env
env\scripts\activate
pip install -r requirements.txt
```

A continuación, establezca la variable de entorno `FLASK_APP` en el módulo de entrada de la aplicación y ejecute el servidor de desarrollo de Flask:

```powershell
Set-Item Env:FLASK_APP ".\application.py"
flask run
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

En primer lugar, cree un entorno virtual e instale las dependencias:

```cmd
py -3 -m venv env
env\scripts\activate
pip install -r requirements.txt
```

A continuación, establezca la variable de entorno `FLASK_APP` en el módulo de entrada de la aplicación y ejecute el servidor de desarrollo de Flask:

```cmd
SET FLASK_APP=application.py
flask run
```

---

Abra un explorador web y vaya a la aplicación de ejemplo en `http://localhost:5000/`. La aplicación muestra el mensaje **Hola mundo**.

![Ejecución de una aplicación de Python de ejemplo en el entorno local](./media/quickstart-python/run-hello-world-sample-python-app-in-browser-localhost.png)

En la ventana de terminal, presione **Ctrl**+**C** para salir del servidor de desarrollo de Flask.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="deploy-the-sample"></a>Implementación del ejemplo

Implemente el código en la carpeta local (*python-docs-hello-world*) mediante el comando `az webapp up`:

```azurecli
az webapp up --sku F1 -n <app-name>
```

- Si no se reconoce el comando `az`, asegúrese de que tiene instalada la CLI de Azure tal y como se describe en [Configuración del entorno inicial](#set-up-your-initial-environment).
- Reemplace `<app_name>` por un nombre que sea único en todo Azure (*los caracteres válidos son `a-z`, `0-9` y `-`* ). Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.
- El argumento `--sku F1` crea la aplicación web en el plan de tarifa Gratuito. Omita este argumento para usar un nivel Premium más rápido, lo que supondrá un costo por hora.
- También tiene la opción de incluir el argumento `-l <location-name>`, donde `<location_name>` es una región de Azure, como **centralus**, **eastasia**, **westeurope**, **koreasouth**, **brazilsouth**, **centralindia**, etc. Puede recuperar una lista de las regiones permitidas para su cuenta de Azure mediante la ejecución del comando [`az account list-locations`](/cli/azure/appservice#az-appservice-list-locations).
- Si ve el error "No se pudo detectar automáticamente la pila en tiempo de ejecución de la aplicación", asegúrese de que ejecuta el comando en la carpeta *python-docs-hello-world* que contiene el archivo *requirements.txt*. (Consulte [Solución de problemas de detección automática con az webapp up](https://github.com/Azure/app-service-linux-docs/blob/master/AzWebAppUP/runtime_detection.md) (GitHub)).

El comando puede tardar varios minutos en completarse. Mientras se ejecuta, proporciona mensajes sobre cómo crear el grupo de recursos, el plan de App Service y la aplicación de hospedaje, configurar el registro y, a continuación, realizar la implementación del archivo ZIP. A continuación, se muestra el mensaje "Puede iniciar la aplicación en http://&lt;nombre_de_la_aplicación&gt;.azurewebsites.net", que es la dirección URL de la aplicación en Azure.

![Salida de ejemplo del comando az webapp up](./media/quickstart-python/az-webapp-up-output.png)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

[!INCLUDE [AZ Webapp Up Note](../../includes/app-service-web-az-webapp-up-note.md)]

## <a name="browse-to-the-app"></a>Navegación hasta la aplicación

Vaya a la aplicación implementada mediante el explorador web en la dirección URL `http://<app-name>.azurewebsites.net`.

El código de ejemplo de Python se ejecuta en un contenedor de Linux en App Service con una imagen integrada.

![Ejecución de una aplicación de Python de ejemplo en Azure](./media/quickstart-python/run-hello-world-sample-python-app-in-browser.png)

**¡Enhorabuena!** Ha implementado su primera aplicación Python en App Service en Linux.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="redeploy-updates"></a>Reimplementación de actualizaciones

En el editor de código que prefiera, abra *application.py* y actualice la función `hello` como se indica a continuación. Este cambio agrega una instrucción `print` para generar la salida de registro con la que trabajará en la sección siguiente. 

```python
def hello():
    print("Handling request to home page.")
    return "Hello Azure!"
```

Guarde los cambios y salga del editor. 

Vuelva a implementar la aplicación con el comando `az webapp up`:

```azurecli
az webapp up
```

Este comando utiliza valores que se almacenan en caché de forma local en el archivo *.azure/config*, incluidos el nombre de la aplicación, el grupo de recursos y el plan de App Service.

Una vez completada la implementación, vuelva a la ventana del explorador abierta en `http://<app-name>.azurewebsites.net`. Actualice la página, lo que debería mostrar el mensaje modificado:

![Ejecución de una aplicación de Python de ejemplo actualizada en Azure](./media/quickstart-python/run-updated-hello-world-sample-python-app-in-browser.png)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

> [!TIP]
> Visual Studio Code proporciona extensiones eficaces para Python y Azure App Service que simplifican el proceso de implementación de aplicaciones web de Python en App Service. Para más información, consulte [Deploy Python apps to App Service from Visual Studio Code](/azure/python/tutorial-deploy-app-service-on-linux-01) (Implementación de aplicaciones de Python en App Service desde Visual Studio Code).

## <a name="stream-logs"></a>Transmisión de registros

Puede acceder a los registros de consola generados desde dentro de la aplicación y del contenedor en que esta se ejecuta. Los registros incluyen todas las salidas generadas mediante instrucciones `print`.

Para transmitir registros, ejecute el siguiente comando:

```azurecli
az webapp log tail
```

Actualice la aplicación en el explorador para que se generen registros de la consola, que incluyen mensajes que describen las solicitudes HTTP a la aplicación. Si no aparece la salida inmediatamente, inténtelo de nuevo en 30 segundos.

También puede inspeccionar los archivos de registro desde el explorador en `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.

Para detener el streaming de registro en cualquier momento, escriba **Ctrl**+**C**.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="manage-the-azure-app"></a>Administración de la aplicación de Azure

Vaya a <a href="https://portal.azure.com" target="_blank">Azure Portal</a> para administrar la aplicación que ha creado. Busque y seleccione **App Services**.

![Ir a App Services en Azure Portal](./media/quickstart-python/navigate-to-app-services-in-the-azure-portal.png)

Seleccione el nombre de la aplicación de Azure.

![Aplicación de Python en App Services desde Azure Portal](./media/quickstart-python/navigate-to-app-in-app-services-in-the-azure-portal.png)

Al seleccionar la aplicación, se abre la página **Información general**, donde puede realizar tareas de administración básicas como examinar, detener, iniciar, reiniciar y eliminar.

![Administración de la aplicación de Python en la página de información general de Azure Portal](./media/quickstart-python/manage-an-app-in-app-services-in-the-azure-portal.png)

El menú de App Service proporciona distintas páginas para configurar la aplicación.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de recursos. El grupo de recursos tiene un nombre como "appsvc_rg_Linux_CentralUS", en función de su ubicación. Si usa una SKU de App Service que sea de un nivel distinto del F1 gratuito, estos recursos incurrirán en costos continuos (consulte [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/linux/)).

Si prevé que no necesitará estos recursos en el futuro, ejecute el siguiente comando para eliminar el grupo de recursos:

```azurecli
az group delete
```

Este comando usa el nombre del grupo de recursos almacenado en la memoria caché del archivo *.azure/config*.

Este comando puede tardar hasta un minuto en completarse.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación web Python (Django) con PostgreSQL](tutorial-python-postgresql-app.md)

> [!div class="nextstepaction"]
> [Incorporación del inicio de sesión de un usuario a una aplicación web de Python](../active-directory/develop/quickstart-v2-python-webapp.md)

> [!div class="nextstepaction"]
> [Configuración de una aplicación de Python](configure-language-python.md)

> [!div class="nextstepaction"]
> [Tutorial: Ejecución de la aplicación Python en un contenedor personalizado](tutorial-custom-container.md)
