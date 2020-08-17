---
title: 'Instalación de Office en la imagen de un disco duro virtual maestro: Azure'
description: Cómo instalar y personalizar Office en una imagen maestra de Windows Virtual Desktop en Azure.
author: Heidilohr
ms.topic: how-to
ms.date: 05/02/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: 3e53d8bf8f7cb024b468983f596d3d1bd5c91ee7
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88007308"
---
# <a name="install-office-on-a-master-vhd-image"></a>Instalación de Office en la imagen de un disco duro virtual principal

En este artículo se indica cómo instalar Aplicaciones de Microsoft 365 para empresas, OneDrive y otras aplicaciones habituales en una imagen de un disco duro virtual maestro para que se carguen en Azure. Si los usuarios necesitan acceder a determinadas aplicaciones de línea de negocio (LOB), se recomienda instalarlas después de completar las instrucciones de este artículo.

En este artículo se da por supuesto que ya ha creado una máquina virtual (VM). En caso contrario, consulte [Preparación y personalización de una imagen de disco duro virtual maestro](set-up-customize-master-image.md#create-a-vm).

En este artículo también se da por supuesto que ha concedido privilegios de acceso elevados sobre la máquina virtual sin importar si está aprovisionada en Azure o en Hyper-V Manager. Si no es así, consulte [Elevación de los privilegios de acceso para administrar todas las suscripciones y los grupos de administración de Azure](../role-based-access-control/elevate-access-global-admin.md).

>[!NOTE]
>Estas instrucciones son específicas de una configuración de Windows Virtual Desktop concreta que se puede usar con los procesos existentes de su organización.

## <a name="install-office-in-shared-computer-activation-mode"></a>Instalación de Office en modo de activación en equipos compartidos

La activación en equipos compartidos le permite implementar Aplicaciones de Microsoft 365 para empresas en un equipo de la organización al que acceden varios usuarios. Para más información sobre la activación en equipos compartidos, consulte la [introducción a la activación en equipos compartidos para Aplicaciones de Microsoft 365](/deployoffice/overview-shared-computer-activation).

Use la [herramienta de implementación de Office](https://www.microsoft.com/download/details.aspx?id=49117) para instalar Office. Sesión múltiple de Windows 10 Enterprise solo admite las siguientes versiones de Office:

   - Aplicaciones de Microsoft 365 para empresas
   - Aplicaciones de Microsoft 365 para empresas que incluyen una suscripción a Microsoft 365 Empresa Premium

La herramienta de implementación de Office requiere un archivo XML de configuración. Para personalizar el ejemplo siguiente, consulte [las opciones de configuración de la herramienta de implementación de Office](/deployoffice/configuration-options-for-the-office-2016-deployment-tool/).

Este archivo XML de configuración de ejemplo que hemos proporcionado hará lo siguiente:

   - Instalar Office desde el Canal de empresa mensual y entregar actualizaciones desde este mismo canal.
   - Usar la arquitectura x64.
   - Deshabilitar las actualizaciones automáticas.
   - Eliminar las instalaciones existentes de Office y migrar su configuración.
   - Habilitar la activación en equipos compartidos.

>[!NOTE]
>Es posible que la característica de búsqueda de la galería de símbolos de Visio no funcione como se esperaba en Windows Virtual Desktop.

Esto es lo que el archivo XML de configuración de ejemplo no hará:

   - Instalar Skype Empresarial
   - Instalar OneDrive en modo por usuario. Para más información, consulte [Instalación de OneDrive en modo por máquina](#install-onedrive-in-per-machine-mode).

>[!NOTE]
>La activación en equipos compartidos se puede configurar mediante objetos de directiva de grupo (GPO) o valores de registro. El GPO se encuentra en **Configuración del equipo\\Directivas\\Plantillas administrativas\\Microsoft Office 2016 (equipo)\\Configuración de licencias**

La herramienta de implementación de Office contiene setup.exe. Para instalar Office, ejecute el comando siguiente en una línea de comandos:

```batch
Setup.exe /configure configuration.xml
```

#### <a name="sample-configurationxml"></a>Archivo configuration.xml de ejemplo

En el siguiente ejemplo XML se instalará la versión de Canal de empresa mensual.

```xml
<Configuration>
  <Add OfficeClientEdition="64" Channel="MonthlyEnterprise">
    <Product ID="O365ProPlusRetail">
      <Language ID="en-US" />
      <Language ID="MatchOS" />
      <ExcludeApp ID="Groove" />
      <ExcludeApp ID="Lync" />
      <ExcludeApp ID="OneDrive" />
      <ExcludeApp ID="Teams" />
    </Product>
  </Add>
  <RemoveMSI/>
  <Updates Enabled="FALSE"/>
  <Display Level="None" AcceptEULA="TRUE" />
  <Logging Level=" Standard" Path="%temp%\WVDOfficeInstall" />
  <Property Name="FORCEAPPSHUTDOWN" Value="TRUE"/>
  <Property Name="SharedComputerLicensing" Value="1"/>
</Configuration>
```

>[!NOTE]
>El equipo de Office recomienda el uso de la instalación de 64 bits en el parámetro **OfficeClientEdition**.

Después de instalar Office, puede actualizar el comportamiento predeterminado de Office. Ejecute los comandos siguientes individualmente o en un archivo por lotes para actualizar el comportamiento.

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

## <a name="install-onedrive-in-per-machine-mode"></a>Instalación de OneDrive en modo por máquina

OneDrive se instala normalmente en modo por usuario. En este entorno, se debe instalar por máquina.

Aquí se indica cómo instalar OneDrive en modo por máquina:

1. En primer lugar, cree una ubicación para almacenar provisionalmente el instalador de OneDrive. Una carpeta del disco local o ubicación [\\\\unc] (file://unc) es adecuada.

2. Descargue el archivo OneDriveSetup.exe en la ubicación de almacenamiento provisional con este vínculo: <https://aka.ms/OneDriveWVD-Installer>

3. Si ha instalado Office con OneDrive omitiendo **\<ExcludeApp ID="OneDrive" /\>** , desinstale todas las instalaciones por usuario de OneDrive mediante un símbolo del sistema con privilegios elevados ejecutando el comando siguiente:

    ```batch
    "[staged location]\OneDriveSetup.exe" /uninstall
    ```

4. Ejecute este comando desde un símbolo del sistema con privilegios elevados para establecer el valor de registro **AllUsersInstall**:

    ```batch
    REG ADD "HKLM\Software\Microsoft\OneDrive" /v "AllUsersInstall" /t REG_DWORD /d 1 /reg:64
    ```

5. Ejecute este comando para instalar OneDrive en modo por máquina:

    ```batch
    Run "[staged location]\OneDriveSetup.exe" /allusers
    ```

6. Ejecute este comando para configurar OneDrive para que se inicie en el inicio de sesión de todos los usuarios:

    ```batch
    REG ADD "HKLM\Software\Microsoft\Windows\CurrentVersion\Run" /v OneDrive /t REG_SZ /d "C:\Program Files (x86)\Microsoft OneDrive\OneDrive.exe /background" /f
    ```

7. Habilite **Silently configure user account** (Configuración silenciosa de la cuenta de usuario) ejecutando el comando siguiente.

    ```batch
    REG ADD "HKLM\SOFTWARE\Policies\Microsoft\OneDrive" /v "SilentAccountConfig" /t REG_DWORD /d 1 /f
    ```

8. Redirija y mueva las carpetas conocidas de Windows a OneDrive mediante la ejecución del comando siguiente.

    ```batch
    REG ADD "HKLM\SOFTWARE\Policies\Microsoft\OneDrive" /v "KFMSilentOptIn" /t REG_SZ /d "<your-AzureAdTenantId>" /f
    ```

## <a name="microsoft-teams-and-skype-for-business"></a>Microsoft Teams y Skype Empresarial

Windows Virtual Desktop no admite Skype Empresarial ni Teams.

Para obtener ayuda con la instalación de Microsoft Teams, consulte [Uso de Microsoft Teams en Windows Virtual Desktop](teams-on-wvd.md). La optimización multimedia para Microsoft Teams en Windows Virtual Desktop está disponible en versión preliminar.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha agregado Office a la imagen, puede continuar personalizando la imagen del disco duro virtual maestro. Consulte [Preparación y personalización de una imagen de disco duro virtual maestro](set-up-customize-master-image.md).
