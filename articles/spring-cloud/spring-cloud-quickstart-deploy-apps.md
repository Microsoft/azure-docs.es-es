---
title: 'Inicio rápido: Compilación e implementación de aplicaciones en Azure Spring Cloud'
description: Describe la implementación de aplicaciones en Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 08/03/2020
ms.custom: devx-track-java
ms.openlocfilehash: 8931c22c3656cf9708756153268ab1d9d87b8343
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050835"
---
# <a name="quickstart-build-and-deploy-apps-to-azure-spring-cloud"></a>Inicio rápido: Compilación e implementación de aplicaciones en Azure Spring Cloud

En este documento se explica cómo compilar e implementar aplicaciones de microservicios en Azure Spring Cloud mediante:
* la CLI de Azure
* Complemento de Maven
* Intellij

Antes de realizar la implementación mediante la CLI de Azure o Maven, complete los ejemplos que [aprovisionan una instancia de Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md) y [configure el servidor de configuración](spring-cloud-quickstart-setup-config-server.md).

## <a name="prerequisites"></a>Requisitos previos

* [Instalación de JDK 8](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable)
* [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)
* (Opcional) [Instale la versión 2.0.67 de la CLI de Azure, o cualquier versión superior](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest), e instale la extensión de Azure Spring Cloud con el comando `az extension add --name spring-cloud`
* (Opcional) [Instale Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) e [inicie sesión](https://docs.microsoft.com/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in)

## <a name="deployment-procedures"></a>Procedimientos de implementación

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

### <a name="build-the-microservices-applications-locally"></a>Compilación local de aplicaciones de microservicios

1. Clone el repositorio de la aplicación de ejemplo en su cuenta de Azure Cloud.  

    ```azurecli
    git clone https://github.com/Azure-Samples/piggymetrics
    ```

2. Cambie de directorio y compile el proyecto.

    ```azurecli
    cd piggymetrics
    mvn clean package -DskipTests
    ```

La compilación del proyecto tarda aproximadamente 5 minutos. Una vez completado, debe tener archivos JAR individuales para cada servicio en sus respectivas carpetas.

### <a name="create-and-deploy-the-apps"></a>Creación e implementación de las aplicaciones

1. Establezca el nombre del grupo de recursos y el nombre del clúster predeterminados con los siguientes comandos:

    ```azurecli
    az configure --defaults group=<resource group name>
    az configure --defaults spring-cloud=<service instance name>
    ```

1. Cree microservicios de Azure Spring Cloud mediante los archivos JAR compilados en el paso anterior. Creará tres aplicaciones: **gateway**, **auth-service** y **account-service**.

    ```azurecli
    az spring-cloud app create --name gateway
    az spring-cloud app create --name auth-service
    az spring-cloud app create --name account-service
    ```

1. Necesitamos implementar las aplicaciones que se han creado en el paso anterior en Azure. Use los siguientes comandos para implementar las tres aplicaciones:

    ```azurecli
    az spring-cloud app deploy -n gateway --jar-path ./gateway/target/gateway.jar
    az spring-cloud app deploy -n account-service --jar-path ./account-service/target/account-service.jar
    az spring-cloud app deploy -n auth-service --jar-path ./auth-service/target/auth-service.jar
    ```

### <a name="assign-public-endpoint-to-gateway"></a>Asignación de un punto de conexión público a una puerta de enlace

Necesitamos una forma de acceder a la aplicación mediante un explorador web. La aplicación de puerta de enlace necesita un punto de conexión que esté de cara al público.

1. Use el siguiente comando para asignar el punto de conexión:

    ```azurecli
    az spring-cloud app update -n gateway --is-public true
    ```

2. Consulte la aplicación de **puerta de enlace** para ver su dirección IP pública, con el fin de que pueda comprobar que la aplicación se está ejecutando:

    ```azurecli
    az spring-cloud app show --name gateway --query properties.url
    ```

#### <a name="maven"></a>[Maven](#tab/Maven)

### <a name="clone-and-build-the-sample-application-repository"></a>Clonación y compilación del repositorio de aplicaciones de ejemplo

1. Para clonar el repositorio de Git, ejecute el siguiente comando:

    ```
    git clone https://github.com/Azure-Samples/PiggyMetrics
    ```
  
1. Ejecute el siguiente comando para cambiar el directorio y compilar el proyecto:

    ```
    cd piggymetrics
    mvn clean package -DskipTests
    ```

### <a name="generate-configurations-and-deploy-to-the-azure-spring-cloud"></a>Generación de configuraciones e implementación en Azure Spring Cloud

1. Genere configuraciones mediante la ejecución del siguiente comando en la carpeta raíz de PiggyMetrics que contiene el POM primario. Si ya ha iniciado sesión con la CLI de Azure, el comando tomará automáticamente las credenciales. Si no lo ha hecho, le proporcionará instrucciones para iniciar sesión. Para más información, consulte la [página wiki](https://github.com/microsoft/azure-maven-plugins/wiki/Authentication).

    ```
    mvn com.microsoft.azure:azure-spring-cloud-maven-plugin:1.1.0:config
    ```
    
    Se le pedirá que seleccione:
    * **Módulos:** Seleccione `gateway`, `auth-service` y `account-service`.
    * **Subscription** (Suscripción): se trata de la suscripción que se usa para crear una instancia de Azure Spring Cloud.
    * **Instancia de servicio**: es el nombre de la instancia de Azure Spring Cloud.
    * **Punto de conexión público:** en la lista de proyectos que se proporcionan, escriba el número correspondiente a `gateway`.  Esto le proporciona acceso público.

1. El archivo POM ahora contiene las dependencias y configuraciones del complemento. Implemente las aplicaciones mediante el comando siguiente. 

    ```
    mvn azure-spring-cloud:deploy
    ```

#### <a name="intellij"></a>[IntelliJ](#tab/IntelliJ)

### <a name="import-sample-project-in-intellij"></a>Importación del proyecto de ejemplo en IntelliJ

1. Descargue y descomprima el repositorio de origen de este tutorial o clónelo con Git: `git clone https://github.com/Azure-Samples/piggymetrics`. 

1. Abra el cuadro de diálogo de **bienvenida** de IntelliJ y seleccione **Import Project** (Importar proyecto) para abrir el asistente de importación.

1. Seleccione la carpeta `piggymetric`.

    ![Importar proyecto](media/spring-cloud-intellij-howto/revision-import-project-1.png)

### <a name="deploy-gateway-app-to-azure-spring-cloud"></a>Implementación de una aplicación de puerta de enlace en Azure Spring Cloud
Para realizar la implementación en Azure, debe iniciar sesión con su cuenta de Azure con Azure Toolkit for IntelliJ y elegir su suscripción. Puede encontrar información sobre el inicio de sesión en [Instalación e inicio de sesión](https://docs.microsoft.com/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

1. Vaya al explorador de proyectos de IntelliJ, haga clic con el botón derecho en el proyecto y seleccione **Azure** -> **Deploy to Azure Spring Cloud** (Implementar en Azure Spring Cloud).

    ![Implementación en Azure 1](media/spring-cloud-intellij-howto/revision-deploy-to-azure-1.png)

1. En el campo **Nombre** anexe *:gateway* al valor de **Nombre** existente que hace referencia a la configuración.
1. En el cuadro de texto **Artefacto**, seleccione *com.piggymetrics:gateway:1.0-SNAPSHOT*.
1. En el cuadro de texto **Suscripción**, compruebe su suscripción.
1. En el cuadro de texto **Spring Cloud**, seleccione la instancia de Azure Spring Cloud que creó en el artículo sobre el [aprovisionamiento de una instancia de Azure Spring Cloud](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart-provision-service-instance).
1. En **Punto de conexión público**, seleccione *Habilitar*.
1. En el cuadro de texto **Aplicación:** , seleccione **Crear aplicación...** .
1. Escriba *gateway* y haga clic en **Aceptar**.

    ![Implementación en Azure: OK (Aceptar)](media/spring-cloud-intellij-howto/revision-deploy-to-azure-2.png)

1. En la sección **Before launch** (Antes del inicio) del cuadro de diálogo, haga doble clic en *Run Maven Goal* (Ejecutar objetivo de Maven).
1. En el cuadro de texto **Working directory** (Directorio de trabajo), vaya a la carpeta *piggymetrics/gateway*.
1. En el cuadro de texto **Command line** (Línea de comandos), escriba *package -DskipTests*. Haga clic en **OK**.
1. Para iniciar la implementación haga clic en el botón **Run** (Ejecutar) en la parte inferior del cuadro de diálogo **Deploy Azure Spring Cloud app** (Implementar una aplicación de Azure Spring Cloud). El complemento ejecutará el comando `mvn package` en la aplicación `gateway` e implementará el archivo .jar que ha generado el comando `package`.

### <a name="deploy-auth-service-and-account-service-apps-to-azure-spring-cloud"></a>Implementación de las aplicaciones auth-service and account-service en Azure Spring Cloud
Puede repetir los pasos anteriores para implementar las aplicaciones `auth-service` y `account-service` en Azure Spring Cloud:

1. Modifique los valores de **Nombre** y **Artefacto** para identificar la aplicación `auth-service`.
1. En el cuadro de texto **Aplicación:** , seleccione **Crear aplicación...** para crear la aplicación `auth-service`.
1. Compruebe que en la opción **Punto de conexión público** se ha seleccionado *Deshabilitado*.
1. En la sección **Before launch** (Antes del inicio) del cuadro de diálogo, cambie el valor de **Working directory** (Directorio de trabajo) a la carpeta *piggymetrics/auth-service*.
1. Para iniciar la implementación haga clic en el botón **Run** (Ejecutar) en la parte inferior del cuadro de diálogo **Deploy Azure Spring Cloud app** (Implementar una aplicación de Azure Spring Cloud). 
1. Repita estos procedimientos para configurar e implementar `account-service`.
---

Vaya a la dirección URL que se proporciona en la salida de los pasos anteriores para acceder a la aplicación PiggyMetrics. p. ej. `https://<service instance name>-gateway.azuremicroservices.io`

![Acceso a PiggyMetrics](media/spring-cloud-quickstart-launch-app-cli/launch-app.png)

También puede navegar por el Azure Portal para buscar la dirección URL. 
1. Vaya al servicio.
2. Seleccione **Aplicaciones**.
3. Seleccione **gateway**.

    ![Desplazamiento a la aplicación](media/spring-cloud-quickstart-launch-app-cli/navigate-app1.png)
    
4. Busque la dirección URL en la página **puerta de enlace | Información general**.

    ![Desplazamiento a la segunda aplicación](media/spring-cloud-quickstart-launch-app-cli/navigate-app2-url.png)

## <a name="clean-up-resources"></a>Limpieza de recursos
En los pasos anteriores, creó recursos de Azure en un grupo de recursos. Si prevé que no va a necesitar estos recursos en el futuro, elimine el grupo de recursos del portal, para lo que debe ejecutar el siguiente comando en Cloud Shell:
```azurecli
az group delete --name <your resource group name; for example: helloworld-1558400876966-rg> --yes
```
En los pasos anteriores, también se establece el nombre del grupo de recursos predeterminado. Para borrar ese grupo predeterminado, ejecute el siguiente comando en Cloud Shell:
```azurecli
az configure --defaults group=
```
## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Registros, métricas y seguimiento](spring-cloud-quickstart-logs-metrics-tracing.md)
