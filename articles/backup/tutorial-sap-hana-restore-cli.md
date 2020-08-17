---
title: 'Tutorial: restauración de SAP HANA DB en Azure mediante la CLI'
description: En este tutorial aprenderá a restaurar las bases de datos de SAP HANA que se ejecutan en una máquina virtual de Azure de un almacén de Recovery Services de Azure Backup mediante la CLI de Azure.
ms.topic: tutorial
ms.date: 12/4/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 282f0ee61ffae455d6d3e49ea445d5ddc2fe56ac
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500834"
---
# <a name="tutorial-restore-sap-hana-databases-in-an-azure-vm-using-azure-cli"></a>Tutorial: Restauración de las bases de datos de SAP HANA en una máquina virtual de Azure con la CLI de Azure

La CLI de Azure se usa para crear y administrar los recursos de Azure desde la línea de comandos o mediante scripts. En esta documentación se detalla cómo restaurar una base de datos de SAP HANA con copia de seguridad en una máquina virtual de Azure con la CLI de Azure. También puede llevar a cabo estos pasos con [Azure Portal](./sap-hana-db-restore.md).

Use [Azure Cloud Shell](tutorial-sap-hana-backup-cli.md) para ejecutar comandos de la CLI.

Al final de este tutorial podrá realizar lo siguiente:

> [!div class="checklist"]
>
> * Ver los puntos de restauración de una base de datos con copia de seguridad
> * Restaurar una base de datos

En este tutorial se presupone que tiene una base de datos de SAP HANA que se ejecuta en una máquina virtual de Azure con copia de seguridad realizada mediante Azure Backup. Si ha usado [Realización de una copia de seguridad de una base de datos de SAP HANA en Azure mediante la CLI](tutorial-sap-hana-backup-cli.md) para realizar la copia de seguridad de la base de datos de SAP HANA, estará usando los siguientes recursos:

* un grupo de recursos denominado *saphanaResourceGroup*,
* un almacén denominado *saphanaVault*,
* un contenedor protegido denominado *VMAppContainer;Compute;saphanaResourceGroup;saphanaVM*,
* el elemento o la base de datos con copia de seguridad denominado *saphanadatabase;hxe;hxe* y
* los recursos de la región *westus2*.

## <a name="view-restore-points-for-a-backed-up-database"></a>Ver los puntos de restauración de una base de datos con copia de seguridad

Para ver la lista de todos los puntos de recuperación de una base de datos, use el cmdlet [az backup recoverpoint list](/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-show-log-chain) como se indica a continuación:

```azurecli-interactive
az backup recoverypoint list --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
   --output table
```

La lista de puntos de recuperación tendrá el siguiente aspecto:

```output
Name                      Time                               BackupManagementType   Item Name               RecoveryPointType
-------------------       ---------------------------------  ---------------------  ----------------------  ------------------
7660777527047692711       2019-12-10T04:00:32.346000+00:00   AzureWorkload          SAPHanaDtabase;hxe;hxe  Full
7896624824685666836       2019-12-15T10:33:32.346000+00:00   AzureWorkload          SAPHanaDtabase;hxe;hxe  Differential
DefaultRangeRecoveryPoint                                    AzureWorkload          SAPHanaDtabase;hxe;hxe  Log
```

Como puede ver, la lista anterior contiene tres puntos de recuperación: para las copias de seguridad completas, las diferenciales y de registro.

>[!NOTE]
>También puede ver los puntos inicial y final de cada cadena de copia de seguridad de registro íntegra mediante el cmdlet [az backup recoverypoin show-log-chain](/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-show-log-chain).

## <a name="prerequisites-to-restore-a-database"></a>Requisitos previos para restaurar una base de datos

Antes de restaurar una base de datos, asegúrese de que se cumplen los requisitos previos siguientes:

* Solo puede restaurar la base de datos en una instancia de SAP HANA que se encuentre en la misma región.
* La instancia de destino debe estar registrada en el mismo almacén que la de origen.
* Azure Backup no puede identificar dos instancias de SAP HANA diferentes en la misma máquina virtual. Por lo tanto, no es posible restaurar los datos de una instancia en otra en la misma máquina virtual.

## <a name="restore-a-database"></a>Restaurar una base de datos

Azure Backup puede restaurar bases de datos de SAP HANA que se ejecutan en máquinas virtuales de Azure como se indica a continuación:

* Restaurar a una fecha u hora específicas (con precisión de segundos) mediante copias de seguridad de registros. Azure Backup determina automáticamente la copia de seguridad diferencial o completa apropiada, y la cadena de copias de seguridad de registros necesarias para restaurar los datos en función del tiempo seleccionado.
* Restaurar una copia de seguridad completa o diferencial específica para restaurar a un punto de recuperación específico.

Para restaurar una base de datos, use el cmdlet [az restore-azurewl](/cli/azure/backup/restore?view=azure-cli-latest#az-backup-restore-restore-azurewl), que requiere un objeto de configuración de recuperación como una de las entradas. Este objeto se puede generar mediante el cmdlet [az backup recoveryconfig show](/cli/azure/backup/recoveryconfig?view=azure-cli-latest#az-backup-recoveryconfig-show). El objeto de configuración de recuperación contiene todos los detalles para realizar la restauración. Uno de ellos es el modo de restauración: **OriginalWorkloadRestore** o **AlternateWorkloadRestore**.

>[!NOTE]
> **OriginalWorkloadRestore**: restaura los datos en la misma instancia de SAP HANA de origen. Esta opción sobrescribe la base de datos original. <br>
> **AlternateWorkloadRestore**: restaura la base de datos en una ubicación alternativa y mantiene la base de datos de origen original.

## <a name="restore-to-alternate-location"></a>Restauración a una ubicación alternativa

Para restaurar una base de datos en una ubicación alternativa, use **AlternateWorkloadRestore** como modo de restauración. A continuación debe elegir el punto de restauración, que puede ser un momento anterior o cualquiera de los puntos de restauración anteriores.

En este tutorial se realizará la restauración a un punto de restauración anterior. [Consulte la lista de puntos de restauración](#view-restore-points-for-a-backed-up-database) para la base de datos y elija el momento al que desea realizar la restauración. En este tutorial se usará el punto de restauración con el nombre *7660777527047692711*.

Con el nombre de punto de restauración anterior y el modo de restauración, vamos a crear el objeto de configuración de recuperación mediante el cmdlet [az backup recoveryconfig show](/cli/azure/backup/recoveryconfig?view=azure-cli-latest#az-backup-recoveryconfig-show). Echemos un vistazo a lo que significa cada uno de los parámetros restantes de este cmdlet:

* **--target-item-name** es el nombre que utilizará la base de datos restaurada. En este caso, *restored_database*.
* **--target-server-name** es el nombre de un servidor SAP HANA que se ha registrado correctamente en un almacén de Recovery Services y se encuentra en la misma región que la base de datos que se va a restaurar. En este tutorial se va a restaurar la base de datos en el mismo servidor SAP HANA que se ha protegido, denominado *hxehost*.
* **--target-server-type**; para la restauración de las bases de datos de SAP HANA, se debe usar **SapHanaDatabase**.

```azurecli-interactive

az backup recoveryconfig show --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --restore-mode AlternateWorkloadRestore \
    --rp-name 7660777527047692711 \
    --target-item-name restored_database \
    --target-server-name hxehost \
    --target-server-type HANAInstance \
    --workload-type SAPHANA \
    --output json
```

La respuesta a la consulta anterior será un objeto de configuración de recuperación con un aspecto similar al siguiente:

```output
{"restore_mode": "AlternateLocation", "container_uri": " VMAppContainer;Compute;saphanaResourceGroup;saphanaVM ", "item_uri": "SAPHanaDatabase;hxe;hxe", "recovery_point_id": "7660777527047692711", "item_type": "SAPHana", "source_resource_id": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.Compute/virtualMachines/saphanavm", "database_name": null, "container_id": null, "alternate_directory_paths": null}
```

Ahora, para restaurar la base de datos, ejecute el cmdlet [az restore-azurewl](/cli/azure/backup/restore?view=azure-cli-latest#az-backup-restore-restore-azurewl). Para usar este comando, se especificará la salida JSON anterior, guardada en un archivo denominado *recoveryconfig.json*.

```azurecli-interactive
az backup restore restore-azurewl --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --restore-config recoveryconfig.json \
    --output table
```

El resultado tendrá un aspecto similar al siguiente:

```output
Name                                  Resource
------------------------------------  -------
5b198508-9712-43df-844b-977e5dfc30ea  SAPHANA
```

La respuesta le proporcionará el nombre del trabajo. Este nombre de trabajo se puede usar para realizar el seguimiento del estado del trabajo mediante el cmdlet [az backup job show](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-show).

## <a name="restore-and-overwrite"></a>Restauración y sobrescritura

Para la restauración en la ubicación original, usaremos **OrignialWorkloadRestore** como modo de restauración. A continuación debe elegir el punto de restauración, que puede ser un momento anterior o cualquiera de los puntos de restauración anteriores.

En este tutorial elegiremos el momento anterior "28-11-2019-09:53:00" para la restauración. Puede proporcionar este punto de restauración en los siguientes formatos: dd-mm-aaaa, dd-mm-aaaa-hh:mm:ss. Para elegir un momento dado para la restauración, use el cmdlet [az backup recoverypoint show-log-chain](/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-show-log-chain), que muestra los intervalos de copias de seguridad de la cadena de registro íntegras.

```azurecli-interactive
az backup recoveryconfig show --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --restore-mode OriginalWorkloadRestore \
    --log-point-in-time 28-11-2019-09:53:00 \
    --output json
```

La respuesta a la consulta anterior será un objeto de configuración de recuperación con el aspecto siguiente:

```output
{"restore_mode": "OriginalLocation", "container_uri": " VMAppContainer;Compute;saphanaResourceGroup;saphanaVM ", "item_uri": "SAPHanaDatabase;hxe;hxe", "recovery_point_id": "DefaultRangeRecoveryPoint", "log_point_in_time": "28-11-2019-09:53:00", "item_type": "SAPHana", "source_resource_id": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.Compute/virtualMachines/saphanavm", "database_name": null, "container_id": null, "alternate_directory_paths": null}"
```

Ahora, para restaurar la base de datos, ejecute el cmdlet [az restore-azurewl](/cli/azure/backup/restore?view=azure-cli-latest#az-backup-restore-restore-azurewl). Para usar este comando, se especificará la salida JSON anterior, guardada en un archivo denominado *recoveryconfig.json*.

```azurecli-interactive
az backup restore restore-azurewl --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --restore-config recoveryconfig.json \
    --output table
```

El resultado tendrá un aspecto similar al siguiente:

```output
Name                                  Resource
------------------------------------  --------
5b198508-9712-43df-844b-977e5dfc30ea  SAPHANA
```

La respuesta le proporcionará el nombre del trabajo. Este nombre de trabajo se puede usar para realizar el seguimiento del estado del trabajo mediante el cmdlet [az backup job show](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-show).

## <a name="restore-as-files"></a>Restaurar como archivos

Para restaurar los datos de la copia de seguridad como archivos en lugar de una base de datos, utilizaremos **Restaurar como archivos** como el modo de restauración. A continuación, elija el punto de restauración, que puede ser un momento anterior o cualquiera de los puntos de restauración anteriores. Cuando los archivos se vuelcan en una ruta de acceso especificada, puede llevar estos archivos a cualquier máquina de SAP HANA en la que quiera restaurarlos como base de datos. Dado que puede mover estos archivos a cualquier máquina, ahora puede restaurar los datos entre suscripciones y regiones.

En este tutorial, para la restauración elegiremos el momento anterior `28-11-2019-09:53:00` y la ubicación para volcar los archivos de copia de seguridad como `/home/saphana/restoreasfiles` en el mismo servidor de SAP HANA. Puede proporcionar este punto de restauración en uno de los siguientes formatos: **dd-mm-aaaa** o **dd-mm-aaaa-hh:mm:ss**. Para elegir un momento dado para la restauración, use el cmdlet [az backup recoverypoint show-log-chain](/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-show-log-chain), que muestra los intervalos de copias de seguridad de la cadena de registro íntegras.

Con el nombre de punto de restauración anterior y el modo de restauración, vamos a crear el objeto de configuración de recuperación mediante el cmdlet [az backup recoveryconfig show](/cli/azure/backup/recoveryconfig?view=azure-cli-latest#az-backup-recoveryconfig-show). Echemos un vistazo a lo que significa cada uno de los parámetros restantes de este cmdlet:

* **--target-container-name** es el nombre de un servidor de SAP HANA que se ha registrado correctamente en un almacén de Recovery Services y se encuentra en la misma región que la base de datos que se va a restaurar. En este tutorial, se va a restaurar la base de datos como archivos en el mismo servidor de SAP HANA que se ha protegido, llamado *hxehost*.
* **--rp-name** Para una restauración a un momento dado, el nombre del punto de restauración será **DefaultRangeRecoveryPoint**.

```azurecli-interactive
az backup recoveryconfig show --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --item-name saphanadatabase;hxe;hxe \
    --restore-mode RestoreAsFiles \
    --log-point-in-time 28-11-2019-09:53:00 \
    --rp-name DefaultRangeRecoveryPoint \
    --target-container-name VMAppContainer;Compute;saphanaResourceGroup;saphanaVM \
    --filepath /home/saphana/restoreasfiles \
    --output json
```

La respuesta a la consulta anterior será un objeto de configuración de recuperación con el aspecto siguiente:

```output
{
  "alternate_directory_paths": null,
  "container_id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.RecoveryServices/vaults/SAPHANAVault/backupFabrics/Azure/protectionContainers/VMAppContainer;Compute;SAPHANA;hanamachine",
  "container_uri": "VMAppContainer;compute;saphana;hanamachine",
  "database_name": null,
  "filepath": "/home/",
  "item_type": "SAPHana",
  "item_uri": "SAPHanaDatabase;hxe;hxe",
  "log_point_in_time": "04-07-2020-09:53:00",
  "recovery_mode": "FileRecovery",
  "recovery_point_id": "DefaultRangeRecoveryPoint",
  "restore_mode": "AlternateLocation",
  "source_resource_id": "/subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/saphanaResourceGroup/providers/Microsoft.Compute/virtualMachines/hanamachine"
}
```

Ahora, para restaurar la base de datos como archivos, ejecute el cmdlet [az restore-azurewl](/cli/azure/backup/restore?view=azure-cli-latest#az-backup-restore-restore-azurewl). Para usar este comando, se especificará la salida JSON anterior, guardada en un archivo llamado *recoveryconfig.json*.

```azurecli-interactive
az backup restore restore-azurewl --resource-group saphanaResourceGroup \
    --vault-name saphanaVault \
    --restore-config recoveryconfig.json \
    --output json
```

El resultado tendrá un aspecto similar al siguiente:

```output
{
  "eTag": null,
  "id": "/Subscriptions/ef4ab5a7-c2c0-4304-af80-af49f48af3d1/resourceGroups/SAPHANARESOURCEGROUP/providers/Microsoft.RecoveryServices/vaults/SAPHANAVault/backupJobs/608e737e-c001-47ca-8c37-57d909c8a704",
  "location": null,
  "name": "608e737e-c001-47ca-8c37-57d909c8a704",
  "properties": {
    "actionsInfo": [
      "Cancellable"
    ],
    "activityId": "7ddd3c3a-c0eb-11ea-a5f8-54ee75ec272a",
    "backupManagementType": "AzureWorkload",
    "duration": "0:00:01.781847",
    "endTime": null,
    "entityFriendlyName": "HXE [hxehost]",
    "errorDetails": null,
    "extendedInfo": {
      "dynamicErrorMessage": null,
      "propertyBag": {
        "Job Type": "Restore as files"
      },
      "tasksList": [
        {
          "status": "InProgress",
          "taskId": "Transfer data from vault"
        }
      ]
    },
    "jobType": "AzureWorkloadJob",
    "operation": "Restore",
    "startTime": "2020-07-08T07:20:29.336434+00:00",
    "status": "InProgress",
    "workloadType": "SAPHanaDatabase"
  },
  "resourceGroup": "saphanaResourceGroup",
  "tags": null,
  "type": "Microsoft.RecoveryServices/vaults/backupJobs"
}
```

La respuesta le proporcionará el nombre del trabajo. Este nombre de trabajo se puede usar para realizar el seguimiento del estado del trabajo mediante el cmdlet [az backup job show](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-show).

Los archivos que se vuelcan en el contenedor de destino son:

* Archivos de copia de seguridad de bases de datos
* Archivos de catálogo
* Archivos de metadatos JSON (para cada archivo de copia de seguridad implicado)

Típicamente, una ruta de acceso a un recurso compartido de red o una ruta de acceso de un recurso compartido de archivos de Azure montado cuando se especifica como una ruta de acceso de destino facilita el acceso a estos archivos de parte de otras máquinas en la red o en el mismo recurso compartido de archivos de Azure montado en ellas.

>[!NOTE]
>Para restaurar los archivos de copia de seguridad de base de datos en un recurso compartido de archivos de Azure montado en la VM registrada de destino, asegúrese de que la cuenta raíz tenga accesos de lectura o escritura en el recurso compartido de archivos de Azure.

En función del tipo de punto de restauración elegido (**Momento dado** o **Completa y diferencial**), se crearán una o varias carpetas en la ruta de acceso de destino. Una de las carpetas, denominada `Data_<date and time of restore>`, contiene las copias de seguridad completas y diferenciales y, la otra, llamada `Log`, contiene las copias de seguridad de registros.

Mueva estos archivos restaurados al servidor de SAP HANA en el que quiere restaurarlos como base de datos. A continuación, siga estos pasos para restaurar la base de datos:

1. Establezca permisos en la carpeta o el directorio en que se almacenan los archivos de copia de seguridad mediante el comando siguiente:

    ```bash
    chown -R <SID>adm:sapsys <directory>
    ```

1. Ejecute el siguiente conjunto de comandos como `<SID>adm`.

    ```bash
    su - <sid>adm
    ```

1. Genere el archivo de catálogo para la restauración. Extraiga **BackupId** del archivo de metadatos JSON para la copia de seguridad completa, que se usará más adelante en la operación de restauración. Asegúrese de que las copias de seguridad completas y de registros se encuentran en carpetas diferentes y elimine los archivos de catálogo y los archivos de metadatos JSON de estas carpetas.

    ```bash
    hdbbackupdiag --generate --dataDir <DataFileDir> --logDirs <LogFilesDir> -d <PathToPlaceCatalogFile>
    ```

    En el comando anterior:

    * `<DataFileDir>`: carpeta que contiene las copias de seguridad completas
    * `<LogFilesDir>`: carpeta que contiene las copias de seguridad de registros
    * `<PathToPlaceCatalogFile>`: carpeta donde se debe colocar el archivo de catálogo generado

1. Realice la restauración con el archivo de catálogo recién generado mediante HANA Studio o ejecute la consulta de restauración HDBSQL con este catálogo recién generado. Las consultas HDBSQL se enumeran a continuación:

    * Para restaurar a un momento dado:

        Si está creando una nueva base de datos restaurada, ejecute el comando HDBSQL para crear una nueva base de datos `<DatabaseName>` y, a continuación, detenga la base de datos para la restauración. Sin embargo, si solo está restaurando una base de datos existente, ejecute el comando HDBSQL para detenerla.

        A continuación, ejecute el siguiente comando para restaurar la base de datos:

        ```hdbsql
        RECOVER DATABASE FOR <DatabaseName> UNTIL TIMESTAMP '<TimeStamp>' CLEAR LOG USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING LOG PATH (' <LogFileDir>') USING DATA PATH ('<DataFileDir>') USING BACKUP_ID <BackupIdFromJsonFile> CHECK ACCESS USING FILE
        ```

        * `<DatabaseName>`: nombre de la base de datos nueva o existente que quiere restaurar
        * `<Timestamp>`: marca de tiempo exacta de la restauración a un momento dado
        * `<DatabaseName@HostName>`: nombre de la base de datos cuya copia de seguridad se usa para la restauración y nombre del servidor de SAP HANA o **host** en que reside esta base de datos. La opción `USING SOURCE <DatabaseName@HostName>` especifica que la copia de seguridad de datos (usada para la restauración) es de una base de datos con un SID o nombre diferente al de la máquina de SAP HANA de destino. Por lo tanto, no es necesario especificarla para las restauraciones realizadas en el mismo servidor de HANA del que se toma la copia de seguridad.
        * `<PathToGeneratedCatalogInStep3>`: ruta de acceso al archivo de catálogo generado en el **paso 3**
        * `<DataFileDir>`: carpeta que contiene las copias de seguridad completas
        * `<LogFilesDir>`: carpeta que contiene las copias de seguridad de registros
        * `<BackupIdFromJsonFile>`: **BackupId** extraído en el **paso 3**

    * Para restaurar una copia de seguridad completa o diferencial determinada:

        Si está creando una nueva base de datos restaurada, ejecute el comando HDBSQL para crear una nueva base de datos `<DatabaseName>` y, a continuación, detenga la base de datos para la restauración. Sin embargo, si solo está restaurando una base de datos existente, ejecute el comando HDBSQL para detenerla:

        ```hdbsql
        RECOVER DATA FOR <DatabaseName> USING BACKUP_ID <BackupIdFromJsonFile> USING SOURCE '<DatabaseName@HostName>'  USING CATALOG PATH ('<PathToGeneratedCatalogInStep3>') USING DATA PATH ('<DataFileDir>')  CLEAR LOG
        ```

        * `<DatabaseName>`: nombre de la base de datos nueva o existente que quiere restaurar
        * `<Timestamp>`: marca de tiempo exacta de la restauración a un momento dado
        * `<DatabaseName@HostName>`: nombre de la base de datos cuya copia de seguridad se usa para la restauración y nombre del servidor de SAP HANA o **host** en que reside esta base de datos. La opción `USING SOURCE <DatabaseName@HostName>` especifica que la copia de seguridad de datos (usada para la restauración) es de una base de datos con un SID o nombre diferente al de la máquina de SAP HANA de destino. Por lo tanto, no es necesario especificarla para las restauraciones realizadas en el mismo servidor de HANA del que se toma la copia de seguridad.
        * `<PathToGeneratedCatalogInStep3>`: ruta de acceso al archivo de catálogo generado en el **paso 3**
        * `<DataFileDir>`: carpeta que contiene las copias de seguridad completas
        * `<LogFilesDir>`: carpeta que contiene las copias de seguridad de registros
        * `<BackupIdFromJsonFile>`: **BackupId** extraído en el **paso 3**

## <a name="next-steps"></a>Pasos siguientes

* Para aprender a administrar las bases de datos de SAP HANA con copia de seguridad realizada mediante la CLI de Azure, continúe con el tutorial: [Administración de una base de datos de SAP HANA en una máquina virtual de Azure con la CLI](tutorial-sap-hana-backup-cli.md)

* Para aprender a restaurar una base de datos de SAP HANA que se ejecuta en una máquina virtual de Azure mediante Azure Portal, consulte [Restauración de bases de datos de SAP HANA en máquinas virtuales de Azure](./sap-hana-db-restore.md)
