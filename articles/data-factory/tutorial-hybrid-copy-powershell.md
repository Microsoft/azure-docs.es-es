---
title: Copia de datos de SQL Server a Blob Storage mediante PowerShell
description: Aprenda a copiar datos de un almacén de datos local a la nube de Azure mediante un runtime de integración autohospedado en Azure Data Factory.
services: data-factory
author: nabhishek
ms.author: abnarain
manager: anandsub
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.topic: tutorial
ms.custom: seo-lt-2019; seo-dt-2019, devx-track-azurepowershell
ms.date: 01/22/2018
ms.openlocfilehash: d148bc81e81c4afa32682fa1462ecfbf5427f164
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89079241"
---
# <a name="tutorial-copy-data-from-a-sql-server-database-to-azure-blob-storage"></a>Tutorial: Copia de datos de una base de datos de SQL Server a Azure Blob Storage

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

En este tutorial, usará Azure PowerShell para crear una canalización de Data Factory que copie los datos de una base de datos de SQL Server a Azure Blob Storage. Cree y use una instancia de Integration Runtime autohospedado, que mueve los datos entre almacenes locales y en la nube.

> [!NOTE]
> En este artículo no se ofrece una introducción detallada al servicio Data Factory. Para más información, consulte [Introducción a Azure Data Factory](introduction.md).

En este tutorial, realizará los siguientes pasos:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Cree una instancia de Integration Runtime autohospedada.
> * Creación de los servicios vinculados SQL Server y Azure Storage.
> * Creación de los conjuntos de datos de SQL Server y Azure Blob.
> * Creación de una canalización con una actividad de copia para mover los datos.
> * Inicio de la ejecución de una canalización.
> * Supervisión de la ejecución de la canalización

## <a name="prerequisites"></a>Requisitos previos
### <a name="azure-subscription"></a>Suscripción de Azure
Antes de empezar, si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/).

### <a name="azure-roles"></a>Roles de Azure
Para crear instancias de Data Factory, la cuenta de usuario que use para iniciar sesión en Azure debe tener un rol *Colaborador* o *Propietario* asignado o ser *administrador* de la suscripción de Azure.

Para ver los permisos que tiene en la suscripción, vaya a Azure Portal, seleccione su nombre de usuario en la esquina superior derecha y, después, seleccione **Permisos**. Si tiene acceso a varias suscripciones, elija la correspondiente. Para instrucciones de ejemplo sobre cómo agregar un rol a un usuario, consulte el artículo [Administración del acceso mediante RBAC y Azure Portal](../role-based-access-control/role-assignments-portal.md).

### <a name="sql-server-2014-2016-and-2017"></a>SQL Server 2014, 2016 y 2017
En este tutorial, usará una base de datos de SQL Server como almacén de datos de *origen*. La canalización de Data Factory que crea en este tutorial copia los datos de esta base de datos de SQL Server (origen) a Azure Blob Storage (receptor). Luego, cree una tabla denominada **emp** en la base de datos de SQL Server e inserte un par de entradas de ejemplo en la tabla.

1. Inicie SQL Server Management Studio. Si no está instalada en su equipo, vaya a [Descarga de SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms).

1. Conéctese a una instancia de SQL Server con sus credenciales.

1. Cree una base de datos de ejemplo. En la vista de árbol, haga clic con el botón derecho en **Bases de datos** y, luego, seleccione **Nueva base de datos**.

1. En el cuadro de diálogo **Nueva base de datos**, escriba el nombre de la base de datos y haga clic en **Aceptar**.

1. Para crear la tabla **emp** e insertar en ella algunos datos de ejemplo, ejecute el siguiente script de consulta en la base de datos. En la vista de árbol, haga clic con el botón derecho en la base de datos que ha creado y, después, haga clic en **Nueva consulta**.

    ```sql
    CREATE TABLE dbo.emp
    (
        ID int IDENTITY(1,1) NOT NULL,
        FirstName varchar(50),
        LastName varchar(50)
    )
    GO

    INSERT INTO emp (FirstName, LastName) VALUES ('John', 'Doe')
    INSERT INTO emp (FirstName, LastName) VALUES ('Jane', 'Doe')
    GO
    ```


### <a name="azure-storage-account"></a>Cuenta de Azure Storage
En esta guía de inicio rápido, use una cuenta de Azure Storage (en concreto Blob Storage) de uso general como almacén de datos de destino o receptor en este tutorial. Si no dispone de una cuenta de Azure Storage de uso general, consulte [Creación de una cuenta de almacenamiento](../storage/common/storage-account-create.md). La canalización de Data Factory que crea en este tutorial copia los datos de la base de datos de SQL Server (origen) a esta instancia de Azure Blob Storage (receptor). 

#### <a name="get-storage-account-name-and-account-key"></a>Obtener un nombre y una clave de cuenta de Storage
En este tutorial, use el nombre y la clave de su cuenta de Azure Storage. Para obtener el nombre y la clave de una cuenta de almacenamiento, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con el nombre de usuario y la contraseña de Azure.

1. En el panel izquierdo, seleccione **Más servicios**, use la palabra clave **Almacenamiento** para realizar el filtro y, luego, seleccione **Cuentas de almacenamiento**.

    ![Buscar cuenta de Storage](media/doc-common-process/search-storage-account.png)

1. En la lista de cuentas de almacenamiento, filtre por su cuenta de almacenamiento (si fuera necesario) y, después, seleccione su cuenta de almacenamiento.

1. En la ventana **Cuenta de almacenamiento**, seleccione **Claves de acceso**.

1. En los cuadros **Nombre de la cuenta de almacenamiento** y **key1**, copie los valores y péguelos en el Bloc de notas, u otro editor, para su uso posterior en el tutorial.

#### <a name="create-the-adftutorial-container"></a>Creación del contenedor adftutorial
En esta sección se crea un contenedor de blobs denominado **adftutorial** en la instancia de Azure Blob Storage.

1. En la ventana **Cuenta de almacenamiento**, cambie a **Información general** y, después, seleccione **Blobs**.

    ![Selección de la opción Blobs](media/tutorial-hybrid-copy-powershell/select-blobs.png)

1. En la ventana **Blob service**, seleccione **Contenedor**.

1. En la ventana **Nuevo contenedor**, en el cuadro **Nombre**, escriba **adftutorial**y, después, seleccione **Aceptar**.

    ![Especificación de un nombre de contenedor](media/tutorial-hybrid-copy-powershell/new-container-dialog.png)

1. En la lista de contenedores, seleccione **adftutorial**.  

1. Mantenga abierta la ventana **contenedor** de **adftutorial**. Úselo para comprobar la salida al final de este tutorial. Data Factory crea automáticamente la carpeta de salida de este contenedor, por lo que no es necesario que el usuario la cree.


### <a name="windows-powershell"></a>Windows PowerShell

#### <a name="install-azure-powershell"></a>Instalar Azure Powershell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si no está en el equipo, instale la versión más reciente de Azure PowerShell. Para obtener instrucciones detalladas, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/install-Az-ps).

#### <a name="log-in-to-powershell"></a>Inicio de sesión en PowerShell

1. Inicie PowerShell en su equipo y manténgalo abierto hasta que finalice este tutorial de inicio rápido. Si lo cierra y vuelve a abrirlo, tendrá que volver a ejecutar estos comandos.

1. Ejecute el siguiente comando y escriba el nombre de usuario y la contraseña de Azure que utiliza para iniciar sesión en Azure Portal:

    ```powershell
    Connect-AzAccount
    ```        

1. Si tiene varias suscripciones de Azure, ejecute el siguiente comando para seleccionar la suscripción con la que desea trabajar. Reemplace **SubscriptionId** con el identificador de la suscripción de Azure:

    ```powershell
    Select-AzSubscription -SubscriptionId "<SubscriptionId>"    
    ```

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. Defina una variable para el nombre del grupo de recursos que usará en los comandos de PowerShell más adelante. Copie el comando siguiente en PowerShell, especifique el nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) (entre comillas dobles; por ejemplo, `"adfrg"`) y ejecute el comando. 
   
    ```powershell
    $resourceGroupName = "ADFTutorialResourceGroup"
    ```

1. Para crear el grupo de recursos de Azure, ejecute el comando siguiente:

    ```powershell
    New-AzResourceGroup $resourceGroupName -location 'East US'
    ```

    Si el grupo de recursos ya existe, puede que no desee sobrescribirlo. Asigne otro valor a la variable `$resourceGroupName` y ejecute el comando de nuevo.

1. Defina una variable para el nombre de la factoría de datos que pueda usar en los comandos de PowerShell más adelante. El nombre deben comenzar por una letra o un número y solo pueden contener letras, números y el carácter de guion (-).

    > [!IMPORTANT]
    >  Actualice el nombre de la factoría de datos por otro que sea globalmente único. Un ejemplo es ADFTutorialFactorySP1127.

    ```powershell
    $dataFactoryName = "ADFTutorialFactory"
    ```

1. Defina una variable para la ubicación de la factoría de datos:

    ```powershell
    $location = "East US"
    ```  

1. Para crear la factoría de datos, ejecute el siguiente cmdlet `Set-AzDataFactoryV2`:

    ```powershell       
    Set-AzDataFactoryV2 -ResourceGroupName $resourceGroupName -Location $location -Name $dataFactoryName
    ```

> [!NOTE]
>
> * El nombre de la factoría de datos debe ser globalmente único. Si recibe el siguiente error, cambie el nombre y vuelva a intentarlo.
>    ```
>    The specified data factory name 'ADFv2TutorialDataFactory' is already in use. Data factory names must be globally unique.
>    ```
> * Para crear instancias de Data Factory, a la cuenta de usuario que use para iniciar sesión en Azure debe se le deben asignar los roles *colaborador* o *propietario*, o bien debe de ser de un *administrador* de la suscripción a Azure.
> * Para una lista de las regiones de Azure en las que Data Factory está disponible actualmente, seleccione las regiones que le interesen en la página siguiente y expanda **Análisis** para poder encontrar **Data Factory**: [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). Los almacenes de datos (Azure Storage, Azure SQL Database, etc.) y los procesos (Azure HDInsight, etc.) que usa la factoría de datos pueden encontrarse en otras regiones.
>
>

## <a name="create-a-self-hosted-integration-runtime"></a>Creación de una instancia de Integration Runtime autohospedada

En esta sección se crea una instancia de Integration Runtime autohospedada y se asocia con un equipo local con la base de datos de SQL Server. Integration Runtime autohospedado es el componente que copia los datos de la base de datos de SQL Server del equipo a Azure Blob Storage.

1. Cree una variable para el nombre del runtime de integración. Use un nombre único y anótelo. Lo usará más adelante en este tutorial.

    ```powershell
   $integrationRuntimeName = "ADFTutorialIR"
    ```

1. Cree una instancia de Integration Runtime autohospedada.

    ```powershell
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $integrationRuntimeName -Type SelfHosted -Description "selfhosted IR description"
    ```

    Este es la salida de ejemplo:

    ```json
    Name              : ADFTutorialIR
    Type              : SelfHosted
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Description       : selfhosted IR description
    Id                : /subscriptions/<subscription ID>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/<dataFactoryName>/integrationruntimes/<integrationRuntimeName>
    ```

1. Para recuperar el estado de la instancia de Integration Runtime creada, ejecute el siguiente comando:

    ```powershell
   Get-AzDataFactoryV2IntegrationRuntime -name $integrationRuntimeName -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Status
    ```

    Este es la salida de ejemplo:

    ```json
    State                     : NeedRegistration
    Version                   :
    CreateTime                : 9/10/2019 3:24:09 AM
    AutoUpdate                : On
    ScheduledUpdateDate       :
    UpdateDelayOffset         :
    LocalTimeZoneOffset       :
    InternalChannelEncryption :
    Capabilities              : {}
    ServiceUrls               : {eu.frontend.clouddatahub.net}
    Nodes                     : {}
    Links                     : {}
    Name                      : <Integration Runtime name>
    Type                      : SelfHosted
    ResourceGroupName         : <resourceGroup name>
    DataFactoryName           : <dataFactory name>
    Description               : selfhosted IR description
    Id                        : /subscriptions/<subscription ID>/resourceGroups/<resourceGroupName>/providers/Microsoft.DataFactory/factories/<dataFactoryName>/integrationruntimes/<integrationRuntimeName>
    ```

1. Para recuperar las *claves de autenticación* para registrar la instancia de Integration Runtime autohospedado con el servicio Data Factory en la nube, ejecute el siguiente comando. Copie una de las claves (sin las comillas dobles) para registrar la instancia de Integration Runtime autohospedado que instalará en el equipo en el paso siguiente.

    ```powershell
    Get-AzDataFactoryV2IntegrationRuntimeKey -Name $integrationRuntimeName -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName | ConvertTo-Json
    ```

    Este es la salida de ejemplo:

    ```json
    {
        "AuthKey1":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=",
        "AuthKey2":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy="
    }
    ```

## <a name="install-the-integration-runtime"></a>Instalación de Integration Runtime
1. Descargue [Microsoft Integration Runtime](https://www.microsoft.com/download/details.aspx?id=39717) en un equipo Windows local y ejecute la instalación.

1. En el **Asistente para la instalación de Microsoft Integration Runtime**, haga clic en **Siguiente**.  

1. En la ventana **Contrato de licencia para el usuario final**, acepte los términos del Contrato de licencia y haga clic en **Siguiente**.

1. En la ventana **Carpeta de destino**, seleccione **Siguiente**.

1. En la ventana **Preparado para instalar Microsoft Integration Runtime**, haga clic en **Instalar**.

1. En **Ha completado el Asistente para la instalación de Microsoft Integration Runtime Setup**, seleccione **Finalizar**.

1. En la ventana **Registro de Integration Runtime (autohospedado)** , pegue la clave que guardó en la sección anterior y haga clic en **Registrar**.

    ![Registro de Integration Runtime](media/tutorial-hybrid-copy-powershell/register-integration-runtime.png)

1. En la ventana **Nuevo nodo de Integration Runtime (autohospedado)** , seleccione **Finalizar**.

    ![Ventana Nuevo nodo de Integration Runtime](media/tutorial-hybrid-copy-powershell/new-integration-runtime-node-page.png)

 1. Cuando Integration Runtime autohospedado se haya registrado correctamente, se muestra el siguiente mensaje:

    ![Se registró correctamente](media/tutorial-hybrid-copy-powershell/registered-successfully.png)

1. En la ventana **Registro de Integration Runtime (autohospedado)** , haga clic en **Iniciar Configuration Manager**.

1. Cuando el nodo se conecte al servicio en la nube, se mostrará el servicio en la nube:

    ![El nodo está conectado](media/tutorial-hybrid-copy-powershell/node-is-connected.png)

1. Para probar la conectividad a la base de datos de SQL Server, siga estos pasos:

    a. En la ventana **Configuration Manager**, cambie a la pestaña **Diagnósticos**.

    b. En el cuadro **Tipo de origen de datos**, seleccione **SqlServer**.

    c. Escriba el nombre del servidor.

    d. Escriba el nombre de la base de datos.

    e. Seleccione el modo de autenticación.

    f. Escriba el nombre de usuario.

    g. Escriba la contraseña asociada con el nombre de usuario.

    h. Para confirmar que Integration Runtime puede conectarse a SQL Server, seleccione **Probar**.  
    ![Conectividad correcta](media/tutorial-hybrid-copy-powershell/config-manager-diagnostics-tab.png)

    Si la conexión se ha realizado correctamente, verá una marca de verificación verde. De lo contrario, recibirá un mensaje de error asociado al error. Solucione los problemas y asegúrese de que Integration Runtime puede conectarse a la instancia de SQL Server.

    Anote todos los valores anteriores, ya que los usará más adelante en este mismo tutorial.

## <a name="create-linked-services"></a>Crear servicios vinculados
Para vincular los almacenes de datos y los servicios de proceso a la factoría de datos, cree servicios vinculados en una factoría de datos. En este tutorial, vincula su cuenta de Azure Storage y una instancia de SQL Server al almacén de datos. Los servicios vinculados tienen la información de conexión que usa el servicio Data Factory en el entorno de tiempo de ejecución para conectarse a ellos.

### <a name="create-an-azure-storage-linked-service-destinationsink"></a>Creación de un servicio vinculado a Azure Storage (destino/receptor)
En este paso, vinculará su cuenta de Azure Storage a la factoría de datos.

1. Cree un archivo JSON llamado *AzureStorageLinkedService.json* en la carpeta *C:\ADFv2Tutorial* con el siguiente código. Si la carpeta *ADFv2Tutorial* no existe, créela.  

    > [!IMPORTANT]
    > Antes de guardar el archivo, reemplace \<accountName> y \<accountKey> por el nombre y la clave de la cuenta de almacenamiento de Azure. Los anotó en la sección [Requisitos previos](#get-storage-account-name-and-account-key).

   ```json
    {
        "name": "AzureStorageLinkedService",
        "properties": {
            "annotations": [],
            "type": "AzureBlobStorage",
            "typeProperties": {
                "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountName>;AccountKey=<accountKey>;EndpointSuffix=core.windows.net"
            }
        }
    }
   ```

1. En PowerShell, cambie a la carpeta *C:\ADFv2Tutorial*.
   ```powershell
   Set-Location 'C:\ADFv2Tutorial'    
   ```

1. Para crear el servicio vinculado, AzureStorageLinkedService, ejecute el siguiente cmdlet `Set-AzDataFactoryV2LinkedService`:

   ```powershell
   Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "AzureStorageLinkedService" -File ".\AzureStorageLinkedService.json"
   ```

   Este es una salida de ejemplo:

    ```json
    LinkedServiceName : AzureStorageLinkedService
    ResourceGroupName : <resourceGroup name>
    DataFactoryName   : <dataFactory name>
    Properties        : Microsoft.Azure.Management.DataFactory.Models.AzureBlobStorageLinkedService
    ```

    Si recibe un error de "archivo no encontrado", confirme que existe el archivo, para lo que debe ejecutar el comando `dir`. Si el nombre del archivo tiene la extensión *.txt* (por ejemplo, AzureStorageLinkedService.json.txt), quítela y vuelva a ejecutar el comando de PowerShell.

### <a name="create-and-encrypt-a-sql-server-linked-service-source"></a>Creación y cifrado de un servicio vinculado de SQL Server (origen)
En este paso, vinculará la instancia de SQL Server a la factoría de datos.

1. Cree un archivo JSON llamado *SqlServerLinkedService.json* en la carpeta *C:\ADFv2Tutorial* mediante el siguiente código:

    > [!IMPORTANT]
    > Seleccione la sección que se basa en la autenticación que usa para conectarse a SQL Server.

    **Uso de la autenticación de SQL (sa):**

    ```json
    {  
        "name":"SqlServerLinkedService",
        "type":"Microsoft.DataFactory/factories/linkedservices",
        "properties":{  
            "annotations":[  

            ],
            "type":"SqlServer",
            "typeProperties":{  
                "connectionString":"integrated security=False;data source=<serverName>;initial catalog=<databaseName>;user id=<userName>;password=<password>"
            },
            "connectVia":{  
                "referenceName":"<integration runtime name> ",
                "type":"IntegrationRuntimeReference"
            }
        }
    }
   ```    

    **Con autenticación de Windows:**

    ```json
    {  
        "name":"SqlServerLinkedService",
        "type":"Microsoft.DataFactory/factories/linkedservices",
        "properties":{  
            "annotations":[  

            ],
            "type":"SqlServer",
            "typeProperties":{  
                "connectionString":"integrated security=True;data source=<serverName>;initial catalog=<databaseName>",
                "userName":"<username> or <domain>\\<username>",
                "password":{  
                    "type":"SecureString",
                    "value":"<password>"
                }
            },
            "connectVia":{  
                "referenceName":"<integration runtime name>",
                "type":"IntegrationRuntimeReference"
            }
        }
    }
    ```

    > [!IMPORTANT]
    > - Seleccione la sección que se basa en la autenticación que usa para conectarse a su instancia de SQL Server.
    > - Reemplace **\<integration runtime name>** por el nombre de su instancia del entorno de ejecución de integración.
    > - Antes de guardar el archivo, reemplace **\<servername>** , **\<databasename>** , **\<username>** y **\<password>** por los valores de su instancia de SQL Server.
    > - Si necesita usar una barra diagonal inversa (\\) en la cuenta del usuario o en el nombre del servidor, utilice el carácter de escape (\\) antes. Por ejemplo, use *mydomain\\\\myuser*.

1. Para cifrar los datos confidenciales (nombre de usuario, contraseña, etc.), ejecute el cmdlet `New-AzDataFactoryV2LinkedServiceEncryptedCredential`.  
    Este cifrado que las credenciales se cifran mediante la API de protección de datos (DPAPI). Las credenciales cifradas se almacenan de manera local en el nodo de Integration Runtime autohospedado (máquina local). La carga de salida se puede redirigir a otro archivo JSON (en este caso, *encryptedLinkedService.json*) que contiene las credenciales cifradas.

   ```powershell
   New-AzDataFactoryV2LinkedServiceEncryptedCredential -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -IntegrationRuntimeName $integrationRuntimeName -File ".\SQLServerLinkedService.json" > encryptedSQLServerLinkedService.json
   ```

1. Ejecute el comando siguiente, que crea EncryptedSqlServerLinkedService:

   ```powershell
   Set-AzDataFactoryV2LinkedService -DataFactoryName $dataFactoryName -ResourceGroupName $ResourceGroupName -Name "EncryptedSqlServerLinkedService" -File ".\encryptedSqlServerLinkedService.json"
   ```


## <a name="create-datasets"></a>Creación de conjuntos de datos
En este paso, crea conjuntos de datos de entrada y salida. Estos representan los datos de entrada y de salida para la operación de copia, en la que se copian datos de la base de datos de SQL Server a Azure Blob Storage.

### <a name="create-a-dataset-for-the-source-sql-server-database"></a>Creación de un conjunto de datos para la base de datos de origen SQL Server
En este paso, defina un conjunto de datos que represente los datos de la instancia de la base de datos de SQL Server. El conjunto de datos es del tipo SqlServerTable. Hace referencia al servicio vinculado de SQL Server que creó en el paso anterior. El servicio vinculado tiene la información de conexión que el servicio Data Factory usa para conectarse a su instancia de SQL Server en el runtime. Este conjunto de datos especifica la tabla SQL de la base de datos que contiene los datos. En este tutorial, la tabla **emp** contiene los datos de origen.

1. Cree un archivo JSON denominado *SqlServerDataset.json* en la carpeta *C:\ADFv2Tutorial* con el siguiente código:  
    ```json
    {  
        "name":"SqlServerDataset",
        "properties":{  
            "linkedServiceName":{  
                "referenceName":"EncryptedSqlServerLinkedService",
                "type":"LinkedServiceReference"
            },
            "annotations":[  

            ],
            "type":"SqlServerTable",
            "schema":[  

            ],
            "typeProperties":{  
                "schema":"dbo",
                "table":"emp"
            }
        }
    }
    ```

1. Para crear el conjunto de datos SqlServerDataset, ejecute el cmdlet `Set-AzDataFactoryV2Dataset`.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SqlServerDataset" -File ".\SqlServerDataset.json"
    ```

    Este es la salida de ejemplo:

    ```json
    DatasetName       : SqlServerDataset
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.SqlServerTableDataset
    ```

### <a name="create-a-dataset-for-azure-blob-storage-sink"></a>Creación de un conjunto de datos para Azure Blob Storage (receptor)
En este paso, se define un conjunto de datos que representa los datos que se van a copiar a Azure Blob Storage. El conjunto de datos es del tipo AzureBlob. Hace referencia al servicio vinculado Azure Storage que creó anteriormente en este tutorial.

El servicio vinculado tiene la información de conexión que Data Factory usa en el runtime para conectarse a su cuenta de Azure Storage. Este conjunto de datos especifica la carpeta del almacenamiento de Azure a la que se copian los datos desde la base de datos de SQL Server. En este tutorial, la carpeta es: *adftutorial/fromonprem*, donde `fromonprem` es el contenedor de blobs y `adftutorial` es la carpeta.

1. Cree un archivo JSON denominado *AzureBlobDataset.json* en la carpeta *C:\ADFv2Tutorial* con el siguiente código:

    ```json
    {  
        "name":"AzureBlobDataset",
        "properties":{  
            "linkedServiceName":{  
                "referenceName":"AzureStorageLinkedService",
                "type":"LinkedServiceReference"
            },
            "annotations":[  

            ],
            "type":"DelimitedText",
            "typeProperties":{  
                "location":{  
                    "type":"AzureBlobStorageLocation",
                    "folderPath":"fromonprem",
                    "container":"adftutorial"
                },
                "columnDelimiter":",",
                "escapeChar":"\\",
                "quoteChar":"\""
            },
            "schema":[  

            ]
        },
        "type":"Microsoft.DataFactory/factories/datasets"
    }
    ```

1. Para crear el conjunto de datos AzureBlobDataset, ejecute el cmdlet `Set-AzDataFactoryV2Dataset`.

    ```powershell
    Set-AzDataFactoryV2Dataset -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "AzureBlobDataset" -File ".\AzureBlobDataset.json"
    ```

    Este es la salida de ejemplo:

    ```json
    DatasetName       : AzureBlobDataset
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Structure         :
    Properties        : Microsoft.Azure.Management.DataFactory.Models.DelimitedTextDataset
    ```

## <a name="create-a-pipeline"></a>Crear una canalización
En este tutorial, creará una canalización con una actividad de copia. La actividad de copia usa SqlServerDataset como el conjunto de datos de entrada y AzureBlobDataset como conjunto de datos de salida. El tipo de origen se establece en *SqlSource* y el tipo de receptor en *BlobSink*.

1. Cree un archivo JSON denominado *SqlServerToBlobPipeline.json* en la carpeta *C:\ADFv2Tutorial* con el siguiente código:

    ```json
    {  
        "name":"SqlServerToBlobPipeline",
        "properties":{  
            "activities":[  
                {  
                    "name":"CopySqlServerToAzureBlobActivity",
                    "type":"Copy",
                    "dependsOn":[  

                    ],
                    "policy":{  
                        "timeout":"7.00:00:00",
                        "retry":0,
                        "retryIntervalInSeconds":30,
                        "secureOutput":false,
                        "secureInput":false
                    },
                    "userProperties":[  

                    ],
                    "typeProperties":{  
                        "source":{  
                            "type":"SqlServerSource"
                        },
                        "sink":{  
                            "type":"DelimitedTextSink",
                            "storeSettings":{  
                                "type":"AzureBlobStorageWriteSettings"
                            },
                            "formatSettings":{  
                                "type":"DelimitedTextWriteSettings",
                                "quoteAllText":true,
                                "fileExtension":".txt"
                            }
                        },
                        "enableStaging":false
                    },
                    "inputs":[  
                        {  
                            "referenceName":"SqlServerDataset",
                            "type":"DatasetReference"
                        }
                    ],
                    "outputs":[  
                        {  
                            "referenceName":"AzureBlobDataset",
                            "type":"DatasetReference"
                        }
                    ]
                }
            ],
            "annotations":[  

            ]
        }
    }
    ```

1. Para crear la canalización SqlServerToBlobPipeline, ejecute el cmdlet `Set-AzDataFactoryV2Pipeline`.

    ```powershell
    Set-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -Name "SQLServerToBlobPipeline" -File ".\SQLServerToBlobPipeline.json"
    ```

    Este es la salida de ejemplo:

    ```json
    PipelineName      : SQLServerToBlobPipeline
    ResourceGroupName : <resourceGroupName>
    DataFactoryName   : <dataFactoryName>
    Activities        : {CopySqlServerToAzureBlobActivity}
    Parameters        :  
    ```

## <a name="create-a-pipeline-run"></a>Creación de una ejecución de canalización
Inicie la ejecución de la canalización SQLServerToBlobPipeline y capture el identificador de dicha ejecución para poder supervisarla en el futuro.

```powershell
$runId = Invoke-AzDataFactoryV2Pipeline -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineName 'SQLServerToBlobPipeline'
```

## <a name="monitor-the-pipeline-run"></a>Supervisión de la ejecución de la canalización

1. Para comprobar continuamente el estado de ejecución de la canalización SQLServerToBlobPipeline, ejecute el siguiente script en PowerShell e imprima el resultado final:

    ```powershell
    while ($True) {
        $result = Get-AzDataFactoryV2ActivityRun -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName -PipelineRunId $runId -RunStartedAfter (Get-Date).AddMinutes(-30) -RunStartedBefore (Get-Date).AddMinutes(30)

        if (($result | Where-Object { $_.Status -eq "InProgress" } | Measure-Object).count -ne 0) {
            Write-Host "Pipeline run status: In Progress" -foregroundcolor "Yellow"
            Start-Sleep -Seconds 30
        }
        else {
            Write-Host "Pipeline 'SQLServerToBlobPipeline' run finished. Result:" -foregroundcolor "Yellow"
            $result
            break
        }
    }
    ```

    Este es el resultado de la ejecución de ejemplo:

    ```JSON
    ResourceGroupName    : <resourceGroupName>
    DataFactoryName      : <dataFactoryName>
    ActivityRunId        : 24af7cf6-efca-4a95-931d-067c5c921c25
    ActivityName         : CopySqlServerToAzureBlobActivity
    ActivityType         : Copy
    PipelineRunId        : 7b538846-fd4e-409c-99ef-2475329f5729
    PipelineName         : SQLServerToBlobPipeline
    Input                : {source, sink, enableStaging}
    Output               : {dataRead, dataWritten, filesWritten, sourcePeakConnections...}
    LinkedServiceName    :
    ActivityRunStart     : 9/11/2019 7:10:37 AM
    ActivityRunEnd       : 9/11/2019 7:10:58 AM
    DurationInMs         : 21094
    Status               : Succeeded
    Error                : {errorCode, message, failureType, target}
    AdditionalProperties : {[retryAttempt, ], [iterationHash, ], [userProperties, {}], [recoveryStatus, None]...}
    ```

1. Mediante la ejecución del comando siguiente puede obtener el identificador de la ejecución de la canalización SQLServerToBlobPipeline y comprobar el resultado detallado de la ejecución de la actividad:

    ```powershell
    Write-Host "Pipeline 'SQLServerToBlobPipeline' run result:" -foregroundcolor "Yellow"
    ($result | Where-Object {$_.ActivityName -eq "CopySqlServerToAzureBlobActivity"}).Output.ToString()
    ```

    Este es el resultado de la ejecución de ejemplo:

    ```json
    {  
        "dataRead":36,
        "dataWritten":32,
        "filesWritten":1,
        "sourcePeakConnections":1,
        "sinkPeakConnections":1,
        "rowsRead":2,
        "rowsCopied":2,
        "copyDuration":18,
        "throughput":0.01,
        "errors":[  

        ],
        "effectiveIntegrationRuntime":"ADFTutorialIR",
        "usedParallelCopies":1,
        "executionDetails":[  
            {  
                "source":{  
                    "type":"SqlServer"
                },
                "sink":{  
                    "type":"AzureBlobStorage",
                    "region":"CentralUS"
                },
                "status":"Succeeded",
                "start":"2019-09-11T07:10:38.2342905Z",
                "duration":18,
                "usedParallelCopies":1,
                "detailedDurations":{  
                    "queuingDuration":6,
                    "timeToFirstByte":0,
                    "transferDuration":5
                }
            }
        ]
    }
    ```

## <a name="verify-the-output"></a>Comprobación del resultado
La canalización automáticamente la carpeta de salida *fromonprem* en el contenedor de blobs `adftutorial`. Confirme que ve el archivo *dbo.emp.txt* en la carpeta de salida.

1. En Azure Portal, en la ventana Contenedor **adftutorial**, seleccione **Actualizar** para ver la carpeta de salida.
1. Seleccione `fromonprem` en la lista de carpetas.
1. Confirme que ve un archivo denominado `dbo.emp.txt`.

    ![Archivo de salida](media/tutorial-hybrid-copy-powershell/fromonprem-file.png)


## <a name="next-steps"></a>Pasos siguientes
La canalización de este ejemplo copia los datos de una ubicación a otra en Azure Blob Storage. Ha aprendido a:

> [!div class="checklist"]
> * Creación de una factoría de datos.
> * Cree una instancia de Integration Runtime autohospedada.
> * Creación de los servicios vinculados SQL Server y Azure Storage.
> * Creación de los conjuntos de datos de SQL Server y Azure Blob.
> * Creación de una canalización con una actividad de copia para mover los datos.
> * Inicio de la ejecución de una canalización.
> * Supervisión de la ejecución de la canalización

Para ver una lista de los almacenes de datos compatibles con Data Factory, consulte los [almacenes de datos disponibles](copy-activity-overview.md#supported-data-stores-and-formats).

Para obtener información acerca de cómo copiar datos de forma masiva de un origen a un destino, pase al tutorial siguiente:

> [!div class="nextstepaction"]
>[Copiar datos de forma masiva](tutorial-bulk-copy.md)
