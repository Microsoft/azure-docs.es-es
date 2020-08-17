---
title: Preparación y personalización de una imagen de disco duro virtual maestro - Azure
description: Cómo preparar, personalizar y cargar una imagen maestra de Windows Virtual Desktop en Azure.
author: Heidilohr
ms.topic: how-to
ms.date: 10/14/2019
ms.author: helohr
manager: lizross
ms.openlocfilehash: 2a10a32a98a240f740f48f7b25e6fa6ac3f2e873
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009518"
---
# <a name="prepare-and-customize-a-master-vhd-image"></a>Preparación y personalización de una imagen de disco duro virtual maestro

En este artículo se explica cómo preparar una imagen de disco duro virtual (VHD) maestro para cargarla en Azure, así como el modo de crear máquinas virtuales e instalar software en ellos. Estas instrucciones son específicas de una configuración de Windows Virtual Desktop concreta que se puede usar con los procesos existentes de su organización.

## <a name="create-a-vm"></a>Crear una VM

La sesión múltiple de Windows 10 Enterprise está disponible en la galería de imágenes de Azure. Existen dos formas de personalizar esta imagen.

La primera consiste en aprovisionar una máquina virtual en Azure siguiendo las instrucciones de [Crear una máquina virtual a partir de una imagen administrada](../virtual-machines/windows/create-vm-generalized-managed.md) y, luego, ir directamente a [Preparación e instalación del software](set-up-customize-master-image.md#software-preparation-and-installation).

La segunda es crear la imagen localmente, para lo cual hay que descargar la imagen, aprovisionar una máquina virtual de Hyper-V y personalizarla de forma que cubra sus necesidades, lo que trataremos en la sección siguiente.

### <a name="local-image-creation"></a>Creación de imágenes locales

Cuando haya descargado la imagen en una ubicación local, abra el **Administrador de Hyper-V** para crear una máquina virtual con el disco duro virtual que ha copiado. Las siguientes instrucciones son una versión simple, pero puede encontrar instrucciones más detalladas en [Crear una máquina virtual en Hyper-V](/windows-server/virtualization/hyper-v/get-started/create-a-virtual-machine-in-hyper-v/).

Para crear una máquina virtual mediante el VHD copiado:

1. Abra el **Asistente para crear nueva máquina virtual**.

2. En la página Especificar generación, seleccione **Generación 1**.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página Especificar generación. La opción "Generación 1" está seleccionada.](media/a41174fd41302a181e46385e1e701975.png)

3. En Tipo de punto de control, deshabilite puntos de control desactivando las casillas correspondientes.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la sección Tipo de punto de control de la página Puntos de control](media/20c6dda51d7cafef33251188ae1c0c6a.png)

También puede ejecutar el siguiente cmdlet de PowerShell para deshabilitar puntos de control.

```powershell
Set-VM -Name <VMNAME> -CheckpointType Disabled
```

### <a name="fixed-disk"></a>Disco fijo

Si crea una máquina virtual desde un disco duro virtual existente, se crea un disco dinámico de forma predeterminada. Este disco se puede cambiar a disco fijo si se selecciona **Editar disco...** , como se muestra en la siguiente imagen. Para obtener más información, vea [Preparación de un VHD o un VHDX de Windows para cargar en Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md).

> [!div class="mx-imgBorder"]
> ![Captura de pantalla de la opción Editar disco.](media/35772414b5a0f81f06f54065561d1414.png)

También puede ejecutar el siguiente cmdlet de PowerShell para cambiar el disco a disco fijo.

```powershell
Convert-VHD –Path c:\test\MY-VM.vhdx –DestinationPath c:\test\MY-NEW-VM.vhd -VHDType Fixed
```

## <a name="software-preparation-and-installation"></a>Preparación e instalación del software

En esta sección se describe la preparación y la instalación de FSLogix y Windows Defender, además de algunas opciones de configuración básicas de las aplicaciones y del registro de la imagen.

Si va a instalar Aplicaciones de Microsoft 365 para negocios y OneDrive en la máquina virtual, vaya a [Instalación de Office en la imagen de un disco duro virtual principal](install-office-on-wvd-master-image.md) y siga las instrucciones que se indican para instalar las aplicaciones. Cuando haya terminado, vuelva a este artículo.

Si los usuarios necesitan tener acceso a determinadas aplicaciones de línea de negocio, se recomienda instalarlas después de completar las instrucciones de esta sección.

### <a name="set-up-user-profile-container-fslogix"></a>Configurar un contenedor de perfiles de usuario (FSLogix)

Para incluir el contenedor de FSLogix como parte de la imagen, siga las instrucciones en [Creación de un contenedor de perfiles para un grupo host mediante un recurso compartido de archivos](create-host-pools-user-profile.md#configure-the-fslogix-profile-container). Puede probar la funcionalidad del contenedor de FSLogix con [este inicio rápido](/fslogix/configure-cloud-cache-tutorial/).

### <a name="configure-windows-defender"></a>Configurar Windows Defender

Si Windows Defender está configurado en la máquina virtual, asegúrese de que no analiza todo el contenido de los archivos VHD y VHDX durante la asociación.

Si se configura así, solamente se quita el análisis de archivos VHD y VHDX durante la asociación; esto no afectará al análisis en tiempo real.

Para obtener más instrucciones sobre cómo configurar Windows Defender en Windows Server, vea [Configurar exclusiones del Antivirus de Windows Defender en Windows Server](/windows/security/threat-protection/windows-defender-antivirus/configure-server-exclusions-windows-defender-antivirus/).

Para obtener más información sobre cómo configurar Windows Defender para que excluya determinados archivos del análisis, vea [Configurar y validar exclusiones en función de la extensión de archivo y la ubicación de carpeta](/windows/security/threat-protection/windows-defender-antivirus/configure-extension-file-exclusions-windows-defender-antivirus/).

### <a name="disable-automatic-updates"></a>Deshabilitar las actualizaciones automáticas

Para deshabilitar las actualizaciones automáticas mediante la directiva de grupo local:

1. Abra **Editor de directivas de grupo local\\Plantillas administrativas\\Componentes de Windows\\Windows Update**.
2. Haga clic con el botón derecho en **Configurar Actualizaciones automáticas** y establézcalo en **Deshabilitado**.

También puede ejecutar el siguiente comando en un símbolo del sistema para deshabilitar las actualizaciones automáticas.

```batch
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU" /v NoAutoUpdate /t REG_DWORD /d 1 /f
```

### <a name="specify-start-layout-for-windows-10-pcs-optional"></a>Especificar un diseño de la pantalla Inicio en equipos Windows 10 (opcional)

Ejecute este comando para especificar un diseño de la pantalla Inicio en equipos Windows 10.

```batch
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer" /v SpecialRoamingOverrideAllowed /t REG_DWORD /d 1 /f
```

### <a name="set-up-time-zone-redirection"></a>Configurar la redirección de zona horaria

Se puede aplicar la redirección de zona horaria en el nivel de la directiva de grupo, ya que todas las máquinas virtuales de un grupo de hosts forman parte del mismo grupo de seguridad.

Para redirigir zonas horarias:

1. En el servidor de Active Directory, abra la **Consola de administración de directivas de grupo**.
2. Expanda su dominio y Objetos de directiva de grupo.
3. Haga clic con el botón derecho en el **objeto de directiva de grupo** que creó para la configuración de directiva de grupo y seleccione **Editar**.
4. En el **Editor de administración de directivas de grupo**, vaya a **Configuración del equipo** > **Directivas** > **Plantillas administrativas** > **Componentes de Windows** > **Servicios de Escritorio remoto** > **Host de sesión de Escritorio remoto**  > **Redirección de dispositivo o recurso**.
5. Habilite la configuración **Permitir redirección de zona horaria**.

También puede ejecutar este comando en la imagen maestra para redirigir zonas horarias:

```batch
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" /v fEnableTimeZoneRedirection /t REG_DWORD /d 1 /f
```

### <a name="disable-storage-sense"></a>Deshabilitar el sensor de almacenamiento

Se recomienda deshabilitar el sensor de almacenamiento en el host de sesión de Windows Virtual Desktop que usa Windows 10 Enterprise o la sesión múltiple de Windows 10 Enterprise. El sensor de almacenamiento se puede deshabilitar en la sección **Almacenamiento** del menú Configuración, como se muestra en la siguiente captura de pantalla:

> [!div class="mx-imgBorder"]
> ![Captura de pantalla del menú Almacenamiento dentro de Configuración. La opción "Sensor de almacenamiento" está desactivada.](media/storagesense.png)

También puede cambiar la configuración con el Registro; para ello, ejecute el siguiente comando:

```batch
reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy" /v 01 /t REG_DWORD /d 0 /f
```

### <a name="include-additional-language-support"></a>Incluir compatibilidad con más idiomas

En este artículo no se describe cómo configurar la compatibilidad con regiones e idiomas. Para más información, consulte los siguientes artículos.

- [Agregar idiomas a imágenes de Windows](/windows-hardware/manufacture/desktop/add-language-packs-to-windows/)
- [Características a petición](/windows-hardware/manufacture/desktop/features-on-demand-v2--capabilities/)
- [Características de idioma y región a petición (FOD)](/windows-hardware/manufacture/desktop/features-on-demand-language-fod/)

### <a name="other-applications-and-registry-configuration"></a>Otras aplicaciones y configuración del Registro

En esta sección se aborda la configuración de aplicaciones y del sistema operativo. Toda la configuración descrita en esta sección se realiza mediante entradas del Registro que se pueden ejecutar mediante la línea de comandos y herramientas de regedit.

>[!NOTE]
>Puede implementar procedimientos recomendados de configuración con objetos de directiva de grupo (GPO) o con importaciones del Registro. El administrador puede elegir cualquiera de ambas opciones en función de los requisitos de su organización.

Ejecute el siguiente comando para recopilar del Centro de opiniones diferentes datos de telemetría relativos a la sesión múltiple de Windows 10 Enterprise:

```batch
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection" /v AllowTelemetry /t REG_DWORD /d 3 /f
```

Ejecute el siguiente comando para corregir bloqueos de Watson:

```batch
remove CorporateWerServer* from Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Windows Error Reporting
```

Escriba los siguientes comandos en el Editor del Registro para corregir la compatibilidad con la resolución 5k. Estos comandos se deben ejecutar antes de poder habilitar la pila en paralelo.

```batch
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MaxMonitors /t REG_DWORD /d 4 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MaxXResolution /t REG_DWORD /d 5120 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v MaxYResolution /t REG_DWORD /d 2880 /f

reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\rdp-sxs" /v MaxMonitors /t REG_DWORD /d 4 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\rdp-sxs" /v MaxXResolution /t REG_DWORD /d 5120 /f
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\rdp-sxs" /v MaxYResolution /t REG_DWORD /d 2880 /f
```

## <a name="prepare-the-image-for-upload-to-azure"></a>Preparar la imagen que se va a cargar en Azure

Cuando haya terminado de configurar e instalar todas las aplicaciones, siga las instrucciones de [Preparación de un VHD o un VHDX de Windows para cargar en Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md) para preparar la imagen.

Después de preparar la imagen para su carga, asegúrese de que la máquina virtual permanece en estado desactivado o desasignado.

## <a name="upload-master-image-to-a-storage-account-in-azure"></a>Cargar una imagen maestra en una cuenta de almacenamiento de Azure

Esta sección solo procede si la imagen maestra se ha creado localmente.

Las siguientes instrucciones le indicarán cómo cargar la imagen maestra en una cuenta de Azure Storage. Si aún no tiene una cuenta de Azure Storage, siga las instrucciones de [este artículo](/azure/developer/javascript/tutorial-vscode-static-website-node-03) para crearla.

1. Si aún no lo ha hecho, convierta la imagen de máquina virtual (VHD) en fija. Si no lo hace, no podrá crear la imagen correctamente.

2. Cargue la VHD en un contenedor de blobs en la cuenta de almacenamiento. Puede hacerlo rápidamente con la [herramienta Explorador de Storage](https://azure.microsoft.com/features/storage-explorer/). Para obtener más información sobre la herramienta Explorador de Storage, vea [este artículo](https://docs.microsoft.com/azure/vs-azure-tools-storage-manage-with-storage-explorer?tabs=windows).

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la ventana de búsqueda de la herramienta Explorador de Microsoft Azure Storage. La casilla "Cargar archivos .vhd/vhdx como blobs en páginas (recomendado)" está activada.](media/897aa9a9b6acc0aa775c31e7fd82df02.png)

3. Después, vaya a Azure Portal en el navegador y busque "Imágenes". La búsqueda debería llevarle a la página **Crear imagen**, como se muestra en la siguiente captura de pantalla:

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página Crear imagen de Azure Portal rellenada con valores de ejemplo de la imagen.](media/d3c840fe3e2430c8b9b1f44b27d2bf4f.png)

4. Una vez que haya creado la imagen, debería ver una notificación como la de la siguiente captura de pantalla:

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la notificación "Se creó correctamente la imagen"](media/1f41b7192824a2950718a2b7bb9e9d69.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que tiene una imagen, puede crear o actualizar los grupos de hosts. Para obtener más información sobre cómo crear y actualizar grupos de hosts, vea los artículos siguientes:

- [Creación de un grupo host con una plantilla de Azure Resource Manager](./virtual-desktop-fall-2019/create-host-pools-arm-template.md)
- [Tutorial: Creación de un grupo host con Azure Marketplace](create-host-pools-azure-marketplace.md)
- [Creación de un grupo host con PowerShell](create-host-pools-powershell.md)
- [Creación de un contenedor de perfiles para un grupo host mediante un recurso compartido de archivos](create-host-pools-user-profile.md)
- [Configuración del método de equilibrio de carga de Windows Virtual Desktop](configure-host-pool-load-balancing.md)
