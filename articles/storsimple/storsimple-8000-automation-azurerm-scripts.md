---
title: Uso de scripts de AzureRM PowerShell para administrar dispositivos de StorSimple
description: Obtenga información sobre cómo usar los scripts de Azure Resource Manager para automatizar trabajos de StorSimple
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 10/03/2017
ms.author: alkohli
ms.openlocfilehash: 3b9a7bf243931443b8273253dee7618c0e594aaf
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87020872"
---
# <a name="use-azure-resource-manager-sdk-based-scripts-to-manage-storsimple-devices"></a>Uso de scripts basados en el SDK de Azure Resource Manager para administrar dispositivos de StorSimple

En este artículo se describe cómo se pueden usar los scripts basados en el SDK de Azure Resource Manager para administrar un dispositivo de la serie StorSimple 8000. También se incluye un script de ejemplo para guiarle por los pasos necesarios para configurar un entorno para ejecutar estos scripts.

Este artículo se aplica solo a los dispositivos de la serie StorSimple 8000 que se ejecutan en Azure Portal.

## <a name="sample-scripts"></a>Muestras de scripts

Los scripts de ejemplo siguientes están disponibles para automatizar diversos trabajos de StorSimple.

#### <a name="table-of-azure-resource-manager-sdk-based-sample-scripts"></a>Tabla de scripts de ejemplo basados en el SDK de Azure Resource Manager

| Script de Azure Resource Manager                    | Descripción                                                                                                                                                                                                       |
|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [Authorize-ServiceEncryptionRollover.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Authorize-ServiceEncryptionRollover.ps1)          | Este script permite autorizar que un dispositivo de StorSimple cambie la clave de cifrado de datos del servicio.                                                                                                           |
| [Create-StorSimpleCloudAppliance.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Create-StorSimpleCloudAppliance.ps1)              | Este script crea una instancia de StorSimple Cloud Appliance 8010 u 8020. El dispositivo en la nube se puede configurar y registrar con el servicio StorSimple Data Manager.                                                       |
| [CreateOrUpdate-Volume.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/CreateOrUpdate-Volume.ps1)                        | Este script crea o modifica los volúmenes de StorSimple.                                                                                                                                                             |
| [Get-DeviceBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceBackup.ps1)                             | Este script muestra todas las copias de seguridad de un dispositivo registrado con el servidor Administrador de dispositivos de StorSimple.                                                                                                          |
| [Get-DeviceBackupPolicy.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceBackupPolicy.ps1)                       | Este script muestra todas las directivas de copia de seguridad de un dispositivo de StorSimple.                                                                                                                                                 |
| [Get-DeviceJobs.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1)                               | Este script obtiene todos los trabajos de StorSimple que se ejecutan en el servicio Administrador de dispositivos de StorSimple.                                                                                                                     |
| [Get-DeviceUpdateAvailability.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceUpdateAvailability.ps1)                 | Este script examina el servidor de actualización y le permite saber si hay actualizaciones disponibles para instalarlas en el dispositivo de StorSimple.                                                                                          |
| [Install-DeviceUpdate.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Install-DeviceUpdate.ps1)                         | Este script instala todas las actualizaciones disponibles en el dispositivo de StorSimple.                                                                                                                                           |
| [Manage-CloudSnapshots.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Manage-CloudSnapshots.ps1)                        | Este script inicia una instantánea de nube manual y elimina las instantáneas de nube anteriores a los días de retención especificados.                                                                                                   |
| [Monitor-Backups.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Monitor-Backups.ps1)                              | Este script de PowerShell de Runbook de Azure Automation informa el estado de todos los trabajos de copia de seguridad.                                                                                                              |
| [Remove-DeviceBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Remove-DeviceBackup.ps1)                          | Este script elimina un objeto de copia de seguridad único.                                                                                                                                                           |
| [Start-DeviceBackupJob.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Start-DeviceBackupJob.ps1)                        | Este script inicia una copia de seguridad manual en el dispositivo de StorSimple.                                                                                                                                       |
| [Update-CloudApplianceServiceEncryptionKey.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Update-CloudApplianceServiceEncryptionKey.ps1)    | Este script actualiza la clave de cifrado de datos del servicio de todas las instancias de StorSimple Cloud Appliance 8010/8020 registradas en el servicio Administrador de dispositivos de StorSimple.                                     |
| [Verify-BackupScheduleAndBackup.ps1](https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Verify-BackupScheduleAndBackup.ps1)               | Este script resalta las copias de seguridad que faltan después de analizar todas las programaciones asociadas con las directivas de copia de seguridad. También comprueba el catálogo de copias de seguridad con la lista de copias de seguridad disponibles.             |




## <a name="configure-and-run-a-sample-script"></a>Configuración y ejecución de un script de ejemplo

En esta sección se usa un script de ejemplo y se detallan los diversos pasos que se requieren para ejecutar el script.

### <a name="prerequisites"></a>Prerequisites

Antes de comenzar, asegúrese de que dispone de:

*   Azure PowerShell instalado. Para instalar módulos de Azure PowerShell:
    * En un entorno Windows, siga los pasos que aparecen en [Instale y configure Azure PowerShell](/powershell/azure/install-az-ps). Puede instalar Azure PowerShell en el host Windows Server para una instancia de StorSimple, si usa.
    * En un entorno Linux o MacOS, siga los pasos que aparecen en [Instalación y configuración de Azure PowerShell en MacOS o Linux](/powershell/azure/install-az-ps).

Para más información sobre cómo usar Azure PowerShell, vaya a [Introducción a Azure PowerShell](https://docs.microsoft.com/powershell/azure/get-started-azureps).

### <a name="run-azure-powershell-script"></a>Ejecución del script de Azure PowerShell

El script que se usa en este ejemplo muestra todos los trabajos de un dispositivo de StorSimple. Esto incluye todos los trabajos que se ejecutaron correctamente, los que no se pudieron ejecutar o los que están en curso. Siga estos pasos para descargar y ejecutar el script.

1. Inicie Azure PowerShell. Cree una carpeta nueva y cambie de directorio a la carpeta nueva.

    ```
        mkdir C:\scripts\StorSimpleSDKTools
        cd C:\scripts\StorSimpleSDKTools
    ```    
2. [Descargue la CLI de NuGet](https://www.nuget.org/downloads) en la carpeta que se creó en el paso anterior. Existen varias versiones de _nuget.exe_. Elija la versión que corresponde a su SDK. Cada vínculo de descarga apunta directamente a un archivo _.exe_. Asegúrese de hacer clic con el botón derecho y de guardar el archivo en el equipo en lugar de ejecutarlo desde el explorador.

    También puede ejecutar el comando siguiente para descargar y almacenar el script en la misma carpeta que creó anteriormente.
    
    ```
        wget https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -Out C:\scripts\StorSimpleSDKTools\nuget.exe
    ```
3. Descargue el SDK dependiente.

    ```
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Azure.Management.Storsimple8000series
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.IdentityModel.Clients.ActiveDirectory -Version 2.28.3
        C:\scripts\StorSimpleSDKTools\nuget.exe install Microsoft.Rest.ClientRuntime.Azure.Authentication -Version 2.2.9-preview
    ```    
4. Descargue el script desde el proyecto GitHub de ejemplo.

    ```
        wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1 -Out Get-DeviceJobs.ps1

    ```

5. Ejecute el script. Cuando se le solicite que se autentique, proporcione sus credenciales de Azure. Este script debería generar una lista filtrada de todos los trabajos en el dispositivo de StorSimple.
           
    ```           
        .\Get-StorSimpleJob.ps1 -SubscriptionId [subid] -TenantId [tenant id] -DeviceName [name of device] -ResourceGroupName [name of resource group] -ManagerName[name of device manager] -FilterByStatus [Filter for job status] -FilterByJobType [Filter for job type] -FilterByStartTime [Filter for start date time] -FilterByEndTime [Filter for end date time]

    ```

### <a name="sample-output"></a>Salida de ejemplo

La salida siguiente aparece cuando se ejecuta el script de ejemplo. La salida contiene todos los trabajos que se ejecutaron en un dispositivo registrado que comenzaron el 25 de septiembre de 2017 y se completaron hasta el 2 de octubre de 2017.

```
-----------------------------------------
Windows PowerShell
Copyright (C) 2016 Microsoft Corporation. All rights reserved.

PS C:\Scripts\StorSimpleSDKTools> wget https://raw.githubusercontent.com/anoobbacker/storsimpledevicemgmttools/master/Get-DeviceJobs.ps1 -Out Get-DeviceJobs.ps1
PS C:\Scripts\StorSimpleSDKTools> .\Get-DeviceJobs.ps1 -SubscriptionId 1234ab5c-678d-910e-9fc4-0accc9c0166e -TenantId 12a345bc-67d8-91ef-01ab-2c7cd123ef45 -DeviceName 8600-ABC1234567D89EF -ResourceGroupName Contoso -ManagerName ContosoDeviceMgr -FilterByStartTime "09/25/2017 08:10:02" -FilterByEndTime "10/02/2017 08:10:02"


Status            : Succeeded
StartTime         : 10/2/2017 10:30:02 PM
EndTime           : 10/2/2017 10:31:36 PM
PercentComplete   : 100
Error             :
JobType           : ScheduledBackup
DataStats         : Microsoft.Azure.Management.StorSimple8000Series.Models.DataStatistics
EntityLabel       : ss-asr-policy1
EntityType        : Microsoft.StorSimple/managers/devices/backupPolicies
JobStages         :
DeviceId          : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-SHG0997877L71FC
IsCancellable     : True
BackupType        : CloudSnapshot
SourceDeviceId    :
BackupPointInTime : 1/1/0001 12:00:00 AM
Id                : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-SHG0997877L71FC/jobs/75905c48-b153-4af1-8b21-4b9a2ff9
                    825b
Name              : 75905c48-b153-4af1-8b21-4b9a2ff9825b
Type              : Microsoft.StorSimple/managers/devices/jobs
Kind              : Series8000
-------------------------------------------
CUT              CUT  
-------------------------------------------
Status            : Succeeded
StartTime         : 9/26/2017 5:00:02 PM
EndTime           : 9/26/2017 5:01:20 PM
PercentComplete   : 100
Error             :
JobType           : ScheduledBackup
DataStats         : Microsoft.Azure.Management.StorSimple8000Series.Models.DataStatistics
EntityLabel       : 8010 policy
EntityType        : Microsoft.StorSimple/managers/devices/backupPolicies
JobStages         :
DeviceId          : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-ABC1234567D89EF
IsCancellable     : True
BackupType        : CloudSnapshot
SourceDeviceId    :
BackupPointInTime : 1/1/0001 12:00:00 AM
Id                : /subscriptions/1234ab5c-678d-910e-9fc4-0accc9c0166e/resourceGroups/Contoso/providers/Microsoft.Stor
                    Simple/managers/ContosoDeviceMgr/devices/8600-ABC1234567D89EF/jobs/3cfd8108-db60-4e9a-a8da-6d8fe457
                    8d2b
Name              : 3cfd8108-db60-4e9a-a8da-6d8fe4578d2b
Type              : Microsoft.StorSimple/managers/devices/jobs
Kind              : Series8000



PS C:\Scripts\StorSimpleSDKTools>
--------------------------------------------

```


## <a name="next-steps"></a>Pasos siguientes

[Uso del servicio Administrador de dispositivos de StorSimple para administrar un dispositivo de StorSimple](storsimple-8000-manager-service-administration.md).
