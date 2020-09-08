---
title: Copia de seguridad y restauración de bases de datos SQL en máquinas virtuales de Azure mediante PowerShell
description: Copia de seguridad y restauración de bases de datos SQL en Azure VM con Azure Backup y PowerShell.
ms.topic: conceptual
ms.date: 03/15/2019
ms.assetid: 57854626-91f9-4677-b6a2-5d12b6a866e1
ms.openlocfilehash: 1fe3af3b2a12cf6fdfc0e71d36d36046858c50af
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88892429"
---
# <a name="back-up-and-restore-sql-databases-in-azure-vms-with-powershell"></a>Copia de seguridad y restauración de bases de datos SQL en máquinas virtuales de Azure con PowerShell

En este artículo se describe cómo usar Azure PowerShell para realizar copias de seguridad y recuperar una base de datos SQL dentro de una máquina virtual de Azure mediante el almacén de Recovery Services de [Azure Backup](backup-overview.md).

En este artículo se explica lo siguiente:

> [!div class="checklist"]
>
> * Configurar PowerShell y registrar el proveedor de Azure Recovery Services.
> * Cree un almacén de Recovery Services.
> * Configurar la copia de seguridad de SQL DB dentro de una máquina virtual de Azure.
> * Ejecutar un trabajo de copia de seguridad.
> * Restaurar una copia de seguridad de base de datos SQL.
> * Supervisar los trabajos de copia de seguridad y restauración.

## <a name="before-you-start"></a>Antes de comenzar

* [Más información](backup-azure-recovery-services-vault-overview.md) sobre los almacenes de Recovery Services.
* Obtenga información sobre las funcionalidades de características para [la copia de seguridad de bases de datos SQL dentro de las máquinas virtuales de Azure](backup-azure-sql-database.md#before-you-start).
* Revise la jerarquía de objetos de PowerShell para Recovery Services.

### <a name="recovery-services-object-hierarchy"></a>Jerarquía de objetos de Recovery Services

En el diagrama siguiente, se resume la jerarquía de objetos.

![Jerarquía de objetos de Recovery Services](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

Revise la [referencia de cmdlet](/powershell/module/az.recoveryservices) de **Az.RecoveryServices** en la biblioteca de Azure.

### <a name="set-up-and-install"></a>Configuración e instalación

Configure PowerShell como sigue:

1. [Descargue la versión más reciente de Azure PowerShell](/powershell/azure/install-az-ps). La versión 1.5.0 es la versión mínima requerida.

2. Busque los cmdlets de PowerShell de Azure Backup con este comando:

    ```powershell
    Get-Command *azrecoveryservices*
    ```

3. Revise los alias y cmdlets de Azure Backup y el almacén de Recovery Services. Este es un ejemplo de lo que puede ver. No es la lista completa de los cmdlets.

    ![Lista de cmdlets de Recovery Services](./media/backup-azure-afs-automation/list-of-recoveryservices-ps-az.png)

4. Inicie sesión en su cuenta de Azure con el cmdlet **Connect-AzAccount**.
5. En la página web que aparece, se le pedirá que escriba las credenciales de su cuenta.

    * Como alternativa, puede incluir las credenciales de su cuenta como un parámetro en el cmdlet **Connect-AzAccount** mediante el parámetro **-Credential**.
    * Si es un asociado CSP que trabaja en nombre de un inquilino, especifique el cliente como inquilino usando su TenantID o su nombre de dominio principal de inquilino. Un ejemplo es **Connect-AzAccount -Tenant** fabrikam.com.

6. Asocie la suscripción que quiere usar con la cuenta, dado que una cuenta puede tener varias suscripciones.

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

7. Si utiliza Azure Backup por primera vez, use el cmdlet **Register-AzResourceProvider** para registrar el proveedor de Azure Recovery Services con su suscripción.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

8. Compruebe que los proveedores se han registrado correctamente:

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

9. En la salida del comando, compruebe que **RegistrationState** cambia a **Registered**. En caso contrario, vuelva a ejecutar el cmdlet **Register-AzResourceProvider**.

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Siga estos pasos para crear un almacén de Recovery Services.

El almacén de Recovery Services es un recurso de Resource Manager, por lo que deberá colocarlo dentro de un grupo de recursos. Puede usar un grupo de recursos existente o crear uno con el cmdlet **New-AzResourceGroup**. Al crear un grupo de recursos, especifique el nombre y la ubicación.

1. Un almacén se coloca en un grupo de recursos. Si no tiene un grupo de recursos, cree uno con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo, se crea un grupo de recursos en la región Oeste de EE. UU.

    ```powershell
    New-AzResourceGroup -Name "test-rg" -Location "West US"
    ```

2. Use el cmdlet [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) para crear el almacén. Especifique para el almacén la misma ubicación del grupo de recursos.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

3. Especifique el tipo de redundancia para el almacenamiento.

    * Puede usar [almacenamiento con redundancia local](../storage/common/storage-redundancy.md) o [almacenamiento con redundancia geográfica](../storage/common/storage-redundancy.md).
    * En el siguiente ejemplo se establece la opción **-BackupStorageRedundancy** para el cmdlet [Set-AzRecoveryServicesBackupProperty](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupproperty) para **testvault** se establece en **GeoRedundant**.

    ```powershell
    $vault1 = Get-AzRecoveryServicesVault -Name "testvault"
    Set-AzRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant
    ```

### <a name="view-the-vaults-in-a-subscription"></a>Visualización de los almacenes de una suscripción

Para ver todos los almacenes de la suscripción, use [Get-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault).

```powershell
Get-AzRecoveryServicesVault
```

La salida será similar a la siguiente. Se proporcionan el grupo de recursos y la ubicación asociados.

```output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

### <a name="set-the-vault-context"></a>Establecer el contexto de almacén

Almacene el objeto de almacén en una variable y establezca el contexto de almacén.

* Muchos de los cmdlets de Azure Backup requieren el objeto de almacén de Recovery Services como una entrada, por lo que es conveniente almacenar el objeto de almacén en una variable.
* El contexto de almacén es el tipo de los datos protegidos en el almacén. Establézcalo con [Set-AzRecoveryServicesVaultContext](/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext). Una vez que se haya establecido el contexto, se aplica a todos los cmdlets posteriores.

En el ejemplo siguiente se establece el contexto del almacén de **testvault**.

```powershell
Get-AzRecoveryServicesVault -Name "testvault" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Recuperación del identificador del almacén

Tenemos previsto dejar de usar la configuración del contexto de almacén según las directrices de Azure PowerShell. En su lugar, puede almacenar o recuperar el identificador del almacén y pasarlo a los comandos pertinentes, como sigue:

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
```

## <a name="configure-a-backup-policy"></a>Configuración de una directiva de copia de seguridad

Una directiva de copia de seguridad especifica la programación para las copias de seguridad y cuánto tiempo se deben mantener los puntos de recuperación de copia de seguridad:

* Una directiva de copia de seguridad está asociada con al menos una directiva de retención. Una directiva de retención define el tiempo que se conserva un punto de recuperación antes de que se elimine.
* Vea la retención de la directiva de copia de seguridad predeterminada con [Get-AzRecoveryServicesBackupRetentionPolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject).
* Vea la programación de la directiva de copia de seguridad predeterminada con [Get-AzRecoveryServicesBackupSchedulePolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject).
* El cmdlet [New AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy) se usa para crear una directiva de copia de seguridad. Introduzca los objetos de directiva de retención y programación.

De forma predeterminada, se define una hora de inicio en el objeto de directiva de programación. Use el siguiente ejemplo para cambiar la hora de inicio a la hora de inicio deseada. La hora de inicio deseada debe estar también en formato UTC. En el ejemplo siguiente se supone que la hora de inicio deseada es la 1:00 UTC para las copias de seguridad diarias.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "MSSQL"
$UtcTime = Get-Date -Date "2019-03-20 01:30:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

> [!IMPORTANT]
> Solo tiene que proporcionar la hora de inicio en múltiplos de 30 minutos. En el ejemplo anterior, solo puede ser "01:00:00" o "02:30:00". La hora de inicio no puede ser "01:15:00".

En el ejemplo siguiente se almacenan la directiva de programación y la directiva de retención en variables. A continuación, usa estas variables como parámetros para una nueva directiva (**NewSQLPolicy**). **NewSQLPolicy** toma una copia de seguridad diario "completa" diaria, la conserva durante 180 días y toma una copia de seguridad del registro cada 2 horas

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "MSSQL"
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "MSSQL"
$NewSQLPolicy = New-AzRecoveryServicesBackupProtectionPolicy -Name "NewSQLPolicy" -WorkloadType "MSSQL" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

La salida será similar a la siguiente.

```output
Name                 WorkloadType       BackupManagementType BackupTime                Frequency                                IsDifferentialBackup IsLogBackupEnabled
                                                                                                                                Enabled
----                 ------------       -------------------- ----------                ---------                                -------------------- ------------------
NewSQLPolicy         MSSQL              AzureWorkload        3/15/2019 01:30:00 AM      Daily                                    False                True
```

## <a name="enable-backup"></a>Habilitación de la copia de seguridad

### <a name="registering-the-sql-vm"></a>Registro de la máquina virtual SQL

Para las copias de seguridad de máquinas virtuales de Azure y los recursos compartidos de Azure File, el servicio Backup puede conectarse a estos recursos de Azure Resource Manager y capturar los detalles pertinentes. Puesto que SQL es una aplicación dentro de una máquina virtual de Azure, el servicio Backup necesita permiso para acceder a la aplicación y capturar los detalles necesarios. Para ello, debe *"registrar"* la máquina virtual de Azure que contiene la aplicación de SQL con un almacén de Recovery Services. Una vez que registra una máquina virtual de SQL con un almacén, puede proteger las bases de datos SQL solo en dicho almacén. Use el cmdlet [Register-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/register-azrecoveryservicesbackupcontainer) de PowerShell para registrar la máquina virtual.

```powershell
 $myVM = Get-AzVM -ResourceGroupName <VMRG Name> -Name <VMName>
Register-AzRecoveryServicesBackupContainer -ResourceId $myVM.ID -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID -Force
```

El comando devolverá un contenedor de copia de seguridad de este recurso y el estado se registrará.

> [!NOTE]
> Si no se especifica el parámetro force, se pide al usuario que lo confirme con el mensaje "¿Desea deshabilitar la protección de este contenedor?". Por favor, ignore este texto e indique "Y" para confirmar. Se trata de un problema conocido y estamos trabajando para quitar el mensaje y los requisitos para el parámetro force.

### <a name="fetching-sql-dbs"></a>Obtención de bases de datos SQL

Una vez que se realiza el registro, el servicio Backup podrá mostrar todos los componentes SQL disponibles en la máquina virtual. Para ver todos los componentes SQL que aún tienen que incluirse en la copia de seguridad para este almacén, use el cmdlet [Get-AzRecoveryServicesBackupProtectableItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectableitem) de PowerShell.

```powershell
Get-AzRecoveryServicesBackupProtectableItem -WorkloadType MSSQL -VaultId $targetVault.ID
```

La salida mostrará todos los componentes SQL no protegidos en todas las máquinas virtuales SQL registradas en este almacén con Item Type (Tipo de elemento) y ServerName (Nombre de servidor). Puede filtrar aún más una máquina virtual SQL determinada si pasa el parámetro "-Container" o usa la combinación de "Name" y "ServerName" junto con la marca ItemType para llegar a un único elemento de SQL.

```powershell
$SQLDB = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLDataBase -VaultId $targetVault.ID -Name "<Item Name>" -ServerName "<Server Name>"
```

### <a name="configuring-backup"></a>Configuración de la copia de seguridad

Ahora que tenemos la base de datos SQL y la directiva necesarias con la que necesita incluirse en la copia de seguridad, podemos utilizar el cmdlet [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) para configurar la copia de seguridad para esta base de datos de SQL.

```output
Enable-AzRecoveryServicesBackupProtection -ProtectableItem $SQLDB -Policy $NewSQLPolicy
```

El comando espera hasta que la copia de seguridad de la configuración se complete y devuelve la salida siguiente.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
master           ConfigureBackup      Completed            3/18/2019 6:00:21 PM      3/18/2019 6:01:35 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="fetching-new-sql-dbs"></a>Obtención de nuevas bases de datos SQL

Una vez registrada la máquina, el servicio Backup capturará los detalles de las bases de datos disponibles entonces. Si se agregan instancias de SQL o bases de datos SQL a la máquina registrada más adelante, es necesario desencadenar manualmente el servicio Backup para llevar a cabo una nueva consulta y obtener **todas** las bases de datos no protegidas (incluidas las recién agregadas). Use el cmdlet [Initialize-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/initialize-azrecoveryservicesbackupprotectableitem) de PowerShell en la máquina virtual de SQL para realizar una consulta nueva. El comando espera hasta que se completa la operación. Use más adelante el cmdlet [Get AzRecoveryServicesBackupProtectableItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectableitem) de PowerShell para obtener la lista de componentes SQL más recientes sin protección.

```powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
Initialize-AzRecoveryServicesBackupProtectableItem -Container $SQLContainer -WorkloadType MSSQL -VaultId $targetvault.ID
Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLDataBase -VaultId $targetVault.ID
```

Una vez que se capturan los elementos relevantes que se pueden proteger, habilite las copias de seguridad como se indica en la [sección anterior](#configuring-backup).
Si no se desea detectar manualmente las bases de datos nuevas, puede optar por la protección automática, como se explica [a continuación](#enable-autoprotection).

## <a name="enable-autoprotection"></a>Habilitar la protección automática

Puede configurar la copia de seguridad de modo que todas las bases de datos que se agreguen en el futuro se protejan automáticamente con una directiva determinada. Para habilitar la protección automática, use el cmdlet [Enable AzRecoveryServicesBackupAutoProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupautoprotection) de PowerShell.

Puesto que la indicación es realizar la copia de seguridad de todas las bases de datos posteriores, la operación se realiza en un nivel de SQLInstance.

```powershell
$SQLInstance = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLInstance -VaultId $targetVault.ID -Name "<Protectable Item name>" -ServerName "<Server Name>"
Enable-AzRecoveryServicesBackupAutoProtection -InputItem $SQLInstance -BackupManagementType AzureWorkload -WorkloadType MSSQL -Policy $NewSQLPolicy -VaultId $targetvault.ID
```

Una vez que se intenta la protección automática, la consulta en la máquina para capturar las bases de datos recién agregadas se lleva a cabo como una tarea programada en segundo plano cada 8 horas.

## <a name="restore-sql-dbs"></a>Restauración de las bases de datos SQL

Azure Backup puede restaurar las bases de datos SQL Server que se ejecutan en las máquinas virtuales de Azure como se indica a continuación:

* Restaure a una fecha u hora específicas (hasta los segundos) mediante copias de seguridad del registro de transacciones. Azure Backup determina automáticamente la copia de seguridad diferencial completa apropiada y la cadena de copias de seguridad de registros necesarias para restaurar los datos en función del tiempo seleccionado.
* Restaure una copia de seguridad completa o diferencial específica para restaurar a un punto de recuperación específico.

Compruebe los requisitos previos mencionados [aquí](restore-sql-database-azure-vm.md#prerequisites) antes de restaurar las bases de datos SQL.

Capture primero la copia de seguridad correspondiente de la base de datos SQL mediante el cmdlet [Get AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupitem) de PowerShell.

```powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
```

### <a name="fetch-the-relevant-restore-time"></a>Captura de la hora de restauración pertinente

Tal como se describió anteriormente, puede restaurar la base de datos SQL de la copia de seguridad a una copia completa o diferencial **O** a un registro a un momento dado.

#### <a name="fetch-distinct-recovery-points"></a>Captura de los diferentes puntos de recuperación

Use [Get AzRecoveryServicesBackupRecoveryPoint](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverypoint) para capturar distintos puntos de recuperación (completa o diferencial) de una base de datos de SQL de copia de seguridad.

```powershell
$startDate = (Get-Date).AddDays(-7).ToUniversalTime()
$endDate = (Get-Date).ToUniversalTime()
Get-AzRecoveryServicesBackupRecoveryPoint -Item $bkpItem -VaultId $targetVault.ID -StartDate $startdate -EndDate $endDate
```

La salida es similar a la del ejemplo siguiente.

```output
RecoveryPointId    RecoveryPointType  RecoveryPointTime      ItemName                             BackupManagemen
                                                                                                  tType
---------------    -----------------  -----------------      --------                             ---------------
6660368097802      Full               3/18/2019 8:09:35 PM   MSSQLSERVER;model             AzureWorkload
```

Use el filtro "RecoveryPointId" o un filtro de matriz para capturar el punto de recuperación correspondiente.

```powershell
$FullRP = Get-AzRecoveryServicesBackupRecoveryPoint -Item $bkpItem -VaultId $targetVault.ID -RecoveryPointId "6660368097802"
```

#### <a name="fetch-point-in-time-recovery-point"></a>Recuperación a un momento dado

Si quiere restaurar la base de datos a un momento dado, use el cmdlet [Get AzRecoveryServicesBackupRecoveryLogChain](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackuprecoverylogchain) de PowerShell. El cmdlet devuelve una lista de fechas que representan las horas de inicio y finalización de una cadena continua, ininterrumpida de registro para ese elemento de copia de seguridad de SQL. El momento deseado debería estar dentro de este intervalo.

```powershell
Get-AzRecoveryServicesBackupRecoveryLogChain -Item $bkpItem -Item -VaultId $targetVault.ID
```

El resultado será similar al ejemplo siguiente.

```output
ItemName                       StartTime                      EndTime
--------                       ---------                      -------
SQLDataBase;MSSQLSERVER;azu... 3/18/2019 8:09:35 PM           3/19/2019 12:08:32 PM
```

La salida anterior significa que puede restaurar a cualquier momento entre la hora de inicio y la hora de finalización mostradas. Todas las horas se muestran en UTC. Cree un momento en PowerShell que esté dentro del intervalo que se muestra arriba.

> [!NOTE]
> Cuando se selecciona un registro a un momento dado para restaurar, no tiene que especificar el punto de partida, es decir, la copia de seguridad completa a partir de la que se restaura la base de datos. El servicio Azure Backup se encargará del plan de recuperación completa, es decir, qué copia de seguridad completa elegir, qué copias de seguridad del registro aplicar, etc.

### <a name="determine-recovery-configuration"></a>Determinación de la configuración ante desastres

En el caso de la restauración de una base de datos SQL, se admiten los siguientes escenarios de restauración.

* Invalidación de la base de datos SQL de copia de seguridad con los datos de otro punto de recuperación: OriginalWorkloadRestore
* Restauración de la base de datos SQL como una base de datos nueva en la misma instancia SQL: AlternateWorkloadRestore
* Restauración de la base de datos SQL como una base de datos nueva en la otra instancia SQL de otra máquina virtual SQL: AlternateWorkloadRestore
* Restaurar la base de datos de SQL como archivos bak.: RestoreAsFiles

Después de capturar el punto de recuperación correspondiente (distinto o registro punto en el tiempo), use el cmdlet [Get-AzRecoveryServicesBackupWorkloadRecoveryConfig](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupworkloadrecoveryconfig) de PowerShell para capturar el objeto de configuración de recuperación según el plan de recuperación deseado.

#### <a name="original-workload-restore"></a>Restauración de carga de trabajo original

Para reemplazar la base de datos de copia de seguridad con los datos a partir del punto de recuperación, simplemente especifique la marca adecuada y el punto de recuperación correspondiente como se muestra en los ejemplos siguientes.

##### <a name="original-restore-with-distinct-recovery-point"></a>Restauración original con un punto de recuperación distinto

```powershell
$OverwriteWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -OriginalWorkloadRestore -VaultId $targetVault.ID
```

##### <a name="original-restore-with-log-point-in-time"></a>Restauración original con el registro a un momento dado

```powershell
$OverwriteWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -Item $bkpItem  -OriginalWorkloadRestore -VaultId $targetVault.ID
```

#### <a name="alternate-workload-restore"></a>Restauración de carga de trabajo alternativa

> [!IMPORTANT]
> Una copia de seguridad de una base de datos SQL se puede restaurar como una base de datos solo a otra instancia SQLInstance, en una máquina virtual de Azure registrada en este almacén.

Como se describió anteriormente, si la instancia SQLInstance de destino se encuentra dentro de otra máquina virtual de Azure, asegúrese de que se [registra en este almacén](#registering-the-sql-vm) y de que la instancia SQLInstance correspondiente aparece como un elemento que se pueda proteger.

```powershell
$TargetInstance = Get-AzRecoveryServicesBackupProtectableItem -WorkloadType MSSQL -ItemType SQLInstance -Name "<SQLInstance Name>" -ServerName "<SQL VM name>" -VaultId $targetVault.ID
```

A continuación, simplemente pase el punto de recuperación pertinente y la instancia SQL de destino con la marca derecha, como se muestra a continuación.

##### <a name="alternate-restore-with-distinct-recovery-point"></a>Restauración alternativa con un punto de recuperación distinto

```powershell
$AnotherInstanceWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -TargetItem $TargetInstance -AlternateWorkloadRestore -VaultId $targetVault.ID
```

##### <a name="alternate-restore-with-log-point-in-time"></a>Restauración alternativa con el registro a un momento dado

```powershell
$AnotherInstanceWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -Item $bkpItem -AlternateWorkloadRestore -VaultId $targetVault.ID
```

##### <a name="restore-as-files"></a>Restaurar como Archivos

Para restaurar los datos de copia de seguridad como archivos .bak en lugar de una base de datos, seleccione la opción **Restaurar como archivos**. La base de datos SQL de la que se ha realizado una copia de seguridad se puede restaurar en cualquier máquina virtual de destino registrada en este almacén.

```powershell
$TargetContainer= Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName "VM name" -VaultId $vaultID
```

##### <a name="restore-as-files-with-distinct-recovery-point"></a>Restaurar como archivos con un punto de recuperación distinto

```powershell
$FileRestoreWithFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -RecoveryPoint $FullRP -TargetContainer $TargetContainer -RestoreAsFiles -FilePath "<>" -VaultId $targetVault.ID
```

##### <a name="restore-as-files-with-log-point-in-time-from-latest-full"></a>Restaurar como archivos con el punto de registro en el tiempo desde la última versión completa

```powershell
$FileRestoreWithLogConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -TargetContainer $TargetContainer -RestoreAsFiles -FilePath "<>" -VaultId $targetVault.ID
```

##### <a name="restore-as-files-with-log-point-in-time-from-a-specified-full"></a>Restaurar como archivos con el punto de registro en el tiempo desde un completo especificado

Si desea proporcionar un completo específico que se debe usar para la restauración, use el siguiente comando:

```powershell
$FileRestoreWithLogAndSpecificFullConfig = Get-AzRecoveryServicesBackupWorkloadRecoveryConfig -PointInTime $PointInTime -FromFull $FullRP -TargetContainer $TargetContainer -RestoreAsFiles -FilePath "<>" -VaultId $targetVault.ID
```

El objeto de configuración del punto de recuperación final obtenido con el cmdlet [Get-AzRecoveryServicesBackupWorkloadRecoveryConfig](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupworkloadrecoveryconfig) de PowerShell tiene toda la información pertinente para la restauración y es como se muestra a continuación.

```output
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
```

Puede editar los campos del nombre de la base de datos restaurada, OverwriteWLIfpresent, NoRecoveryMode y targetPhysicalPath. Obtenga más detalles sobre las rutas de acceso del archivo de destino tal y como se muestra a continuación.

```powershell
$AnotherInstanceWithFullConfig.targetPhysicalPath
```

```output
MappingType SourceLogicalName SourcePath                  TargetPath
----------- ----------------- ----------                  ----------
Data        azurebackup1      F:\Data\azurebackup1.mdf    F:\Data\azurebackup1_1553001753.mdf
Log         azurebackup1_log  F:\Log\azurebackup1_log.ldf F:\Log\azurebackup1_log_1553001753.ldf
```

Establezca las propiedades correspondientes de PowerShell como valores de cadena, como se muestra a continuación.

```powershell
$AnotherInstanceWithFullConfig.OverwriteWLIfpresent = "Yes"
$AnotherInstanceWithFullConfig | fl
```

```output
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
```

> [!IMPORTANT]
> Asegúrese de que el objeto de configuración de recuperación final tiene todos los valores adecuados y necesarios puesto que la operación de restauración se basará en el objeto de configuración.

### <a name="restore-with-relevant-configuration"></a>Restauración con la configuración pertinente

Una vez se obtenga y se compruebe el objeto de configuración de recuperación correspondiente, utilice el cmdlet [restauración AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/restore-azrecoveryservicesbackupitem) de PowerShell para iniciar el proceso de restauración.

```powershell
Restore-AzRecoveryServicesBackupItem -WLRecoveryConfig $AnotherInstanceWithLogConfig -VaultId $targetVault.ID
```

La operación de restauración devuelve un trabajo para realizar el seguimiento.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
MSSQLSERVER/m... Restore              InProgress           3/17/2019 10:02:45 AM                                3274xg2b-e4fg-5952-89b4-8cb566gc1748
```

## <a name="manage-sql-backups"></a>Administración de las copias de seguridad SQL

### <a name="on-demand-backup"></a>Copia de seguridad a petición

Una vez que se ha habilitado la copia de seguridad para una base de datos, también puede desencadenar una copia de seguridad a petición para la base de datos con el cmdlet [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem) de PowerShell. El siguiente ejemplo desencadena una copia de seguridad completa en una base de datos SQL con la compresión habilitada y la copia de seguridad completa debe conservarse durante 60 días.

```powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
$endDate = (Get-Date).AddDays(60).ToUniversalTime()
Backup-AzRecoveryServicesBackupItem -Item $bkpItem -BackupType Full -EnableCompression -VaultId $targetVault.ID -ExpiryDateTimeUTC $endDate
```

El comando de copia de seguridad a petición devuelve un trabajo para realizar un seguimiento.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
MSSQLSERVER/m... Backup               InProgress           3/18/2019 8:41:27 PM                                2516bb1a-d3ef-4841-97a3-9ba455fb0637
```

Si el resultado se pierde o si desea obtener el identificador de trabajo correspondiente, [obtenga la lista de trabajos](#track-azure-backup-jobs) del servicio Azure Backup y, a continuación, realice un seguimiento del mismo y de sus detalles.

### <a name="change-policy-for-backup-items"></a>Cambio de la directiva para los elementos de copia de seguridad

Puede cambiar la directiva del elemento de copia de seguridad de *Policy1* a *Policy2*. Para cambiar las directivas para un elemento de copia de seguridad, capture la directiva correspondiente y haga una copia de seguridad del elemento, y use el comando [Enable-AzRecoveryServices](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection) con el elemento de copia de seguridad como parámetro.

```powershell
$TargetPol1 = Get-AzRecoveryServicesBackupProtectionPolicy -Name <PolicyName>
$anotherBkpItem = Get-AzRecoveryServicesBackupItem -WorkloadType MSSQL -BackupManagementType AzureWorkload -Name "<BackupItemName>"
Enable-AzRecoveryServicesBackupProtection -Item $anotherBkpItem -Policy $TargetPol1
```

El comando espera hasta que la copia de seguridad de la configuración se complete y devuelve la salida siguiente.

```output
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
master           ConfigureBackup      Completed            3/18/2019 8:00:21 PM      3/18/2019 8:02:16 PM      654e8aa2-4096-402b-b5a9-e5e71a496c4e
```

### <a name="edit-an-existing-backup-policy"></a>Edición de una directiva de copia de seguridad existente

Para editar una directiva existente, use el comando [Set-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy).

```powershell
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $Pol -SchedulePolicy $SchPol -RetentionPolicy $RetPol
```

Compruebe los trabajos de copia de seguridad después de que haya transcurrido algún tiempo para realizar un seguimiento de posibles errores. Si hubiera errores, debe corregirlos. Después, vuelva a ejecutar el comando de edición de la directiva con el parámetro **FixForInconsistentItems** para volver a intentar editar la directiva en todos los elementos de copia de seguridad para los que se produjo un error en la operación anteriormente.

```powershell
Set-AzRecoveryServicesBackupProtectionPolicy -Policy $Pol -FixForInconsistentItems
```

### <a name="re-register-sql-vms"></a>Volver a registrar las máquinas virtuales SQL

> [!WARNING]
> Asegúrese de leer este [documento](backup-sql-server-azure-troubleshoot.md#re-registration-failures) para comprender los síntomas de error y las causas antes de volver a intentar el registro.

Para desencadenar el nuevo registro de la máquina virtual SQL, capture el contenedor de copia de seguridad pertinente y páselo al cmdlet de registro.

```powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
Register-AzRecoveryServicesBackupContainer -Container $SQLContainer -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetVault.ID
```

### <a name="stop-protection"></a>Detener protección

#### <a name="retain-data"></a>Conservación de los datos

Si quiere detener la protección, puede usar el cmdlet [Disable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupprotection) de PowerShell. De esta forma, se detendrán las copias de seguridad programadas, pero los datos que se hayan incluido en ellas hasta el momento se conservarán.

```powershell
$bkpItem = Get-AzRecoveryServicesBackupItem -BackupManagementType AzureWorkload -WorkloadType MSSQL -Name "<backup item name>" -VaultId $targetVault.ID
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID
```

#### <a name="delete-backup-data"></a>Eliminación de datos de copia de seguridad

Para quitar completamente los datos de copia de seguridad almacenados en el almacén, solo tiene que agregar el marcador "-RemoveRecoveryPoints" al [comando de protección "disable"](#retain-data).

```powershell
Disable-AzRecoveryServicesBackupProtection -Item $bkpItem -VaultId $targetVault.ID -RemoveRecoveryPoints
```

#### <a name="disable-auto-protection"></a>Deshabilitación de la protección automática

Si la protección automática se configuró en una instancia SQLInstance, el usuario puede deshabilitarla con el cmdlet [Disable-AzRecoveryServicesBackupAutoProtection](/powershell/module/az.recoveryservices/disable-azrecoveryservicesbackupautoprotection) de PowerShell.

```powershell
$SQLInstance = Get-AzRecoveryServicesBackupProtectableItem -workloadType MSSQL -ItemType SQLInstance -VaultId $targetVault.ID -Name "<Protectable Item name>" -ServerName "<Server Name>"
Disable-AzRecoveryServicesBackupAutoProtection -InputItem $SQLInstance -BackupManagementType AzureWorkload -WorkloadType MSSQL -VaultId $targetvault.ID
```

#### <a name="unregister-sql-vm"></a>Anulación del registro de la máquina virtual SQL

Si todas las bases de datos de SQL Server [ya no están protegidas y no hay ninguna copia de seguridad](#delete-backup-data), puede anular el registro de la máquina virtual SQL desde este almacén. Solo entonces, podrá proteger las bases de datos en otro almacén. Use el cmdlet [Unregister-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/unregister-azrecoveryservicesbackupcontainer) de PowerShell para registrar la máquina virtual SQL.

```powershell
$SQLContainer = Get-AzRecoveryServicesBackupContainer -ContainerType AzureVMAppContainer -FriendlyName <VM name> -VaultId $targetvault.ID
 Unregister-AzRecoveryServicesBackupContainer -Container $SQLContainer -VaultId $targetvault.ID
```

### <a name="track-azure-backup-jobs"></a>Seguimiento de los trabajos de Azure Backup

Es importante tener en cuenta que Azure Backup solo realiza el seguimiento de los trabajos de usuario que se desencadenan en la copia de seguridad SQL. Las copias de seguridad programadas (incluidas las del registro) no están visibles en el portal ni en Powershell. Sin embargo, si alguno de los trabajos programados produce un error, se genera una [alerta de copia de seguridad](backup-azure-monitoring-built-in-monitor.md#backup-alerts-in-recovery-services-vault) y se muestra en el portal. [Use Azure Monitor](backup-azure-monitoring-use-azuremonitor.md) para realizar un seguimiento de todos los trabajos programados y otra información pertinente.

Los usuarios pueden seguir las operaciones desencadenadas a petición o de usuario con el identificador JobID devuelto en la [salida](#on-demand-backup) de los trabajos asincrónicos, como la copia de seguridad. Use el cmdlet [Get-AzRecoveryServicesBackupJobDetail](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjobdetail) de PowerShell para realizar un seguimiento del trabajo y sus detalles.

```powershell
 Get-AzRecoveryServicesBackupJobDetails -JobId 2516bb1a-d3ef-4841-97a3-9ba455fb0637 -VaultId $targetVault.ID
```

Para obtener la lista de los trabajos a petición y sus estados del servicio Azure Backup, use el cmdlet [Get- AzRecoveryServicesBackupJob](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupjob) de PowerShell. El ejemplo siguiente devuelve todos los trabajos SQL en curso.

```powershell
Get-AzRecoveryServicesBackupJob -Status InProgress -BackupManagementType AzureWorkload
```

Para cancelar un trabajo en curso, utilice el cmdlet [Stop-AzRecoveryServicesBackupJob](/powershell/module/az.recoveryservices/stop-azrecoveryservicesbackupjob) de PowerShell.

## <a name="managing-sql-always-on-availability-groups"></a>Administración de los grupos de disponibilidad AlwaysOn de SQL

Con los grupos de disponibilidad AlwaysOn de SQL, no olvide [registrar todos los nodos](#registering-the-sql-vm) del grupo de disponibilidad (AG). Una vez que el registro se realiza para todos los nodos, se crea lógicamente un objeto de grupo de disponibilidad SQL en los elementos que se pueden proteger. Las bases de datos del grupo de disponibilidad de SQL se mostrarán como "SQLDatabase". Los nodos se mostrarán como instancias independientes y las bases de datos SQL predeterminadas en ellos se mostrarán también como las bases de datos SQL.

Por ejemplo, supongamos que un grupo de disponibilidad SQL tiene dos nodos: *sql-server-0* y *sql-server-1*, y una base de datos SQL AG. Una vez que ambos nodos se registran, si [enumera los elementos que se pueden proteger](#fetching-sql-dbs), se muestran los siguientes componentes.

* Un objeto de grupo de disponibilidad SQL: tipo de elemento que se puede proteger como SQLAvailabilityGroup
* Una base de datos de grupo de disponibilidad de SQL: tipo de elemento que se puede proteger como SQLDatabase
* SQL-server-0: tipo de elemento que se puede proteger como SQLInstance
* SQL-server-1: tipo de elemento que se puede proteger como SQLInstance
* Cualquier valor predeterminado de base de datos SQL (master, model, msdb) en sql-server-0: tipo de elemento que se puede proteger como SQLDatabase
* Cualquier valor predeterminado de base de datos SQL (master, model, msdb) en sql-server-0: tipo de elemento que se puede proteger como SQLDatabase

sql-server-0, sql-server-1 también se mostrará como "AzureVMAppContainer" cuando [se enumeren los contenedores de copia de seguridad](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupcontainer).

Obtenga tan solo la base de datos pertinente para [habilitar la copia de seguridad](#configuring-backup) y la [copia de seguridad a petición](#on-demand-backup) y los [cmdlets de PowerShell de restauración](#restore-sql-dbs) son idénticos.
