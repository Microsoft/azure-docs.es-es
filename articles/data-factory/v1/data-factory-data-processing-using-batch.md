---
title: Procesamiento de conjuntos de datos a gran escala mediante Data Factory y Batch
description: Se describe cómo procesar grandes cantidades de datos en una canalización de Azure Data Factory mediante la funcionalidad de procesamiento paralelo de Azure Batch.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 3d0e608e1afae77afd44d7351b7c3f1f269bd8a8
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998091"
---
# <a name="process-large-scale-datasets-by-using-data-factory-and-batch"></a>Procesamiento de conjuntos de datos a gran escala mediante Data Factory y Batch
> [!NOTE]
> Este artículo se aplica a la versión 1 de Azure Data Factory, que está disponible con carácter general. Si utiliza la versión actual del servicio Data Factory, consulte [Custom activities in Data Factory](../transform-data-using-dotnet-custom-activity.md) (Actividades personalizadas en Data Factory).

En este artículo se describe la arquitectura de una solución de ejemplo en la que se mueven y se procesan conjuntos de datos a gran escala de forma automática y programada. También se incluye un tutorial de un extremo a otro para implementar la solución con Data Factory y Azure Batch.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Este artículo es más largo de lo habitual, ya que contiene un tutorial de una solución de ejemplo completa. Si no tiene experiencia con Batch y Data Factory, puede informarse sobre estos servicios y cómo interactúan. Si sabe algo sobre los servicios y va a diseñar una solución, puede centrarse en la sección de diseño del artículo. Si va a desarrollar un prototipo o una solución, pruebe con las instrucciones detalladas del tutorial. No dude en hacernos llegar cualquier comentario que tenga sobre el contenido y su uso.

En primer lugar, vamos a ver cómo los servicios Data Factory y Batch pueden ayudar a procesar grandes conjuntos de datos en la nube.     


## <a name="why-azure-batch"></a>¿Por qué elegir Azure Batch?
 Puede usar Batch para ejecutar aplicaciones de informática de alto rendimiento (HPC) en paralelo y a gran escala, de manera eficaz en la nube. Se trata de un servicio de plataforma que programa el trabajo que hace un uso intensivo de los recursos de proceso para que se ejecute en una colección administrada de máquinas virtuales. Pueden escalar automáticamente los recursos de proceso para satisfacer las necesidades de los trabajos.

Con el servicio Batch, se definen los recursos de procesos de Azure para ejecutar las aplicaciones en paralelo y a escala. Puede ejecutar trabajos programados o a petición. No necesita crear, configurar ni administrar manualmente un clúster de HPC, máquinas virtuales individuales, redes virtuales ni una infraestructura compleja de programación de tareas y trabajos.

 Si no está familiarizado con Batch, los siguientes artículos le ayudarán a comprender la arquitectura e implementación de la solución que se describe en este artículo:   

* [Conceptos básicos de Batch](../../azure-sql/database/sql-database-paas-overview.md)
* [Información general de las características de Batch](../../batch/batch-service-workflow-features.md)

Si quiere, para obtener más información sobre Batch, vea la [documentación de Batch](https://docs.microsoft.com/azure/batch/).

## <a name="why-azure-data-factory"></a>¿Por qué elegir Azure Data Factory?
Factoría de datos es un servicio de integración de datos basado en la nube que organiza y automatiza el movimiento y la transformación de datos. Puede utilizar Data Factory para crear canalizaciones de datos administrados que mueven datos desde el entorno local y los almacenes de datos en la nube a un almacén de datos centralizado. Un ejemplo es Azure Blob Storage. Puede usar Data Factory para procesar o transformar datos mediante el uso de servicios como Azure HDInsight y Azure Machine Learning. También puede programar las canalizaciones de datos para que se ejecuten de forma programada (por ejemplo, cada hora, diariamente y semanalmente). Puede supervisar y administrar las canalizaciones de un vistazo para identificar problemas y llevar a cabo acciones.

  Si no está familiarizado con Data Factory, los siguientes artículos le ayudarán a comprender la arquitectura e implementación de la solución que se describe en este artículo:  

* [Introducción al servicio Factoría de datos de Azure](data-factory-introduction.md)
* [Creación de la primera canalización de datos](data-factory-build-your-first-pipeline.md)   

Si quiere, para obtener más información sobre Data Factory, consulte la [documentación de Data Factory](https://docs.microsoft.com/rest/api/datafactory/v1/data-factory-data-factory).

## <a name="data-factory-and-batch-together"></a>Data Factory y Batch juntos
Data Factory incluye actividades integradas. Por ejemplo, la actividad de copia se usa para copiar y mover datos de un almacén de datos de origen a uno de destino. La actividad de Hive se usa para procesar los datos mediante el uso de clústeres de Hadoop (HDInsight) en Azure. Para obtener una lista de actividades de transformación compatibles, vea las [actividades de transformación de datos](data-factory-data-transformation-activities.md).

También puede crear actividades personalizadas de .NET para mover o procesar datos con su propia lógica. Puede ejecutar estas actividades en un clúster de HDInsight o en un grupo de máquinas virtuales de Batch. Cuando use Batch, podrá configurar el grupo para que se escale automáticamente (se agreguen o quiten máquinas virtuales en función de la carga de trabajo) según una fórmula especificada.     

## <a name="architecture-of-a-sample-solution"></a>Arquitectura de una solución de ejemplo
  La arquitectura que se describe en este artículo es para una solución simple. También es relevante para escenarios complejos, como el modelado de riesgos de los servicios financieros, el procesamiento y la representación de imágenes y el análisis de genoma.

En el diagrama se ilustra cómo Data Factory organiza el movimiento y procesamiento de los datos. También se muestra cómo Batch procesa los datos en paralelo. Descargue e imprima el diagrama para facilitar su consulta (11 x 17 pulgadas o tamaño A3). Para acceder al diagrama para que se pueda imprimir, vea [HPC and data orchestration by using Batch and Data Factory](https://go.microsoft.com/fwlink/?LinkId=717686) (Orquestación de datos y HPC con Batch y Data Factory).

[![Diagrama de procesamiento de datos de gran escala](./media/data-factory-data-processing-using-batch/image1.png)](https://go.microsoft.com/fwlink/?LinkId=717686)

La lista siguiente proporciona los pasos básicos del proceso. La solución incluye código y explicaciones para compilar la solución completa.

* **Configure Batch con un grupo de nodos de ejecución (máquinas virtuales)** . Puede especificar el número de nodos y el tamaño de cada nodo.

* **Cree una instancia de Data Factory** que esté configurada con entidades que representen Blob Storage, el servicio de proceso de Batch, los datos de entrada y salida y un flujo de trabajo o una canalización con las actividades que mueven y transforman datos.

* **Cree una actividad personalizada de .NET en la canalización de Data Factory**. La actividad es el código de usuario que se ejecutará en el grupo de Batch.

* **Almacene grandes cantidades de datos de entrada como blobs en Azure Storage**. Los datos se dividen en segmentos lógicos (normalmente, en función de la fecha y la hora).

* **Data Factory copia los datos que se procesan en paralelo** en la ubicación secundaria.

* **Data Factory ejecuta la actividad personalizada con el grupo que asigna Batch**. Además, puede ejecutar actividades al mismo tiempo. Cada actividad procesa un segmento de datos. Los resultados se almacenan en Storage.

* **Data Factory mueve los resultados finales a una tercera ubicación** para que se distribuyan con una aplicación o para que se sigan procesando con otras herramientas.

## <a name="implementation-of-the-sample-solution"></a>Implementación de la solución de ejemplo
La solución de ejemplo es sencilla a propósito. Se ha diseñado para mostrar cómo usar Data Factory y Batch de forma conjunta para procesar conjuntos de datos. La solución cuenta el número de repeticiones del término de búsqueda "Microsoft" en los archivos de entrada que están organizados en una serie temporal. Luego genera el recuento en archivos de salida.

**Tiempo**: si está familiarizado con los aspectos básicos de Azure, Data Factory y Batch, y cumple los requisitos previos que se indican a continuación, esta solución tardará entre una y dos horas en completarse.

### <a name="prerequisites"></a>Requisitos previos
#### <a name="azure-subscription"></a>Suscripción de Azure
Si no tiene una suscripción a Azure, puede crear una cuenta de evaluación gratuita rápidamente. Para más información, vea cómo [obtener una evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).

#### <a name="azure-storage-account"></a>Cuenta de almacenamiento de Azure
En este tutorial, se usa una cuenta de almacenamiento para almacenar los datos. Si aún no dispone de una cuenta de almacenamiento, vea [Create a storage account](../../storage/common/storage-account-create.md) (Creación de una cuenta de almacenamiento). En la solución de ejemplo, se usa Blob Storage.

#### <a name="azure-batch-account"></a>Cuenta de Azure Batch
Cree una cuenta de Batch mediante [Azure Portal](https://portal.azure.com/). Para más información, vea [Creación y administración de una cuenta de Batch](../../batch/batch-account-create-portal.md). Anote el nombre y la clave de la cuenta de Batch. También puede usar el cmdlet [New-AzBatchAccount](https://docs.microsoft.com/powershell/module/az.batch/new-azbatchaccount) para crear una cuenta de Batch. Para obtener instrucciones detalladas para usar este cmdlet, vea [Introducción a los cmdlets de PowerShell de Batch](../../batch/batch-powershell-cmdlets-get-started.md).

La solución de ejemplo usa Batch (de forma indirecta mediante una canalización de Data Factory) para procesar datos en paralelo en un grupo de nodos de ejecución, que es una colección administrada de máquinas virtuales.

#### <a name="azure-batch-pool-of-virtual-machines"></a>Grupo de máquinas virtuales de Azure Batch
Cree un grupo de Batch con al menos dos nodos de ejecución.

1. En [Azure Portal](https://portal.azure.com), seleccione **Examinar** en el menú izquierdo y, luego, **Cuentas de Batch**.

1. Seleccione la cuenta de Batch para abrir la hoja **Cuenta de Batch**.

1. Seleccione el icono **Grupos**.

1. En la hoja **Grupos**, seleccione el botón **Agregar** en la barra de herramientas para agregar un grupo.

   a. Especifique un identificador para el grupo (**Identificador del grupo**). Anote el identificador del grupo. Lo necesitará para crear la solución de Data Factory.

   b. Especifique **Windows Server 2012 R2** en **Familia del sistema operativo**.

   c. Seleccione un **plan de tarifa de nodos**.

   d. Escriba **2** como valor en la configuración **Dedicada a destino**.

   e. Escriba **2** como valor en la configuración **Máximo de tareas por nodo**.

   f. Seleccione **Aceptar** para crear el grupo.

#### <a name="azure-storage-explorer"></a>Explorador de Azure Storage
Use el [Explorador de Azure Storage 6](https://azurestorageexplorer.codeplex.com/) o [CloudXplorer](https://clumsyleaf.com/products/cloudxplorer) (de ClumsyLeaf Software) para inspeccionar y modificar los datos de los proyectos de Storage. También puede inspeccionar y modificar los datos de los registros de las aplicaciones hospedadas en la nube.

1. Cree un contenedor denominado **mycontainer** con acceso privado (sin acceso anónimo).

1. Si usa CloudXplorer, cree carpetas y subcarpetas con la estructura siguiente:

   ![Estructura de carpetas y subcarpetas](./media/data-factory-data-processing-using-batch/image3.png)

   `Inputfolder` y `outputfolder` son carpetas de nivel superior en `mycontainer`. La carpeta `inputfolder` tiene subcarpetas con marcas de fecha y hora (AAAA-MM-DD-HH).

   Si usa el Explorador de Storage, en el paso siguiente, cargue archivos con los nombres siguientes: `inputfolder/2015-11-16-00/file.txt`, `inputfolder/2015-11-16-01/file.txt`, etc. Este paso creará automáticamente las carpetas.

1. Cree un archivo de texto **file.txt** en el equipo con contenido que incluya la palabra clave **Microsoft**. Un ejemplo es "test custom activity Microsoft test custom activity Microsoft".

1. Cargue el archivo a las siguientes carpetas de entrada en Blob Storage:

   ![Carpetas de entrada](./media/data-factory-data-processing-using-batch/image4.png)

   Si usa el Explorador de Storage, cargue el archivo **file.txt** en **mycontainer**. Seleccione **Copiar** en la barra de herramientas para crear una copia del blob. En el cuadro de diálogo **Copy Blob** (Copiar blob), cambie el **nombre del blob de destino** a `inputfolder/2015-11-16-00/file.txt`. Repita este paso para crear `inputfolder/2015-11-16-01/file.txt`, `inputfolder/2015-11-16-02/file.txt`, `inputfolder/2015-11-16-03/file.txt`, `inputfolder/2015-11-16-04/file.txt`, etc. Esta acción crea automáticamente las carpetas.

1. Cree otro contenedor denominado `customactivitycontainer`. Cargue el archivo ZIP de actividad personalizada a este contenedor.

#### <a name="visual-studio"></a>Visual Studio
Instale Visual Studio 2012 o una versión posterior para crear la actividad personalizada de Batch que se utilizará en la solución de Data Factory.

### <a name="high-level-steps-to-create-the-solution"></a>Pasos de alto nivel para crear la solución
1. Cree una actividad personalizada que contenga la lógica de procesamiento de datos.

1. Cree una factoría de datos que use la actividad personalizada.

### <a name="create-the-custom-activity"></a>Creación de la actividad personalizada
La actividad personalizada de factoría de datos es el núcleo de esta solución de ejemplo. La solución de ejemplo usa Batch para ejecutar la actividad personalizada. Para obtener información sobre cómo desarrollar actividades personalizadas y usarlas en las canalizaciones de Data Factory, vea [Uso de actividades personalizadas en una canalización de Azure Data Factory](data-factory-use-custom-activities.md).

Para crear una actividad personalizada de .NET que se pueda usar en una canalización de Data Factory, cree un proyecto de biblioteca de clases .NET con una clase que implemente la interfaz IDotNetActivity. Esta interfaz tiene un solo método: Execute. Esta es la firma de dicho método:

```csharp
public IDictionary<string, string> Execute(
            IEnumerable<LinkedService> linkedServices,
            IEnumerable<Dataset> datasets,
            Activity activity,
            IActivityLogger logger)
```

El método tiene algunos componentes clave que debe conocer:

* El método toma cuatro parámetros:

  * **linkedServices**. Este parámetro es una lista enumerable de servicios vinculados que vincula los orígenes de datos de entrada y salida (por ejemplo: Blob Storage) a la factoría de datos. En este ejemplo, hay solo un servicio vinculado del tipo Azure Storage que se utilice como entrada y salida.
  * **datasets**. Este parámetro es una lista enumerable de conjuntos de datos. Este parámetro se puede usar para obtener las ubicaciones y esquemas que definen los conjuntos de datos de entrada y salida.
  * **activity**. Este parámetro representa la entidad de proceso actual. En este caso, es un servicio de Batch.
  * **logger**. Puede usar el parámetro logger para escribir comentarios de depuración que se muestran como el registro de "User" en la canalización.
* El método devuelve un diccionario que se puede usar para encadenar actividades personalizadas en el futuro. Esta característica todavía no está implementada, así que solo devuelva un diccionario vacío en el método.

#### <a name="procedure-create-the-custom-activity"></a>Procedimiento: Creación de la actividad personalizada
1. Cree un proyecto de biblioteca de clases .NET en Visual Studio.

   a. Inicie Visual Studio 2012/2013/2015.

   b. Seleccione **File (Archivo)**  > **New (Nuevo)**  > **Project (Proyecto)** .

   c. Expanda **Plantillas** y seleccione **Visual C\#** . En este tutorial se usa C\#, pero puede usar cualquier lenguaje .NET para desarrollar la actividad personalizada.

   d. Seleccione **Biblioteca de clases** en la lista de tipos de proyecto de la derecha.

   e. Escriba **MyDotNetActivity** for the **Nombre**.

   f. Seleccione **C:\\ADF** para **Ubicación**. Cree la carpeta **ADF** si no existe.

   g. Seleccione **Aceptar** para crear el proyecto.

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**.

1. En la Consola del Administrador de paquetes, ejecute el siguiente comando para importar Microsoft.Azure.Management.DataFactories:

    ```powershell
    Install-Package Microsoft.Azure.Management.DataFactories
    ```
1. Importe el paquete NuGet de **Azure Storage** en el proyecto. Necesita este paquete porque va a usar la API de Blob Storage en este ejemplo:

    ```powershell
    Install-Package Az.Storage
    ```
1. Agregue las siguientes directivas using al archivo de origen en el proyecto:

    ```csharp
    using System.IO;
    using System.Globalization;
    using System.Diagnostics;
    using System.Linq;

    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Runtime;

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```
1. Cambie el nombre del espacio de nombres por **MyDotNetActivityNS**.

    ```csharp
    namespace MyDotNetActivityNS
    ```
1. Cambie el nombre de la clase a **MyDotNetActivity** y derívela desde la interfaz **IDotNetActivity** tal como se muestra a continuación:

    ```csharp
    public class MyDotNetActivity : IDotNetActivity
    ```
1. Implemente (agregue) el método **Execute** de la interfaz **IDotNetActivity** en la clase **MyDotNetActivity**. Copie el siguiente código de ejemplo en el método. Consulte la sección [Método Execute](#execute-method) para obtener una explicación de la lógica usada en este método.

    ```csharp
    /// <summary>
    /// The Execute method is the only method of IDotNetActivity interface you must implement.
    /// In this sample, the method invokes the Calculate method to perform the core logic.  
    /// </summary>
    public IDictionary<string, string> Execute(
       IEnumerable<LinkedService> linkedServices,
       IEnumerable<Dataset> datasets,
       Activity activity,
       IActivityLogger logger)
    {

       // Declare types for the input and output data stores.
       AzureStorageLinkedService inputLinkedService;

       Dataset inputDataset = datasets.Single(dataset => dataset.Name == activity.Inputs.Single().Name);

       foreach (LinkedService ls in linkedServices)
           logger.Write("linkedService.Name {0}", ls.Name);

       // Use the First method instead of Single because we are using the same
       // Azure Storage linked service for input and output.
       inputLinkedService = linkedServices.First(
           linkedService =>
           linkedService.Name ==
           inputDataset.Properties.LinkedServiceName).Properties.TypeProperties
           as AzureStorageLinkedService;

       string connectionString = inputLinkedService.ConnectionString; // To create an input storage client.
       string folderPath = GetFolderPath(inputDataset);
       string output = string.Empty; // for use later.

       // Create the storage client for input. Pass the connection string.
       CloudStorageAccount inputStorageAccount = CloudStorageAccount.Parse(connectionString);
       CloudBlobClient inputClient = inputStorageAccount.CreateCloudBlobClient();

       // Initialize the continuation token before using it in the do-while loop.
       BlobContinuationToken continuationToken = null;
       do
       {   // get the list of input blobs from the input storage client object.
           BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,
                                    true,
                                    BlobListingDetails.Metadata,
                                    null,
                                    continuationToken,
                                    null,
                                    null);

           // The Calculate method returns the number of occurrences of
           // the search term "Microsoft" in each blob associated
           // with the data slice.
           //
           // The definition of the method is shown in the next step.
           output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

       } while (continuationToken != null);

       // Get the output dataset by using the name of the dataset matched to a name in the Activity output collection.
       Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);

       folderPath = GetFolderPath(outputDataset);

       logger.Write("Writing blob to the folder: {0}", folderPath);

       // Create a storage object for the output blob.
       CloudStorageAccount outputStorageAccount = CloudStorageAccount.Parse(connectionString);
       // Write the name of the file.
       Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));

       logger.Write("output blob URI: {0}", outputBlobUri.ToString());
       // Create a blob and upload the output text.
       CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
       logger.Write("Writing {0} to the output blob", output);
       outputBlob.UploadText(output);

       // The dictionary can be used to chain custom activities together in the future.
       // This feature is not implemented yet, so just return an empty dictionary.
       return new Dictionary<string, string>();
    }
    ```
1. Agregue los siguientes métodos auxiliares a la clase. Estos métodos se invocan desde el método **Execute** . Y lo más importante, el método **Calculate** aísla el código que procesa una iteración de cada blob.

    ```csharp
    /// <summary>
    /// Gets the folderPath value from the input/output dataset.
    /// </summary>
    private static string GetFolderPath(Dataset dataArtifact)
    {
       if (dataArtifact == null || dataArtifact.Properties == null)
       {
           return null;
       }

       AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
       if (blobDataset == null)
       {
           return null;
       }

       return blobDataset.FolderPath;
    }

    /// <summary>
    /// Gets the fileName value from the input/output dataset.
    /// </summary>

    private static string GetFileName(Dataset dataArtifact)
    {
       if (dataArtifact == null || dataArtifact.Properties == null)
       {
           return null;
       }

       AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
       if (blobDataset == null)
       {
           return null;
       }

       return blobDataset.FileName;
    }

    /// <summary>
    /// Iterates through each blob (file) in the folder, counts the number of instances of the search term in the file,
    /// and prepares the output text that is written to the output blob.
    /// </summary>

    public static string Calculate(BlobResultSegment Bresult, IActivityLogger logger, string folderPath, ref BlobContinuationToken token, string searchTerm)
    {
       string output = string.Empty;
       logger.Write("number of blobs found: {0}", Bresult.Results.Count<IListBlobItem>());
       foreach (IListBlobItem listBlobItem in Bresult.Results)
       {
           CloudBlockBlob inputBlob = listBlobItem as CloudBlockBlob;
           if ((inputBlob != null) && (inputBlob.Name.IndexOf("$$$.$$$") == -1))
           {
               string blobText = inputBlob.DownloadText(Encoding.ASCII, null, null, null);
               logger.Write("input blob text: {0}", blobText);
               string[] source = blobText.Split(new char[] { '.', '?', '!', ' ', ';', ':', ',' }, StringSplitOptions.RemoveEmptyEntries);
               var matchQuery = from word in source
                                where word.ToLowerInvariant() == searchTerm.ToLowerInvariant()
                                select word;
               int wordCount = matchQuery.Count();
               output += string.Format("{0} occurrences(s) of the search term \"{1}\" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
           }
       }
       return output;
    }
    ```
    El método GetFolderPath devuelve la ruta de acceso a la carpeta a la que apunta el conjunto de datos y el método GetFileName devuelve el nombre del blob o archivo a la que apunta el conjunto de datos.

    ```csharp

    "name": "InputDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "StorageLinkedService",
        "typeProperties": {
            "fileName": "file.txt",
            "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
    ```

    El método Calculate calcula el número de instancias de la palabra clave "Microsoft" en los archivos de entrada (los blobs de la carpeta). El término de búsqueda "Microsoft" está codificado de forma rígida en el código.

1. Compile el proyecto. Seleccione **Compilar** en el menú y después seleccione **Compilar solución**.

1. Inicie el Explorador de Windows y vaya a la carpeta **bin\\debug** o **bin\\release**. La elección de la carpeta depende del tipo de compilación.

1. Cree un archivo ZIP **MyDotNetActivity.zip** que contenga todos los archivos binarios en la carpeta **\\bin\\Debug**. Puede incluir el archivo MyDotNetActivity.**pdb** para obtener detalles adicionales, como el número de línea en el código fuente que causó el problema, en caso de error.

   ![Lista de la carpeta bin\Debug](./media/data-factory-data-processing-using-batch/image5.png)

1. Cargue **MyDotNetActivity.zip** como blob en el contenedor de blobs `customactivitycontainer` en la instancia de Blob Storage que usa el servicio vinculado StorageLinkedService de ADFTutorialDataFactory. Cree el contenedor de blobs `customactivitycontainer` si aún no existe.

#### <a name="execute-method"></a>Método Execute
En esta sección se proporcionan más detalles sobre el código del método Execute.

1. Los miembros para procesar una iteración en la colección de entrada se encuentran en el espacio de nombres [Microsoft.WindowsAzure.Storage.Blob](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob) . El procesamiento de una iteración en la colección de blobs requiere el uso de la clase **BlobContinuationToken**. En esencia, es preciso usar un bucle do-while con el token como mecanismo para salir del bucle. Para más información, vea [Uso de Blob Storage de .NET](../../storage/blobs/storage-dotnet-how-to-use-blobs.md). Aquí se muestra un bucle básico:

    ```csharp
    // Initialize the continuation token.
    BlobContinuationToken continuationToken = null;
    do
    {
    // Get the list of input blobs from the input storage client object.
    BlobResultSegment blobList = inputClient.ListBlobsSegmented(folderPath,

                         true,
                                   BlobListingDetails.Metadata,
                                   null,
                                   continuationToken,
                                   null,
                                   null);
    // Return a string derived from parsing each blob.

     output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

    } while (continuationToken != null);

    ```
   Para más información, vea la documentación del método [ListBlobsSegmented](https://docs.microsoft.com/java/api/com.microsoft.azure.storage.blob.cloudblobcontainer.listblobssegmented).

1. Lógicamente, el código para trabajar en el conjunto de blobs lógicamente está dentro del bucle do-while. En el método **Execute**, el bucle do-while pasa la lista de blobs a un método denominado **Calculate**. El método devuelve una variable de cadena denominada **output** que es el resultado de haber procesado una iteración en todos los blobs del segmento.

   Devuelve el número de apariciones del término de búsqueda "Microsoft" en el blob que se pasa al método **Calculate**.

    ```csharp
    output += string.Format("{0} occurrences of the search term \"{1}\" were found in the file {2}.\r\n", wordCount, searchTerm, inputBlob.Name);
    ```
1. Una vez que el método **Calculate** ha terminado, debe escribirse en un blob nuevo. Por cada conjunto de blobs que se procesa, se puede escribir un blob nuevo con los resultados. Para escribir en un blob nuevo, primero es preciso buscar el conjunto de datos de salida.

    ```csharp
    // Get the output dataset by using the name of the dataset matched to a name in the Activity output collection.
    Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);
    ```
1. El código también llama al método auxiliar **GetFolderPath** para recuperar la ruta de acceso a la carpeta (el nombre del contenedor de almacenamiento).

    ```csharp
    folderPath = GetFolderPath(outputDataset);
    ```
   El método GetFolderPath convierte el objeto DataSet en una clase AzureBlobDataSet, que tiene una propiedad denominada FolderPath.

    ```csharp
    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

    return blobDataset.FolderPath;
    ```
1. El código llama al método **GetFileName** para recuperar el nombre de archivo (nombre del blob). El código es similar al código anterior usado para obtener la ruta de acceso a la carpeta.

    ```csharp
    AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;

    return blobDataset.FileName;
    ```
1. El nombre del archivo se escribe mediante la creación de un objeto URI. El constructor URI usa la propiedad **BlobEndpoint** para devolver el nombre del contenedor. Tanto la ruta de la carpeta como el nombre de archivo se agregan al construir el identificador URI del blob de salida.  

    ```csharp
    // Write the name of the file.
    Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));
    ```
1. Una vez escrito el nombre del archivo, puede escribir la cadena de salida desde el método **Calculate** en un blob nuevo:

    ```csharp
    // Create a blob and upload the output text.
    CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
    logger.Write("Writing {0} to the output blob", output);
    outputBlob.UploadText(output);
    ```

### <a name="create-the-data-factory"></a>Creación de la factoría de datos
En la sección [Creación de la actividad personalizada](#create-the-custom-activity), se creó una actividad personalizada y se cargó el archivo ZIP con archivos binarios y el archivo PDB en un contenedor de blobs. En esta sección, creará una factoría de datos con una canalización que usa la actividad personalizada.

El conjunto de datos de entrada de la actividad personalizada representa los blobs (archivos) de la carpeta de entrada (`mycontainer\\inputfolder`) de Blob Storage. El conjunto de datos de salida de la actividad representa los blobs de salida de la carpeta de salida (`mycontainer\\outputfolder`) de Blob Storage.

Coloque uno o varios archivos en las carpetas de entrada:

```
mycontainer -\> inputfolder
    2015-11-16-00
    2015-11-16-01
    2015-11-16-02
    2015-11-16-03
    2015-11-16-04
```

Por ejemplo, coloque un archivo (file.txt) con el siguiente contenido en cada una de las carpetas:

```
test custom activity Microsoft test custom activity Microsoft
```

Cada carpeta de entrada corresponde a un segmento de la factoría de datos, aunque la carpeta contenga dos o más archivos. Cuando la canalización procesa cada segmento, la actividad personalizada procesa una iteración en todos los blobs de la carpeta de entrada de dicho segmento.

Verá cinco archivos de salida con el mismo contenido. Por ejemplo, el archivo de salida resultante de procesar el archivo en la carpeta 2015-11-16-00 tiene el siguiente contenido:

```
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
```

Si coloca varios archivos (file.txt, file2.txt, file3.txt) con el mismo contenido en la carpeta de entrada, verá el siguiente contenido en el archivo de salida. Cada carpeta (2015-11-16-00, etc.) corresponde a un segmento en este ejemplo, incluso si la carpeta tiene varios archivos de entrada.

```csharp
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file2.txt.
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file3.txt.
```

El archivo de salida tiene ahora tres líneas, una por cada archivo de entrada (blob) en la carpeta asociada con el segmento (2015-11-16-00).

Se crea una tarea para cada ejecución de actividad. En este ejemplo, solamente hay una actividad en la canalización. Cuando se procesa un segmento en la canalización, la actividad personalizada se ejecuta en Batch para procesar el segmento. Puesto que hay cinco segmentos (cada segmento puede tener varios blobs o archivos), se crearán cinco tareas en Batch. Cuando se ejecuta una tarea en Batch, lo que se ejecuta es la actividad personalizada.

En el siguiente tutorial, se proporcionan más detalles.

#### <a name="step-1-create-the-data-factory"></a>Paso 1: Creación de la factoría de datos
1. Tras iniciar sesión en [Azure Portal](https://portal.azure.com/), siga estos pasos:

   a. Seleccione **NUEVO** en el menú de la izquierda.

   b. Seleccione **Datos y análisis** en la hoja **Nuevo**.

   c. Haga clic en **Data Factory** en la hoja **Análisis de datos**.

1. En la hoja **Nueva factoría de datos**, escriba **CustomActivityFactory** en el campo Nombre. El nombre de la factoría de datos debe ser globalmente único. Si aparece el error "El nombre CustomActivityFactory de factoría de datos no está disponible", cambie dicho nombre. Por ejemplo, use sunombreCustomActivityFactory y vuelva a crear la factoría de datos.

1. Seleccione **NOMBRE DEL GRUPO DE RECURSOS** y seleccione un grupo de recursos existente o cree uno.

1. Compruebe que la suscripción y la región en las que desea que se cree la factoría de datos son correctas.

1. Seleccione **Crear** en la hoja **Nueva factoría de datos**.

1. La factoría de datos se crea en el panel del portal.

1. Tras crear correctamente la factoría de datos, se ve la página de **Data Factory**, que muestra el contenido de la misma.

   ![Página Data Factory](./media/data-factory-data-processing-using-batch/image6.png)

#### <a name="step-2-create-linked-services"></a>Paso 2: Crear servicios vinculados
Los servicios vinculados vinculan almacenes de datos o servicios de proceso con una factoría de datos. En este paso, vinculará la cuenta de almacenamiento y la cuenta de Batch con su factoría de datos.

#### <a name="create-an-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage
1. Seleccione el icono **Crear e implementar** de la hoja **Data Factory** de **CustomActivityFactory**. Aparece Data Factory Editor.

1. Seleccione **Nuevo almacén de datos** en la barra de comandos y elija **Azure Storage**. El script JSON que usa para crear un servicio vinculado de Storage aparece en el editor.

   ![Nuevo almacén de datos](./media/data-factory-data-processing-using-batch/image7.png)

1. Reemplace **account name** por el nombre de su cuenta de almacenamiento. Reemplace **account key** por la clave de acceso de la cuenta de almacenamiento. Para aprender a obtener la clave de acceso de almacenamiento, consulte [Administración de claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md).

1. Seleccione **Implementar** en la barra de comandos para implementar el servicio vinculado.

   ![Implementación](./media/data-factory-data-processing-using-batch/image8.png)

#### <a name="create-an-azure-batch-linked-service"></a>Creación de un servicio vinculado de Azure Batch
En este paso, creará un servicio vinculado para su cuenta de Batch que se usará para ejecutar la actividad personalizada de Data Factory.

1. Seleccione **Nuevo proceso** en la barra de comandos y elija **Azure Batch**. El script JSON que usa para crear un servicio vinculado de Batch aparece en el editor.

1. En el script JSON:

   a. Reemplace **account name** por el nombre de la cuenta de Batch.

   b. Reemplace **access key** por la clave de acceso de la cuenta de Batch.

   c. Escriba el identificador del grupo de la propiedad **poolName**. Para esta propiedad, puede especificar cualquier nombre de grupo o identificador de grupo.

   d. Escriba el identificador URI de lote para la propiedad **batchUri** de JSON.

      > [!IMPORTANT]
      > La dirección URL de la hoja **Cuenta de Batch** tiene el formato siguiente: \<accountname\>.\<region\>.batch.azure.com. Para la propiedad **batchUri** en el script JSON, necesita quitar a88"accountname."** de la dirección URL. Un ejemplo es `"batchUri": "https://eastus.batch.azure.com"`.
      >
      >

      ![Hoja Cuenta de Batch](./media/data-factory-data-processing-using-batch/image9.png)

      En la propiedad **poolName**, también puede especificar el identificador del grupo, en lugar del nombre del grupo.

      > [!NOTE]
      > El servicio Data Factory no admite una opción a petición para Batch como lo hace para HDInsight. Puede usar solo su propio grupo de Batch en una factoría de datos.
      >
      >

   e. Especifique **StorageLinkedService** for the **linkedServiceName** . Ha creado este servicio vinculado en el paso anterior. Este almacenamiento se usa como área de almacenamiento provisional para archivos y registros.

1. Seleccione **Implementar** en la barra de comandos para implementar el servicio vinculado.

#### <a name="step-3-create-datasets"></a>Paso 3: Creación de conjuntos de datos
En este paso, crea conjuntos de datos que representan los datos de entrada y salida.

#### <a name="create-the-input-dataset"></a>Creación del conjunto de datos de entrada
1. En Data Factory Editor, seleccione el botón **Nuevo conjunto de datos** en la barra de herramientas. Seleccione **Azure Blob Storage** en la lista desplegable.

1. Reemplace el script JSON en el panel derecho con el siguiente fragmento de código JSON:

    ```json
    {
       "name": "InputDataset",
       "properties": {
           "type": "AzureBlob",
           "linkedServiceName": "AzureStorageLinkedService",
           "typeProperties": {
               "folderPath": "mycontainer/inputfolder/{Year}-{Month}-{Day}-{Hour}",
               "format": {
                   "type": "TextFormat"
               },
               "partitionedBy": [
                   {
                       "name": "Year",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "yyyy"
                       }
                   },
                   {
                       "name": "Month",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "MM"
                       }
                   },
                   {
                       "name": "Day",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "dd"
                       }
                   },
                   {
                       "name": "Hour",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "HH"
                       }
                   }
               ]
           },
           "availability": {
               "frequency": "Hour",
               "interval": 1
           },
           "external": true,
           "policy": {}
       }
    }
    ```

    Más adelante, en este mismo tutorial, creará una canalización cuya hora de inicio es 2015-11-16T00:00:00Z, y cuya hora de finalización es 2015-11-16T05:00:00Z. Está programada para generar datos cada hora, por lo que hay cinco segmentos de entrada o salida (entre **00**:00:00 -\> **05**:00:00).

    Los valores de **frequency** e **interval** del conjunto de datos de entrada se establecen en **Hour** y **1** respectivamente, lo que significa que el segmento de entrada está disponible cada hora.

    La hora de inicio se representa mediante la variable del sistema **SliceStart** en el fragmento de código JSON anterior. Estas son las horas de inicio para cada segmento.

    | **Segmento** | **Hora de inicio**          |
    |-----------|-------------------------|
    | 1         | 2015-11-16T**00**:00:00 |
    | 2         | 2015-11-16T**01**:00:00 |
    | 3         | 2015-11-16T**02**:00:00 |
    | 4         | 2015-11-16T**03**:00:00 |
    | 5         | 2015-11-16T**04**:00:00 |

    La propiedad **folderPath** se calcula usando la parte de año, mes, día y hora de la hora de inicio del segmento (**SliceStart**). Aquí se muestra cómo se asigna una carpeta de entrada a un segmento.

    | **Segmento** | **Hora de inicio**          | **Carpeta de entrada**  |
    |-----------|-------------------------|-------------------|
    | 1         | 2015-11-16T**00**:00:00 | 2015-11-16-**00** |
    | 2         | 2015-11-16T**01**:00:00 | 2015-11-16-**01** |
    | 3         | 2015-11-16T**02**:00:00 | 2015-11-16-**02** |
    | 4         | 2015-11-16T**03**:00:00 | 2015-11-16-**03** |
    | 5         | 2015-11-16T**04**:00:00 | 2015-11-16-**04** |

1. Seleccione **Implementar** en la barra de herramientas para crear e implementar la tabla **InputDataset**.

#### <a name="create-the-output-dataset"></a>Creación del conjunto de datos de salida
En este paso, creará otro conjunto de datos de tipo AzureBlob para representar los datos de salida.

1. En Data Factory Editor, seleccione el botón **Nuevo conjunto de datos** en la barra de herramientas. Seleccione **Azure Blob Storage** en la lista desplegable.

1. Reemplace el script JSON en el panel derecho con el siguiente fragmento de código JSON:

    ```json
    {
       "name": "OutputDataset",
       "properties": {
           "type": "AzureBlob",
           "linkedServiceName": "AzureStorageLinkedService",
           "typeProperties": {
               "fileName": "{slice}.txt",
               "folderPath": "mycontainer/outputfolder",
               "partitionedBy": [
                   {
                       "name": "slice",
                       "value": {
                           "type": "DateTime",
                           "date": "SliceStart",
                           "format": "yyyy-MM-dd-HH"
                       }
                   }
               ]
           },
           "availability": {
               "frequency": "Hour",
               "interval": 1
           }
       }
    }
    ```

    Se genera un blob o archivo de salida para cada segmento de entrada. Así es cómo se asigna el nombre al archivo de salida de cada segmento. Todos los archivos de salida se generan en una sola carpeta de salida, `mycontainer\\outputfolder`.

    | **Segmento** | **Hora de inicio**          | **Archivo de salida**       |
    |-----------|-------------------------|-----------------------|
    | 1         | 2015-11-16T**00**:00:00 | 2015-11-16-**00.txt** |
    | 2         | 2015-11-16T**01**:00:00 | 2015-11-16-**01.txt** |
    | 3         | 2015-11-16T**02**:00:00 | 2015-11-16-**02.txt** |
    | 4         | 2015-11-16T**03**:00:00 | 2015-11-16-**03.txt** |
    | 5         | 2015-11-16T**04**:00:00 | 2015-11-16-**04.txt** |

    Recuerde que todos los archivos de una carpeta de entrada (por ejemplo, 2015-11-16-00) forman parte de un segmento con la hora de inicio 2015-11-16-00. Cuando se procesa este segmento, la actividad personalizada examinan todos los archivos y produce una línea en el archivo de salida con el número de apariciones del término de búsqueda "Microsoft". Si hay tres archivos en la carpeta 2015-11-16-00, habrá tres líneas en el archivo de salida 2015-11-16-00.txt.

1. Seleccione **Implementar** en la barra de herramientas para crear e implementar **OutputDataset**.

#### <a name="step-4-create-and-run-the-pipeline-with-a-custom-activity"></a>Paso 4: Creación y ejecución de la canalización con una actividad personalizada
En este paso, creará una canalización con la actividad personalizada que creó antes.

> [!IMPORTANT]
> Si no ha cargado **file.txt** a las carpetas de entrada en el contenedor de blobs, hágalo antes de crear la canalización. La propiedad **isPaused** está establecida en false en el código JSON de la canalización, por lo que esta se ejecutará de inmediato, porque la fecha de inicio (**start**) ya ha pasado.
>
>

1. En Data Factory Editor, haga clic en **Nueva canalización** en la barra de comandos. Si no ve el comando, seleccione el símbolo de puntos suspensivos para que se muestre.

1. Reemplace el script JSON en el panel derecho con el siguiente fragmento de código JSON:

    ```json
    {
       "name": "PipelineCustom",
       "properties": {
           "description": "Use custom activity",
           "activities": [
               {
                   "type": "DotNetActivity",
                   "typeProperties": {
                       "assemblyName": "MyDotNetActivity.dll",
                       "entryPoint": "MyDotNetActivityNS.MyDotNetActivity",
                       "packageLinkedService": "AzureStorageLinkedService",
                       "packageFile": "customactivitycontainer/MyDotNetActivity.zip"
                   },
                   "inputs": [
                       {
                           "name": "InputDataset"
                       }
                   ],
                   "outputs": [
                       {
                           "name": "OutputDataset"
                       }
                   ],
                   "policy": {
                       "timeout": "00:30:00",
                       "concurrency": 5,
                       "retry": 3
                   },
                   "scheduler": {
                       "frequency": "Hour",
                       "interval": 1
                   },
                   "name": "MyDotNetActivity",
                   "linkedServiceName": "AzureBatchLinkedService"
               }
           ],
           "start": "2015-11-16T00:00:00Z",
           "end": "2015-11-16T05:00:00Z",
           "isPaused": false
      }
    }
    ```
   Tenga en cuenta los siguientes puntos:

   * Solo hay una actividad en la canalización y es del tipo **DotNetActivity**.
   * **AssemblyName** se establece en el nombre del archivo DLL **MyDotNetActivity.dll**.
   * **EntryPoint** se establece **MyDotNetActivityNS.MyDotNetActivity**. Es básicamente \<namespace\>.\<classname\> en el código.
   * **PackageLinkedService** está establecido en **StorageLinkedService**, que apunta a la instancia de Blob Storage que contiene el archivo ZIP de la actividad personalizada. Si usa diferentes cuentas de almacenamiento para los archivos de entrada y salida y el archivo ZIP de actividad personalizada, tendrá que crear otro servicio vinculado de Storage. En este artículo se asume que usa la misma cuenta de almacenamiento.
   * **PackageFile** se establece en **customactivitycontainer/MyDotNetActivity.zip**. Está en el formato \<containerforthezip\>/\<nameofthezip.zip\>.
   * La actividad personalizada toma **InputDataset** como entrada y **OutputDataset** como salida.
   * La propiedad **linkedServiceName** de la actividad personalizada apunta a **AzureBatchLinkedService**, que indica a Data Factory que la actividad personalizada debe ejecutarse en Batch.
   * La configuración **concurrency** es importante. Si usa el valor predeterminado, que es 1, aunque tenga dos o más nodos de ejecución en el grupo de Batch, los segmentos se procesarán uno tras otro. Por lo tanto, no se aprovecha la ventaja de la funcionalidad de procesamiento paralelo de Batch. Si establece **concurrency** en un valor mayor, por ejemplo 2, significa que se pueden procesar dos segmentos (que se corresponden con dos tareas en Batch) al mismo tiempo. En este caso,se usan ambas máquinas virtuales en el grupo de Batch. Establezca la propiedad concurrency correctamente.
   * De manera predeterminada, solo se ejecuta una tarea (segmento) en una máquina virtual en un momento dado. De forma predeterminada, la opción **Maximum tasks per VM** (Máximo de tareas por máquina virtual) está establecida en 1 para un grupo de Batch. Como parte de los requisitos previos, creó un grupo con esta propiedad establecida en 2. Por lo tanto, dos segmentos de la factoría de datos se pueden ejecutar en una máquina virtual al mismo tiempo.
     - La propiedad **isPaused** se establece en false de forma predeterminada. La canalización se ejecuta inmediatamente en este ejemplo, ya que los segmentos se inician en el pasado. Esta propiedad se puede establecer en **true** para pausar la canalización y se puede volver a establecer en **false** para reiniciarla.
     -   Las horas de inicio (**start**) y fin (**end**) se separan por un intervalo de cinco horas. Los segmentos se generan cada hora, por lo que la canalización crea cinco segmentos.

1. Seleccione **Implementar** en la barra de comandos para implementar la canalización.

#### <a name="step-5-test-the-pipeline"></a>Paso 5: Prueba de la canalización
En este paso, probará la canalización colocando archivos en las carpetas de entrada. Empiece por probar la canalización con un archivo por cada carpeta de entrada.

1. En la hoja **Data Factory** de Azure Portal, seleccione **Diagrama**.

   ![Diagrama](./media/data-factory-data-processing-using-batch/image10.png)

1. En la vista de **diagrama**, haga doble clic en el conjunto de datos de entrada **InputDataset**.

   ![InputDataset](./media/data-factory-data-processing-using-batch/image11.png)

1. La hoja**InputDataset** se abre con los cinco segmentos listos. Observe los valores de **HORA DE INICIO DE SEGMENTO** y **HORA DE FINALIZACIÓN DE SEGMENTO** para cada segmento.

   ![Horas de inicio y finalización del segmento de entrada](./media/data-factory-data-processing-using-batch/image12.png)

1. En la vista **Diagrama**, seleccione **OutputDataset**.

1. Los cinco segmentos de salida aparecen con el estado **Listo** si se han generado.

   ![Horas de inicio y finalización del segmento de salida](./media/data-factory-data-processing-using-batch/image13.png)

1. Use Azure Portal para ver las tareas asociadas con los segmentos y para comprobar en qué máquina virtual se ejecuta cada segmento. Para más información, vea la sección [Integración de Data Factory y Batch](#data-factory-and-batch-integration).

1. Los archivos de salida aparecen en `mycontainer` en `outputfolder` en Blob Storage.

   ![Archivos de salida de Storage](./media/data-factory-data-processing-using-batch/image15.png)

   Se enumeran cinco archivos de salida, uno para cada segmento de entrada. Cada archivo de salida tiene un contenido similar a la siguiente salida:

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-00/file.txt.
    ```
   En el siguiente diagrama, se ilustra cómo los segmentos de Data Factory se asignan a tareas de Batch. En este ejemplo, un segmento tiene solo una ejecución.

   ![Diagrama de asignación de segmentos](./media/data-factory-data-processing-using-batch/image16.png)

1. Ahora, pruebe con varios archivos en una carpeta. Cree los archivos **file2.txt**, **file3.txt**, **file4.txt** y **file5.txt** con el mismo contenido que file.txt en la carpeta **2015-11-06-01**.

1. En la carpeta de salida, elimine el archivo de salida **2015-11-16-01.txt**.

1. En la hoja **OutputDataset**, haga clic con el botón derecho en el segmento con el valor de **HORA DE INICIO DE SEGMENTO** establecido en **11/16/2015 01:00:00 AM**. Seleccione **Ejecutar** para volver a ejecutar o procesar el segmento. Ahora, el segmento tiene cinco archivos en lugar de uno.

    ![Ejecute](./media/data-factory-data-processing-using-batch/image17.png)

1. Una vez que se ejecuta el segmento y su estado es **Listo**, compruebe el contenido del archivo de salida para este segmento (**2015-11-16-01.txt**). El archivo de salida aparece en `mycontainer` en `outputfolder` en Blob Storage. Debería aparecer una línea para cada archivo del segmento.

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file2.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file3.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file4.txt.
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2015-11-16-01/file5.txt.
    ```

> [!NOTE]
> Si no ha eliminado el archivo de salida 2015-11-16-01.txt antes de probar con cinco archivos de entrada, verá una línea de la anterior ejecución del segmento y cinco líneas de la actual. De forma predeterminada, se anexa el contenido al archivo de salida si ya existe.
>
>

#### <a name="data-factory-and-batch-integration"></a>Integración de Data Factory y Batch
El servicio Data Factory crea un trabajo en Batch llamado `adf-poolname:job-xxx`.

![Trabajos por lotes](media/data-factory-data-processing-using-batch/data-factory-batch-jobs.png)

Se crea una tarea en el trabajo para cada ejecución de actividad de un segmento. Si hay diez segmentos listos para procesarse, se crean diez tareas en este trabajo. Si el grupo tiene varios nodos de procesos, puede haber más de un segmento en ejecución en paralelo. Si el número máximo de tareas por nodo de ejecución está establecido en más de una, significa que se puede ejecutar más de un segmento en el mismo proceso.

En este ejemplo, hay cinco segmentos y, por tanto, cinco tareas en Batch. Ahora que el valor **concurrency** está establecido en **5** en el script JSON de la canalización de Data Factory y que **Maximum tasks per VM** (Máximo de tareas por máquina virtual) está establecido en **2** en el grupo de Batch con **2** máquinas virtuales, las tareas se ejecutarán muy rápido. (Compruebe las horas de inicio y finalización de las tareas).

Use el portal para ver el trabajo de Batch y las tareas que están asociadas con los segmentos y para comprobar en qué máquina virtual se ejecuta cada segmento.

![Tareas de trabajos por lotes](media/data-factory-data-processing-using-batch/data-factory-batch-job-tasks.png)

### <a name="debug-the-pipeline"></a>Depuración de la canalización
La depuración se compone de varias técnicas básicas.

1. Si el segmento de entrada no está establecido en **Listo**, confirme que la estructura de la carpeta de entrada es correcta y que file.txt se encuentra en las carpetas de entrada.

   ![Estructura de carpetas de entrada](./media/data-factory-data-processing-using-batch/image3.png)

1. En el método **Execute** de la actividad personalizada, use el objeto **IActivityLogger** para registrar información que ayude a solucionar problemas. Los mensajes registrados se mostrarán en el archivo user\_0.log.

   En la hoja **OutputDataset**, seleccione el segmento para ver la hoja **Segmento de datos** de dicho segmento. En **Ejecuciones de actividad**, debe aparecer una ejecución de actividad del segmento. Si selecciona **Ejecutar** en la barra de comandos, podrá iniciar otra ejecución de actividad en el mismo segmento.

   Al seleccionar la ejecución de actividad, verá la hoja **Detalles de la ejecución de actividad** con una lista de archivos de registro. Los mensajes registrados se encuentran en el archivo user\_0.log. Si se produce un error, verá tres ejecuciones de actividad, ya que el número de reintentos está establecido en 3 en la canalización o actividad JSON. Al seleccionar la ejecución de actividad, verá los archivos de registro que puede revisar para solucionar el error.

   ![Hojas OutputDataset y Segmento de datos](./media/data-factory-data-processing-using-batch/image18.png)

   En la lista de archivos de registro, seleccione **user-0.log**. En el panel derecho, aparecen los resultados del uso del método **IActivityLogger.Write**.

   ![Hoja Detalles de la ejecución de actividad](./media/data-factory-data-processing-using-batch/image19.png)

   Compruebe system-0.log para ver si hay alguna excepción o mensaje de error del sistema.

    ```
    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Loading assembly file MyDotNetActivity...

    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Creating an instance of MyDotNetActivityNS.MyDotNetActivity from assembly file MyDotNetActivity...

    Trace\_T\_D\_12/6/2015 1:43:35 AM\_T\_D\_\_T\_D\_Verbose\_T\_D\_0\_T\_D\_Executing Module

    Trace\_T\_D\_12/6/2015 1:43:38 AM\_T\_D\_\_T\_D\_Information\_T\_D\_0\_T\_D\_Activity e3817da0-d843-4c5c-85c6-40ba7424dce2 finished successfully
    ```
1. Incluya el archivo **PDB** en el archivo ZIP para que los detalles sobre un error contengan información como la pila de llamadas cuando se produce un error.

1. Todos los archivos incluidos en el archivo ZIP de la actividad personalizada deben estar en el nivel superior; no debe haber subcarpetas.

   ![Lista de archivos ZIP de la actividad personalizada](./media/data-factory-data-processing-using-batch/image20.png)

1. Asegúrese de que en **assemblyName** (MyDotNetActivity.dll), **entryPoint** (MyDotNetActivityNS.MyDotNetActivity), **packageFile** (customactivitycontainer/MyDotNetActivity.zip) y **packageLinkedService** (debe apuntar a la instancia de Blob Storage que contiene el archivo ZIP) se han establecido los valores correctos.

1. Si corrigió algún error y desea volver a procesar el segmento, haga clic con el botón derecho en el segmento en la hoja **OutputDataset** y seleccione **Ejecutar**.

   ![Opción Ejecutar de la hoja OutputDataset](./media/data-factory-data-processing-using-batch/image21.png)

   > [!NOTE]
   > En la instancia de Blob Storage se almacena un contenedor llamado `adfjobs`. Este contenedor no se elimina automáticamente, pero puede eliminarlo de forma segura cuando termine de probar la solución. De la misma forma, la solución de Data Factory crea un trabajo de Batch llamado `adf-\<pool ID/name\>:job-0000000001`. Puede eliminarlo después de probar la solución, si lo desea.
   >
   >
1. La actividad personalizada no utiliza el archivo **app.config** desde el paquete. Por lo tanto, si el código lee cualquier cadena de conexión del archivo de configuración, no funcionará en tiempo de ejecución. El procedimiento recomendado cuando se usa Batch es almacenar los secretos en Azure Key Vault. A continuación, use a una entidad de servicio basada en certificados para proteger el almacén de claves y distribuir el certificado al grupo de Batch. Tras ello, la actividad personalizada de .NET podrá acceder a los secretos desde el almacén de claves en tiempo de ejecución. Esta solución genérica se puede extrapolar a cualquier tipo de secreto, no solo a cadenas de conexión.

    Hay una solución alternativa más sencilla, pero no se trata de un procedimiento recomendado. Puede crear un servicio vinculado de SQL Database con la configuración de una cadena de conexión. A continuación, puede crear un conjunto de datos que utilice el servicio vinculado y encadene el conjunto de datos como un conjunto de datos de entrada ficticio con la actividad de .NET personalizada. Tras ello, podrá acceder a la cadena de conexión del servicio vinculado del código de la actividad personalizada. Debería funcionar bien en tiempo de ejecución.  

#### <a name="extend-the-sample"></a>Extensión del ejemplo
Puede extender este ejemplo para obtener más información sobre las características Data Factory y Batch. Por ejemplo, para procesar los segmentos de un intervalo de tiempo diferente, realice los siguientes pasos:

1. Agregue las siguientes subcarpetas en `inputfolder`: 2015-11-16-05, 2015-11-16-06, 201-11-16-07, 2011-11-16-08 y 2015-11-16-09. Coloque los archivos de entrada en esas carpetas. Cambie la hora de finalización de la canalización de `2015-11-16T05:00:00Z` a `2015-11-16T10:00:00Z`. En la vista **Diagrama**, haga doble clic en **InputDataset** y confirme que los segmentos de entrada están listos. Haga doble clic en **OuptutDataset** para ver el estado de los segmentos de salida. Si se encuentran en el estado **Listo**, compruebe si los archivos de salida se encuentran en la carpeta de salida.

1. Aumente o disminuya la configuración de **concurrency** para saber cómo afecta al rendimiento de la solución, especialmente el procesamiento que se produce en Batch. Para más información sobre la **simultaneidad**, consulte "Paso 4: Creación y ejecución de la canalización con una actividad personalizada".

1. Cree un grupo con un valor mayor o menor en **Máximo de tareas por máquina virtual**. Actualice el servicio vinculado Batch en la solución de Data Factory para que use el nuevo grupo que creó. Para más información sobre el **Máximo de tareas por máquina virtual**, consulte "Paso 4: Creación y ejecución de la canalización con una actividad personalizada".

1. Cree un grupo de Batch con la característica de **escalado automático**. El escalado automático de los nodos de ejecución de un grupo de Batch es el ajuste dinámico de la potencia de procesamiento que usa su aplicación.

    La fórmula del ejemplo obtiene el comportamiento siguiente. Cuando el grupo se crea inicialmente, se inicia con una máquina virtual. La métrica $PendingTasks define el número de tareas que están en ejecución y activas (en cola). La fórmula busca el número promedio de tareas pendientes en los últimos 180 segundos y establece TargetDedicated en consecuencia. Garantiza que TargetDedicated nunca supera las 25 VM. El grupo crece automáticamente a medida que se envían nuevas tareas. Cuando las tareas finalizan, las máquinas virtuales quedan libres una a una y el escalado automático reduce esas máquinas virtuales. Puede adaptar startingNumberOfVMs y maxNumberofVMs a sus necesidades.

    Fórmula de escalado automático:

    ```
    startingNumberOfVMs = 1;
    maxNumberofVMs = 25;
    pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
    pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
    $TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
    ```

   Para más información, vea [Escalado automático de los nodos de ejecución en un grupo de Batch](../../batch/batch-automatic-scaling.md).

   Si el grupo usa el valor predeterminado de la propiedad [autoScaleEvaluationInterval](https://msdn.microsoft.com/library/azure/dn820173.aspx), el servicio Batch puede tardar de quince a treinta minutos en preparar la máquina virtual antes de ejecutar la actividad personalizada. Si el grupo usa otro valor de autoScaleEvaluationInterval diferente, el servicio Batch puede tardar el valor de autoScaleEvaluationInterval más diez minutos.

1. En la solución de ejemplo, el método **Execute** invoca al método **Calculate** que procesa un segmento de datos de entrada para generar un segmento de datos de salida. Puede escribir su propio método para procesar los datos de entrada y reemplazar la llamada al método **Calculate** en el método **Execute** por una llamada a su método.

### <a name="next-steps-consume-the-data"></a>Pasos siguientes: Consumo de datos
Después de procesar datos, puede consumirlos con herramientas en línea como Power BI. Estos vínculos lo ayudarán a comprender Power BI y aprender a usarlo en Azure:

* [Exploración de un conjunto de datos en Power BI](https://powerbi.microsoft.com/documentation/powerbi-service-get-data/)
* [Introducción a Power BI Desktop](https://docs.microsoft.com/power-bi/fundamentals/desktop-getting-started)
* [Actualizar datos en Power BI](https://powerbi.microsoft.com/documentation/powerbi-refresh-data/)
* [Azure y Power BI Información general básica](https://powerbi.microsoft.com/documentation/powerbi-azure-and-power-bi/)

## <a name="references"></a>Referencias
* [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/)

  * [Introducción al servicio Data Factory](data-factory-introduction.md)
  * [Introducción a Data Factory](data-factory-build-your-first-pipeline.md)
  * [Uso de actividades personalizadas en una canalización de Data Factory](data-factory-use-custom-activities.md)
* [Azure Batch](https://azure.microsoft.com/documentation/services/batch/)

  * [Conceptos básicos de Batch](../../azure-sql/database/sql-database-paas-overview.md)
  * [Información general sobre las características de Batch](../../batch/batch-service-workflow-features.md)
  * [Creación y administración de una cuenta de Batch en Azure Portal](../../batch/batch-account-create-portal.md)
  * [Introducción a la biblioteca cliente de Batch para .NET](../../batch/quick-run-dotnet.md)

[batch-explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[batch-explorer-walkthrough]: https://blogs.technet.com/b/windowshpc/archive/2015/01/20/azure-batch-explorer-sample-walkthrough.aspx
