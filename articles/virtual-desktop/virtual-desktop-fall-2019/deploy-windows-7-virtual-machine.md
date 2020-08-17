---
title: Implementación de una máquina virtual con Windows 7 en Windows Virtual Desktop (clásico) - Azure
description: Configuración e implementación de una máquina virtual con Windows 7 en Windows Virtual Desktop (clásico)
author: Heidilohr
ms.topic: how-to
ms.date: 03/30/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 259e49fbdd6a0eb392ddf6a3cd3c318798cfabd0
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005059"
---
# <a name="deploy-a-windows-7-virtual-machine-on-windows-virtual-desktop-classic"></a>Implementación de una máquina virtual con Windows 7 en Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager. Si está tratando de administrar objetos de Windows Virtual Desktop para Azure Resource Manager incorporados en la versión actual de Windows Virtual Desktop, vea [este artículo](../deploy-windows-7-virtual-machine.md).

El proceso de implementación de una máquina virtual con Windows 7 en Windows Virtual Desktop es ligeramente diferente al de las máquinas virtuales que ejecutan versiones posteriores de Windows. En esta guía se explica cómo implementar Windows 7.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, siga las instrucciones de [Creación de un grupo hosts con PowerShell](create-host-pools-powershell-2019.md) para crear un grupo de hosts. Después, siga las instrucciones de [Creación de grupos de hosts en Azure Marketplace](create-host-pools-azure-marketplace-2019.md#optional-assign-additional-users-to-the-desktop-application-group) para asignar uno o varios usuarios al grupo de aplicaciones de escritorio.

## <a name="configure-a-windows-7-virtual-machine"></a>Configuración de una máquina virtual con Windows 7

Una vez que haya realizado los requisitos previos, estará listo para configurar la máquina virtual de Windows 7 para su implementación en Windows Virtual Desktop.

Para configurar una máquina virtual de Windows 7 en Windows Virtual Desktop:

1. Inicie sesión en el Azure Portal y busque la imagen de Windows 7 Enterprise o cargue su propia imagen personalizada de Windows 7 Enterprise (x64).
2. Implemente una o varias máquinas virtuales con Windows 7 Enterprise como sistema operativo host. Asegúrese de que las máquinas virtuales permiten el Protocolo de escritorio remoto (RDP) (el puerto TCP/3389).
3. Conéctese al host de Windows 7 Enterprise mediante el RDP y autentíquese con las credenciales que definió al configurar la implementación.
4. Agregue la cuenta que usó al conectarse al host con RDP al grupo "Usuario de Escritorio remoto". Si no lo hace, es posible que no pueda conectarse a la máquina virtual después de conectarla a su dominio de Active Directory.
5. Vaya a Windows Update en la máquina virtual.
6. Instale todas las actualizaciones de Windows en la categoría Importante.
7. Instale todas las actualizaciones de Windows en la categoría Opcional (excepto los paquetes de idioma). Esto instala la actualización del Protocolo de escritorio remoto 8.0 ([KB2592687](https://www.microsoft.com/download/details.aspx?id=35387)) que necesita para completar estas instrucciones.
8. Abra el Editor de directivas de grupo local y vaya a **Configuración del equipo** > **Plantillas administrativas** > **Componentes de Windows** > **Servicios de escritorio remoto** > **Host de sesión de escritorio remoto** > **Entorno de sesión remota**.
9. Habilite la directiva del Protocolo de escritorio remoto 8.0.
10. Una esta máquina virtual al dominio de Active Directory.
11. Reinicie la máquina virtual, para lo que debe ejecutar el siguiente comando:

     ```cmd
     shutdown /r /t 0
     ```

12. Siga las instrucciones descritas [aquí](/powershell/module/windowsvirtualdesktop/export-rdsregistrationinfo/) para obtener un token de registro.
13. [Descargue el agente de Windows Virtual Desktop para Windows 7](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE3JZCm).
14. [Descargue el administrador de agente de Windows Virtual Desktop para Windows 7](https://query.prod.cms.rt.microsoft.com/cms/api/am/binary/RE3K2e3).
15. Abra el instalador del agente de Windows Virtual Desktop y siga las instrucciones. Cuando se le solicite, proporcione la clave de registro que ha creado en el paso 12.
16. Abra Windows Virtual Desktop Agent Manager y siga las instrucciones.
17. Opcionalmente, bloquee el puerto TCP/3389 para quitar el acceso directo del protocolo de escritorio remoto a la máquina virtual.
18. También puede confirmar que .NET Framework se encuentra al menos en la versión 4.7.2. Esto es especialmente importante si va a crear una imagen personalizada.

## <a name="next-steps"></a>Pasos siguientes

La implementación de Windows Virtual Desktop ya está lista para usarse. [Descargue la versión más reciente del cliente de Windows Virtual Desktop](https://aka.ms/wvd/clients/windows) para comenzar.

Para obtener una lista de los problemas conocidos, así como instrucciones para la solución de problemas de Windows 7 en Windows Virtual Desktop, consulte nuestro artículo para la solución de problemas en [Solución de los problemas de las máquinas virtuales con Windows 7 en Windows Virtual Desktop](troubleshoot-windows-7-vm.md).
