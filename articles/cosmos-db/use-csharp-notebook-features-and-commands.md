---
title: Uso de los comandos y las características de cuadernos integrados en cuadernos en C# de Azure Cosmos DB (versión preliminar)
description: Aprenda a usar los comandos y las características integradas para realizar operaciones comunes con los cuadernos en C# integrados de Azure Cosmos DB.
author: deborahc
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/19/2020
ms.author: dech
ms.custom: devx-track-python, devx-track-csharp
ms.openlocfilehash: 47ac6b4caf97b264df58732f8d031fc63091171e
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019273"
---
# <a name="use-built-in-notebook-commands-and-features-in-azure-cosmos-db-c-notebooks-preview"></a>Uso de los comandos y las características de cuadernos integrados en cuadernos en C# de Azure Cosmos DB (versión preliminar)

Los cuadernos de Jupyter integrados en Azure Cosmos DB permiten analizar y visualizar los datos desde Azure Portal. En este artículo se describe cómo usar los comandos y características de cuadernos integrados para realizar operaciones comunes en cuadernos en C#.

## <a name="install-a-new-nuget-package"></a>Instalar un nuevo paquete NuGet
Después de habilitar la compatibilidad con el cuaderno en sus cuentas de Azure Cosmos, puede abrir un nuevo cuaderno e instalar un paquete.

En una celda de código nueva, inserte y ejecute el código siguiente, reemplazando ``PackageToBeInstalled`` por el paquete NuGet que quiera y ``optionalVersion`` con una versión específica del paquete si lo desea. 

```csharp
#r "nuget: PackageToBeInstalled, optionalVersion"
```

Puede instalar varios paquetes NuGet en la misma celda. Los paquetes estarán disponibles en cualquier cuaderno del área de trabajo de la cuenta de Azure Cosmos. 

Actualmente, el área de trabajo de cuadernos en C# no admite la resolución recursiva de paquetes NuGet. Si un paquete NuGet tiene dependencias de otros paquetes NuGet que no están instalados, tendrá que hacer referencia a ellos explícitamente junto con el paquete primario.

> [!TIP]
> Si el cuaderno requiere un paquete personalizado, es recomendable que agregue una celda en el cuaderno para instalar el paquete y que la convierta en la primera celda. Esto reduce la posibilidad de conflictos con otros paquetes que Azure Cosmos DB carga de forma predeterminada. También es fácil volver a instalar los paquetes si [restablece el área de trabajo](#reset-notebooks-workspace), lo que elimina todos los paquetes. 

## <a name="use-the-built-in-azure-cosmos-db-net-sdk"></a>Uso del SDK de .NET de Azure Cosmos DB integrado
La versión 3 del [SDK de .NET de Azure Cosmos DB para la API de SQL](https://github.com/Azure/azure-cosmos-dotnet-v3) está instalada e incluida en el entorno del cuaderno para la cuenta de Azure Cosmos.

Cree una instancia de ``CosmosClient`` para ejecutar cualquier operación del SDK. 

Por ejemplo:

```csharp
// Include usings
using System;
using Microsoft.Azure.Cosmos; //namespace for Azure Cosmos DB .NET V3 SDK
using System.Collections;

// Initialize a new instance of CosmosClient using the built-in account endpoint and key parameters
CosmosClient cosmosClient = new CosmosClient(Cosmos.Endpoint, Cosmos.Key);

// Create a new database and container with 400 RU/s
Database database = await cosmosClient.CreateDatabaseIfNotExistsAsync("DatabaseName");
Container container = await database.CreateContainerIfNotExistsAsync("ContainerName", "/partitionKey", 400);
```
Para obtener más información, vea las [muestras del SDK de .NET V3](https://github.com/Azure/azure-cosmos-dotnet-v3/tree/master/Microsoft.Azure.Cosmos.Samples/Usage). 

> [!IMPORTANT]
> El SDK de .NET de Azure Cosmos DB integrado solo se admite para las cuentas de la API de SQL (Core). Para otras API, deberá [instalar el controlador de .NET correspondiente](#install-a-new-nuget-package) a la API. 

## <a name="set-custom-options-using-cosmosclientoptions"></a>Establecer opciones personalizadas mediante ``CosmosClientOptions``
Para mayor flexibilidad, puede establecer la propiedad ``CosmosClientOptions`` personalizada y pasarla en la instancia de ``CosmosClient``. Puede usar esta propiedad para lo siguiente:

- Definir un nombre de aplicación en el sufijo user-agent para usarlo en cada solicitud.
- Definir la región preferida que se usará al ejecutar operaciones en el servicio.
- Definir una directiva de reintentos personalizada para controlar las solicitudes con limitación de frecuencia.

Consulte en el artículo [Referencia de la API de CosmosClientOptions](/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions) todas las configuraciones admitidas. A continuación se muestra un ejemplo en el que se indica cómo establecer la propiedad `cosmosClientOptions`:

```csharp
using Microsoft.Azure.Cosmos;

// Configure CosmosClientOptions
var cosmosClientOptions = new CosmosClientOptions
{
    ApplicationName = "ContosoNotebookAppName",
    ApplicationRegion = Regions.RegionName, // By default, this is the region you first created your account in
    MaxRetryAttemptsOnRateLimitedRequests = 9, // By default, this value is 9
};

var client = new CosmosClient(Cosmos.Endpoint, Cosmos.Key, cosmosClientOptions);
```

## <a name="access-the-account-endpoint-and-primary-key-variables"></a>Acceso al punto de conexión de la cuenta y a las variables de la clave principal
Puede acceder al punto de conexión integrado y a la clave de la cuenta de Azure Cosmos en que existe el cuaderno.

```csharp
var key = Cosmos.Key;
var endpoint = Cosmos.Endpoint;
```

> [!IMPORTANT]
> Las variables ``Cosmos.Key`` y ``Cosmos.Endpoint`` solo se pueden aplicar a la API de SQL. En cuanto a otras API, busque el punto de conexión y la clave en la hoja de las **cadenas de conexión** o las **claves** en su cuenta de Azure Cosmos.  

## <a name="print-console-output-in-c-code"></a>Impresión de la salida de la consola en el código C#
En el código C#, puede usar la sintaxis Display.AsMarkdown() con [interpolación de cadenas](/dotnet/csharp/language-reference/tokens/interpolated) para imprimir la salida de la consola que se mostrará al ejecutar la celda. 

Por ejemplo: 

```csharp
// Print text in the output
Display.AsMarkdown($"Hello world!");

// Print a variable in the output
for (int i = 0; i < 5; i++) {
    Display.AsMarkdown($"Printing out variable: {i}");
}
```
> [!IMPORTANT]
> Actualmente, la sintaxis de Console.WriteLine() no se admite en cuadernos en C#. Use Display.AsMarkdown para imprimir la salida de la consola del programa. 

## <a name="use-built-in-nteract-data-explorer"></a>Uso del explorador de datos nteract integrado
Puede usar el [explorador de datos integrado nteract](https://blog.nteract.io/designing-the-nteract-data-explorer-f4476d53f897) para filtrar y visualizar una colección de elementos. En una celda, coloque la variable que quiera visualizar en la última línea, que se muestra automáticamente en nteract al ejecutar la celda.

En el ejemplo *GetingStarted_Csharp.ipynb*, podemos imprimir la variable con nuestro resultado: ``telemetryEvents``. Puede consultar toda la muestra en el [cuaderno GettingStarted_Csharp.ipynb](https://github.com/Azure-Samples/cosmos-notebooks/blob/master/CSharp_quickstarts/GettingStarted_CSharp.ipynb). 

:::image type="content" source="media/use-notebook-features-and-commands/csharp-query-cell.png" alt-text="Celda de consulta de CSharp":::

:::image type="content" source="media/use-notebook-features-and-commands/csharp-nteract-built-in-chart.png" alt-text="Explorador de datos nteract":::

## <a name="use-built-in-dictionary-viewer"></a>Uso del visor de diccionario integrado
Puede usar el visor de diccionario integrado para ver una variable. En una celda, coloque la variable que quiera visualizar en la última línea, que se mostrará automáticamente al ejecutar la celda.

:::image type="content" source="media/use-notebook-features-and-commands/csharp-built-in-dictionary-viewer.png" alt-text="Visor de diccionario integrado":::

## <a name="upload-json-items-to-a-container"></a>Carga de elementos JSON en un contenedor
Puede usar el comando mágico ``%%upload`` para cargar datos de un archivo JSON en un contenedor específico de Azure Cosmos. Ejecute el siguiente comando para cargar los elementos:

```csharp
%%upload --databaseName {database_id} --containerName {container_id} --url {url_location_of_file}
```

- Reemplace ``{database_id}`` y ``{container_id}`` por el nombre de la base de datos y el contenedor en la cuenta de Azure Cosmos. 
- Reemplace ``{url_location_of_file}`` por la ubicación del archivo JSON. El archivo debe ser una matriz de objetos JSON válidos y accesible a través de la red pública de Internet.

Por ejemplo:

```csharp
%%upload --database databaseName --container containerName --url 
https://contoso.com/path/to/data.json
```
```
Documents successfully uploaded to ContainerName
Total number of documents imported : 2654
Total time taken : 00:00:38.1228087 hours
Total RUs consumed : 25022.58
```

Con las estadísticas de salida, puede calcular las RU/s reales que se usan para cargar los elementos. Por ejemplo, si 25 000 RU se consumieron más de 38 segundos, las RU/s reales son 25 000 RU/38 segundos = 658 RU/s.

## <a name="run-another-notebook-in-current-notebook"></a>Ejecución de otro cuaderno en el actual 
Puede usar el comando mágico ``%%run`` para ejecutar otro cuaderno en el área de trabajo desde el cuaderno actual. La sintaxis es la siguiente:

```csharp
%%run ./path/to/{notebookName}.ipynb
```
Reemplace ``{notebookName}`` por el nombre del cuaderno que quiere ejecutar. El cuaderno debe estar en el área de trabajo "My Notebooks" (Mis cuadernos) actual. 

## <a name="reset-notebooks-workspace"></a>Restablecer el área de trabajo de los cuadernos
Para restablecer la configuración predeterminada del área de trabajo de los cuadernos, seleccione **Reset Workspace** (Restablecer el área de trabajo) en la barra de comandos. Se quitarán todos los paquetes instalados personalizados y se reiniciará el servidor de Jupyter. Los cuadernos, los archivos y los recursos de Azure Cosmos no se verán afectados.  

:::image type="content" source="media/use-notebook-features-and-commands/reset-workspace.png" alt-text="Restablecer el área de trabajo de los cuadernos":::

## <a name="next-steps"></a>Pasos siguientes

- Conozca las ventajas de los [cuadernos de Jupyter para Azure Cosmos DB](cosmosdb-jupyter-notebooks.md)
- Obtenga información acerca del [SDK de .NET para Azure Cosmos DB para la API de SQL](https://github.com/Azure/azure-cosmos-dotnet-v3)
