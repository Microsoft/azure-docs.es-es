---
title: Solución de problemas de copia de seguridad de bases de datos de SQL Server
description: Información para solución de problemas para realizar copias de seguridad de bases de datos de SQL Server que se ejecutan en máquinas virtuales de Azure con Azure Backup.
ms.topic: troubleshooting
ms.date: 06/18/2019
ms.openlocfilehash: 53b701e5bfae9313732f4b76a4e13b63afb3864a
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88826725"
---
# <a name="troubleshoot-sql-server-database-backup-by-using-azure-backup"></a>Solución de problemas de la copia de seguridad de base de datos de SQL Server con Azure Backup

En este artículo se proporciona información para la solución de problemas de las bases de datos de SQL Server que se ejecutan en máquinas virtuales de Azure.

Para más información sobre el proceso y las limitaciones de las copias de seguridad, consulte [Acerca de la copia de seguridad de SQL Server en máquinas virtuales de Azure](sql-support-matrix.md#feature-considerations-and-limitations).

## <a name="sql-server-permissions"></a>Permisos de SQL Server

Para configurar la protección de una base de datos de SQL Server en una máquina virtual, debe instalar en ella la extensión **AzureBackupWindowsWorkload**. Si aparece el error **UserErrorSQLNoSysadminMembership**, significa que la instancia de SQL Server no tiene los permisos de copia de seguridad requeridos. Para corregir este error, siga los pasos descritos en [Establecer permisos de máquina virtual](backup-azure-sql-database.md#set-vm-permissions).

## <a name="troubleshoot-discover-and-configure-issues"></a>Solución de problemas de detección y configuración

Después de crear y configurar un almacén de Recovery Services, la detección de bases de datos y la configuración de la copia de seguridad es un proceso de dos pasos.<br>

![Objetivo de Backup: SQL Server en máquinas virtuales de Azure](./media/backup-azure-sql-database/sql.png)

Durante la configuración de copia de seguridad, si la máquina virtual de SQL y sus instancias no están visibles en **Detección de bases de datos en máquinas virtuales** y **Configurar copia de seguridad** (consulte la imagen anterior), asegúrese de lo siguiente:

### <a name="step-1-discovery-dbs-in-vms"></a>Paso 1: Detección de bases de datos en máquinas virtuales

- Si la máquina virtual no aparece en la lista de máquinas virtuales detectadas ni tampoco está registrada como copia de seguridad de SQL en otro almacén, siga los pasos de [Detección de copia de seguridad de SQL Server](./backup-sql-server-database-azure-vms.md#discover-sql-server-databases).

### <a name="step-2-configure-backup"></a>Paso 2: Configurar la copia de seguridad

- Si el almacén en el que se registra la máquina virtual de SQL en el mismo almacén que se usa para proteger las bases de datos, siga los pasos de [Configuración de la copia de seguridad](./backup-sql-server-database-azure-vms.md#configure-backup).

Si hay que registrar la máquina virtual de SQL en el nuevo almacén, se debe anular el registro del almacén anterior.  La anulación del registro de una máquina virtual de SQL del almacén requiere que se detenga la protección de todos los orígenes de datos protegidos y, después, se pueden eliminar los datos cuya copia de seguridad se ha realizado. La eliminación de datos de copia de seguridad es una operación destructiva.  Una vez que haya revisado y tomado todas las precauciones para anular el registro de la máquina virtual de SQL, registre la misma máquina virtual en un nuevo almacén y vuelva a intentar la operación de copia de seguridad.

## <a name="troubleshoot-backup-and-recovery-issues"></a>Solución de incidencias de copia de seguridad y recuperación  

En ocasiones, es posible que se produzcan errores aleatorios en las operaciones de restauración y copia de seguridad, o dichas operaciones pueden dejar de responder. Los programas antivirus en la máquina virtual pueden ser la causa de ello. Como procedimiento recomendado, se sugieren los siguientes pasos:

1. Excluya las siguientes carpetas del examen del antivirus:

    `C:\Program Files\Azure Workload Backup` `C:\WindowsAzure\Logs\Plugins\Microsoft.Azure.RecoveryServices.WorkloadBackup.Edp.AzureBackupWindowsWorkload`

    Reemplace `C:\` por la letra de su unidad *SystemDrive*.

1. Excluya del examen del antivirus los siguientes tres procesos en ejecución de la máquina virtual:

    - IaasWLPluginSvc.exe
    - IaasWorkloadCoordinaorService.exe
    - TriggerExtensionJob.exe

1. SQL también ofrece algunas instrucciones sobre cómo trabajar con programas antivirus. Consulte [este artículo](https://support.microsoft.com/help/309422/choosing-antivirus-software-for-computers-that-run-sql-server) para más información.

## <a name="error-messages"></a>Mensajes de error

### <a name="backup-type-unsupported"></a>Tipo de copia de seguridad no compatible

| severity | Descripción | Causas posibles | Acción recomendada |
|---|---|---|---|
| Advertencia | La configuración actual de esta base de datos no admite determinados tipos de copia de seguridad presentes en la directiva asociada. | <li>Solo se puede realizar una operación de copia de seguridad de bases de datos completa en la base de datos maestra. No es posible realizar una copia de seguridad diferencial y una copia de seguridad del registro de transacciones. </li> <li>Ninguna base de datos del modelo de recuperación simple admite la copia de seguridad de registros de transacciones.</li> | Modifique la configuración de la base de datos para que se admitan todos los tipos de copia de seguridad de la directiva. O bien cambie la directiva actual para incluir solo los tipos de copia de seguridad compatibles. De lo contrario, se omitirán los tipos de copia de seguridad no compatibles durante la copia de seguridad programada o el trabajo de copia de seguridad generará errores en caso de que se trate de una copia de seguridad a petición.

### <a name="usererrorsqlpodoesnotsupportbackuptype"></a>UserErrorSQLPODoesNotSupportBackupType

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| Esta base de datos SQL no admite el tipo de copia de seguridad solicitado. | Se produce cuando el modelo de recuperación de base de datos no admite el tipo de copia de seguridad solicitado. El error puede ocurrir en las siguientes situaciones: <br/><ul><li>Una base de datos que usa un modelo de recuperación simple no permite las copias de seguridad de registros.</li><li>No se permiten las copias de seguridad diferenciales ni de registros para las bases de datos maestras.</li></ul>Para más información, consulte el documento [Modelos de recuperación (SQL Server)](/sql/relational-databases/backup-restore/recovery-models-sql-server). | Si se produce un error en la copia de seguridad de registros de la base de datos en el modelo de recuperación simple, pruebe una de estas opciones:<ul><li>Si la base de datos está en modo de recuperación simple, deshabilite las copias de seguridad de registros.</li><li>Use la [documentación de SQL Server](/sql/relational-databases/backup-restore/view-or-change-the-recovery-model-of-a-database-sql-server) para cambiar el modelo de recuperación de base de datos a completo o registro masivo. </li><li> Si no desea cambiar el modelo de recuperación y tiene una directiva estándar para realizar copias de seguridad de varias bases de datos que no se puede cambiar, ignore el error. Las copias de seguridad completas y diferenciales funcionarán en función de la programación. Se omitirán las copias de seguridad del registro, que es lo esperable en este caso.</li></ul>Si es una base de datos maestra y ha configurado una base de datos diferencial o de registros, siga cualquiera de estos pasos:<ul><li>Use el portal para cambiar la programación de la directiva de copia de seguridad para la base de datos maestra a completa.</li><li>Si tiene una directiva estándar para realizar copias de seguridad de varias bases de datos que no se puede cambiar, ignore el error. La copia de seguridad completa funcionará según la programación. No se realizarán copias de seguridad diferenciales o de registro, que es lo que se espera en este caso.</li></ul> |
| La operación se canceló, porque ya se estaba ejecutando una operación en conflicto en la misma base de datos. | Consulte la [entrada del blog acerca de las limitaciones en las operaciones de copia de seguridad y restauración](https://deep.data.blog/2008/12/30/concurrency-of-full-differential-and-log-backups-on-the-same-database/) que se ejecutan de forma concurrente.| [Utilice SQL Server Management Studio (SSMS) para supervisar los trabajos de la base de datos](manage-monitor-sql-database-backup.md). Una vez que se produzca un error en la operación en conflicto, reinicie la operación.|

### <a name="usererrorsqlpodoesnotexist"></a>UserErrorSQLPODoesNotExist

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| La base de datos SQL no existe. | La base de datos se eliminó o se cambió de nombre. | Compruebe si la base de datos se ha eliminado o cambiado de nombre por accidente.<br/><br/> Si la base de datos se eliminó por accidente, para continuar realizando copias de seguridad, restaure la base de datos a la ubicación original.<br/><br/> Si ha eliminado la base de datos y no va a necesitar copias de seguridad en el futuro, en el almacén de Recovery Services, seleccione **Detener copia de seguridad** con **Retener datos de copia de seguridad** o **Eliminar datos de la copia de seguridad**. Para más información, consulte [Administración y supervisión de bases de datos SQL Server con copia de seguridad](manage-monitor-sql-database-backup.md).

### <a name="usererrorsqllsnvalidationfailure"></a>UserErrorSQLLSNValidationFailure

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| La cadena de registros se ha interrumpido. | Se realiza una copia de seguridad de la base de datos o de la máquina virtual a través de otra solución de copia de seguridad, lo que trunca la cadena de registros.|<ul><li>Compruebe si hay otra solución de copia de seguridad o script en uso. En ese caso, detenga la otra solución de copia de seguridad. </li><li>Si la copia de seguridad fue una copia de seguridad de registros a petición, desencadene una copia de seguridad completa para iniciar una nueva cadena de registros. Para las copias de seguridad de registros, no es preciso realizar ninguna acción, ya que el servicio Azure Backup desencadenará automáticamente una copia de seguridad completa para corregir este problema.</li>|

### <a name="usererroropeningsqlconnection"></a>UserErrorOpeningSQLConnection

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| Azure Backup no puede conectarse a la instancia de SQL. | Azure Backup no puede conectarse a la instancia de SQL Server. | Utilice los detalles adicionales del menú del error de Azure Portal para reducir las causas principales. Para solucionar el error, consulte [Solución de problemas de copia de seguridad de SQL](/sql/database-engine/configure-windows/troubleshoot-connecting-to-the-sql-server-database-engine).<br/><ul><li>Si la configuración predeterminada de SQL no permite conexiones remotas, cámbiela. Consulte los siguientes artículos para más información sobre cómo cambiar la configuración:<ul><li>[MSSQLSERVER_-1](/sql/relational-databases/errors-events/mssqlserver-1-database-engine-error)</li><li>[MSSQLSERVER_2](/sql/relational-databases/errors-events/mssqlserver-2-database-engine-error)</li><li>[MSSQLSERVER_53](/sql/relational-databases/errors-events/mssqlserver-53-database-engine-error)</li></ul></li></ul><ul><li>Si hay problemas de inicio de sesión, use estos vínculos para corregirlos:<ul><li>[MSSQLSERVER_18456](/sql/relational-databases/errors-events/mssqlserver-18456-database-engine-error)</li><li>[MSSQLSERVER_18452](/sql/relational-databases/errors-events/mssqlserver-18452-database-engine-error)</li></ul></li></ul> |

### <a name="usererrorparentfullbackupmissing"></a>UserErrorParentFullBackupMissing

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| Falta la primera copia de seguridad completa de este origen de datos. | Falta la copia de seguridad completa de la base de datos. Las copias de seguridad diferenciales y de registros son elementos principales de una copia de seguridad completa, por lo tanto, asegúrese de realizar copias de seguridad completas antes de desencadenar copias de seguridad diferenciales o de registros. | Desencadenar una copia de seguridad a petición.   |

### <a name="usererrorbackupfailedastransactionlogisfull"></a>UserErrorBackupFailedAsTransactionLogIsFull

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| No se puede realizar una copia de seguridad porque el registro de transacciones del origen de datos está lleno. | El espacio del registro de transacciones de la base de datos está lleno. | Para corregir este problema, consulte la [documentación de SQL Server](/sql/relational-databases/errors-events/mssqlserver-9002-database-engine-error). |

### <a name="usererrorcannotrestoreexistingdbwithoutforceoverwrite"></a>UserErrorCannotRestoreExistingDBWithoutForceOverwrite

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| Ya existe una base de datos con el mismo nombre en la ubicación de destino | El destino de la restauración ya tiene una base de datos con el mismo nombre.  | <ul><li>Cambie el nombre de la base de datos de destino.</li><li>O bien, use la opción de forzar la sobrescritura en la página de restauración.</li> |

### <a name="usererrorrestorefaileddatabasecannotbeofflined"></a>UserErrorRestoreFailedDatabaseCannotBeOfflined

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| Se produjo un error en la restauración, ya que no se pudo desconectar. | Al realizar una restauración, la base de datos de destino debe desconectarse. Azure Backup no puede ofrecer estos datos sin conexión. | Utilice los detalles adicionales del menú del error de Azure Portal para reducir las causas principales. Para más información, consulte la [documentación de SQL Server](/sql/relational-databases/backup-restore/restore-a-database-backup-using-ssms). |

### <a name="wlextgenericiofaultusererror"></a>WlExtGenericIOFaultUserError

|Mensaje de error |Causas posibles  |Acción recomendada  |
|---------|---------|---------|
|Error de entrada/salida durante la operación. Compruebe los errores comunes de E/S en la máquina virtual.   |   Permisos de acceso o restricciones de espacio en el destino.       |  Compruebe los errores comunes de E/S en la máquina virtual. Asegúrese de que la unidad de disco o recurso compartido de red de destino de la máquina: <li> tiene permiso de lectura/escritura para la cuenta NT AUTHORITY\SYSTEM en el equipo. <li> tiene suficiente espacio para que la operación se complete correctamente.<br> Para más información, consulte [Restaurar como archivos](restore-sql-database-azure-vm.md#restore-as-files).
       |

### <a name="usererrorcannotfindservercertificatewiththumbprint"></a>UserErrorCannotFindServerCertificateWithThumbprint

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| No se puede encontrar el certificado de servidor con la huella digital en el destino. | La base de datos maestra de la instancia de destino no tiene una huella digital de cifrado válida. | Importe en la instancia de destino la huella digital del certificado válida utilizada en la instancia de origen. |

### <a name="usererrorrestorenotpossiblebecauselogbackupcontainsbulkloggedchanges"></a>UserErrorRestoreNotPossibleBecauseLogBackupContainsBulkLoggedChanges

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| La copia de seguridad de registros usada para la recuperación contiene cambios registrados de forma masiva. No se puede usar para detenerse en un punto arbitrario en el tiempo según las instrucciones de SQL. | Cuando una base de datos está en modo de recuperación de registro masivo, no se pueden recuperar los datos entre una transacción registrada de forma masiva y la siguiente transacción del registro. | Elija un punto diferente en el tiempo para la recuperación. [Más información](/sql/relational-databases/backup-restore/recovery-models-sql-server?view=sql-server-ver15).

### <a name="fabricsvcbackuppreferencecheckfailedusererror"></a>FabricSvcBackupPreferenceCheckFailedUserError

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| No puede cumplir la preferencia de copia de seguridad del grupo de disponibilidad Always On de SQL, ya que algunos nodos del grupo de disponibilidad no están registrados. | Los nodos necesarios para realizar copias de seguridad no están registrados o no se puede acceder a ellos. | <ul><li>Asegúrese de que todos los nodos necesarios para realizar copias de seguridad de esta base de datos están registrados y en buen estado, y vuelva a intentar realizar la operación.</li><li>Cambie la preferencia de copia de seguridad para el grupo de disponibilidad Always On de SQL Server.</li></ul> |

### <a name="vmnotinrunningstateusererror"></a>VMNotInRunningStateUserError

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| La máquina virtual con SQL Server está apagada o no se puede acceder al servicio Azure Backup. | La máquina virtual está apagada. | Asegúrese de que la instancia de SQL Server se está ejecutando. |

### <a name="guestagentstatusunavailableusererror"></a>GuestAgentStatusUnavailableUserError

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| El servicio Azure Backup usa el agente invitado de la máquina virtual de Azure para realizar la copia de seguridad, pero dicho agente no está disponible en el servidor de destino. | El agente invitado no está habilitado o es incorrecto. | [Instale el agente invitado de la máquina virtual](../virtual-machines/extensions/agent-windows.md) manualmente. |

### <a name="autoprotectioncancelledornotvalid"></a>AutoProtectionCancelledOrNotValid

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
| La intención de la protección automática se ha quitado o ya no es válida. | Cuando se habilita la protección automática en una instancia de SQL Server, se ejecutan los trabajos de **Configurar copia de seguridad**  para todas las bases de datos de esa instancia. Si se deshabilita la protección automática mientras se ejecutan los trabajos, los trabajos **en curso** se cancelan con este código de error. | Habilite la protección automática una vez más para ayudar a proteger todas las bases de datos restantes. |

### <a name="clouddosabsolutelimitreached"></a>CloudDosAbsoluteLimitReached

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
La operación se bloquea porque ha alcanzado el límite en el número de operaciones permitidas en 24 horas. | Este error aparecerá cuando haya alcanzado el límite máximo permitido para una operación en un intervalo de 24 horas. <br> Por ejemplo: Si ha alcanzado el límite para el número de trabajos de configuración de copia de seguridad que se pueden desencadenar al día e intenta configurar la copia de seguridad de un nuevo elemento, verá este error. | Normalmente, si se vuelve a intentar la operación después de 24 horas, se resuelve este problema. Sin embargo, si el problema persiste, puede ponerse en contacto con el equipo de soporte técnico de Microsoft para obtener ayuda.

### <a name="clouddosabsolutelimitreachedwithretry"></a>CloudDosAbsoluteLimitReachedWithRetry

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
La operación está bloqueada porque el almacén ha alcanzado su límite máximo permitido para ese tipo de operaciones en un intervalo de 24 horas. | Este error aparecerá cuando haya alcanzado el límite máximo permitido para una operación en un intervalo de 24 horas. Normalmente, este error aparece cuando hay operaciones a escala, como la modificación de una directiva o la protección automática. A diferencia de lo que sucede en el caso de CloudDosAbsoluteLimitReached, no se puede hacer mucho para resolver este estado. De hecho, el servicio de Azure Backup volverá a intentar internamente las operaciones para todos los elementos en cuestión.<br> Por ejemplo: si tiene un gran número de orígenes de datos protegidos con una directiva e intenta modificar esa directiva, se desencadenará la configuración de los trabajos de protección para cada uno de los elementos protegidos y, en ocasiones, puede alcanzar el límite máximo permitido para tales operaciones al día.| El servicio Azure Backup volverá a intentar esta operación automáticamente después de 24 horas.

### <a name="usererrorvminternetconnectivityissue"></a>UserErrorVMInternetConnectivityIssue

| Mensaje de error | Causas posibles | Acción recomendada |
|---|---|---|
La máquina virtual no es capaz de ponerse en contacto con el servicio Azure Backup debido a problemas de conectividad de Internet. | La máquina virtual necesita conectividad de salida con el servicio Azure Backup y los servicios Azure Storage o Azure Active Directory.| - Si usa NSG para restringir la conectividad, debe usar la etiqueta de servicio AzureBackup para permitir el acceso saliente al servicio de Azure Backup o los servicios Azure Storage y Azure Active Directory. Siga estos [pasos](./backup-sql-server-database-azure-vms.md#nsg-tags) para conceder acceso.<br>- Asegúrese de que DNS está resolviendo los puntos de conexión de Azure.<br>- Compruebe si la máquina virtual está detrás de un equilibrador de carga que bloquea el acceso a Internet. Mediante la asignación de una dirección IP pública a las máquinas virtuales, la detección funcionará.<br>- Compruebe que no hay ningún firewall/antivirus/proxy que esté bloqueando las llamadas a los tres servicios de destino anteriores.

## <a name="re-registration-failures"></a>Errores de repetición del registro

Compruebe uno o varios de los siguientes síntomas antes de desencadenar la operación de repetición del registro:

- Se producen errores en todas las operaciones (tales como copia de seguridad, restauración y configuración de copia de seguridad) en la máquina virtual con uno de los códigos de error siguientes: **WorkloadExtensionNotReachable**, **UserErrorWorkloadExtensionNotInstalled**, **WorkloadExtensionNotPresent**, **WorkloadExtensionDidntDequeueMsg**.
- Si el área **Estado de copia de seguridad** del elemento de copia de seguridad se muestra como **no accesible**, descarte todas las demás causas que podrían dar lugar al mismo estado:

  - Falta de permiso para realizar operaciones relacionadas con la copia de seguridad en la máquina virtual.
  - Apagado de la máquina virtual, por lo que no se pueden realizar copias de seguridad.
  - Problemas de red.

   ![Repetición del registro de máquinas virtuales](./media/backup-azure-sql-database/re-register-vm.png)

- En el caso de un grupo de disponibilidad AlwaysOn, las copias de seguridad comienzan a generar errores después de cambiar la preferencia de copia de seguridad o alterar una conmutación por error.

Estos síntomas pueden surgir por uno o varios de los siguientes motivos:

- Se ha eliminado o desinstalado una extensión del portal.
- Se ha desinstalado una extensión del **Panel de control** de la máquina virtual en **Desinstalar o cambiar este programa**.
- Se ha restaurado la máquina virtual retrocediendo en el tiempo a través de la restauración del disco o discos en contexto.
- Se ha apagado la máquina virtual durante un largo período, por lo que la configuración de la extensión en ella ha caducado.
- Se ha eliminado la máquina virtual, y se ha creado otra con el mismo nombre y en el mismo grupo de recursos que la máquina virtual eliminada.
- Uno de los nodos del grupo de disponibilidad no recibió la configuración de copia de seguridad completa. Esto puede ocurrir cuando el grupo de disponibilidad se registra en el almacén o cuando se agrega un nuevo nodo.

En los escenarios anteriores, se recomienda desencadenar una operación de nuevo registro en la máquina virtual. [Aquí](./backup-azure-sql-automation.md#enable-backup) se ofrecen instrucciones sobre cómo realizar esta tarea en PowerShell.

## <a name="size-limit-for-files"></a>Límite de tamaño para los archivos

El tamaño total de la cadena de archivos no solo depende del número de archivos, sino también de sus nombres y rutas de acceso. Para cada archivo de base de datos, obtenga el nombre de archivo lógico y la ruta de acceso física. Puede usar esta consulta SQL:

```sql
SELECT mf.name AS LogicalName, Physical_Name AS Location FROM sys.master_files mf
               INNER JOIN sys.databases db ON db.database_id = mf.database_id
               WHERE db.name = N'<Database Name>'"
```

Ahora organícelos en el siguiente formato:

```json
[{"path":"<Location>","logicalName":"<LogicalName>","isDir":false},{"path":"<Location>","logicalName":"<LogicalName>","isDir":false}]}
```

Este es un ejemplo:

```json
[{"path":"F:\\Data\\TestDB12.mdf","logicalName":"TestDB12","isDir":false},{"path":"F:\\Log\\TestDB12_log.ldf","logicalName":"TestDB12_log","isDir":false}]}
```

Si el tamaño de la cadena del contenido supera los 20 000 bytes, los archivos de base de datos se almacenarán de forma diferente. Durante la recuperación, no podrá establecer la ruta de acceso del archivo de destino para la restauración. Los archivos se restaurarán en la ruta de acceso predeterminada de SQL proporcionada por SQL Server.

### <a name="override-the-default-target-restore-file-path"></a>Reemplazo de la ruta de acceso predeterminada del archivo de restauración de destino

Puede reemplazar la ruta de acceso del archivo de restauración de destino durante la operación de restauración mediante la colocación de un archivo JSON que contenga la asignación del archivo de base de datos a la ruta de acceso de restauración de destino. Cree un archivo `database_name.json` y colóquelo en la ubicación `C:\Program Files\Azure Workload Backup\bin\plugins\SQL*`.

El contenido del archivo debe tener este formato:

```json
[
  {
    "Path": "<Restore_Path>",
    "LogicalName": "<LogicalName>",
    "IsDir": "false"
  },
  {
    "Path": "<Restore_Path>",
    "LogicalName": "LogicalName",
    "IsDir": "false"
  },  
]
```

Este es un ejemplo:

```json
[
  {
   "Path": "F:\\Data\\testdb2_1546408741449456.mdf",
   "LogicalName": "testdb7",
   "IsDir": "false"
  },
  {
    "Path": "F:\\Log\\testdb2_log_1546408741449456.ldf",
    "LogicalName": "testdb7_log",
    "IsDir": "false"
  },  
]
```

En el contenido anterior puede obtener el nombre lógico del archivo de base de datos mediante la siguiente consulta SQL:

```sql
SELECT mf.name AS LogicalName FROM sys.master_files mf
                INNER JOIN sys.databases db ON db.database_id = mf.database_id
                WHERE db.name = N'<Database Name>'"
```

Este archivo se debe colocar antes de desencadenar la operación de restauración.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Azure Backup para las máquinas virtuales con SQL Server (versión preliminar pública), consulte [Azure Backup para máquinas virtuales SQL](../azure-sql/virtual-machines/windows/backup-restore.md#azbackup).
