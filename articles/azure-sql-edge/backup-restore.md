---
title: 'Copia de seguridad y restauración de bases de datos: Azure SQL Edge'
description: Información sobre las funcionalidades de copia de seguridad y restauración en Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: f2cc8901ee3952f7d258d768e175412254ec5d1a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90905945"
---
# <a name="back-up-and-restore-databases-in-azure-sql-edge"></a>Copia de seguridad y restauración de bases de datos en Azure SQL Edge 

Azure SQL Edge se crea según las versiones más recientes del motor de base de datos de Microsoft SQL. Proporciona funcionalidades de copia de seguridad y restauración de bases de datos similares a las disponibles en SQL Server en Linux y SQL Server que se ejecuta en contenedores. El componente de copias de seguridad y restauración ofrece una protección esencial para los datos almacenados en las bases de datos de Azure SQL Edge. 

Para minimizar el riesgo de una pérdida de datos catastrófica, debe realizar, de forma periódica, copias de seguridad de las bases de datos para conservar las modificaciones realizadas en los datos. Una estrategia bien diseñada de copia de seguridad y restauración le ayuda a proteger las bases de datos frente a la pérdida de datos provocada por diversos errores. Pruebe su estrategia con la restauración de un conjunto de copias de seguridad y, después, recupere la base de datos para prepararse para dar una respuesta eficaz en caso de desastre.

Para más información sobre la importancia de las copias de seguridad, consulte [Realizar copias de seguridad y restaurar bases de datos de SQL Server](/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases/).

Azure SQL Edge le permite realizar copias de seguridad y restaurar desde el almacenamiento local y blobs de Azure. Para más información, consulte [Copia de seguridad y restauración de SQL Server con el servicio Microsoft Azure Blob Storage](/sql/relational-databases/backup-restore/sql-server-backup-and-restore-with-microsoft-azure-blob-storage-service/) y [Copia de seguridad en URL de SQL Server](/sql/relational-databases/backup-restore/sql-server-backup-to-url).

## <a name="back-up-a-database-in-azure-sql-edge"></a>Copia de seguridad de una base de datos en Azure SQL Edge

Azure SQL Edge admite los mismos tipos de copia de seguridad que SQL Server. Para obtener una lista completa, consulte [Introducción a la copia de seguridad](/sql/relational-databases/backup-restore/backup-overview-sql-server/).

> [!IMPORTANT] 
> Las bases de datos creadas en Azure SQL Edge utilizan el modelo de recuperación simple de manera predeterminada. Por ello, no se pueden realizar copias de seguridad de registros en estas bases de datos. Si necesita hacerlo, requerirá un administrador para cambiar el modelo de recuperación de bases de datos al modelo de recuperación completa. Para obtener una lista completa de los modelos de recuperación admitidos por SQL Server, consulte [Introducción al modelo de recuperación](/sql/relational-databases/backup-restore/recovery-models-sql-server#RMov).

### <a name="back-up-to-local-disk"></a>Copia de seguridad en un disco local

En el ejemplo siguiente, use el comando de Transact-SQL `BACKUP DATABASE` para crear una copia de seguridad de base de datos en el contenedor. Para este ejemplo, cree una carpeta denominada *backup* para almacenar los archivos de copia de seguridad.

1. Cree una carpeta para las copias de seguridad. Ejecute este comando en el host donde se ejecuta el contenedor de Azure SQL Edge. En el comando siguiente, reemplace **<AzureSQLEdge_Container_Name>** por el nombre del contenedor de Azure SQL Edge en la implementación.

    ```bash
    sudo docker exec -it <AzureSQLEdge_Container_Name> mkdir /var/opt/mssql/backup
    ```

2. Conéctese a la instancia de Azure SQL Edge mediante SQL Server Management Studio (SSMS) o Azure Data Studio. Ejecute el comando `BACKUP DATABASE` para realizar la copia de seguridad de la base de datos de usuario. En el ejemplo siguiente, lleva a cabo la copia de seguridad de la base de datos *IronOreSilicaPrediction*.

    ```sql
    BACKUP DATABASE [IronOreSilicaPrediction] 
    TO DISK = N'/var/opt/mssql/backup/IronOrePredictDB.bak' 
    WITH NOFORMAT, NOINIT,  NAME = N'IronOreSilicaPrediction-Full Database Backup', 
    SKIP, NOREWIND, NOUNLOAD, COMPRESSION,  STATS = 10
    GO
    ```

3. Después de ejecutar el comando, si la copia de seguridad de la base de datos se realiza correctamente, verá mensajes similares a los siguientes en la sección de resultados de SSMS o Azure Data Studio.

    ```txt
    10 percent processed.
    20 percent processed.
    30 percent processed.
    40 percent processed.
    50 percent processed.
    60 percent processed.
    70 percent processed.
    80 percent processed.
    90 percent processed.
    100 percent processed.
    Processed 51648 pages for database 'IronOreSilicaPrediction', file 'IronOreSilicaPrediction' on file 1.
    Processed 2 pages for database 'IronOreSilicaPrediction', file 'IronOreSilicaPrediction_log' on file 1.
    BACKUP DATABASE successfully processed 51650 pages in 3.588 seconds (112.461 MB/sec).

    Completion time: 2020-04-09T23:54:48.4957691-07:00
    ```

### <a name="back-up-to-url"></a>Copia de seguridad en dirección URL

Azure SQL Edge admite copias de seguridad en blobs en páginas y blobs en bloques. Para más información, consulte [Copia de seguridad en blobs en bloques y blobs en páginas](https://docs.microsoft.com/sql/relational-databases/backup-restore/sql-server-backup-to-url#blockbloborpageblob). En el ejemplo siguiente, se realiza una copia de seguridad de la base de datos *IronOreSilicaPrediction* en un blob en bloques. 

1. Para configurar las copias de seguridad en blobs en bloques, primero tiene que generar un token de firma de acceso compartido (SAS) que puede usar para crear una credencial de SQL Server en Azure SQL Edge. El script crea una SAS que está asociada a una directiva de acceso almacenada. Para más información, consulte el artículo sobre las [firmas de acceso compartido, parte 1: Descripción del modelo SAS](https://azure.microsoft.com/documentation/articles/storage-dotnet-shared-access-signature-part-1/). El script también escribe el comando de T-SQL necesario para crear la credencial en SQL Server. En el siguiente script se supone que ya tiene una suscripción de Azure con una cuenta de almacenamiento y un contenedor de almacenamiento para las copias de seguridad.

    ```PowerShell
    # Define global variables for the script  
    $subscriptionName='<your subscription name>'   # the name of subscription name you will use
    $resourcegroupName = '<your resource group name>' # the name of resource group you will use
    $storageAccountName= '<your storage account name>' # the storage account name you will use for backups
    $containerName= '<your storage container name>'  # the storage container name to which you will attach the SAS policy with its SAS token  
    $policyName = 'SASPolicy' # the name of the SAS policy  

    # adds an authenticated Azure account for use in the session
    Login-AzAccount

    # set the tenant, subscription and environment for use in the rest of
    Select-AzSubscription -Subscription $subscriptionName

    # Generate the SAS token
    $sa = Get-AzStorageAccount -ResourceGroupName $resourcegroupName -Name $storageAccountName 
    $storagekey = Get-AzStorageAccountKey -ResourceGroupName $resourcegroupName -Name $storageAccountName 
    $storageContext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storagekey[0].Value
    $cbc = Get-AzStorageContainer -Name $containerName -Context $storageContext
    $policy = New-AzStorageContainerStoredAccessPolicy -Container $containerName -Policy $policyName -Context $storageContext -ExpiryTime $(Get-Date).ToUniversalTime().AddYears(10) -Permission "rwld"
    $sas = New-AzStorageContainerSASToken -Policy $policyName -Context $storageContext -Container $containerName
    Write-Host 'Shared Access Signature= '$($sas.Substring(1))''

    # Outputs the Transact SQL to the clipboard and to the screen to create the credential using the Shared Access Signature  
    Write-Host 'Credential T-SQL'  
    $tSql = "CREATE CREDENTIAL [{0}] WITH IDENTITY='Shared Access Signature', SECRET='{1}'" -f $cbc.CloudBlobContainer.Uri.AbsoluteUri,$sas.Substring(1)
    $tSql | clip  
    Write-Host $tSql
    ```

    Después de ejecutar correctamente el script, copie el comando `CREATE CREDENTIAL` en una herramienta de consulta. A continuación, conéctese a una instancia de SQL Server y ejecute el comando para crear la credencial con la SAS.

2. Conéctese a la instancia de Azure SQL Edge mediante SSMS o Azure Data Studio y cree la credencial con el comando del paso anterior. Asegúrese de reemplazar el comando `CREATE CREDENTIAL` por la salida real del paso anterior.

    ```sql
    IF NOT EXISTS  
    (SELECT * FROM sys.credentials
    WHERE name = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>')  
    CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>]
       WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
       SECRET = '<SAS_TOKEN>';
    ```

3. El comando siguiente realiza una copia de seguridad de *IronOreSilicaPrediction* en el contenedor de almacenamiento de Azure.

    ```sql
    BACKUP DATABASE IronOreSilicaPrediction
    TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mycontainername>/IronOreSilicaPrediction.bak'
    With MAXTRANSFERSIZE = 4194304,BLOCKSIZE=65536;  
    GO
    ```

## <a name="restore-a-database-in-azure-sql-edge"></a>Restauración de una base de datos en Azure SQL Edge

En Azure SQL Edge puede restaurar desde un disco local, una ubicación de red o una cuenta de Azure Blob Storage. Para más información sobre la restauración y recuperación en SQL Server, consulte [Información general sobre restauración y recuperación (SQL Server)](https://docs.microsoft.com/sql/relational-databases/backup-restore/restore-and-recovery-overview-sql-server). Para obtener información general sobre el modelo de recuperación simple en SQL Server, consulte [Restauraciones de base de datos completas (modelo de recuperación simple)](https://docs.microsoft.com/sql/relational-databases/backup-restore/complete-database-restores-simple-recovery-model).

> [!IMPORTANT] 
> Las bases de datos creadas en Azure SQL Edge no se pueden restaurar en una instancia de Microsoft SQL Server ni Azure SQL. Por otra parte, una base de datos creada en Microsoft SQL Server o Azure SQL se puede restaurar en Azure SQL Edge, siempre y cuando no contenga ninguna de las características que Azure SQL Edge no admite. 

### <a name="restore-from-a-local-disk"></a>Restauración desde un disco local

En este ejemplo se utiliza la copia de seguridad *IronOreSilicaPrediction* que ha realizado en el ejemplo anterior. Ahora, la restaurará como una nueva base de datos con un nombre diferente.

1. Si el archivo de copia de seguridad de base de datos aún no está presente en el contenedor, puede utilizar el comando siguiente para copiar el archivo en el contenedor. En el ejemplo siguiente, se supone que el archivo de copia de seguridad está presente en el host local y que se copia en la carpeta /var/opt/mssql/backup en un contenedor de Azure SQL Edge denominado *sql1*.

    ```bash
    sudo docker cp IronOrePredictDB.bak sql1:/var/opt/mssql/backup
    ```

2. Conéctese a la instancia de Azure SQL Edge mediante SSMS o Azure Data Studio para ejecutar el comando de restauración. En el ejemplo siguiente, se restaura **IronOrePredictDB.bak** para crear una nueva base de datos **IronOreSilicaPrediction_2**.

    ```sql
    Restore FilelistOnly from disk = N'/var/opt/mssql/backup/IronOrePredictDB.bak'

    Restore Database IronOreSilicaPrediction_2
    From disk = N'/var/opt/mssql/backup/IronOrePredictDB.bak'
    WITH MOVE 'IronOreSilicaPrediction' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_2.mdf',
    MOVE 'IronOreSilicaPrediction_log' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_2.ldf'
    ```

3. Después de ejecutar el comando de restauración y si la operación de restauración se realiza correctamente, verá mensajes similares a los siguientes en la ventana salida. 

    ```txt
    Processed 51648 pages for database 'IronOreSilicaPrediction_2', file 'IronOreSilicaPrediction' on file 1.
    Processed 2 pages for database 'IronOreSilicaPrediction_2', file 'IronOreSilicaPrediction_log' on file 1.
    RESTORE DATABASE successfully processed 51650 pages in 6.543 seconds (61.670 MB/sec).

    Completion time: 2020-04-13T23:49:21.1600986-07:00
    ```

### <a name="restore-from-url"></a>Restauración desde URL

Azure SQL Edge también admite la restauración de una base de datos desde una cuenta de Azure Storage. Puede restaurar desde los blobs en bloques o desde las copias de seguridad de blobs en páginas. En el ejemplo siguiente, se restaura el archivo de copia de seguridad de base de datos *IronOreSilicaPrediction_2020_04_16.bak* en un blob en bloques para crear la base de datos *IronOreSilicaPrediction_3*.

```sql
RESTORE DATABASE IronOreSilicaPrediction_3
FROM URL = 'https://mystorageaccount.blob.core.windows.net/mysecondcontainer/IronOreSilicaPrediction_2020_04_16.bak'
WITH MOVE 'IronOreSilicaPrediction' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_3.mdf', 
MOVE 'IronOreSilicaPrediction_log' TO '/var/opt/mssql/data/IronOreSilicaPrediction_Primary_3.ldf',
STATS = 10;
```


