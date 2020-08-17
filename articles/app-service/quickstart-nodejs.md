---
title: 'Inicio rápido: Crear una aplicación web de Node.js'
description: Implementación de su primera aplicación Hola mundo de Node.js en Azure App Service en cuestión de minutos.
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
ms.date: 08/01/2020
ms.custom: mvc, devcenter, seodec18
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: 4ce14b9d467e4badc833a1e1e4d8e1d08d232b1a
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88084339"
---
# <a name="create-a-nodejs-web-app-in-azure"></a>Creación de una aplicación web de Node.js en Azure

::: zone pivot="platform-windows"  

Para empezar a trabajar con Azure App Service, cree una aplicación de Node.js o Express localmente con Visual Studio Code y, a continuación, implemente la aplicación en la nube. Dado que usa un nivel de App Service gratuito, completar este inicio rápido no supone ningún costo.

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-app-service-extension&mktingSource=vscode-tutorial-app-service-extension).
- [Node.js y npm](https://nodejs.org) Ejecute el comando `node --version` para comprobar que Node.js está instalado.
- [Visual Studio Code](https://code.visualstudio.com/).
- La [extensión Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice) para Visual Studio Code.

## <a name="clone-and-run-a-local-nodejs-application"></a>Clonación y ejecución de una aplicación de Node.js local

1. En el equipo local, abra un terminal y clone el repositorio de ejemplo:

    ```bash
    git clone https://github.com/Azure-Samples/nodejs-docs-hello-world
    ```

1. Vaya a la carpeta de la nueva aplicación:

    ```bash
    cd nodejs-docs-hello-world
    ```

1. Inicie la aplicación para probarla localmente:

    ```bash
    npm start
    ```
    
1. Abra el explorador web y vaya a `http://localhost:1337`. El explorador debe mostrar "¡Hola mundo!".

1. Presione **Ctrl**+**C** en el terminal para detener el servidor.

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=create-app)

## <a name="deploy-the-app-to-azure"></a>Implementar la aplicación en Azure

En esta sección, implementará la aplicación de Node.js en Azure con VS Code y la extensión Azure App Service.

1. En el terminal, asegúrese de que se encuentra en la carpeta *nodejs-docs-hello-world* y, a continuación, inicie Visual Studio Code con el siguiente comando:

    ```bash
    code .
    ```

1. En la barra de actividades de VS Code, seleccione el logotipo de Azure para mostrar el explorador de **AZURE APP SERVICE**. Seleccione **Iniciar sesión en Azure…** y siga las instrucciones. Si se producen errores, consulte [Solución de problemas de inicio de sesión en Azure](#troubleshooting-azure-sign-in) a continuación. Una vez que haya iniciado sesión, el explorador debe mostrar el nombre de la suscripción de Azure.

    ![Inicio de sesión en Azure](media/quickstart-nodejs/sign-in.png)

1. En el explorador de **AZURE APP SERVICE** de VS Code, seleccione el icono de flecha arriba azul para implementar la aplicación en Azure. También puede invocar el mismo comando desde la **Paleta de comandos** (**Ctrl**+**Mayús**+**P**); para hacerlo, escriba "deploy to web app" y elija **Azure App Service: Deploy to Web App** (Azure App Service: Implementar en aplicación web).

    ![Implementación en Web App](media/quickstart-nodejs/deploy.png)
        
1. Elija la carpeta *nodejs-docs-hello-world*.

1. Elija una opción de creación basada en el sistema operativo en el que desea realizar la implementación:

    - Linux: seleccione la opción **Create new Web App** (Crear nueva aplicación web).
    - Windows: seleccione la opción **Create new Web App... Advanced** (Crear nueva aplicación web...Avanzado).

1. Escriba un nombre único global para la aplicación web y presione **Intro**. El nombre debe ser único en todo Azure y usar solo caracteres alfanuméricos ("A-Z", "a-z" y "0-9") y guiones ("-").

1. Si el destino es Linux, seleccione una versión de Node.js cuando se le solicite. Se recomienda una versión **LTS**.

1. Si el destino es Windows, siga las indicaciones adicionales:
    1. Seleccione **Create a new resource group** (Crear un grupo de recursos nuevo) y escriba un nombre para el grupo de recursos, como `AppServiceQS-rg`.
    1. Seleccione **Windows** como sistema operativo.
    1. Seleccione **Create new App Service plan** (Crear nuevo plan de App Service) y, a continuación, escriba un nombre para el plan (por ejemplo, `AppServiceQS-plan`); después, seleccione **F1 Free** (F1 gratis) como plan de tarifa.
    1. Elija **Omitir por ahora** cuando se le pregunte sobre Application Insights.
    1. Elija una región cerca de usted o de los recursos a los que desea acceder.

1. Tras responder a todos los mensajes, VS Code muestra los recursos de Azure que se estén creando para la aplicación en la ventana emergente de notificación.

    Al realizar la implementación en Linux, seleccione **Yes** (Sí) cuando se le pida que actualice la configuración para ejecutar `npm install` en el servidor Linux de destino.

    ![Solicitud de actualización de la configuración en el servidor Linux de destino](media/quickstart-nodejs/server-build.png)

1. Seleccione **Yes** (Sí) cuando aparezca el mensaje **Always deploy the workspace "nodejs-docs-hello-world" to (app name)** (Implementar siempre el área de trabajo "nodejs-docs-hello-world" en [nombre de la aplicación]). Al seleccionar **Yes** (Sí) se indica a VS Code que el destino sea automáticamente la misma aplicación web de App Service en las implementaciones posteriores.

1. Si implementa en Linux, seleccione **Browse Website** (Examinar sitio web) en el mensaje para ver la aplicación web recién implementada una vez finalizado el proceso. El explorador debe mostrar "¡Hola mundo!"

1. Si implementa en Windows, primero debe establecer el número de versión de Node.js para la aplicación web:

    1. En VS Code, expanda el nodo del nuevo servicio de aplicaciones, haga clic con el botón derecho en **Application Settings** (Configuración de la aplicación) y seleccione **Add New Setting...** (Agregar nueva configuración):

        ![Comando para agregar configuración de aplicación](media/quickstart-nodejs/add-setting.png)

    1. Escriba `WEBSITE_NODE_DEFAULT_VERSION` como clave de configuración.
    1. Escriba `10.15.2` como valor de configuración.
    1. Haga clic con el botón derecho en el nodo del servicio de aplicaciones y seleccione **Restart** (Reiniciar).

        ![Comando para reiniciar el servicio de aplicaciones](media/quickstart-nodejs/restart.png)

    1. Haga clic con el botón derecho en el nodo del servicio de aplicaciones una vez más y seleccione **Browse Website** (Examinar sitio web).

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=deploy-app)

### <a name="troubleshooting-azure-sign-in"></a>Solución de problemas de inicio de sesión en Azure

Si ve el error **"Cannot find subscription with name [subscription ID]"** ("No se encuentra la suscripción con el nombre [identificador de suscripción]"), puede que sea debido a que esté detrás de un proxy y no se pueda establecer la conexión con la API de Azure. Configure las variables de entorno `HTTP_PROXY` y `HTTPS_PROXY` con su información de proxy en el terminal mediante `export`.

```bash
export HTTPS_PROXY=https://username:password@proxy:8080
export HTTP_PROXY=http://username:password@proxy:8080
```

Si el problema no se corrige después de configurar las variables de entorno, seleccione el botón anterior **He tenido un problema** para ponerse en contacto con nosotros.

### <a name="update-the-app"></a>Actualización de la aplicación

Puede implementar los cambios en esta aplicación si realiza modificaciones en VS Code, guarda los archivos y, a continuación, usa el mismo proceso que antes, solo que esta vez eligiendo la aplicación existente en lugar de crear una nueva.

## <a name="viewing-logs"></a>Visualización de los registros

Puede ver la salida del registro (llamadas a `console.log`) desde la aplicación directamente en la ventana de salida de VS Code.

1. En el explorador de **AZURE APP SERVICE**, haga clic con el botón derecho en el nodo de la aplicación y elija **Start Streaming Logs** (Iniciar transmisión de registros).

    ![Inicio de transmisión de registros](media/quickstart-nodejs/view-logs.png)

1. Cuando se le pida, seleccione la opción para habilitar el registro y reinicie la aplicación. Después de reiniciar la aplicación, se abrirá la ventana de salida de VS Code con una conexión al streaming de registros. 

    ![Activación de los registros y reinicio](media/quickstart-nodejs/enable-restart.png)

1. Después de unos segundos, la ventana de salida muestra un mensaje donde se indica que está conectado al servicio de transmisión de registros. Puede generar más actividad de salida actualizando la página en el explorador.

    <pre>
    Connecting to log stream...
    2020-03-04T19:29:44  Welcome, you are now connected to log-streaming service. The default timeout is 2 hours.
    Change the timeout with the App Setting SCM_LOGSTREAM_TIMEOUT (in seconds).
    </pre>

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=tailing-logs)

## <a name="next-steps"></a>Pasos siguientes

Ha completado correctamente este inicio rápido.

> [!div class="nextstepaction"]
> [Tutorial: Aplicación Node.js con MongoDB](tutorial-nodejs-mongodb-app.md)

> [!div class="nextstepaction"]
> [Configuración de la aplicación Node.js](configure-language-nodejs.md)

Compruebe las demás extensiones de Azure.

* [Cosmos DB](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Funciones de Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Herramientas de Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
* [Herramientas de la CLI de Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

También puede conseguirlas todas si instala el paquete de extensiones [Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).
::: zone-end

::: zone pivot="platform-linux"  
## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una cuenta de Azure, [regístrese hoy mismo](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-app-service-extension&mktingSource=vscode-tutorial-app-service-extension) para obtener una cuenta gratuita con 200 USD en créditos de Azure para probar cualquier combinación de los servicios.

Necesita tener instalado [Visual Studio Code](https://code.visualstudio.com/), además de [Node.js y npm](https://nodejs.org/en/download), el administrador de paquetes de Node.js.

También tendrá que instalar la [extensión Azure App Service](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice), que puede usar para crear, administrar e implementar Azure App Service en Linux en la plataforma como servicio de Azure (PaaS).

### <a name="sign-in"></a>Iniciar sesión

Después de instalar la extensión, inicie sesión en su cuenta de Azure. En la barra de actividades, seleccione el logotipo de Azure para mostrar el explorador de **AZURE APP SERVICE**. Seleccione **Iniciar sesión en Azure…** y siga las instrucciones.

![inicio de sesión en Azure](./media/quickstart-nodejs/sign-in.png)

### <a name="troubleshooting"></a>Solución de problemas

Si ve el error **“No se encuentra la suscripción con el nombre [identificador de suscripción]”** , puede que sea debido a que esté detrás de un proxy y no se pueda establecer la conexión con la API de Azure. Configure las variables de entorno `HTTP_PROXY` y `HTTPS_PROXY` con su información de proxy en el terminal mediante `export`.

```sh
export HTTPS_PROXY=https://username:password@proxy:8080
export HTTP_PROXY=http://username:password@proxy:8080
```

Si el problema no se corrige después de configurar las variables de entorno, seleccione el botón siguiente **He tenido un problema** para ponerse en contacto con nosotros.

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

Antes de continuar, asegúrese de haber instalado y configurado todos los requisitos previos.

En VS Code, verá su dirección de correo electrónico de Azure en la barra de estado, y la suscripción en el explorador de **AZURE APP SERVICE**.

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=getting-started)

## <a name="create-your-nodejs-application"></a>Creación de una aplicación de Node.js

Después, cree una aplicación de Node.js que se pueda implementar en la nube. En este inicio rápido, se usa un generador de aplicaciones para aplicar scaffolding rápidamente en la aplicación desde un terminal.

> [!TIP]
> Si ya ha completado el [tutorial de Node.js](https://code.visualstudio.com/docs/nodejs/nodejs-tutorial), puede ir directamente al paso [Implementación en Azure](#deploy-to-azure).

### <a name="scaffold-a-new-application-with-the-express-generator"></a>Aplicación de scaffolding en una nueva aplicación con Express Generator

[Express](https://www.expressjs.com) es un marco popular para crear y ejecutar aplicaciones de Node.js. Puede aplicar scaffolding en una nueva aplicación de Express (lo que equivale a crear un aplicación) mediante la herramienta [Express Generator](https://expressjs.com/en/starter/generator.html). Express Generator se proporciona como módulo de npm y se puede ejecutar directamente (sin instalación) mediante la herramienta de línea de comandos de npm `npx`.

```bash
npx express-generator myExpressApp --view pug --git
```

Los parámetros `--view pug --git` indican al generador que use el motor de plantillas [pug](https://pugjs.org/api/getting-started.html) (antes conocido como `jade`) y que cree un archivo de `.gitignore`.

Para instalar todas las dependencias de la aplicación, vaya a la nueva carpeta y ejecute `npm install`.

```bash
cd myExpressApp
npm install
```

### <a name="run-the-application"></a>Ejecución de la aplicación

A continuación, asegúrese de que se ejecute la aplicación. Desde el terminal, inicie la aplicación con el comando `npm start` para iniciar el servidor.

```bash
npm start
```

Ahora, abra el explorador y vaya a `http://localhost:3000`, donde verá algo similar a esto:

![Ejecución de una aplicación de Express](./media/quickstart-nodejs/express.png)

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=create-app)

## <a name="deploy-to-azure"></a>Implementar en Azure

En esta sección, implementará la aplicación de Node.js con VS Code y la extensión Azure App Service. En este inicio rápido, se usa el modelo de implementación más básico, donde la aplicación se comprime e implementa en una instancia de Azure App Service en Linux.

### <a name="deploy-using-azure-app-service"></a>Implementación mediante Azure App Service

Primero, abra la carpeta de la aplicación en VS Code.

```bash
code .
```

En el explorador de **AZURE APP SERVICE**, seleccione el icono de flecha arriba azul para implementar la aplicación en Azure.

![Implementación en Web App](./media/quickstart-nodejs/deploy.png)

> [!TIP]
> También puede implementar desde la **Paleta de comandos** (CTRL + MAYÚS + P); para hacerlo, escriba “implementar en Web App” y ejecute el comando **Azure App Service: implementar en Web App**.

1. Seleccione el directorio que tenga abierto actualmente, `myExpressApp`.

1. Elija **Create new Web App** (Crear aplicación web), que se implementa en App Service en Linux de forma predeterminada.

1. Escriba un nombre único global para la Web App y presione ENTRAR. Los caracteres válidos para un nombre de aplicación son “a-z”, “0-9” y “-”.

1. Seleccione la **versión de Node.js** (le recomendamos que use “LTS”).

    En el canal de notificación, se muestran los recursos de Azure que se estén creando para la aplicación.

1. Seleccione **Sí** cuando se le pida que actualice la configuración para ejecutar `npm install` en el servidor de destino. Después, se implementará la aplicación.

    ![Implementación configurada](./media/quickstart-nodejs/server-build.png)

1. Cuando se inicie la aplicación, se le pedirá que actualice el área de trabajo para que las implementaciones posteriores usen como destino automáticamente la misma instancia de App Service Web App. Seleccione **Sí** para asegurarse de que los cambios se implementen en la aplicación correcta.

    ![Implementación configurada](./media/quickstart-nodejs/save-configuration.png)

> [!TIP]
> Asegúrese de que la aplicación escuche en el puerto proporcionado por la variable de entorno PORT: `process.env.PORT`.

### <a name="browse-the-app-in-azure"></a>Examen de la aplicación en Azure

Una vez finalizada la implementación, seleccione **Browse Website** (Examinar sitio web) en el mensaje para ver la aplicación web recién implementada.

### <a name="troubleshooting"></a>Solución de problemas

Si ve el error **“No tiene permiso para ver este directorio o esta página”** , puede que la aplicación no se haya iniciado correctamente. Vaya a la sección siguiente y vea la salida de registros para identificar y corregir el error. Si no puede corregirlo, seleccione el botón siguiente **He tenido un problema** para ponerse en contacto con nosotros. Estaremos encantados de ayudarle.

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=deploy-app)

### <a name="update-the-app"></a>Actualización de la aplicación

Para implementar los cambios en esta aplicación, siga el mismo proceso y seleccione la aplicación existente, en lugar de crear otra.

## <a name="viewing-logs"></a>Visualización de los registros

En esta sección aprenderá a ver (o "copiar") los registros desde la aplicación de App Service en ejecución. Las llamadas realizadas a `console.log` en la aplicación se muestran en la ventana de salida en Visual Studio Code.

Busque la aplicación en el explorador de **AZURE APP SERVICE**, haga clic con el botón derecho en la aplicación y seleccione **Ver registros de streaming**.

La ventana de salida de VS Code se abre con una conexión a la transmisión de registros.

![Visualización de los registros de streaming](./media/quickstart-nodejs/view-logs.png)

![Activación de los registros y reinicio](./media/quickstart-nodejs/enable-restart.png)

Después de unos segundos, verá un mensaje donde se indica que está conectado al servicio de streaming de registros. Actualice la página varias veces para ver más actividad.

<pre>
2019-09-20 20:37:39.574 INFO  - Initiating warmup request to container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node
2019-09-20 20:37:55.011 INFO  - Waiting for response to warmup request for container msdocs-vscode-node_2_00ac292a. Elapsed time = 15.4373071 sec
2019-09-20 20:38:08.233 INFO  - Container msdocs-vscode-node_2_00ac292a for site msdocs-vscode-node initialized successfully and is ready to serve requests.
2019-09-20T20:38:21  Startup Request, url: /Default.cshtml, method: GET, type: request, pid: 61,1,7, SCM_SKIP_SSL_VALIDATION: 0, SCM_BIN_PATH: /opt/Kudu/bin, ScmType: None
</pre>

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/PWZWZ52?tutorial=node-deployment-azure-app-service&step=tailing-logs)

## <a name="next-steps"></a>Pasos siguientes

Ha completado correctamente este inicio rápido.

A continuación, vea otras extensiones de Azure.

* [Cosmos DB](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Funciones de Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Herramientas de Docker](https://marketplace.visualstudio.com/items?itemName=PeterJausovec.vscode-docker)
* [Herramientas de la CLI de Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

También puede conseguirlas todas si instala el paquete de extensiones [Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).


::: zone-end
