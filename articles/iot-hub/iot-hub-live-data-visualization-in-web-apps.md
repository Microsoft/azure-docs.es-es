---
title: Visualización de datos IoT Hub en tiempo real en una aplicación web
description: Use una aplicación web para visualizar los datos de temperatura y de humedad que se recopilan desde el sensor y se le envían a IoT Hub.
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 05/31/2019
ms.author: robinsh
ms.custom:
- 'Role: Cloud Development'
- 'Role: Data Analytics'
ms.openlocfilehash: 6a8f39ae5d73bade2c86a7e15efe75956c2aed24
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87327572"
---
# <a name="visualize-real-time-sensor-data-from-your-azure-iot-hub-in-a-web-application"></a>Visualización de datos del sensor en tiempo real desde Azure IoT Hub en una aplicación web

![Diagrama integral](./media/iot-hub-live-data-visualization-in-web-apps/1_iot-hub-end-to-end-diagram.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

## <a name="what-you-learn"></a>Conocimientos que adquirirá

En este tutorial, obtendrá información sobre cómo visualizar los datos del sensor en tiempo real que recibe IoT Hub con una aplicación web node.js que se ejecuta en el equipo local. Después de ejecutar la aplicación web localmente, si lo desea, puede seguir los pasos para hospedar la aplicación web en Azure App Service. Si quiere intentar visualizar los datos en su IoT Hub con Power BI, vea [Visualización de datos del sensor en tiempo real desde Azure IoT Hub mediante Power BI](iot-hub-live-data-visualization-in-power-bi.md).

## <a name="what-you-do"></a>Qué debe hacer

* Agregar un grupo de consumidores a la instancia de IoT Hub que la aplicación web usará para leer datos del sensor
* Descargar el código de la aplicación web desde GitHub
* Examen del código de la aplicación web
* Configurar las variables de entorno para contener los artefactos de IoT Hub necesarios para la aplicación web
* Ejecutar la aplicación web en la máquina de desarrollo
* Abrir la aplicación web para ver los datos de temperatura y de humedad en tiempo real desde IoT Hub
* (Opcional) Usar la CLI de Azure para hospedar la aplicación web en Azure App Service

## <a name="what-you-need"></a>Lo que necesita

* Completar el tutorial [Conectar el simulador en línea de Raspberry Pi a Azure IoT Hub (Node.js)](iot-hub-raspberry-pi-web-simulator-get-started.md) o uno de los tutoriales del dispositivo, por ejemplo, [Conectar Raspberry Pi a Azure IoT Hub (Node.js)](iot-hub-raspberry-pi-kit-node-get-started.md). Estos tutoriales abarcan los requisitos siguientes:

  * Una suscripción de Azure activa.
  * Un IoT Hub en su suscripción.
  * Una aplicación cliente que envíe mensajes a su IoT Hub.

* [Descargar Git](https://www.git-scm.com/downloads)

* En los pasos de este artículo se supone que hay una máquina de desarrollo Windows; sin embargo, puede realizarlos fácilmente en un sistema Linux, en el shell que prefiera.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Ejecute el siguiente comando para agregar la extensión IoT de Microsoft Azure para la CLI de Azure a la instancia de Cloud Shell. La extensión IOT agrega comandos específicos de IoT Hub, IoT Edge e IoT Device Provisioning Service (DPS) a la CLI de Azure.

```azurecli-interactive
az extension add --name azure-iot
```

## <a name="add-a-consumer-group-to-your-iot-hub"></a>Adición de un grupo de consumidores a IoT Hub

Los [grupos de consumidores](https://docs.microsoft.com/azure/event-hubs/event-hubs-features#event-consumers) proporcionan vistas independientes en la secuencia de eventos que permiten a las aplicaciones y a los servicios de Azure consumir datos de forma independiente desde el mismo punto de conexión del centro de eventos. En esta sección, agregará un grupo de consumidores al punto de conexión integrado de IoT Hub que se usará posteriormente en este tutorial para leer los datos.

Ejecute el siguiente comando para agregar un grupo de consumidores al punto de conexión integrado de IoT Hub:

```azurecli-interactive
az iot hub consumer-group create --hub-name YourIoTHubName --name YourConsumerGroupName
```

Anote el nombre que elija, lo necesitará más adelante en este tutorial.

## <a name="get-a-service-connection-string-for-your-iot-hub"></a>Obtención de una cadena de conexión de servicio para la instancia de IoT Hub

Las instancias de IoT Hub se crean con diversas directivas de acceso predeterminadas. Una directiva de este tipo es la de **servicio**, que proporciona los permisos necesarios para que un servicio lea y escriba los puntos de conexión de IoT Hub. Ejecute el siguiente comando para obtener una cadena de conexión de IoT Hub que se adhiera a la directiva del servicio:

```azurecli-interactive
az iot hub show-connection-string --hub-name YourIotHub --policy-name service
```

La cadena de conexión debe tener un aspecto similar al siguiente:

```javascript
"HostName={YourIotHubName}.azure-devices.net;SharedAccessKeyName=service;SharedAccessKey={YourSharedAccessKey}"
```

Anote la cadena de conexión del servicio, la necesitará más adelante en este tutorial.

## <a name="download-the-web-app-from-github"></a>Descarga de la aplicación web desde GitHub

Abra una ventana de comandos y escriba los comandos siguientes para descargar el ejemplo de GitHub y cambiar al directorio de ejemplo:

```cmd
git clone https://github.com/Azure-Samples/web-apps-node-iot-hub-data-visualization.git
cd web-apps-node-iot-hub-data-visualization
```

## <a name="examine-the-web-app-code"></a>Examen del código de la aplicación web

En el directorio web-apps-node-iot-hub-data-visualization, abra la aplicación web en el editor que desee. A continuación se muestra la estructura de archivos que se ve en VS Code:

![Estructura de archivos de la aplicación web](./media/iot-hub-live-data-visualization-in-web-apps/web-app-files.png)

Dedique un momento a examinar los archivos siguientes:

* **Server.js** es un script del lado del servicio que inicializa el socket web y la clase contenedora del centro de eventos. Proporciona una devolución de llamada a la clase de contenedor del centro de eventos que la clase utiliza para difundir los mensajes entrantes al socket web.

* **Event-hub-reader.js** es un script del lado del servicio que se conecta al punto de conexión integrado de IoT Hub con el grupo de consumidores y la cadena de conexión especificados. Extrae el valor de DeviceId y EnqueuedTimeUtc de los metadatos en los mensajes entrantes y, a continuación, transmite el mensaje con el método de devolución de llamada registrado por server.js.

* **Chart-device-data.js** es un script de cliente que escucha en el socket web, realiza un seguimiento de cada identificador de dispositivo y almacena los 50 últimos puntos de datos de entrada para cada dispositivo. A continuación, enlaza los datos del dispositivo seleccionado al objeto de gráfico.

* **Index.HTML** controla el diseño de la interfaz de usuario para la página web y hace referencia a los scripts necesarios para la lógica del lado cliente.

## <a name="configure-environment-variables-for-the-web-app"></a>Configuración de las variables de entorno para la aplicación web

Para leer datos desde IoT Hub, la aplicación web necesita su cadena de conexión y el nombre del grupo de consumidores que debe leer. Obtiene estas cadenas del entorno de proceso en las siguientes líneas de server.js:

```javascript
const iotHubConnectionString = process.env.IotHubConnectionString;
const eventHubConsumerGroup = process.env.EventHubConsumerGroup;
```

Establezca las variables de entorno en la ventana de comandos con los siguientes comandos. Reemplace los valores de marcador de posición por la cadena de conexión del servicio de su instancia de IoT Hub y el nombre del grupo de consumidores que creó anteriormente. No incluya las cadenas entre comillas.

```cmd
set IotHubConnectionString=YourIoTHubConnectionString
set EventHubConsumerGroup=YourConsumerGroupName
```

## <a name="run-the-web-app"></a>Ejecución de la aplicación web

1. Asegúrese de que el dispositivo está ejecutándose y enviando datos.

2. En la ventana de comandos, ejecute las líneas siguientes para descargar e instalar los paquetes a los que se hace referencia e iniciar el sitio web:

   ```cmd
   npm install
   npm start
   ```

3. Debería ver la salida en la consola que indica que la aplicación web se ha conectado correctamente a IoT Hub y está escuchando en el puerto 3000:

   ![Aplicación web iniciada en la consola](./media/iot-hub-live-data-visualization-in-web-apps/web-app-console-start.png)

## <a name="open-a-web-page-to-see-data-from-your-iot-hub"></a>Apertura de una página web para ver los datos desde IoT Hub

Abra un explorador en `http://localhost:3000`.

En la lista **Seleccionar un dispositivo**, seleccione el dispositivo para ver un gráfico de ejecución de los 50 últimos puntos de datos de temperatura y de humedad enviados por el dispositivo a IoT Hub.

![Página de aplicación web que muestra la temperatura y humedad en tiempo real](./media/iot-hub-live-data-visualization-in-web-apps/web-page-output.png)

También debería ver la salida en la consola que muestra los mensajes que se la aplicación web difunde al explorador del cliente:  

![Salida de la difusión de la aplicación web en la consola](./media/iot-hub-live-data-visualization-in-web-apps/web-app-console-broadcast.png)

## <a name="host-the-web-app-in-app-service"></a>Hospedaje de la aplicación web en App Service

La [característica Web Apps de Azure App Service](https://docs.microsoft.com/azure/app-service/overview) proporciona una plataforma como servicio (PAAS) para hospedar aplicaciones web. Las aplicaciones web hospedadas en Azure App Service pueden beneficiarse de las eficaces características de Azure, como la seguridad adicional, el equilibrio de carga y la escalabilidad, además de las soluciones DevOps de sus socios y de Azure, como la implementación continua, la administración de paquetes, etc. Azure App Service admite aplicaciones web desarrolladas en muchos lenguajes muy conocidos e implementadas en la infraestructura de Windows o de Linux.

En esta sección, aprovisiona una aplicación web en App Service e implementa el código en ella con comandos de la CLI de Azure. Puede encontrar detalles de los comandos usados en la documentación de [az webapp](https://docs.microsoft.com/cli/azure/webapp?view=azure-cli-latest). Antes de comenzar, asegúrese de que ha completado los pasos necesarios para [agregar un grupo de recursos a su instancia de IoT Hub](#add-a-consumer-group-to-your-iot-hub), [obtener una cadena de conexión de servicio de IoT Hub](#get-a-service-connection-string-for-your-iot-hub) y [descargar la aplicación web desde GitHub](#download-the-web-app-from-github).

1. Un [plan de App Service](https://docs.microsoft.com/azure/app-service/overview-hosting-plans) define un conjunto de recursos de proceso para que una aplicación hospedada en App Service se ejecute. En este tutorial, usamos el nivel gratis o de desarrollador para hospedar la aplicación web. Con el nivel gratis, la aplicación web se ejecuta en los recursos compartidos de Windows con otras aplicaciones de App Service, incluidas las de otros clientes. Azure también ofrece planes de App Service para implementar aplicaciones web en recursos de proceso Linux. Puede omitir este paso si ya tiene un plan de App Service que le gustaría usar.

   Para crear un plan de App Service con el nivel gratuito de Windows, ejecute el siguiente comando. Use el mismo grupo de recursos que su instancia de IoT Hub. El nombre del plan de servicio puede contener letras mayúsculas y minúsculas, números y guiones.

   ```azurecli-interactive
   az appservice plan create --name <app service plan name> --resource-group <your resource group name> --sku FREE
   ```

2. Ahora puede aprovisionar una aplicación web en su plan de App Service. El parámetro `--deployment-local-git` permite que el código de la aplicación web se cargue e implemente desde un repositorio de Git en la máquina local. El nombre de la aplicación web debe ser único globalmente y puede contener letras mayúsculas y minúsculas, números y guiones. Asegúrese de especificar la versión de Node 10.6 o posterior para el parámetro `--runtime`, en función de la versión del tiempo de ejecución de Node.js que esté usando. Puede usar el comando `az webapp list-runtimes` para obtener una lista de los tiempos de ejecución admitidos.

   ```azurecli-interactive
   az webapp create -n <your web app name> -g <your resource group name> -p <your app service plan name> --runtime "node|10.6" --deployment-local-git
   ```

3. Ahora agregue la configuración de la aplicación para las variables de entorno que especifican la cadena de conexión de IoT Hub y el grupo de consumidores del centro de eventos. Las opciones de configuración individuales están delimitadas mediante espacios en el parámetro `-settings`. Use la cadena de conexión del servicio para su instancia de IoT Hub y el grupo de consumidores que creó anteriormente en este tutorial. No incluya los valores entre comillas.

   ```azurecli-interactive
   az webapp config appsettings set -n <your web app name> -g <your resource group name> --settings EventHubConsumerGroup=<your consumer group> IotHubConnectionString="<your IoT hub connection string>"
   ```

4. Habilite el protocolo Web Sockets para la aplicación web y establezca que la aplicación web reciba las solicitudes HTTPS únicamente (las solicitudes HTTP se redirigen a HTTPS).

   ```azurecli-interactive
   az webapp config set -n <your web app name> -g <your resource group name> --web-sockets-enabled true
   az webapp update -n <your web app name> -g <your resource group name> --https-only true
   ```

5. Para implementar el código en App Service, deberá usar sus [credenciales de implementación de nivel de usuario](https://docs.microsoft.com/azure/app-service/deploy-configure-credentials). Las credenciales de implementación de nivel de usuario son diferentes de las credenciales de Azure y se usan para las implementaciones de FTP y Git local en una aplicación web. Una vez establecidas, son válidas en todas las aplicaciones de App Service de todas las suscripciones de su cuenta de Azure. Si previamente ha configurado las credenciales de implementación de nivel de usuario, puede usarlas.

   Si anteriormente no ha establecido las credenciales de implementación de nivel de usuario o si no recuerda su contraseña, ejecute el siguiente comando. El nombre de usuario de la implementación debe ser único dentro de Azure y no debe contener el símbolo ‘@’ para las inserciones de Git local. Cuando se le pida, escriba la nueva contraseña y confírmela. La contraseña debe tener al menos ocho caracteres y dos de los tres elementos siguientes: letras, números y símbolos.

   ```azurecli-interactive
   az webapp deployment user set --user-name <your deployment user name>
   ```

6. Obtenga la dirección URL de Git que se debe usar para insertar el código en App Service.

   ```azurecli-interactive
   az webapp deployment source config-local-git -n <your web app name> -g <your resource group name>
   ```

7. Agregue un control remoto para el clon que hace referencia al repositorio de Git para la aplicación web en App Service. En \<Git clone URL\>, use la dirección URL devuelta en el paso anterior. Ejecute el siguiente comando desde la ventana de comandos.

   ```cmd
   git remote add webapp <Git clone URL>
   ```

8. Para implementar el código en App Service, escriba el siguiente comando en la ventana de comandos. Si le piden las credenciales, escriba las credenciales de implementación de nivel de usuario que creó en el paso 5. Asegúrese de insertar en la rama principal del repositorio remoto de App Service.

    ```cmd
    git push webapp master:master
    ```

9. El progreso de la implementación se actualizará en la ventana de comandos. Si la implementación se realizó correctamente, terminará con líneas similares a las de la siguiente salida:

    ```cmd
    remote:
    remote: Finished successfully.
    remote: Running post deployment command(s)...
    remote: Deployment successful.
    To https://contoso-web-app-3.scm.azurewebsites.net/contoso-web-app-3.git
    6b132dd..7cbc994  master -> master
    ```

10. Ejecute el comando siguiente para consultar el estado de la aplicación web y asegúrese de que se está ejecutando:

    ```azurecli-interactive
    az webapp show -n <your web app name> -g <your resource group name> --query state
    ```

11. Vaya a `https://<your web app name>.azurewebsites.net` en un explorador. Se muestra una página web similar a la que vio cuando ejecutó la aplicación web que se muestra localmente. Suponiendo que el dispositivo se está ejecutando y enviando datos, debería ver un gráfico de ejecución de las 50 lecturas más recientes de temperatura y humedad enviadas por el dispositivo.

## <a name="troubleshooting"></a>Solución de problemas

Si encuentra algún problema con este ejemplo, pruebe los pasos descritos en las secciones siguientes. Si sigue teniendo problemas, envíenos comentarios en la parte inferior de este tema.

### <a name="client-issues"></a>Problemas del cliente

* Si un dispositivo no aparece en la lista o no se dibuja ningún gráfico, asegúrese de que el código de dispositivo se está ejecutando en el dispositivo.

* En el explorador, abra las herramientas de desarrollo (en la mayoría de exploradores, se abrirán al presionar la tecla F12) y busque la consola. Busque las advertencias o errores impresos.

* Puede depurar el script de cliente en /js/chat-device-data.js.

### <a name="local-website-issues"></a>Problemas del sitio web local

* Vea la salida en la ventana donde inició el nodo para el resultado de la consola.

* Depure el código de servidor, en concreto, server.js y /scripts/event-hub-reader.js.

### <a name="azure-app-service-issues"></a>Problemas de Azure App Service

* En Azure Portal, vaya a la aplicación web. En **Supervisión** en el panel izquierdo, seleccione **Registros de App Service**. Active **Application Logging (File System)** [Registro de la aplicación (sistema de archivos)], establezca **Nivel** en Error y, a continuación, seleccione **Guardar**. A continuación, abra **Secuencia de registro** (bajo **Supervisión**).

* Desde la aplicación web en Azure Portal, en **Herramientas de desarrollo**, seleccione **Consola** y compruebe las versiones de node y npm con `node -v` y `npm -v`.

* Si ve un error que indica que no se encuentra un paquete, puede haber ejecutado los pasos desordenados. Cuando se implementa el sitio (con `git push`), se ejecuta el servicio de aplicaciones `npm install`, y lo hace según la versión actual del nodo que ha configurado. Si esto se cambia en la configuración más adelante, deberá realizar un pequeño cambio en el código y volver a insertar.

## <a name="next-steps"></a>Pasos siguientes

Ha usado correctamente una aplicación web para visualizar datos del sensor en tiempo real desde su IoT Hub.

Para otra forma de visualizar los datos desde Azure IoT Hub, consulte [Visualización de datos del sensor en tiempo real desde Azure IoT Hub mediante Power BI](iot-hub-live-data-visualization-in-power-bi.md).

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
