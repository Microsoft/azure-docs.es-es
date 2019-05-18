---
title: 'Azure Backup: Copia de seguridad y restaurar bases de datos de SQL en Azure Virtual Machines con Azure Backup y PowerShell'
description: Copia de seguridad y restaurar bases de datos de SQL en Azure Virtual Machines con Azure Backup y PowerShell.
services: backup
author: pvrk
manager: vijayts
keywords: Azure Backup; SQL;
ms.service: backup
ms.topic: conceptual
ms.date: 03/15/2019
ms.author: pullabhk
ms.assetid: 57854626-91f9-4677-b6a2-5d12b6a866e1
ms.openlocfilehash: 6d17d5c2c0eaebc694abe820318f6ac0c70b0be8
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544609"
---
# <a name="back-up-and-restore-sql-databases-in-azure--vms-with-powershell"></a>Copia de seguridad y restaurar bases de datos de SQL en máquinas virtuales de Azure con PowerShell

En este artículo se describe cómo usar Azure PowerShell para realizar copias de seguridad y recuperar una base de datos SQL dentro de una máquina virtual de Azure mediante [Azure Backup](backup-overview.md) almacén de Recovery Services.

Este tutorial explica cómo realizar lo siguiente:

> [!div class="checklist"]
> * Configuración de PowerShell y registrar el proveedor de Azure Recovery Services.
> * Cree un almacén de Recovery Services.
> * Configurar copia de seguridad de base de datos SQL dentro de una máquina virtual de Azure.
> * Ejecutar un trabajo de copia de seguridad.
> * Restaurar una copia de seguridad de base de datos SQL.
> * Supervisar copias de seguridad y restaurar trabajos.

## <a name="before-you-start"></a>Antes de comenzar

* [Obtenga más información](backup-azure-recovery-services-vault-overview.md) sobre almacenes de Recovery Services.
* Obtenga información sobre las capacidades de característica para [la copia de seguridad de bases de datos SQL dentro de máquinas virtuales de Azure](backup-azure-sql-database.md#before-you-start).
* Revise la jerarquía de objetos de PowerShell para servicios de recuperación.

### <a name="recovery-services-object-hierarchy"></a>Jerarquía de objetos de Recovery Services

En el diagrama siguiente, se resume la jerarquía de objetos.

![Jerarquía de objetos de Recovery Services](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

Revise el **Az.RecoveryServices** [referencia de cmdlet](/powershell/module/az.recoveryservices) referencia en la biblioteca de Azure.

### <a name="set-up-and-install"></a>Configurar e instalar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Configuración de PowerShell como sigue:

1. [Descargue la versión más reciente de Azure PowerShell](/powershell/azure/install-az-ps). La versión mínima requerida es la versión 1.5.0.

2. Buscar los cmdlets de PowerShell de Azure Backup con este comando:

    ```powershell
    Get-Command *azrecoveryservices*
    ```

3. Revise los alias y cmdlets para copias de seguridad de Azure y el almacén de Recovery Services. Este es un ejemplo de lo que podría ver. No es una lista completa de los cmdlets.

    ![Lista de cmdlets de Recovery Services](./media/backup-azure-afs-automation/list-of-recoveryservices-ps-az.png)

4. Inicie sesión en su cuenta de Azure con **Connect AzAccount**.
5. En la página web que aparece, se le pedirá que escriba sus credenciales de cuenta.

    * Como alternativa, puede incluir sus credenciales de cuenta como un parámetro en el **Connect AzAccount** cmdlet con **-Credential**.
    * Si es un asociado CSP que trabaja para un inquilino, especifique al cliente como un inquilino, mediante su nombre de dominio principal tenantID o inquilino. Un ejemplo es **Connect-AzAccount -Tenant** fabrikam.com.

6. Asocie la suscripción que desea usar con la cuenta, ya que una cuenta puede tener varias suscripciones.

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

7. Si utiliza Azure Backup por primera vez, use el cmdlet **Register-AzResourceProvider** para registrar el proveedor de Azure Recovery Services con su suscripción.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

8. Compruebe que los proveedores se registraron correctamente:

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

9. En la salida del comando, compruebe que **RegistrationState** cambia a **registrado**. Si no, se ejecuta el **Register AzResourceProvider** cmdlet nuevo.

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Siga estos pasos para crear un almacén de Recovery Services.

El almacén de Recovery Services es un recurso de Resource Manager, por lo que deberá colocarlo dentro de un grupo de recursos. Puede usar un grupo de recursos existente o crear uno con el cmdlet **New-AzResourceGroup**. Al crear un grupo de recursos, especifique el nombre y la ubicación.

1. Un almacén se coloca en un grupo de recursos. Si no tiene un recurso existente de grupo, cree uno nuevo con el [New AzResourceGroup](https://docs.microsoft.com/powershell/module/az.resources/new-azresourcegroup?view=azps-1.4.0). En este ejemplo, crearemos un nuevo grupo de recursos en la región Oeste de Estados Unidos.

    ```powershell
    New-AzResourceGroup -Name "test-rg" -Location "West US"
    ```

2. Use la [New AzRecoveryServicesVault](https://docs.microsoft.com/powershell/module/az.recoveryservices/New-AzRecoveryServicesVault?view=azps-1.4.0) cmdlet para crear el almacén. Especifique para el almacén la misma ubicación del grupo de recursos.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

3. Especifique el tipo de redundancia que se usará para el almacenamiento del almacén.

    * Puede usar [almacenamiento con redundancia local](../storage/common/storage-redundancy-lrs.md) o [almacenamiento con redundancia geográfica](../storage/common/storage-redundancy-grs.md).
    * El ejemplo siguiente se establece la **- BackupStorageRedundancy** opción el[conjunto AzRecoveryServicesBackupProperty](https://docs.microsoft.com/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty) cmd para **testvault** establecido en  **Con redundancia geográfica**.

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault -Name "testvault"
    Set-AzRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
    ```

### <a name="view-the-vaults-in-a-subscription"></a>Visualización de los almacenes de una suscripción

Para ver todos los almacenes de la suscripción, use [Get-AzRecoveryServicesVault](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesvault?view=azps-1.4.0).

```powershell
Get-AzRecoveryServicesVault
```

El resultado es similar al siguiente. El grupo de recursos asociado y la ubicación se proporcionan.

```powershell
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

### <a name="set-the-vault-context"></a>Establecer el contexto de almacén

Store el objeto de almacén en una variable y establezca el contexto de almacén.

* Muchos de los cmdlets de Azure Backup requieren el objeto de almacén de Recovery Services como una entrada, por lo que es conveniente almacenar el objeto de almacén en una variable.
* El contexto de almacén es el tipo de los datos protegidos en el almacén. Establecerlo con [conjunto AzRecoveryServicesVaultContext](https://docs.microsoft.com/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext?view=azps-1.4.0). Después de establece el contexto, se aplica a todos los cmdlets posteriores.

En el ejemplo siguiente se establece el contexto del almacén de **testvault**.

```powershell
Get-AzRecoveryServicesVault -Name "testvault" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Obtener el identificador del almacén

Tenemos previsto dejar de usar el contexto de almacén configuración de acuerdo con las directrices de Azure PowerShell. En su lugar, puede almacenar o recuperar el identificador del almacén y pase a los comandos pertinentes, como sigue:

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
```

## <a name="configure-a-backup-policy"></a>Configuración de una directiva de copia de seguridad

Una directiva de copia de seguridad especifica la programación para copias de seguridad y cuánto tiempo los puntos de recuperación de copia de seguridad se deben mantener:

* Una directiva de copia de seguridad está asociada con al menos una directiva de retención. Una directiva de retención define el tiempo que se conserva un punto de recuperación antes de que se elimine.
* La retención de directiva de copia de seguridad predeterminada mediante la vista [Get AzRecoveryServicesBackupRetentionPolicyObject](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject?view=azps-1.4.0).
* La programación de directiva de copia de seguridad predeterminada mediante la vista [Get AzRecoveryServicesBackupSchedulePolicyObject](https://docs.microsoft.com/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject?view=azps-1.4.0).
* Usa el [New AzRecoveryServicesBackupProtectionPolicy](https://docs.microsoft.com/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy?view=azps-1.4.0) para crear una nueva directiva de copia de seguridad. Había entrada de los objetos de directiva de retención y programación.

En el ejemplo siguiente se almacenan la directiva de programación y la directiva de retención en variables. A continuación, usa estas variables como parámetros para una nueva directiva (**NewSQLPolicy**). **NewSQLPolicy** toma un diario "Full" copia de seguridad, conserva durante 180 días y toma una copia de seguridad del registro cada 2 horas

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "MSSQL"
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "MSSQL"
$NewSQLPolicy = New-AzRecoveryServicesBackupProtectionPolicy -Name "NewSQLPolicy" -WorkloadType "MSSQL" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

El resultado es similar al siguiente.

```powershell
Name                 WorkloadType       BackupManagementType BackupTime                Frequency                                IsDifferentialBackup IsLogBackupEnabled
                                                                                                                                Enabled
----                 ------------       -------------------- ----------                ---------                                -------------------- ------------------
NewSQLPolicy         MSSQL              AzureWorkload        3/15/2019 9:00:00 PM      Daily                                    False                True
```

## <a name="enable-backup"></a>Habilitación de la copia de seguridad

### <a name="registering-the-sql-vm"></a>Registrar la máquina virtual de SQL

Para las copias de seguridad de máquina virtual de Azure y recursos compartidos de archivos de Azure, servicio de copia de seguridad puede conectarse a estos recursos de Azure Resource Manager y capturar los detalles pertinentes. Puesto que SQL es una aplicación dentro de una máquina virtual de Azure, servicio de copia de seguridad necesita permiso para acceder a la aplicación y capturar los detalles necesarios. Para ello, deberá *'registro'* la máquina virtual de Azure que contiene la aplicación de SQL con un almacén de Recovery services. Una vez que registra una VM de SQL con un almacén, puede proteger las bases de datos SQL en dicho almacén solo. Use [Register AzRecoveryServicesBackupContainer](https://docs.microsoft.com/powershell/module/az.recoveryservices/Register-AzRecoveryServicesBackupContainer?view=azps-1.5.0) cmdlet PS para registrar la máquina virtual.

````powershell
 $myVM = Get-AzVM -ResourceGroupName <VMRG Name> -Name <VMName>
Register-AzRecoveryServicesBackupContainer -ResourceId $myVM.ID -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID -Force
````

El comando devolverá un "contenedor de copia de seguridad' de este recurso y el estado se 'registra'

> [!NOTE]
> Si no se especifica el parámetro force, usuario se le pide que confirme con un texto '¿desea deshabilitar la protección de este contenedor'. Por favor, ignore este texto y diga "Y" para confirmar. Se trata de un problema conocido y estamos trabajando para quitar el texto y los requisitos para el parámetro force

### <a name="fetching-sql-dbs"></a>Obtención de bases de datos SQL

Una vez que se realiza el registro, el servicio de copia de seguridad podrá mostrar todos los componentes SQL disponibles en la máquina virtual. Para ver todos los componentes SQL aún una copia de seguridad para este almacén, use [Get AzRecoveryServicesBackupProtectableItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupProtectableItem?view=azps-1.5.0) cmdlet de PS

````powershell
Get-AzRecoveryServicesBackupProtectableItem -WorkloadType MSSQL -VaultId $targetVault.ID
````

La salida mostrará todos los componentes SQL no protegidos en todas las máquinas virtuales de SQL registrado en este almacén con el tipo de elemento y ServerName. Puede filtrar aún más a una VM de SQL determinada pasando el '-contenedor ' parámetro o use la combinación de 'Name' y 'ServerName' junto con el tipo de elemento se marca para llegar a un único elemento de SQL.

````powershell
$SQLDB = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLDataBase -VaultId $targetVault.ID -Name "<Item Name>" -ServerName "<Server Name>"
````

### <a name="configuring-backup"></a>Configuración de copia de seguridad

Ahora que tenemos la base de datos de SQL necesarios y la directiva con la TI necesita una copia de seguridad, podemos utilizar el [Enable AzRecoveryServicesBackupProtection](https://docs.microsoft.com/powershell/module/az.recoveryservices/Enable-AzRecoveryServicesBackupProtection?view=azps-1.5.0) cmdlet para configurar la copia de seguridad para esta base de datos de SQL.

````powershell
Enable-AzRecoveryServicesBackupProtection -ProtectableItem $SQLDB -Policy $NewSQLPolicy
````

El comando espera hasta que la copia de seguridad de la configuración se ha completado y devuelve el resultado siguiente.

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
master           ConfigureBackup      Completed            3/18/2019 6:00:21 PM      3/18/2019 6:01:35 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="fetching-new-sql-dbs"></a>Obtención de nuevas bases de datos de SQL

Una vez registrada la máquina, el servicio de copia de seguridad capturará los detalles de las bases de datos disponibles, a continuación. Si el usuario agrega las instancias SQL/bases de datos de SQL a la máquina registrada más adelante, es necesario desencadenar manualmente el servicio de copia de seguridad para llevar a cabo una nueva "consulta" para obtener todas las bases de datos no protegidos (incluidos los que acaba de agregar) nuevo. Use la [Initialize AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/Initialize-AzRecoveryServicesBackupProtectableItem?view=azps-1.5.0) cmdlet PS en la VM de SQL para realizar una consulta nueva. El comando espera hasta que se complete la operación. Usar más adelante el [Get AzRecoveryServicesBackupProtectableItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupProtectableItem?view=azps-1.5.0) cmdlet PS para obtener la lista de componentes SQL más recientes sin protección

````powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
Initialize-AzRecoveryServicesBackupProtectableItem -Container $SQLContainer -WorkloadType MSSQL -VaultId $targetvault.ID
Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLDataBase -VaultId $targetVault.ID
````

Una vez que se capturan los elementos relevantes que se pueden proteger, habilite las copias de seguridad como se indica en la [por encima de la sección](#configuring-backup).
Si uno no desea detectar manualmente las bases de datos nueva, puede optar por autoprotection como se explicó [debajo](#enable-autoprotection).

## <a name="enable-autoprotection"></a>Habilitar AutoProtection

Un usuario puede configurar copia de seguridad de modo que todas las bases de datos que agregó en el futuro se protegen automáticamente con una directiva determinada. Para habilitar autoprotection, use [Enable AzRecoveryServicesBackupAutoProtection](https://docs.microsoft.com/powershell/module/az.recoveryservices/Enable-AzRecoveryServicesBackupAutoProtection?view=azps-1.5.0) cmdlet PS.

Puesto que la instrucción es una copia de seguridad de todas las bases de datos futuros, la operación se realiza en un SQLInstance nivel.

```powershell
$SQLInstance = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLInstance -VaultId $targetVault.ID -Name "<Protectable Item name>" -ServerName "<Server Name>"
Enable-AzRecoveryServicesBackupAutoProtection -InputItem $SQLInstance -BackupManagementType AzureWorkload -WorkloadType MSSQL -Policy $targetPolicy -VaultId $targetvault.ID
```

Una vez que se indica la intención de autoprotection, la consulta en la máquina para capturar recién agregado bases de datos se lleva a cabo como una tarea programada en segundo plano cada 8 horas.

## <a name="restore-sql-dbs"></a>Restaurar bases de datos SQL

Copia de seguridad de Azure puede restaurar las bases de datos de SQL Server que se ejecutan en máquinas virtuales de Azure como sigue:

1. Restaurar en una fecha u hora específicas (para la segunda) mediante el uso de copias de seguridad de registro de transacciones. Azure Backup determina automáticamente la copia de seguridad diferencial completas adecuado y la cadena de copias de seguridad del registro que son necesarias para restaurar según la hora seleccionada.
2. Restaurar una copia de seguridad completa o diferencial específica para restaurar a un punto de recuperación específico.

Compruebe los requisitos previos mencionados [aquí](restore-sql-database-azure-vm.md#prerequisites) antes de restaurar las bases de datos SQL.

Capturar primero la correspondiente copia de seguridad SQL DB mediante el [Get AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupItem?view=azps-1.5.0) cmdlet PS.

````powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
````

### <a name="fetch-the-relevant-restore-time"></a>Capturar la hora de restauración relevantes

Tal como se describió anteriormente, el usuario puede restaurar la base de datos de SQL de copia de seguridad a una copia completa o diferencial **o** a un registro en el momento.

#### <a name="fetch-distinct-recovery-points"></a>Capturar los puntos de recuperación diferentes

Use [Get AzRecoveryServicesBackupRecoveryPoint](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupRecoveryPoint?view=azps-1.5.0) capturar distintos puntos de recuperación (completa o diferencial) de una base de datos de SQL de copia de seguridad.

````powershell
$startDate = (Get-Date).AddDays(-7).ToUniversalTime()
$endDate = Get-Date.ToUniversalTime()
Get-AzRecoveryServicesBackupRecoveryPoint -Item $bkpItem -VaultId $targetVault.ID -StartDate $startdate -EndDate $endDate
````

El resultado es similar al ejemplo siguiente

````powershell
RecoveryPointId    RecoveryPointType  RecoveryPointTime      ItemName                             BackupManagemen
                                                                                                  tType
---------------    -----------------  -----------------      --------                             ---------------
6660368097802      Full               3/18/2019 8:09:35 PM   MSSQLSERVER;model             AzureWorkload
````

Usar el filtro 'RecoveryPointId' o una matriz para capturar el punto de recuperación correspondiente.

````powershell
$FullRP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $bkpItem -VaultId $targetVault.ID -RecoveryPointId "6660368097802"
````

#### <a name="fetch-point-in-time-recovery-point"></a>Capturar el punto de recuperación en un momento

Si el usuario desea restaurar la base de datos a un determinado en un momento, use [Get AzRecoveryServicesBackupRecoveryLogChain](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupRecoveryLogChain?view=azps-1.5.0) cmdlet PS. El cmdlet devuelve una lista de fechas que representan las horas de inicio y finalización de una cadena continua, ininterrumpida de registro para ese elemento de copia de seguridad de SQL. Debe ser el momento deseado dentro de este intervalo.

```powershell
Get-AzRecoveryServicesBackupRecoveryLogChain -Item $bkpItem -Item -VaultId $targetVault.ID
```

El resultado será similar al ejemplo siguiente.

````powershell
ItemName                       StartTime                      EndTime
--------                       ---------                      -------
SQLDataBase;MSSQLSERVER;azu... 3/18/2019 8:09:35 PM           3/19/2019 12:08:32 PM
````

La salida anterior significa que el usuario puede restaurar a cualquier punto en el tiempo entre la hora de inicio mostrada y la hora de finalización. Las horas están en UTC. Construir cualquier punto en el momento en PS que está dentro del intervalo que se muestra arriba.

> [!NOTE]
> Cuando un registro punto-in-time seleccionado para la restauración, los usuarios no necesitan especificar el punto de partida es decir, desde el que se restaura la base de datos de copia de seguridad completa. El servicio Azure Backup se encargará del plan de recuperación completa, es decir, que completa la copia de seguridad para elegir, lo que las copias de seguridad para aplicar etcetera de registro.

### <a name="determine-recovery-configuration"></a>Determinar la configuración de recuperación

En el caso de restauración de base de datos SQL, se admiten los siguientes escenarios de restauración.

1. Invalidación de la base de datos de SQL de copia de seguridad con los datos de otro punto de recuperación - OriginalWorkloadRestore
2. Restaurar la base de datos SQL como una base de datos en la misma instancia SQL - AlternateWorkloadRestore
3. Restaurar la base de datos SQL como una base de datos en otra instancia SQL en otra VM de SQL - AlternateWorkloadRestore

Después de capturar el punto de recuperación correspondiente (distinct o inicie sesión en un momento), utilice [Get AzRecoveryServicesBackupWorkloadRecoveryConfig](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupWorkloadRecoveryConfig?view=azps-1.5.0) cmdlet PS para capturar el objeto de configuración de recuperación según el plan de recuperación deseado.

#### <a name="original-workload-restore"></a>Restauración de carga de trabajo original

Para reemplazar la base de datos de copia de seguridad con los datos desde el punto de recuperación, simplemente especifique el marcador adecuado y el punto de recuperación correspondiente como se muestra en los ejemplos siguientes.

##### <a name="original-restore-with-distinct-recovery-point"></a>Restauración original con el punto de recuperación distinto

````powershell
$OverwriteWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -OriginalWorkloadRestore -VaultId $targetVault.ID
````

##### <a name="original-restore-with-log-point-in-time"></a>Restauración original con el registro en un momento

```powershell
$OverwriteWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -Item $bkpItem  -OriginalWorkloadRestore -VaultId $targetVault.ID
```

#### <a name="alternate-workload-restore"></a>Restauración de la carga de trabajo alternativo

> [!IMPORTANT]
> Una copia de seguridad de base de datos SQL se puede restaurar como una base de datos a otro SQLInstance solo, en una máquina virtual de Azure registrado en este almacén.

Como se describió anteriormente, si el destino SQLInstance se encuentra dentro de otra máquina virtual de Azure, asegúrese de que es [registrado en este almacén](#registering-the-sql-vm) y el parámetro SQLInstance relevante aparece como un elemento que se pueden proteger.

````powershell
$TargetInstance = Get-AzRecoveryServicesBackupProtectableItem -WorkloadType MSSQL -ItemType SQLInstance -Name "<SQLInstance Name>" -ServerName "<SQL VM name>" -VaultId $targetVault.ID
````

A continuación, simplemente pase el punto de recuperación pertinentes, instancia de SQL de destino con el indicador de derecho como se muestra a continuación.

##### <a name="alternate-restore-with-distinct-recovery-point"></a>Restauración alternativa con distintos puntos de recuperación

````powershell
$AnotherInstanceWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -TargetItem $TargetInstance -AlternateWorkloadRestore -VaultId $targetVault.ID
````

##### <a name="alternate-restore-with-log-point-in-time"></a>Restauración alternativa con el registro en un momento

```powershell
$AnotherInstanceWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -Item $bkpItem -AlternateWorkloadRestore -VaultId $targetVault.ID
```

El objeto de configuración de punto de recuperación final obtenida [Get AzRecoveryServicesBackupWorkloadRecoveryConfig](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupWorkloadRecoveryConfig?view=azps-1.5.0) cmdlet PS tiene toda la información relevante para la restauración y es como se muestra a continuación.

````powershell
TargetServer         : <SQL server name>
TargetInstance       : <Target Instance name>
RestoredDBName       : <Target Instance name>/azurebackup1_restored_3_19_2019_1850
OverwriteWLIfpresent : No
NoRecoveryMode       : Disabled
targetPhysicalPath   : {azurebackup1, azurebackup1_log}
ContainerId          : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/vmappcontainer;compute;computeRG;SQLVMName
SourceResourceId     : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/computeRG/VMAppContainer/SQLVMName
RestoreRequestType   : Alternate WL Restore
RecoveryPoint        : Microsoft.Azure.Commands.RecoveryServices.Backup.Cmdlets.Models.AzureWorkloadRecoveryPoint
PointInTime          : 1/1/0001 12:00:00 AM
````

Puede editar los campos de nombre, OverwriteWLIfpresent, NoRecoveryMode y targetPhysicalPath base de datos restaurados. Obtener más detalles sobre las rutas de acceso del archivo de destino tal y como se muestra a continuación.

````powershell
$AnotherInstanceWithFullConfig.targetPhysicalPath

MappingType SourceLogicalName SourcePath                  TargetPath
----------- ----------------- ----------                  ----------
Data        azurebackup1      F:\Data\azurebackup1.mdf    F:\Data\azurebackup1_1553001753.mdf
Log         azurebackup1_log  F:\Log\azurebackup1_log.ldf F:\Log\azurebackup1_log_1553001753.ldf
````

Establecer las propiedades correspondientes de PS como valores de cadena, como se muestra a continuación.

````powershell
$AnotherInstanceWithFullConfig.OverwriteWLIfpresent = "Yes"
$AnotherInstanceWithFullConfig | fl

TargetServer         : <SQL server name>
TargetInstance       : <Target Instance name>
RestoredDBName       : <Target Instance name>/azurebackup1_restored_3_19_2019_1850
OverwriteWLIfpresent : Yes
NoRecoveryMode       : Disabled
targetPhysicalPath   : {azurebackup1, azurebackup1_log}
ContainerId          : /Subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.RecoveryServices/vaults/testVault/backupFabrics/Azure/protectionContainers/vmappcontainer;compute;computeRG;SQLVMName
SourceResourceId     : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/computeRG/VMAppContainer/SQLVMName
RestoreRequestType   : Alternate WL Restore
RecoveryPoint        : Microsoft.Azure.Commands.RecoveryServices.Backup.Cmdlets.Models.AzureWorkloadRecoveryPoint
PointInTime          : 1/1/0001 12:00:00 AM
````

> [!IMPORTANT]
> Asegúrese de que el objeto de configuración de recuperación final tiene todos los valores adecuados y necesarios puesto que la operación de restauración se basará en el objeto de configuración.

### <a name="restore-with-relevant-configuration"></a>Restaurar con la configuración pertinente

Una vez obtenido el objeto de configuración de recuperación correspondiente y comprobado, utilice el [restauración AzRecoveryServicesBackupItem](https://docs.microsoft.com/powershell/module/az.recoveryservices/Restore-AzRecoveryServicesBackupItem?view=azps-1.5.0) cmdlet PS para iniciar el proceso de restauración.

````powershell
Restore-AzRecoveryServicesBackupItem -WLRecoveryConfig $AnotherInstanceWithLogConfig -VaultId $targetVault.ID
````

La operación de restauración devuelve un trabajo para realizar un seguimiento.

````powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
MSSQLSERVER/m... Restore              InProgress           3/17/2019 10:02:45 AM                                3274xg2b-e4fg-5952-89b4-8cb566gc1748
````

## <a name="manage-sql-backups"></a>Administrar copias de seguridad SQL

### <a name="on-demand-backup"></a>Copia de seguridad y a petición

Una vez que se ha habilitado la copia de seguridad para una base de datos, el usuario también puede desencadenar una copia de seguridad y a petición para la base de datos mediante [AzRecoveryServicesBackupItem de copia de seguridad](https://docs.microsoft.com/powershell/module/az.recoveryservices/Backup-AzRecoveryServicesBackupItem?view=azps-1.5.0) cmdlet PS. El siguiente ejemplo desencadena una copia de seguridad completa en una base de datos SQL con compresión habilitada y la copia de seguridad debe conservarse durante 60 días.

````powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
$endDate = (Get-Date).AddDays(60).ToUniversalTime()
Backup-AzRecoveryServicesBackupItem -Item $bkpItem -BackupType Full -EnableCompression -VaultId $targetVault.ID -ExpiryDateTimeUTC $endDate
````

El comando de copia de seguridad ad hoc devuelve un trabajo para realizar un seguimiento.

````powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
MSSQLSERVER/m... Backup               InProgress           3/18/2019 8:41:27 PM                                2516bb1a-d3ef-4841-97a3-9ba455fb0637
````

Si el resultado se pierde o si desea obtener el Id. de trabajo relevantes, [obtener la lista de trabajos](#track-azure-backup-jobs) de copia de seguridad de Azure del servicio y, a continuación, realizar un seguimiento de él y sus detalles.

### <a name="change-policy-for-backup-items"></a>Cambiar la directiva para los elementos de copia de seguridad

Usuario puede modificar la directiva existente o cambiar la directiva del elemento de copia de seguridad de Policy1 a política 2. Para cambiar las directivas para un elemento de copia de seguridad, simplemente capturar la directiva correspondiente y hacer una copia de seguridad de elemento y usar la [Enable AzRecoveryServices](https://docs.microsoft.com/powershell/module/az.recoveryservices/Enable-AzRecoveryServicesBackupProtection?view=azps-1.5.0) comando con el elemento de copia de seguridad como parámetro.

````powershell
$TargetPol1 = Get-AzRecoveryServicesBackupProtectionPolicy -Name <PolicyName>
$anotherBkpItem = Get-AzRecoveryServicesBackupItem -WorkloadType MSSQL -BackupManagementType AzureWorkload -Name "<BackupItemName>"
Enable-AzRecoveryServicesBackupProtection -Item $anotherBkpItem -Policy $TargetPol1
````

El comando espera hasta que la copia de seguridad de la configuración se ha completado y devuelve el resultado siguiente.

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
master           ConfigureBackup      Completed            3/18/2019 8:00:21 PM      3/18/2019 8:02:16 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="re-register-sql-vms"></a>Volver a registrar las máquinas virtuales SQL

> [!WARNING]
> Asegúrese de leer este [documento](backup-sql-server-azure-troubleshoot.md#re-registration-failures) para comprender los síntomas de error y las causas antes de intentar volver a registrar

Para desencadenar el nuevo registro de la VM de SQL, capturar el contenedor de copia de seguridad pertinente y páselo al cmdlet de registro.

````powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
Register-AzRecoveryServicesBackupContainer -Container $SQLContainer -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID
````

### <a name="stop-protection"></a>Detener la protección

#### <a name="retain-data"></a>Conservar datos

Si el usuario desea detener la protección, puede usar el [Disable AzRecoveryServicesBackupProtection](https://docs.microsoft.com/powershell/module/az.recoveryservices/Disable-AzRecoveryServicesBackupProtection?view=azps-1.5.0) cmdlet PS. Esto detendrá la copias de seguridad programadas, pero la seguridad de los datos hasta que ahora se conservan para siempre.

````powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID
````

#### <a name="delete-backup-data"></a>Eliminación de datos de copia de seguridad

Para quitar completamente los datos de copia de seguridad almacenados en el almacén, solo tiene que agregar '-marcador o switch de RemoveRecoveryPoints la ['disable' comando protección](#retain-data).

````powershell
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID -RemoveRecoveryPoints
````

#### <a name="disable-auto-protection"></a>Deshabilite la protección automática

Si autoprotection se configuró en un SQLInstance, el usuario puede deshabilitar mediante la [Disable AzRecoveryServicesBackupAutoProtection](https://docs.microsoft.com/powershell/module/az.recoveryservices/Disable-AzRecoveryServicesBackupAutoProtection?view=azps-1.5.0) cmdlet PS.

````powershell
$SQLInstance = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLInstance -VaultId $targetVault.ID -Name "<Protectable Item name>" -ServerName "<Server Name>"
Disable-AzRecoveryServicesBackupAutoProtection -InputItem $SQLInstance -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetvault.ID
````

#### <a name="unregister-sql-vm"></a>Anular el registro de la máquina virtual de SQL

Si todas las bases de datos de SQL server [ya no son datos protegidos y copia de seguridad no existen](#delete-backup-data), usuario puede anular el registro de la VM de SQL de este almacén. Solo entonces el usuario pueda proteger bases de datos a otro almacén. Use [Unregister-AzRecoveryServicesBackupContainer](https://docs.microsoft.com/powershell/module/az.recoveryservices/Unregister-AzRecoveryServicesBackupContainer?view=azps-1.5.0) cmdlet de PS para anular el registro de la VM de SQL.

````powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
 Unregister-AzRecoveryServicesBackupContainer -Container $SQLContainer -VaultId $targetvault.ID
````

### <a name="track-azure-backup-jobs"></a>Realizar un seguimiento de los trabajos de copia de seguridad de Azure

Es importante tener en cuenta que Azure Backup solo realiza el seguimiento de los trabajos de usuario que se desencadena en copia de seguridad SQL. Copias de seguridad programadas (incluidas las copias de seguridad del registro) no están visibles en el portal o powershell. Sin embargo, si cualquier programa trabajos producirá un error, un [alerta de copia de seguridad](backup-azure-monitoring-built-in-monitor.md#backup-alerts-in-recovery-services-vault) se genera y se muestra en el portal. [Usar Azure Monitor](backup-azure-monitoring-use-azuremonitor.md) para realizar un seguimiento de todos los trabajos programados y otra información relevante.

Los usuarios pueden seguir las operaciones de ad hoc o usuario desencadenada con JobID devuelto en el [salida](#on-demand-backup) de trabajos asincrónicos como copia de seguridad. Use [Get AzRecoveryServicesBackupJobDetail](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupJobDetail) cmdlet PS para realizar un seguimiento del trabajo y sus detalles.

````powershell
 Get-AzRecoveryServicesBackupJobDetails -JobId 2516bb1a-d3ef-4841-97a3-9ba455fb0637 -VaultId $targetVault.ID
````

Para obtener la lista de trabajos de ad hoc y sus Estados de servicio de copia de seguridad de Azure, use [Get AzRecoveryServicesBackupJob](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupJob?view=azps-1.5.0) cmdlet PS. El ejemplo siguiente devuelve todos los trabajos SQL en curso.

```powershell
Get-AzRecoveryServicesBackupJob -Status InProgress -BackupManagementType AzureWorkload
```

Para cancelar un trabajo en curso, utilice el [Stop AzRecoveryServicesBackupJob](https://docs.microsoft.com/powershell/module/az.recoveryservices/Stop-AzRecoveryServicesBackupJob?view=azps-1.5.0) cmdlet PS.

## <a name="managing-sql-always-on-availability-groups"></a>Administración de grupos de SQL disponibilidad Always On

Para SQL grupos de disponibilidad AlwaysOn, no olvide [registrar todos los nodos](#registering-the-sql-vm) del grupo de disponibilidad (AG). Una vez que el registro se realiza para todos los nodos, un objeto de grupo de disponibilidad SQL está creado lógicamente en elementos que se pueden proteger. Las bases de datos en el grupo de disponibilidad de SQL se mostrará como "SQLDatabase". Los nodos se mostrarán como instancias independientes y las bases de datos SQL de forma predeterminada en ellos se mostrará como las bases de datos como SQL.

Por ejemplo, supongamos que un grupo de disponibilidad de SQL tiene dos nodos: "sql-server-0" y "sql-server-1" y 1 SQL AG DB. Una vez que se registran estos dos nodos, si usuario [se enumeran los elementos que se pueden proteger](#fetching-sql-dbs), enumera los siguientes componentes

1. Un objeto de grupo de disponibilidad de SQL - que se pueden proteger tipo como SQLAvailabilityGroup de elemento
2. Un grupo de disponibilidad de SQL DB - tipo de elemento que se pueden proteger como SQLDatabase
3. SQL-server-0 - tipo de elemento que se pueden proteger como SQLInstance
4. SQL-server-1: tipo de elemento que se pueden proteger como SQLInstance
5. Cualquier valor predeterminado de bases de datos de SQL (master, model, msdb) en sql-server-0 - tipo de elemento que se pueden proteger como SQLDatabase
6. Cualquier valor predeterminado de bases de datos de SQL (master, model, msdb) en sql-server-1: tipo de elemento que se pueden proteger como SQLDatabase

SQL-server-0, 1 de sql server también se mostrará como "AzureVMAppContainer" cuando [se muestran los contenedores de copia de seguridad](https://docs.microsoft.com/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupContainer?view=azps-1.5.0).

Obtener tan sólo la base de datos SQL pertinente a [habilitar copia de seguridad](#configuring-backup) y [copia de seguridad ad hoc](#on-demand-backup) y [restaurar los cmdlets PS](#restore-sql-dbs) son idénticos.
