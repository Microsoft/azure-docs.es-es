---
title: 'Tutorial: Uso de funciones de Java con Azure Cosmos DB y Event Hubs'
description: En este tutorial se muestra cómo consumir eventos de Event Hubs para hacer realizar actualizaciones en Azure Cosmos DB mediante una función escrita en Java.
author: KarlErickson
ms.topic: tutorial
ms.date: 11/04/2019
ms.author: karler
ms.custom: devx-track-java
ms.openlocfilehash: ef2b774195f522be3520850d2e6e7193e70dd4fb
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87372457"
---
# <a name="tutorial-create-a-function-in-java-with-an-event-hub-trigger-and-an-azure-cosmos-db-output-binding"></a>Tutorial: Creación de una función en Java con un desencadenador de centro de eventos y un enlace de salida de Azure Cosmos DB

En este tutorial se muestra cómo usar Azure Functions para crear una función de Java que analice un flujo continuo de datos de temperatura y presión. Los eventos del centro de eventos que representan las lecturas de sensor desencadenan la función. La función procesa los datos del evento y, a continuación, agrega entradas de estado a una instancia de Azure Cosmos DB.

En este tutorial, hará lo siguiente:

> [!div class="checklist"]
> * Creación y configuración de recursos de Azure con la CLI de Azure.
> * Creación y prueba de las funciones Java que interactúan con estos recursos.
> * Implementación de sus funciones en Azure y supervisión con Application Insights.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará instalar los siguientes elementos:

* [Kit para desarrolladores de Java](https://aka.ms/azure-jdks), versión 8
* [Apache Maven](https://maven.apache.org), versión 3.0 o posterior
* [CLI de Azure](/cli/azure/install-azure-cli) si prefiere no usar Cloud Shell
* [Azure Functions Core Tools](https://www.npmjs.com/package/azure-functions-core-tools), versión 2.6.666 u otra posterior

> [!IMPORTANT]
> La variable de entorno `JAVA_HOME` se debe establecer en la ubicación de instalación del JDK para completar este tutorial.

Si prefiere usar directamente el código para este tutorial, consulte el repositorio de ejemplo [java-functions-eventhub-cosmosdb](https://github.com/Azure-Samples/java-functions-eventhub-cosmosdb).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-azure-resources"></a>Creación de recursos de Azure

En este tutorial, necesitará de estos recursos:

* Un grupo de recursos para contener a los demás recursos.
* Un espacio de nombres de Event Hubs, un centro de eventos y una regla de autorización.
* Una cuenta, una base de datos y una colección de Cosmos DB.
* Una aplicación de funciones y una cuenta de almacenamiento para hospedarla.

Las secciones siguientes muestran cómo crear los recursos mediante la CLI de Azure.

### <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Si no está usando Cloud Shell, deberá usar la CLI de Azure de forma local para acceder a su cuenta. Use el comando `az login` desde el símbolo del sistema de Bash para iniciar la experiencia de inicio de sesión basada en el explorador. Si tiene acceso a más de una suscripción a Azure, establezca la suscripción predeterminada con `az account set --subscription` seguido del Id. de suscripción.

### <a name="set-environment-variables"></a>Establecimiento de variables de entorno

A continuación, cree algunas variables de entorno para los nombres y ubicación de los recursos que va a crear. Use los comandos siguientes y reemplace los marcadores de posición `<value>` por los valores de su elección. Los valores deben cumplir con las [reglas y restricciones de nomenclatura para los recursos de Azure](/azure/architecture/best-practices/resource-naming). Para la variable `LOCATION`, use uno de los valores generados por el comando `az functionapp list-consumption-locations`.

```azurecli-interactive
RESOURCE_GROUP=<value>
EVENT_HUB_NAMESPACE=<value>
EVENT_HUB_NAME=<value>
EVENT_HUB_AUTHORIZATION_RULE=<value>
COSMOS_DB_ACCOUNT=<value>
STORAGE_ACCOUNT=<value>
FUNCTION_APP=<value>
LOCATION=<value>
```

En el resto de este tutorial se usan estas variables. Tenga en cuenta que estas variables se conservan solo mientras dure la sesión actual de la CLI de Azure o Cloud Shell. Tendrá que volver a ejecutar estos comandos si usa una ventana de terminal local diferente o se agota el tiempo de espera de la sesión de Cloud Shell.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Azure usa grupos de recursos para recopilar todos los recursos relacionados en su cuenta. De este modo, puede verlos como una unidad y eliminarlos con un solo comando cuando haya terminado con ellos.

Use el siguiente comando para crear un grupo de recursos:

```azurecli-interactive
az group create \
    --name $RESOURCE_GROUP \
    --location $LOCATION
```

### <a name="create-an-event-hub"></a>Creación de un centro de eventos

A continuación, cree un espacio de nombres de Azure Event Hubs, un centro de eventos y una regla de autorización mediante los siguientes comandos:

```azurecli-interactive
az eventhubs namespace create \
    --resource-group $RESOURCE_GROUP \
    --name $EVENT_HUB_NAMESPACE
az eventhubs eventhub create \
    --resource-group $RESOURCE_GROUP \
    --name $EVENT_HUB_NAME \
    --namespace-name $EVENT_HUB_NAMESPACE \
    --message-retention 1
az eventhubs eventhub authorization-rule create \
    --resource-group $RESOURCE_GROUP \
    --name $EVENT_HUB_AUTHORIZATION_RULE \
    --eventhub-name $EVENT_HUB_NAME \
    --namespace-name $EVENT_HUB_NAMESPACE \
    --rights Listen Send
```

El espacio de nombres de Event Hubs contiene el al centro de eventos real y su regla de autorización. La regla de autorización permite que las funciones envíen mensajes al centro y escuchen los eventos correspondientes. Una función envía mensajes que representan datos de telemetría. Otra función escucha los eventos, analiza los datos de evento y almacena los resultados en Azure Cosmos DB.

### <a name="create-an-azure-cosmos-db"></a>Creación de una base de datos de Azure Cosmos DB

A continuación, cree una cuenta, una base de datos y una colección de Azure Cosmos DB con los siguientes comandos:

```azurecli-interactive
az cosmosdb create \
    --resource-group $RESOURCE_GROUP \
    --name $COSMOS_DB_ACCOUNT
az cosmosdb database create \
    --resource-group-name $RESOURCE_GROUP \
    --name $COSMOS_DB_ACCOUNT \
    --db-name TelemetryDb
az cosmosdb collection create \
    --resource-group-name $RESOURCE_GROUP \
    --name $COSMOS_DB_ACCOUNT \
    --collection-name TelemetryInfo \
    --db-name TelemetryDb \
    --partition-key-path '/temperatureStatus'
```

El valor `partition-key-path` particiona los datos en función del valor `temperatureStatus` de cada elemento. La clave de partición permite que Cosmos DB aumente el rendimiento al dividir los datos en subconjuntos distintos a los que puede tener acceso de forma independiente.

### <a name="create-a-storage-account-and-function-app"></a>Creación de una cuenta de almacenamiento y una aplicación de funciones

A continuación, cree una cuenta de Azure Storage (que es necesaria en Azure Functions) y luego cree la aplicación de funciones. Use los comandos siguientes:

```azurecli-interactive
az storage account create \
    --resource-group $RESOURCE_GROUP \
    --name $STORAGE_ACCOUNT \
    --sku Standard_LRS
az functionapp create \
    --resource-group $RESOURCE_GROUP \
    --name $FUNCTION_APP \
    --storage-account $STORAGE_ACCOUNT \
    --consumption-plan-location $LOCATION \
    --runtime java
```

Cuando el comando `az functionapp create` crea la aplicación de funciones, también crea un recurso de Application Insights con el mismo nombre. La aplicación de funciones se configura automáticamente con un valor denominado `APPINSIGHTS_INSTRUMENTATIONKEY` que la conecta a Application Insights. Puede ver los datos de telemetría de la aplicación después de implementar las funciones en Azure, como se describe más adelante en este tutorial.

## <a name="configure-your-function-app"></a>Configuración de la aplicación de funciones

La aplicación de funciones tendrá que acceder a los demás recursos para funcionar correctamente. En las secciones siguientes se muestra cómo configurar la aplicación de funciones para que se pueda ejecutar en el equipo local.

### <a name="retrieve-resource-connection-strings"></a>Recuperación de las cadenas de conexión de recurso

Use los siguientes comandos para recuperar las cadenas de conexión del almacenamiento, el centro de eventos y Cosmos DB y guárdelas en variables de entorno:

```azurecli-interactive
AZURE_WEB_JOBS_STORAGE=$( \
    az storage account show-connection-string \
        --name $STORAGE_ACCOUNT \
        --query connectionString \
        --output tsv)
echo $AZURE_WEB_JOBS_STORAGE
EVENT_HUB_CONNECTION_STRING=$( \
    az eventhubs eventhub authorization-rule keys list \
        --resource-group $RESOURCE_GROUP \
        --name $EVENT_HUB_AUTHORIZATION_RULE \
        --eventhub-name $EVENT_HUB_NAME \
        --namespace-name $EVENT_HUB_NAMESPACE \
        --query primaryConnectionString \
        --output tsv)
echo $EVENT_HUB_CONNECTION_STRING
COSMOS_DB_CONNECTION_STRING=$( \
    az cosmosdb keys list \
        --resource-group $RESOURCE_GROUP \
        --name $COSMOS_DB_ACCOUNT \
        --type connection-strings \
        --query connectionStrings[0].connectionString \
        --output tsv)
echo $COSMOS_DB_CONNECTION_STRING
```

Estas variables se establecen en valores recuperados de comandos de la CLI de Azure. Cada comando usa una consulta JMESPath para extraer la cadena de conexión de la carga útil JSON devuelta. Las cadenas de conexión también se muestran mediante `echo` para que pueda confirmar que se recuperaron correctamente.

### <a name="update-your-function-app-settings"></a>Actualización de la configuración de la aplicación de funciones

A continuación, use el siguiente comando para transferir los valores de la cadena de conexión a la configuración de la aplicación en la cuenta de Azure Functions:

```azurecli-interactive
az functionapp config appsettings set \
    --resource-group $RESOURCE_GROUP \
    --name $FUNCTION_APP \
    --settings \
        AzureWebJobsStorage=$AZURE_WEB_JOBS_STORAGE \
        EventHubConnectionString=$EVENT_HUB_CONNECTION_STRING \
        CosmosDBConnectionString=$COSMOS_DB_CONNECTION_STRING
```

Ahora se han creado y configurado los recursos de Azure para funcionar correctamente.

## <a name="create-and-test-your-functions"></a>Creación y prueba de las funciones

A continuación, creará un proyecto en el equipo local, le agregará código Java y lo probará. Usará comandos que funcionan con el complemento de Maven para Azure Functions y Azure Functions Core Tools. Las funciones se ejecutarán localmente, pero usarán los recursos basados en la nube que ha creado. Cuando las funciones funcionen de forma local, puede usar Maven para implementarlas en la nube y observar cómo se acumulan los datos y los análisis.

Si usó Cloud Shell para crear los recursos, no se conectará a Azure de forma local. En este caso, use el comando `az login` para iniciar el proceso de inicio de sesión basado en el explorador. A continuación, si es necesario, establezca la suscripción predeterminada con `az account set --subscription` seguido del Id. de la suscripción. Por último, ejecute los siguientes comandos para volver a crear algunas variables de entorno en el equipo local. Reemplace los marcadores de posición `<value>` con los mismos valores que usó anteriormente.

```bash
RESOURCE_GROUP=<value>
FUNCTION_APP=<value>
```

### <a name="create-a-local-functions-project"></a>Creación de un proyecto local de Functions

Use el siguiente comando de Maven para crear un proyecto de funciones y agregar las dependencias necesarias.

```bash
mvn archetype:generate --batch-mode \
    -DarchetypeGroupId=com.microsoft.azure \
    -DarchetypeArtifactId=azure-functions-archetype \
    -DappName=$FUNCTION_APP \
    -DresourceGroup=$RESOURCE_GROUP \
    -DgroupId=com.example \
    -DartifactId=telemetry-functions
```

Este comando genera varios archivos dentro de una carpeta de `telemetry-functions`:

* Un archivo `pom.xml` para su uso con Maven.
* Un archivo `local.settings.json` para almacenar la configuración de la aplicación para las pruebas locales.
* Un archivo `host.json` que habilita el paquete de extensiones de Azure Functions, necesario para los enlaces de salida de Cosmos DB en la función de análisis de datos.
* Un archivo `Function.java` que incluye una implementación de funciones predeterminada.
* Algunos archivos de prueba que no necesita para este tutorial.

Para evitar errores de compilación, debe eliminar los archivos de prueba. Ejecute los siguientes comandos para ir a la nueva carpeta del proyecto y eliminar la carpeta de pruebas:

```bash
cd telemetry-functions
rm -r src/test
```

### <a name="retrieve-your-function-app-settings-for-local-use"></a>Recuperación de la configuración de la aplicación de funciones para uso local

En el caso de las pruebas locales, el proyecto de funciones necesitará las cadenas de conexión que agregó a la aplicación de funciones en Azure previamente en este tutorial. Use el siguiente comando de Azure Functions Core Tools, que recupera todas las opciones de configuración de la aplicación de funciones almacenadas en la nube y las agrega al archivo de `local.settings.json`:

```bash
func azure functionapp fetch-app-settings $FUNCTION_APP
```

### <a name="add-java-code"></a>Adición de código Java

Después, abra el archivo `Function.java` y reemplace el contenido por el código siguiente.

```java
package com.example;

import com.example.TelemetryItem.status;
import com.microsoft.azure.functions.annotation.Cardinality;
import com.microsoft.azure.functions.annotation.CosmosDBOutput;
import com.microsoft.azure.functions.annotation.EventHubOutput;
import com.microsoft.azure.functions.annotation.EventHubTrigger;
import com.microsoft.azure.functions.annotation.FunctionName;
import com.microsoft.azure.functions.annotation.TimerTrigger;
import com.microsoft.azure.functions.ExecutionContext;
import com.microsoft.azure.functions.OutputBinding;

public class Function {

    @FunctionName("generateSensorData")
    @EventHubOutput(
        name = "event",
        eventHubName = "", // blank because the value is included in the connection string
        connection = "EventHubConnectionString")
    public TelemetryItem generateSensorData(
        @TimerTrigger(
            name = "timerInfo",
            schedule = "*/10 * * * * *") // every 10 seconds
            String timerInfo,
        final ExecutionContext context) {

        context.getLogger().info("Java Timer trigger function executed at: "
            + java.time.LocalDateTime.now());
        double temperature = Math.random() * 100;
        double pressure = Math.random() * 50;
        return new TelemetryItem(temperature, pressure);
    }

    @FunctionName("processSensorData")
    public void processSensorData(
        @EventHubTrigger(
            name = "msg",
            eventHubName = "", // blank because the value is included in the connection string
            cardinality = Cardinality.ONE,
            connection = "EventHubConnectionString")
            TelemetryItem item,
        @CosmosDBOutput(
            name = "databaseOutput",
            databaseName = "TelemetryDb",
            collectionName = "TelemetryInfo",
            connectionStringSetting = "CosmosDBConnectionString")
            OutputBinding<TelemetryItem> document,
        final ExecutionContext context) {

        context.getLogger().info("Event hub message received: " + item.toString());

        if (item.getPressure() > 30) {
            item.setNormalPressure(false);
        } else {
            item.setNormalPressure(true);
        }

        if (item.getTemperature() < 40) {
            item.setTemperatureStatus(status.COOL);
        } else if (item.getTemperature() > 90) {
            item.setTemperatureStatus(status.HOT);
        } else {
            item.setTemperatureStatus(status.WARM);
        }

        document.setValue(item);
    }
}
```

Como puede ver, el archivo contiene dos funciones, `generateSensorData` y `processSensorData`. La función `generateSensorData` simula un sensor que envía lecturas de temperatura y presión al centro de eventos. Un desencadenador de temporizador ejecuta la función cada diez segundos y un enlace de salida del centro de eventos envía el valor devuelto al centro de eventos.

Cuando el centro de eventos recibe el mensaje, genera un evento. La función `processSensorData` se ejecuta cuando se recibe el evento. Después procesa los datos de evento y usa un enlace de salida de Azure Cosmos DB para enviar los resultados a Azure Cosmos DB.

Los datos utilizados por estas funciones se almacenan mediante una clase llamada `TelemetryItem`, que deberá implementar. Cree un nuevo archivo llamado `TelemetryItem.java` en la misma ubicación que `Function.java` y agregue el código siguiente:

```java
package com.example;

public class TelemetryItem {

    private String id;
    private double temperature;
    private double pressure;
    private boolean isNormalPressure;
    private status temperatureStatus;
    static enum status {
        COOL,
        WARM,
        HOT
    }

    public TelemetryItem(double temperature, double pressure) {
        this.temperature = temperature;
        this.pressure = pressure;
    }

    public String getId() {
        return id;
    }

    public double getTemperature() {
        return temperature;
    }

    public double getPressure() {
        return pressure;
    }

    @Override
    public String toString() {
        return "TelemetryItem={id=" + id + ",temperature="
            + temperature + ",pressure=" + pressure + "}";
    }

    public boolean isNormalPressure() {
        return isNormalPressure;
    }

    public void setNormalPressure(boolean isNormal) {
        this.isNormalPressure = isNormal;
    }

    public status getTemperatureStatus() {
        return temperatureStatus;
    }

    public void setTemperatureStatus(status temperatureStatus) {
        this.temperatureStatus = temperatureStatus;
    }
}
```

### <a name="run-locally"></a>Ejecución en modo local

Ahora puede compilar y ejecutar las funciones de forma local y ver cómo los datos aparecen en Azure Cosmos DB.

Use los siguientes comandos de Maven para compilar y ejecutar las funciones:

```bash
mvn clean package
mvn azure-functions:run
```

Después de algunos mensajes de compilación e inicio, verá una salida similar al ejemplo siguiente cada vez que se ejecuten las funciones:

```output
[10/22/19 4:01:30 AM] Executing 'Functions.generateSensorData' (Reason='Timer fired at 2019-10-21T21:01:30.0016769-07:00', Id=c1927c7f-4f70-4a78-83eb-bc077d838410)
[10/22/19 4:01:30 AM] Java Timer trigger function executed at: 2019-10-21T21:01:30.015
[10/22/19 4:01:30 AM] Function "generateSensorData" (Id: c1927c7f-4f70-4a78-83eb-bc077d838410) invoked by Java Worker
[10/22/19 4:01:30 AM] Executed 'Functions.generateSensorData' (Succeeded, Id=c1927c7f-4f70-4a78-83eb-bc077d838410)
[10/22/19 4:01:30 AM] Executing 'Functions.processSensorData' (Reason='', Id=f4c3b4d7-9576-45d0-9c6e-85646bb52122)
[10/22/19 4:01:30 AM] Event hub message received: TelemetryItem={id=null,temperature=32.728691307527015,pressure=10.122563042388165}
[10/22/19 4:01:30 AM] Function "processSensorData" (Id: f4c3b4d7-9576-45d0-9c6e-85646bb52122) invoked by Java Worker
[10/22/19 4:01:38 AM] Executed 'Functions.processSensorData' (Succeeded, Id=1cf0382b-0c98-4cc8-9240-ee2a2f71800d)
```

Después puede ir a [Azure Portal](https://portal.azure.com) y navegar hasta su cuenta de Azure Cosmos DB. Seleccione **Explorador de datos**, expanda **TelemetryInfo** y después seleccione **Elementos** para ver los datos cuando lleguen.

![Explorador de datos de Cosmos DB](media/functions-event-hub-cosmos-db/data-explorer.png)

## <a name="deploy-to-azure-and-view-app-telemetry"></a>Implementación en Azure y visualización de los datos de telemetría de la aplicación

Por último, puede implementar la aplicación en Azure y comprobar que sigue funcionando de la misma forma en que lo hacía localmente.

Implemente el proyecto en Azure mediante el siguiente comando:

```bash
mvn azure-functions:deploy
```

Ahora, las funciones se ejecutan en Azure y continúan acumulando datos en su instancia de Azure Cosmos DB. Puede ver la aplicación de funciones implementada en Azure Portal y ver los datos de telemetría de la aplicación a través del recurso de Application Insights conectado, tal como se muestra en las siguientes capturas de pantallas:

**Live Metrics Stream:**

![Live Metrics Stream de Application Insights](media/functions-event-hub-cosmos-db/application-insights-live-metrics-stream.png)

**Rendimiento:**

![Hoja de rendimiento de Application Insights](media/functions-event-hub-cosmos-db/application-insights-performance.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con los recursos de Azure que creó en este tutorial, puede eliminarlos con el siguiente comando:

```azurecli-interactive
az group delete --name $RESOURCE_GROUP
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprendió a crear una función de Azure que controla eventos del centro de eventos y actualiza una instancia de Cosmos DB. Para obtener más información, consulte la [Guía de Azure Functions para desarrolladores de Java](./functions-reference-java.md). Para obtener información sobre las anotaciones usadas, consulte la referencia [com.microsoft.azure.functions.annotation](/java/api/com.microsoft.azure.functions.annotation).

En este tutorial se usan las variables de entorno y la configuración de la aplicación para almacenar secretos como cadenas de conexión. Para obtener información sobre cómo almacenar estos secretos en Azure Key Vault, consulte [Uso de referencias de Key Vault para App Service y Azure Functions](../app-service/app-service-key-vault-references.md).

A continuación, aprenda a usar Azure Pipelines CI/CD para las implementaciones automatizadas:

> [!div class="nextstepaction"]
> [Compilación e implementación de Java en Azure Functions](/azure/devops/pipelines/ecosystems/java-function)
