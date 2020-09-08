---
title: 'Tutorial: entorno básico de Active Directory local y de Azure AD.'
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 12/02/2019
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e8a6fc1291615760320deb488511fdd9d964d25d
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89227872"
---
# <a name="tutorial-basic-active-directory-environment"></a>Tutorial: Entorno básico de Active Directory

Este tutorial le guiará a través de la creación de un entorno de Active Directory básico. 

![Crear](media/tutorial-single-forest/diagram1.png)

Puede usar el entorno que crea en el tutorial para probar varios aspectos de escenarios de identidad híbrida, lo que será un requisito previo para algunos de los tutoriales.  Si ya tiene un entorno de Active Directory existente, puede usarlo como sustituto.  Esta información se proporciona para usuarios que pueden estar empezando de cero.

Este tutorial consta de
## <a name="prerequisites"></a>Prerrequisitos
A continuación, se indican los requisitos previos necesarios para completar este tutorial:
- Un equipo con [Hyper-V](/windows-server/virtualization/hyper-v/hyper-v-technology-overview) instalado.  Se recomienda hacer esto en un equipo con [Windows 10](/virtualization/hyper-v-on-windows/about/supported-guest-os) o [Windows Server 2016](/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows).
- Un [adaptador de red externo](/virtualization/hyper-v-on-windows/quick-start/connect-to-network) para permitir que la máquina virtual se comunique con Internet.
- [Una suscripción de Azure](https://azure.microsoft.com/free)
- Una copia de Windows Server 2016.
- [Microsoft .NET Framework 4.7.1](https://www.microsoft.com/download/details.aspx?id=56115)

> [!NOTE]
> En este tutorial se utilizan scripts de PowerShell para que pueda crear el entorno de tutorial de la forma más rápida posible.  Cada uno de los scripts usa variables que se declaran al principio de estos.  Puede y debe cambiar las variables para reflejar su entorno.
>
>Los scripts usados crean un entorno general de Active Directory antes de instalar el agente de aprovisionamiento en la nube de Azure AD Connect.  Son relevantes para todos los tutoriales.
>
> Puede encontrar copias de los scripts de PowerShell que se usan en este tutorial disponibles en GitHub [aquí](https://github.com/billmath/tutorial-phs).

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual
Lo primero que debe hacer con el fin de poner en funcionamiento el entorno de identidad híbrida es crear una máquina virtual que se usará como nuestro servidor local de Active Directory.  Haga lo siguiente:

1. Abra PowerShell ISE como administrador.
2. Ejecute el siguiente script.

    ```powershell
    #Declare variables
    $VMName = 'DC1'
    $Switch = 'External'
    $InstallMedia = 'D:\ISO\en_windows_server_2016_updated_feb_2018_x64_dvd_11636692.iso'
    $Path = 'D:\VM'
    $VHDPath = 'D:\VM\DC1\DC1.vhdx'
    $VHDSize = '64424509440'

    #Create New Virtual Machine
    New-VM -Name $VMName -MemoryStartupBytes 16GB -BootDevice VHD -Path $Path -NewVHDPath $VHDPath -NewVHDSizeBytes $VHDSize  -Generation 2 -Switch $Switch  

    #Set the memory to be non-dynamic
    Set-VMMemory $VMName -DynamicMemoryEnabled $false

    #Add DVD Drive to Virtual Machine
    Add-VMDvdDrive -VMName $VMName -ControllerNumber 0 -ControllerLocation 1 -Path $InstallMedia

    #Mount Installation Media
    $DVDDrive = Get-VMDvdDrive -VMName $VMName

    #Configure Virtual Machine to Boot from DVD
    Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive 
    ```

## <a name="complete-the-operating-system-deployment"></a>Finalización de la implementación de sistema operativo
Para terminar de crear la máquina virtual, deberá finalizar la instalación del sistema operativo.

1. En el Administrador de Hyper-V, haga doble clic en la máquina virtual.
2. Haga clic en el botón Iniciar.
3. Se le solicitará que realice la siguiente acción: "Press any key to boot from CD or DVD" ("Presionar cualquier tecla para realizar un arranque desde CD o DVD"). Continúe.
4. En la pantalla de inicio de Windows Server, seleccione el idioma y haga clic en **Siguiente**.
5. Haga clic en **Instalar ahora**.
6. Escriba la clave de licencia y haga clic en **Siguiente**.
7. Marque **Acepto los términos de licencia y haga clic en **Siguiente**.
8. Seleccione **Personalizada:  instalar solo Windows (avanzado)**
9. Haga clic en **Siguiente**.
10. Una vez completada la instalación, reinicie la máquina virtual, inicie sesión y ejecute las actualizaciones de Windows para asegurarse de que la versión de la máquina virtual es la más actualizada.  Instale las actualizaciones más recientes.

## <a name="install-active-directory-prerequisites"></a>Requisitos previos de la instalación de Active Directory
Ahora que tiene una máquina virtual en funcionamiento, debe realizar algunas acciones antes de instalar Active Directory.  Es necesario cambiar el nombre de la máquina virtual, establecer una dirección IP estática y la información de DNS e instalar las herramientas de administración remota del servidor.   Haga lo siguiente:

1. Abra PowerShell ISE como administrador.
2. Ejecute el siguiente script.

    ```powershell
    #Declare variables
    $ipaddress = "10.0.1.117" 
    $ipprefix = "24" 
    $ipgw = "10.0.1.1" 
    $ipdns = "10.0.1.117"
    $ipdns2 = "8.8.8.8" 
    $ipif = (Get-NetAdapter).ifIndex 
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $newname = "DC1"
    $addsTools = "RSAT-AD-Tools" 

    #Set static IP address
    New-NetIPAddress -IPAddress $ipaddress -PrefixLength $ipprefix -InterfaceIndex $ipif -DefaultGateway $ipgw 

    # Set the DNS servers
    Set-DnsClientServerAddress -InterfaceIndex $ipif -ServerAddresses ($ipdns, $ipdns2)

    #Rename the computer 
    Rename-Computer -NewName $newname -force 

    #Install features 
    New-Item $featureLogPath -ItemType file -Force 
    Add-WindowsFeature $addsTools 
    Get-WindowsFeature | Where installed >>$featureLogPath 

    #Restart the computer 
    Restart-Computer
    ```

## <a name="create-a-windows-server-ad-environment"></a>Creación de un entorno de Windows Server AD
Ahora que ha creado la máquina virtual, cambiado su nombre y establecido una dirección IP estática, puede continuar con la instalación y configuración de Active Directory Domain Services.  Haga lo siguiente:

1. Abra PowerShell ISE como administrador.
2. Ejecute el siguiente script.

    ```powershell 
    #Declare variables
    $DatabasePath = "c:\windows\NTDS"
    $DomainMode = "WinThreshold"
    $DomainName = "contoso.com"
    $DomaninNetBIOSName = "CONTOSO"
    $ForestMode = "WinThreshold"
    $LogPath = "c:\windows\NTDS"
    $SysVolPath = "c:\windows\SYSVOL"
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $Password = "Pass1w0rd"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force

    #Install AD DS, DNS and GPMC 
    start-job -Name addFeature -ScriptBlock { 
    Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "dns" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "gpmc" -IncludeAllSubFeature -IncludeManagementTools } 
    Wait-Job -Name addFeature 
    Get-WindowsFeature | Where installed >>$featureLogPath

    #Create New AD Forest
    Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath $DatabasePath -DomainMode $DomainMode -DomainName $DomainName -SafeModeAdministratorPassword $SecureString -DomainNetbiosName $DomainNetBIOSName -ForestMode $ForestMode -InstallDns:$true -LogPath $LogPath -NoRebootOnCompletion:$false -SysvolPath $SysVolPath -Force:$true
    ```

## <a name="create-a-windows-server-ad-user"></a>Creación de un usuario de Windows Server AD
Ahora que tiene nuestro entorno de Active Directory, necesita una cuenta de prueba.  Esta cuenta se creará en nuestro entorno local de AD y, a continuación, se sincronizará con Azure AD.  Haga lo siguiente:

1. Abra PowerShell ISE como administrador.
2. Ejecute el siguiente script.

    ```powershell 
    # Filename:    4_CreateUser.ps1
    # Description: Creates a user in Active Directory.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $Givenname = "Allie"
    $Surname = "McCray"
    $Displayname = "Allie McCray"
    $Name = "amccray"
    $Password = "Pass1w0rd"
    $Identity = "CN=ammccray,CN=Users,DC=contoso,DC=com"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force


    #Create the user
    New-ADUser -Name $Name -GivenName $Givenname -Surname $Surname -DisplayName $Displayname -AccountPassword $SecureString

    #Set the password to never expire
    Set-ADUser -Identity $Identity -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Enabled $true
    ```


## <a name="create-an-azure-ad-tenant"></a>Creación de un inquilino de Azure AD
Ahora debe crear un inquilino de Azure AD para poder sincronizar nuestros usuarios con la nube.  Para crear un nuevo inquilino de Azure AD, haga lo siguiente:

1. Vaya a [Azure Portal](https://portal.azure.com) e inicie sesión con una cuenta que tenga una suscripción de Azure.
2. Seleccione el **icono de signo más (+)** y busque **Azure Active Directory**.
3. En la lista de resultados, seleccione **Azure Active Directory**.
4. Seleccione **Crear**.</br>
![Creación](media/tutorial-single-forest/create1.png)</br>
5. Proporcione un **nombre para la organización**, junto con el **nombre de dominio inicial**. Seleccione **Crear**. Se creará el directorio.
6. Una vez que se haya completado, haga clic en el vínculo **aquí** para administrar el directorio.

## <a name="create-a-global-administrator-in-azure-ad"></a>Creación de un administrador global en Azure AD
Ahora que tiene un inquilino de Azure AD, creará una cuenta de administrador global.  Para crear la cuenta de administrador global, haga lo siguiente:

1.  En **Administrar**, seleccione **Usuarios**.</br>
![Creación](media/tutorial-single-forest/administrator1.png)</br>
2.  Seleccione **Todos los usuarios** y, a continuación, seleccione **+ Nuevo usuario**.
3.  Proporcione un nombre y el nombre de usuario para este usuario. Este será el administrador global del inquilino. También deberá cambiar el **rol del directorio** a **Administrador global**. También puede mostrar la contraseña temporal. Cuando termine, seleccione **Crear**.</br>
![Creación](media/tutorial-single-forest/administrator2.png)</br>
4. Una vez completado, abra un nuevo explorador web e inicie sesión en myapps.microsoft.com con la nueva cuenta de administrador global y la contraseña temporal.
5. Cambie la contraseña de administrador global por algo que pueda recordar.

## <a name="optional--additional-server-and-forest"></a>Opcional:  Servidor y bosque adicional
La siguiente es una sección opcional que proporciona los pasos necesarios para crear un servidor o bosque adicional.  Este puede usarse en algunos de los tutoriales más avanzados, como [Realización de una prueba piloto para el aprovisionamiento en la nube de Azure AD Connect](tutorial-pilot-aadc-aadccp.md).

Si solo necesita un servidor adicional, puede detenerse después del paso de **creación de una máquina virtual** y unir el servidor al dominio existente que se creó anteriormente.  

### <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

1. Abra PowerShell ISE como administrador.
2. Ejecute el siguiente script.

    ```powershell
    # Filename:    1_CreateVM_CP.ps1
    # Description: Creates a VM to be used in the tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. #This script is made available to you without any express, implied or statutory warranty, not even the implied warranty of merchantability or fitness for a particular purpose, or the warranty of title or non-infringement. The entire risk of the use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $VMName = 'CP1'
    $Switch = 'External'
    $InstallMedia = 'D:\ISO\en_windows_server_2016_updated_feb_2018_x64_dvd_11636692.iso'
    $Path = 'D:\VM'
    $VHDPath = 'D:\VM\CP1\CP1.vhdx'
    $VHDSize = '64424509440'

    #Create New Virtual Machine
    New-VM -Name $VMName -MemoryStartupBytes 16GB -BootDevice VHD -Path $Path -NewVHDPath $VHDPath -NewVHDSizeBytes $VHDSize  -Generation 2 -Switch $Switch  

    #Set the memory to be non-dynamic
    Set-VMMemory $VMName -DynamicMemoryEnabled $false

    #Add DVD Drive to Virtual Machine
    Add-VMDvdDrive -VMName $VMName -ControllerNumber 0 -ControllerLocation 1 -Path $InstallMedia

    #Mount Installation Media
    $DVDDrive = Get-VMDvdDrive -VMName $VMName

    #Configure Virtual Machine to Boot from DVD
    Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive
    ```

### <a name="complete-the-operating-system-deployment"></a>Finalización de la implementación de sistema operativo
Para terminar de crear la máquina virtual, deberá finalizar la instalación del sistema operativo.

1. En el Administrador de Hyper-V, haga doble clic en la máquina virtual.
2. Haga clic en el botón Iniciar.
3. Se le solicitará que realice la siguiente acción: "Press any key to boot from CD or DVD" ("Presionar cualquier tecla para realizar un arranque desde CD o DVD"). Continúe.
4. En la pantalla de inicio de Windows Server, seleccione el idioma y haga clic en **Siguiente**.
5. Haga clic en **Instalar ahora**.
6. Escriba la clave de licencia y haga clic en **Siguiente**.
7. Marque **Acepto los términos de licencia y haga clic en **Siguiente**.
8. Seleccione **Personalizada:  instalar solo Windows (avanzado)**
9. Haga clic en **Siguiente**.
10. Una vez completada la instalación, reinicie la máquina virtual, inicie sesión y ejecute las actualizaciones de Windows para asegurarse de que la versión de la máquina virtual es la más actualizada.  Instale las actualizaciones más recientes.

### <a name="install-active-directory-prerequisites"></a>Requisitos previos de la instalación de Active Directory
Ahora que tiene una máquina virtual en funcionamiento, debe realizar algunas acciones antes de instalar Active Directory.  Es necesario cambiar el nombre de la máquina virtual, establecer una dirección IP estática y la información de DNS e instalar las herramientas de administración remota del servidor.   Haga lo siguiente:

1. Abra PowerShell ISE como administrador.
2. Ejecute el siguiente script.

    ```powershell
    # Filename:    2_ADPrep_CP.ps1
    # Description: Prepares your environment for Active Directory.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $ipaddress = "10.0.1.118" 
    $ipprefix = "24" 
    $ipgw = "10.0.1.1" 
    $ipdns = "10.0.1.118"
    $ipdns2 = "8.8.8.8" 
    $ipif = (Get-NetAdapter).ifIndex 
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $newname = "CP1"
    $addsTools = "RSAT-AD-Tools" 

    #Set static IP address
    New-NetIPAddress -IPAddress $ipaddress -PrefixLength $ipprefix -InterfaceIndex $ipif -DefaultGateway $ipgw 

    #Set the DNS servers
    Set-DnsClientServerAddress -InterfaceIndex $ipif -ServerAddresses ($ipdns, $ipdns2)

    #Rename the computer 
    Rename-Computer -NewName $newname -force 

    #Install features 
    New-Item $featureLogPath -ItemType file -Force 
    Add-WindowsFeature $addsTools 
    Get-WindowsFeature | Where installed >>$featureLogPath 

    #Restart the computer 
    Restart-Computer
    ```
### <a name="create-a-windows-server-ad-environment"></a>Creación de un entorno de Windows Server AD
Ahora que ha creado la máquina virtual, cambiado su nombre y establecido una dirección IP estática, puede continuar con la instalación y configuración de Active Directory Domain Services.  Haga lo siguiente:

1. Abra PowerShell ISE como administrador.
2. Ejecute el siguiente script.

    ```powershell
    # Filename:    3_InstallAD_CP.ps1
    # Description: Creates an on-premises AD envrionment.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $DatabasePath = "c:\windows\NTDS"
    $DomainMode = "WinThreshold"
    $DomainName = "fabrikam.com"
    $DomaninNetBIOSName = "FABRIKAM"
    $ForestMode = "WinThreshold"
    $LogPath = "c:\windows\NTDS"
    $SysVolPath = "c:\windows\SYSVOL"
    $featureLogPath = "c:\poshlog\featurelog.txt" 
    $Password = "Pass1w0rd"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force

    #Install AD DS, DNS and GPMC 
    start-job -Name addFeature -ScriptBlock { 
    Add-WindowsFeature -Name "ad-domain-services" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "dns" -IncludeAllSubFeature -IncludeManagementTools 
    Add-WindowsFeature -Name "gpmc" -IncludeAllSubFeature -IncludeManagementTools } 
    Wait-Job -Name addFeature 
    Get-WindowsFeature | Where installed >>$featureLogPath

    #Create New AD Forest
    Install-ADDSForest -CreateDnsDelegation:$false -DatabasePath $DatabasePath -DomainMode $DomainMode -DomainName $DomainName -SafeModeAdministratorPassword $SecureString -DomainNetbiosName $DomainNetBIOSName -ForestMode $ForestMode -InstallDns:$true -LogPath $LogPath -NoRebootOnCompletion:$false -SysvolPath $SysVolPath -Force:$true
    ```

### <a name="create-a-windows-server-ad-user"></a>Creación de un usuario de Windows Server AD
Ahora que tiene nuestro entorno de Active Directory, necesita una cuenta de prueba.  Esta cuenta se creará en nuestro entorno local de AD y, a continuación, se sincronizará con Azure AD.  Haga lo siguiente:

1. Abra PowerShell ISE como administrador.
2. Ejecute el siguiente script.

    ```powershell 
    # Filename:    4_CreateUser_CP.ps1
    # Description: Creates a user in Active Directory.  This is part of
    #              the Azure AD Connect password hash sync tutorial.
    #
    # DISCLAIMER:
    # Copyright (c) Microsoft Corporation. All rights reserved. This 
    # script is made available to you without any express, implied or 
    # statutory warranty, not even the implied warranty of 
    # merchantability or fitness for a particular purpose, or the 
    # warranty of title or non-infringement. The entire risk of the 
    # use or the results from the use of this script remains with you.
    #
    #
    #
    #
    #Declare variables
    $Givenname = "Anna"
    $Surname = "Ringdal"
    $Displayname = "Anna Ringdal"
    $Name = "aringdal"
    $Password = "Pass1w0rd"
    $Identity = "CN=aringdal,CN=Users,DC=fabrikam,DC=com"
    $SecureString = ConvertTo-SecureString $Password -AsPlainText -Force


    #Create the user
    New-ADUser -Name $Name -GivenName $Givenname -Surname $Surname -DisplayName $Displayname -AccountPassword $SecureString

    #Set the password to never expire
    Set-ADUser -Identity $Identity -PasswordNeverExpires $true -ChangePasswordAtLogon $false -Enabled $true
    ```

## <a name="conclusion"></a>Conclusión
Ahora tiene un entorno que se puede usar para los tutoriales existentes y para probar características adicionales que proporciona el aprovisionamiento en la nube.

## <a name="next-steps"></a>Pasos siguientes 

- [¿Qué es el aprovisionamiento?](what-is-provisioning.md)
- [¿Qué es el aprovisionamiento en la nube de Azure AD Connect?](what-is-cloud-provisioning.md)