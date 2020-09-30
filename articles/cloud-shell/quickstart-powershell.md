---
title: 'Inicio rápido de Azure Cloud Shell: PowerShell'
description: Aprenda a usar PowerShell en el explorador con Azure Cloud Shell.
author: maertendmsft
ms.author: damaerte
tags: azure-resource-manager
ms.service: azure
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: article
ms.date: 10/18/2018
ms.openlocfilehash: d4a7f1453ec686cfa16d260101ba81f429ce1da0
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89469463"
---
# <a name="quickstart-for-powershell-in-azure-cloud-shell"></a>Guía de inicio rápido de PowerShell en Azure Cloud Shell

En este documento se detalla cómo usar PowerShell en Cloud Shell en [Azure Portal](https://portal.azure.com/).

> [!NOTE]
> También hay disponible una guía de inicio rápido de [Bash en Azure Cloud Shell](quickstart.md).

## <a name="start-cloud-shell"></a>Inicio de Cloud Shell

1. Haga clic en el botón **Cloud Shell** en la barra de navegación superior de Azure Portal

   ![Captura de pantalla en la que se muestra cómo iniciar Azure Cloud Shell desde Azure Portal.](media/quickstart-powershell/shell-icon.png)

2. Seleccione el entorno PowerShell en el menú desplegable y estará en la unidad `(Azure:)` de Azure

   ![Captura de pantalla en la que se muestra cómo seleccionar el entorno de PowerShell para Azure Cloud Shell.](media/quickstart-powershell/environment-ps.png)

## <a name="run-powershell-commands"></a>Ejecución de comandos de PowerShell

Ejecute los comandos habituales de PowerShell en Cloud Shell, como:

```azurepowershell-interactive
PS Azure:\> Get-Date

# Expected Output
Friday, July 27, 2018 7:08:48 AM

PS Azure:\> Get-AzVM -Status

# Expected Output
ResourceGroupName       Name       Location                VmSize   OsType     ProvisioningState  PowerState
-----------------       ----       --------                ------   ------     -----------------  ----------
MyResourceGroup2        Demo        westus         Standard_DS1_v2  Windows    Succeeded           running
MyResourceGroup         MyVM1       eastus            Standard_DS1  Windows    Succeeded           running
MyResourceGroup         MyVM2       eastus   Standard_DS2_v2_Promo  Windows    Succeeded           deallocated
```

## <a name="navigate-azure-resources"></a>Navegación de los recursos de Azure

 1. Enumere todas las suscripciones de la unidad `Azure`.

    ```azurepowershell-interactive
    PS Azure:\> dir
    ```

 2. `cd` a la suscripción de su preferencia

    ```azurepowershell-interactive
    PS Azure:\> cd MySubscriptionName
    PS Azure:\MySubscriptionName>
    ```

 3. Vea todos los recursos de Azure en la suscripción actual

    Escriba `dir` para mostrar varias vistas de los recursos de Azure.

    ```azurepowershell-interactive
    PS Azure:\MySubscriptionName> dir

        Directory: azure:\MySubscriptionName

    Mode Name
    ---- ----
    +    AllResources
    +    ResourceGroups
    +    StorageAccounts
    +    VirtualMachines
    +    WebApps
    ```

### <a name="allresources-view"></a>Vista AllResources

Escriba `dir` en el directorio `AllResources` para ver los recursos de Azure.

```azurepowershell-interactive
PS Azure:\MySubscriptionName> dir AllResources
```

### <a name="explore-resource-groups"></a>Exploración de grupos de recursos

 Puede ir al directorio `ResourceGroups` y encontrar máquinas virtuales dentro de un grupo de recursos específicos.

```azurepowershell-interactive
PS Azure:\MySubscriptionName> cd ResourceGroups\MyResourceGroup1\Microsoft.Compute\virtualMachines

PS Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup1\Microsoft.Compute\virtualMachines> dir


    Directory: Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup1\Microsoft.Compute\virtualMachines


VMName    Location   ProvisioningState VMSize          OS            SKU             OSVersion AdminUserName  NetworkInterfaceName
------    --------   ----------------- ------          --            ---             --------- -------------  --------------------
TestVm1   westus     Succeeded         Standard_DS2_v2 WindowsServer 2016-Datacenter Latest    AdminUser      demo371
TestVm2   westus     Succeeded         Standard_DS1_v2 WindowsServer 2016-Datacenter Latest    AdminUser      demo271
```

> [!NOTE]
> Es posible que observe que la segunda vez que escribe `dir`, Cloud Shell muestra los elementos mucho más rápido.
> Esto se debe a que los elementos secundarios se almacenan en la memoria caché para brindar una mejor experiencia del usuario.
Sin embargo, siempre puede usar `dir -Force` para obtener datos actualizados.

### <a name="navigate-storage-resources"></a>Navegación de los recursos de almacenamiento

Para navegar fácilmente por los recursos de almacenamiento, acceda al directorio `StorageAccounts`.

```azurepowershell-interactive
PS Azure:\MySubscriptionName\StorageAccounts\MyStorageAccountName\Files> dir

    Directory: Azure:\MySubscriptionNameStorageAccounts\MyStorageAccountName\Files

Name          ConnectionString
----          ----------------
MyFileShare1  \\MyStorageAccountName.file.core.windows.net\MyFileShare1;AccountName=MyStorageAccountName AccountKey=<key>
MyFileShare2  \\MyStorageAccountName.file.core.windows.net\MyFileShare2;AccountName=MyStorageAccountName AccountKey=<key>
MyFileShare3  \\MyStorageAccountName.file.core.windows.net\MyFileShare3;AccountName=MyStorageAccountName AccountKey=<key>
```

Con la cadena de conexión, puede usar el comando siguiente para montar el recurso compartido de Azure Files.

```azurepowershell-interactive
net use <DesiredDriveLetter>: \\<MyStorageAccountName>.file.core.windows.net\<MyFileShareName> <AccountKey> /user:Azure\<MyStorageAccountName>
```

Para detalles, consulte [Montaje de un recurso compartido de Azure Files y acceso al recurso compartido en Windows][azmount].

También puede navegar en los directorios del recurso compartido de Azure Files de la manera siguiente:

```azurepowershell-interactive
PS Azure:\MySubscriptionName\StorageAccounts\MyStorageAccountName\Files> cd .\MyFileShare1\
PS Azure:\MySubscriptionName\StorageAccounts\MyStorageAccountName\Files\MyFileShare1> dir

Mode  Name
----  ----
+     TestFolder
.     hello.ps1
```

### <a name="interact-with-virtual-machines"></a>Interacción con máquinas virtuales

Puede encontrar todas las máquinas virtuales de la suscripción actual mediante el directorio `VirtualMachines`.

```azurepowershell-interactive
PS Azure:\MySubscriptionName\VirtualMachines> dir

    Directory: Azure:\MySubscriptionName\VirtualMachines


Name       ResourceGroupName  Location  VmSize          OsType              NIC ProvisioningState  PowerState
----       -----------------  --------  ------          ------              --- -----------------  ----------
TestVm1    MyResourceGroup1   westus    Standard_DS2_v2 Windows       my2008r213         Succeeded     stopped
TestVm2    MyResourceGroup1   westus    Standard_DS1_v2 Windows          jpstest         Succeeded deallocated
TestVm10   MyResourceGroup2   eastus    Standard_DS1_v2 Windows           mytest         Succeeded     running
```

#### <a name="invoke-powershell-script-across-remote-vms"></a>Invocación del script de PowerShell a través de máquinas virtuales remotas

 > [!WARNING]
 > Consulte el artículo sobre la [solución de problemas de la administración remota de máquinas virtuales de Azure](troubleshooting.md#troubleshooting-remote-management-of-azure-vms).

  Supongamos que tiene una máquina virtual llamada MyVM1. Usemos entonces `Invoke-AzVMCommand` para invocar un bloque de scripts de PowerShell en la máquina remota.

  ```azurepowershell-interactive
  Enable-AzVMPSRemoting -Name MyVM1 -ResourceGroupname MyResourceGroup
  Invoke-AzVMCommand -Name MyVM1 -ResourceGroupName MyResourceGroup -Scriptblock {Get-ComputerInfo} -Credential (Get-Credential)
  ```

  También puede ir primero al directorio VirtualMachines y ejecutar `Invoke-AzVMCommand` de la manera siguiente.

  ```azurepowershell-interactive
  PS Azure:\> cd MySubscriptionName\ResourceGroups\MyResourceGroup\Microsoft.Compute\virtualMachines
  PS Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup\Microsoft.Compute\virtualMachines> Get-Item MyVM1 | Invoke-AzVMCommand -Scriptblock {Get-ComputerInfo} -Credential (Get-Credential)

  # You will see output similar to the following:

  PSComputerName                                          : 65.52.28.207
  RunspaceId                                              : 2c2b60da-f9b9-4f42-a282-93316cb06fe1
  WindowsBuildLabEx                                       : 14393.1066.amd64fre.rs1_release_sec.170327-1835
  WindowsCurrentVersion                                   : 6.3
  WindowsEditionId                                        : ServerDatacenter
  WindowsInstallationType                                 : Server
  WindowsInstallDateFromRegistry                          : 5/18/2017 11:26:08 PM
  WindowsProductId                                        : 00376-40000-00000-AA947
  WindowsProductName                                      : Windows Server 2016 Datacenter
  WindowsRegisteredOrganization                           :
   ...
  ```

#### <a name="interactively-log-on-to-a-remote-vm"></a>Inicio de sesión interactivo en una máquina virtual remota

Puede usar `Enter-AzVM` para iniciar sesión de manera interactiva en una máquina virtual que se ejecuta en Azure.

  ```azurepowershell-interactive
  PS Azure:\> Enter-AzVM -Name MyVM1 -ResourceGroupName MyResourceGroup -Credential (Get-Credential)
  ```

También puede navegar primero al directorio `VirtualMachines` y ejecute `Enter-AzVM` de la siguiente manera

  ```azurepowershell-interactive
 PS Azure:\MySubscriptionName\ResourceGroups\MyResourceGroup\Microsoft.Compute\virtualMachines> Get-Item MyVM1 | Enter-AzVM -Credential (Get-Credential)
 ```

### <a name="discover-webapps"></a>Detección de WebApps

Para navegar fácilmente por los recursos de aplicaciones web, acceda al directorio `WebApps`.

```azurepowershell-interactive
PS Azure:\MySubscriptionName> dir .\WebApps\

    Directory: Azure:\MySubscriptionName\WebApps

Name            State    ResourceGroup      EnabledHostNames                  Location
----            -----    -------------      ----------------                  --------
mywebapp1       Stopped  MyResourceGroup1   {mywebapp1.azurewebsites.net...   West US
mywebapp2       Running  MyResourceGroup2   {mywebapp2.azurewebsites.net...   West Europe
mywebapp3       Running  MyResourceGroup3   {mywebapp3.azurewebsites.net...   South Central US

# You can use Azure cmdlets to Start/Stop your web apps
PS Azure:\MySubscriptionName\WebApps> Start-AzWebApp -Name mywebapp1 -ResourceGroupName MyResourceGroup1

Name           State    ResourceGroup        EnabledHostNames                   Location
----           -----    -------------        ----------------                   --------
mywebapp1      Running  MyResourceGroup1     {mywebapp1.azurewebsites.net ...   West US

# Refresh the current state with -Force
PS Azure:\MySubscriptionName\WebApps> dir -Force

    Directory: Azure:\MySubscriptionName\WebApps

Name            State    ResourceGroup      EnabledHostNames                  Location
----            -----    -------------      ----------------                  --------
mywebapp1       Running  MyResourceGroup1   {mywebapp1.azurewebsites.net...   West US
mywebapp2       Running  MyResourceGroup2   {mywebapp2.azurewebsites.net...   West Europe
mywebapp3       Running  MyResourceGroup3   {mywebapp3.azurewebsites.net...   South Central US
```

## <a name="ssh"></a>SSH

Para realizar la autenticación en servidores o máquinas virtuales mediante SSH, genere el par de claves pública y privada en Cloud Shell y publique la clave pública en `authorized_keys` en la máquina remota, por ejemplo, `/home/user/.ssh/authorized_keys`.

> [!NOTE]
> Puede crear las claves pública y privada de SSH mediante `ssh-keygen` y publicarlas en `$env:USERPROFILE\.ssh` en Cloud Shell.

### <a name="using-ssh"></a>Uso de SSH

Siga las instrucciones que se indican [aquí](../virtual-machines/linux/quick-create-powershell.md) para crear una nueva configuración de máquina virtual mediante cmdlets de Azure PowerShell.
Antes de llamar a `New-AzVM` para iniciar la implementación, agregue la clave pública SSH a la configuración de la máquina virtual.
La máquina virtual recién creada contendrá la clave pública en la ubicación `~\.ssh\authorized_keys`, lo cual permitirá una sesión SSH sin credenciales en la máquina virtual.

```azurepowershell-interactive
# Create VM config object - $vmConfig using instructions on linked page above

# Generate SSH keys in Cloud Shell
ssh-keygen -t rsa -b 2048 -f $HOME\.ssh\id_rsa 

# Ensure VM config is updated with SSH keys
$sshPublicKey = Get-Content "$HOME\.ssh\id_rsa.pub"
Add-AzVMSshPublicKey -VM $vmConfig -KeyData $sshPublicKey -Path "/home/azureuser/.ssh/authorized_keys"

# Create a virtual machine
New-AzVM -ResourceGroupName <yourResourceGroup> -Location <vmLocation> -VM $vmConfig

# SSH to the VM
ssh azureuser@MyVM.Domain.Com
```

## <a name="list-available-commands"></a>Enumeración de los comandos disponibles

En la unidad `Azure`, escriba `Get-AzCommand` para obtener comandos de Azure específicos del contexto.

De manera alternativa, siempre puede usar `Get-Command *az* -Module Az.*` para saber cuáles son los comandos disponibles de Azure.

## <a name="install-custom-modules"></a>Instalación de módulos personalizados

Puede ejecutar `Install-Module` para instalar módulos desde la [Galería de PowerShell][gallery].

## <a name="get-help"></a>Get-Help

Escriba `Get-Help` para obtener información sobre PowerShell en Azure Cloud Shell.

```azurepowershell-interactive
Get-Help
```

En un comando específico, todavía puede ejecutar `Get-Help` seguido de un cmdlet.

```azurepowershell-interactive
Get-Help Get-AzVM
```

## <a name="use-azure-files-to-store-your-data"></a>Uso de Azure Files para almacenar los datos

Para crear un script, diga `helloworld.ps1` y guárdelo en `clouddrive` para usarlo en distintas sesiones de shell.

```azurepowershell-interactive
cd $HOME\clouddrive
# Create a new file in clouddrive directory
New-Item helloworld.ps1
# Open the new file for editing
code .\helloworld.ps1
# Add the content, such as 'Hello World!'
.\helloworld.ps1
Hello World!
```

La próxima vez que use PowerShell en Cloud Shell, el archivo `helloworld.ps1` existirá en el directorio `$HOME\clouddrive` que monta el recurso compartido de Azure Files.

## <a name="use-custom-profile"></a>Uso de un perfil personalizado

Puede crear los perfiles de PowerShell `profile.ps1` o `Microsoft.PowerShell_profile.ps1` para personalizar el entorno de PowerShell.
Guárdelo en `$profile.CurrentUserAllHosts` (o `$profile.CurrentUserAllHosts`), de modo que se pueda cargar en cada sesión de PowerShell en Cloud Shell.

Para saber cómo crear un perfil, consulte la [información sobre los perfiles][profile].

## <a name="use-git"></a>Uso de Git

Para clonar un repositorio Git en Cloud Shell, debe crear un [token de acceso personal][githubtoken] y usarlo como el nombre de usuario. Una vez que tenga el token, clone el repositorio de la manera siguiente:

```azurepowershell-interactive
  git clone https://<your-access-token>@github.com/username/repo.git
```

## <a name="exit-the-shell"></a>Salga del shell

Escriba `exit` para finalizar la sesión.

[bashqs]:quickstart.md
[gallery]:https://www.powershellgallery.com/
[customex]:https://docs.microsoft.com/azure/virtual-machines/windows/extensions-customscript
[profile]: /powershell/module/microsoft.powershell.core/about/about_profiles
[azmount]: ../storage/files/storage-how-to-use-files-windows.md
[githubtoken]: https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/