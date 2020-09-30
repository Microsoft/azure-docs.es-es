---
title: 'Inicio rápido: Compilación e implementación de aplicaciones en Azure Spring Cloud'
description: Describe la implementación de aplicaciones en Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 08/03/2020
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 94caa879aa005f8f41e44b8a56400e87f6174247
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90908347"
---
# <a name="quickstart-build-and-deploy-apps-to-azure-spring-cloud"></a>Inicio rápido: Compilación e implementación de aplicaciones en Azure Spring Cloud

::: zone pivot="programming-language-csharp"
En esta guía de inicio rápido, se compilan e implementan aplicaciones de microservicios en Azure Spring Cloud mediante la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

* Complete las guías de inicio rápido anteriores de esta serie:

  * [Aprovisionamiento del servicio Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md).
  * [Configuración del servidor de configuración de Azure Spring Cloud](spring-cloud-quickstart-setup-config-server.md).

## <a name="download-the-sample-app"></a>Descarga de la aplicación de ejemplo

Si ha estado usando Azure Cloud Shell hasta este momento, cambie a un símbolo del sistema local para los pasos siguientes.

1. Cree una carpeta y clone el repositorio de la aplicación de ejemplo.

   ```console
   mkdir source-code
   ```

   ```console
   cd source-code
   ```

   ```console
   git clone https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples
   ```

1. Navegue al directorio del repositorio.

   ```console
   cd Azure-Spring-Cloud-Samples
   ```

## <a name="deploy-planetweatherprovider"></a>Implementación de PlanetWeatherProvider

1. Cree una aplicación para el proyecto PlanetWeatherProvider en la instancia de Azure Spring Cloud.

   ```azurecli
   az spring-cloud app create --name planet-weather-provider --runtime-version NetCore_31
   ```

   Para habilitar el registro automático de servicios, ha dado a la aplicación el mismo nombre que el valor de `spring.application.name` en el archivo *appsettings.json* del proyecto:

   ```json
   "spring": {
     "application": {
       "name": "planet-weather-provider"
     }
   }
   ```

   La ejecución de este comando puede llevar varios minutos.

1. Cambie el directorio a la carpeta del proyecto `PlanetWeatherProvider`.

   ```console
   cd steeltoe-sample/src/planet-weather-provider
   ```

1. Cree los archivos binarios y el archivo *.zip* que se va a implementar.

   ```console
   dotnet publish -c release -o ./publish
   ```

   > [!TIP]
   > El archivo del proyecto contiene el siguiente código XML para empaquetar los archivos binarios en un archivo *.zip* después de escribirlos en la carpeta *./publish*:
   >
   > ```xml
   > <Target Name="Publish-Zip" AfterTargets="Publish">
   >   <ZipDirectory SourceDirectory="$(PublishDir)" DestinationFile="$(MSBuildProjectDirectory)/publish-deploy-planet.zip" Overwrite="true" />
   > </Target>
   > ```

1. Implemente en Azure.

   Asegúrese de que el símbolo del sistema se encuentre en la carpeta del proyecto antes de ejecutar el siguiente comando.

   ```console
   az spring-cloud app deploy -n planet-weather-provider --runtime-version NetCore_31 --main-entry Microsoft.Azure.SpringCloud.Sample.PlanetWeatherProvider.dll --artifact-path ./publish-deploy-planet.zip
   ```

   La opción `--main-entry` especifica la ruta de acceso relativa de la carpeta raíz del archivo *.zip* al archivo *.dll* que contiene el punto de entrada de la aplicación. Una vez que el servicio carga el archivo *.zip*, extrae todos los archivos y carpetas e intenta ejecutar el punto de entrada en el archivo *.dll* especificado.

   La ejecución de este comando puede llevar varios minutos.

## <a name="deploy-solarsystemweather"></a>Implementación de SolarSystemWeather

1. Cree otra aplicación en la instancia de Azure Spring Cloud, esta vez para el proyecto SolarSystemWeather:

   ```azurecli
   az spring-cloud app create --name solar-system-weather --runtime-version NetCore_31
   ```

   `solar-system-weather` es el nombre que se especifica en el archivo *appsettings.json* del proyecto `SolarSystemWeather`.

   La ejecución de este comando puede llevar varios minutos.

1. Cambie el directorio al proyecto `SolarSystemWeather`.

   ```console
   cd ../solar-system-weather
   ```

1. Cree los archivos binarios y el archivo *.zip* que se va a implementar.

   ```console
   dotnet publish -c release -o ./publish
   ```

1. Implemente en Azure.

   ```console
   az spring-cloud app deploy -n solar-system-weather --runtime-version NetCore_31 --main-entry Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.dll --artifact-path ./publish-deploy-solar.zip
   ```
   
   La ejecución de este comando puede llevar varios minutos.

## <a name="assign-public-endpoint"></a>Asignación del punto de conexión público

Para probar la aplicación, envíe una solicitud HTTP GET a la aplicación `solar-system-weather` desde un explorador.  Para ello, necesita un punto de conexión público para la solicitud.

1. Para asignar el punto de conexión, ejecute el comando siguiente.

   ```azurecli
   az spring-cloud app update -n solar-system-weather --is-public true
   ```

1. Para obtener la dirección URL del punto de conexión, ejecute el comando siguiente.

   Windows:

   ```azurecli
   az spring-cloud app show -n solar-system-weather -o table
   ```

   Linux:

   ```azurecli
   az spring-cloud app show --name solar-system-weather | grep url
   ```

## <a name="test-the-application"></a>Prueba de la aplicación

Envíe una solicitud GET a la aplicación `solar-system-weather`. En un explorador, vaya a la dirección URL pública con `/weatherforecast` anexado al final de esta. Por ejemplo:

```
https://servicename-solar-system-weather.azuremicroservices.io/weatherforecast
```

El resultado JSON es:

```json
[{"Key":"Mercury","Value":"very warm"},{"Key":"Venus","Value":"quite unpleasant"},{"Key":"Mars","Value":"very cool"},{"Key":"Saturn","Value":"a little bit sandy"}]
```

Esta respuesta muestra que ambas aplicaciones de microservicio funcionan. La aplicación `SolarSystemWeather` devuelve los datos recuperados de la aplicación `PlanetWeatherProvider`.
::: zone-end

::: zone pivot="programming-language-java"
En este documento se explica cómo compilar e implementar aplicaciones de microservicios en Azure Spring Cloud mediante:
* la CLI de Azure
* Complemento de Maven
* Intellij

Antes de realizar la implementación mediante la CLI de Azure o Maven, complete los ejemplos que [aprovisionan una instancia de Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md) y [configure el servidor de configuración](spring-cloud-quickstart-setup-config-server.md).

## <a name="prerequisites"></a>Requisitos previos

* [Instalación de JDK 8](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable&preserve-view=true)
* [Registro para obtener una suscripción a Azure](https://azure.microsoft.com/free/)
* (Opcional) [Instale la versión 2.0.67 de la CLI de Azure, o cualquier versión superior](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true), e instale la extensión de Azure Spring Cloud con el comando `az extension add --name spring-cloud`
* (Opcional) [Instale Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) e [inicie sesión](https://docs.microsoft.com/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

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

1. Genere configuraciones mediante la ejecución del siguiente comando en la carpeta raíz de PiggyMetrics que contiene el POM primario. Si ya ha iniciado sesión con la CLI de Azure, el comando tomará automáticamente las credenciales. Si no lo ha hecho, le proporcionará instrucciones para iniciar sesión. Para obtener más información, consulte nuestra [página wiki](https://github.com/microsoft/azure-maven-plugins/wiki/Authentication).

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

1. En el campo **Nombre**, anexe *:gateway* al **Nombre** existente.
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

Vaya a la dirección URL que se proporciona en la salida de los pasos anteriores para acceder a la aplicación PiggyMetrics. Por ejemplo: `https://<service instance name>-gateway.azuremicroservices.io`

![Acceso a PiggyMetrics](media/spring-cloud-quickstart-launch-app-cli/launch-app.png)

También puede navegar por el Azure Portal para buscar la dirección URL. 
1. Vaya al servicio.
2. Seleccione **Aplicaciones**.
3. Seleccione **gateway**.

    ![Desplazamiento a la aplicación](media/spring-cloud-quickstart-launch-app-cli/navigate-app1.png)
    
4. Busque la dirección URL en la página **puerta de enlace | Información general**.

    ![Desplazamiento a la segunda aplicación](media/spring-cloud-quickstart-launch-app-cli/navigate-app2-url.png)

::: zone-end

## <a name="clean-up-resources"></a>Limpieza de recursos

Si desea continuar con la siguiente guía de inicio rápido de esta serie, omita este paso.

En estas guías de inicio rápido, ha creado recursos de Azure que seguirán acumulando cargos si permanecen en la suscripción. Si no prevé continuar con la siguiente guía de inicio rápido y espera que no va a necesitar estos recursos en el futuro, elimine el grupo de recursos desde el portal o con el siguiente comando en Cloud Shell:

```azurecli
az group delete --name <your resource group name; for example: helloworld-1558400876966-rg> --yes
```

En una guía de inicio rápido anterior, también estableció el nombre del grupo de recursos predeterminado. Si no desea continuar con la siguiente guía de inicio rápido, borre ese nombre predeterminado con el siguiente comando de la CLI:

```azurecli
az configure --defaults group=
```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Registros, métricas y seguimiento](spring-cloud-quickstart-logs-metrics-tracing.md)

