---
title: Migración de una base de datos desde SQL Server a SQL Managed Instance habilitado para Azure Arc
description: Migración de una base de datos desde SQL Server a SQL Managed Instance habilitado para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 86563b0a44bade2cedaf76af3c247821756111fe
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932328"
---
# <a name="migrate-sql-server-to-azure-arc-enabled-sql-managed-instance"></a>Migración: de SQL Server a SQL Managed Instance habilitado para Azure Arc

En este escenario se le guía por los pasos necesarios para migrar una base de datos de una instancia de SQL Server a Azure SQL Managed Instance en Azure Arc mediante dos métodos diferentes de copia de seguridad y restauración.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="use-azure-blob-storage"></a>Uso de Azure Blob Storage 

Use Azure Blob Storage para migrar a SQL Managed Instance habilitado para Azure Arc.

Este método usa Azure Blob Storage como una ubicación de almacenamiento temporal en la que se puede hacer una copia de seguridad y desde la que se puede restaurar posteriormente.

### <a name="prerequisites"></a>Requisitos previos

- [Instalación de Azure Data Studio](install-client-tools.md)
- [Instalación del Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/)
- Suscripción de Azure

### <a name="step-1-provision-azure-blob-storage"></a>Paso 1: aprovisionamiento de Azure Blob Storage

1. Siga los pasos descritos en [Creación de una cuenta de Azure Blob Storage](../../storage/blobs/storage-blob-create-account-block-blob.md?tabs=azure-portal)
1. Inicio del Explorador de Azure Storage
1. [Inicie la sesión en Azure](../../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows#sign-in-to-azure) para acceder al almacenamiento de blobs creado en el paso anterior
1. Haga clic con el botón derecho en la cuenta de almacenamiento de blobs y seleccione **Crear contenedor de blobs** para crear un contenedor en el que se almacenará el archivo de copia de seguridad

### <a name="step-2-get-storage-blob-credentials"></a>Paso 2: obtención de credenciales de blobs de almacenamiento

1. En el Explorador de Microsoft Azure Storage, haga clic con el botón derecho en el contenedor de blobs que se acaba de crear y seleccione **Obtener firma de acceso compartido**

1. Seleccione **Lectura**, **Escritura** y **Lista**

1. Seleccione **Crear**

   Tome nota del URI y de la cadena de consultas de esta pantalla. Se necesitará en los pasos posteriores. Haga clic en el botón **Copiar** para guardarlos en el Bloc de notas, OneNote, etc.

1. Cierre la ventana **Firma de acceso compartido**.

### <a name="step-3-backup-database-file-to-azure-blob-storage"></a>Paso 3: copia del archivo de base de datos en Azure Blob Storage

En este paso, se conectará al SQL Server de origen y creará el archivo de copia de seguridad de la base de datos que deseamos migrar a SQL Managed Instance - Azure Arc.

1. Inicio de Azure Data Studio
1. Conéctese a la instancia de SQL Server que tiene la base de datos que quiere migrar a SQL Managed Instance - Azure Arc.
1. Haga clic con el botón derecho en la base de datos y seleccione **Nueva consulta**
1. Prepare la consulta con el formato siguiente, reemplazando los marcadores de posición indicados por el `<...>` usando la información de la firma de acceso compartido en los pasos anteriores.  Una vez que haya sustituido los valores, ejecute la consulta.

   ```sql
   IF NOT EXISTS  
   (SELECT * FROM sys.credentials
   WHERE name = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>')  
   CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>]
     WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
      SECRET = '<SAS_TOKEN>';  
   ```

1. Del mismo modo, prepare el comando **BACKUP DATABASE** tal como se indica a continuación para crear un archivo de copia de seguridad en el contenedor de blobs.  Una vez que haya sustituido los valores, ejecute la consulta.

   ```sql
   BACKUP DATABASE <database name> TO URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>'
   ```

1. Abra el Explorador de Microsoft Azure Storage y compruebe que el archivo de copia de seguridad creado en el paso anterior esté visible en el contenedor de blobs.

### <a name="step-4-restore-the-database-from-azure-blob-storage-to-sql-managed-instance---azure-arc"></a>Paso 4: restauración la base de datos de Azure Blob Storage a SQL Managed Instance - Azure Arc

1. Desde Azure Data Studio, inicie la sesión y conéctese a SQL Managed Instance - Azure Arc.
1. Expanda **Bases de datos del sistema**, haga clic con el botón derecho en **Base de datos maestra** y seleccione **Nueva consulta**.
1. En la ventana del editor de consultas, prepare y ejecute la misma consulta del paso anterior para crear las credenciales.

   ```sql
   IF NOT EXISTS  
   (SELECT * FROM sys.credentials
   WHERE name = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>')  
   CREATE CREDENTIAL [https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>]
     WITH IDENTITY = 'SHARED ACCESS SIGNATURE',  
     SECRET = '<SAS_TOKEN>';  
   ```

1. Prepare y ejecute el siguiente comando para comprobar que el archivo de copia de seguridad es legible y que está intacto.

   ```console
   RESTORE FILELISTONLY FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>/<file name>.bak'
   ```

1. Prepare y ejecute el comando **RESTORE DATABASE** tal como se indica a continuación para restaurar el archivo de copia de seguridad en una base de datos en SQL Managed Instance - Azure Arc

   ```sql
   RESTORE DATABASE <database name> FROM URL = 'https://<mystorageaccountname>.blob.core.windows.net/<mystorageaccountcontainername>/<file name>'
   WITH MOVE 'Test' to '/var/opt/mssql/data/<file name>.mdf'
   ,MOVE 'Test_log' to '/var/opt/mssql/data/<file name>.ldf'
   ,RECOVERY  
   ,REPLACE  
   ,STATS = 5;  
   GO
   ```

Obtenga más información sobre las copias de seguridad en URL aquí:

[Documentos de copia de seguridad en URL](/sql/relational-databases/backup-restore/sql-server-backup-to-url)

[Copia de seguridad en URL mediante SQL Server Management Studio (SSMS)](/sql/relational-databases/tutorial-sql-server-backup-and-restore-to-azure-blob-storage-service)

-------

## <a name="method-2-copy-the-backup-file-into-an-azure-sql-managed-instance---azure-arc-pod-using-kubectl"></a>Método 2: copia del archivo de copia de seguridad en un pod de Azure SQL Managed Instance - Azure Arc mediante kubectl

Mediante este método se toma un archivo de copia de seguridad creado a través de cualquier método y, a continuación, se copia en el almacenamiento local en el pod de Azure SQL Managed Instance para que pueda hacer la restauración desde ahí del mismo modo que lo haría en un sistema de archivos típico en Windows o Linux. En este escenario, usará el comando `kubectl cp` para copiar el archivo de un lugar al sistema de archivos del pod.

### <a name="prerequisites"></a>Requisitos previos

- Instalar y configurar kubectl para que apunte al clúster de Kubernetes donde se implementan los servicios de datos de Azure Arc
- Tenga una herramienta como Azure Data Studio o SQL Server Management Server instalada y conectada al SQL Server en el que desea crear el archivo de copia de seguridad o tenga un archivo. bak existente ya creado en el sistema de archivos local.

### <a name="step-1-backup-the-database-if-you-havent-already"></a>Paso 1: copia de seguridad de la base de datos si aún no lo ha hecho

Haga una copia de seguridad de la base de datos de SQL Server en la ruta de acceso de archivos locales, como una copia de seguridad de SQL Server típica en el disco:

 ```sql
BACKUP DATABASE Test
TO DISK = 'c:\tmp\test.bak'
WITH FORMAT, MEDIANAME = 'Test’ ;
GO
```

### <a name="step-2-copy-the-backup-file-into-the-pods-file-system"></a>Paso 2: copia del archivo de copia de seguridad en el sistema de archivos del pod

Busque el nombre del pod donde se implementa la instancia de SQL. El aspecto será parecido al siguiente: `pod/<sqlinstancename>-0`

Para obtener la lista de todos los pods, ejecute:

 ```console
kubectl get pods -n <namespace of data controller>
```

Ejemplo:

Copie el archivo de copia de seguridad del almacenamiento local en el pod de SQL del clúster.

 ```console
kubectl cp <source file location> <pod name>:var/opt/mssql/data/<file name> -n <namespace name>

#Example:
kubectl cp C:\Backupfiles\test.bak sqlinstance1-0:var/opt/mssql/data/test.bak -n arc
```

### <a name="step-3-restore-the-database"></a>Paso 3: Restaurar la base de datos

Prepare y ejecute el comando RESTORE para restaurar el archivo de copia de seguridad en la instancia administrada de Azure SQL - Azure Arc

```sql
RESTORE DATABASE test FROM DISK = '/var/opt/mssql/data/<file name>.bak'
WITH MOVE '<database name>' to '/var/opt/mssql/data/<file name>.mdf'  
,MOVE '<database name>' to '/var/opt/mssql/data/<file name>_log.ldf'  
,RECOVERY  
,REPLACE  
,STATS = 5;  
GO
```

Ejemplo:

```sql
RESTORE DATABASE test FROM DISK = '/var/opt/mssql/data/test.bak'
WITH MOVE 'test' to '/var/opt/mssql/data/test.mdf'  
,MOVE 'test' to '/var/opt/mssql/data/test_log.ldf'  
,RECOVERY  
,REPLACE  
,STATS = 5;  
GO
```


## <a name="next-steps"></a>Pasos siguientes

[Más información sobre las características y capacidades de SQL Managed Instance habilitado para Azure Arc](managed-instance-features.md)

[Comienzo con la creación de un controlador de datos](create-data-controller.md)

[¿Ya ha creado un controlador de datos? Creación de una instancia de SQL Managed Instance habilitado para Azure Arc](create-sql-managed-instance.md)