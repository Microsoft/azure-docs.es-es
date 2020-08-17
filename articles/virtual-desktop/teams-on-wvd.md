---
title: 'Microsoft Teams en Windows Virtual Desktop: Azure'
description: Procedimiento para usar Microsoft Teams en Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 07/28/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 6fd20819d17861ed5171bf61e4c485fcceba7985
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88006118"
---
# <a name="use-microsoft-teams-on-windows-virtual-desktop"></a>Uso de Microsoft Teams en Windows Virtual Desktop

>[!IMPORTANT]
>La optimización multimedia para Teams no es compatible con entornos de Microsoft 365 Government.

>[!NOTE]
>La optimización multimedia de Microsoft Teams solo está disponible para el cliente de escritorio de Windows en máquinas con Windows 10. Las optimizaciones multimedia requieren la versión de cliente de escritorio de Windows 1.2.1026.0 o posterior.

Microsoft Teams en Windows Virtual Desktop admite chat y colaboración. Con las optimizaciones multimedia, también se admite la función de llamada y reunión. Para obtener más información sobre cómo usar Microsoft Teams en entornos de infraestructura de escritorio virtual (VDI), consulte [Teams para la infraestructura de escritorio virtualizada](/microsoftteams/teams-for-vdi/).

Con la optimización multimedia de Microsoft Teams, el cliente de escritorio de Windows controla a nivel local el audio y el vídeo de las llamadas y reuniones de Teams. Puede seguir usando Microsoft Teams en Windows Virtual Desktop con otros clientes sin la optimización de llamadas y reuniones. Las características de chat y colaboración de Teams se admiten en todas las plataformas. Para redirigir dispositivos locales de la sesión remota, consulte [Personalización de las propiedades de Protocolo de escritorio remoto para un grupo de hosts](#customize-remote-desktop-protocol-properties-for-a-host-pool).

## <a name="prerequisites"></a>Requisitos previos

Para poder usar Microsoft Teams en Windows Virtual Desktop, tendrá que hacer lo siguiente:

- [Prepare la red](/microsoftteams/prepare-network/) para Microsoft Teams.
- Instale el [cliente de escritorio de Windows](connect-windows-7-10.md) en un dispositivo con Windows 10 o Windows 10 IoT Enterprise que cumpla los [requisitos de hardware de Teams de un equipo Windows](/microsoftteams/hardware-requirements-for-the-teams-app#hardware-requirements-for-teams-on-a-windows-pc/).
- Conéctese a una máquina virtual (VM) de sesión múltiple de Windows 10 o Windows 10 Enterprise.
- Instale la aplicación de escritorio Teams en el host mediante la instalación por máquina. Para la optimización multimedia de Microsoft Teams se necesita la versión 1.3.00.4461 o posterior de la aplicación de escritorio Teams.

## <a name="install-the-teams-desktop-app"></a>Instalación de la aplicación de escritorio Teams

En esta sección se muestra cómo instalar la aplicación de escritorio Teams en la imagen de la máquina virtual de Windows 10 o Windows 10 Enterprise. Para obtener más información, consulte [Instalación o actualización de la aplicación de escritorio Teams en VDI](/microsoftteams/teams-for-vdi#install-or-update-the-teams-desktop-app-on-vdi/).

### <a name="prepare-your-image-for-teams"></a>Preparación de la imagen para Teams

Para habilitar la optimización multimedia para Teams, establezca la siguiente clave del Registro en el host:

1. Desde el menú de Inicio, ejecute **RegEdit** como administrador. Vaya a **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Teams**.
2. Cree el siguiente valor para la clave de Teams:

| Nombre             | Tipo   | Datos/valor  |
|------------------|--------|-------------|
| IsWVDEnvironment | DWORD  | 1           |

### <a name="install-the-teams-websocket-service"></a>Instalación del servicio WebSocket de Teams

Instale el [servicio WebSocket](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE4AQBt) más reciente en la imagen de máquina virtual. Si se produce un error de instalación, instale la [última versión de Microsoft Visual C++ Redistributable](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads) e inténtelo de nuevo.

#### <a name="latest-websocket-service-versions"></a>Versiones más recientes del servicio WebSocket

En la tabla siguiente se enumeran las versiones más recientes del servicio WebSocket:

|Versión        |Fecha de la versión  |
|---------------|--------------|
|1.0.2006.11001 |28/07/2020    |
|0.11.0         |29/05/2020    |

#### <a name="updates-for-version-10200611001"></a>Actualizaciones de la versión 1.0.2006.11001

- Se ha corregido un problema que provocaba la caída del vídeo de entrada al minimizar la aplicación de Teams durante una llamada o reunión.
- Se ha agregado compatibilidad para seleccionar un monitor para compartir en sesiones de escritorio de varios monitores.

### <a name="install-microsoft-teams"></a>Instalación de Microsoft Teams

Puede implementar la aplicación de escritorio Teams mediante una instalación por máquina o por usuario. Para instalar Microsoft Teams en el entorno de Windows Virtual Desktop:

1. Descargue el [paquete MSI de Teams](/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm/) apropiado para el entorno. En un sistema operativo de 64 bits se recomienda usar el instalador de 64 bits.

      > [!NOTE]
      > Para la optimización multimedia de Microsoft Teams se necesita la versión 1.3.00.4461 o posterior de la aplicación de escritorio Teams.

2. Ejecute uno de los siguientes comandos para instalar el MSI en la máquina virtual del host:

    - Instalación por usuario

        ```powershell
        msiexec /i <path_to_msi> /l*v <install_logfile_name> ALLUSERS=1
        ```

        Este proceso es la instalación predeterminada, que instala Teams en la carpeta de usuario **%AppData%** . Los equipos no funcionarán correctamente con la instalación por usuario en una instalación no persistente.

    - Instalación por máquina

        ```powershell
        msiexec /i <path_to_msi> /l*v <install_logfile_name> ALLUSER=1 ALLUSERS=1
        ```

        Teams se instala en la carpeta Program Files (x86) de un sistema operativo de 64 bits y en la carpeta Archivos de programa de un sistema operativo de 32 bits. En este momento se ha completado la instalación de la imagen dorada. En el caso de instalaciones no persistentes, es necesario instalar Teams por cada máquina.

        La próxima vez que abra Teams en una sesión, se le pedirán sus credenciales.

        > [!NOTE]
        > En la actualidad, los usuarios y administradores no pueden deshabilitar el inicio automático para Teams durante el inicio de sesión.

3. Para desinstalar el archivo MSI de la máquina virtual del host, ejecute este comando:

      ```powershell
      msiexec /passive /x <msi_name> /l*v <uninstall_logfile_name>
      ```

      Teams se desinstala de la carpeta Program Files (x86) o de la carpeta Archivos de programa, según el entorno del sistema operativo.

      > [!NOTE]
      > Si instala Teams con el valor de MSI ALLUSER=1, se deshabilitarán las actualizaciones automáticas. La recomendación es que se asegure de actualizar Teams al menos una vez al mes. Para más información sobre la implementación de la aplicación de escritorio Teams, consulte [Implementación de la aplicación de escritorio de Teams en la máquina virtual](/microsoftteams/teams-for-vdi#deploy-the-teams-desktop-app-to-the-vm/)de la máquina virtual.

### <a name="verify-media-optimizations-loaded"></a>Comprobación de las optimizaciones multimedia cargadas

Después de instalar el servicio WebSocket y la aplicación de escritorio Teams, siga estos pasos para comprobar que se han cargado las optimizaciones multimedia de dicha aplicación:

1. Seleccione su imagen de perfil de usuario y, luego, seleccione **Acerca de**.
2. Seleccione **Versión**.

      Si están cargadas las optimizaciones multimedia, el mensaje emergente mostrará **WVD Media optimized** (Multimedia de WVD optimizado). Si el mensaje emergente muestra **WVD Media not connected** (Multimedia de WVD no conectado), salga de la aplicación Teams e inténtelo de nuevo.

3. Seleccione su imagen de perfil de usuario y, luego, elija **Configuración**.

      Si las optimizaciones multimedia están cargadas, los dispositivos de audio y las cámaras disponibles localmente se enumerarán en el menú de dispositivos. Si el menú muestra **Remote audio** (Audio remoto), salga de la aplicación Teams e inténtelo de nuevo. Si los dispositivos siguen sin aparecer en el menú, compruebe la configuración de privacidad en el PC local. Asegúrese de que en **Configuración** > **Privacidad** > **Permisos de aplicación** la opción de configuración **Permitir que las aplicaciones tengan acceso al micrófono** está establecida en **Activada**. Desconéctese de la sesión remota, vuelva a conectarse y compruebe de nuevo los dispositivos de audio y vídeo. Para unirse a llamadas y reuniones con vídeo, también debe conceder permiso para que las aplicaciones tengan acceso a la cámara.

## <a name="known-issues-and-limitations"></a>Limitaciones y problemas conocidos

El uso de Teams en un entorno virtualizado es diferente de su uso en un entorno no virtualizado. Para obtener más información sobre las limitaciones de Teams en entornos virtualizados, consulte [Teams para la infraestructura de escritorio virtualizada](/microsoftteams/teams-for-vdi#known-issues-and-limitations/).

### <a name="client-deployment-installation-and-setup"></a>Implementación, instalación y configuración del cliente

- Con la instalación por máquina, Teams en VDI no se actualiza automáticamente de la misma forma en que lo hacen los clientes de Teams no VDI. Para actualizar el cliente, debe actualizar la imagen de la máquina virtual mediante la instalación de una nueva MSI.
- Teams solo muestra actualmente la zona horaria UTC en el chat, los canales y el calendario.
- La optimización multimedia para Teams solo se admite para el cliente de escritorio de Windows en máquinas que ejecutan Windows 10.
- No se admite el uso de proxies HTTP explícitos definidos en un punto de conexión.

### <a name="calls-and-meetings"></a>Llamadas y reuniones

- El cliente de escritorio de Teams del entorno de Windows Virtual Desktop no admite eventos en directo. Por ahora, se recomienda combinar eventos en directo desde el [cliente web de Teams](https://teams.microsoft.com) en su sesión remota.
- Las llamadas o las reuniones no admiten actualmente el uso compartido de aplicaciones. Las sesiones de escritorio admiten el uso compartido de escritorio.
- No se admiten actualmente las acciones de dar y tomar el control.
- Teams en Windows Virtual Desktop solo admite una entrada de vídeo entrante cada vez. Esto significa que cada vez que alguien intente compartir su pantalla, la pantalla de esa persona aparecerá en lugar de la pantalla del coordinador de la reunión.
- Debido a las limitaciones de WebRTC, la resolución de la secuencia de vídeo entrante y saliente está limitada a 720p.
- La aplicación Teams no admite botones HID o controles LED con otros dispositivos.

En el caso de problemas conocidos de Teams que no están relacionados con los entornos virtualizados, consulte [Soporte para Microsoft Teams en la organización](/microsoftteams/known-issues/).

## <a name="uservoice-site"></a>Sitio de UserVoice

Envíe comentarios a Microsoft Teams sobre Windows Virtual Desktop en el [sitio de UserVoice](https://microsoftteams.uservoice.com/) de Teams.

## <a name="collect-teams-logs"></a>Recopilación de registros de Teams

Si tiene problemas con la aplicación de escritorio Teams en el entorno de Windows Virtual Desktop, recopile los registros de cliente en **%appdata%\Microsoft\Teams\logs.txt** en la máquina virtual de host.

Si tiene problemas con las llamadas y las reuniones, recopile registros de cliente web de Teams con la combinación de teclas **Ctrl** + **Alt** + **Mayús** + **1**. Los registros se escribirán en **%userprofile%\Downloads\MSTeams Diagnostics Log DATE_TIME.txt** en la máquina virtual de host.

## <a name="contact-microsoft-teams-support"></a>Contacto con el soporte técnico de Microsoft Teams

Para ponerse en contacto con el soporte técnico de Microsoft Teams, vaya al [centro de administración de Microsoft 365](/microsoft-365/admin/contact-support-for-business-products).

## <a name="customize-remote-desktop-protocol-properties-for-a-host-pool"></a>Personalización de las propiedades de Protocolo de escritorio remoto para un grupo de hosts

La personalización de las propiedades del Protocolo de Escritorio remoto (RDP) de un grupo de hosts, como el uso de varios monitores, la habilitación del micrófono y el redireccionamiento de audio, permite ofrecer una experiencia óptima a los usuarios en función de sus necesidades.

No es necesario habilitar el redireccionamiento de dispositivos cuando se usa Teams con optimización multimedia. Si usa Teams sin ella, establezca las siguientes propiedades de RDP para permitir el redireccionamiento del micrófono y de la cámara:

- `audiocapturemode:i:1` permite la captura de audio desde el dispositivo local y redirige las aplicaciones de audio en la sesión remota.
- `audiomode:i:0` reproduce audio en el equipo local.
- `camerastoredirect:s:*` redirige todas las cámaras.

Para obtener más información, consulte [Personalización de las propiedades de Protocolo de escritorio remoto para un grupo de hosts](customize-rdp-properties.md).
