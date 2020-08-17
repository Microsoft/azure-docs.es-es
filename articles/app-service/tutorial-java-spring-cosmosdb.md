---
title: 'Tutorial: Aplicación Java en Linux con MongoDB'
description: Aprenda a poner en funcionamiento una aplicación Java en Linux controlada por datos en Azure App Service, con conexión a MongoDB que se ejecuta en Azure (Cosmos DB).
author: rloutlaw
ms.author: routlaw
ms.devlang: java
ms.topic: tutorial
ms.date: 12/10/2018
ms.custom: mvc, seodec18, seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 0018e952516e08d6d01e2e79fea038745acadf62
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88082496"
---
# <a name="tutorial-build-a-java-spring-boot-web-app-with-azure-app-service-on-linux-and-azure-cosmos-db"></a>Tutorial: Compilación de una aplicación web de Java Spring Boot con Azure App Service en Linux y Azure Cosmos DB

Este tutorial lo guiará en el proceso de compilación, configuración, implementación y escalado de aplicaciones web de Java en Azure. Cuando termine, tendrá una aplicación [Spring Boot](https://projects.spring.io/spring-boot/) que almacena datos en [Azure Cosmos DB](/azure/cosmos-db) y que se ejecuta en [Azure App Service en Linux](overview.md).

![La aplicación Spring Boot almacena datos en Azure Cosmos DB](./media/tutorial-java-spring-cosmosdb/spring-todo-app-running-locally.jpg)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una base de datos de Cosmos DB.
> * Conectar una aplicación de ejemplo a la base de datos y probarla de manera local.
> * Implementar la aplicación de ejemplo en Azure.
> * Transmitir registros de diagnóstico desde App Service.
> * Agregar instancias adicionales para escalar horizontalmente la aplicación de ejemplo.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

* La [CLI de Azure](https://docs.microsoft.com/cli/azure/overview), instalada en su propio equipo. 
* [Git](https://git-scm.com/)
* [Java JDK](https://aka.ms/azure-jdks)
* [Maven](https://maven.apache.org)

## <a name="clone-the-sample-todo-app-and-prepare-the-repo"></a>Clonación de la aplicación de tareas pendientes de ejemplo y preparación del repositorio

En este tutorial se usa una aplicación de lista de tareas pendientes de ejemplo con una interfaz de usuario web que llama a una API REST de Spring con el respaldo de [Azure Cosmos DB Spring Data](https://github.com/Microsoft/spring-data-cosmosdb). El código de la aplicación está disponible [en GitHub](https://github.com/Microsoft/spring-todo-app). Para más información sobre cómo escribir aplicaciones Java mediante Spring y Cosmos DB, consulte el [tutorial de Spring Boot Starter con SQL API de Azure Cosmos DB](https://docs.microsoft.com/java/azure/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db ) y el [inicio rápido de Azure Cosmos DB Spring Data ](https://github.com/Microsoft/spring-data-cosmosdb#quick-start).


Ejecute los comandos siguientes en su terminal para clonar el repositorio de ejemplo y configurar el entorno de aplicación de ejemplo.

```bash
git clone --recurse-submodules https://github.com/Azure-Samples/e2e-java-experience-in-app-service-linux-part-2.git
cd e2e-java-experience-in-app-service-linux-part-2
yes | cp -rf .prep/* .
```

## <a name="create-an-azure-cosmos-db"></a>Creación de una base de datos de Azure Cosmos DB

Siga estos pasos para crear una base de datos de Azure Cosmos DB en la suscripción. La aplicación de lista de tareas pendientes se conectará a esta base de datos y almacenará sus datos al ejecutarse, conservando el estado de la aplicación independiente de dónde se ejecute la aplicación.

1. Inicie sesión en la CLI de Azure y, de manera opcional, establezca la suscripción si tiene más de una conectada en las credenciales de inicio de sesión.

    ```bash
    az login
    az account set -s <your-subscription-id>
    ```   

2. Cree un grupo de recursos de Azure, teniendo en cuenta el nombre del grupo de recursos.

    ```bash
    az group create -n <your-azure-group-name> \
        -l <your-resource-group-region>
    ```

3. Cree la base de datos de Azure Cosmos DB con el tipo `GlobalDocumentDB`. Solo debe usar letras minúsculas en el nombre de Cosmos DB. Anote el campo `documentEndpoint` en la respuesta del comando.

    ```bash
    az cosmosdb create --kind GlobalDocumentDB \
        -g <your-azure-group-name> \
        -n <your-azure-COSMOS-DB-name-in-lower-case-letters>
    ```

4. Obtenga su clave de Azure Cosmos DB para conectarse a la aplicación. Tenga `primaryMasterKey`, `documentEndpoint` a mano, porque los necesitará en el próximo paso.

    ```bash
    az cosmosdb list-keys -g <your-azure-group-name> -n <your-azure-COSMOSDB-name>
    ```

## <a name="configure-the-todo-app-properties"></a>Configuración de las propiedades de la aplicación de tareas pendientes

Abra un terminal en el equipo. Copie el archivo de script de ejemplo en el repositorio clonado para que pueda personalizarlo para la base de datos de Cosmos DB que acaba de crear.

```bash
cd initial/spring-todo-app
cp set-env-variables-template.sh .scripts/set-env-variables.sh
```
 
Edite `.scripts/set-env-variables.sh` en el editor que prefiera y suministre información de conexión de Azure Cosmos DB. En la configuración de App Service Linux, use la misma región que usó antes (`your-resource-group-region`) y el grupo de recursos (`your-azure-group-name`) usado al crear la base de datos de Cosmos DB. Elija un WEBAPP_NAME único, porque no se puede duplicar ningún nombre de aplicación web de ninguna implementación de Azure.

```bash
export COSMOSDB_URI=<put-your-COSMOS-DB-documentEndpoint-URI-here>
export COSMOSDB_KEY=<put-your-COSMOS-DB-primaryMasterKey-here>
export COSMOSDB_DBNAME=<put-your-COSMOS-DB-name-here>

# App Service Linux Configuration
export RESOURCEGROUP_NAME=<put-your-resource-group-name-here>
export WEBAPP_NAME=<put-your-Webapp-name-here>
export REGION=<put-your-REGION-here>
```

Luego, ejecute el script:

```bash
source .scripts/set-env-variables.sh
```
   
Estas variables de entorno se usan en `application.properties` de la aplicación de lista de tareas pendientes. Los campos de los archivos de propiedades establecen una configuración de repositorio predeterminado para Spring Data:

```properties
azure.cosmosdb.uri=${COSMOSDB_URI}
azure.cosmosdb.key=${COSMOSDB_KEY}
azure.cosmosdb.database=${COSMOSDB_DBNAME}
```

```java
@Repository
public interface TodoItemRepository extends DocumentDbRepository<TodoItem, String> {
}
```

Luego, la aplicación de ejemplo usa la anotación `@Document` importada desde `com.microsoft.azure.spring.data.cosmosdb.core.mapping.Document` para configurar un tipo de entidad que Cosmos DB almacene y administre:

```java
@Document
public class TodoItem {
    private String id;
    private String description;
    private String owner;
    private boolean finished;
```

## <a name="run-the-sample-app"></a>Ejecutar la aplicación de ejemplo

Use Maven para ejecutar el ejemplo.

```bash
mvn package spring-boot:run
```

La salida debe tener un aspecto similar al siguiente.

```bash
bash-3.2$ mvn package spring-boot:run
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building spring-todo-app 2.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 


[INFO] SimpleUrlHandlerMapping - Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[INFO] SimpleUrlHandlerMapping - Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
[INFO] WelcomePageHandlerMapping - Adding welcome page: class path resource [static/index.html]
2018-10-28 15:04:32.101  INFO 7673 --- [           main] c.m.azure.documentdb.DocumentClient      : Initializing DocumentClient with serviceEndpoint [https://sample-cosmos-db-westus.documents.azure.com:443/], ConnectionPolicy [ConnectionPolicy [requestTimeout=60, mediaRequestTimeout=300, connectionMode=Gateway, mediaReadMode=Buffered, maxPoolSize=800, idleConnectionTimeout=60, userAgentSuffix=;spring-data/2.0.6;098063be661ab767976bd5a2ec350e978faba99348207e8627375e8033277cb2, retryOptions=com.microsoft.azure.documentdb.RetryOptions@6b9fb84d, enableEndpointDiscovery=true, preferredLocations=null]], ConsistencyLevel [null]
[INFO] AnnotationMBeanExporter - Registering beans for JMX exposure on startup
[INFO] TomcatWebServer - Tomcat started on port(s): 8080 (http) with context path ''
[INFO] TodoApplication - Started TodoApplication in 45.573 seconds (JVM running for 76.534)
```

Puede tener acceso a la aplicación de tareas pendientes de Spring de manera local con este vínculo una vez que se inicia la aplicación: `http://localhost:8080/`.

 ![Acceso local a la aplicación de tareas pendientes de Spring](./media/tutorial-java-spring-cosmosdb/spring-todo-app-running-locally.jpg)

Si ve excepciones en lugar del mensaje "Started TodoApplication", compruebe que el script `bash` del paso anterior exportó correctamente las variables de entorno y que los valores son correctos para la base de datos de Azure Cosmos DB que creó.

## <a name="configure-azure-deployment"></a>Configuración de la implementación de Azure

Abra el archivo `pom.xml` del directorio `initial/spring-boot-todo` y agregue la configuración del [complemento de Maven para Azure App Service](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md) siguiente.

```xml    
<plugins> 

    <!--*************************************************-->
    <!-- Deploy to Java SE in App Service Linux           -->
    <!--*************************************************-->
       
    <plugin>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-webapp-maven-plugin</artifactId>
        <version>1.9.1</version>
        <configuration>
            <schemaVersion>v2</schemaVersion>

            <!-- Web App information -->
            <resourceGroup>${RESOURCEGROUP_NAME}</resourceGroup>
            <appName>${WEBAPP_NAME}</appName>
            <region>${REGION}</region>

            <!-- Java Runtime Stack for Web App on Linux-->
            <runtime>
                 <os>linux</os>
                 <javaVersion>jre8</javaVersion>
                 <webContainer>jre8</webContainer>
             </runtime>
             <deployment>
                 <resources>
                 <resource>
                     <directory>${project.basedir}/target</directory>
                     <includes>
                     <include>*.jar</include>
                     </includes>
                 </resource>
                 </resources>
             </deployment>

            <appSettings>
                <property>
                    <name>COSMOSDB_URI</name>
                    <value>${COSMOSDB_URI}</value>
                </property> 
                <property>
                    <name>COSMOSDB_KEY</name>
                    <value>${COSMOSDB_KEY}</value>
                </property>
                <property>
                    <name>COSMOSDB_DBNAME</name>
                    <value>${COSMOSDB_DBNAME}</value>
                </property>
                <property>
                    <name>JAVA_OPTS</name>
                    <value>-Dserver.port=80</value>
                </property>
            </appSettings>

        </configuration>
    </plugin>           
    ...
</plugins>
```

## <a name="deploy-to-app-service-on-linux"></a>Implementación en App Service en Linux

Use el objetivo `azure-webapp:deploy` de Maven para implementar la aplicación de tareas pendientes en Azure App Service en Linux.

```bash

# Deploy
bash-3.2$ mvn azure-webapp:deploy
[INFO] Scanning for projects...
[INFO] 
[INFO] ------------------------------------------------------------------------
[INFO] Building spring-todo-app 2.0-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- azure-webapp-maven-plugin:1.9.1:deploy (default-cli) @ spring-todo-app ---
[INFO] Auth Type : AZURE_CLI, Auth Files : [C:\Users\testuser\.azure\azureProfile.json, C:\Users\testuser\.azure\accessTokens.json]
[INFO] Subscription : xxxxxxxxx
[INFO] Target Web App doesn't exist. Creating a new one...
[INFO] Creating App Service Plan 'ServicePlanb6ba8178-5bbb-49e7'...
[INFO] Successfully created App Service Plan.
[INFO] Successfully created Web App.
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource to /home/test/e2e-java-experience-in-app-service-linux-part-2/initial/spring-todo-app/target/azure-webapp/spring-todo-app-61bb5207-6fb8-44c4-8230-c1c9e4c099f7
[INFO] Trying to deploy artifact to spring-todo-app...
[INFO] Renaming /home/test/e2e-java-experience-in-app-service-linux-part-2/initial/spring-todo-app/target/azure-webapp/spring-todo-app-61bb5207-6fb8-44c4-8230-c1c9e4c099f7/spring-todo-app-2.0-SNAPSHOT.jar to app.jar
[INFO] Deploying the zip package spring-todo-app-61bb5207-6fb8-44c4-8230-c1c9e4c099f7718326714198381983.zip...
[INFO] Successfully deployed the artifact to https://spring-todo-app.azurewebsites.net
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 02:19 min
[INFO] Finished at: 2019-11-06T15:32:03-07:00
[INFO] Final Memory: 50M/574M
[INFO] ------------------------------------------------------------------------
```

La salida incluye la dirección URL de la aplicación implementada (en este ejemplo, `https://spring-todo-app.azurewebsites.net`). Puede copiar esta dirección URL en el explorador web o ejecutar el comando siguiente en la ventana de terminal para cargar la aplicación.

```bash
open https://spring-todo-app.azurewebsites.net
```

Debería ver que la aplicación se ejecuta con la dirección URL remota en la barra de direcciones:

 ![La aplicación Spring Boot se ejecuta con una dirección URL remota](./media/tutorial-java-spring-cosmosdb/spring-todo-app-running-in-app-service.jpg)

## <a name="stream-diagnostic-logs"></a>Transmisión de registros de diagnóstico

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]


## <a name="scale-out-the-todo-app"></a>Escalado horizontal de la aplicación de tareas pendientes

Para agregar otro trabajo, escale horizontalmente la aplicación:

```bash
az appservice plan update --number-of-workers 2 \
   --name ${WEBAPP_PLAN_NAME} \
   --resource-group <your-azure-group-name>
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si no necesita estos recursos para otro tutorial (consulte [Pasos siguientes](#next)), ejecute el siguiente comando para eliminarlos en Cloud Shell: 
  
```bash
az group delete --name <your-azure-group-name>
```

<a name="next"></a>

## <a name="next-steps"></a>Pasos siguientes

[Azure para desarrolladores de Java](/java/azure/)
[Spring Boot](https://spring.io/projects/spring-boot), [Datos de Spring para Cosmos DB](/java/azure/spring-framework/configure-spring-boot-starter-java-app-with-cosmos-db?view=azure-java-stable), [Azure Cosmos DB](/azure/cosmos-db/sql-api-introduction) y [Azure App Service en Linux](overview.md).

Más información acerca de la ejecución de aplicaciones de Java en App Service en Linux en la guía del desarrollador.

> [!div class="nextstepaction"] 
> [Guía para desarrolladores de Java para App Service en Linux](configure-language-java.md?pivots=platform-linux)
