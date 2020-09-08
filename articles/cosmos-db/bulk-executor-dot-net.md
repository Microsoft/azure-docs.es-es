---
title: Uso de la biblioteca Bulk Executor en .NET en Azure Cosmos DB para realizar operaciones de actualización e importación en bloque
description: Importe y actualice en bloque los documentos de Azure Cosmos DB mediante la biblioteca BulkExecutor en .NET.
author: tknandu
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.devlang: dotnet
ms.topic: how-to
ms.date: 03/23/2020
ms.author: ramkris
ms.reviewer: sngun
ms.custom: devx-track-csharp
ms.openlocfilehash: 829ddc0b63031722cdcb572a2833926a7837d05d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89004126"
---
# <a name="use-the-bulk-executor-net-library-to-perform-bulk-operations-in-azure-cosmos-db"></a>Uso de la biblioteca BulkExecutor en .NET para realizar operaciones en bloque en Azure Cosmos DB

> [!NOTE]
> La biblioteca de Bulk Executor que se describe en este artículo se mantiene para las aplicaciones que usan la versión 2.x del SDK de .NET. En el caso de las aplicaciones nuevas, puede usar la **compatibilidad con lotes** que está disponible directamente con el [SDK de .NET versión 3.x](tutorial-sql-api-dotnet-bulk-import.md) y no requiere ninguna biblioteca externa. 

> Si actualmente usa la biblioteca Bulk Executor y planea migrar a la compatibilidad con lotes del SDK más reciente, siga los pasos descritos en la [guía de migración](how-to-migrate-from-bulk-executor-library.md) para migrar la aplicación.

En este tutorial se proporcionan instrucciones sobre cómo usar la biblioteca BulkExecutor en .NET para importar y actualizar documentos en un contenedor de Azure Cosmos. Para información sobre la biblioteca BulkExecutor y cómo ayuda a aprovechar la capacidad masiva de almacenamiento y proceso, consulte el artículo de [información general sobre la biblioteca BulkExecutor](bulk-executor-overview.md). En este tutorial verá una aplicación de .NET de ejemplo que importa en bloque documentos generados de manera aleatoria en un contenedor de Azure Cosmos. Después de la importación, muestra cómo se pueden actualizar en masa los datos importados al especificar revisiones como operaciones que se deben realizar en campos de documentos específicos.

Actualmente, la biblioteca BulkExecutor solo es compatible con las cuentas de API de SQL y de Gremlin API de Azure Cosmos DB. En este artículo se describe cómo usar la biblioteca BulkExecutor en .NET con las cuentas de API de SQL. Para obtener información acerca de cómo utilizar la biblioteca BulkExecutor en .NET con cuentas de Gremlin API, consulte la información sobre la [realización de operaciones en bloque en Gremlin API de Azure Cosmos DB](bulk-executor-graph-dotnet.md).

## <a name="prerequisites"></a>Requisitos previos

* Si aún no tiene Visual Studio 2019 instalado, puede descargar y usar la versión [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Asegúrese de que habilita "Desarrollo de Azure" durante la instalación de Visual Studio.

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.

* También puede [probar gratis Azure Cosmos DB](https://azure.microsoft.com/try/cosmosdb/) sin una suscripción de Azure, de forma gratuita y sin compromiso. O bien, puede usar el [emulador de Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/local-emulator) con el punto de conexión `https://localhost:8081`. La clave principal se proporciona en [Authenticating requests](local-emulator.md#authenticating-requests) (Autenticación de solicitudes).

* Cree una cuenta de API de SQL de Azure Cosmos DB mediante los pasos descritos en la sección de [creación de una cuenta de base de datos](create-sql-api-dotnet.md#create-account) del artículo de inicio rápido de .NET.

## <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

Ahora vamos a pasar a trabajar con el código mediante la descarga de una aplicación de .NET de ejemplo de GitHub. Esta aplicación realiza operaciones en bloque en los datos almacenados en la cuenta de Azure Cosmos. Para clonar la aplicación, abra un símbolo del sistema, vaya al directorio donde quiere copiarla y ejecute el siguiente comando:

```bash
git clone https://github.com/Azure/azure-cosmosdb-bulkexecutor-dotnet-getting-started.git
```

El repositorio clonado contiene dos ejemplos: "BulkImportSample" y "BulkUpdateSample". Puede abrir cualquiera de las aplicaciones de ejemplo, actualizar las cadenas de conexión en un archivo App.config con las cadenas de conexión de la cuenta de Azure Cosmos DB, compilar la solución y ejecutarla.

La aplicación "BulkImportSample" genera documentos aleatorios y los importa en bloque en la cuenta de Azure Cosmos. La aplicación "BulkUpdateSample" realiza actualizaciones en masa de los documentos importados al especificar revisiones como operaciones que se deben realizar en campos de documentos específicos. En las secciones siguientes, se revisará el código en cada una de estas aplicaciones de ejemplo.

## <a name="bulk-import-data-to-an-azure-cosmos-account"></a>Importación en bloque de datos a una cuenta de Azure Cosmos

1. Navegue hasta la carpeta "BulkImportSample" y abra el archivo "BulkImportSample.sln".  

2. Las cadenas de conexión de Azure Cosmos DB se recuperan del archivo App.config como se muestra en el código siguiente:  

   ```csharp
   private static readonly string EndpointUrl = ConfigurationManager.AppSettings["EndPointUrl"];
   private static readonly string AuthorizationKey = ConfigurationManager.AppSettings["AuthorizationKey"];
   private static readonly string DatabaseName = ConfigurationManager.AppSettings["DatabaseName"];
   private static readonly string CollectionName = ConfigurationManager.AppSettings["CollectionName"];
   private static readonly int CollectionThroughput = int.Parse(ConfigurationManager.AppSettings["CollectionThroughput"]);
   ```

   El importador en bloque crea una base de datos y un contenedor con los valores de nombre de la base de datos, nombre del contenedor y capacidad de proceso especificados en el archivo App.config.

3. A continuación, se inicializa el objeto DocumentClient con el modo de conexión TCP directo:  

   ```csharp
   ConnectionPolicy connectionPolicy = new ConnectionPolicy
   {
      ConnectionMode = ConnectionMode.Direct,
      ConnectionProtocol = Protocol.Tcp
   };
   DocumentClient client = new DocumentClient(new Uri(endpointUrl),authorizationKey,
   connectionPolicy)
   ```

4. El objeto BulkExecutor se inicializa con un valor alto de reintentos para el tiempo de espera y las solicitudes limitadas. Además, se establecen en 0 para pasar el control de congestión de BulkExecutor durante su vigencia.  

   ```csharp
   // Set retry options high during initialization (default values).
   client.ConnectionPolicy.RetryOptions.MaxRetryWaitTimeInSeconds = 30;
   client.ConnectionPolicy.RetryOptions.MaxRetryAttemptsOnThrottledRequests = 9;

   IBulkExecutor bulkExecutor = new BulkExecutor(client, dataCollection);
   await bulkExecutor.InitializeAsync();

   // Set retries to 0 to pass complete control to bulk executor.
   client.ConnectionPolicy.RetryOptions.MaxRetryWaitTimeInSeconds = 0;
   client.ConnectionPolicy.RetryOptions.MaxRetryAttemptsOnThrottledRequests = 0;
   ```

5. La aplicación llama a la API BulkImportAsync. La biblioteca de .NET ofrece dos sobrecargas de la API de importación en bloque; una que acepta una lista de documentos JSON serializados y otra que acepta una lista de documentos POCO deserializados. Para más información sobre las definiciones de cada uno de estos métodos sobrecargados, consulte la [documentación de la API](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkexecutor.bulkimportasync?view=azure-dotnet).

   ```csharp
   BulkImportResponse bulkImportResponse = await bulkExecutor.BulkImportAsync(
     documents: documentsToImportInBatch,
     enableUpsert: true,
     disableAutomaticIdGeneration: true,
     maxConcurrencyPerPartitionKeyRange: null,
     maxInMemorySortingBatchSize: null,
     cancellationToken: token);
   ```
   **El método BulkImportAsync acepta los siguientes parámetros:**
   
   |**Parámetro**  |**Descripción** |
   |---------|---------|
   |enableUpsert    |   Una marca para habilitar las operaciones upsert en los documentos. Si ya existe un documento con el identificador indicado, se actualiza. De forma predeterminada, se establece en false.      |
   |disableAutomaticIdGeneration    |    Una marca para deshabilitar la generación automática de identificador. De forma predeterminada, se establece en true.     |
   |maxConcurrencyPerPartitionKeyRange    | El grado máximo de simultaneidad por intervalo con clave de partición; si se establece en null, la biblioteca usará el valor predeterminado de 20. |
   |maxInMemorySortingBatchSize     |  El número máximo de documentos que se extraen desde el enumerador de documento que se pasa a la llamada API en cada fase. Si en la fase de ordenación en memoria que se produce antes de la importación en bloque se establece este parámetro en null, la biblioteca usará el valor mínimo predeterminado (documents.count, 1000000).       |
   |cancellationToken    |    El token de cancelación para cerrar correctamente la operación de importación en bloque.     |

   **Definición del objeto de respuesta de importación en bloque** El resultado de la llamada API de importación en bloque contiene los siguientes atributos:

   |**Parámetro**  |**Descripción**  |
   |---------|---------|
   |NumberOfDocumentsImported (long)   |  El número total de documentos importados correctamente respecto al total de los documentos proporcionados a la llamada API de importación en bloque.       |
   |TotalRequestUnitsConsumed (double)   |   Total de unidades de solicitud (RU) consumidas por la llamada API de importación en bloque.      |
   |TotalTimeTaken (TimeSpan)    |   El tiempo total que tarda la llamada API de importación en bloque en completar la ejecución.      |
   |BadInputDocuments (List\<object>)   |     La lista de documentos con formato incorrecto no importados correctamente en la llamada API de importación en bloque. Corrija los documentos devueltos y vuelva a intentar realizar la importación. Los documentos con formato incorrecto incluyen documentos cuyo valor de identificador no es una cadena (null o cualquier otro tipo de datos se consideran no válidos).    |

## <a name="bulk-update-data-in-your-azure-cosmos-account"></a>Actualización en bloque de datos de la cuenta de Azure Cosmos

Puede actualizar los documentos existentes con el uso de la API BulkUpdateAsync. En este ejemplo, se establece el campo `Name` en un nuevo valor y se quita el campo `Description` de los documentos existentes. Para conocer el conjunto completo de operaciones de actualización admitidas, consulte la [documentación de la API](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkupdate?view=azure-dotnet).

1. Navegue hasta la carpeta "BulkUpdateSample" y abra el archivo "BulkUpdateSample.sln".  

2. Defina los elementos de actualización junto con las operaciones de actualización de campos correspondientes. En este ejemplo, usará `SetUpdateOperation` para actualizar el campo `Name` y `UnsetUpdateOperation` para quitar el campo `Description` de todos los documentos. También puede realizar otras operaciones, como incrementar un campo de documento con un valor determinado, insertar valores específicos en un campo de matriz o quitar un valor concreto de un campo de matriz. Para obtener información sobre los distintos métodos proporcionados por la API de actualización en masa, vea la [documentación de la API](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.bulkupdate?view=azure-dotnet).

   ```csharp
   SetUpdateOperation<string> nameUpdate = new SetUpdateOperation<string>("Name", "UpdatedDoc");
   UnsetUpdateOperation descriptionUpdate = new UnsetUpdateOperation("description");

   List<UpdateOperation> updateOperations = new List<UpdateOperation>();
   updateOperations.Add(nameUpdate);
   updateOperations.Add(descriptionUpdate);

   List<UpdateItem> updateItems = new List<UpdateItem>();
   for (int i = 0; i < 10; i++)
   {
    updateItems.Add(new UpdateItem(i.ToString(), i.ToString(), updateOperations));
   }
   ```

3. La aplicación llama a la API BulkUpdateAsync. Para obtener información sobre la definición del método BulkUpdateAsync, consulte la [documentación de la API](https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmosdb.bulkexecutor.ibulkexecutor.bulkupdateasync?view=azure-dotnet).  

   ```csharp
   BulkUpdateResponse bulkUpdateResponse = await bulkExecutor.BulkUpdateAsync(
     updateItems: updateItems,
     maxConcurrencyPerPartitionKeyRange: null,
     maxInMemorySortingBatchSize: null,
     cancellationToken: token);
   ```  
   **El método BulkUpdateAsync acepta los siguientes parámetros:**

   |**Parámetro**  |**Descripción** |
   |---------|---------|
   |maxConcurrencyPerPartitionKeyRange    |   El grado máximo de simultaneidad por intervalo de claves de partición; si este parámetro se establece en null, la biblioteca usará el valor predeterminado (20).   |
   |maxInMemorySortingBatchSize    |    Número máximo de elementos de actualización extraídos del enumerador de elementos de actualización pasado a la llamada API en cada fase. Si en la fase de ordenación en memoria que se produce antes de la actualización en bloque se establece este parámetro en null, la biblioteca usará el valor mínimo predeterminado (updateItems.count, 1000000).     |
   | cancellationToken|El token de cancelación para cerrar correctamente la operación de actualización en bloque. |

   **Definición del objeto de respuesta de actualización en masa** El resultado de la llamada API de actualización en masa contiene los siguientes atributos:

   |**Parámetro**  |**Descripción** |
   |---------|---------|
   |NumberOfDocumentsUpdated (long)    |   El número de documentos actualizados correctamente respecto al total de documentos proporcionados a la llamada API de actualización en bloque.      |
   |TotalRequestUnitsConsumed (double)   |    Total de unidades de solicitud (RU) consumidas por la llamada API de actualización en bloque.    |
   |TotalTimeTaken (TimeSpan)   | El tiempo total que le llevó a la llamada API de actualización en bloque completar la ejecución. |
    
## <a name="performance-tips"></a>Consejos de rendimiento 

Tenga en cuenta los siguientes puntos para mejorar el rendimiento al utilizar la biblioteca BulkExecutor:

* Para obtener el mejor rendimiento, ejecute la aplicación desde una máquina virtual de Azure que se encuentre en la misma región que la región de escritura de la cuenta de Azure Cosmos.  

* Se recomienda crear una instancia de un único objeto `BulkExecutor` para toda la aplicación dentro de una sola máquina virtual que se corresponda a un contenedor específico de Azure Cosmos.  

* Dado que una única ejecución de la API de operación en bloque consume un gran fragmento de la CPU y de la E/S de red del equipo cliente (esto sucede debido a la generación de varias tareas internamente), evite generar varias tareas simultáneas en el proceso de la aplicación que ejecuten llamadas API de operaciones en bloque. Si una única llamada API de operación en bloque en una única máquina virtual no puede consumir la capacidad de proceso de todo el contenedor (si la capacidad de proceso del contenedor es superior a 1 millón RU/s), es preferible crear máquinas virtuales independientes para ejecutar simultáneamente las llamadas API de operaciones en bloque.  

* Asegúrese de que se invoca el método `InitializeAsync()` después de crear una instancia de un objeto BulkExecutor para capturar el mapa de particiones del contenedor de Cosmos de destino.  

* En el archivo App.Config de la aplicación, asegúrese de que **gcServer** está habilitado para mejorar el rendimiento.
  ```xml  
  <runtime>
    <gcServer enabled="true" />
  </runtime>
  ```
* La biblioteca emite seguimientos que se pueden recopilar en un archivo de registro o en la consola. Para habilitar ambas opciones, agregue el siguiente código al archivo App.Config de la aplicación.

  ```xml
  <system.diagnostics>
    <trace autoflush="false" indentsize="4">
      <listeners>
        <add name="logListener" type="System.Diagnostics.TextWriterTraceListener" initializeData="application.log" />
        <add name="consoleListener" type="System.Diagnostics.ConsoleTraceListener" />
      </listeners>
    </trace>
  </system.diagnostics>
  ```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre los detalles del paquete NuGet y las notas de la versión, vea los [detalles del SDK de Bulk Executor](sql-api-sdk-bulk-executor-dot-net.md).
