---
title: Uso de actividades personalizadas en una canalización de Azure Data Factory
description: Obtenga información acerca de cómo crear actividades personalizadas y usarlas en una canalización de Azure Data Factory.
services: data-factory
documentationcenter: ''
ms.assetid: 8dd7ba14-15d2-4fd9-9ada-0b2c684327e9
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
author: nabhishek
ms.author: abnarain
ms.custom: devx-track-csharp
manager: anandsub
robots: noindex
ms.openlocfilehash: b8935b9f2c3c598aee7c5d0eb37f21d8114dac42
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88997462"
---
# <a name="use-custom-activities-in-an-azure-data-factory-version-1-pipeline"></a>Uso de actividades personalizadas en una canalización de Azure Data Factory versión 1
> [!div class="op_single_selector" title1="Seleccione la versión del servicio Data Factory que usa:"]
> * [Versión 1](data-factory-use-custom-activities.md)
> * [Versión 2 (versión actual)](../transform-data-using-dotnet-custom-activity.md)

> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si utiliza la versión actual del servicio Data Factory, consulte [Custom activities in V2](../transform-data-using-dotnet-custom-activity.md) (Actividades personalizadas en V2).

Hay dos tipos de actividades que puede usar en una canalización de Azure Data Factory.

- [Actividades de movimiento de datos](data-factory-data-movement-activities.md) para mover datos entre [almacenes de datos de origen y receptor compatibles](data-factory-data-movement-activities.md#supported-data-stores-and-formats).
- [Actividades de transformación de datos](data-factory-data-transformation-activities.md) para transformar datos mediante procesos como Azure HDInsight, Azure Batch y Azure Machine Learning.

Para mover datos a o desde un almacén de datos no compatible con Data Factory, cree una **actividad personalizada** con su propia lógica de movimiento de datos y utilícela en una canalización. De forma similar, para transformar y procesar datos de algún modo no compatible con Data Factory, cree una actividad personalizada con su propia lógica de transformación de datos y utilícela en una canalización.

Puede configurar una actividad personalizada para que se ejecute en un grupo de máquinas virtuales de **Azure Batch**. Si usa Azure Batch, solo puede utilizar un grupo de Azure Batch existente.

En el siguiente tutorial se proporcionan instrucciones paso a paso para crear una actividad de .NET personalizada y utilizarla en una canalización. En el tutorial se utiliza un servicio vinculado de **Azure Batch**.

> [!IMPORTANT]
> - No es posible usar una puerta de enlace de administración de datos procedente de una actividad personalizada para acceder a orígenes de datos locales. Actualmente, la [puerta de enlace de administración de datos](data-factory-data-management-gateway.md) solo admite las actividades de copia y de procedimiento almacenado en Data Factory.

## <a name="walkthrough-create-a-custom-activity"></a>Tutorial: creación de una actividad personalizada
### <a name="prerequisites"></a>Requisitos previos
* Visual Studio 2012/2013/2015/2017
* Descargue e instale el [SDK de .NET de Azure](https://azure.microsoft.com/downloads/)

### <a name="azure-batch-prerequisites"></a>Requisitos previos de Azure Batch
En el tutorial, ejecutará sus actividades de .NET personalizadas con Azure Batch como recurso de proceso. **Azure Batch** es un servicio de plataforma para ejecutar aplicaciones en paralelo a gran escala y de informática de alto rendimiento (HPC) de manera eficaz en la nube. Azure Batch programa el trabajo de proceso intensivo para que se ejecute en una **colección administrada de máquinas virtuales** y puede escalar automáticamente los recursos de proceso para satisfacer las necesidades de sus trabajos. Consulte el artículo [Datos básicos de Azure Batch][batch-technical-overview] para ver una descripción detallada del servicio Azure Batch.

Para el tutorial, cree una cuenta de Azure Batch con un grupo de máquinas virtuales. He aquí los pasos:

1. Cree una **cuenta de Azure Batch** en [Azure Portal](https://portal.azure.com). Para obtener instrucciones, consulte el artículo [Creación y administración de una cuenta de Azure Batch][batch-create-account].
2. Anote el nombre y la clave de la cuenta de Azure Batch el URI y el nombre del grupo. Los necesita para crear un servicio vinculado de Azure Batch.
    1. En la página de inicio de la cuenta de Azure Batch, verá una **dirección URL** con el siguiente formato: `https://myaccount.westus.batch.azure.com`. En este ejemplo, **myaccount** es el nombre de la cuenta de Azure Batch. El URI que se utiliza en la definición del servicio vinculado es la dirección URL sin el nombre de la cuenta. Por ejemplo: `https://<region>.batch.azure.com`.
    2. Haga clic en **Claves** en el menú de la izquierda y copie la **CLAVE DE ACCESO PRIMARIA**.
    3. Para usar un grupo existente, haga clic en **Grupos** en el menú y anote el **identificador** del grupo. Si no tienes ningún grupo, ve al paso siguiente.
2. Cree un **grupo de Azure Batch**.

   1. En [Azure Portal](https://portal.azure.com), haga clic en **Examinar** en el menú izquierdo y haga clic en **Cuentas de Batch**.
   2. Seleccione la cuenta de Azure Batch para abrir la hoja **Cuenta de Batch**.
   3. Haga clic en el icono **Grupos** .
   4. En la hoja **Grupos** , haga clic en el botón Agregar en la barra de herramientas para agregar un grupo.
      1. Especifique un identificador para el grupo (Identificador del grupo). Anote el **identificador del grupo**; lo necesitará al crear la solución de Data Factory.
      2. Especifique **Windows Server 2012 R2** en Familia del sistema operativo.
      3. Seleccione un **plan de tarifa de nodos**.
      4. Escriba **2** como valor en la configuración **Dedicada a destino**.
      5. Escriba **2** como valor en la configuración **Máximo de tareas por nodo**.
   5. Haga clic en **Aceptar** para crear el grupo.
   6. Anote el **identificador** del grupo.

### <a name="high-level-steps"></a>Pasos de alto nivel
Estos son los dos pasos de alto nivel que se realizan como parte de este tutorial:

1. Cree una actividad personalizada que contenga una lógica de procesamiento o transformación de datos simple.
2. Cree una instancia de Azure Data Factory con una canalización que use la actividad personalizada.

### <a name="create-a-custom-activity"></a>creación de una actividad personalizada
Para crear una actividad personalizada de .NET, cree un proyecto de la **biblioteca de clases .NET** con una clase que implemente la interfaz **IDotNetActivity**. Esta interfaz tiene un solo método: [Ejecutar](https://msdn.microsoft.com/library/azure/mt603945.aspx) y su firma es:

```csharp
public IDictionary<string, string> Execute(
    IEnumerable<LinkedService> linkedServices,
    IEnumerable<Dataset> datasets,
    Activity activity,
    IActivityLogger logger)
```

El método toma cuatro parámetros:

- **linkedServices**. Esta propiedad es una lista enumerable de los servicios vinculados al almacén de datos a los que hacen referencia los conjuntos de datos de entrada y salida de la actividad.
- **datasets**. Esta propiedad es una lista enumerable de los conjuntos de datos de entrada/salida de la actividad. Este parámetro se puede usar para obtener las ubicaciones y esquemas que definen los conjuntos de datos de entrada y salida.
- **activity**. Esta propiedad representa la actividad actual. Se puede utilizar para acceder a propiedades extendidas asociadas a la actividad personalizada. Consulte [Acceso a las propiedades extendidas](#access-extended-properties) para más información.
- **logger**. Este objeto permite escribir comentarios de depuración que se muestran en el registro de usuario en la canalización.

El método devuelve un diccionario que se puede usar para encadenar actividades personalizadas en el futuro. Esta característica todavía no está implementada, así que devuelve un diccionario vacío del método.

### <a name="procedure"></a>Procedimiento
1. Cree un proyecto de **biblioteca de clases .NET** .
   <ol type="a">
     <li>Inicie Visual Studio.</li>
     <li>Haga clic en <b>Archivo</b>, seleccione <b>Nuevo</b> y, luego, haga clic en <b>Proyecto</b>.</li>
     <li>Expanda <b>Plantillas</b> y seleccione <b>Visual C#</b>. En este tutorial se usa C#, pero puede usar cualquier lenguaje .NET para desarrollar la actividad personalizada.</li>
     <li>Seleccione <b>Biblioteca de clases</b> en la lista de tipos de proyecto de la derecha. En Visual Studio, elija <b>Biblioteca de clases (.NET Framework)</b> </li>
     <li>Escriba <b>MyDotNetActivity</b> for the <b>Nombre</b>.</li>
     <li>Seleccione <b>C:\ADFGetStarted</b> como <b>Ubicación</b>.</li>
     <li>Haga clic en <b>Aceptar</b> para crear el proyecto.</li>
   </ol>

2. Haga clic en **Herramientas**, seleccione **Administrador de paquetes de NuGet** y haga clic en **Consola del Administrador de paquetes**.

3. En la Consola del Administrador de paquetes, ejecute el siguiente comando para importar **Microsoft.Azure.Management.DataFactories**.

    ```powershell
    Install-Package Microsoft.Azure.Management.DataFactories
    ```
4. Importe el paquete NuGet de **Azure Storage** en el proyecto.

    ```powershell
    Install-Package WindowsAzure.Storage -Version 4.3.0
    ```

    > [!IMPORTANT]
    > El iniciador del servicio Data Factory requiere la versión 4.3 de WindowsAzure.Storage. Si agrega una referencia a una versión posterior del ensamblado de Azure Storage en el proyecto de actividad personalizada, verá un error cuando se ejecuta la actividad. Para resolver el error, consulte la sección [Aislamiento de AppDomain](#appdomain-isolation).
5. Agregue las siguientes instrucciones **using** al archivo de origen en el proyecto.

    ```csharp

    // Comment these lines if using VS 2017
    using System.IO;
    using System.Globalization;
    using System.Diagnostics;
    using System.Linq;
    // --------------------

    // Comment these lines if using <= VS 2015
    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Text;
    using System.Threading.Tasks;
    // ---------------------

    using Microsoft.Azure.Management.DataFactories.Models;
    using Microsoft.Azure.Management.DataFactories.Runtime;

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```
6. Cambie el nombre del **espacio de nombres** por **MyDotNetActivityNS**.

    ```csharp
    namespace MyDotNetActivityNS
    ```
7. Cambie el nombre de la clase por **MyDotNetActivity** y derívela desde la interfaz **IDotNetActivity** como se muestra en el siguiente fragmento de código:

    ```csharp
    public class MyDotNetActivity : IDotNetActivity
    ```
8. Implemente (agregue) el método **Execute** de la interfaz **IDotNetActivity** en la clase **MyDotNetActivity** y copie el siguiente código de ejemplo en el método.

    El ejemplo siguiente cuenta el número de apariciones del término de búsqueda (“Microsoft”) en cada blob asociado con un segmento de datos.

    ```csharp
    /// <summary>
    /// Execute method is the only method of IDotNetActivity interface you must implement.
    /// In this sample, the method invokes the Calculate method to perform the core logic.
    /// </summary>

    public IDictionary<string, string> Execute(
        IEnumerable<LinkedService> linkedServices,
        IEnumerable<Dataset> datasets,
        Activity activity,
        IActivityLogger logger)
    {
        // get extended properties defined in activity JSON definition
        // (for example: SliceStart)
        DotNetActivity dotNetActivity = (DotNetActivity)activity.TypeProperties;
        string sliceStartString = dotNetActivity.ExtendedProperties["SliceStart"];

        // to log information, use the logger object
        // log all extended properties
        IDictionary<string, string> extendedProperties = dotNetActivity.ExtendedProperties;
        logger.Write("Logging extended properties if any...");
        foreach (KeyValuePair<string, string> entry in extendedProperties)
        {
            logger.Write("<key:{0}> <value:{1}>", entry.Key, entry.Value);
        }

        // linked service for input and output data stores
        // in this example, same storage is used for both input/output
        AzureStorageLinkedService inputLinkedService;

        // get the input dataset
        Dataset inputDataset = datasets.Single(dataset => dataset.Name == activity.Inputs.Single().Name);

        // declare variables to hold type properties of input/output datasets
        AzureBlobDataset inputTypeProperties, outputTypeProperties;

        // get type properties from the dataset object
        inputTypeProperties = inputDataset.Properties.TypeProperties as AzureBlobDataset;

        // log linked services passed in linkedServices parameter
        // you will see two linked services of type: AzureStorage
        // one for input dataset and the other for output dataset
        foreach (LinkedService ls in linkedServices)
            logger.Write("linkedService.Name {0}", ls.Name);

        // get the first Azure Storage linked service from linkedServices object
        // using First method instead of Single since we are using the same
        // Azure Storage linked service for input and output.
        inputLinkedService = linkedServices.First(
            linkedService =>
            linkedService.Name ==
            inputDataset.Properties.LinkedServiceName).Properties.TypeProperties
            as AzureStorageLinkedService;

        // get the connection string in the linked service
        string connectionString = inputLinkedService.ConnectionString;

        // get the folder path from the input dataset definition
        string folderPath = GetFolderPath(inputDataset);
        string output = string.Empty; // for use later.

        // create storage client for input. Pass the connection string.
        CloudStorageAccount inputStorageAccount = CloudStorageAccount.Parse(connectionString);
        CloudBlobClient inputClient = inputStorageAccount.CreateCloudBlobClient();

        // initialize the continuation token before using it in the do-while loop.
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

            // Calculate method returns the number of occurrences of
            // the search term (“Microsoft”) in each blob associated
            // with the data slice. definition of the method is shown in the next step.

            output = Calculate(blobList, logger, folderPath, ref continuationToken, "Microsoft");

        } while (continuationToken != null);

        // get the output dataset using the name of the dataset matched to a name in the Activity output collection.
        Dataset outputDataset = datasets.Single(dataset => dataset.Name == activity.Outputs.Single().Name);

        // get type properties for the output dataset
        outputTypeProperties = outputDataset.Properties.TypeProperties as AzureBlobDataset;

        // get the folder path from the output dataset definition
        folderPath = GetFolderPath(outputDataset);

        // log the output folder path
        logger.Write("Writing blob to the folder: {0}", folderPath);

        // create a storage object for the output blob.
        CloudStorageAccount outputStorageAccount = CloudStorageAccount.Parse(connectionString);
        // write the name of the file.
        Uri outputBlobUri = new Uri(outputStorageAccount.BlobEndpoint, folderPath + "/" + GetFileName(outputDataset));

        // log the output file name
        logger.Write("output blob URI: {0}", outputBlobUri.ToString());

        // create a blob and upload the output text.
        CloudBlockBlob outputBlob = new CloudBlockBlob(outputBlobUri, outputStorageAccount.Credentials);
        logger.Write("Writing {0} to the output blob", output);
        outputBlob.UploadText(output);

        // The dictionary can be used to chain custom activities together in the future.
        // This feature is not implemented yet, so just return an empty dictionary.

        return new Dictionary<string, string>();
    }
    ```
9. Agregue los siguientes métodos auxiliares:

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

        // get type properties of the dataset
        AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
        if (blobDataset == null)
        {
            return null;
        }

        // return the folder path found in the type properties
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

        // get type properties of the dataset
        AzureBlobDataset blobDataset = dataArtifact.Properties.TypeProperties as AzureBlobDataset;
        if (blobDataset == null)
        {
            return null;
        }

        // return the blob/file name in the type properties
        return blobDataset.FileName;
    }

    /// <summary>
    /// Iterates through each blob (file) in the folder, counts the number of instances of search term in the file,
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

    El método GetFolderPath devuelve la ruta de acceso a la carpeta a la que apunta el conjunto de datos y el método GetFileName devuelve el nombre del blob o archivo a la que apunta el conjunto de datos. Si tiene definiciones de folderPath que usan variables como {Year}, {Month}, {Day}, etc., el método devuelve la cadena tal cual, sin sustituir dichas variables por los valores de tiempo de ejecución. Consulte la sección [Acceso a las propiedades extendidas](#access-extended-properties) para más información sobre cómo acceder a SliceStart, SliceEnd, etc.

    ```JSON
    "name": "InputDataset",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties": {
            "fileName": "file.txt",
            "folderPath": "adftutorial/inputfolder/",
    ```

    El método Calculate calcula el número de instancias de la palabra clave Microsoft en los archivos de entrada (los blobs de la carpeta). El término de búsqueda ("Microsoft") está codificado de forma rígida en el código.
10. Compile el proyecto. Haga clic en **Compilar** en el menú y haga clic en **Compilar solución**.

    > [!IMPORTANT]
    > Establezca la versión 4.5.2 de .NET Framework como la plataforma de destino para el proyecto: haga clic con el botón derecho en el proyecto y en **Propiedades** para establecer el marco de destino. Data Factory no admite actividades personalizadas que se compilaron con versiones de .NET Framework posteriores a la 4.5.2.

11. Inicie el **Explorador de Windows** y vaya a la carpeta **bin\debug** o **bin\release**, según el tipo de compilación.
12. Cree un archivo ZIP **MyDotNetActivity.zip** que contenga todos los archivos binarios en la carpeta \<project folder\>\bin\Debug. Incluya el archivo **MyDotNetActivity.pdb** para ver detalles adicionales, como el número de línea en el código fuente que causó el problema, si hubo un error.

    > [!IMPORTANT]
    > Todos los archivos incluidos en el archivo ZIP de la actividad personalizada deben estar en el **nivel superior** ; no debe haber subcarpetas.

    ![Archivos de salida binarios](./media/data-factory-use-custom-activities/Binaries.png)
14. Cree el contenedor de blobs **customactivitycontainer**, si aún no existe.
15. Cargue MyDotNetActivity.zip como un blob para customactivitycontainer en una instancia de Azure Blob Storage **de uso general** (Blob Storage de acceso esporádico/no frecuente) al que AzureStorageLinkedService hace referencia.

> [!IMPORTANT]
> Si agrega este proyecto de actividad de .NET a una solución en Visual Studio que contenga un proyecto de Data Factory, y agrega una referencia al proyecto de actividad .NET del proyecto de la aplicación de Data Factory, no tendrá que realizar los últimos dos pasos para crear manualmente el archivo ZIP y cargarlo a la instancia de Azure Blob Storage de uso general. Al publicar las entidades de la factoría de datos con Visual Studio el proceso de publicación realizar automáticamente estos pasos. Para obtener más información, consulte la sección [Proyecto de Data Factory en Visual Studio](#data-factory-project-in-visual-studio).

## <a name="create-a-pipeline-with-custom-activity"></a>Crear una canalización con una actividad personalizada
Ha creado una actividad personalizada y cargado el archivo zip con datos binarios en un contenedor de blobs en una cuenta de Azure Storage **de uso general**. En esta sección, creará una instancia de Azure Data Factory con una canalización que usa la actividad personalizada.

El conjunto de datos de entrada de la actividad personalizada representa los blobs (archivos) de la carpeta customactivityinput del contenedor adftutorial en Blob Storage. El conjunto de datos de salida de la actividad representa los blobs de salida de la carpeta customactivityinput del contenedor adftutorial en Blob Storage.

Cree el archivo **file.txt** con el siguiente contenido y cárguelo en la carpeta **customactivityinput** del contenedor **adftutorial**. Cree el contenedor adftutorial si aún no existe.

```
test custom activity Microsoft test custom activity Microsoft
```

La carpeta de entrada corresponde a un segmento de Azure Data Factory, aunque la carpeta contenga dos, o más, archivos. Cuando la canalización procesa cada segmento, la actividad personalizada procesa una iteración en todos los blobs de la carpeta de entrada de dicho segmento.

Verá un archivo de salida en la carpeta adftutorial\customactivityoutput con una o varias líneas (tantas como blobs haya en la carpeta de entrada):

```
2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2016-11-16-00/file.txt.
```


Estos son los pasos que se realizan en esta sección:

1. Crear una **factoría de datos**.
2. Crear **servicios vinculados** del grupo de máquinas virtuales de Azure Batch en el que se ejecuta la actividad personalizada y la instancia de Azure Storage que contiene los blobs de entrada o salida.
3. Crear **conjuntos de datos** de entrada y salida que representan la entrada y salida de la actividad personalizada.
4. Crear una **canalización** que usa la actividad personalizada.

> [!NOTE]
> Cree el archivo **file.txt** y cárguelo en un contenedor de blobs, si no lo ha hecho aún. Consulte las instrucciones de la sección anterior.

### <a name="step-1-create-the-data-factory"></a>Paso 1: Creación de la factoría de datos
1. Tras iniciar sesión en Azure Portal, siga estos pasos:
   1. En el menú izquierdo, haga clic en **Crear un recurso**.
   2. Haga clic en **Datos y análisis** en la hoja **Nuevo**.
   3. Haga clic en **Factoría de datos** en la hoja **Análisis de datos**.

      ![Nuevo menú de Azure Data Factory](media/data-factory-use-custom-activities/new-azure-data-factory-menu.png)
2. En la hoja **Nueva factoría de datos**, escriba **CustomActivityFactory** en el campo Nombre. El nombre de la instancia de Azure Data Factory debe ser único de forma global. Si recibe el error: **El nombre "CustomActivityFactory" de factoría de datos no está disponible**, cambie el nombre de la factoría de datos (por ejemplo, **suNombreCustomActivityFactory**) e intente crearla de nuevo.

    ![Nueva hoja de Azure Data Factory](media/data-factory-use-custom-activities/new-azure-data-factory-blade.png)
3. Haga clic en **NOMBRE DEL GRUPO DE RECURSOS**y seleccione un grupo de recursos existente o cree uno.
4. Compruebe que usa la **suscripción** y la **región** correctas en las que desea que se cree la factoría de datos.
5. Haga clic en **Crear** en la hoja **Nueva fábrica de datos**.
6. Verá la factoría de datos que se crea en el **Panel** del Portal de Azure.
7. Una vez que la factoría de datos se ha creado correctamente, verá la hoja Data Factory, donde se muestra su contenido.

    ![Hoja de la Factoría de datos](media/data-factory-use-custom-activities/data-factory-blade.png)

### <a name="step-2-create-linked-services"></a>Paso 2: Crear servicios vinculados
Los servicios vinculados vinculan almacenes de datos o servicios de proceso con una factoría de datos de Azure. En este paso, vinculará su cuenta de Azure Storage y la cuenta de Azure Batch con su factoría de datos.

#### <a name="create-azure-storage-linked-service"></a>Creación de un servicio vinculado de Azure Storage
1. Haga clic en el icono **Crear e implementar** de la hoja **FACTORÍA DE DATOS** de **CustomActivityFactory**. Aparecerá Data Factory Editor.
2. Haga clic en **Nuevo almacén de datos** en la barra de comandos y elija **Almacenamiento de Azure**. Debería ver el script JSON para crear un servicio vinculado de Azure Storage en el editor.

    ![Nuevo almacén de datos: Azure Storage](media/data-factory-use-custom-activities/new-data-store-menu.png)
3. Reemplace `<accountname>` por el nombre de la cuenta de Azure Storage y `<accountkey>` por la clave de acceso de la cuenta de Azure Storage. Para aprender a obtener la clave de acceso de almacenamiento, consulte [Administración de claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md).

    ![Servicio vinculado de Azure Storage](media/data-factory-use-custom-activities/azure-storage-linked-service.png)
4. Haga clic en **Implementar** en la barra de comandos para implementar el servicio vinculado.

#### <a name="create-azure-batch-linked-service"></a>Creación del servicio vinculado de Azure Batch
1. En Data Factory Editor, haga clic en **... Más** en la barra de comandos, haga clic en **Nuevo proceso** y luego seleccione **Azure Batch** en el menú.

    ![Nuevo proceso: Azure Batch](media/data-factory-use-custom-activities/new-azure-compute-batch.png)
2. Realice los siguientes cambios en el script JSON:

   1. Especifique el nombre de cuenta de Azure Batch en la propiedad **accountName** . La **dirección URL** de la **hoja de la cuenta de Azure Batch** tiene el formato siguiente: `http://accountname.region.batch.azure.com`. Para la propiedad **batchUri** en JSON, tiene que quitar `accountname.` de la dirección URL y usar `accountname` para la propiedad JSON `accountName`.
   2. Especifique la clave de cuenta de Azure Batch en la propiedad **accessKey** .
   3. Especifique el nombre del grupo que creó como parte de los requisitos previos en la propiedad **poolName**. También puede especificar el id. del grupo, en lugar del nombre.
   4. Especifique el URI de Azure Batch en la propiedad **batchUri** . Ejemplo: `https://westus.batch.azure.com`.
   5. Especifique el servicio **AzureStorageLinkedService** for the **linkedServiceName** .

        ```json
        {
          "name": "AzureBatchLinkedService",
          "properties": {
            "type": "AzureBatch",
            "typeProperties": {
              "accountName": "myazurebatchaccount",
              "batchUri": "https://westus.batch.azure.com",
              "accessKey": "<yourbatchaccountkey>",
              "poolName": "myazurebatchpool",
              "linkedServiceName": "AzureStorageLinkedService"
            }
          }
        }
        ```

       En la propiedad **poolName** , también puede especificar el identificador del grupo, en lugar del nombre del grupo.

### <a name="step-3-create-datasets"></a>Paso 3: Creación de conjuntos de datos
En este paso, crea conjuntos de datos que representan los datos de entrada y salida.

#### <a name="create-input-dataset"></a>Creación de un conjunto de datos de entrada
1. En el **Editor** en la instancia de Data Factory, haga clic en **... Más** en la barra de comandos y luego en **Nuevo conjunto de datos** y después, seleccione **Azure Blob Storage** desde el menú desplegable.
2. Reemplace el script JSON del panel derecho por el siguiente fragmento de código JSON:

    ```json
    {
        "name": "InputDataset",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "folderPath": "adftutorial/customactivityinput/",
                "format": {
                    "type": "TextFormat"
                }
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

   Más adelante en este tutorial se creará una canalización con la hora de inicio: 2016-11-16T00:00:00Z y hora de finalización: 2016-11-16T05:00:00Z. Está programada para generar datos cada hora, por lo que hay cinco segmentos de entrada o salida (entre **00**:00:00 -> **05**:00:00).

   Los valores de **frequency** e **interval** del conjunto de datos de entrada se establecen en **Hour** y **1** respectivamente, lo que significa que el segmento de entrada está disponible cada hora. En este ejemplo, es el mismo archivo (file.txt) de la carpeta de entrada.

   Estas son las horas de inicio de cada segmento, que se representa mediante la variable de sistema SliceStart en el fragmento de código JSON anterior.
3. Haga clic en **Implementar** en la barra de herramientas para crear e implementar **InputDataset**. Confirme que aparece el mensaje **TABLA CREADA CORRECTAMENTE** en la barra de título del Editor.

#### <a name="create-an-output-dataset"></a>Crear un conjunto de datos de salida
1. En el **Data Factory Editor**, haga clic en **... Más** en la barra de comandos, haga clic en **Nuevo conjunto de datos** y después seleccione **Azure Blob Storage**.
2. Reemplace el script JSON en el panel derecho con el siguiente script JSON:

    ```JSON
    {
        "name": "OutputDataset",
        "properties": {
            "type": "AzureBlob",
            "linkedServiceName": "AzureStorageLinkedService",
            "typeProperties": {
                "fileName": "{slice}.txt",
                "folderPath": "adftutorial/customactivityoutput/",
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

     La ubicación de salida es **adftutorial/customactivityoutput/** y el nombre del archivo de salida, yyyy-MM-dd-HH.txt, donde yyyy-MM-dd-HH es el año, el mes, el día y la hora del segmento que se está generando. Consulte la [Referencia para desarrolladores][adf-developer-reference] para obtener más información.

    Se genera un blob o archivo de salida para cada segmento de entrada. Así es cómo se asigna el nombre al archivo de salida de cada segmento. Todos los archivos de salida se generan en una carpeta de salida: **adftutorial\customactivityoutput**.

   | Segmento | Hora de inicio | Archivo de salida |
   |:--- |:--- |:--- |
   | 1 |2016-11-16T00:00:00 |2016-11-16-00.txt |
   | 2 |2016-11-16T01:00:00 |2016-11-16-01.txt |
   | 3 |2016-11-16T02:00:00 |2016-11-16-02.txt |
   | 4 |2016-11-16T03:00:00 |2016-11-16-03.txt |
   | 5 |2016-11-16T04:00:00 |2016-11-16-04.txt |

    Recuerde que todos los archivos de una carpeta de entrada forman parte de un segmento con las horas de inicio mencionadas anteriormente. Cuando se procesa este segmento, la actividad personalizada examinan todos los archivos y produce una línea en el archivo de salida con el número de apariciones del término de búsqueda ("Microsoft"). Si hay tres archivos en la carpeta de entrada, hay tres líneas en el archivo de salida para cada segmento horario: 2016-11-16-00.txt, 2016-11-16:01:00:00.txt, etc.
3. Para implementar **OutputDataset**, haga clic en **Implementar** en la barra de comandos.

### <a name="create-and-run-a-pipeline-that-uses-the-custom-activity"></a>Creación y ejecución de una canalización que usa la actividad personalizada
1. En Data Factory Editor, haga clic en **... Más** y luego seleccione **Nueva canalización** en la barra de comandos.
2. Reemplace el script JSON del panel derecho con el siguiente script JSON:

    ```JSON
    {
      "name": "ADFTutorialPipelineCustom",
      "properties": {
        "description": "Use custom activity",
        "activities": [
          {
            "Name": "MyDotNetActivity",
            "Type": "DotNetActivity",
            "Inputs": [
              {
                "Name": "InputDataset"
              }
            ],
            "Outputs": [
              {
                "Name": "OutputDataset"
              }
            ],
            "LinkedServiceName": "AzureBatchLinkedService",
            "typeProperties": {
              "AssemblyName": "MyDotNetActivity.dll",
              "EntryPoint": "MyDotNetActivityNS.MyDotNetActivity",
              "PackageLinkedService": "AzureStorageLinkedService",
              "PackageFile": "customactivitycontainer/MyDotNetActivity.zip",
              "extendedProperties": {
                "SliceStart": "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))"
              }
            },
            "Policy": {
              "Concurrency": 2,
              "ExecutionPriorityOrder": "OldestFirst",
              "Retry": 3,
              "Timeout": "00:30:00",
              "Delay": "00:00:00"
            }
          }
        ],
        "start": "2016-11-16T00:00:00Z",
        "end": "2016-11-16T05:00:00Z",
        "isPaused": false
      }
    }
    ```

    Tenga en cuenta los siguientes puntos:

   * **Concurrency** se establece en **2** para que dos máquinas virtuales del grupo de Azure Batch procesen dos segmentos en paralelo.
   * Hay una actividad en la sección de actividades y es de tipo: **DotNetActivity**.
   * **AssemblyName** se establece en el nombre del archivo DLL: **MyDotnetActivity.dll**.
   * **EntryPoint** se establece **MyDotNetActivityNS.MyDotNetActivity**.
   * **PackageLinkedService** se establece en **AzureStorageLinkedService**, que apunta al almacenamiento de blobs que contiene el archivo ZIP de la actividad personalizada. Si usa diferentes cuentas de Azure Storage para los archivos de entrada y salida y el archivo ZIP de actividad personalizada, tiene que crear otro servicio vinculado a Azure Storage. En este artículo, se da por supuesto que usa la misma cuenta de Azure Storage.
   * **PackageFile** se establece en **customactivitycontainer/MyDotNetActivity.zip**. Está en el formato <contenedorDelZIP>/<nombreDelZIP.zip>.
   * La actividad personalizada toma **InputDataset** como entrada y **OutputDataset** como salida.
   * La propiedad linkedServiceName de la actividad personalizada apunta a **AzureBatchLinkedService**, que indica a Azure Data Factory que la actividad personalizada debe ejecutarse en Lote de Azure.
   * **isPaused** se establece en **false** de forma predeterminada. La canalización se ejecuta inmediatamente en este ejemplo, ya que los segmentos se inician en el pasado. Esta propiedad se puede establecer en true para pausar la canalización y se puede volver a establecer en false para reiniciarla.
   * Hay una diferencia de **cinco** horas entre la hora de **inicio** y la de **finalización**, y los segmentos se producen cada hora, por lo que la canalización produce cinco segmentos.
3. Haga clic en **Implementar** en la barra de comandos para implementar la canalización.

### <a name="monitor-the-pipeline"></a>Supervisar la canalización
1. En la hoja Data Factory del Portal de Azure, haga clic en **Diagrama**.

    ![Icono Diagrama](./media/data-factory-use-custom-activities/DataFactoryBlade.png)
2. En la vista Diagrama, haga clic en OutputDataset.

    ![Vista Diagrama](./media/data-factory-use-custom-activities/diagram.png)
3. Debería ver que los cinco segmentos están en estado Listo. Si no están en estado Listo, aún no se han producido.

   ![Segmentos de salida](./media/data-factory-use-custom-activities/OutputSlices.png)
4. Compruebe que los archivos de salida se generan en el almacenamiento de blobs del contenedor **adftutorial** .

   ![salida de la actividad personalizada][image-data-factory-output-from-custom-activity]
5. Si abre el archivo de salida, debería ver un resultado similar al siguiente:

    ```
    2 occurrences(s) of the search term "Microsoft" were found in the file inputfolder/2016-11-16-00/file.txt.
    ```
6. Use [Azure Portal][azure-preview-portal] o los cmdlets de Azure PowerShell para supervisar la factoría de datos, las canalizaciones y los conjuntos de datos. Puede ver mensajes desde **ActivityLogger** en el código de la actividad personalizada en los registros (de forma específica user-0.log) que puede descargar desde el portal o con cmdlets.

   ![registros de descarga de la actividad personalizada][image-data-factory-download-logs-from-custom-activity]

Consulte [Supervisión y administración de canalizaciones de la Factoría de datos de Azure](data-factory-monitor-manage-pipelines.md) para obtener información detallada sobre los pasos que hay que seguir para supervisar los conjuntos de datos y las canalizaciones.

## <a name="data-factory-project-in-visual-studio"></a>Proyecto de Data Factory en Visual Studio
Puede crear y publicar entidades de Data Factory usando Visual Studio en lugar de Azure Portal. Para obtener información sobre la creación y publicación de entidades de Data Factory mediante Visual Studio, consulte los artículos [Build your first pipeline using Visual Studio (Compilar la primera canalización mediante Visual Studio)](data-factory-build-your-first-pipeline-using-vs.md) y [Copy data from Azure Blob to Azure SQL (Copiar datos de Azure Blob a Azure SQL)](data-factory-copy-activity-tutorial-using-visual-studio.md).

Si crea el proyecto de Data Factory en Visual Studio, realice los siguientes pasos adicionales:

1. Agregue el proyecto de Data Factory a la solución de Visual Studio que contiene el proyecto de la actividad personalizada.
2. Agregue una referencia al proyecto de la actividad de .NET desde el proyecto de Data Factory. Haga clic con el botón derecho en el proyecto de Data Factory, seleccione **Agregar** y, a continuación, haga clic en **Referencia**.
3. En el cuadro de diálogo **Agregar referencia**, seleccione el proyecto **MyDotNetActivity** y, después, haga clic en **Aceptar**.
4. Compilar y publicar la solución.

    > [!IMPORTANT]
    > Cuando publica las entidades de Data Factory, se crea un archivo ZIP automáticamente y se carga en el contenedor de blobs: customactivitycontainer. Si no existe el contenedor de blobs, también se crea automáticamente.

## <a name="data-factory-and-batch-integration"></a>Integración de Data Factory y Batch
El servicio Data Factory crea un trabajo en Azure Batch con el nombre **adf-poolname: job-xxx**. Haga clic en **Trabajos** en el menú izquierdo.

![Azure Data Factory: trabajos por lotes](media/data-factory-use-custom-activities/data-factory-batch-jobs.png)

Se crea una tarea para cada ejecución de actividad de un segmento. Si hay cinco segmentos listos para ser procesados, se crean cinco tareas de este trabajo. Si hay varios nodos de proceso en el grupo de Batch, se pueden ejecutar dos o más segmentos en paralelo. También puede haber más de un segmento en ejecución en el mismo proceso si el número máximo de tareas se establece un valor mayor que 1.

![Azure Data Factory: tareas de trabajos por lotes](media/data-factory-use-custom-activities/data-factory-batch-job-tasks.png)

En el diagrama siguiente se ilustra la relación entre las tareas de Azure Data Factory y Batch.

![Data Factory y Batch](./media/data-factory-use-custom-activities/DataFactoryAndBatch.png)

## <a name="troubleshoot-failures"></a>Solución de errores
La solución de problemas se compone de varias técnicas básicas:

1. Si ve el siguiente error, puede que esté usando una instancia de Blob Storage activa o de acceso esporádico en lugar de usar una de uso general. Cargue el archivo zip en una **cuenta de Azure Storage de uso general**.

    ```
    Error in Activity: Job encountered scheduling error. Code: BlobDownloadMiscError Category: ServerError Message: Miscellaneous error encountered while downloading one of the specified Azure Blob(s).
    ```
2. Si aparece el siguiente error, confirme que el nombre de la clase en el archivo CS coincide con el nombre especificado para la propiedad **EntryPoint** en el JSON de la canalización. En el tutorial, el nombre de la clase es: MyDotNetActivity y EntryPoint en el archivo JSON es: MyDotNetActivityNS.**MyDotNetActivity**.

    ```
    MyDotNetActivity assembly does not exist or doesn't implement the type Microsoft.DataFactories.Runtime.IDotNetActivity properly
    ```

   Si los nombres coinciden, confirme que todos los archivos binarios se encuentran en la **carpeta raíz** del archivo zip. Es decir, cuando abra el archivo zip verá todos los archivos de la carpeta raíz, pero no los que están en las subcarpetas.
3. Si el segmento de entrada no está establecido en **Listo**, confirme que la estructura de la carpeta de entrada es correcta y que **file.txt** se encuentra en las carpetas de entrada.
3. En el método **Execute** de la actividad personalizada, use el objeto **IActivityLogger** para registrar información que ayude a solucionar problemas. Los mensajes registrados se muestran en los archivos de registro del usuario (uno o varios archivos llamados user-0.log, user-1.log, user-2.log, etc.).

   En la hoja **OutputDataset**, haga clic en el segmento para ver la hoja **SEGMENTO DE DATOS** de dicho segmento. Verá **ejecuciones de actividad** para ese segmento. Debería ver una ejecución de actividad del segmento. Si hace clic en Ejecutar en la barra de comandos, podrá iniciar otra ejecución de actividad en el mismo segmento.

   Al hacer clic en la ejecución de actividad, verá la hoja **DETALLES DE LA EJECUCIÓN DE ACTIVIDAD** con una lista de archivos de registro. Los mensajes registrados se encuentran en el archivo user_0.log. Si se produce un error, verá tres ejecuciones de actividad, ya que el número de reintentos está establecido en 3 en la canalización o actividad JSON. Al hacer clic en la ejecución de actividad, verá los archivos de registro que puede revisar para solucionar el error.

   En la lista de archivos de registro, haga clic en **user-0.log**. En el panel derecho, se encuentran los resultados del uso del método **IActivityLogger.Write** . Si no ve todos los mensajes, compruebe si tiene más archivos de registro llamados user_1.log, user_2.log, etc. De lo contrario, es posible que el código haya generado algún error tras el último mensaje registrado.

   Además, consulte **system-0.log** para ver si hay alguna excepción o mensaje de error del sistema.
4. Incluya el archivo **PDB** en el archivo ZIP para que los detalles sobre un error contengan información como la **pila de llamadas** cuando se produce un error.
5. Todos los archivos incluidos en el archivo ZIP de la actividad personalizada deben estar en el **nivel superior** ; no debe haber subcarpetas.
6. Asegúrese de que en **assemblyName** (MyDotNetActivity.dll), **entryPoint** (MyDotNetActivityNS.MyDotNetActivity), **packageFile** (customactivitycontainer/MyDotNetActivity.zip) y **packageLinkedService** (debe apuntar a la instancia de Azure Blob Storage **de uso general** que contiene el archivo ZIP) se han seleccionado los valores correctos.
7. Si corrigió algún error y desea volver a procesar el segmento, haga clic con el botón derecho en el segmento, en la hoja **OutputDataset**, y haga clic en **Run**.
8. Si ve el error siguiente, significa que está usando la versión 4.3.0 o posterior del paquete de Azure Storage. El iniciador del servicio Data Factory requiere la versión 4.3 de WindowsAzure.Storage. Consulte la sección [Aislamiento de AppDomain](#appdomain-isolation) para ver una solución alternativa si tiene que utilizar la versión más reciente del ensamblado de Azure Storage.

    ```
    Error in Activity: Unknown error in module: System.Reflection.TargetInvocationException: Exception has been thrown by the target of an invocation. ---> System.TypeLoadException: Could not load type 'Microsoft.WindowsAzure.Storage.Blob.CloudBlob' from assembly 'Microsoft.WindowsAzure.Storage, Version=4.3.0.0, Culture=neutral,
    ```

    Si puede usar la versión 4.3.0 del paquete de Azure Storage, quite la referencia al paquete de Azure.Storage de la versión 4.3.0 o posterior. A continuación ejecute el siguiente comando de la Consola del administrador de paquetes de NuGet.

    ```powershell
    Install-Package WindowsAzure.Storage -Version 4.3.0
    ```

    Compile el proyecto. Elimine la carpeta bin\Debug del ensamblado Azure.Storage de la versión 4.3.0 o posterior. Cree un archivo .zip con los datos binarios y el archivo PDB. Reemplace el antiguo archivo .zip por este otro en el contenedor de blobs (customactivitycontainer). Vuelva a ejecutar los segmentos con errores (haga clic con el botón derecho en el segmento y, después, haga clic en Ejecutar).
8. La actividad personalizada no utiliza el archivo **app.config** desde el paquete. Por lo tanto, si el código lee cualquier cadena de conexión del archivo de configuración, no funcionará en tiempo de ejecución. El procedimiento recomendado al usar Azure Batch consiste en conservar los secretos en una instancia de **Azure Key Vault**, usar una entidad de servicio basada en certificados para proteger el **almacén de claves** y distribuir el certificado al grupo de Azure Batch. Tras ello, la actividad personalizada de .NET podrá acceder a los secretos desde el almacén de claves en tiempo de ejecución. Esta es una solución genérica y se puede extrapolar a cualquier tipo de secreto, no solo a cadenas de conexión.

   Existe una solución más sencilla, pero no es un procedimiento recomendado: puede crear un **servicio vinculado de SQL Azure** con configuración de cadena de conexión, crear un conjunto de datos que utilice el servicio vinculado y vincular el conjunto de datos (configurado con carácter de entrada ficticio) con la actividad de .NET personalizada. Tras ello, podrá acceder a la cadena de conexión del servicio vinculado del código de la actividad personalizada.

## <a name="update-custom-activity"></a>Actualización de una actividad personalizada
Si actualiza el código de la actividad personalizada, compílelo y cargue el archivo comprimido que contiene los nuevos binarios en el almacenamiento de blobs.

## <a name="appdomain-isolation"></a>Aislamiento de AppDomain
Consulte el [ejemplo Cross AppDomain](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/CrossAppDomainDotNetActivitySample), que muestra cómo crear una actividad personalizada que no esté restringida a las versiones de ensamblado que usa el iniciador de Data Factory (por ejemplo: WindowsAzure.Storage v4.3.0, Newtonsoft.Json v6.0.x, etc.).

## <a name="access-extended-properties"></a>Acceso a las propiedades extendidas
Puede declarar propiedades extendidas en el JSON de actividad como se muestra en el siguiente ejemplo:

```JSON
"typeProperties": {
  "AssemblyName": "MyDotNetActivity.dll",
  "EntryPoint": "MyDotNetActivityNS.MyDotNetActivity",
  "PackageLinkedService": "AzureStorageLinkedService",
  "PackageFile": "customactivitycontainer/MyDotNetActivity.zip",
  "extendedProperties": {
    "SliceStart": "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))",
    "DataFactoryName": "CustomActivityFactory"
  }
},
```

En el ejemplo, hay dos propiedades extendidas: **SliceStart** y **DataFactoryName**. El valor de SliceStart se basa en la variable del sistema SliceStart. Consulte el artículo sobre las [variables del sistema](data-factory-functions-variables.md) para ver una lista de las variables del sistema admitidas. El valor de DataFactoryName está incrustado directamente en CustomActivityFactory.

Para acceder a estas propiedades extendidas en el método **Execute** , use código similar al siguiente:

```csharp
// to get extended properties (for example: SliceStart)
DotNetActivity dotNetActivity = (DotNetActivity)activity.TypeProperties;
string sliceStartString = dotNetActivity.ExtendedProperties["SliceStart"];

// to log all extended properties
IDictionary<string, string> extendedProperties = dotNetActivity.ExtendedProperties;
logger.Write("Logging extended properties if any...");
foreach (KeyValuePair<string, string> entry in extendedProperties)
{
    logger.Write("<key:{0}> <value:{1}>", entry.Key, entry.Value);
}
```

## <a name="auto-scaling-of-azure-batch"></a>Escalado automático de Azure Batch
También puede crear un grupo de Azure Batch con la característica **autoescala** . Por ejemplo, podría crear un grupo de Azure Batch con 0 VM dedicadas y una fórmula de escalado automático basada en el número de tareas pendientes.

La fórmula del ejemplo obtiene el comportamiento siguiente: Cuando el grupo se crea inicialmente, se inicia con una máquina virtual. La métrica $PendingTasks define el número de tareas que están en ejecución o activas (en cola).  La fórmula busca el número promedio de tareas pendientes en los últimos 180 segundos y establece TargetDedicated en consecuencia. Garantiza que TargetDedicated nunca supera las 25 VM. Por tanto, a medida que se envían nuevas tareas, el grupo crece automáticamente y a medida que estás se completan, las VM se liberan una a una y el escalado automático las reduce. startingNumberOfVMs y maxNumberofVMs se pueden adaptar a sus necesidades.

Fórmula de escalado automático:

```
startingNumberOfVMs = 1;
maxNumberofVMs = 25;
pendingTaskSamplePercent = $PendingTasks.GetSamplePercent(180 * TimeInterval_Second);
pendingTaskSamples = pendingTaskSamplePercent < 70 ? startingNumberOfVMs : avg($PendingTasks.GetSample(180 * TimeInterval_Second));
$TargetDedicated=min(maxNumberofVMs,pendingTaskSamples);
```

Para más información, consulte [Escalado automático de los nodos de proceso en un grupo de Azure Batch](../../batch/batch-automatic-scaling.md) .

Si el grupo usa el valor predeterminado de la propiedad [autoScaleEvaluationInterval](https://msdn.microsoft.com/library/azure/dn820173.aspx), el servicio Batch puede tardar de 15 a 30 minutos en preparar la máquina virtual antes de ejecutar la actividad personalizada.  Si el grupo usa otro valor de autoScaleEvaluationInterval diferente, el servicio Batch podría tardar el valor de autoScaleEvaluationInterval más 10 minutos.


## <a name="create-a-custom-activity-by-using-net-sdk"></a>Creación de una actividad personalizada mediante el SDK de .NET
En el tutorial de este artículo, se crea una factoría de datos con una canalización que usa la actividad personalizada mediante Azure Portal. El código siguiente muestra cómo crear la factoría de datos con el SDK de .NET en su lugar. Puede encontrar más detalles sobre cómo usar el SDK para crear mediante programación las canalizaciones en el artículo [crear una canalización con la actividad de copia mediante la API de NET](data-factory-copy-activity-tutorial-using-dotnet-api.md).

```csharp
using System;
using System.Configuration;
using System.Collections.ObjectModel;
using System.Threading;
using System.Threading.Tasks;

using Microsoft.Azure;
using Microsoft.Azure.Management.DataFactories;
using Microsoft.Azure.Management.DataFactories.Models;
using Microsoft.Azure.Management.DataFactories.Common.Models;

using Microsoft.IdentityModel.Clients.ActiveDirectory;
using System.Collections.Generic;

namespace DataFactoryAPITestApp
{
    class Program
    {
        static void Main(string[] args)
        {
            // create data factory management client

            // TODO: replace ADFTutorialResourceGroup with the name of your resource group.
            string resourceGroupName = "ADFTutorialResourceGroup";

            // TODO: replace APITutorialFactory with a name that is globally unique. For example: APITutorialFactory04212017
            string dataFactoryName = "APITutorialFactory";

            TokenCloudCredentials aadTokenCredentials = new TokenCloudCredentials(
                ConfigurationManager.AppSettings["SubscriptionId"],
                GetAuthorizationHeader().Result);

            Uri resourceManagerUri = new Uri(ConfigurationManager.AppSettings["ResourceManagerEndpoint"]);

            DataFactoryManagementClient client = new DataFactoryManagementClient(aadTokenCredentials, resourceManagerUri);

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
                            // TODO: Replace <accountname> and <accountkey> with name and key of your Azure Storage account.
                            new AzureStorageLinkedService("DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>")
                        )
                    }
                }
            );

            // create a linked service for output data store: Azure SQL Database
            Console.WriteLine("Creating Azure Batch linked service");
            client.LinkedServices.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new LinkedServiceCreateOrUpdateParameters()
                {
                    LinkedService = new LinkedService()
                    {
                        Name = "AzureBatchLinkedService",
                        Properties = new LinkedServiceProperties
                        (
                            // TODO: replace <batchaccountname> and <yourbatchaccountkey> with name and key of your Azure Batch account
                            new AzureBatchLinkedService("<batchaccountname>", "https://westus.batch.azure.com", "<yourbatchaccountkey>", "myazurebatchpool", "AzureStorageLinkedService")
                        )
                    }
                }
            );

            // create input and output datasets
            Console.WriteLine("Creating input and output datasets");
            string Dataset_Source = "InputDataset";
            string Dataset_Destination = "OutputDataset";

            Console.WriteLine("Creating input dataset of type: Azure Blob");
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
                                FolderPath = "adftutorial/customactivityinput/",
                                Format = new TextFormat()
                            },
                            External = true,
                            Availability = new Availability()
                            {
                                Frequency = SchedulePeriod.Hour,
                                Interval = 1,
                            },

                            Policy = new Policy() { }
                        }
                    }
                });

            Console.WriteLine("Creating output dataset of type: Azure Blob");
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
                                FileName = "{slice}.txt",
                                FolderPath = "adftutorial/customactivityoutput/",
                                PartitionedBy = new List<Partition>()
                                {
                                    new Partition()
                                    {
                                        Name = "slice",
                                        Value = new DateTimePartitionValue()
                                        {
                                            Date = "SliceStart",
                                            Format = "yyyy-MM-dd-HH"
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

            Console.WriteLine("Creating a custom activity pipeline");
            DateTime PipelineActivePeriodStartTime = new DateTime(2017, 3, 9, 0, 0, 0, 0, DateTimeKind.Utc);
            DateTime PipelineActivePeriodEndTime = PipelineActivePeriodStartTime.AddMinutes(60);
            string PipelineName = "ADFTutorialPipelineCustom";

            client.Pipelines.CreateOrUpdate(resourceGroupName, dataFactoryName,
                new PipelineCreateOrUpdateParameters()
                {
                    Pipeline = new Pipeline()
                    {
                        Name = PipelineName,
                        Properties = new PipelineProperties()
                        {
                            Description = "Use custom activity",

                            // Initial value for pipeline's active period. With this, you won't need to set slice status
                            Start = PipelineActivePeriodStartTime,
                            End = PipelineActivePeriodEndTime,
                            IsPaused = false,

                            Activities = new List<Activity>()
                            {
                                new Activity()
                                {
                                    Name = "MyDotNetActivity",
                                    Inputs = new List<ActivityInput>()
                                    {
                                        new ActivityInput() {
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
                                    LinkedServiceName = "AzureBatchLinkedService",
                                    TypeProperties = new DotNetActivity()
                                    {
                                        AssemblyName = "MyDotNetActivity.dll",
                                        EntryPoint = "MyDotNetActivityNS.MyDotNetActivity",
                                        PackageLinkedService = "AzureStorageLinkedService",
                                        PackageFile = "customactivitycontainer/MyDotNetActivity.zip",
                                        ExtendedProperties = new Dictionary<string, string>()
                                        {
                                            { "SliceStart", "$$Text.Format('{0:yyyyMMddHH-mm}', Time.AddMinutes(SliceStart, 0))"}
                                        }
                                    },
                                    Policy = new ActivityPolicy()
                                    {
                                        Concurrency = 2,
                                        ExecutionPriorityOrder = "OldestFirst",
                                        Retry = 3,
                                        Timeout = new TimeSpan(0,0,30,0),
                                        Delay = new TimeSpan()
                                    }
                                }
                            }
                        }
                    }
                });
        }

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
    }
}
```

## <a name="debug-custom-activity-in-visual-studio"></a>Depurar una actividad personalizada en Visual Studio
El ejemplo de [Azure Data Factory: entorno local](https://github.com/gbrueckl/Azure.DataFactory.LocalEnvironment) en GitHub incluye una herramienta que permite depurar actividades personalizadas de .NET en Visual Studio.

## <a name="sample-custom-activities-on-github"></a>Actividades personalizadas de ejemplo en GitHub
| Muestra | Qué hace la actividad personalizada |
| --- | --- |
| [Descargador de datos HTTP](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/HttpDataDownloaderSample). |Descarga datos de un punto de conexión HTTP a Azure Blob Storage a través de una actividad personalizada de C# en Data Factory. |
| [Ejemplo de Análisis de opiniones de Twitter.](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/TwitterAnalysisSample-CustomC%23Activity) |Invoca un modelo de Azure Machine Learning Studio y realiza un análisis de opiniones, puntuación, predicción, etc. |
| [Ejecutar script R](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample). |Invoca el script de R; para ello, ejecuta RScript.exe en el clúster de HDInsight que ya tiene instalado R. |
| [Actividad .NET entre AppDomain](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/CrossAppDomainDotNetActivitySample) |Utiliza otras versiones de ensamblado que las utilizadas por el iniciador de Data Factory. |
| [Reproceso de un modelo en Azure Analysis Services](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/AzureAnalysisServicesProcessSample) |  Reprocesa un modelo en Azure Analysis Services. |

[batch-net-library]: ../../batch/batch-dotnet-get-started.md
[batch-create-account]: ../../batch/batch-account-create-portal.md
[batch-technical-overview]:../../azure-sql/database/sql-database-paas-overview.md
[batch-get-started]: ../../batch/batch-dotnet-get-started.md
[use-custom-activities]: data-factory-use-custom-activities.md
[troubleshoot]: data-factory-troubleshoot.md
[data-factory-introduction]: data-factory-introduction.md
[azure-powershell-install]: https://github.com/Azure/azure-sdk-tools/releases


[developer-reference]: https://go.microsoft.com/fwlink/?LinkId=516908
[cmdlet-reference]: https://go.microsoft.com/fwlink/?LinkId=517456

[new-azure-batch-account]: https://msdn.microsoft.com/library/mt125880.aspx
[new-azure-batch-pool]: https://msdn.microsoft.com/library/mt125936.aspx
[azure-batch-blog]: https://blogs.technet.com/b/windowshpc/archive/2014/10/28/using-azure-powershell-to-manage-azure-batch-account.aspx

[nuget-package]: https://go.microsoft.com/fwlink/?LinkId=517478
[adf-developer-reference]: https://go.microsoft.com/fwlink/?LinkId=516908
[azure-preview-portal]: https://portal.azure.com/

[adfgetstarted]: data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[hivewalkthrough]: data-factory-data-transformation-activities.md

[image-data-factory-output-from-custom-activity]: ./media/data-factory-use-custom-activities/OutputFilesFromCustomActivity.png

[image-data-factory-download-logs-from-custom-activity]: ./media/data-factory-use-custom-activities/DownloadLogsFromCustomActivity.png
