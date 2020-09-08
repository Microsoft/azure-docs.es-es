---
title: Implementar Azure File Sync | Microsoft Docs
description: Aprenda a implementar Azure File Sync, desde el principio hasta el final, mediante Azure Portal, PowerShell o la CLI de Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 07/19/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: f2c8dbebce685eea67672a2b8c93d51e356ac69c
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88226060"
---
# <a name="deploy-azure-file-sync"></a>Implementación de Azure File Sync
Use Azure File Sync para centralizar los recursos compartidos de archivos de su organización en Azure Files sin renunciar a la flexibilidad, el rendimiento y la compatibilidad de un servidor de archivos local. Azure File Sync transforma Windows Server en una caché rápida de los recursos compartidos de archivos de Azure. Puede usar cualquier protocolo disponible en Windows Server para acceder a sus datos localmente, como SMB, NFS y FTPS. Puede tener todas las cachés que necesite en todo el mundo.

Se recomienda encarecidamente leer [Planeamiento de una implementación de Azure Files](storage-files-planning.md) y [Planeamiento de una implementación de Azure File Sync](storage-sync-files-planning.md) antes de seguir los pasos que se describen en este artículo.

## <a name="prerequisites"></a>Requisitos previos

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Un recurso compartido de archivos de Azure en la misma región en la que quiere implementar Azure File Sync. Para más información, consulte:
    - [Disponibilidad en regiones](storage-sync-files-planning.md#azure-file-sync-region-availability) de Azure File Sync.
    - [Creación de un recurso compartido de archivos](storage-how-to-create-file-share.md) para obtener una descripción paso a paso sobre cómo crear un recurso compartido de archivos.
1. Al menos una instancia de Windows Server o un clúster de Windows Server compatible para la sincronización con Azure File Sync. Para más información sobre las versiones compatibles de Windows Server y los recursos del sistema recomendados, consulte [Consideraciones sobre el servidor de archivos de Windows](storage-sync-files-planning.md#windows-file-server-considerations).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Un recurso compartido de archivos de Azure en la misma región en la que quiere implementar Azure File Sync. Para más información, consulte:
    - [Disponibilidad en regiones](storage-sync-files-planning.md#azure-file-sync-region-availability) de Azure File Sync.
    - [Creación de un recurso compartido de archivos](storage-how-to-create-file-share.md) para obtener una descripción paso a paso sobre cómo crear un recurso compartido de archivos.
1. Al menos una instancia de Windows Server o un clúster de Windows Server compatible para la sincronización con Azure File Sync. Para más información sobre las versiones compatibles de Windows Server y los recursos del sistema recomendados, consulte [Consideraciones sobre el servidor de archivos de Windows](storage-sync-files-planning.md#windows-file-server-considerations).

1. El módulo Az PowerShell puede usarse con PowerShell 5.1 o PowerShell 6 +. Aunque puede usar el módulo Az PowerShell para Azure File Sync en cualquier sistema compatible, incluidos los sistemas que no son Windows, el cmdlet de registro de servidor siempre se debe ejecutar en la instancia de Windows Server que se va a registrar (esto se puede hacer directamente o a través de la comunicación remota de PowerShell). En Windows Server 2012 R2, para comprobar que se ejecuta al menos PowerShell 5.1.\* es preciso examinar el valor de la propiedad **PSVersion** del objeto **$PSVersionTable**:

    ```powershell
    $PSVersionTable.PSVersion
    ```

    Si el valor de **PSVersion** es menor que 5.1.\*, como sucede con la mayoría de las instalaciones nuevas de Windows Server 2012 R2, puede realizar la actualización fácilmente si descarga e instala [Windows Management Framework (WMF) 5.1](https://www.microsoft.com/download/details.aspx?id=54616). El paquete adecuado que hay que descargar e instalar para Windows Server 2012 R2 es **Win8.1AndW2K12R2-KB\*\*\*\*\*\*\*-x64.msu**. 

    PowerShell 6 + se puede usar con cualquier sistema compatible y puede descargarse mediante su [página de GitHub](https://github.com/PowerShell/PowerShell#get-powershell). 

    > [!Important]  
    > Si tiene previsto usar la interfaz de usuario de registro del servidor, en lugar de registrar directamente desde PowerShell, debe usar PowerShell 5.1.

1. Si ha optado por usar PowerShell 5.1, asegúrese de que al menos .NET 4.7.2 esté instalado. Más información sobre las [versiones y dependencias de .NET Framework](https://docs.microsoft.com/dotnet/framework/migration-guide/versions-and-dependencies) en el sistema.

    > [!Important]  
    > Si va a instalar .NET 4.7.2+ en Windows Server Core, debe instalarlo con las marcas `quiet` y `norestart` o se producirá un error en la instalación. Por ejemplo, si instala .NET 4,8, el comando tendría un aspecto similar al siguiente:
    > ```PowerShell
    > Start-Process -FilePath "ndp48-x86-x64-allos-enu.exe" -ArgumentList "/q /norestart" -Wait
    > ```

1. El módulo Az PowerShell, que se puede instalar siguiendo estas instrucciones: [Instale y configure Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps).
     
    > [!Note]  
    > El módulo Az.StorageSync ahora se instala automáticamente al instalar el módulo Az PowerShell.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

1. Un recurso compartido de archivos de Azure en la misma región en la que quiere implementar Azure File Sync. Para más información, consulte:
    - [Disponibilidad en regiones](storage-sync-files-planning.md#azure-file-sync-region-availability) de Azure File Sync.
    - [Creación de un recurso compartido de archivos](storage-how-to-create-file-share.md) para obtener una descripción paso a paso sobre cómo crear un recurso compartido de archivos.
1. Al menos una instancia de Windows Server o un clúster de Windows Server compatible para la sincronización con Azure File Sync. Para más información sobre las versiones compatibles de Windows Server y los recursos del sistema recomendados, consulte [Consideraciones sobre el servidor de archivos de Windows](storage-sync-files-planning.md#windows-file-server-considerations).

1. [Instalación de la CLI de Azure](/cli/azure/install-azure-cli)

   Si lo prefiere, también puede usar Azure Cloud Shell para completar los pasos de este tutorial.  Azure Cloud Shell es un entorno de shell interactivo que se utiliza mediante el explorador.  Inicie Cloud Shell con uno de estos métodos:

   - Seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. Al seleccionar **Pruébelo**, se abrirá Azure Cloud Shell, pero no se copiará automáticamente el código en Cloud Shell.

   - Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Cloud Shell.

   - Seleccione el botón **Cloud Shell** en la barra de menús de la esquina superior derecha de [Azure Portal](https://portal.azure.com).

1. Inicie sesión.

   Si está usando una instalación local de la CLI, inicie sesión con el comando [az login](/cli/azure/reference-index#az-login).

   ```azurecli
   az login
   ```

    Siga los pasos que se muestran en el terminal para completar el proceso de autenticación.

1. Instale la extensión de la CLI de Azure [az filesync](/cli/azure/ext/storagesync/storagesync).

   ```azurecli
   az extension add --name storagesync
   ```

   Después de instalar la referencia de la extensión **storagesync**, recibirá la siguiente advertencia.

   ```output
   The installed extension 'storagesync' is experimental and not covered by customer support. Please use with discretion.
   ```

---

## <a name="prepare-windows-server-to-use-with-azure-file-sync"></a>Preparación de Windows Server para su uso con Azure File Sync
Para cada servidor que se va a usar con Azure File Sync, incluyendo cada nodo de servidor en un clúster de conmutación por error, deshabilite la **Configuración de seguridad mejorada de Internet Explorer**. Esto solo es necesario para el registro inicial del servidor. Se puede volver a habilitar una vez registrado el servidor.

# <a name="portal"></a>[Portal](#tab/azure-portal)
> [!Note]  
> Puede omitir este paso si va a implementar Azure File Sync en Windows Server Core.

1. Abra el Administrador del servidor.
2. Haga clic en **Servidor Local**:  
    !["Servidor local" en el lado izquierdo de la interfaz de usuario Administrador del servidor](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-1.PNG)
3. En el panel secundario **Propiedades**, seleccione el vínculo de **Configuración de seguridad mejorada de IE**.  
    ![Panel "Configuración de seguridad mejorada de IE" en la interfaz de usuario Administrador del servidor](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-2.PNG)
4. En el cuadro de diálogo **Configuración de seguridad mejorada de Internet Explorer**, seleccione **Desactivado** para **Administradores** y **Usuarios**:  
    ![Ventana emergente de la configuración de seguridad mejorada de Internet Explorer con "Desactivado" seleccionado](media/storage-sync-files-deployment-guide/prepare-server-disable-IEESC-3.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Para deshabilitar la configuración de seguridad mejorada de Internet Explorer, ejecute lo siguiente desde una sesión de PowerShell con privilegios elevados:

```powershell
$installType = (Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\").InstallationType

# This step is not required for Server Core
if ($installType -ne "Server Core") {
    # Disable Internet Explorer Enhanced Security Configuration 
    # for Administrators
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A7-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force
    
    # Disable Internet Explorer Enhanced Security Configuration 
    # for Users
    Set-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Active Setup\Installed Components\{A509B1A8-37EF-4b3f-8CFC-4F3A74704073}" -Name "IsInstalled" -Value 0 -Force
    
    # Force Internet Explorer closed, if open. This is required to fully apply the setting.
    # Save any work you have open in the IE browser. This will not affect other browsers,
    # including Microsoft Edge.
    Stop-Process -Name iexplore -ErrorAction SilentlyContinue
}
``` 

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Siga las instrucciones para Azure Portal o PowerShell.

---

## <a name="deploy-the-storage-sync-service"></a>Implementación del servicio de sincronización de almacenamiento 
La implementación de Azure File Sync comienza por situar un recurso del **servicio de sincronización de almacenamiento** en un grupo de recursos de la suscripción seleccionada. Se recomienda aprovisionar únicamente los necesarios. Se creará una relación de confianza entre los servidores y este recurso y un servidor solo se puede registrar en un servicio de sincronización de almacenamiento. Como resultado, se recomienda implementar tantos servicios de sincronización de almacenamiento como necesite para separar los grupos de servidores. Tenga en cuenta que los servidores de servicios de sincronización de almacenamiento diferentes no se pueden sincronizar entre sí.

> [!Note]
> El servicio de sincronización de almacenamiento hereda los permisos de acceso de la suscripción y el grupo de recursos en los que se ha implementado. Se recomienda que compruebe cuidadosamente quién tiene acceso al mismo. Las entidades con acceso de escritura pueden empezar a sincronizar nuevos conjuntos de archivos de servidores registrados en este servicio de sincronización de almacenamiento y hacer que los datos fluyan al almacenamiento de Azure al que tengan acceso.

# <a name="portal"></a>[Portal](#tab/azure-portal)
Para implementar un servicio de sincronización de almacenamiento, vaya a [Azure Portal](https://portal.azure.com/), haga clic en *Crear un recurso* y busque Azure File Sync. En los resultados de la búsqueda, seleccione **Azure File Sync** y, a continuación, seleccione **Crear** para abrir la pestaña **Implementar la sincronización del almacenamiento**.

En el panel que se abre, escriba la siguiente información:

- **Name**: Un nombre único (por suscripción) para el servicio de sincronización de almacenamiento.
- **Suscripción**: la suscripción en la que quiere crear el servicio de sincronización de almacenamiento. Según la estrategia de configuración de la organización, puede tener acceso a una o más suscripciones. Una suscripción de Azure es el contenedor más básico para la facturación de cada servicio en la nube (como Azure Files).
- **Grupo de recursos**: un grupo de recursos es un grupo lógico de recursos de Azure, como una cuenta de almacenamiento o un servicio de sincronización de almacenamiento. Puede crear un nuevo grupo de recursos o seleccionar uno existente para Azure File Sync. (Se recomienda usar los grupos de recursos como contenedores para aislar los recursos de manera lógica para su organización, como la agrupación de recursos de RR. HH. o recursos de un proyecto específico).
- **Ubicación**: la región en la que quiere implementar Azure File Sync. Solo las regiones admitidas están disponibles en esta lista.

Cuando haya terminado, seleccione **Crear** para implementar el servicio de sincronización de almacenamiento.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Reemplace `<Az_Region>`, `<RG_Name>` y `<my_storage_sync_service>` por sus propios valores y use los siguientes comandos para crear e implementar un servicio de sincronización de almacenamiento:

```powershell
$hostType = (Get-Host).Name

if ($installType -eq "Server Core" -or $hostType -eq "ServerRemoteHost") {
    Connect-AzAccount -UseDeviceAuthentication
}
else {
    Connect-AzAccount
}

# this variable holds the Azure region you want to deploy 
# Azure File Sync into
$region = '<Az_Region>'

# Check to ensure Azure File Sync is available in the selected Azure
# region.
$regions = @()
Get-AzLocation | ForEach-Object { 
    if ($_.Providers -contains "Microsoft.StorageSync") { 
        $regions += $_.Location 
    } 
}

if ($regions -notcontains $region) {
    throw [System.Exception]::new("Azure File Sync is either not available in the selected Azure Region or the region is mistyped.")
}

# the resource group to deploy the Storage Sync Service into
$resourceGroup = '<RG_Name>'

# Check to ensure resource group exists and create it if doesn't
$resourceGroups = @()
Get-AzResourceGroup | ForEach-Object { 
    $resourceGroups += $_.ResourceGroupName 
}

if ($resourceGroups -notcontains $resourceGroup) {
    New-AzResourceGroup -Name $resourceGroup -Location $region
}

$storageSyncName = "<my_storage_sync_service>"
$storageSync = New-AzStorageSyncService -ResourceGroupName $resourceGroup -Name $storageSyncName -Location $region
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Siga las instrucciones para Azure Portal o PowerShell.

---

## <a name="install-the-azure-file-sync-agent"></a>Instalación del agente de Azure File Sync
El agente de Azure File Sync es un paquete descargable que permite la sincronización de Windows Server con un recurso compartido de archivos de Azure. 

# <a name="portal"></a>[Portal](#tab/azure-portal)
Puede descargar el agente desde el [Centro de descarga de Microsoft](https://go.microsoft.com/fwlink/?linkid=858257). Una finalice la descarga, haga doble clic en el paquete MSI para iniciar la instalación del agente de Azure File Sync.

> [!Important]  
> Si piensa usar Azure File Sync con un clúster de conmutación por error, el agente de Azure File Sync se debe instalar en todos los nodos del clúster. Cada nodo del clúster debe estar registrado para funcionar con Azure File Sync.

Se recomienda hacer lo siguiente:
- Dejar la ruta de instalación predeterminada (C:\Archivos de programa\Azure\StorageSyncAgent), para simplificar las operaciones de solución de problemas y mantenimiento del servidor.
- Habilitar Microsoft Update para mantener actualizado Azure File Sync. Todas las actualizaciones del agente de Azure File Sync, incluidas las actualizaciones y revisiones de características, se realizan desde Microsoft Update. Se recomienda instalar la actualización más reciente de Azure File Sync. Para obtener más información, consulte [Directiva de actualización del agente de Azure File Sync](storage-sync-files-planning.md#azure-file-sync-agent-update-policy).

Al término de la instalación del agente de Azure File Sync, la interfaz de usuario Registro del servidor se abre automáticamente. Debe tener un servicio de sincronización de almacenamiento antes del registro; vea la sección siguiente sobre cómo crear un servicio de sincronización de almacenamiento.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Ejecute el siguiente código de PowerShell para descargar la versión adecuada del agente de Azure File Sync para su sistema operativo e instálelo en el sistema.

> [!Important]  
> Si piensa usar Azure File Sync con un clúster de conmutación por error, el agente de Azure File Sync se debe instalar en todos los nodos del clúster. Cada nodo del clúster debe estar registrado para funcionar con Azure File Sync.

```powershell
# Gather the OS version
$osver = [System.Environment]::OSVersion.Version

# Download the appropriate version of the Azure File Sync agent for your OS.
if ($osver.Equals([System.Version]::new(10, 0, 17763, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2019 `
        -OutFile "StorageSyncAgent.msi" 
} elseif ($osver.Equals([System.Version]::new(10, 0, 14393, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2016 `
        -OutFile "StorageSyncAgent.msi" 
} elseif ($osver.Equals([System.Version]::new(6, 3, 9600, 0))) {
    Invoke-WebRequest `
        -Uri https://aka.ms/afs/agent/Server2012R2 `
        -OutFile "StorageSyncAgent.msi" 
} else {
    throw [System.PlatformNotSupportedException]::new("Azure File Sync is only supported on Windows Server 2012 R2, Windows Server 2016, and Windows Server 2019")
}

# Install the MSI. Start-Process is used to PowerShell blocks until the operation is complete.
# Note that the installer currently forces all PowerShell sessions closed - this is a known issue.
Start-Process -FilePath "StorageSyncAgent.msi" -ArgumentList "/quiet" -Wait

# Note that this cmdlet will need to be run in a new session based on the above comment.
# You may remove the temp folder containing the MSI and the EXE installer
Remove-Item -Path ".\StorageSyncAgent.msi" -Recurse -Force
```
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Siga las instrucciones para Azure Portal o PowerShell.

---

## <a name="register-windows-server-with-storage-sync-service"></a>Registro de un servidor de Windows Server con el servicio de sincronización de almacenamiento
El registro del servidor Windows Server con un servicio de sincronización de almacenamiento establece una relación de confianza entre el servidor (o clúster) y el servicio de sincronización de almacenamiento. Un servidor solo se puede registrar en un servicio de sincronización de almacenamiento y puede sincronizarse con otros servidores y recursos compartidos de archivos de Azure asociados con el mismo servicio de sincronización de almacenamiento.

> [!Note]
> El registro del servidor usa sus credenciales de Azure para crear una relación de confianza entre el servicio de sincronización de almacenamiento y su instancia de Windows Server; sin embargo, posteriormente, el servidor crea y usa su propia identidad, que es válida mientras el servidor permanezca registrado y el token de la firma de acceso compartido (SAS de almacenamiento) actual sea válido. No se puede emitir un nuevo token de SAS para el servidor una vez que se anula el registro del servidor, lo que elimina la capacidad del servidor de acceder a los recursos compartidos de archivos de Azure, deteniendo cualquier sincronización.

El administrador que registra el servidor debe ser miembro de los roles de administración **Propietario** o **Colaborador** para el servicio de sincronización de almacenamiento determinado. Esto se puede configurar en **Access Control (IAM)** en Azure Portal para el servicio de sincronización de almacenamiento.

También es posible diferenciar a los administradores que pueden registrar servidores de los que pueden configurar también la sincronización en un servicio de sincronización de almacenamiento. Para ello, debe crear un rol personalizado en el que se muestren los administradores que solo tienen permiso para registrar servidores y conceder a su rol personalizado los permisos siguientes:

* "Microsoft.StorageSync/storageSyncServices/registeredServers/write"
* "Microsoft.StorageSync/storageSyncServices/read"
* "Microsoft.StorageSync/storageSyncServices/workflows/read"
* "Microsoft.StorageSync/storageSyncServices/workflows/operations/read"

# <a name="portal"></a>[Portal](#tab/azure-portal)
Al término de la instalación del agente de Azure File Sync, la interfaz de usuario Registro del servidor debería abrirse automáticamente. Si no es así, puede abrirla manualmente desde su ubicación de archivo: C:\Program Files\Azure\StorageSyncAgent\ServerRegistration.exe. Cuando la interfaz de usuario Registro del servidor se abra, seleccione **Iniciar sesión** para comenzar.

Después de iniciar sesión se le pedirá la información siguiente:

![Captura de pantalla de la interfaz de usuario Registro del servidor](media/storage-sync-files-deployment-guide/register-server-scubed-1.png)

- **Suscripción de Azure**: la suscripción que contiene el servicio de sincronización de almacenamiento (consulte [Implementación del servicio de sincronización de almacenamiento](#deploy-the-storage-sync-service)). 
- **Grupo de recursos**: el grupo de recursos que contiene el servicio de sincronización de almacenamiento.
- **Servicio de sincronización de almacenamiento**: el nombre del servicio de sincronización de almacenamiento con el que quiere registrar.

Una vez que haya seleccionado la información adecuada, haga clic en **Registrar** para completar el registro del servidor. Como parte del proceso de registro, se le solicita un inicio de sesión adicional.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
```powershell
$registeredServer = Register-AzStorageSyncServer -ParentObject $storageSync
```
# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Siga las instrucciones para Azure Portal o PowerShell.

---

## <a name="create-a-sync-group-and-a-cloud-endpoint"></a>Creación de un grupo de sincronización y un punto de conexión en la nube
Un grupo de sincronización define la topología de sincronización de un conjunto de archivos. Los puntos de conexión dentro de un grupo de sincronización se mantienen sincronizados entre sí. Un grupo de sincronización debe contener al menos un punto de conexión en la nube, que representa un recurso compartido de archivos de Azure y uno o varios puntos de conexión de servidor. Un punto de conexión de servidor representa una ruta de acceso en un servidor registrado. Un servidor puede tener puntos de conexión de servidor en varios grupos de sincronización. Puede crear tantos grupos de sincronización como sea necesario para describir correctamente la topología de sincronización deseada.

Un punto de conexión en la nube es un puntero a un recurso compartido de archivos de Azure. Todos los puntos de conexión de servidor se sincronizarán con un punto de conexión en la nube, lo que hace que el punto de conexión en la nube actúe como concentrador. La cuenta de almacenamiento para el recurso compartido de archivos de Azure debe encontrarse en la misma región que el servicio de sincronización de almacenamiento. Se sincronizará la totalidad del recurso compartido de archivos de Azure, con una excepción: se aprovisionará una carpeta especial, comparable a la carpeta oculta "System Volume Information" de un volumen NTFS. Este directorio se llama ".SystemShareInformation". Contiene metadatos de sincronización importantes que no se sincronizarán con otros puntos de conexión. No la utilice ni elimine.

> [!Important]  
> Puede realizar cambios en cualquier punto de conexión en la nube o punto de conexión de servidor en el grupo de sincronización y sincronizar los archivos con los demás puntos de conexión del grupo de sincronización. Si realiza algún cambio directamente en el punto de conexión en la nube (recurso compartido de archivos de Azure), tenga en cuenta que un trabajo de detección de cambios de Azure File Sync deberá detectar primero esos cambios. Se inicia un trabajo de detección de cambios para un punto de conexión en la nube solo una vez cada 24 horas. Para obtener más información, consulte [Preguntas más frecuentes de Azure Files](storage-files-faq.md#afs-change-detection).

El administrador que crea el punto de conexión de nube debe ser miembro del rol de administración **Propietario** para la cuenta de almacenamiento que contiene el recurso compartido de archivos de Azure al que apunta el punto de conexión de nube. Esto se puede configurar en **Access Control (IAM)** en Azure Portal para la cuenta de almacenamiento.

# <a name="portal"></a>[Portal](#tab/azure-portal)
Para crear un grupo de sincronización, en [Azure Portal](https://portal.azure.com/), vaya al servicio de sincronización de almacenamiento y haga clic en **+ Grupo de sincronización**:

![Creación de un grupo de sincronización en Azure Portal](media/storage-sync-files-deployment-guide/create-sync-group-1.png)

En el panel que se abre, escriba la información siguiente para crear un grupo de sincronización con un punto de conexión en la nube:

- **Nombre del grupo de sincronización**: el nombre del grupo de sincronización que se va a crear. Este nombre debe ser único dentro del servicio de sincronización de almacenamiento, pero puede ser cualquier nombre que considere lógico.
- **Suscripción**: la suscripción en la que ha implementado el servicio de sincronización de almacenamiento en [Implementación del servicio de sincronización de almacenamiento](#deploy-the-storage-sync-service).
- **Cuenta de almacenamiento**: si elige **Seleccionar cuenta de almacenamiento**, aparecerá otro panel donde puede seleccionar la cuenta de almacenamiento que tiene el recurso compartido de archivos de Azure con el que quiere realizar la sincronización.
- **Recurso compartido de archivos de Azure**: el nombre del recurso compartido de archivos de Azure con el que desea realizar la sincronización.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Para crear el grupo de sincronización, ejecute el comando de PowerShell siguiente. No olvide reemplazar `<my-sync-group>` por el nombre deseado del grupo de sincronización.

```powershell
$syncGroupName = "<my-sync-group>"
$syncGroup = New-AzStorageSyncGroup -ParentObject $storageSync -Name $syncGroupName
```

Una vez que se ha creado correctamente el grupo de sincronización, puede crear el punto de conexión en la nube. No olvide reemplazar `<my-storage-account>` y `<my-file-share>` por los valores esperados.

```powershell
# Get or create a storage account with desired name
$storageAccountName = "<my-storage-account>"
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroup | Where-Object {
    $_.StorageAccountName -eq $storageAccountName
}

if ($storageAccount -eq $null) {
    $storageAccount = New-AzStorageAccount `
        -Name $storageAccountName `
        -ResourceGroupName $resourceGroup `
        -Location $region `
        -SkuName Standard_LRS `
        -Kind StorageV2 `
        -EnableHttpsTrafficOnly:$true
}

# Get or create an Azure file share within the desired storage account
$fileShareName = "<my-file-share>"
$fileShare = Get-AzStorageShare -Context $storageAccount.Context | Where-Object {
    $_.Name -eq $fileShareName -and $_.IsSnapshot -eq $false
}

if ($fileShare -eq $null) {
    $fileShare = New-AzStorageShare -Context $storageAccount.Context -Name $fileShareName
}

# Create the cloud endpoint
New-AzStorageSyncCloudEndpoint `
    -Name $fileShare.Name `
    -ParentObject $syncGroup `
    -StorageAccountResourceId $storageAccount.Id `
    -AzureFileShareName $fileShare.Name
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az storagesync sync-group](/cli/azure/ext/storagesync/storagesync/sync-group#ext-storagesync-az-storagesync-sync-group-create) para crear un grupo de sincronización.  Para establecer como valor predeterminado un grupo de recursos para todos los comandos de la CLI, use [az configure](/cli/azure/reference-index#az-configure).

```azurecli
az storagesync sync-group create --resource-group myResourceGroupName \
                                 --name myNewSyncGroupName \
                                 --storage-sync-service myStorageSyncServiceName \
```

Use el comando [az storagesync sync-group cloud-endpoint](/cli/azure/ext/storagesync/storagesync/sync-group/cloud-endpoint#ext-storagesync-az-storagesync-sync-group-cloud-endpoint-create) para crear un punto de conexión en la nube.

```azurecli
az storagesync sync-group cloud-endpoint create --resource-group myResourceGroup \
                                                --storage-sync-service myStorageSyncServiceName \
                                                --sync-group-name mySyncGroupName \
                                                --name myNewCloudEndpointName \
                                                --storage-account mystorageaccountname \
                                                --azure-file-share-name azure-file-share-name
```

---

## <a name="create-a-server-endpoint"></a>Creación de un punto de conexión de servidor
Un punto de conexión de servidor representa una ubicación específica en un servidor registrado, como una carpeta en un volumen de servidor. Un punto de conexión de servidor debe ser una ruta de acceso en un servidor registrado (en lugar de un recurso compartido montado) y para usar niveles de nube, la ruta de acceso debe estar en un volumen que no sea del sistema. No se admite el almacenamiento conectado a la red (NAS).

# <a name="portal"></a>[Portal](#tab/azure-portal)
Para agregar un punto de conexión de servidor, vaya al grupo de sincronización recién creado y seleccione **Agregar punto de conexión del servidor**.

![Adición de un nuevo punto de conexión de servidor al panel Grupo de sincronización](media/storage-sync-files-deployment-guide/create-sync-group-2.png)

En el panel **Agregar punto de conexión de servidor**, escriba la siguiente información para crear un punto de conexión de servidor:

- **Servidor registrado**: el nombre del servidor o el clúster en el que se va a crear el punto de conexión de servidor.
- **Ruta de acceso**: la ruta de acceso de Windows Server que se va a sincronizar como parte del grupo de sincronización.
- **Nube por niveles**: un conmutador para habilitar o deshabilitar la nube por niveles. Con los niveles de la nube, los archivos que se usen o a los que se acceda con poca frecuencia se pueden colocar en capas en Azure Files.
- **Espacio disponible del volumen**: la cantidad de espacio libre que se reserva en el volumen en el que reside el punto de conexión de servidor. Por ejemplo, si el espacio disponible del volumen se establece en el 50 % en un volumen con un único punto de conexión de servidor, casi la mitad de la cantidad de datos se coloca en capas en Azure Files. Con independencia de si la característica de niveles en la nube está habilitada, el recurso compartido de archivos de Azure siempre tiene una copia completa de los datos en el grupo de sincronización.
- **Initial download mode** (Modo de descarga inicial): esta es una selección opcional, a partir de la versión 11 del agente, que puede ser útil cuando hay archivos en el recurso compartido de archivos de Azure, pero no en el servidor. Tal situación puede existir, por ejemplo, cuando se crea un punto de conexión de servidor para agregar otro servidor de sucursal a un grupo de sincronización o cuando se recupera de un desastre un servidor que no funciona. Si la nube por niveles está habilitada, el valor predeterminado es recuperar inicialmente solo el espacio de nombres, no el contenido del archivo. Esta opción es útil si cree que es preferible que las solicitudes de acceso de los usuarios decidan qué contenido de archivo se recupera en el servidor. Si la nube por niveles está deshabilitada, el valor predeterminado es que el espacio de nombres se descargue primero y, luego, se recuperen los archivos en función de la marca de tiempo de última modificación hasta que se alcance la capacidad local. Sin embargo, se puede cambiar el modo de descarga inicial para que se descargue solo el espacio de nombres. Existe un tercer modo que solo se puede usar si la nube por niveles está deshabilitada para este punto de conexión de servidor. Este modo evita que se recupere primero el espacio de nombres. Los archivos solo aparecerán en el servidor local si tienen la oportunidad de descargarse por completo. Este modo es útil si, por ejemplo, una aplicación necesita que haya archivos completos y no puede tolerar archivos en capas en su espacio de nombres.

Para agregar el punto de conexión de servidor, seleccione **Crear**. Los archivos se mantienen ahora sincronizados entre el recurso compartido de archivos de Azure y Windows Server. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Ejecute los siguientes comandos de PowerShell para crear el punto de conexión de servidor y asegúrese de reemplazar `<your-server-endpoint-path>` y `<your-volume-free-space>` por los valores deseados; compruebe también el valor opcional de la directiva de descarga inicial opcional.

```powershell
$serverEndpointPath = "<your-server-endpoint-path>"
$cloudTieringDesired = $true
$volumeFreeSpacePercentage = <your-volume-free-space>
# Optional property. Choose from: [NamespaceOnly] default when cloud tiering is enabled. [NamespaceThenModifiedFiles] default when cloud tiering is disabled. [AvoidTieredFiles] only available when cloud tiering is disabled.
$initialDownloadPolicy = NamespaceOnly

if ($cloudTieringDesired) {
    # Ensure endpoint path is not the system volume
    $directoryRoot = [System.IO.Directory]::GetDirectoryRoot($serverEndpointPath)
    $osVolume = "$($env:SystemDrive)\"
    if ($directoryRoot -eq $osVolume) {
        throw [System.Exception]::new("Cloud tiering cannot be enabled on the system volume")
    }

    # Create server endpoint
    New-AzStorageSyncServerEndpoint `
        -Name $registeredServer.FriendlyName `
        -SyncGroup $syncGroup `
        -ServerResourceId $registeredServer.ResourceId `
        -ServerLocalPath $serverEndpointPath `
        -CloudTiering `
        -VolumeFreeSpacePercent $volumeFreeSpacePercentage `
        -InitialDownloadPolicy $initialDownloadPolicy
} else {
    # Create server endpoint
    New-AzStorageSyncServerEndpoint `
        -Name $registeredServer.FriendlyName `
        -SyncGroup $syncGroup `
        -ServerResourceId $registeredServer.ResourceId `
        -ServerLocalPath $serverEndpointPath `
        -InitialDownloadPolicy $initialDownloadPolicy
}
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Use el comando [az storagesync sync-group server-endpoint](/cli/azure/ext/storagesync/storagesync/sync-group/server-endpoint#ext-storagesync-az-storagesync-sync-group-server-endpoint-create) para crear un punto de conexión de servidor.

```azurecli
# Create a new sync group server endpoint 
az storagesync sync-group server-endpoint create --resource-group myResourceGroupName \
                                                 --name myNewServerEndpointName
                                                 --registered-server-id 91beed22-7e9e-4bda-9313-fec96c286e0
                                                 --server-local-path d:\myPath
                                                 --storage-sync-service myStorageSyncServiceNAme
                                                 --sync-group-name mySyncGroupName

# Create a new sync group server endpoint with additional optional parameters
az storagesync sync-group server-endpoint create --resource-group myResourceGroupName \
                                                 --storage-sync-service myStorageSyncServiceName \
                                                 --sync-group-name mySyncGroupName \
                                                 --name myNewServerEndpointName \
                                                 --registered-server-id 91beed22-7e9e-4bda-9313-fec96c286e0 \
                                                 --server-local-path d:\myPath \
                                                 --cloud-tiering on \
                                                 --volume-free-space-percent 85 \
                                                 --tier-files-older-than-days 15 \
                                                 --initial-download-policy NamespaceOnly [OR] NamespaceThenModifiedFiles [OR] AvoidTieredFiles
                                                 --offline-data-transfer on \
                                                 --offline-data-transfer-share-name myfilesharename \

```

---

## <a name="configure-firewall-and-virtual-network-settings"></a>Configuración de los ajustes de red virtual y del firewall

### <a name="portal"></a>Portal
Si quiere configurar Azure File Sync para que funcione con la configuración de red virtual y del firewall, haga lo siguiente:

1. En Azure Portal, vaya a la cuenta de almacenamiento que quiere proteger.
1. Haga clic en el botón **Firewalls y redes virtuales** del menú de la izquierda.
1. Haga clic en **Redes seleccionadas** en **Permitir el acceso desde**.
1. Asegúrese de que la dirección IP o la red virtual de los servidores aparece en la sección correspondiente.
1. Asegúrese de que la opción **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento** está habilitada.
1. Haga clic en **Guardar** para guardar la configuración.

![Configuración de los ajustes del firewall y de la red virtual para trabajar con Azure File Sync](media/storage-sync-files-deployment-guide/firewall-and-vnet.png)

## <a name="onboarding-with-azure-file-sync"></a>Incorporación con Azure File Sync
Los pasos recomendados para la incorporación en Azure File Sync por primera vez con un tiempo de inactividad nulo conservando al mismo tiempo la fidelidad total en los archivos y la lista de control de acceso (ACL) son los siguientes:
 
1. Implementar un servicio de sincronización de almacenamiento.
2. Crear un grupo de sincronización.
3. Instalar el agente de Azure File Sync en el servidor con el conjunto de datos completo.
4. Registrar ese servidor y crear un punto de conexión de servidor en el recurso compartido. 
5. Permitir que la sincronización realice la carga completa al recurso compartido de archivos de Azure (punto de conexión en la nube).  
6. Una vez finalizada la carga inicial, instalar el agente de Azure File Sync en cada uno de los demás servidores.
7. Crear nuevos recursos compartidos de archivos en cada uno de los demás servidores.
8. Crear puntos de conexión de servidor en nuevos recursos compartidos de archivos con la directiva de niveles de nube, si lo desea. (Este paso requiere que haya almacenamiento adicional disponible para la instalación inicial).
9. Deje que el agente de Azure File Sync realice una restauración rápida del espacio de nombres completo sin la transferencia de datos real. Después de la sincronización del espacio de nombres completo, el motor de sincronización rellenará el espacio en disco local en función de la directiva de niveles de nube para el punto de conexión del servidor. 
10. Asegúrese de que la sincronización finaliza y pruebe la topología según sea necesario. 
11. Redirigir a los usuarios y las aplicaciones a este recurso compartido nuevo.
12. Si lo desea, se pueden eliminar todos los recursos compartidos duplicados en los servidores.
 
Si no tiene almacenamiento adicional para la incorporación inicial y le gustaría conectarse a los recursos compartidos existentes, puede inicializar previamente los datos de los recursos compartidos de archivos de Azure. Se recomienda este enfoque si y solo si puede aceptar un tiempo de inactividad y garantiza de forma absoluta que no cambia ningún dato en los recursos compartidos del servidor durante el proceso de incorporación inicial. 
 
1. Asegúrese que los datos no pueden cambiar en ninguno de los servidores durante el proceso de incorporación.
2. Inicialice previamente los recursos compartidos de archivos de Azure con los datos del servidor mediante cualquier herramienta de transferencia de datos a través de SMB, por ejemplo Robocopy, copia directa de SMB. Puesto que AzCopy no carga datos a través de SMB, no se puede usar para la inicialización previa.
3. Cree la topología de Azure File Sync con los puntos de conexión de servidor que desee que apunten a los recursos compartidos existentes.
4. Permita que la sincronización finalice el proceso de conciliación en todos los puntos de conexión. 
5. Una vez completada la conciliación, puede abrir recursos compartidos para los cambios.
 
Actualmente, el enfoque de inicialización previa tiene algunas limitaciones: 
- No se conserva la fidelidad total en los archivos. Por ejemplo, los archivos pierden las ACL y las marcas de tiempo.
- Los datos cambian en el servidor antes de que la topología de sincronización esté totalmente activa y la ejecución puede producir conflictos en los puntos de conexión del servidor.  
- Después de crearse el punto de conexión en la nube, Azure File Sync ejecuta un proceso para detectar los archivos en la nube antes de llevar a cabo la sincronización inicial. El tiempo necesario para completar este proceso depende de distintos factores, como la velocidad de la red, el ancho de banda disponible y el número de archivos y carpetas. Para una estimación aproximada del lanzamiento de la versión preliminar, el proceso de detección se ejecuta a una velocidad aproximada de diez archivos/s. Por lo tanto, incluso si la inicialización previa se ejecuta rápido, el tiempo total para obtener un sistema totalmente operativo puede ser considerablemente mayor cuando los datos se inicializan previamente en la nube.

## <a name="self-service-restore-through-previous-versions-and-vss-volume-shadow-copy-service"></a>Autoservicio de restauración a través de versiones anteriores y VSS (Servicio de instantáneas de volumen)

> [!IMPORTANT]
> La siguiente información solo se puede usar con la versión 9 (o superior) del agente de sincronización de almacenamiento. Las versiones inferiores a la 9 no tendrán los cmdlets de StorageSyncSelfService.

Versiones anteriores es una característica de Windows que permite el uso de instantáneas de VSS del servidor de un volumen para presentar versiones restaurables de un archivo a un cliente de SMB.
Esto permite un escenario eficaz, comúnmente denominado autoservicio de restauración, directamente para los trabajadores de la información en lugar de depender de la restauración de un administrador de TI.

Las instantáneas de VSS y Versiones anteriores funcionan independientemente de Azure File Sync. Sin embargo, la nube por niveles debe establecerse en un modo compatible. Muchos puntos de conexión de Azure File Sync Server pueden existir en el mismo volumen. Debe realizar la siguiente llamada de PowerShell por volumen que tenga incluso un punto de conexión de servidor en el que esté usando o vaya a usar la nube por niveles.

```powershell
Import-Module '<SyncAgentInstallPath>\StorageSync.Management.ServerCmdlets.dll'
Enable-StorageSyncSelfServiceRestore [-DriveLetter] <string> [[-Force]] 
```

Las instantáneas de VSS se obtienen de un volumen completo. De manera predeterminada, pueden existir hasta 64 instantáneas para un volumen determinado, siempre que haya espacio suficiente para almacenar las instantáneas. VSS lo trata automáticamente. La programación de instantáneas predeterminada toma dos instantáneas al día, de lunes a viernes. Esa programación se puede configurar a través de una tarea programada de Windows. El anterior cmdlet de PowerShell hace dos cosas:
1. Configura la nube por niveles de Azure File Sync en el volumen especificado para que sea compatible con las versiones anteriores y garantiza que se puede restaurar un archivo a partir de una versión anterior, aunque se haya organizado por niveles en la nube en el servidor. 
2. Habilita la programación predeterminada de VSS, que puede decidir modificar más adelante. 

> [!Note]  
> Hay dos aspectos importantes que se deben tener en cuenta:
>- Si usa el parámetro -Force y VSS está habilitado actualmente, sobrescribirá la programación de instantáneas de VSS actual y la reemplazará por la programación predeterminada. Asegúrese de guardar la configuración personalizada antes de ejecutar el cmdlet.
> - Si usa este cmdlet en un nodo de clúster, también debe ejecutarlo en todos los demás nodos del clúster. 

Para ver si la compatibilidad con el autoservicio de restauración está habilitada, puede ejecutar el siguiente cmdlet.

```powershell
Get-StorageSyncSelfServiceRestore [[-Driveletter] <string>]
```

Enumerará todos los volúmenes del servidor, así como el número de días compatibles con la nube por niveles para cada uno. Este número se calcula automáticamente en función de las instantáneas máximas posibles por volumen y la programación de instantáneas predeterminada. De manera predeterminada, todas las versiones anteriores presentadas a un trabajador de la información se pueden usar para restaurar a partir de ellas. Se aplica lo mismo si cambia la programación predeterminada para tomar más instantáneas.
Sin embargo, si cambia la programación de forma que se produzca una instantánea disponible en el volumen anterior al valor de días compatibles, los usuarios no podrán usar esta instantánea anterior (versión anterior) para restaurar a partir de ella.

> [!Note]
> La habilitación del autoservicio de restauración puede afectar al consumo y la facturación de Azure Storage. Este impacto se limita a los archivos que están actualmente en capas en el servidor. La habilitación de esta característica garantiza que haya una versión de archivo disponible en la nube a la que se puede hacer referencia a través de una entrada de versiones anteriores (instantánea de VSS).
>
> Si deshabilita la característica, el consumo de Azure Storage disminuirá lentamente hasta que hayan transcurrido los días compatibles. No hay ninguna manera de acelerar este proceso. 

El número máximo predeterminado de instantáneas de VSS por volumen (64), así como la programación predeterminada para tomarlas, da como resultado un máximo de 45 días de versiones anteriores a partir de las que un trabajador de la información puede restaurar en función de cuántas instantáneas de VSS pueda almacenar en el volumen.

Si el máximo de 64 instantáneas de VSS por volumen no corresponde a la configuración correcta, puede [cambiar ese valor mediante una clave del Registro](https://docs.microsoft.com/windows/win32/backup/registry-keys-for-backup-and-restore#maxshadowcopies).
Para que el nuevo límite surta efecto, debe volver a ejecutar el cmdlet para habilitar la compatibilidad con versiones anteriores en todos los volúmenes que se habilitaron anteriormente, con la marca -Force para que tenga en cuenta el nuevo número máximo de instantáneas de VSS por volumen. Esto dará lugar a un número de días compatibles recién calculado. Tenga en cuenta que este cambio solo se aplicará a los archivos recién organizados en capas y sobrescribirá cualquier personalización de la programación de VSS que haya realizado.

<a id="proactive-recall"></a>
## <a name="proactively-recall-new-and-changed-files-from-an-azure-file-share"></a>Recuperación proactiva de archivos nuevos y modificados desde un recurso compartido de archivos de Azure

Con la versión 11 del agente, hay disponible un nuevo modo en un punto de conexión de servidor. Este modo permite a las compañías distribuidas por todo el mundo tener la caché del servidor en una región remota rellenada previamente incluso antes de que los usuarios locales tengan acceso a los archivos. Cuando se habilita en un punto de conexión de servidor, este modo hará que este servidor recupere los archivos creados o modificados en el recurso compartido de archivos de Azure.

### <a name="scenario"></a>Escenario

Una empresa distribuida por todo el mundo tiene sucursales en Estados Unidos y en la India. Por la mañana (hora de EE. UU.), los trabajadores de la información crean una carpeta y los archivos de un nuevo proyecto en el que trabajan todo el día. Azure File Sync sincronizará la carpeta y los archivos con el recurso compartido de archivos de Azure (punto de conexión de nube). Los trabajadores de la información de la India seguirán trabajando en el proyecto en su zona horaria. Cuando lleguen por la mañana, el servidor local de Azure File Sync habilitado en la India debe tener disponibles estos nuevos archivos a nivel local, de modo que el equipo de la India pueda trabajar eficazmente fuera de una caché local. Al habilitar este modo, se evita que el acceso inicial al archivo sea más lento debido a la recuperación a petición y permite que el servidor recupere los archivos de forma proactiva en cuanto se han modificado o creado en el recurso compartido de archivos de Azure.

> [!IMPORTANT]
> Es importante tener en cuentan que el seguimiento de los cambios en el recurso compartido de archivos de Azure en el servidor puede aumentar el tráfico de salida y la factura de Azure. Si los archivos recuperados en el servidor no son realmente necesarios localmente, la recuperación innecesaria en el servidor puede tener consecuencias negativas. Utilice este modo si sabe que rellenar previamente la caché de un servidor con los cambios recientes en la nube tendrá un efecto positivo en los usuarios o las aplicaciones que usan los archivos de dicho servidor.

### <a name="enable-a-server-endpoint-to-proactively-recall-what-changed-in-an-azure-file-share"></a>Habilitación de un punto de conexión de servidor para recuperar de forma proactiva lo que ha cambiado en un recurso compartido de archivos de Azure

# <a name="portal"></a>[Portal](#tab/proactive-portal)

1. En [Azure Portal](https://portal.azure.com/), vaya al servicio de sincronización de almacenamiento, seleccione el grupo de sincronización correcto y, luego, identifique el punto de conexión de servidor cuyos cambios en el recurso compartido de archivos de Azure (punto de conexión en la nube) quiere seguir.
1. En la sección de la nube por niveles, busque el tema "Azure file share download" (Descarga de recursos compartidos de archivos de Azure). Verá el modo seleccionado actualmente, que puede modificarlo para realizar un seguimiento de los cambios en el recurso compartido de archivos de Azure de forma más estrecha y recuperarlos en el servidor.

:::image type="content" source="media/storage-sync-files-deployment-guide/proactive-download.png" alt-text="Imagen que muestra el comportamiento de descarga del recurso compartido de archivos de Azure para un punto de conexión de servidor actualmente en vigor y un botón para abrir un menú que permite cambiarlo.":::

# <a name="powershell"></a>[PowerShell](#tab/proactive-powershell)

Puede modificar las propiedades del punto de conexión de servidor en PowerShell mediante el cmdlet [Set-AzStorageSyncServerEndpoint](https://docs.microsoft.com/powershell/module/az.storagesync/set-azstoragesyncserverendpoint).

```powershell
# Optional parameter. Default: "UpdateLocallyCachedFiles", alternative behavior: "DownloadNewAndModifiedFiles"
$recallBehavior = "DownloadNewAndModifiedFiles"

Set-AzStorageSyncServerEndpoint -InputObject <PSServerEndpoint> -LocalCacheMode $recallBehavior
```

---

## <a name="migrate-a-dfs-replication-dfs-r-deployment-to-azure-file-sync"></a>Migración de una implementación de la replicación DFS (DFS-R) a Azure File Sync
Para migrar una implementación de DFS-R a Azure File Sync:

1. Cree un grupo de sincronización para representar la topología DFS-R que se va a reemplazar.
2. Comience por el servidor que tiene el conjunto completo de datos en la topología DFS-R que se va a migrar. Instale Azure File Sync en ese servidor.
3. Registre ese servidor y cree un punto de conexión de servidor para el primer servidor que desea migrar. No habilite niveles en la nube.
4. Deje que todos los datos se sincronicen con el recurso compartido de archivos de Azure (punto de conexión en la nube).
5. Instale y registre el agente de Azure File Sync en cada uno de los demás servidores de DFS-R.
6. Deshabilite DFS-R. 
7. Cree un punto de conexión de servidor en cada uno de los servidores de DFS-R. No habilite niveles en la nube.
8. Asegúrese de que la sincronización finaliza y pruebe la topología según sea necesario.
9. Retire DFS-R.
10. Ahora se pueden habilitar los niveles en la nube en cualquier punto de conexión de servidor según sea necesario.

Para más información, consulte [Interoperabilidad de Azure File Sync con el sistema de archivos distribuido (DFS)](storage-sync-files-planning.md#distributed-file-system-dfs).

## <a name="next-steps"></a>Pasos siguientes
- [Adición o eliminación de un punto de conexión de servidor de Azure File Sync ](storage-sync-files-server-endpoint.md)
- [Registro y cancelación del registro de un servidor con Azure File Sync](storage-sync-files-server-registration.md)
- [Supervisión de Azure File Sync](storage-sync-files-monitoring.md)
