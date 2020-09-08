---
title: Copia de datos de Azure Blob Storage a Azure SQL Database
description: Este tutorial proporciona instrucciones detalladas sobre cómo copiar datos desde Azure Blob Storage a Azure SQL Database.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.date: 11/08/2019
ms.author: jingwang
ms.openlocfilehash: b2293c0dd74903921abb58037afd8eb5db3659d9
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85513260"
---
# <a name="copy-data-from-azure-blob-to-azure-sql-database-using-azure-data-factory"></a>Copia de datos del blob de Azure a Azure SQL Database mediante Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial, creará una canalización de Data Factory que copia datos de Azure Blob Storage a Azure SQL Database. El patrón de configuración de este tutorial se aplica a la copia de un almacén de datos basado en archivos a un almacén de datos relacional. Para una lista de los almacenes de datos que se admiten como orígenes y receptores, consulte [Almacenes de datos y formatos que se admiten](copy-activity-overview.md#supported-data-stores-and-formats).

En este tutorial, realice los siguientes pasos:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Creación de servicios vinculados de Azure SQL Database y Azure Storage
> * Creación de conjuntos de datos de Azure SQL Database y el blob de Azure
> * Creación de una canalización que contiene una actividad de copia
> * Inicio de la ejecución de una canalización.
> * Supervisión de las ejecuciones de canalización y actividad.

En este tutorial se usa SDK de .NET. Puede usar otros mecanismos para interactuar con Azure Data Factory. Consulte los ejemplos de **Inicios rápidos**.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Requisitos previos

* *Cuenta de Azure Storage*. Blob Storage se puede usar como almacén de datos de *origen*. Si no dispone de una cuenta de Azure Storage, consulte [Creación de una cuenta de almacenamiento de uso general](../storage/common/storage-account-create.md).
* *Azure SQL Database*. La base de datos se puede usar como almacén de datos *receptor*. Si no tiene una base de datos en Azure SQL Database, consulte el artículo [Creación de una base de datos en Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md).
* *Visual Studio*. En el tutorial de este artículo se usa Visual Studio 2019.
* *[SDK de Azure para .NET](/dotnet/azure/dotnet-tools)* .
* *Aplicación de Azure Active Directory*. Si no tiene una aplicación de Azure Active Directory, consulte la sección [Creación de una aplicación de Azure Active Directory](../active-directory/develop/howto-create-service-principal-portal.md#register-an-application-with-azure-ad-and-create-a-service-principal) de [Procedimientos: Creación de una aplicación de Azure AD mediante el portal](../active-directory/develop/howto-create-service-principal-portal.md). Copie los siguientes valores para su uso en pasos posteriores: **Id. de aplicación (cliente)** , **Clave de autenticación** e **Id. de directorio (inquilino)** . Siga las instrucciones del mismo artículo para asignar la aplicación al rol **Colaborador**.

### <a name="create-a-blob-and-a-sql-table"></a>Creación de un blob y una tabla SQL

Ahora cree un blob de origen y una tabla de SQL receptora para preparar su blob de Azure y Azure SQL Database para el tutorial.

#### <a name="create-a-source-blob"></a>Creación de un blob de origen

En primer lugar, cree un contenedor y cargue un archivo de texto de entrada en él para crear un blob de origen:

1. Abra Bloc de notas. Copie el texto siguiente y guárdelo en un archivo local llamado *inputEmp.txt*.

    ```inputEmp.txt
    John|Doe
    Jane|Doe
    ```

2. Use una herramienta como [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para crear el contenedor *adfv2tutorial* y cargar el archivo *inputEmp.txt* en el contenedor.

#### <a name="create-a-sink-sql-table"></a>Creación de una tabla SQL receptora

A continuación, cree una tabla de SQL receptora:

1. Use el siguiente script de SQL para crear la tabla *dbo.emp* en su base de datos de Azure SQL.

    ```sql
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50)
    )
    GO

    CREATE CLUSTERED INDEX IX_emp_ID ON dbo.emp (ID);
    ```

2. Permita que los servicios de Azure accedan a SQL Database. Asegúrese de permitir el acceso a servicios de Azure en el servidor para que el servicio Data Factory pueda escribir datos en SQL Database. Para comprobar y activar esta configuración, realice los siguientes pasos:

    1. Vaya a [Azure Portal](https://portal.azure.com) para administrar la instancia de SQL Server. Busque y seleccione **Servidores SQL Server**.

    2. Seleccione el servidor.

    3. En el encabezado **Seguridad** del menú de SQL Server, seleccione **Firewalls y redes virtuales**.

    4. En la página **Firewall y redes virtuales**, en **Permitir que los servicios y recursos de Azure tengan acceso a este servidor**, seleccione **Activar**.

## <a name="create-a-visual-studio-project"></a>Creación de un proyecto de Visual Studio

Con Visual Studio, cree una aplicación de consola .NET de C#.

1. Abra Visual Studio.
2. En la ventana **Inicio**, seleccione **Crear un proyecto**.
3. En la ventana **Crear un proyecto**, seleccione la versión de C# de **Aplicación de consola (.NET Framework)** de la lista de tipos de proyecto. Luego, seleccione **Siguiente**.
4. En la ventana **Configure su nuevo proyecto**, escriba un valor para **Nombre de proyecto** en *ADFv2Tutorial*. En **Ubicación**, busque o cree el directorio donde guardar el proyecto. Seleccione **Crear**. El nuevo proyecto aparece en el IDE de Visual Studio.

## <a name="install-nuget-packages"></a>Instalación de paquetes NuGet

A continuación, instale los paquetes de biblioteca necesarios mediante el administrador de paquetes NuGet.

1. En la barra de menús, seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**.
2. En el panel **Consola del Administrador de paquetes**, ejecute los comandos siguientes para instalar los paquetes. Para información sobre el paquete NuGet de Azure Data Factory, consulte el [Microsoft.Azure.Management.DataFactory](https://www.nuget.org/packages/Microsoft.Azure.Management.DataFactory/).

    ```package manager console
    Install-Package Microsoft.Azure.Management.DataFactory
    Install-Package Microsoft.Azure.Management.ResourceManager -PreRelease
    Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory
    ```

## <a name="create-a-data-factory-client"></a>Creación de un cliente de factoría de datos

Para crear un cliente de factoría de datos, siga estos pasos.

1. Abra *Program.cs*, sobrescriba las instrucciones `using` existentes con el código siguiente para agregar referencias a los espacios de nombres.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using Microsoft.Rest;
    using Microsoft.Rest.Serialization;
    using Microsoft.Azure.Management.ResourceManager;
    using Microsoft.Azure.Management.DataFactory;
    using Microsoft.Azure.Management.DataFactory.Models;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    ```

2. Agregue el siguiente código al método `Main` que establece las variables. Reemplace los 14 marcadores de posición por sus propios valores.

    Para ver una lista de las regiones de Azure en las que Data Factory está disponible actualmente, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). En la lista desplegable **Productos**, elija **Examinar** > **Análisis** > **Data Factory**. A continuación, en la lista desplegable **Regiones**, elija las regiones que le interesen. Aparecerá una cuadrícula con el estado de disponibilidad de los productos de Data Factory en las regiones seleccionadas.

    > [!NOTE]
    > Los almacenes de datos (como Azure Storage y Azure SQL Database) y los procesos (como HDInsight) que Data Factory usa pueden encontrarse en otras regiones distintas de las elegidas para Data Factory.

    ```csharp
    // Set variables
    string tenantID = "<your tenant ID>";
    string applicationId = "<your application ID>";
    string authenticationKey = "<your authentication key for the application>";
    string subscriptionId = "<your subscription ID to create the factory>";
    string resourceGroup = "<your resource group to create the factory>";

    string region = "<location to create the data factory in, such as East US>";
    string dataFactoryName = "<name of data factory to create (must be globally unique)>";

    // Specify the source Azure Blob information
    string storageAccount = "<your storage account name to copy data>";
    string storageKey = "<your storage account key>";
    string inputBlobPath = "adfv2tutorial/";
    string inputBlobName = "inputEmp.txt";

    // Specify the sink Azure SQL Database information
    string azureSqlConnString =
        "Server=tcp:<your server name>.database.windows.net,1433;" +
        "Database=<your database name>;" +
        "User ID=<your username>@<your server name>;" +
        "Password=<your password>;" +
        "Trusted_Connection=False;Encrypt=True;Connection Timeout=30";
    string azureSqlTableName = "dbo.emp";

    string storageLinkedServiceName = "AzureStorageLinkedService";
    string sqlDbLinkedServiceName = "AzureSqlDbLinkedService";
    string blobDatasetName = "BlobDataset";
    string sqlDatasetName = "SqlDataset";
    string pipelineName = "Adfv2TutorialBlobToSqlCopy";
    ```

3. Agregue el código siguiente al método `Main` que crea una instancia de la clase `DataFactoryManagementClient`. Este objeto se usa para crear una factoría de datos, un servicio vinculado, los conjuntos de datos y una canalización. También se usa para supervisar los detalles de ejecución de la canalización.

    ```csharp
    // Authenticate and create a data factory management client
    var context = new AuthenticationContext("https://login.windows.net/" + tenantID);
    ClientCredential cc = new ClientCredential(applicationId, authenticationKey);
    AuthenticationResult result = context.AcquireTokenAsync(
        "https://management.azure.com/", cc
    ).Result;
    ServiceClientCredentials cred = new TokenCredentials(result.AccessToken);
    var client = new DataFactoryManagementClient(cred) { SubscriptionId = subscriptionId };
    ```

## <a name="create-a-data-factory"></a>Crear una factoría de datos

Agregue el código siguiente al método `Main` que crea una *factoría de datos*.

```csharp
// Create a data factory
Console.WriteLine("Creating a data factory " + dataFactoryName + "...");
Factory dataFactory = new Factory
{
    Location = region,
    Identity = new FactoryIdentity()
};

client.Factories.CreateOrUpdate(resourceGroup, dataFactoryName, dataFactory);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(dataFactory, client.SerializationSettings)
);

while (
    client.Factories.Get(
        resourceGroup, dataFactoryName
    ).ProvisioningState == "PendingCreation"
)
{
    System.Threading.Thread.Sleep(1000);
}
```

## <a name="create-linked-services"></a>Crear servicios vinculados

En este tutorial se crean dos servicios vinculados para el origen y el receptor respectivamente.

### <a name="create-an-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage

Agregue el código siguiente al método `Main` que crea un *servicio vinculado de Azure Storage*. Para información sobre las propiedades admitidas y sus detalles, consulte [Propiedades del servicio vinculado de Azure Blob Storage](connector-azure-blob-storage.md#linked-service-properties).

```csharp
// Create an Azure Storage linked service
Console.WriteLine("Creating linked service " + storageLinkedServiceName + "...");

LinkedServiceResource storageLinkedService = new LinkedServiceResource(
    new AzureStorageLinkedService
    {
        ConnectionString = new SecureString(
            "DefaultEndpointsProtocol=https;AccountName=" + storageAccount +
            ";AccountKey=" + storageKey
        )
    }
);

client.LinkedServices.CreateOrUpdate(
    resourceGroup, dataFactoryName, storageLinkedServiceName, storageLinkedService
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(storageLinkedService, client.SerializationSettings)
);
```

### <a name="create-an-azure-sql-database-linked-service"></a>Creación de un servicio vinculado de Azure SQL Database

Agregue el código siguiente al método `Main` que crea un *servicio vinculado de Azure SQL Database*. Para información sobre las propiedades admitidas y sus detalles, consulte [Propiedades del servicio vinculado de Azure SQL Database](connector-azure-sql-database.md#linked-service-properties).

```csharp
// Create an Azure SQL Database linked service
Console.WriteLine("Creating linked service " + sqlDbLinkedServiceName + "...");

LinkedServiceResource sqlDbLinkedService = new LinkedServiceResource(
    new AzureSqlDatabaseLinkedService
    {
        ConnectionString = new SecureString(azureSqlConnString)
    }
);

client.LinkedServices.CreateOrUpdate(
    resourceGroup, dataFactoryName, sqlDbLinkedServiceName, sqlDbLinkedService
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(sqlDbLinkedService, client.SerializationSettings)
);
```

## <a name="create-datasets"></a>Creación de conjuntos de datos

En esta sección se crean dos conjuntos de datos: uno para el origen y otro para el receptor.

### <a name="create-a-dataset-for-source-azure-blob"></a>Creación de un conjunto de datos para el blob de Azure de origen

Agregue el código siguiente al método `Main` que crea un *conjunto de datos de blob de Azure*. Para información sobre las propiedades admitidas y sus detalles, consulte [Propiedades del conjunto de datos de blob de Azure](connector-azure-blob-storage.md#dataset-properties).

Se define un conjunto de datos que representa los datos de origen del blob de Azure. Este conjunto de datos de blob hace referencia al servicio vinculado de Azure Storage que creó en el paso anterior y describe:

- La ubicación del blob de donde realizar la copia (`FolderPath`y `FileName`)
- El formato de blob que indica cómo analizar el contenido (`TextFormat` y su configuración, como el delimitador de columna)
- La estructura de datos, incluidos los tipos de datos y los nombres de columna que se asignan en este ejemplo a la tabla de SQL receptora.

```csharp
// Create an Azure Blob dataset
Console.WriteLine("Creating dataset " + blobDatasetName + "...");
DatasetResource blobDataset = new DatasetResource(
    new AzureBlobDataset
    {
        LinkedServiceName = new LinkedServiceReference {
            ReferenceName = storageLinkedServiceName
        },
        FolderPath = inputBlobPath,
        FileName = inputBlobName,
        Format = new TextFormat { ColumnDelimiter = "|" },
        Structure = new List<DatasetDataElement>
        {
            new DatasetDataElement { Name = "FirstName", Type = "String" },
            new DatasetDataElement { Name = "LastName", Type = "String" }
        }
    }
);

client.Datasets.CreateOrUpdate(
    resourceGroup, dataFactoryName, blobDatasetName, blobDataset
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(blobDataset, client.SerializationSettings)
);
```

### <a name="create-a-dataset-for-sink-azure-sql-database"></a>Creación de un conjunto de datos para la base de datos de Azure SQL receptora

Agregue el código siguiente al método `Main` que crea un *conjunto de datos de Azure SQL Database*. Para información sobre las propiedades admitidas y sus detalles, consulte [Propiedades del conjunto de datos de Azure SQL Database](connector-azure-sql-database.md#dataset-properties).

Se define un conjunto de datos que representa los datos del receptor de Azure SQL Database. Este conjunto de datos hace referencia al servicio vinculado de Azure SQL Database que creó en el paso anterior. También especifica la tabla SQL que contiene los datos copiados.

```csharp
// Create an Azure SQL Database dataset
Console.WriteLine("Creating dataset " + sqlDatasetName + "...");
DatasetResource sqlDataset = new DatasetResource(
    new AzureSqlTableDataset
    {
        LinkedServiceName = new LinkedServiceReference
        {
            ReferenceName = sqlDbLinkedServiceName
        },
        TableName = azureSqlTableName
    }
);

client.Datasets.CreateOrUpdate(
    resourceGroup, dataFactoryName, sqlDatasetName, sqlDataset
);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(sqlDataset, client.SerializationSettings)
);
```

## <a name="create-a-pipeline"></a>Crear una canalización

Agregue el código siguiente al método `Main` que crea una *canalización con una actividad de copia*. En este tutorial, esta canalización contiene una actividad: `CopyActivity`, que toma el conjunto de datos del blob como origen y el conjunto de datos SQL como receptor. Para información sobre los detalles de la actividad de copia, consulte [Actividad de copia en Azure Data Factory](copy-activity-overview.md).

```csharp
// Create a pipeline with copy activity
Console.WriteLine("Creating pipeline " + pipelineName + "...");
PipelineResource pipeline = new PipelineResource
{
    Activities = new List<Activity>
    {
        new CopyActivity
        {
            Name = "CopyFromBlobToSQL",
            Inputs = new List<DatasetReference>
            {
                new DatasetReference() { ReferenceName = blobDatasetName }
            },
            Outputs = new List<DatasetReference>
            {
                new DatasetReference { ReferenceName = sqlDatasetName }
            },
            Source = new BlobSource { },
            Sink = new SqlSink { }
        }
    }
};

client.Pipelines.CreateOrUpdate(resourceGroup, dataFactoryName, pipelineName, pipeline);
Console.WriteLine(
    SafeJsonConvert.SerializeObject(pipeline, client.SerializationSettings)
);
```

## <a name="create-a-pipeline-run"></a>Creación de una ejecución de canalización

Agregue el código siguiente al método `Main` que *desencadena una ejecución de canalización*.

```csharp
// Create a pipeline run
Console.WriteLine("Creating pipeline run...");
CreateRunResponse runResponse = client.Pipelines.CreateRunWithHttpMessagesAsync(
    resourceGroup, dataFactoryName, pipelineName
).Result.Body;
Console.WriteLine("Pipeline run ID: " + runResponse.RunId);
```

## <a name="monitor-a-pipeline-run"></a>Supervisar una ejecución de canalización

Ahora inserte el código para comprobar los estados de ejecución de la canalización y obtener detalles sobre la ejecución de la actividad de copia.

1. Agregue el código siguiente al método `Main` para comprobar continuamente los estados de ejecución de la canalización hasta que termine de copiar los datos.

    ```csharp
    // Monitor the pipeline run
    Console.WriteLine("Checking pipeline run status...");
    PipelineRun pipelineRun;
    while (true)
    {
        pipelineRun = client.PipelineRuns.Get(
            resourceGroup, dataFactoryName, runResponse.RunId
        );
        Console.WriteLine("Status: " + pipelineRun.Status);
        if (pipelineRun.Status == "InProgress")
            System.Threading.Thread.Sleep(15000);
        else
            break;
    }
    ```

2. Agregue el código siguiente al método `Main` que recupera detalles de la ejecución de actividad de copia, como el tamaño de los datos que se han leído o escrito.

    ```csharp
    // Check the copy activity run details
    Console.WriteLine("Checking copy activity run details...");

    RunFilterParameters filterParams = new RunFilterParameters(
        DateTime.UtcNow.AddMinutes(-10), DateTime.UtcNow.AddMinutes(10)
    );

    ActivityRunsQueryResponse queryResponse = client.ActivityRuns.QueryByPipelineRun(
        resourceGroup, dataFactoryName, runResponse.RunId, filterParams
    );

    if (pipelineRun.Status == "Succeeded")
    {
        Console.WriteLine(queryResponse.Value.First().Output);
    }
    else
        Console.WriteLine(queryResponse.Value.First().Error);

    Console.WriteLine("\nPress any key to exit...");
    Console.ReadKey();
    ```

## <a name="run-the-code"></a>Ejecución del código

Para compilar la aplicación, elija **Compilar** > **Crear solución**. A continuación, para iniciar la aplicación, elija **Depurar** > **Iniciar depuración** y compruebe la ejecución de la canalización.

La consola imprime el progreso de creación de una factoría de datos, un servicio vinculado, conjuntos de datos, una canalización y una ejecución de canalización. A continuación, comprueba el estado de la ejecución de canalización. Espere hasta que vea los detalles de ejecución de actividad de copia con el tamaño de los datos leídos/escritos. A continuación, use herramientas como SQL Server Management Studio (SSMS) o Visual Studio para conectarse a su instancia de Azure SQL Database de destino y compruebe si la tabla especificada contiene los datos copiados.

### <a name="sample-output"></a>Salida de ejemplo

```json
Creating a data factory AdfV2Tutorial...
{
  "identity": {
    "type": "SystemAssigned"
  },
  "location": "East US"
}
Creating linked service AzureStorageLinkedService...
{
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": {
        "type": "SecureString",
        "value": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>"
      }
    }
  }
}
Creating linked service AzureSqlDbLinkedService...
{
  "properties": {
    "type": "AzureSqlDatabase",
    "typeProperties": {
      "connectionString": {
        "type": "SecureString",
        "value": "Server=tcp:<servername>.database.windows.net,1433;Database=<databasename>;User ID=<username>@<servername>;Password=<password>;Trusted_Connection=False;Encrypt=True;Connection Timeout=30"
      }
    }
  }
}
Creating dataset BlobDataset...
{
  "properties": {
    "type": "AzureBlob",
    "typeProperties": {
      "folderPath": "adfv2tutorial/",
      "fileName": "inputEmp.txt",
      "format": {
        "type": "TextFormat",
        "columnDelimiter": "|"
      }
    },
    "structure": [
      {
        "name": "FirstName",
        "type": "String"
      },
      {
        "name": "LastName",
        "type": "String"
      }
    ],
    "linkedServiceName": {
      "type": "LinkedServiceReference",
      "referenceName": "AzureStorageLinkedService"
    }
  }
}
Creating dataset SqlDataset...
{
  "properties": {
    "type": "AzureSqlTable",
    "typeProperties": {
      "tableName": "dbo.emp"
    },
    "linkedServiceName": {
      "type": "LinkedServiceReference",
      "referenceName": "AzureSqlDbLinkedService"
    }
  }
}
Creating pipeline Adfv2TutorialBlobToSqlCopy...
{
  "properties": {
    "activities": [
      {
        "type": "Copy",
        "typeProperties": {
          "source": {
            "type": "BlobSource"
          },
          "sink": {
            "type": "SqlSink"
          }
        },
        "inputs": [
          {
            "type": "DatasetReference",
            "referenceName": "BlobDataset"
          }
        ],
        "outputs": [
          {
            "type": "DatasetReference",
            "referenceName": "SqlDataset"
          }
        ],
        "name": "CopyFromBlobToSQL"
      }
    ]
  }
}
Creating pipeline run...
Pipeline run ID: 1cd03653-88a0-4c90-aabc-ae12d843e252
Checking pipeline run status...
Status: InProgress
Status: InProgress
Status: Succeeded
Checking copy activity run details...
{
  "dataRead": 18,
  "dataWritten": 28,
  "rowsCopied": 2,
  "copyDuration": 2,
  "throughput": 0.01,
  "errors": [],
  "effectiveIntegrationRuntime": "DefaultIntegrationRuntime (East US)",
  "usedDataIntegrationUnits": 2,
  "billedDuration": 2
}

Press any key to exit...
```

## <a name="next-steps"></a>Pasos siguientes

La canalización de este ejemplo copia los datos de una ubicación a otra en una instancia de Azure Blob Storage. Ha aprendido a:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Creación de servicios vinculados de Azure SQL Database y Azure Storage
> * Creación de conjuntos de datos de Azure SQL Database y el blob de Azure
> * Creación de una canalización que contenga una actividad de copia.
> * Inicio de la ejecución de una canalización.
> * Supervisión de las ejecuciones de canalización y actividad.

Pase al tutorial siguiente para obtener información sobre cómo copiar datos desde el entorno local a la nube:

> [!div class="nextstepaction"]
>[Copiar datos del entorno local a la nube](tutorial-hybrid-copy-powershell.md)
