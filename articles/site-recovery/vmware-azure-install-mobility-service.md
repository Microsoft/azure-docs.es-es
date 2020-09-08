---
title: Preparación de máquinas de origen para instalar Mobility Service a través de la instalación de inserción para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure | Microsoft Docs
description: Aprenda a preparar el servidor para instalar el agente de Mobility Service a través de la instalación de inserción para la recuperación ante desastres de máquinas virtuales de VMware y servidores físicos en Azure con el servicio Azure Site Recovery.
author: Rajeswari-Mamilla
ms.service: site-recovery
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: ramamill
ms.openlocfilehash: f75723aedae390a0d41956d63acadf6370f390d9
ms.sourcegitcommit: d661149f8db075800242bef070ea30f82448981e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88606513"
---
# <a name="prepare-source-machine-for-push-installation-of-mobility-agent"></a>Preparación de una máquina de origen para la instalación de inserción del agente de Mobility

Al configurar la recuperación ante desastres para máquinas virtuales VMware y servidores físicos con [Azure Site Recovery](site-recovery-overview.md), instala el servicio [Site Recovery Mobility](vmware-physical-mobility-service-overview.md) en cada máquina virtual VMware local y servidor físico.  Mobility Service captura las escrituras de datos en la máquina y las reenvía al servidor de procesos de Site Recovery.

## <a name="install-on-windows-machine"></a>Instalación en una máquina Windows

En cada máquina Windows que quiera proteger, realice lo siguiente:

1. Asegúrese de que haya conectividad de red entre la máquina y el servidor de procesos. Si no ha configurado un servidor de procesos independiente, se ejecuta de forma predeterminada en el servidor de configuración.
1. Cree una cuenta que el servidor de procesos pueda utilizar para acceder al equipo. La cuenta debe tener derechos de administrador (local o dominio). Use esta cuenta solo para la instalación de inserción y para las actualizaciones del agente.
2. Si no usa una cuenta de dominio, deshabilite el control de acceso de usuario remoto en el equipo local de la manera siguiente:
    - En la clave del Registro HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System, agregue un nuevo valor de DWORD: **LocalAccountTokenFilterPolicy**. Establezca el valor en **1**.
    -  Para hacerlo en un símbolo del sistema, ejecute el siguiente comando:  
   `REG ADD HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System /v LocalAccountTokenFilterPolicy /t REG_DWORD /d
3. En el Firewall de Windows de la máquina que quiere proteger, seleccione **Allow an app or feature through Firewall** (Permitir una aplicación o una característica a través de Firewall). Habilite **Compartir archivos e impresoras** e **Instrumental de administración de Windows (WMI)** . En el caso de los equipos que pertenecen a un dominio, el firewall se puede configurar mediante un objeto de directiva de grupo (GPO).

   ![Configuración de firewall](./media/vmware-azure-install-mobility-service/mobility1.png)

4. Agregue la cuenta que creó en CSPSConfigtool. Para ello, inicie sesión en el servidor de configuración.
5. Abra **cspsconfigtool.exe**. Esta herramienta está disponible como acceso directo en el escritorio y se encuentra en la carpeta %ProgramData%\ASR\home\svsystems\bin.
6. En la pestaña **Administrar cuentas**, seleccione en **Agregar cuenta**.
7. Agregue la cuenta que creó.
8. Especifique las credenciales que utiliza al habilitar la replicación en un equipo.

## <a name="install-on-linux-machine"></a>Instalación en una máquina Linux

En cada máquina Linux que quiere proteger, realice lo siguiente:

1. Asegúrese de que haya conectividad de red entre la máquina Linux y el servidor de procesos.
2. Cree una cuenta que el servidor de procesos pueda utilizar para acceder al equipo. La cuenta debe ser un usuario **raíz** en el servidor Linux de origen. Esta cuenta solo se usa para la instalación de inserción y para las actualizaciones.
3. Compruebe que el archivo /etc/hosts del servidor Linux de origen tiene entradas que asignan el nombre de host local a las direcciones IP asociadas con todos los adaptadores de red.
4. Instale los paquetes openssh, openssh-server y openssl más recientes en el equipo que desea replicar.
5. Asegúrese de que Secure Shell (SSH) está habilitado y se ejecuta en el puerto 22.
4. Habilitación de la autenticación de la contraseña y el subsistema SFTP en el archivo sshd_config. Para ello, inicie sesión como **raíz**.
5. En el archivo **/etc/ssh/sshd_config**, encuentre la línea que comienza por **PasswordAuthentication**.
6. Quite la marca de comentario de la línea y cambie el valor **yes**.
7. Busque la línea que comienza por **Subsystem** y quite la marca de comentario.

      ![Linux](./media/vmware-azure-install-mobility-service/mobility2.png)

8. Reinicie el servicio **sshd**.
9. Agregue la cuenta que creó en CSPSConfigtool. Para ello, inicie sesión en el servidor de configuración.
10. Abra **cspsconfigtool.exe**. Esta herramienta está disponible como acceso directo en el escritorio y se encuentra en la carpeta %ProgramData%\home\svsystems\bin.
11. En la pestaña **Administrar cuentas**, seleccione en **Agregar cuenta**.
12. Agregue la cuenta que creó.
13. Especifique las credenciales que utiliza al habilitar la replicación en un equipo.
1. Paso adicional para actualizar o proteger máquinas de SUSE Linux Enterprise Server 11 SP3. [Asegúrese de que la versión más reciente está disponible en el servidor de configuración](vmware-physical-mobility-service-overview.md#download-latest-mobility-agent-installer-for-suse-11-sp3-server).

## <a name="anti-virus-on-replicated-machines"></a>Antivirus en máquinas replicadas

Si las máquinas que desea replicar tiene software antivirus activo en ejecución, asegúrese de excluir la carpeta de instalación del servicio de movilidad de las operaciones antivirus (*C:\ProgramData\ASR\agent*). Esto garantiza que la replicación funciona según lo previsto.

## <a name="next-steps"></a>Pasos siguientes

Después de instalar Mobility Service, en Azure Portal, seleccione **+Replicar** para empezar a proteger estas máquinas virtuales. Obtenga más información sobre cómo habilitar la replicación para [máquinas virtuales de VMware](vmware-azure-enable-replication.md) y [servidores físicos](physical-azure-disaster-recovery.md#enable-replication).


