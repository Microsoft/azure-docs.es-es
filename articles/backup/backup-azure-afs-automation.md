---
title: Copia de seguridad de un recurso compartido de archivos de Azure mediante PowerShell
description: En este artículo aprenderá a realizar copias de seguridad de un recurso compartido de Azure Files mediante el servicio Azure Backup y PowerShell.
ms.topic: conceptual
ms.date: 08/20/2019
ms.openlocfilehash: 948931764769bc967b88e7942b7e8384b0f93dff
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077008"
---
# <a name="back-up-an-azure-file-share-by-using-powershell"></a>Copia de seguridad de un recurso compartido de archivos de Azure mediante PowerShell

En este artículo se describe cómo usar Azure PowerShell para realizar copias de seguridad de un recurso compartido de archivos de Azure Files mediante el almacén de Recovery Services de [Azure Backup](backup-overview.md).

En este artículo se explica lo siguiente:

> [!div class="checklist"]
>
> * Configure PowerShell y registre el proveedor de Recovery Services.
> * Cree un almacén de Recovery Services.
> * Configurar la copia de seguridad de un recurso compartido de archivos de Azure.
> * Ejecutar un trabajo de copia de seguridad.

## <a name="before-you-start"></a>Antes de comenzar

* [Más información](backup-azure-recovery-services-vault-overview.md) sobre los almacenes de Recovery Services.
* Revise la [referencia de cmdlet](/powershell/module/az.recoveryservices) de Az.RecoveryServices en la biblioteca de Azure.
* Revise la siguiente jerarquía de objetos de PowerShell para Recovery Services:

  ![Jerarquía de objetos de Recovery Services](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

## <a name="set-up-powershell"></a>Configurar PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Configure PowerShell como sigue:

1. [Descargue la versión más reciente de Azure PowerShell](/powershell/azure/install-az-ps).

    > [!NOTE]
    > La versión mínima de PowerShell necesaria para realizar la copia de seguridad de recursos compartidos de archivos de Azure es Az.RecoveryServices 2.6.0. Si usa la versión más reciente, o al menos la versión mínima, evitará tener problemas con los scripts existentes. Instale la versión mínima mediante el siguiente comando de PowerShell:
    >
    > ```powershell
    > Install-module -Name Az.RecoveryServices -RequiredVersion 2.6.0
    > ```

2. Busque los cmdlets de PowerShell de Azure Backup con este comando:

    ```powershell
    Get-Command *azrecoveryservices*
    ```

3. Revise los alias y cmdlets de Azure Backup, Azure Site Recovery y el almacén de Recovery Services. Este es un ejemplo de lo que puede ver. No es la lista completa de los cmdlets.

    ![Lista de cmdlets de Recovery Services](./media/backup-azure-afs-automation/list-of-recoveryservices-ps-az.png)

4. Inicie sesión en su cuenta de Azure mediante el cmdlet **Connect-AzAccount**.
5. En la página web que aparece, se le pedirá que escriba las credenciales de su cuenta.

    Como alternativa, puede incluir las credenciales de la cuenta como un parámetro en el cmdlet **Connect-AzAccount** mediante **-Credential**.

    Si es un asociado de CSP que trabaja en nombre de un inquilino, especifique el cliente como inquilino. Use su id. de inquilino o el nombre de dominio principal del inquilino. Por ejemplo, **Connect-AzAccount -Tenant "fabrikam.com"** .

6. Debe asociar la suscripción que quiere usar con la cuenta, ya que una cuenta puede tener varias suscripciones:

    ```powershell
    Select-AzSubscription -SubscriptionName $SubscriptionName
    ```

7. Si usa Azure Backup por primera vez, use el cmdlet **Register-AzResourceProvider** para registrar el proveedor de Azure Recovery Services con la suscripción:

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

8. Compruebe que los proveedores se han registrado correctamente:

    ```powershell
    Get-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

9. En la salida del comando, compruebe que **RegistrationState** cambia a **Registered**. En caso contrario, vuelva a ejecutar el cmdlet **Register-AzResourceProvider**.

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

El almacén de Recovery Services es un recurso de Resource Manager, por lo que deberá colocarlo en un grupo de recursos. Puede usar un grupo de recursos existente o crear uno mediante el cmdlet **New-AzResourceGroup**. Al crear un grupo de recursos, especifique el nombre y la ubicación del mismo.

Siga estos pasos para crear un almacén de Recovery Services:

1. Si no tiene un grupo de recursos, cree uno con el cmdlet [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo, se crea un grupo de recursos en la región Oeste de EE. UU:

   ```powershell
   New-AzResourceGroup -Name "test-rg" -Location "West US"
   ```

1. Use el cmdlet [New-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/new-azrecoveryservicesvault) para crear el almacén. Especifique la misma ubicación para el almacén que usó en el grupo de recursos.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName "test-rg" -Location "West US"
    ```

### <a name="view-the-vaults-in-a-subscription"></a>Visualización de los almacenes de una suscripción

Para ver todos los almacenes de la suscripción, use [Get-AzRecoveryServicesVault](/powershell/module/az.recoveryservices/get-azrecoveryservicesvault):

```powershell
Get-AzRecoveryServicesVault
```

La salida será similar a la siguiente. Tenga en cuenta que la salida proporciona el grupo de recursos y la ubicación asociados.

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

Almacene el objeto de almacén en una variable y establezca el contexto de almacén.

Muchos de los cmdlets de Azure Backup requieren el objeto de almacén de Recovery Services como una entrada, por lo que es conveniente almacenar el objeto de almacén en una variable.

El contexto de almacén es el tipo de los datos protegidos en el almacén. Establézcalo mediante [Set-AzRecoveryServicesVaultContext](/powershell/module/az.recoveryservices/set-azrecoveryservicesvaultcontext). Una vez que se haya establecido el contexto, se aplica a todos los cmdlets posteriores.

En el ejemplo siguiente se establece el contexto del almacén de **testvault**:

```powershell
Get-AzRecoveryServicesVault -Name "testvault" | Set-AzRecoveryServicesVaultContext
```

### <a name="fetch-the-vault-id"></a>Recuperación del identificador del almacén

Tenemos previsto dejar de usar la configuración del contexto de almacén según las directrices de Azure PowerShell. En su lugar, puede almacenar o recuperar el identificador del almacén y pasarlo a los comandos pertinentes. Por lo tanto, si no ha establecido el contexto de almacén o quiere especificar el comando que se va a ejecutar para un determinado almacén, pase el id. de almacén como `-vaultID` a todos los comandos pertinentes de la manera siguiente:

```powershell
$vaultID = Get-AzRecoveryServicesVault -ResourceGroupName "Contoso-docs-rg" -Name "testvault" | select -ExpandProperty ID
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewAFSPolicy" -WorkloadType "AzureFiles" -RetentionPolicy $retPol -SchedulePolicy $schPol -VaultID $vaultID
```

## <a name="configure-a-backup-policy"></a>Configuración de una directiva de copia de seguridad

Una directiva de copia de seguridad especifica la programación de las copias de seguridad y cuánto tiempo se deben mantener los puntos de recuperación de copia de seguridad.

Una directiva de copia de seguridad está asociada con al menos una directiva de retención. Una directiva de retención define el tiempo que se conserva un punto de recuperación antes de que se elimine. Puede configurar copias de seguridad con retención diaria, semanal, mensual o anual.

Estos son algunos cmdlets para las directivas de copia de seguridad:

* Consulte la retención de la directiva de copia de seguridad predeterminada mediante [Get-AzRecoveryServicesBackupRetentionPolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupretentionpolicyobject).
* Consulte la programación de la directiva de copia de seguridad predeterminada mediante [Get-AzRecoveryServicesBackupSchedulePolicyObject](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupschedulepolicyobject).
* Cree una directiva de copia de seguridad mediante [New AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/set-azrecoveryservicesbackupprotectionpolicy). Especifique los objetos de directiva de retención y programación como entradas.

De forma predeterminada, se define una hora de inicio en el objeto de directiva de programación. Use el siguiente ejemplo para cambiar la hora de inicio a la hora de inicio deseada. La hora de inicio deseada debe ser la hora universal coordinada (UTC). En el ejemplo siguiente se supone que la hora de inicio deseada es 01:00 UTC para las copias de seguridad diarias.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureFiles"
$UtcTime = Get-Date -Date "2019-03-20 01:30:00Z"
$UtcTime = $UtcTime.ToUniversalTime()
$schpol.ScheduleRunTimes[0] = $UtcTime
```

> [!IMPORTANT]
> Solo tiene que proporcionar la hora de inicio en múltiplos de 30 minutos. En el ejemplo anterior, solo puede ser "01:00:00" o "02:30:00". La hora de inicio no puede ser "01:15:00".

En el ejemplo siguiente se almacenan la directiva de programación y la directiva de retención en variables. Después usa estas variables como parámetros para una nueva directiva (**NewAFSPolicy**). **NewAFSPolicy** realiza una copia de seguridad diaria y la conserva durante 30 días.

```powershell
$schPol = Get-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureFiles"
$retPol = Get-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureFiles"
New-AzRecoveryServicesBackupProtectionPolicy -Name "NewAFSPolicy" -WorkloadType "AzureFiles" -RetentionPolicy $retPol -SchedulePolicy $schPol
```

La salida será similar a la siguiente:

```powershell
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
NewAFSPolicy           AzureFiles            AzureStorage              10/24/2019 1:30:00 AM
```

## <a name="enable-backup"></a>Habilitación de la copia de seguridad

Una vez que haya definido la directiva de copia de seguridad, puede habilitar la protección para el recurso compartido de archivos de Azure con esta directiva.

### <a name="retrieve-a-backup-policy"></a>Recuperación de una directiva de copia de seguridad

Recupere el objeto de directiva pertinente mediante [Get-AzRecoveryServicesBackupProtectionPolicy](/powershell/module/az.recoveryservices/get-azrecoveryservicesbackupprotectionpolicy). Use este cmdlet para obtener una directiva específica o para ver las directivas asociadas a un tipo de carga de trabajo.

#### <a name="retrieve-a-policy-for-a-workload-type"></a>Recuperación de una directiva para un tipo de carga de trabajo

En el ejemplo siguiente se recuperan las directivas para el tipo de carga de trabajo **AzureFiles**:

```powershell
Get-AzRecoveryServicesBackupProtectionPolicy -WorkloadType "AzureFiles"
```

La salida será similar a la siguiente:

```powershell
Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
----                 ------------       -------------------- ----------                ----------
dailyafs             AzureFiles         AzureStorage         1/10/2018 12:30:00 AM
```

> [!NOTE]
> La zona horaria del campo **BackupTime** en PowerShell es UTC. Cuando el tiempo de copia de seguridad se muestra en Azure Portal, la hora se ajusta a la zona horaria local.

#### <a name="retrieve-a-specific-policy"></a>Recuperación de una directiva específica

La directiva siguiente recupera la directiva de copia de seguridad llamada **dailyafs**:

```powershell
$afsPol =  Get-AzRecoveryServicesBackupProtectionPolicy -Name "dailyafs"
```

### <a name="enable-protection-and-apply-the-policy"></a>Habilitación de la protección y aplicación de la directiva

Habilite la protección mediante [Enable-AzRecoveryServicesBackupProtection](/powershell/module/az.recoveryservices/enable-azrecoveryservicesbackupprotection). Una vez que la directiva se ha asociado al almacén, las copias de seguridad se desencadenan según la programación de la directiva.

En el ejemplo siguiente se habilita la protección para el recurso compartido de archivos de Azure **testAzureFileShare**, en la cuenta de almacenamiento **testStorageAcct**, con la directiva **dailyafs**:

```powershell
Enable-AzRecoveryServicesBackupProtection -StorageAccountName "testStorageAcct" -Name "testAzureFS" -Policy $afsPol
```

El comando espera hasta que el trabajo de protección configurado termina y proporciona una salida similar, tal como se muestra en el siguiente ejemplo:

```cmd
WorkloadName       Operation            Status                 StartTime                                                                                                         EndTime                   JobID
------------             ---------            ------               ---------                                  -------                   -----
testAzureFS       ConfigureBackup      Completed            11/12/2018 2:15:26 PM     11/12/2018 2:16:11 PM     ec7d4f1d-40bd-46a4-9edb-3193c41f6bf6
```

Para más información sobre cómo obtener una lista de recursos compartidos de archivos para una cuenta de almacenamiento, consulte [este artículo](/powershell/module/az.storage/get-azstorageshare).

## <a name="important-notice-backup-item-identification"></a>Aviso importante: Identificación del elemento de copia de seguridad

En esta sección se describe un cambio importante en las copias de seguridad de recursos compartidos de archivos de Azure en el proceso de preparación para la disponibilidad general.

Al habilitar la copia de seguridad los recursos compartidos de archivos de Azure, el usuario proporciona al cliente el nombre del recurso compartido de archivos como nombre de la entidad; una vez hecho esto, se crea un elemento de copia de seguridad. El nombre del elemento de copia de seguridad es un identificador único que crea el servicio Azure Backup. Normalmente, el identificador es el nombre descriptivo del usuario. De todos modos, para controlar el escenario de eliminación temporal, donde se puede eliminar un recurso compartido de archivos y crear otro con el mismo nombre, la identidad única del recurso compartido de archivos de Azure ahora es un id.

Para conocer el id. único de cada elemento, ejecute el comando **Get-AzRecoveryServicesBackupItem** con los filtros correspondientes para **backupManagementType** y **WorkloadType** para obtener todos los elementos pertinentes. A continuación, observe el campo del nombre en la respuesta o el objeto de PowerShell devuelto.

Siempre se recomienda realizar una lista de los elementos y luego recuperar su nombre único del campo del nombre de la respuesta. Use este valor para filtrar los elementos con el parámetro *Name*. De lo contrario, use el parámetro *FriendlyName* para recuperar el elemento con su id.

> [!IMPORTANT]
> Asegúrese de que PowerShell se actualice a la versión mínima (AZ.RecoveryServices 2.6.0) para las copias de seguridad de recursos compartidos de archivos de Azure. Con esta versión, el filtro *friendlyName* está disponible para el comando **Get-AzRecoveryServicesBackupItem**.
>
> Pase el nombre del recurso compartido de archivos de Azure al parámetro *friendlyName*. Si pasa el nombre del recurso compartido de archivos de Azure al parámetro *Name*, esta versión genera la advertencia de pasar este nombre al parámetro *friendlyName*.
>
> Si no instala esta versión mínima pueden producirse errores en los scripts existentes. Instale la versión mínima de PowerShell mediante el siguiente comando:
>
>```powershell
>Install-module -Name Az.RecoveryServices -RequiredVersion 2.6.0
>```

## <a name="trigger-an-on-demand-backup"></a>Desencadenamiento de una copia de seguridad a petición

Use [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-azrecoveryservicesbackupitem) para ejecutar una copia de seguridad a petición para un recurso compartido de archivos protegido de Azure:

1. Recupere la cuenta de almacenamiento del contenedor del almacén que contenga los datos de copia de seguridad mediante [Get-AzRecoveryServicesBackupContainer](/powershell/module/az.recoveryservices/get-Azrecoveryservicesbackupcontainer).
2. Para iniciar un trabajo de copia de seguridad, obtenga información sobre el recurso compartido de archivos de Azure mediante [Get-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/Get-AzRecoveryServicesBackupItem).
3. Ejecute una copia de seguridad a petición mediante [Backup-AzRecoveryServicesBackupItem](/powershell/module/az.recoveryservices/backup-Azrecoveryservicesbackupitem).

Ejecute la copia de seguridad a petición de la manera siguiente:

```powershell
$afsContainer = Get-AzRecoveryServicesBackupContainer -FriendlyName "testStorageAcct" -ContainerType AzureStorage
$afsBkpItem = Get-AzRecoveryServicesBackupItem -Container $afsContainer -WorkloadType "AzureFiles" -FriendlyName "testAzureFS"
$job =  Backup-AzRecoveryServicesBackupItem -Item $afsBkpItem
```

El comando devuelve un trabajo con un id. del cual se puede realizar un seguimiento, tal como se indica en el ejemplo siguiente:

```powershell
WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
------------     ---------            ------               ---------                 -------                   -----
testAzureFS       Backup               Completed            11/12/2018 2:42:07 PM     11/12/2018 2:42:11 PM     8bdfe3ab-9bf7-4be6-83d6-37ff1ca13ab6
```

Las instantáneas del recurso compartido de archivos de Azure se usan mientras se realizan las copias de seguridad. Normalmente, el trabajo finaliza en el momento en que el comando devuelve este resultado.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo [realizar copias de seguridad de archivos de Azure Files en Azure Portal](backup-afs.md).
* Consulte el [script de ejemplo en GitHub](https://github.com/Azure-Samples/Use-PowerShell-for-long-term-retention-of-Azure-Files-Backup) para usar un runbook de Azure Automation para programar copias de seguridad.
