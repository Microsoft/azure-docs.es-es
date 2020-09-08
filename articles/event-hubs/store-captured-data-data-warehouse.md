---
title: 'Tutorial: Migración de datos de eventos a SQL Data Warehouse: Azure Event Hubs'
description: 'Tutorial: En este tutorial se muestra cómo capturar datos de una instancia de Event Hubs y migrarlos a SQL Data Warehouse mediante el uso de una función de Azure desencadenada por Event Grid.'
services: event-hubs
ms.date: 06/23/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: b6b6466675c8fa258af8370798cadd88e3b25a83
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88997836"
---
# <a name="tutorial-migrate-captured-event-hubs-data-to-a-sql-data-warehouse-using-event-grid-and-azure-functions"></a>Tutorial: Migración de datos de Event Hubs capturados a SQL Data Warehouse mediante Event Grid y Azure Functions

Event Hubs [Capture](./event-hubs-capture-overview.md) es la forma más fácil de enviar automáticamente datos en streaming de Event Hubs a Azure Blob Storage o a Azure Data Lake Store. Posteriormente dichos datos se pueden procesar y enviar a otros destinos de almacenamiento, como SQL Data Warehouse o Cosmos DB. En este tutorial aprenderá a capturar datos de una instancia de Event Hubs y migrarlos a SQL Data Warehouse mediante el uso de una función de Azure desencadenada por [Event Grid](../event-grid/overview.md).

![Visual Studio](./media/store-captured-data-data-warehouse/EventGridIntegrationOverview.PNG)

- En primer lugar, cree una instancia de Event Hubs con la característica **Capture** habilitada y establezca una instancia de Azure Blob Storage como destino. Los datos que genera WindTurbineGenerator se transmiten en secuencias a la instancia de Event Hubs y se capturan automáticamente en Azure Storage como archivos Avro.
- Después, cree una suscripción a Azure Event Grid con el espacio de nombres de Event Hubs como origen y el punto de conexión de Azure Functions como destino.
- Cada vez que un archivo Avro nuevo se envía al blob de Azure Storage mediante la característica Capture de Event Hubs, Event Grid se lo notifica a Azure Functions con el identificador URI del blob. Luego, Azure Functions migra los datos del blob a una instancia de SQL Data Warehouse.

En este tutorial realizará lo siguiente:

> [!div class="checklist"]
>
> - Implementar la infraestructura
> - Publicar código en una aplicación de Functions
> - Crear una suscripción a Event Grid desde la aplicación de Functions
> - Hacer streaming de los datos de ejemplo de en un a instancia de Event Hubs.
> - Comprobar los datos capturados en SQL Data Warehouse

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- [Visual Studio 2019](https://www.visualstudio.com/vs/). Asegúrese de que instala las siguientes con cargas de trabajo: desarrollo de escritorio de .NET, desarrollo de Azure, desarrollo web y de ASP.NET, desarrollo de Node.js y desarrollo de Python
- Descarga del [Ejemplo de Git](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Azure.Messaging.EventHubs/EventHubsCaptureEventGridDemo) La solución de ejemplo contiene los siguientes componentes:

  - *WindTurbineDataGenerator* : un publicador simple que envía datos de turbina eólica de ejemplo a un centro de eventos con la función Capture habilitada
  - *FunctionDWDumper* : una función de Azure que recibe una notificación de Event Grid cuando se captura un archivo Avro en el blob de Azure Storage. Recibe la ruta de acceso, del identificador URI del blob, lee su contenido y envía estos datos a SQL Data Warehouse.

  En este ejemplo se usa el paquete Azure.Messaging.EventHubs más reciente. Puede encontrar el ejemplo anterior que usa el paquete Microsoft.Azure.EventHubs [aquí](https://github.com/Azure/azure-event-hubs/tree/master/samples/e2e/EventHubsCaptureEventGridDemo).

### <a name="deploy-the-infrastructure"></a>Implementar la infraestructura

Use Azure PowerShell o la CLI de Azure para implementar la infraestructura necesaria para este tutorial con este [plantilla de Azure Resource Manager](https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/EventHubsDataMigration.json). Esta plantilla crea los siguientes recursos:

- Event Hubs con la característica Capture habilitada
- Cuenta de almacenamiento de los datos de eventos capturados
- Un plan de Azure App Service para hospedar la aplicación Functions
- Aplicación de función para procesar archivos de eventos capturados
- Un servidor SQL lógico para hospedar Data Warehouse
- Un SQL Data Warehouse para almacenar los datos migrados

Las secciones siguientes proporcionan comandos de CLI de Azure y Azure PowerShell para implementar la infraestructura necesaria para el tutorial. Actualice los nombres de los siguientes objetos antes de ejecutar los comandos: 

- Grupo de recursos de Azure 
- Región del grupo de recursos
- Espacio de nombres de Event Hubs
- Centro de eventos
- Servidor SQL lógico
- Usuario de SQL (y contraseña)
- Azure SQL Database
- Azure Storage 
- Aplicación Azure Functions

Estos scripts tardan algún tiempo en crear todos los artefactos de Azure. Espere hasta que el script se complete antes de continuar. Si la implementación no se realiza por alguna razón, elimine el grupo de recursos, corrija el problema notificado y vuelva a ejecutar el comando. 

#### <a name="azure-cli"></a>Azure CLI

Para implementar la plantilla mediante la CLI de Azure, use los siguientes comandos:

```azurecli-interactive
az group create -l westus -n rgDataMigrationSample

az group deployment create `
  --resource-group rgDataMigrationSample `
  --template-uri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/EventHubsDataMigration.json `
  --parameters eventHubNamespaceName=<event-hub-namespace> eventHubName=hubdatamigration sqlServerName=<sql-server-name> sqlServerUserName=<user-name> sqlServerPassword=<password> sqlServerDatabaseName=<database-name> storageName=<unique-storage-name> functionAppName=<app-name>
```

#### <a name="azure-powershell"></a>Azure PowerShell
Para implementar la plantilla mediante PowerShell, use los siguientes comandos:

```powershell
New-AzResourceGroup -Name rgDataMigration -Location westcentralus

New-AzResourceGroupDeployment -ResourceGroupName rgDataMigration -TemplateUri https://raw.githubusercontent.com/Azure/azure-docs-json-samples/master/event-grid/EventHubsDataMigration.json -eventHubNamespaceName <event-hub-namespace> -eventHubName hubdatamigration -sqlServerName <sql-server-name> -sqlServerUserName <user-name> -sqlServerDatabaseName <database-name> -storageName <unique-storage-name> -functionAppName <app-name>
```

### <a name="create-a-table-in-sql-data-warehouse"></a>Crear una tabla en SQL Data Warehouse

Para crear una tabla en SQL Data Warehouse ejecute el script [CreateDataWarehouseTable.sql](https://github.com/Azure/azure-event-hubs/blob/master/samples/e2e/EventHubsCaptureEventGridDemo/scripts/CreateDataWarehouseTable.sql) mediante [Visual Studio](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-query-visual-studio.md), [SQL Server Management Studio](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-query-ssms.md) o el Editor de consultas del portal. 

```sql
CREATE TABLE [dbo].[Fact_WindTurbineMetrics] (
    [DeviceId] nvarchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL, 
    [MeasureTime] datetime NULL, 
    [GeneratedPower] float NULL, 
    [WindSpeed] float NULL, 
    [TurbineSpeed] float NULL
)
WITH (CLUSTERED COLUMNSTORE INDEX, DISTRIBUTION = ROUND_ROBIN);
```

## <a name="publish-code-to-the-functions-app"></a>Publicación de código en la aplicación de Functions

1. Abra la solución *EventHubsCaptureEventGridDemo.sln* en Visual Studio 2019.

1. En el Explorador de soluciones, haga clic con el botón derecho en *FunctionEGDWDumper* y seleccione **Publicar**.

   ![Publicar la aplicación de función](./media/store-captured-data-data-warehouse/publish-function-app.png)

1. Seleccione **Azure Function App** y **Seleccionar existente**. Seleccione **Publicar**.

   ![Establecer la aplicación de función como destino](./media/store-captured-data-data-warehouse/pick-target.png)

1. Seleccione la aplicación de función que implementó con la plantilla. Seleccione **Aceptar**.

   ![Seleccionar la aplicación de función](./media/store-captured-data-data-warehouse/select-function-app.png)

1. Cuando Visual Studio haya configurado el perfil, seleccione **Publicar**.

   ![Select publish](./media/store-captured-data-data-warehouse/select-publish.png)

Después de publicar la función, estará listo para suscribirse al evento de captura desde Event Hubs.


## <a name="create-an-event-grid-subscription-from-the-functions-app"></a>Crear una suscripción a Event Grid desde la aplicación de Functions
 
1. Vaya a [Azure Portal](https://portal.azure.com/). Seleccione el grupo de recursos y la aplicación de función.

   ![Ver la aplicación de función](./media/store-captured-data-data-warehouse/view-function-app.png)

1. Seleccione la función.

   ![Seleccionar la función](./media/store-captured-data-data-warehouse/select-function.png)

1. Seleccione **Incorporación de una suscripción de Event Grid**.

   ![Agregar una suscripción](./media/store-captured-data-data-warehouse/add-event-grid-subscription.png)

1. Escriba un nombre para la suscripción de Event Grid. Use **Espacios de nombres de Event Hubs** como el tipo de evento. Proporcione los valores para seleccionar la instancia del espacio de nombres de Event Hubs. Deje el punto de conexión de suscriptor con el valor proporcionado. Seleccione **Crear**.

   ![Creación de una suscripción](./media/store-captured-data-data-warehouse/set-subscription-values.png)

## <a name="generate-sample-data"></a>Generación de datos de ejemplo  
Ya ha configurado un centro de eventos, SQL Data Warehouse, Azure Function App y una suscripción a Event Grid. Puede ejecutar WindTurbineDataGenerator.exe para generar flujos de datos para la instancia de Event Hubs después de actualizar la cadena de conexión y el nombre del centro de eventos en el código fuente. 

1. En el portal, seleccione el espacio de nombres del centro de eventos. Seleccione **Cadenas de conexión**.

   ![Seleccionar Cadenas de conexión](./media/store-captured-data-data-warehouse/event-hub-connection.png)

2. Seleccione **RootManageSharedAccessKey**.

   ![Seleccionar la clave](./media/store-captured-data-data-warehouse/show-root-key.png)

3. Copie **Cadena de conexión: clave principal**.

   ![Copiar clave](./media/store-captured-data-data-warehouse/copy-key.png)

4. Vuelva al proyecto de Visual Studio. En el proyecto *WindTurbineDataGenerator*, abra *program.cs*.

5. Actualice los valores de **EventHubConnectionString** y **EventHubName** con la cadena de conexión y el nombre del centro de eventos. 

   ```cs
   private const string EventHubConnectionString = "Endpoint=sb://demomigrationnamespace.servicebus.windows.net/...";
   private const string EventHubName = "hubdatamigration";
   ```

6. Compile la solución y ejecute la aplicación WindTurbineGenerator.exe. 

## <a name="verify-captured-data-in-data-warehouse"></a>Comprobación de los datos capturados en Data Warehouse
Transcurridos unos minutos, realice una consulta en la tabla de SQL Data Warehouse. Observe que los datos que ha generado WindTurbineDataGenerator se han transmitido en secuencias al centro de eventos, se han capturado en un contenedor de Azure Storage y, después, se han migrado a la tabla de SQL Data Warehouse mediante Azure Functions.  

## <a name="next-steps"></a>Pasos siguientes 
Puede usar las eficaces herramientas de visualización de datos con Data Warehouse para conseguir detalles accionables.

En este artículo se muestra cómo usar [Power BI con SQL Data Warehouse](/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect)