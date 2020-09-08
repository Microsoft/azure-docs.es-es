---
title: Actualización del agente de Microsoft Azure Recovery Services (MARS)
description: Obtenga información para actualizar el agente de Microsoft Azure Recovery Services (MARS).
ms.topic: conceptual
ms.date: 03/03/2020
ms.openlocfilehash: a1ee26db962781643e9599069282647658301bac
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89181483"
---
# <a name="upgrade-the-microsoft-azure-recovery-services-mars-agent"></a>Actualización del agente de Microsoft Azure Recovery Services (MARS)

En este artículo, aprenderá a:

* Identificar servidores con versiones anteriores del agente de MARS
* Actualizar instalaciones de MARS en esos servidores

## <a name="identify-servers-with-earlier-versions-of-the-mars-agent"></a>Identificar servidores con versiones anteriores del agente de MARS

En el caso de instalaciones de agente y servidor de Azure Backup:

1. Vaya al almacén de Recovery Services en el que ha registrado los servidores cuya copia de seguridad posiblemente vaya a realizarse mediante versiones anteriores del agente. Puede encontrar una lista representativa de almacenes con agentes de Azure Backup más antiguos en las alertas de Azure Backup Update de Azure.
1. En la sección **Configuración** de la izquierda del almacén de Recovery Services, seleccione **Infraestructura de Backup** en la sección **Administrar**.
1. Para detectar los agentes de Azure Backup instalados como parte de las instalaciones de servidor de Azure Backup, vaya a **Servidores de administración de copias de seguridad** en **Servidores de administración**. Así aparece una lista de los servidores que tienen instalaciones de servidor de Azure Backup junto con el número de versión del agente de Azure Backup asociado.

    ![Lista de agentes de MARS instalados como parte de instalaciones de servidor de Azure Backup](./media/upgrade-mars-agent/backup-management-servers.png)

1. Para comprobar las versiones del agente de instalaciones de agente de Microsoft Azure Recovery Services (MARS) o el agente de Azure Backup, vaya a **Servidores protegidos** en **Servidores de administración**. Luego seleccione **Agente de copia de seguridad de Azure** en Tipo de administración de copias de seguridad. Así se enumeran los servidores que tienen instalaciones de agente de Azure Backup junto con el número de versión de la instalación.

    ![Lista de servidores con el agente de MARS instalado](./media/upgrade-mars-agent/protected-servers.png)

1. Ordene la columna de versión del agente de Azure Backup mediante la selección de la columna **Versión del agente** en las instalaciones de agentes de MARS o la columna **Versión del agente de Azure Backup** en las instalaciones de Azure Backup Server.

1. El paso anterior proporciona la lista de servidores con agentes de Azure Backup que tienen versiones inferiores a la 2.0.9083.0 o versiones de agente que se muestran como espacios en blanco. Estos son los servidores en los que es necesario actualizar los agentes de Azure Backup.

## <a name="update-the-mars-agent-installation-on-the-server"></a>Actualizar la instalación del agente de MARS en el servidor

Una vez identificados los servidores que necesitan una actualización del agente de Azure Backup, realice los pasos siguientes para cada servidor identificado (mediante el servidor de Azure Backup o el agente de MARS). [Descargue la versión más reciente del agente de Azure Backup](https://aka.ms/azurebackup_agent) antes de realizar los pasos siguientes.

1. Seleccione una fila que tenga un agente de Azure Backup con una versión inferior a la 2.0.9083.0 o en blanco. Con esto se abre la pantalla de detalles del servidor.

    ![Servidores protegidos con versiones de agente no actualizadas](./media/upgrade-mars-agent/old-agent-version.png)

    ![Servidores de Azure Backup con versiones del agente no actualizadas](./media/upgrade-mars-agent/backup-management-servers-old-versions.png)

1. Seleccione **Conectar** para recibir un archivo de conexión a Escritorio remoto a fin de conectarse al servidor o hacerlo directamente a través de la conexión a Escritorio remoto en el servidor.

    ![Conexión al servidor a través de la conexión a Escritorio remoto](./media/upgrade-mars-agent/connect-to-server.png)

    >[!NOTE]
    > Si el servidor de la lista no existe o se ha retirado, puede omitir los pasos restantes siguientes y pasar al siguiente servidor.

1. Escriba los detalles de inicio de sesión administrativo e inicie sesión.

1. Si el servidor o el proxy del servidor tiene acceso limitado a Internet, asegúrese de que la configuración del firewall en el servidor o el proxy está establecida para permitir la dirección URL adecuada para la nube de Azure que está usando:

    Nube de Azure | URL
    --- | ---
    Nube de Azure (pública) |   `https://login.windows.net`
    Nube de Azure China 21Vianet   | `https://login.chinacloudapi.cn`
    Nube de Azure US Government |   `https://login.microsoftonline.us`
    Nube de Azure German  |  `https://login.microsoftonline.de`

1. Copie el instalador de actualizaciones del agente de Azure Backup en el servidor.

    ![Copia del instalador de actualizaciones del agente de Azure Backup en el servidor](./media/upgrade-mars-agent/copy-agent-installer.png)

1. Ejecute al programa de instalación. Se abre el Asistente para la actualización del agente de Microsoft Azure Recovery Services.

    ![Asistente para la actualización del agente de Microsoft Azure Recovery Services](./media/upgrade-mars-agent/agent-upgrade-wizard.png)

1. Seleccione **Next** (Siguiente).

1. Seleccione **Actualizar**.

    ![Instalación del agente de Microsoft Azure Recovery Services](./media/upgrade-mars-agent/upgrade-installation.png)

1. La pantalla de confirmación final indica que el agente de Azure Backup se ha actualizado correctamente.

## <a name="for-system-center-data-protection-manager-sc-dpm-customers"></a>En clientes de System Center Data Protection Manager (SC DPM)

Si ha instalado agentes de Azure Backup en los servidores de System Center Data Protection Manager (SC DPM), debe realizar los pasos siguientes para identificar si los servidores de DPM necesitan una actualización del agente de Azure Backup:

1. Inicie sesión como administrador en el servidor de SC DPM.
2. Abra la consola de DPM.
3. Seleccione **Administración** en el panel de navegación inferior izquierdo de la consola.
4. En la información que aparece en el panel de navegación izquierdo, busque la información de versión del agente de Azure Backup.
5. Si la versión es inferior a la 2.0.9083.0, descargue el instalador del agente de Azure Backup más reciente y ejecútelo en el servidor de DPM para actualizar el agente de Azure Backup.

Repita los pasos anteriores para todos los servidores de DPM del entorno.

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre la [Copia de seguridad de máquinas Windows con el agente de MARS de Azure Backup](backup-windows-with-mars-agent.md)
