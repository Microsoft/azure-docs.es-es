---
title: Uso de PowerShell para hacer una copia de seguridad de Windows Server en Azure
description: En este artículo aprenderá a usar PowerShell para configurar Azure Backup en Windows Server o una versión cliente de Windows y para administrar copias de seguridad y recuperaciones.
ms.topic: conceptual
ms.date: 12/2/2019
ms.openlocfilehash: 0deccc49b82d4a8b81889c35174c3efa81b6d74d
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87564034"
---
# <a name="deploy-and-manage-backup-to-azure-for-windows-serverwindows-client-using-powershell"></a>Implementación y administración de copias de seguridad en Azure para Windows Server o cliente de Windows mediante PowerShell

En este artículo se muestra cómo usar PowerShell para configurar Azure Backup en Windows Server o una versión cliente de Windows y para administrar copias de seguridad y recuperaciones.

## <a name="install-azure-powershell"></a>Instalar Azure Powershell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para empezar, [instale la versión más reciente de PowerShell](/powershell/azure/install-az-ps).

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Servicios de recuperación

Los siguientes pasos le guiarán por el proceso de creación de un almacén de Recovery Services. Un almacén de Recovery Services no es lo mismo que un almacén de copia de seguridad.

1. Si usa Azure Backup por primera vez, debe usar el cmdlet **Register-AzResourceProvider** para registrar el proveedor de Azure Recovery Services con su suscripción.

    ```powershell
    Register-AzResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"
    ```

2. El almacén de Recovery Services es un recurso de Azure Resource Manager, por lo que deberá colocarlo dentro de un grupo de recursos. Puede usar un grupo de recursos existente o crear uno nuevo. Al crear un nuevo grupo de recursos, especifique su nombre y su ubicación.  

    ```powershell
    New-AzResourceGroup –Name "test-rg" –Location "WestUS"
    ```

3. Use el cmdlet **New-AzRecoveryServicesVault** para crear el nuevo almacén. Asegúrese de especificar para el almacén la misma ubicación del grupo de recursos.

    ```powershell
    New-AzRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "WestUS"
    ```

4. Especifique el tipo de redundancia de almacenamiento que se va a usar: [almacenamiento con redundancia local (LRS)](../storage/common/storage-redundancy.md) o [almacenamiento con redundancia geográfica (GRS)](../storage/common/storage-redundancy.md). En el ejemplo siguiente se muestra que la opción **-BackupStorageRedundancy** para *testVault* está establecida en **GeoRedundant**.

   > [!TIP]
   > Muchos de los cmdlets de Azure Backup requieren el objeto de almacén de Recovery Services como entrada. Por este motivo, es conveniente almacenar el objeto de almacén de Recovery Services de Backup en una variable.
   >
   >

    ```powershell
    $Vault1 = Get-AzRecoveryServicesVault –Name "testVault"
    Set-AzRecoveryServicesBackupProperties -Vault $Vault1 -BackupStorageRedundancy GeoRedundant
    ```

## <a name="view-the-vaults-in-a-subscription"></a>Visualización de los almacenes de una suscripción

Use **Get-AzRecoveryServicesVault** para ver la lista de todos los almacenes de la suscripción actual. Este comando se puede usar para comprobar que se ha creado un almacén o para ver cuáles almacenes están disponibles en la suscripción.

Ejecute el comando **Get-AzRecoveryServicesVault** y se mostrarán todos los almacenes de la suscripción.

```powershell
Get-AzRecoveryServicesVault
```

```Output
Name              : Contoso-vault
ID                : /subscriptions/1234
Type              : Microsoft.RecoveryServices/vaults
Location          : WestUS
ResourceGroupName : Contoso-docs-rg
SubscriptionId    : 1234-567f-8910-abc
Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties
```

[!INCLUDE [backup-upgrade-mars-agent.md](../../includes/backup-upgrade-mars-agent.md)]

## <a name="installing-the-azure-backup-agent"></a>Instalación del agente de Azure Backup

Antes de instalar el agente de Azure Backup, necesitará tener el instalador descargado y disponible en el servidor de Windows. Puede obtener la versión más reciente del instalador en el [Centro de descarga de Microsoft](https://aka.ms/azurebackup_agent) o en la página Panel del almacén de Recovery Services. Guarde el instalador en una ubicación que tenga fácil acceso, como `C:\Downloads\*`.

Como alternativa, use PowerShell para obtener la aplicación de descarga:

 ```powershell
 $MarsAURL = 'https://aka.ms/Azurebackup_Agent'
 $WC = New-Object System.Net.WebClient
 $WC.DownloadFile($MarsAURL,'C:\downloads\MARSAgentInstaller.EXE')
 C:\Downloads\MARSAgentInstaller.EXE /q
 ```

Para instalar el agente, ejecute el comando siguiente en una consola de PowerShell con privilegios elevados:

```powershell
MARSAgentInstaller.exe /q
```

Esto instala el agente con todas las opciones predeterminadas. La instalación está unos minutos en segundo plano. Si no se especifica la opción */nu*, se abrirá la ventana de **Windows Update** al final de la instalación para comprobar si hay actualizaciones. Una vez instalado, el agente se mostrará en la lista de programas instalados.

Para ver la lista de programas instalados, vaya a **Panel de Control** > **Programas** > **Programas y características**.

![Agente instalado](./media/backup-client-automation/installed-agent-listing.png)

### <a name="installation-options"></a>Opción de instalación

Para ver todas las opciones disponibles a través de la línea de comandos, use el siguiente comando:

```powershell
MARSAgentInstaller.exe /?
```

Las opciones disponibles incluyen:

| Opción | Detalles | Valor predeterminado |
| --- | --- | --- |
| /q |Instalación silenciosa |- |
| /p:"ubicación" |Ruta de acceso a la carpeta de instalación para el agente de Azure Backup. |C:\Archivos de programa\Microsoft Azure Recovery Services Agent |
| /s:"ubicación" |Ruta de acceso a la carpeta de caché para el agente de Azure Backup. |C:\Archivos de programa\Microsoft Azure Recovery Services Agent\Scratch |
| /m |Participación en Microsoft Update |- |
| /nu |No busca actualizaciones una vez completada la instalación |- |
| /d |Desinstala el agente de Microsoft Azure Recovery Services. |- |
| /ph |Dirección host del proxy |- |
| /po |Número de puerto del host de proxy |- |
| /pu |Nombre de usuario del host de proxy |- |
| /pw |Contraseña de proxy |- |

## <a name="registering-windows-server-or-windows-client-machine-to-a-recovery-services-vault"></a>Registro de Windows Server o el equipo cliente de Windows en un almacén de Recovery Services

Después de crear el almacén de Recovery Services, descargue el agente más reciente y las credenciales de almacén y guárdelas en una ubicación adecuada como C:\Downloads.

```powershell
$CredsPath = "C:\downloads"
$CredsFilename = Get-AzRecoveryServicesVaultSettingsFile -Backup -Vault $Vault1 -Path $CredsPath
```

### <a name="registering-using-the-ps-az-module"></a>Registro mediante el módulo Az de PS

> [!NOTE]
> En la versión Az 3.5.0, se corrige un error con la generación del certificado de almacén. Use la versión Az 3.5.0 o posterior para descargar un certificado de almacén.

En el módulo Az más reciente de PowerShell, la descarga de las credenciales del almacén requiere un certificado autofirmado debido a las limitaciones de la plataforma subyacente. En el ejemplo siguiente se muestra cómo proporcionar un certificado autofirmado y descargar las credenciales del almacén.

```powershell
$dt = $(Get-Date).ToString("M-d-yyyy")
$cert = New-SelfSignedCertificate -CertStoreLocation Cert:\CurrentUser\My -FriendlyName 'test-vaultcredentials' -subject "Windows Azure Tools" -KeyExportPolicy Exportable -NotAfter $(Get-Date).AddHours(48) -NotBefore $(Get-Date).AddHours(-24) -KeyProtection None -KeyUsage None -TextExtension @("2.5.29.37={text}1.3.6.1.5.5.7.3.2") -Provider "Microsoft Enhanced Cryptographic Provider v1.0"
$certficate = [convert]::ToBase64String($cert.Export([System.Security.Cryptography.X509Certificates.X509ContentType]::Pfx))
$CredsFilename = Get-AzRecoveryServicesVaultSettingsFile -Backup -Vault $Vault -Path $CredsPath -Certificate $certficate
```

En el sistema con Windows Server o el equipo cliente de Windows, ejecute el cmdlet [Start-OBRegistration](/powershell/module/msonlinebackup/start-obregistration) para registrar la máquina en el almacén.
Este y otros cmdlets usados para copias de seguridad son del módulo MSONLINE que el instalador del agente de MARS agregó como parte del proceso de instalación.

El instalador del agente no actualiza la variable $Env:PSModulePath. Esto significa que se produce un error en la carga automática del módulo. Para resolver este problema, puede hacer lo siguiente:

```powershell
$Env:PSModulePath += ';C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules'
```

Como alternativa, puede cargar manualmente el módulo en el script como se indica a continuación:

```powershell
Import-Module -Name 'C:\Program Files\Microsoft Azure Recovery Services Agent\bin\Modules\MSOnlineBackup'
```

Después de cargar los cmdlets de Online Backup, registre las credenciales del almacén:

```powershell
Start-OBRegistration -VaultCredentials $CredsFilename.FilePath -Confirm:$false
```

```Output
CertThumbprint      : 7a2ef2caa2e74b6ed1222a5e89288ddad438df2
SubscriptionID      : ef4ab577-c2c0-43e4-af80-af49f485f3d1
ServiceResourceName : testvault
Region              : WestUS
Machine registration succeeded.
```

> [!IMPORTANT]
> No use rutas de acceso relativas para especificar el archivo de credenciales del almacén de claves. Debe proporcionar una ruta de acceso absoluta como entrada para el cmdlet.
>
>

## <a name="networking-settings"></a>Configuración de redes

Cuando la conectividad de la máquina de Windows a Internet se realiza a través de un servidor proxy, también se puede proporcionar la configuración del proxy al agente. En este ejemplo, no hay ningún servidor proxy, por lo que se borra explícitamente cualquier información relacionada con el proxy.

También puede controlar el uso de ancho de banda con las opciones de `work hour bandwidth` y `non-work hour bandwidth` para un conjunto determinado de días de la semana.

El establecimiento de los detalles de ancho de banda y del proxy se realiza mediante el cmdlet [Set-OBMachineSetting](/powershell/module/msonlinebackup/set-obmachinesetting) :

```powershell
Set-OBMachineSetting -NoProxy
```

```Output
Server properties updated successfully.
```

```powershell
Set-OBMachineSetting -NoThrottle
```

```Output
Server properties updated successfully.
```

## <a name="encryption-settings"></a>Configuración de cifrado

Los datos de copia de seguridad enviados a Azure Backup se cifran para proteger la confidencialidad de los datos. La frase de contraseña de cifrado es la "contraseña" que permite descifrar los datos en el momento de la restauración.

Debe generar un PIN de seguridad seleccionando **Generar** en **Configuración** > **Propiedades** > **PIN de seguridad** en la sección **Almacén de Recovery Services**.

>[!NOTE]
> El PIN de seguridad solo se puede generar desde Azure Portal.

A continuación, úselo como el `generatedPIN` en el comando:

```powershell
$PassPhrase = ConvertTo-SecureString -String "Complex!123_STRING" -AsPlainText -Force
Set-OBMachineSetting -EncryptionPassPhrase $PassPhrase -SecurityPin "<generatedPIN>"
```

```Output
Server properties updated successfully
```

> [!IMPORTANT]
> Mantenga la información de la frase de contraseña segura una vez establecida. No puede restaurar los datos de Azure sin esta frase de contraseña.
>
>

## <a name="back-up-files-and-folders"></a>Realizar copias de seguridad de archivos y carpetas

Todas las copias de seguridad de servidores y clientes de Windows en Azure Backup se rigen por una directiva. La directiva comprende tres partes:

1. Un **programa de copia de seguridad** que especifica cuándo deben efectuarse y sincronizarse las copias de seguridad con el servicio.
2. Una **programación de retención** que especifica cuánto tiempo se conservarán los puntos de recuperación en Azure.
3. Una **especificación de inclusión o exclusión de archivo** que determina los elementos de los que se debe efectuar una copia de seguridad.

En este documento, dado que se está automatizando la copia de seguridad, supondremos que no se ha configurado nada. Comenzamos creando una nueva directiva de copia de seguridad mediante el cmdlet [New-OBPolicy](/powershell/module/msonlinebackup/new-obpolicy) .

```powershell
$NewPolicy = New-OBPolicy
```

En este momento la directiva está vacía y se necesitan otros cmdlet para definir qué elementos se incluirán o excluirán, cuándo se ejecutarán las copias de seguridad y dónde se almacenarán las copias de seguridad.

### <a name="configuring-the-backup-schedule"></a>Configuración de la programación de la copia de seguridad

La primera de las tres partes de una directiva es la programación de copia de seguridad, que se crea usando el cmdlet [New-OBSchedule](/powershell/module/msonlinebackup/new-obschedule). La programación de copia de seguridad define cuándo deben realizarse copias de seguridad. Al crear una programación se deben especificar dos parámetros de entrada:

* **Días de la semana** en los que se debe ejecutar la copia de seguridad. Puede ejecutar el trabajo de copia de seguridad en solo un día o cada día de la semana, o cualquier combinación intermedia.
* **Horas del día** a las que se debe ejecutar la copia de seguridad. Puede definir hasta tres horas distintas del día en las que se activará la copia de seguridad.

Por ejemplo, podría configurar una directiva de copia de seguridad que se ejecute a las 4 p.m. cada sábado y domingo.

```powershell
$Schedule = New-OBSchedule -DaysOfWeek Saturday, Sunday -TimesOfDay 16:00
```

La programación de copia de seguridad debe estar asociada con una directiva y esto puede lograrse mediante el uso del cmdlet [Set-OBSchedule](/powershell/module/msonlinebackup/set-obschedule) .

```powershell
Set-OBSchedule -Policy $NewPolicy -Schedule $Schedule
```

```Output
BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s) DsList : PolicyName : RetentionPolicy : State : New PolicyState : Valid
```

### <a name="configuring-a-retention-policy"></a>Configuración de una directiva de retención

La directiva de retención define cuánto tiempo se conservan los puntos de recuperación de los trabajos de copia de seguridad. Al crear una nueva directiva de retención mediante el cmdlet [New-OBRetentionPolicy](/powershell/module/msonlinebackup/new-obretentionpolicy), puede especificar el número de días que se conservarán los puntos de recuperación de copia de seguridad con Azure Backup. En el ejemplo siguiente se establece una directiva de retención de siete días.

```powershell
$RetentionPolicy = New-OBRetentionPolicy -RetentionDays 7
```

La directiva de retención debe estar asociada con la directiva principal mediante el cmdlet [Set-OBRetentionPolicy](/powershell/module/msonlinebackup/set-obretentionpolicy):

```powershell
Set-OBRetentionPolicy -Policy $NewPolicy -RetentionPolicy $RetentionPolicy
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          :
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

### <a name="including-and-excluding-files-to-be-backed-up"></a>Incluir y excluir archivos de copia de seguridad

Un objeto `OBFileSpec` define los archivos incluidos y excluidos de una copia de seguridad. Se trata de un conjunto de reglas de ámbito de los archivos y carpetas protegidos de un equipo. Puede tener tantas reglas de inclusión o exclusión de archivos como se necesiten y asociarlas con una directiva. Al crear un nuevo objeto OBFileSpec, puede:

* Especificar los archivos y carpetas que se van a incluir
* Especificar los archivos y carpetas que se van a excluir
* Especificar la copia de seguridad recursiva de los datos en una carpeta (o) si se deben copiar solo los archivos de nivel superior en la carpeta especificada.

Esto último se consigue mediante la marca -NonRecursive del comando New-OBFileSpec.

En el ejemplo siguiente, crearemos copias de seguridad del volumen C: y D: y excluiremos los archivos binarios de sistema operativo de la carpeta de Windows y las carpetas temporales. Para ello, crearemos dos especificaciones de archivos mediante el cmdlet [New-OBFileSpec](/powershell/module/msonlinebackup/new-obfilespec): uno para la inclusión y otro para la exclusión. Una vez que se hayan creado las especificaciones de archivo, se asocian con la directiva mediante el cmdlet [Add-OBFileSpec](/powershell/module/msonlinebackup/add-obfilespec) .

```powershell
$Inclusions = New-OBFileSpec -FileSpec @("C:\", "D:\")
$Exclusions = New-OBFileSpec -FileSpec @("C:\windows", "C:\temp") -Exclude
Add-OBFileSpec -Policy $NewPolicy -FileSpec $Inclusions
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          : {DataSource
                  DatasourceId:0
                  Name:C:\
                  FileSpec:FileSpec
                  FileSpec:C:\
                  IsExclude:False
                  IsRecursive:True

                  , DataSource
                  DatasourceId:0
                  Name:D:\
                  FileSpec:FileSpec
                  FileSpec:D:\
                  IsExclude:False
                  IsRecursive:True

                  }
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

```powershell
Add-OBFileSpec -Policy $NewPolicy -FileSpec $Exclusions
```

```Output
BackupSchedule  : 4:00 PM
                  Saturday, Sunday,
                  Every 1 week(s)
DsList          : {DataSource
                  DatasourceId:0
                  Name:C:\
                  FileSpec:FileSpec
                  FileSpec:C:\
                  IsExclude:False
                  IsRecursive:True
                  ,FileSpec
                  FileSpec:C:\windows
                  IsExclude:True
                  IsRecursive:True
                  ,FileSpec
                  FileSpec:C:\temp
                  IsExclude:True
                  IsRecursive:True

                  , DataSource
                  DatasourceId:0
                  Name:D:\
                  FileSpec:FileSpec
                  FileSpec:D:\
                  IsExclude:False
                  IsRecursive:True

                  }
PolicyName      :
RetentionPolicy : Retention Days : 7

                  WeeklyLTRSchedule :
                  Weekly schedule is not set

                  MonthlyLTRSchedule :
                  Monthly schedule is not set

                  YearlyLTRSchedule :
                  Yearly schedule is not set

State           : New
PolicyState     : Valid
```

### <a name="applying-the-policy"></a>Aplicación de la directiva

Ahora el objeto de la directiva está finalizado y tiene una programación de copia de seguridad asociada, una directiva de retención y una lista de inclusión o exclusión de archivos. Ahora se puede confirmar esta directiva para ser usada por Azure Backup. Antes de aplicar la directiva recién creada, asegúrese de que no haya ninguna directiva de copia de seguridad existente asociada con el servidor mediante el uso del cmdlet [Remove-OBPolicy](/powershell/module/msonlinebackup/remove-obpolicy). Para eliminar la directiva, se le pedirá confirmación. Para omitir el uso de la confirmación, use la marca `-Confirm:$false` con el cmdlet.

```powershell
Get-OBPolicy | Remove-OBPolicy
```

```Output
Microsoft Azure Backup Are you sure you want to remove this backup policy? This will delete all the backed up data. [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
```

La confirmación del objeto de la directiva se lleva a cabo usando el cmdlet [Set-OBPolicy](/powershell/module/msonlinebackup/set-obpolicy) . Esto también requerirá confirmación. Para omitir el uso de la confirmación, use la marca `-Confirm:$false` con el cmdlet.

```powershell
Set-OBPolicy -Policy $NewPolicy
```

```Output
Microsoft Azure Backup Do you want to save this backup policy ? [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s)
DsList : {DataSource
         DatasourceId:4508156004108672185
         Name:C:\
         FileSpec:FileSpec
         FileSpec:C:\
         IsExclude:False
         IsRecursive:True,

         FileSpec
         FileSpec:C:\windows
         IsExclude:True
         IsRecursive:True,

         FileSpec
         FileSpec:C:\temp
         IsExclude:True
         IsRecursive:True,

         DataSource
         DatasourceId:4508156005178868542
         Name:D:\
         FileSpec:FileSpec
         FileSpec:D:\
         IsExclude:False
         IsRecursive:True
    }
PolicyName : c2eb6568-8a06-49f4-a20e-3019ae411bac
RetentionPolicy : Retention Days : 7
              WeeklyLTRSchedule :
              Weekly schedule is not set

              MonthlyLTRSchedule :
              Monthly schedule is not set

              YearlyLTRSchedule :
              Yearly schedule is not set
State : Existing PolicyState : Valid
```

Puede ver los detalles de la directiva de copia de seguridad existente con el cmdlet [Get-OBPolicy](/powershell/module/msonlinebackup/get-obpolicy) . Puede profundizar más mediante el cmdlet [Get-OBSchedule](/powershell/module/msonlinebackup/get-obschedule) para la programación de copia de seguridad y el cmdlet [Get-OBRetentionPolicy](/powershell/module/msonlinebackup/get-obretentionpolicy) para las directivas de retención.

```powershell
Get-OBPolicy | Get-OBSchedule
```

```Output
SchedulePolicyName : 71944081-9950-4f7e-841d-32f0a0a1359a
ScheduleRunDays : {Saturday, Sunday}
ScheduleRunTimes : {16:00:00}
State : Existing
```

```powershell
Get-OBPolicy | Get-OBRetentionPolicy
```

```Output
RetentionDays : 7
RetentionPolicyName : ca3574ec-8331-46fd-a605-c01743a5265e
State : Existing
```

```powershell
Get-OBPolicy | Get-OBFileSpec
```

```Output
FileName : *
FilePath : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\
FileSpec : D:\
IsExclude : False
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\
FileSpec : C:\
IsExclude : False
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\windows
FileSpec : C:\windows
IsExclude : True
IsRecursive : True

FileName : *
FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\temp
FileSpec : C:\temp
IsExclude : True
IsRecursive : True
```

### <a name="performing-an-on-demand-backup"></a>Realización de una copia de seguridad a petición

Una vez establecida una directiva de copia de seguridad, las copias de seguridad se producirán en función de la programación. También es posible desencadenar una copia de seguridad a petición mediante el cmdlet [Start-OBBackup](/powershell/module/msonlinebackup/start-obbackup):

```powershell
Get-OBPolicy | Start-OBBackup
```

```Output
Initializing
Taking snapshot of volumes...
Preparing storage...
Generating backup metadata information and preparing the metadata VHD...
Data transfer is in progress. It might take longer since it is the first backup and all data needs to be transferred...
Data transfer completed and all backed up data is in the cloud. Verifying data integrity...
Data transfer completed
In progress...
Job completed.
The backup operation completed successfully.
```

## <a name="back-up-windows-server-system-state-in-mars-agent"></a>Copia de seguridad del estado del sistema de Windows Server en el agente de MARS

En esta sección se describe el comando de PowerShell para configurar el estado del sistema en el agente de MARS.

### <a name="schedule"></a>Programación

```powershell
$sched = New-OBSchedule -DaysOfWeek Sunday,Monday,Tuesday,Wednesday,Thursday,Friday,Saturday -TimesOfDay 2:00
```

### <a name="retention"></a>Retención

```powershell
$rtn = New-OBRetentionPolicy -RetentionDays 32 -RetentionWeeklyPolicy -RetentionWeeks 13 -WeekDaysOfWeek Sunday -WeekTimesOfDay 2:00  -RetentionMonthlyPolicy -RetentionMonths 13 -MonthDaysOfMonth 1 -MonthTimesOfDay 2:00
```

### <a name="configuring-schedule-and-retention"></a>Configuración de programación y retención

```powershell
New-OBPolicy | Add-OBSystemState |  Set-OBRetentionPolicy -RetentionPolicy $rtn | Set-OBSchedule -Schedule $sched | Set-OBSystemStatePolicy
 ```

### <a name="verifying-the-policy"></a>Comprobación de la directiva

```powershell
Get-OBSystemStatePolicy
 ```

## <a name="restore-data-from-azure-backup"></a>Restaurar datos de Azure Backup

Esta sección le guiará por los pasos necesarios para automatizar la recuperación de datos de Azure Backup. Esto implica los pasos siguientes:

1. Seleccionar el volumen de origen
2. Elegir un punto de copia de seguridad desde el que efectuar la restauración
3. Especificar un elemento para restaurar
4. Desencadenar el proceso de restauración

### <a name="picking-the-source-volume"></a>Selección del volumen de origen

Para restaurar un elemento de Azure Backup, primero deberá identificar el origen del elemento. Dado que los comandos se están ejecutando en el contexto de un servidor o un cliente de Windows, el equipo ya se ha identificado. El siguiente paso para identificar el origen es identificar el volumen que lo contiene. Se puede recuperar una lista de los volúmenes u orígenes de los que se está efectuando una copia de seguridad desde esta máquina mediante la ejecución del cmdlet [Get-OBRecoverableSource](/powershell/module/msonlinebackup/get-obrecoverablesource) . Este comando devuelve una matriz de todos los orígenes de los que se ha efectuado una copia de seguridad desde este servidor/cliente.

```powershell
$Source = Get-OBRecoverableSource
$Source
```

```Output
FriendlyName : C:\
RecoverySourceName : C:\
ServerName : myserver.microsoft.com

FriendlyName : D:\
RecoverySourceName : D:\
ServerName : myserver.microsoft.com
```

### <a name="choosing-a-backup-point-from-which-to-restore"></a>Elección de un punto de copia de seguridad desde el que efectuar la restauración

La lista de puntos de copia de seguridad se puede recuperar ejecutando el cmdlet [Get-OBRecoverableItem](/powershell/module/msonlinebackup/get-obrecoverableitem) con los parámetros adecuados. En nuestro ejemplo, elegiremos el punto de copia de seguridad más reciente para el volumen de origen *C:* y lo usaremos para recuperar un archivo específico.

```powershell
$Rps = Get-OBRecoverableItem $Source[0]
$Rps
```

```Output

IsDir                : False
ItemNameFriendly     : C:\
ItemNameGuid         : \\?\Volume{297cbf7a-0000-0000-0000-401f00000000}\
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : C:\
PointInTime          : 10/17/2019 7:52:13 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime :

IsDir                : False
ItemNameFriendly     : C:\
ItemNameGuid         : \\?\Volume{297cbf7a-0000-0000-0000-401f00000000}\
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : C:\
PointInTime          : 10/16/2019 7:00:19 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime :
```

El objeto `$Rps` es una matriz de puntos de copia de seguridad. El primer elemento es el punto más reciente y el enésimo elemento es el punto más antiguo. Para elegir el punto más reciente, usaremos `$Rps[0]`.

### <a name="specifying-an-item-to-restore"></a>Especificar un elemento para restaurar

Para restaurar un archivo concreto, especifique el nombre de archivo relacionado con el volumen raíz. Por ejemplo, para recuperar C:\Test\Cat.job, ejecute el siguiente comando.

```powershell
$Item = New-OBRecoverableItem $Rps[0] "Test\cat.jpg" $FALSE
$Item
```

```Output
IsDir                : False
ItemNameFriendly     : C:\Test\cat.jpg
ItemNameGuid         :
LocalMountPoint      : C:\
MountPointName       : C:\
Name                 : cat.jpg
PointInTime          : 10/17/2019 7:52:13 PM
ServerName           : myserver.microsoft.com
ItemSize             :
ItemLastModifiedTime : 21-Jun-14 6:43:02 AM

```

### <a name="triggering-the-restore-process"></a>Desencadenar el proceso de restauración

Para desencadenar el proceso de restauración, primero es necesario especificar las opciones de recuperación. Esto puede hacerse mediante el uso del cmdlet [New-OBRecoveryOption](/powershell/module/msonlinebackup/new-obrecoveryoption) . En este ejemplo, supongamos que queremos restaurar los archivos en *C:\temp*. Supongamos también que queremos omitir archivos que ya existen en la carpeta de destino *C:\temp*. Para crear dicha opción de recuperación, use el siguiente comando:

```powershell
$RecoveryOption = New-OBRecoveryOption -DestinationPath "C:\temp" -OverwriteType Skip
```

Ahora, desencadene el proceso de restauración con el comando [Start-OBRecovery](/powershell/module/msonlinebackup/start-obrecovery) en el `$Item` seleccionado desde la salida del cmdlet `Get-OBRecoverableItem`:

```powershell
Start-OBRecovery -RecoverableItem $Item -RecoveryOption $RecoveryOption
```

```Output
Estimating size of backup items...
Estimating size of backup items...
Estimating size of backup items...
Estimating size of backup items...
Job completed.
The recovery operation completed successfully.
```

## <a name="uninstalling-the-azure-backup-agent"></a>Desinstalación del agente de Azure Backup

La desinstalación del agente de Azure Backup se puede realizar con el comando siguiente:

```powershell
.\MARSAgentInstaller.exe /d /q
```

Desinstalación de los archivos binarios del agente de la máquina tiene algunas consecuencias a tener en cuenta:

* Elimina el filtro de archivos de la máquina y se detiene el seguimiento de los cambios.
* Se elimina toda la información de directivas de la máquina, pero continúa almacenada en el servicio.
* Se eliminan todas las programaciones de copia de seguridad y no se realizan más copias de seguridad.

Pero los datos almacenados en Azure permanecen y se mantienen de acuerdo con la configuración de la directiva de retención que haya establecido. Los puntos más antiguos vencen automáticamente.

## <a name="remote-management"></a>Administración remota

Toda la administración relacionada con el agente, las políticas y los orígenes de datos de Azure Backup puede realizarse de forma remota mediante PowerShell. La máquina que se administrará de forma remota debe estar preparada correctamente.

De forma predeterminada, el servicio WinRM está configurado para iniciarse manualmente. El tipo de inicio debe establecerse en *Automatic* y se debe iniciar el servicio. Para comprobar que el servicio WinRM se está ejecutando, el valor de la propiedad Status debe ser *Running*.

```powershell
Get-Service -Name WinRM
```

```Output
Status   Name               DisplayName
------   ----               -----------
Running  winrm              Windows Remote Management (WS-Manag...
```

PowerShell debe configurarse para la comunicación remota.

```powershell
Enable-PSRemoting -Force
```

```Output
WinRM is already set up to receive requests on this computer.
WinRM has been updated for remote management.
WinRM firewall exception enabled.
```

```powershell
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Force
```

La máquina puede ahora administrarse de forma remota: empezando por la instalación del agente. Por ejemplo, el script siguiente copia al agente en la máquina remota y lo instala.

```powershell
$DLoc = "\\REMOTESERVER01\c$\Windows\Temp"
$Agent = "\\REMOTESERVER01\c$\Windows\Temp\MARSAgentInstaller.exe"
$Args = "/q"
Copy-Item "C:\Downloads\MARSAgentInstaller.exe" -Destination $DLoc -Force

$Session = New-PSSession -ComputerName REMOTESERVER01
Invoke-Command -Session $Session -Script { param($D, $A) Start-Process -FilePath $D $A -Wait } -ArgumentList $Agent, $Args
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Backup para Windows Server o cliente de Windows:

* [Introducción a Azure Backup](./backup-overview.md)
* [Copia de seguridad de servidores Windows](backup-windows-with-mars-agent.md)
