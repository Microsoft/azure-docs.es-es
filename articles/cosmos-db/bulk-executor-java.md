---
title: Uso de la biblioteca Bulk Executor de Java en Azure Cosmos DB para realizar operaciones de actualización e importación en bloque
description: Actualización e importación en bloque de documentos de Azure Cosmos DB mediante la biblioteca Bulk Executor de Java
author: tknandu
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: java
ms.topic: how-to
ms.date: 08/26/2020
ms.author: ramkris
ms.reviewer: sngun
ms.custom: devx-track-java
ms.openlocfilehash: a9501df45d598c85f8c694c5d07db4f959615c00
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90968179"
---
# <a name="use-bulk-executor-java-library-to-perform-bulk-operations-on-azure-cosmos-db-data"></a>Uso de la biblioteca BulkExecutor en Java para realizar operaciones en masa con datos de Azure Cosmos DB

En este tutorial se proporcionan instrucciones sobre cómo usar la biblioteca BulkExecutor en Java de Azure Cosmos DB para importar y actualizar documentos de Azure Cosmos DB. Para información sobre la biblioteca BulkExecutor y cómo lo ayuda a aprovechar el almacenamiento y el rendimiento masivo, consulte el artículo de [información general sobre la biblioteca BulkExecutor](bulk-executor-overview.md). En este tutorial, va a compilar una aplicación de Java que genera documentos aleatorios que se importan en bloque en un contenedor de Azure Cosmos. Tras la importación, se actualizarán en masa algunas propiedades de un documento. 

Actualmente, la biblioteca Bulk Executor solo es compatible con las cuentas de API de SQL de Azure Cosmos DB y Gremlin API. En este artículo se describe cómo usar la biblioteca Bulk Executor de Java con las cuentas de API de SQL. Para obtener información acerca de cómo utilizar la biblioteca Bulk Executor de .NET con Gremlin API, consulte [realizar operaciones en masa en Gremlin API de Azure Cosmos DB](bulk-executor-graph-dotnet.md). La biblioteca Bulk Executor descrita está disponible únicamente para la [versión 2 del SDK de Java sincrónico de Azure Cosmos DB](sql-api-sdk-java.md) y es la solución recomendada actualmente para la compatibilidad de Bulk en Java. No está disponible actualmente para las versiones 3.x, 4.x u otras versiones superiores del SDK.

## <a name="prerequisites"></a>Requisitos previos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.  

* También puede [probar gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin ninguna suscripción a Azure, de forma gratuita y sin compromiso. O bien, puede usar el [emulador de Azure Cosmos DB](/azure/cosmos-db/local-emulator) con el punto de conexión `https://localhost:8081`. La clave principal se proporciona en [Authenticating requests](local-emulator.md#authenticate-requests) (Autenticación de solicitudes).  

* [Kit de desarrollo de Java (JDK) 1.7+](/java/azure/jdk/?view=azure-java-stable&preserve-view=true)  
  - En Ubuntu, ejecute `apt-get install default-jdk` para instalar el JDK.  

  - Asegúrese de establecer la variable de entorno JAVA_HOME para que apunte a la carpeta donde está instalado el JDK.

* [Descargar](https://maven.apache.org/download.cgi) e [instalar](https://maven.apache.org/install.html) un archivo binario de [Maven](https://maven.apache.org/)  
  
  - En Ubuntu, puede ejecutar `apt-get install maven` para instalar Maven.

* Cree una cuenta de API de SQL de Azure Cosmos DB mediante los pasos descritos en la sección de [creación de una cuenta de base de datos](create-sql-api-java.md#create-a-database-account) del artículo de inicio rápido de Java.

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a pasar a trabajar con el código mediante la descarga de una aplicación de Java de ejemplo de GitHub. Esta aplicación realiza operaciones en masa con los datos de Azure Cosmos DB. Para clonar la aplicación, abra un símbolo del sistema, vaya al directorio donde quiere copiar la aplicación y ejecute el siguiente comando:

```bash
 git clone https://github.com/Azure/azure-cosmosdb-bulkexecutor-java-getting-started.git 
```

El repositorio clonado contiene dos ejemplos "bulkimport" y "bulkupdate" relacionados con la carpeta "\azure-cosmosdb-bulkexecutor-java-getting-started\samples\bulkexecutor-sample\src\main\java\com\microsoft\azure\cosmosdb\bulkexecutor". La aplicación "bulkimport" genera documentos aleatorios y los importa en Azure Cosmos DB. La aplicación "bulkupdate" actualiza algunos documentos en Azure Cosmos DB. En las secciones siguientes, se revisará el código en cada una de estas aplicaciones de ejemplo. 

## <a name="bulk-import-data-to-azure-cosmos-db"></a>Importación de datos en masa a Azure Cosmos DB

1. Las cadenas de conexión de Azure Cosmos DB se leen como argumentos y se asignan a variables definidas en el archivo CmdLineConfiguration.java.  

2. Después, se inicializa el objeto DocumentClient con el uso de las siguientes instrucciones:  

   ```java
   ConnectionPolicy connectionPolicy = new ConnectionPolicy();
   connectionPolicy.setMaxPoolSize(1000);
   DocumentClient client = new DocumentClient(
      HOST,
      MASTER_KEY, 
      connectionPolicy,
      ConsistencyLevel.Session)
   ```

3. El objeto DocumentBulkExecutor se inicializa con un valor alto de reintentos para el tiempo de espera y las solicitudes limitadas. Además, se establecen en 0 para pasar el control de congestión de DocumentBulkExecutor durante su vigencia.  

   ```java
   // Set client's retry options high for initialization
   client.getConnectionPolicy().getRetryOptions().setMaxRetryWaitTimeInSeconds(30);
   client.getConnectionPolicy().getRetryOptions().setMaxRetryAttemptsOnThrottledRequests(9);

   // Builder pattern
   Builder bulkExecutorBuilder = DocumentBulkExecutor.builder().from(
     client,
     DATABASE_NAME,
     COLLECTION_NAME,
     collection.getPartitionKey(),
     offerThroughput) // throughput you want to allocate for bulk import out of the container's total throughput

   // Instantiate DocumentBulkExecutor
   DocumentBulkExecutor bulkExecutor = bulkExecutorBuilder.build()

   // Set retries to 0 to pass complete control to bulk executor
   client.getConnectionPolicy().getRetryOptions().setMaxRetryWaitTimeInSeconds(0);
   client.getConnectionPolicy().getRetryOptions().setMaxRetryAttemptsOnThrottledRequests(0);
   ```

4. Llame a la API importAll que genera documentos aleatorios para realizar importaciones en bloque en un contenedor de Azure Cosmos. Puede configurar las configuraciones de la línea de comandos en el archivo CmdLineConfiguration.java.

   ```java
   BulkImportResponse bulkImportResponse = bulkExecutor.importAll(documents, false, true, null);
   ```
   La API de importación en bloque acepta una colección de documentos JSON serializados y presenta la siguiente sintaxis; para más detalles, vea la [documentación de la API](/java/api/com.microsoft.azure.documentdb.bulkexecutor):

   ```java
   public BulkImportResponse importAll(
        Collection<String> documents,
        boolean isUpsert,
        boolean disableAutomaticIdGeneration,
        Integer maxConcurrencyPerPartitionRange) throws DocumentClientException;   
   ```

   El método importAll acepta los siguientes parámetros:
 
   |**Parámetro**  |**Descripción**  |
   |---------|---------|
   |isUpsert    |   Una marca para habilitar upsert en los documentos. Si ya existe un documento con un identificador determinado, este se actualiza.  |
   |disableAutomaticIdGeneration     |   Una marca para deshabilitar la generación automática de identificador. De forma predeterminada, se establece en true.   |
   |maxConcurrencyPerPartitionRange    |  El grado máximo de simultaneidad por rango con clave de partición. El valor predeterminado es 20.  |

   **Definición del objeto de respuesta de importación en bloque** El resultado de la llamada API de importación en bloque contiene los siguientes métodos GET:

   |**Parámetro**  |**Descripción**  |
   |---------|---------|
   |int getNumberOfDocumentsImported()  |   El número total de documentos importados correctamente aparte de los documentos proporcionados a la llamada API de importación en bloque.      |
   |double getTotalRequestUnitsConsumed()   |  Total de unidades de solicitud (RU) consumidas por la llamada API de importación en bloque.       |
   |Duration getTotalTimeTaken()   |    El tiempo total que tarda la llamada API de importación en bloque en completar la ejecución.     |
   |List\<Exception> getErrors() |  Obtiene la lista de errores si algunos documentos fuera del lote proporcionados a la llamada API de importación en bloque no se pudieron insertar.       |
   |List\<Object> getBadInputDocuments()  |    La lista de documentos con formato incorrecto no importados correctamente en la llamada API de importación en bloque. El usuario debe corregir los documentos devueltos y reintentar la importación. Los documentos con formato incorrecto incluyen documentos cuyo valor de identificador no es una cadena (null o cualquier otro tipo de datos se consideran no válidos).     |

5. Una vez que la aplicación de importación en bloque está lista, compile la herramienta de línea de comandos desde el origen con el comando "mvn clean package". Este comando genera un archivo jar en la carpeta de destino:  

   ```bash
   mvn clean package
   ```

6. Una vez generadas las dependencias de destino, puede invocar la aplicación de importador en bloque con el siguiente comando:  

   ```bash
   java -Xmx12G -jar bulkexecutor-sample-1.0-SNAPSHOT-jar-with-dependencies.jar -serviceEndpoint *<Fill in your Azure Cosmos DB's endpoint>*  -masterKey *<Fill in your Azure Cosmos DB's primary key>* -databaseId bulkImportDb -collectionId bulkImportColl -operation import -shouldCreateCollection -collectionThroughput 1000000 -partitionKey /profileid -maxConnectionPoolSize 6000 -numberOfDocumentsForEachCheckpoint 1000000 -numberOfCheckpoints 10
   ```

   El importador en bloque crea una base de datos y una colección con el nombre de la base de datos, el nombre de la colección y los valores de la capacidad de proceso especificados en el archivo App.config. 

## <a name="bulk-update-data-in-azure-cosmos-db"></a>Actualización de datos en masa en Azure Cosmos DB

Puede actualizar los documentos existentes con el uso de la API BulkUpdateAsync. En este ejemplo, se establece el nuevo valor en el campo de nombre y se elimina el campo de descripción de los documentos existentes. Para consultar el conjunto completo de operaciones admitidas de actualización de campos, vea la [documentación de la API](/java/api/com.microsoft.azure.documentdb.bulkexecutor). 

1. Define los elementos de actualización junto con las operaciones de actualización de campos correspondientes. En este ejemplo, usará SetUpdateOperation para actualizar el campo de nombre y UnsetUpdateOperation para eliminar el campo de descripción de todos los documentos. También puede realizar otras operaciones, como incrementar un campo de documento con un valor determinado, insertar valores específicos en un campo de matriz o quitar un valor concreto de un campo de matriz. Para obtener información sobre los distintos métodos proporcionados por la API de actualización en masa, vea la [documentación de la API](/java/api/com.microsoft.azure.documentdb.bulkexecutor).  

   ```java
   SetUpdateOperation<String> nameUpdate = new SetUpdateOperation<>("Name","UpdatedDocValue");
   UnsetUpdateOperation descriptionUpdate = new UnsetUpdateOperation("description");

   ArrayList<UpdateOperationBase> updateOperations = new ArrayList<>();
   updateOperations.add(nameUpdate);
   updateOperations.add(descriptionUpdate);

   List<UpdateItem> updateItems = new ArrayList<>(cfg.getNumberOfDocumentsForEachCheckpoint());
   IntStream.range(0, cfg.getNumberOfDocumentsForEachCheckpoint()).mapToObj(j -> {                        
    return new UpdateItem(Long.toString(prefix + j), Long.toString(prefix + j), updateOperations);
    }).collect(Collectors.toCollection(() -> updateItems));
   ```

2. Llame a la API updateAll que genera documentos aleatorios para después importarlos en bloque en un contenedor de Azure Cosmos. Puede configurar los valores de la línea de comandos para pasarlos al archivo CmdLineConfiguration.java.

   ```java
   BulkUpdateResponse bulkUpdateResponse = bulkExecutor.updateAll(updateItems, null)
   ```

   La API de actualización en masa acepta la actualización de una colección de elementos. Cada elemento de actualización especifica la lista de las operaciones de actualización de campos que se realizarán en un documento que se identifica mediante un identificador y un valor con clave de partición. Para más información, vea la [documentación de la API](/java/api/com.microsoft.azure.documentdb.bulkexecutor):

   ```java
   public BulkUpdateResponse updateAll(
        Collection<UpdateItem> updateItems,
        Integer maxConcurrencyPerPartitionRange) throws DocumentClientException;
   ```

   El método updateAll acepta los siguientes parámetros:

   |**Parámetro** |**Descripción** |
   |---------|---------|
   |maxConcurrencyPerPartitionRange   |  El grado máximo de simultaneidad por rango con clave de partición. El valor predeterminado es 20.  |
 
   **Definición del objeto de respuesta de importación en bloque** El resultado de la llamada API de importación en bloque contiene los siguientes métodos GET:

   |**Parámetro** |**Descripción**  |
   |---------|---------|
   |int getNumberOfDocumentsUpdated()  |   El número total de documentos actualizados correctamente aparte de los documentos proporcionados a la llamada API de actualización en masa.      |
   |double getTotalRequestUnitsConsumed() |  Total de unidades de solicitud (RU) consumidas por la llamada API de actualización en masa.       |
   |Duration getTotalTimeTaken()  |   El tiempo total que tarda la llamada API de actualización en masa en completar la ejecución.      |
   |List\<Exception> getErrors()   |       Obtiene la lista de problemas operativos o de redes relacionados con la operación de actualización.      |
   |List\<BulkUpdateFailure> getFailedUpdates()   |       Obtiene la lista de actualizaciones que no se pudieron completar junto con las excepciones específicas que provocan los errores.|

3. Una vez que la aplicación de actualización masiva está lista, compile la herramienta de línea de comandos desde el origen con el comando "mvn clean package". Este comando genera un archivo jar en la carpeta de destino:  

   ```bash
   mvn clean package
   ```

4. Una vez generadas las dependencias de destino, puede invocar la aplicación de actualización en masa con el siguiente comando:

   ```bash
   java -Xmx12G -jar bulkexecutor-sample-1.0-SNAPSHOT-jar-with-dependencies.jar -serviceEndpoint **<Fill in your Azure Cosmos DB's endpoint>* -masterKey **<Fill in your Azure Cosmos DB's primary key>* -databaseId bulkUpdateDb -collectionId bulkUpdateColl -operation update -collectionThroughput 1000000 -partitionKey /profileid -maxConnectionPoolSize 6000 -numberOfDocumentsForEachCheckpoint 1000000 -numberOfCheckpoints 10
   ```

## <a name="performance-tips"></a>Consejos de rendimiento 

Tenga en cuenta los siguientes puntos para mejorar el rendimiento al utilizar la biblioteca BulkExecutor:

* Para obtener el mejor rendimiento, ejecute la aplicación desde una máquina virtual de Azure en la misma región que la región de escritura de la cuenta de Cosmos DB.  
* Para lograr mayor rendimiento:  

   * Establezca el tamaño del montón de JVM en un número lo suficientemente alto para evitar cualquier problema de memoria al controlar grandes volúmenes de documentos. Tamaño de montón sugerido: máx.([3 GB, 3 * tamaño de(todos los documentos pasados a la API de importación en bloque en un lote)].  
   * Hay un tiempo de procesamiento, por el que se obtendrá un mayor rendimiento al realizar operaciones en masa con un gran número de documentos. Por tanto, si va a importar 10 000 000 documentos, es preferible ejecutar la importación en bloque diez veces de diez documentos masivos, cada uno de un tamaño de 1 000 0000, en lugar de ejecutar cien veces la importación en bloque de cien documentos masivos, cada uno de ellos de una tamaño de 100 000 documentos.  

* Se recomienda crear instancias de un único objeto DocumentBulkExecutor en toda la aplicación dentro de una sola máquina virtual que se corresponda a un contenedor específico de Azure Cosmos.  

* Esto se debe a que una única ejecución de API de operaciones en masa consume un gran fragmento de E/S de red y de CPU del equipo cliente. Esto sucede al generar varias tareas internamente y al evitar la creación de varias tareas simultáneas dentro de su proceso de aplicación, donde cada una ejecuta llamadas API de operaciones en masa. Si una única llamada API de operaciones en bloque en una única máquina virtual no puede consumir la capacidad de proceso de todo el contenedor (si la capacidad de proceso del contenedor es superior a 1 millón RU/s), es preferible crear máquinas virtuales independientes para ejecutar llamadas API de operaciones en bloque simultáneamente.

    
## <a name="next-steps"></a>Pasos siguientes
* Para más información sobre el paquete Maven y las notas de la versión de la biblioteca BulkExecutor en Java, vea la [documentación sobre el SDK de BulkExecutor](sql-api-sdk-bulk-executor-java.md).


