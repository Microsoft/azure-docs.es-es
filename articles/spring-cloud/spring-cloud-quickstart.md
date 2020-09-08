---
title: 'Inicio rápido: implementación de la primera aplicación de Azure Spring Cloud'
description: En este inicio rápido, se implementa una aplicación de Spring Cloud en Azure Spring Cloud.
author: bmitchell287
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 08/05/2020
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 245516e0a54865d3a6097c4bb566b850cb738ad6
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89260557"
---
# <a name="quickstart-deploy-your-first-azure-spring-cloud-application"></a>Inicio rápido: implementación de la primera aplicación de Azure Spring Cloud

En este inicio rápido se explica cómo implementar una sencilla aplicación de microservicios de Azure Spring Cloud para que se ejecute en Azure. 

El código de aplicación que se usa en este tutorial es una aplicación sencilla compilada con Spring Initializr. Cuando haya completado este ejemplo, se podrá acceder en línea a la aplicación de ejemplo proporcionada y se podrá administrar mediante Azure Portal.

En este inicio rápido se explica cómo:

> [!div class="checklist"]
> * Generar un proyecto básico de Spring Cloud
> * Aprovisionar una instancia de servicio
> * Compilar e implementar la aplicación con un punto de conexión público
> * Streaming de registros en tiempo real

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía de inicio rápido:

* [Instalación de JDK 8](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable)
* [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)
* (Opcional) [Instale la versión 2.0.67, o cualquier versión superior,](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) y la extensión de Azure Spring Cloud con el comando `az extension add --name spring-cloud`
* (Opcional) [Instale Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) e [inicie sesión](https://docs.microsoft.com/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in)

## <a name="generate-a-spring-cloud-project"></a>Generación de un proyecto de Spring Cloud

Comience con [Spring Initializr](https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.3.3.RELEASE&packaging=jar&jvmVersion=1.8&groupId=com.example&artifactId=hellospring&name=hellospring&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.hellospring&dependencies=web,cloud-eureka,actuator,cloud-starter-sleuth,cloud-starter-zipkin) para generar un proyecto de ejemplo con las dependencias recomendadas para Azure Spring Cloud. En la imagen siguiente se muestra la configuración de Initializr para este proyecto de ejemplo.
```url
https://start.spring.io/#!type=maven-project&language=java&platformVersion=2.3.3.RELEASE&packaging=jar&jvmVersion=1.8&groupId=com.example&artifactId=hellospring&name=hellospring&description=Demo%20project%20for%20Spring%20Boot&packageName=com.example.hellospring&dependencies=web,cloud-eureka,actuator,cloud-starter-sleuth,cloud-starter-zipkin
```

  ![Página de Initializr](media/spring-cloud-quickstart-java/initializr-page.png)

1. Haga clic en **Generate** (Generar) cuando se establezcan todas las dependencias. Descargue y desempaquete el paquete y, después, cree un controlador web para una aplicación web simple agregando `src/main/java/com/example/hellospring/HelloController.java` como se indica a continuación:

    ```java
    package com.example.hellospring;
    
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.bind.annotation.RequestMapping;
    
    @RestController
    public class HelloController {
    
        @RequestMapping("/")
        public String index() {
            return "Greetings from Azure Spring Cloud!";
        }
    
    }
    ```
## <a name="provision-an-instance-of-azure-spring-cloud"></a>Aprovisionamiento de una instancia de Azure Spring Cloud

En el procedimiento siguiente se crea una instancia de Azure Spring Cloud desde Azure Portal.

1. Abra [Azure Portal](https://ms.portal.azure.com/) en una pestaña ventana. 

2. En el cuadro de búsqueda superior, busque *Azure Spring Cloud*.

3. Seleccione *Azure Spring Cloud* en los resultados.

    ![Icono de inicio de ASC](media/spring-cloud-quickstart-launch-app-portal/find-spring-cloud-start.png)

4. En la página Azure Spring Cloud, haga clic en **+ Add** (+ Agregar).

    ![Icono de adición de ASC](media/spring-cloud-quickstart-launch-app-portal/spring-cloud-add.png)

5. Rellene el formulario en la página **Crear** de Azure Spring Cloud.  Tenga en cuenta las directrices siguientes:
    - **Suscripción**: seleccione la suscripción a la que desea que se facture este recurso.
    - **Grupo de recursos**: se recomienda crear grupos de recursos para los nuevos recursos. Se usarán en los pasos posteriores como **\<resource group name\>** .
    - **Detalles o nombre del servicio**: Especifique **\<service instance name\>** .  El nombre debe tener entre 4 y 32 caracteres, y solo puede contener números, letras minúsculas y guiones.  El primer carácter del nombre del servicio debe ser una letra y el último debe ser una letra o un número.
    - **Ubicación**: seleccione la región de la instancia de servicio.

    ![Inicio del portal de ASC](media/spring-cloud-quickstart-launch-app-portal/portal-start.png)

6. Haga clic en **Revisar y crear**.

## <a name="build-and-deploy-the-app"></a>Compilación e implementación de la aplicación
    
#### <a name="cli"></a>[CLI](#tab/Azure-CLI)
El siguiente procedimiento compila e implementa la aplicación mediante la CLI de Azure. Ejecute el siguiente comando en la raíz del proyecto.

1. Compile el proyecto mediante Maven:

    ```console
    mvn clean package -DskipTests
    ```

1. Si aún no la ha instalado, instale la extensión de Azure Spring Cloud para la CLI de Azure:

    ```azurecli
    az extension add --name spring-cloud
    ```
    
1. Cree la aplicación con el punto de conexión público asignado:

    ```azurecli
    az spring-cloud app create -n hellospring -s <service instance name> -g <resource group name> --is-public
    ```

1. Implemente el archivo .jar de la aplicación:

    ```azurecli
    az spring-cloud app deploy -n hellospring -s <service instance name> -g <resource group name> --jar-path target\hellospring-0.0.1-SNAPSHOT.jar
    ```
    
1. La implementación de la aplicación tarda unos minutos en finalizar. Para confirmar que se ha implementado, vaya a la hoja **Aplicaciones** de Azure Portal. Debería ver el estado de la aplicación.

#### <a name="intellij"></a>[IntelliJ](#tab/IntelliJ)

En el procedimiento siguiente se usa el complemento IntelliJ para Azure Spring Cloud para implementar la aplicación de ejemplo en IntelliJ IDEA.  

### <a name="import-project"></a>Importación del proyecto

1. Abra el cuadro de diálogo de **bienvenida** de IntelliJ y seleccione **Import Project** (Importar proyecto) para abrir el asistente para importar.
1. Seleccione la carpeta `hellospring`.

    ![Importar proyecto](media/spring-cloud-quickstart-java/intellij-new-project.png)

### <a name="deploy-the-app"></a>Implementar la aplicación
Para realizar la implementación en Azure, debe iniciar sesión con su cuenta de Azure y elegir su suscripción.  Puede encontrar información sobre el inicio de sesión en [Instalación e inicio de sesión](https://docs.microsoft.com/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

1. Vaya al explorador de proyectos de IntelliJ, haga clic con el botón derecho en el proyecto y seleccione **Azure** -> **Deploy to Azure Spring Cloud** (Implementar en Azure Spring Cloud).

    [![Implementación en Azure 1](media/spring-cloud-quickstart-java/intellij-deploy-azure-1.png) ](media/spring-cloud-quickstart-java/intellij-deploy-azure-1.png#lightbox)

1. Acepte el nombre de la aplicación en el campo **Name** (Nombre). El **nombre** hace referencia a la configuración, no al nombre de la aplicación. Normalmente, los usuarios no tienen que cambiarlo.
1. En el cuadro de texto **Artefacto**, seleccione *hellospring-0.0.1-SNAPSHOT.jar*.
1. En el cuadro de texto **Suscripción**, compruebe su suscripción.
1. En el cuadro de texto **Spring Cloud**, seleccione la instancia de Azure Spring Cloud que creó en el artículo sobre el [aprovisionamiento de una instancia de Azure Spring Cloud](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart-provision-service-instance).
1. En **Punto de conexión público**, seleccione *Habilitar*.
1. En el cuadro de texto **Aplicación:** , seleccione **Crear aplicación...** .
1. Escriba *hellospring* y, después, haga clic en **Aceptar**.

    [ ![Implementación en Azure: OK (Aceptar)](media/spring-cloud-quickstart-java/intellij-deploy-to-azure.png) ](media/spring-cloud-quickstart-java/intellij-deploy-to-azure.png#lightbox)

1. Para iniciar la implementación haga clic en el botón **Run** (Ejecutar) en la parte inferior del cuadro de diálogo **Deploy Azure Spring Cloud app** (Implementar una aplicación de Azure Spring Cloud). El complemento ejecutará el comando `mvn package` en la aplicación `hellospring` e implementará el archivo .jar que ha generado el comando `package`.
---

Una vez completada la implementación, puede acceder a la aplicación en `https://<service instance name>-hellospring.azuremicroservices.io/`.

  [ ![Acceso a la aplicación desde el explorador](media/spring-cloud-quickstart-java/access-app-browser.png) ](media/spring-cloud-quickstart-java/access-app-browser.png#lightbox)

## <a name="streaming-logs-in-real-time"></a>Registros de streaming en tiempo real

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

Use el comando siguiente para obtener registros en tiempo real de la aplicación.

```azurecli
az spring-cloud app logs -n hellospring -s <service instance name> -g <resource group name> --lines 100 -f

```
Los registros aparecen en los resultados:

[ ![Registros de streaming](media/spring-cloud-quickstart-java/streaming-logs.png) ](media/spring-cloud-quickstart-java/streaming-logs.png#lightbox)

>[!TIP]
> Use `az spring-cloud app logs -h` para explorar más parámetros y funcionalidades de las secuencias de registro.

#### <a name="intellij"></a>[IntelliJ](#tab/IntelliJ)

1. Seleccione **Azure Explorer** (Explorador de Azure) y, luego, **Spring Cloud**.
1. Haga clic con el botón derecho en la aplicación en ejecución.
1. Seleccione **Streaming Logs** (Registros de streaming) en la lista desplegable.
1. Seleccione la instancia.

    [ ![Seleccionar registros de streaming](media/spring-cloud-quickstart-java/intellij-get-streaming-logs.png) ](media/spring-cloud-quickstart-java/intellij-get-streaming-logs.png)

1. El registro de streaming se verá en la ventana de salida.

    [ ![Salida del registro de streaming](media/spring-cloud-quickstart-java/intellij-streaming-logs-output.png) ](media/spring-cloud-quickstart-java/intellij-streaming-logs-output.png)
---

Para ver las características avanzadas de los análisis de registros, visite la pestaña **Registros** en el menú de [Azure Portal](https://portal.azure.com/). Aquí los registros tienen una latencia de unos minutos.

[ ![Análisis de registros](media/spring-cloud-quickstart-java/logs-analytics.png) ](media/spring-cloud-quickstart-java/logs-analytics.png#lightbox)

## <a name="clean-up-resources"></a>Limpieza de recursos
En los pasos anteriores, creó recursos de Azure en un grupo de recursos. Si prevé que no va a necesitar estos recursos en el futuro, elimine el grupo de recursos del portal, para lo que debe ejecutar el siguiente comando en Cloud Shell:
```azurecli
az group delete --name <your resource group name; for example: hellospring-1558400876966-rg> --yes
```

En este inicio rápido ha aprendido a:

> [!div class="checklist"]
> * Generar un proyecto de Azure Spring Cloud básico
> * Aprovisionar una instancia de servicio
> * Compilar e implementar la aplicación con un punto de conexión público
> * Registros de streaming en tiempo real
## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Compilación y ejecución de microservicios](spring-cloud-quickstart-sample-app-introduction.md)

Hay más ejemplos disponibles en GitHub: [Ejemplos de Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/service-binding-cosmosdb-sql).
