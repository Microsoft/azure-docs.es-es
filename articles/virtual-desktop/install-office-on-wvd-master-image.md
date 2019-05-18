---
title: 'Instalar Office en una imagen de disco duro virtual principal: Azure'
description: Cómo instalar y personalizar Office en una imagen maestra de vista previa de Escritorio Virtual de Windows en Azure.
services: virtual-desktop
author: ChJenk
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 05/02/2019
ms.author: v-chjenk
ms.openlocfilehash: 92a0ba8c0f43e26b7a1bbe82cc52f61d390fc04d
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65827331"
---
# <a name="install-office-on-a-master-vhd-image"></a>Instalación de Office en la imagen de un disco duro virtual principal

Este artículo explica cómo instalar Office 365 ProPlus, OneDrive y otras aplicaciones comunes en una imagen de disco duro virtual (VHD) maestro para cargarlos en Azure. Si los usuarios necesitan tener acceso a determinadas aplicaciones de línea de negocio (LOB), se recomienda que instalarlos después de completar las instrucciones de este artículo.

En este artículo se da por supuesto que ya ha creado una máquina virtual (VM). Si no, consulte [preparar y personalizar una imagen de disco duro virtual principal](set-up-customize-master-image.md#create-a-vm)

En este artículo también se da por supuesto que acceso ha elevado en la máquina virtual, si se aprovisiona en Azure o Hyper-V Manager. Si no, consulte [elevación del acceso para administrar todos los grupos de administración y la suscripción de Azure](https://docs.microsoft.com/azure/role-based-access-control/elevate-access-global-admin).

>[!NOTE]
>Estas instrucciones son para una configuración de Escritorio Virtual de Windows específicos de la versión preliminar que puede usarse con los procesos existentes de su organización.

## <a name="install-office-in-shared-computer-activation-mode"></a>Instalar Office en modo de activación de equipo compartido

Activación de equipo compartido le permite implementar Office 365 ProPlus en un equipo de la organización que se tiene acceso a varios usuarios. Para obtener más información acerca de la activación de equipos compartidos, consulte [general de activación en equipos compartidos para Office 365 ProPlus](https://docs.microsoft.com/DeployOffice/overview-of-shared-computer-activation-for-office-365-proplus).

Use la [herramienta de implementación de Office](https://www.microsoft.com/download/details.aspx?id=49117) para instalar Office. Varias sesiones de Windows 10 Enterprise solo es compatible con Office 365 ProPlus.

La herramienta de implementación de Office requiere un archivo XML de configuración. Para personalizar el ejemplo siguiente, vea el [las opciones de configuración de la herramienta de implementación de Office](https://docs.microsoft.com/deployoffice/configuration-options-for-the-office-2016-deployment-tool).

Este ejemplo de configuración XML que hemos proporcionado hará lo siguiente:

- Instalar Office desde el canal de Insider y entregar actualizaciones desde el canal Insider cuando sean ejecutados.
- Use el x64 arquitectura.
- Deshabilitar las actualizaciones automáticas.
- Instalación de Visio y Project.
- Quite las instalaciones existentes de Office y migrar su configuración.
- Habilitar la activación de equipos compartidos.

>[!NOTE]
>Característica de búsqueda de galería de símbolos de Visio no funciona en el escritorio Virtual de Windows durante la configuración de vista previa.

Esto es lo que no hacer esta configuración XML de ejemplo:

- Instalación de Skype para la empresa
- Instale OneDrive en modo por el usuario. Para obtener más información, consulte [OneDrive instalar en modo por equipo](#install-onedrive-in-per-machine-mode).

>[!NOTE]
>Activación en equipos compartidos puede configurarse a través de objetos de directiva de grupo (GPO) o la configuración del registro. Se encuentra en el GPO **configuración del equipo\\directivas\\plantillas administrativas\\(equipo) de Microsoft Office 2016\\la configuración de licencias**

La herramienta de implementación de Office contiene setup.exe. Para instalar Office, ejecute el siguiente comando en una línea de comandos:

```batch
Setup.exe /configure configuration.xml
```

#### <a name="sample-configurationxml"></a>Ejemplo configuration.xml

El siguiente ejemplo de XML instalará la versión Insiders.

```xml
<Configuration>
    <Add OfficeClientEdition="64" SourcePath="http://officecdn.microsoft.com/pr/5440fd1f-7ecb-4221-8110-145efaa6372f">
        <Product ID="O365ProPlusRetail">
            <Language ID="en-US" />
            <Language ID="MatchOS" Fallback = "en-US"/>
            <Language ID="MatchPreviousMSI" />
            <ExcludeApp ID="Groove" />
            <ExcludeApp ID="Lync" />
            <ExcludeApp ID="OneDrive" />
            <ExcludeApp ID="Teams" />
        </Product>
        <Product ID="VisioProRetail">
            <Language ID="en-US" />
            <Language ID="MatchOS" Fallback = "en-US"/>
            <Language ID="MatchPreviousMSI" />
            <ExcludeApp ID="Teams" /> 
        </Product>
        <Product ID="ProjectProRetail">
            <Language ID="en-US" />
            <Language ID="MatchOS" Fallback = "en-US"/>
            <Language ID="MatchPreviousMSI" />
            <ExcludeApp ID="Teams" />
        </Product>
    </Add>
    <RemoveMSI All="True" />
    <Updates Enabled="FALSE" UpdatePath="http://officecdn.microsoft.com/pr/5440fd1f-7ecb-4221-8110-145efaa6372f" />
    <Display Level="None" AcceptEULA="TRUE" />
    <Logging Level="Verbose" Path="%temp%\WVDOfficeInstall" />
    <Property Value="TRUE" Name="FORCEAPPSHUTDOWN"/>
    <Property Value="1" Name="SharedComputerLicensing"/>
    <Property Value="TRUE" Name="PinIconsToTaskbar"/>
</Configuration>
```

>[!NOTE]
>El equipo de Office recomienda el uso de la instalación de 64 bits para el **OfficeClientEdition** parámetro.

Después de instalar Office, puede actualizar el comportamiento de Office de forma predeterminada. Ejecute los comandos siguientes individualmente o en un archivo por lotes para el comportamiento de la actualización.

```batch
rem Mount the default user registry hive
reg load HKU\TempDefault C:\Users\Default\NTUSER.DAT
rem Must be executed with default registry hive mounted.
reg add HKU\TempDefault\SOFTWARE\Policies\Microsoft\office\16.0\common /v InsiderSlabBehavior /t REG_DWORD /d 2 /f
rem Set Outlook's Cached Exchange Mode behavior
rem Must be executed with default registry hive mounted.
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v enable /t REG_DWORD /d 1 /f
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v syncwindowsetting /t REG_DWORD /d 1 /f
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v CalendarSyncWindowSetting /t REG_DWORD /d 1 /f
reg add "HKU\TempDefault\software\policies\microsoft\office\16.0\outlook\cached mode" /v CalendarSyncWindowSettingMonths  /t REG_DWORD /d 1 /f
rem Unmount the default user registry hive
reg unload HKU\TempDefault

rem Set the Office Update UI behavior.
reg add HKLM\SOFTWARE\Policies\Microsoft\office\16.0\common\officeupdate /v hideupdatenotifications /t REG_DWORD /d 1 /f
reg add HKLM\SOFTWARE\Policies\Microsoft\office\16.0\common\officeupdate /v hideenabledisableupdates /t REG_DWORD /d 1 /f
```

## <a name="install-onedrive-in-per-machine-mode"></a>Instalar OneDrive en el modo por equipo

OneDrive es normalmente instalados por el usuario. En este entorno, debe ser instalado por equipo.

Aquí le mostramos cómo instalar OneDrive en el modo por equipo:

1. En primer lugar, cree una ubicación para almacenar provisionalmente al instalador de OneDrive. Una carpeta del disco local o [\\\\unc] ubicación (file://unc) es correcto.

2. Descargue OneDriveSetup.exe en la ubicación de ensayo con este vínculo: <https://aka.ms/OneDriveWVD-Installer>

3. Si ha instalado office con OneDrive omitiendo  **\<ExcludeApp ID = "OneDrive" /\>**, desinstalar cualquier las instalaciones existentes por el usuario de OneDrive desde un símbolo del sistema con privilegios elevados, ejecutando el siguiente comando:
    
    ```batch
    "[staged location]\OneDriveSetup.exe" /uninstall
    ```

4. Ejecute este comando desde un símbolo del sistema con privilegios elevados para establecer el **AllUsersInstall** valor del registro:

    ```batch
    REG ADD "HKLM\Software\Microsoft\OneDrive" /v "AllUsersInstall" /t REG_DWORD /d 1 /reg:64
    ```

5. Ejecute este comando para instalar OneDrive en el modo por equipo:

    ```batch
    Run "[staged location]\OneDriveSetup.exe" /allusers
    ```

6. Ejecute este comando para configurar OneDrive para iniciar al inicio de sesión para todos los usuarios:

    ```batch
    REG ADD "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v OneDrive /t REG_SZ /d "C:\Program Files (x86)\Microsoft OneDrive\OneDrive.exe /background" /f
    ```

7. Habilitar **Configurar cuenta de usuario de forma silenciosa** , ejecute el comando siguiente.

    ```batch
    REG ADD "HKLM\SOFTWARE\Policies\Microsoft\OneDrive" /v "SilentAccountConfig" /t REG_DWORD /d 1 /f
    ```

8. Redirigir y mover conocida las carpetas en OneDrive, ejecute el siguiente comando de Windows.

    ```batch
    REG ADD "HKLM\SOFTWARE\Policies\Microsoft\OneDrive" /v "KFMSilentOptIn" /t REG_SZ /d "<your-AzureAdTenantId>" /f
    ```

## <a name="teams-and-skype"></a>Los equipos y Skype

Escritorio Virtual de Windows no es compatible con Skype para la empresa y los equipos.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha agregado a la imagen de Office, puede personalizar la imagen de disco duro virtual principal. Consulte [preparar y personalizar una imagen de disco duro virtual principal](set-up-customize-master-image.md).
