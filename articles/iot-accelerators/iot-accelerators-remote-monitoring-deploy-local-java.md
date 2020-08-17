---
title: 'Implementación local de la solución de supervisión remota: IDE de Intellij| Microsoft Docs'
description: En esta guía paso a paso, se explica cómo se implementa el acelerador de soluciones Supervisión remota en una máquina local utilizando IntelliJ para las pruebas y el desarrollo.
author: v-krghan
manager: dominicbetts
ms.custom: devx-track-java
ms.author: v-krghan
ms.service: iot-accelerators
services: iot-accelerators
ms.date: 01/24/2019
ms.topic: conceptual
ms.openlocfilehash: f7554843db247ade1cddff78902430a5d84debe1
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319174"
---
# <a name="deploy-the-remote-monitoring-solution-accelerator-locally---intellij"></a>Implementación local del acelerador de soluciones de supervisión remota: IntelliJ

[!INCLUDE [iot-accelerators-selector-local](../../includes/iot-accelerators-selector-local.md)]

En este artículo, se explica cómo implementar el acelerador de soluciones de supervisión remota en la máquina local para pruebas y desarrollo. Aprenderá a ejecutar los microservicios en IntelliJ. Las implementaciones locales de microservicios utilizan los siguientes servicios en la nube: IoT Hub, Azure Cosmos DB, Azure Streaming Analytics y Azure Time Series Insights.

Si quiere ejecutar el acelerador de soluciones de supervisión remota en Docker en el equipo local, consulte [Deploy the Remote Monitoring solution accelerator locally - Docker](iot-accelerators-remote-monitoring-deploy-local-docker.md) (Implementación local del acelerador de soluciones de supervisión remota: Docker).

## <a name="prerequisites"></a>Requisitos previos

Para implementar los servicios de Azure que usa el acelerador de soluciones de supervisión remota, necesita una suscripción de Azure activa.

En caso de no tener ninguna, puede crear una cuenta de evaluación gratuita en tan solo unos minutos. Para obtener más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/).

### <a name="machine-setup"></a>Configuración de la máquina

Para completar la implementación local, necesita que las herramientas siguientes estén instaladas en la máquina de desarrollo local:

* [Git](https://git-scm.com/)
* [Docker](https://www.docker.com)
* [Java 8](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* [IntelliJ Community Edition](https://www.jetbrains.com/idea/download/)
* [Complemento Scala de IntelliJ](https://plugins.jetbrains.com/plugin/1347-scala)
* [Complemento SBT de IntelliJ](https://plugins.jetbrains.com/plugin/5007-sbt)
* [Complemento SBT Executor de IntelliJ](https://plugins.jetbrains.com/plugin/7247-sbt-executor)
* [Nginx](https://nginx.org/en/download.html)
* [Node.js v8](https://nodejs.org/)

Node.js v8 es un requisito previo de la CLI de PCS que los scripts utilizan para crear recursos de Azure. No use Node.js v10.

> [!NOTE]
> El IDE de IntelliJ está disponible para Windows y Mac.

## <a name="download-the-source-code"></a>Descarga del código fuente

Los repositorios de código fuente de Supervisión remota incluyen el código fuente y los archivos de configuración de Docker que necesita para ejecutar las imágenes de Docker de los microservicios.

Si desea clonar y crear una versión local del repositorio, use el entorno de línea de comandos para ir a una carpeta adecuada de la máquina local. A continuación, ejecute uno de los siguientes conjuntos de comandos para clonar el repositorio de Java:

* Para descargar la versión más reciente de las implementaciones de microservicios de Java, ejecute el siguiente comando:

  ```cmd/sh
  git clone --recurse-submodules https://github.com/Azure/azure-iot-pcs-remote-monitoring-java.git
  ```

* Para recuperar los submódulos más recientes, ejecute los siguientes comandos:

   ```cmd/sh
   cd azure-iot-pcs-remote-monitoring-java
   git submodule foreach git pull origin master
    ```

> [!NOTE]
> Estos comandos descargan el código fuente de todos los microservicios, además de los scripts que se usan para ejecutar los microservicios localmente. No necesita el código fuente para ejecutar los microservicios en Docker. Sin embargo, le resultará útil si más adelante planea modificar el acelerador de la solución y probar los cambios en el entorno local.

## <a name="deploy-the-azure-services"></a>Implementación de servicios de Azure

Aunque en este artículo se muestra cómo ejecutar los microservicios localmente, dependen de los servicios de Azure que se ejecutan en la nube. Use el siguiente script para implementar los servicios de Azure. En los scripts de ejemplo, se presupone que está utilizando el repositorio de Java en una máquina Windows. Si está trabajando en otro entorno, ajuste las rutas de acceso, las extensiones de archivo y los separadores de ruta de acceso de la manera correcta.

### <a name="create-new-azure-resources"></a>Crear nuevos recursos de Azure

Si aún no ha creado los recursos de Azure necesarios, siga estos pasos:

1. En el entorno de línea de comandos, vaya a la carpeta **\services\scripts\local\launch** de la copia clonada del repositorio.

1. Ejecute los comandos siguientes para instalar la herramienta de la CLI **pcs** e inicie sesión en su cuenta de Azure:

    ```cmd
    npm install -g iot-solutions
    pcs login
    ```

1. Ejecute el script **start.cmd**. El script le pedirá la siguiente información:

   * Nombre de la solución.
   * La suscripción de Azure que se va a usar.
   * La ubicación del centro de datos de Azure que se va a usar.

   El script crea un grupo de recursos de Azure con el nombre de la solución. Este grupo de recursos contiene los recursos de Azure que el acelerador de soluciones utiliza. Puede eliminar este grupo de recursos cuando ya no necesite los recursos correspondientes.

   El script también agrega un conjunto de variables de entorno en el equipo local. Todos los nombres de variable tienen el prefijo **PCS**. Estas variables de entorno proporcionan detalles que permiten a Supervisión remota leer los valores de configuración de un recurso de Azure Key Vault.

   > [!TIP]
   > Cuando el script finaliza, guarda las variables de entorno en un archivo denominado **\<your home folder\>\\.pcs\\\<solution name\>.env**. Podrá utilizarlas en futuras implementaciones del acelerador de soluciones. Tenga en cuenta que cualquier variable de entorno establecida en el equipo local invalidará los valores del archivo **services\\scripts\\local\\.env** al ejecutar **docker-compose**.

1. Cierre el entorno de la línea de comandos.

### <a name="use-existing-azure-resources"></a>Usar recursos de Azure existentes

Si ya ha creado los recursos de Azure necesarios, establezca las variables de entorno correspondientes en el equipo local.
* **PCS_KEYVAULT_NAME**: nombre del recurso de Key Vault.
* **PCS_AAD_APPID**: identificador de la aplicación Azure Active Directory (Azure AD).
* **PCS_AAD_APPSECRET**: secreto de la aplicación Azure AD.

Los valores de configuración se leerán desde este recurso de Key Vault. Estas variables de entorno pueden guardarse en el archivo **\<your home folder\>\\.pcs\\\<solution name\>.env** de la implementación. Tenga en cuenta que las variables de entorno establecidas en el equipo local invalidan los valores del archivo **services\\scripts\\local\\.env** al ejecutar **docker-compose**.

Parte de la configuración necesaria para el microservicio se almacena en una instancia de Key Vault que se ha creado en la implementación inicial. Las variables correspondientes del almacén de claves deben modificarse según sea necesario.

## <a name="run-the-microservices"></a>Ejecución de los microservicios

En esta sección, ejecutará los microservicios de Supervisión remota. Va a ejecutar:

* La interfaz de usuario web de forma nativa.
* Los servicios de simulación de dispositivos de Azure IoT Device, Auth y el administrador de Azure Stream Analytics en Docker.
* Los microservicios en IntelliJ.

### <a name="run-the-device-simulation-service"></a>Ejecución del servicio Simulación de dispositivo

Abra una nueva ventana del símbolo del sistema. Compruebe que tiene acceso a las variables de entorno establecidas mediante el script **start.cmd** en la sección anterior.

Ejecute el siguiente comando para abrir el contenedor de Docker del servicio Simulación de dispositivo. El servicio simula dispositivos para la solución Supervisión remota.

```cmd
<path_to_cloned_repository>\services\device-simulation\scripts\docker\run.cmd
```

### <a name="run-the-auth-service"></a>Ejecución del servicio Auth

Abra una nueva ventana del símbolo del sistema y ejecute el siguiente comando para abrir el contenedor de Docker para el servicio Auth. Con este servicio, puede administrar los usuarios que tienen autorización para acceder a las soluciones de Azure IoT.

```cmd
<path_to_cloned_repository>\services\auth\scripts\docker\run.cmd
```

### <a name="run-the-stream-analytics-manager-service"></a>Ejecución del servicio Stream Analytics Manager

Abra una nueva ventana del símbolo del sistema y ejecute el siguiente comando para abrir el contenedor de Docker para el servicio del administrador de Stream Analytics. Con este servicio, puede administrar los trabajos de Stream Analytics. Algunas de las tareas de administración consisten en configurar el trabajo e iniciar, detener y supervisar el estado del trabajo.

```cmd
<path_to_cloned_repository>\services\asa-manager\scripts\docker\run.cmd
```

### <a name="deploy-all-other-microservices-on-your-local-machine"></a>Implementación de todos los demás microservicios en la máquina local

En los pasos siguientes, se explica cómo se ejecutan los microservicios de Supervisión remota en IntelliJ:

#### <a name="import-a-project"></a>Importación de un proyecto

1. Abra el IDE de IntelliJ.
1. Seleccione **Importar proyecto**.
1. Seleccione **azure-iot-pcs-remote-monitoring-java\services\build.sbt**.

#### <a name="create-run-configurations"></a>Creación de configuraciones de ejecución

1. Seleccione **Ejecutar** > **Editar configuraciones**.
1. Seleccione **Add New Configuration** > **sbt task** (Agregar nueva configuración > tarea de sbt).
1. Escriba el nombre en el campo **Nombre** y especifique **run** en **Tareas**.
1. En **Directorio de trabajo**, seleccione el directorio de trabajo correspondiente al servicio que desea ejecutar.
1. Seleccione **Aplicar** > **Aceptar** para guardar los valores.
1. Cree configuraciones de ejecución para los siguientes servicios web:
    * WebService (services\config)
    * WebService (services\device-telemetry)
    * WebService (services\iothub-manager)
    * WebService (services\storage-adapter)

Como ejemplo, en la siguiente imagen se muestra cómo se agrega una configuración de un servicio:

[![Captura de pantalla de la ventana de configuraciones de ejecución o depuración del IDE de IntelliJ, que muestra la opción storageAdapter resaltada en la lista de tareas sbt del panel izquierdo y entradas en los cuadros de nombre, tareas, directorio de trabajo y parámetros de VM del panel derecho.](./media/deploy-locally-intellij/run-configurations.png)](./media/deploy-locally-intellij/run-configurations.png#lightbox)

#### <a name="create-a-compound-configuration"></a>Creación de una configuración compuesta

1. Para ejecutar juntos todos los servicios, seleccione **Add new Configuration** > **Compound** (Agregar nueva configuración > Compuesta).
1. Escriba un **Nombre** y seleccione **add sbt tasks** (agregar tareas de sbt).
1. Seleccione **Aplicar** > **Aceptar** para guardar los valores.

Como ejemplo, en la siguiente imagen se muestra cómo se agregan todas las tareas de sbt a la misma configuración:

[![Captura de pantalla de la ventana Run/Debug Configurations (Ejecutar/depurar configuraciones) del IDE de IntelliJ, en la que aparece la opción AllServices resaltada en la lista Compound (Compuesta) del panel izquierdo y la opción sbt Task 'deviceTelemetry' aparece resaltada en el panel derecho.](./media/deploy-locally-intellij/all-services.png)](./media/deploy-locally-intellij/all-services.png#lightbox)

Haga clic en **Run** (Ejecutar) para compilar y ejecutar los servicios web en el equipo local.

Cada servicio web abre una ventana del explorador web y del símbolo del sistema. En el símbolo del sistema, aparecerá el resultado del servicio en ejecución. La ventana del explorador permite supervisar el estado. No cierre las ventanas del símbolo del sistema ni las páginas web, ya que detendría el servicio web.

Para acceder al estado de los servicios, vaya a las siguientes direcciones URL:

* Administrador de IoT-Hub: `http://localhost:9002/v1/status`
* Telemetría del dispositivo: `http://localhost:9004/v1/status`
* config: `http://localhost:9005/v1/status`
* storage-adapter: `http://localhost:9022/v1/status`

### <a name="start-the-stream-analytics-job"></a>Inicio del trabajo de Stream Analytics

Siga estos pasos para iniciar el trabajo de Stream Analytics:

1. Vaya a [Azure Portal](https://portal.azure.com).
1. Vaya al **grupo de recursos** creado para la solución. El nombre del grupo de recursos es el nombre que eligió para la solución al ejecutar el script **start.cmd**.
1. Seleccione **Trabajo de Stream Analytics** en la lista de recursos.
1. En la página **Información general** del trabajo de Stream Analytics, seleccione el botón **Iniciar** y haga clic en **Iniciar** para iniciar el trabajo.

### <a name="run-the-web-ui"></a>Ejecución de la interfaz de usuario web

En este paso, inicie la interfaz de usuario web. Abra una nueva ventana del símbolo del sistema. Compruebe que tiene acceso a las variables de entorno establecidas mediante el script **start.cmd**. Vaya a la carpeta **webui** de la copia local del repositorio y ejecute los siguientes comandos:

```cmd
npm install
npm start
```

Cuando el comando **start** se haya completado, se abrirá una página del explorador con la dirección `http://localhost:3000/dashboard`. Los errores de esta página son de esperar. Para ver la aplicación sin errores, siga estos pasos.

### <a name="configure-and-run-nginx"></a>Configuración y ejecución de Nginx

Configure un servidor proxy inverso que vincule la aplicación web con los microservicios que se ejecutan en el equipo local:

1. Copie el archivo **nginx.conf** de la carpeta **webui\scripts\localhost** de la copia local del repositorio en el directorio **nginx\conf** de instalación.
1. Ejecute Nginx.

Para más información acerca de cómo ejecutar Nginx, consulte [nginx para Windows](https://nginx.org/en/docs/windows.html).

### <a name="connect-to-the-dashboard"></a>Conexión al panel

Para acceder al panel de la solución de Supervisión remota, vaya a `http://localhost:9000` en el explorador.

## <a name="clean-up"></a>Limpieza

Para evitar cargos innecesarios, cuando haya terminado las pruebas, quite los servicios en la nube de su suscripción a Azure. Para quitar los servicios, vaya a [Azure Portal](https://ms.portal.azure.com) y elimine el grupo de recursos que se creó con el script **start.cmd**.

También puede eliminar la copia local del repositorio de Supervisión remota que se creó al clonar el código fuente de GitHub.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha implementado la solución de supervisión remota, el paso siguiente es [explorar las funcionalidades del panel de soluciones](quickstart-remote-monitoring-deploy.md).
