---
title: Creación de canalizaciones de datos mediante el SDK de .NET de Azure
description: Conozca cómo puede crear, supervisar y administrar factorías de datos de Azure mediante programación con el SDK de Factoría de datos.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/22/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: c9dce90b0dc563fb0fa7ecaaa0882167eef444a0
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019902"
---
# <a name="create-monitor-and-manage-azure-data-factories-using-azure-data-factory-net-sdk"></a>Creación, supervisión y administración de factorías de datos de Azure mediante el SDK de .NET de Azure Data Factory
> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte el [tutorial de la actividad de copia](../quickstart-create-data-factory-dot-net.md). 

## <a name="overview"></a>Información general
Puede crear, supervisar y administrar factorías de datos de Azure mediante programación con el SDK de .NET de la factoría de datos. Este artículo contiene un tutorial que puede seguir para crear una aplicación de consola .NET de ejemplo que crea y controla una factoría de datos. 

> [!NOTE]
> Este artículo no abarca todas las API de .NET de Data Factory. Para obtener documentación completa sobre la API de .NET para Data Factory, consulte la [referencia de API de .NET de Data Factory](/dotnet/api/index?view=azuremgmtdatafactories-4.12.1). 

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

* Visual Studio 2012, 2013 o 2015
* Descargue e instale el [SDK de .NET de Azure](https://azure.microsoft.com/downloads/).
* Azure PowerShell. Siga las instrucciones del artículo [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para instalar Azure PowerShell en su equipo. Azure PowerShell se usa para crear una aplicación de Azure Active Directory.

### <a name="create-an-application-in-azure-active-directory"></a>Creación de una aplicación en Azure Active Directory
Cree una aplicación de Azure Active Directory, cree una entidad de servicio para dicha aplicación y asígnela al rol **Colaborador de Data Factory** .

1. Inicie **PowerShell**.
2. Ejecute el siguiente comando y escriba el nombre de usuario y la contraseña que utiliza para iniciar sesión en el Portal de Azure.

    ```powershell
    Connect-AzAccount
    ```
3. Ejecute el siguiente comando para ver todas las suscripciones para esta cuenta.

    ```powershell
    Get-AzSubscription
    ```
4. Ejecute el comando siguiente para seleccionar la suscripción con la que desea trabajar. Reemplace **&lt;NameOfAzureSubscription**&gt; por el nombre de su suscripción de Azure.

    ```powershell
    Get-AzSubscription -SubscriptionName <NameOfAzureSubscription> | Set-AzContext
    ```

   > [!IMPORTANT]
   > Anote **SubscriptionId** y **TenantId** de la salida de este comando.

5. Cree un grupo de recursos de Azure con el nombre **ADFTutorialResourceGroup** ejecutando el siguiente comando en PowerShell.

    ```powershell
    New-AzResourceGroup -Name ADFTutorialResourceGroup  -Location "West US"
    ```

    Si el grupo de recursos ya existe, especifique si desea actualizarlo (Y) o mantenerlo como está (N).

    Si usa un otro grupo de recursos, deberá usar su nombre en lugar de ADFTutorialResourceGroup en este tutorial.
6. Cree una aplicación de Azure Active Directory.

    ```powershell
    $azureAdApplication = New-AzADApplication -DisplayName "ADFDotNetWalkthroughApp" -HomePage "https://www.contoso.org" -IdentifierUris "https://www.adfdotnetwalkthroughapp.org/example" -Password "Pass@word1"
    ```

    Si aparece el siguiente error, especifique otra dirección URL distinta y vuelva a ejecutar el comando.
    
    ```powershell
    Another object with the same value for property identifierUris already exists.
    ```
7. Cree la entidad de servicio de AD.

    ```powershell
    New-AzADServicePrincipal -ApplicationId $azureAdApplication.ApplicationId
    ```
8. Agregue la entidad de servicio al rol **Colaborador de Data Factory** .

    ```powershell
    New-AzRoleAssignment -RoleDefinitionName "Data Factory Contributor" -ServicePrincipalName $azureAdApplication.ApplicationId.Guid
    ```
9. Obtenga el identificador de aplicación.

    ```powershell
    $azureAdApplication 
    ```
    Anote el identificador de aplicación (applicationID) de la salida.

Debe tener los cuatro valores siguientes de estos pasos:

* Id. de inquilino
* Id. de suscripción
* Identificador de aplicación
* Contraseña (especificado en el primer comando)

## <a name="walkthrough"></a>Tutorial
En el tutorial, crea una factoría de datos con una canalización que contiene una actividad de copia. Esta actividad copia los datos de una carpeta de Azure Blob Storage a otra carpeta de la misma ubicación. 

La actividad de copia realiza el movimiento de datos en Azure Data Factory. La actividad funciona con un servicio disponible de forma global que puede copiar datos entre varios almacenes de datos de forma segura, confiable y escalable. Consulte el artículo [Actividades de movimiento de datos](data-factory-data-movement-activities.md) para obtener más información sobre la actividad de copia.

1. Con Visual Studio 2012, 2013 o 2015, cree una aplicación de consola .NET de C#.
   1. Inicie **Visual Studio** 2012/2013/2015.
   2. Haga clic en **Archivo**, seleccione **Nuevo** y, luego, haga clic en **Proyecto**.
   3. Expanda **Plantillas** y seleccione **Visual C#** . En este tutorial, se usa C# pero puede usar cualquier lenguaje .NET.
   4. Seleccione **Aplicación de consola** en la lista de tipos de proyecto de la derecha.
   5. Escriba **DataFactoryAPITestApp** en Nombre.
   6. Seleccione **C:\ADFGetStarted** para Ubicación.
   7. Haga clic en **Aceptar** para crear el proyecto.
2. Haga clic en **Herramientas**, seleccione **Administrador de paquetes de NuGet** y haga clic en **Consola del Administrador de paquetes**.
3. En la **Consola del Administrador de paquetes**, siga estos pasos:
   1. Ejecute el comando siguiente para instalar el paquete de Data Factory: `Install-Package Microsoft.Azure.Management.DataFactories`
   2. Ejecute el comando siguiente para instalar el paquete de Azure Active Directory (utilizará la API de Active Directory en el código): `Install-Package Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.19.208020213`
4. Reemplace el contenido del archivo **App.config** del proyecto por el siguiente contenido: 
    
    ```xml
    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <appSettings>
            <add key="ActiveDirectoryEndpoint" value="https://login.microsoftonline.com/" />
            <add key="ResourceManagerEndpoint" value="https://management.azure.com/" />
            <add key="WindowsManagementUri" value="https://management.core.windows.net/" />

            <add key="ApplicationId" value="your application ID" />
            <add key="Password" value="Password you used while creating the AAD application" />
            <add key="SubscriptionId" value= "Subscription ID" />
            <add key="ActiveDirectoryTenantId" value="Tenant ID" />
        </appSettings>
    </configuration>
    ```
5. En el archivo App.Config, actualice los valores de **&lt;Id. de aplicación&gt;** , **&lt;Contraseña&gt;** , **&lt;Id. de suscripción&gt;** e **&lt;Id. de inquilino&gt;** por los suyos propios.
6. Agregue las siguientes instrucciones **using** al archivo **Program.cs** del proyecto.

    ```csharp
    using System.Configuration;
    using System.Collections.ObjectModel;
    using System.Threading;
    using System.Threading.Tasks;

    using Microsoft.Azure;
    using Microsoft.Azure.Management.DataFactories;
    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Common.Models;

    using Microsoft.IdentityModel.Clients.ActiveDirectory;

    ```
6. Agregue el código siguiente que crea una instancia de la clase **DataPipelineManagementClient** al método **Main**. Este objeto se usa para crear una factoría de datos, un servicio vinculado, conjuntos de datos de entrada y salida, y una canalización. También se usa para supervisar los segmentos de un conjunto de datos en tiempo de ejecución.

    ```csharp
    // create data factory management client

    //IMPORTANT: specify the name of Azure resource group here
    string resourceGroupName = "ADFTutorialResourceGroup";

    //IMPORTANT: the name of the data factory must be globally unique.
    // Therefore, update this value. For example:APITutorialFactory05122017
    string dataFactoryName = "APITutorialFactory";

    TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
            ConfigurationManager.AppSettings["SubscriptionId"],
            GetAuthorizationHeader().Result);

    Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

    DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);
    ```

   > [!IMPORTANT]
   > Reemplace el valor de **resourcegroupname** por el nombre de su grupo de recursos de Azure. Puede crear un grupo de recursos con el cmdlet [New-AzureResourceGroup](/powershell/module/az.resources/new-azresourcegroup) .
   >
   > Actualice el nombre de Data Factory (dataFactoryName) para que sea único. El nombre de la factoría de datos debe ser único a nivel global. Consulte el tema [Factoría de datos: reglas de nomenclatura](data-factory-naming-rules.md) para las reglas de nomenclatura para los artefactos de Factoría de datos.
7. Agregue el siguiente código que crea una **factoría de datos** en el método **Main**.

    ```csharp
    // create a data factory
    Console.WriteLine("Creating a data factory");
    client.DataFactories.CreateOrUpdate(resourceGroupName,
        new DataFactoryCreateOrUpdateParameters()
        {
            DataFactory = new DataFactory()
            {
                Name = dataFactoryName,
                Location = "westus",
                Properties = new DataFactoryProperties()
            }
        }
    );
    ```
8. Agregue el siguiente código que crea un servicio vinculado de **Azure Storage** al método **Main**.

   > [!IMPORTANT]
   > Reemplace **storageaccountname** y **accountkey** por el nombre y la clave de su cuenta de Azure Storage.

    ```csharp
    // create a linked service for input data store: Azure Storage
    Console.WriteLine("Creating Azure Storage linked service");
    client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
        new LinkedServiceCreateOrUpdateParameters()
        {
            LinkedService = new LinkedService()
            {
                Name = "AzureStorageLinkedService",
                Properties = new LinkedServiceProperties
                (
                    new AzureStorageLinkedService("DefaultEndpointsProtocol=https;AccountName=<storageaccountname>;AccountKey=<accountkey>")
                )
            }
        }
    );
    ```
9. Agregue el siguiente código que crea **conjuntos de datos de entrada y salida** en el método **Main**.

    Tenga en cuenta que el valor de **FolderPath** para el blob de entrada está establecido en **adftutorial/** , donde **adftutorial** es el nombre del contenedor en el almacenamiento de blobs. Si este contenedor no existe en el almacenamiento de blobs de Azure, cree un contenedor con este nombre: **adftutorial** y cargue un archivo de texto al contenedor.

    Tenga en cuenta que el valor de FolderPath para el blob de salida está establecido en: **adftutorial/apifactoryoutput/{Segmento}** , donde **Segmento** se calcula dinámicamente en función del valor de **SliceStart** (fecha y hora de inicio de cada segmento).

    ```csharp
    // create input and output datasets
    Console.WriteLine("Creating input and output datasets");
    string Dataset_Source = "DatasetBlobSource";
    string Dataset_Destination = "DatasetBlobDestination";
    
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Source,
            Properties = new DatasetProperties()
            {
                LinkedServiceName = "AzureStorageLinkedService",
                TypeProperties = new AzureBlobDataset()
                {
                    FolderPath = "adftutorial/",
                    FileName = "emp.txt"
                },
                External = true,
                Availability = new Availability()
                {
                    Frequency = SchedulePeriod.Hour,
                    Interval = 1,
                },
    
                Policy = new Policy()
                {
                    Validation = new ValidationPolicy()
                    {
                        MinimumRows = 1
                    }
                }
            }
        }
    });
    
    client.Datasets.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new DatasetCreateOrUpdateParameters()
    {
        Dataset = new Dataset()
        {
            Name = Dataset_Destination,
            Properties = new DatasetProperties()
            {
    
                LinkedServiceName = "AzureStorageLinkedService",
                TypeProperties = new AzureBlobDataset()
                {
                    FolderPath = "adftutorial/apifactoryoutput/{Slice}",
                    PartitionedBy = new Collection<Partition>()
                    {
                        new Partition()
                        {
                            Name = "Slice",
                            Value = new DateTimePartitionValue()
                            {
                                Date = "SliceStart",
                                Format = "yyyyMMdd-HH"
                            }
                        }
                    }
                },
    
                Availability = new Availability()
                {
                    Frequency = SchedulePeriod.Hour,
                    Interval = 1,
                },
            }
        }
    });
    ```
10. Agregue el siguiente código que **crea y activa una canalización** al método **Main**. Esta canalización tiene un elemento **CopyActivity** que toma **BlobSource** como origen y **BlobSink** como receptor.

    La actividad de copia realiza el movimiento de datos en Azure Data Factory. La actividad funciona con un servicio disponible de forma global que puede copiar datos entre varios almacenes de datos de forma segura, confiable y escalable. Consulte el artículo [Actividades de movimiento de datos](data-factory-data-movement-activities.md) para obtener más información sobre la actividad de copia.

    ```csharp
    // create a pipeline
    Console.WriteLine("Creating a pipeline");
    DateTime PipelineActivePeriodStartTime = new DateTime(2014, 8, 9, 0, 0, 0, 0, DateTimeKind.Utc);
    DateTime PipelineActivePeriodEndTime = PipelineActivePeriodStartTime.AddMinutes(60);
    string PipelineName = "PipelineBlobSample";
    
    client.Pipelines.CreateOrUpdate(resourceGroupName, dataFactoryName,
    new PipelineCreateOrUpdateParameters()
    {
        Pipeline = new Pipeline()
        {
            Name = PipelineName,
            Properties = new PipelineProperties()
            {
                Description = "Demo Pipeline for data transfer between blobs",
    
                // Initial value for pipeline's active period. With this, you won't need to set slice status
                Start = PipelineActivePeriodStartTime,
                End = PipelineActivePeriodEndTime,
    
                Activities = new List<Activity>()
                {
                    new Activity()
                    {
                        Name = "BlobToBlob",
                        Inputs = new List<ActivityInput>()
                        {
                            new ActivityInput()
                {
                                Name = Dataset_Source
                            }
                        },
                        Outputs = new List<ActivityOutput>()
                        {
                            new ActivityOutput()
                            {
                                Name = Dataset_Destination
                            }
                        },
                        TypeProperties = new CopyActivity()
                        {
                            Source = new BlobSource(),
                            Sink = new BlobSink()
                            {
                                WriteBatchSize = 10000,
                                WriteBatchTimeout = TimeSpan.FromMinutes(10)
                            }
                        }
                    }
    
                },
            }
        }
    });
    ```
12. Agregue el código siguiente al método **Main** para obtener el estado de un segmento de datos del conjunto de datos de salida. Solo se espera un segmento en este ejemplo.

    ```csharp
    // Pulling status within a timeout threshold
    DateTime start = DateTime.Now;
    bool done = false;
    
    while (DateTime.Now - start < TimeSpan.FromMinutes(5) && !done)
    {
        Console.WriteLine("Pulling the slice status");
        // wait before the next status check
        Thread.Sleep(1000 * 12);
    
        var datalistResponse = client.DataSlices.List(resourceGroupName, dataFactoryName, Dataset_Destination,
            new DataSliceListParameters()
            {
                DataSliceRangeStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString(),
                DataSliceRangeEndTime = PipelineActivePeriodEndTime.ConvertToISO8601DateTimeString()
            });
    
        foreach (DataSlice slice in datalistResponse.DataSlices)
        {
            if (slice.State == DataSliceState.Failed || slice.State == DataSliceState.Ready)
            {
                Console.WriteLine("Slice execution is done with status: {0}", slice.State);
                done = true;
                break;
            }
            else
            {
                Console.WriteLine("Slice status is: {0}", slice.State);
            }
        }
    }
    ```
13. **(Opcional)** : agregue el código siguiente para obtener detalles de ejecución de un segmento de datos en el método **Main**.

    ```csharp
    Console.WriteLine("Getting run details of a data slice");
    
    // give it a few minutes for the output slice to be ready
    Console.WriteLine("\nGive it a few minutes for the output slice to be ready and press any key.");
    Console.ReadKey();
    
    var datasliceRunListResponse = client.DataSliceRuns.List(
        resourceGroupName,
        dataFactoryName,
        Dataset_Destination,
        new DataSliceRunListParameters()
        {
            DataSliceStartTime = PipelineActivePeriodStartTime.ConvertToISO8601DateTimeString()
        });
    
    foreach (DataSliceRun run in datasliceRunListResponse.DataSliceRuns)
    {
        Console.WriteLine("Status: \t\t{0}", run.Status);
        Console.WriteLine("DataSliceStart: \t{0}", run.DataSliceStart);
        Console.WriteLine("DataSliceEnd: \t\t{0}", run.DataSliceEnd);
        Console.WriteLine("ActivityId: \t\t{0}", run.ActivityName);
        Console.WriteLine("ProcessingStartTime: \t{0}", run.ProcessingStartTime);
        Console.WriteLine("ProcessingEndTime: \t{0}", run.ProcessingEndTime);
        Console.WriteLine("ErrorMessage: \t{0}", run.ErrorMessage);
    }
    
    Console.WriteLine("\nPress any key to exit.");
    Console.ReadKey();
    ```
14. Agregue el siguiente método auxiliar usado por el método **Main** a la clase **Program**. Este método abre un cuadro de diálogo que le permite proporcionar un **nombre de usuario** y una **contraseña** para iniciar sesión en Azure Portal.

    ```csharp
    public static async Task<string> GetAuthorizationHeader()
    {
        AuthenticationContext context = new AuthenticationContext(ConfigurationManager.AppSettings["ActiveDirectoryEndpoint"] + ConfigurationManager.AppSettings["ActiveDirectoryTenantId"]);
        ClientCredential credential = new ClientCredential(
            ConfigurationManager.AppSettings["ApplicationId"],
            ConfigurationManager.AppSettings["Password"]);
        AuthenticationResult result = await context.AcquireTokenAsync(
            resource: ConfigurationManager.AppSettings["WindowsManagementUri"],
            clientCredential: credential);

        if (result != null)
            return result.AccessToken;

        throw new InvalidOperationException("Failed to acquire token");
    }
    ```

15. En el Explorador de soluciones, expanda el proyecto **DataFactoryAPITestApp**, haga clic con el botón derecho en **Referencias** y haga clic en **Agregar referencia**. Seleccione la casilla del ensamblado de `System.Configuration` y haga clic en **Aceptar**.
15. Compile la aplicación de la consola. Haga clic en **Compilar** en el menú y en **Compilar solución**.
16. Confirme que hay al menos un archivo en el contenedor de adftutorial del almacenamiento de blobs de Azure. En caso contrario, cree el archivo Emp.txt en el Bloc de notas con el siguiente contenido y cárguelo en el contenedor de adftutorial.

    ```
    John, Doe
    Jane, Doe
    ```
17. Para ejecutar el ejemplo haga clic en **Depurar** -> **Iniciar depuración** en el menú. Cuando vea **Getting run details of a data slice**, espere unos minutos y presione **ENTRAR**.
18. Use el Portal de Azure para comprobar que la factoría de datos **APITutorialFactory** se crea con los siguientes artefactos:
    * Servicio vinculado: **AzureStorageLinkedService**
    * Conjunto de datos: **DatasetBlobSource** y **DatasetBlobDestination**.
    * Canalización: **PipelineBlobSample**
19. Compruebe que se crea un archivo de salida en la carpeta **apifactoryoutput** del contenedor **adftutorial**.

## <a name="get-a-list-of-failed-data-slices"></a>Obtener una lista de segmentos de datos con errores 

```csharp
// Parse the resource path
var ResourceGroupName = "ADFTutorialResourceGroup";
var DataFactoryName = "DataFactoryAPITestApp";

var parameters = new ActivityWindowsByDataFactoryListParameters(ResourceGroupName, DataFactoryName);
parameters.WindowState = "Failed";
var response = dataFactoryManagementClient.ActivityWindows.List(parameters);
do
{
    foreach (var activityWindow in response.ActivityWindowListResponseValue.ActivityWindows)
    {
        var row = string.Join(
            "\t",
            activityWindow.WindowStart.ToString(),
            activityWindow.WindowEnd.ToString(),
            activityWindow.RunStart.ToString(),
            activityWindow.RunEnd.ToString(),
            activityWindow.DataFactoryName,
            activityWindow.PipelineName,
            activityWindow.ActivityName,
            string.Join(",", activityWindow.OutputDatasets));
        Console.WriteLine(row);
    }

    if (response.NextLink != null)
    {
        response = dataFactoryManagementClient.ActivityWindows.ListNext(response.NextLink, parameters);
    }
    else
    {
        response = null;
    }
}
while (response != null);
```

## <a name="next-steps"></a>Pasos siguientes
Vea el ejemplo siguiente para crear una canalización mediante el SDK para .NET que copia datos de un almacenamiento de blobs de Azure a Azure SQL Database: 

- [Crear una canalización para copiar datos de Blob Storage a SQL Database](data-factory-copy-activity-tutorial-using-dotnet-api.md)
