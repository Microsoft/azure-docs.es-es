---
title: Configuración de un servidor de procesos de escalabilidad horizontal durante la recuperación ante desastres de VM de VMware y servidores físicos con Azure Site Recovery | Microsoft Docs
description: En este artículo se describe cómo configurar un servidor de procesos de escalabilidad horizontal durante la recuperación ante desastres de VM de VMware y servidores físicos.
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 4/23/2019
ms.author: ramamill
ms.openlocfilehash: 40f912122e6ffb9cccbd32a747f6f0d46fd6c330
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87292807"
---
# <a name="scale-with-additional-process-servers"></a>Escalado de servidores de procesos adicionales

De forma predeterminada, al replicar servidores físicos o máquinas virtuales de VMware en Azure con [Site Recovery](site-recovery-overview.md), un servidor de procesos se instala en la máquina del servidor de configuración y se usa para coordinar la transferencia de datos entre Site Recovery y la infraestructura local. Para aumentar la capacidad y el escalado horizontal de la implementación de replicación, puede agregar servidores de procesos independientes adicionales. En este artículo se describe cómo configurar un servidor de procesos de escalabilidad horizontal.

## <a name="before-you-start"></a>Antes de comenzar

### <a name="capacity-planning"></a>Planificación de capacidad

Asegúrese de que ha realizado la [planeación de la capacidad](site-recovery-plan-capacity-vmware.md) para la replicación de VMware. Esto le ayudará a identificar cómo y cuándo debe implementar servidores de proceso adicionales.

Desde la versión 9.24, se agrega una guía durante la selección del servidor de procesos para nuevas replicaciones. El servidor de procesos se marcará como Correcto, Advertencia y Crítico según determinados criterios. Para comprender los distintos escenarios que pueden influir en el estado del servidor de procesos, revise las [alertas de servidor de procesos](vmware-physical-azure-monitor-process-server.md#process-server-alerts).

> [!NOTE]
> No se admite el uso de componentes de Process Server clonados. Siga los pasos de este artículo para el escalado horizontal de Process Server.

### <a name="sizing-requirements"></a>Requisitos de tamaño 

Compruebe los requisitos de tamaño que se resumen en la tabla. En general, si debe escalar horizontalmente la implementación a más de 200 máquinas de origen o si la tasa de renovación diaria total supera los 2 TB, necesitará servidores de procesos adicionales para controlar el volumen del tráfico.

| **Servidores de procesos adicionales** | **Tamaño del disco de caché** | **Frecuencia de cambio de datos** | **Máquinas protegidas** |
| --- | --- | --- | --- |
|4 vCPU (2 sockets * 2 núcleos \@ 2,5 GHz), 8 GB de memoria |300 GB |250 GB o menos |Replicar 85 máquinas o menos. |
|8 vCPU (2 sockets * 4 núcleos \@ 2,5 GHz), 12 GB de memoria |600 GB |250 GB a 1 TB |Replicar entre 85 y 150 máquinas. |
|12 vCPU (2 sockets * 6 núcleos \@ 2,5 GHz), 24 GB de memoria |1 TB |1 TB a 2 TB |Replicar entre 150 y 225 máquinas. |

Donde cada máquina de origen protegida está configurada con 3 discos de 100 GB cada uno.

### <a name="prerequisites"></a>Prerrequisitos

En la tabla siguiente se resumen los requisitos previos para el servidor de procesos adicional.

[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-configuration-and-scaleout-process-server-requirements.md)]

## <a name="download-installation-file"></a>Descarga del archivo de instalación

Descargue el archivo de instalación del servidor de procesos de la manera siguiente:

1. Inicie sesión en Azure Portal y busque el almacén de Recovery Services.
2. Abra **Infraestructura de Site Recovery** > **VMware and Physical Machines** (Máquinas físicas y VMware)  > **Servidores de configuración** (en For VMware & Physical Machines [Para máquinas físicas y VMware]).
3. Seleccione el servidor de configuración para explorar en profundidad su página de detalles. Luego haga clic en **Servidor de procesos**.
4. En **Agregar servidor de procesos** >  **Elegir dónde quiere implementar el servidor de procesos**, seleccione **Implementar un servidor de procesos de escalado horizontal local**.

   ![Página Agregar servidores](./media/vmware-azure-set-up-process-server-scale/add-process-server.png)
1. Haga clic en **Download the Microsoft Azure Site Recovery Unified Setup** (Descargar la instalación unificada de Microsoft Azure Site Recovery). Se descarga la versión más reciente del archivo de instalación.

   > [!WARNING]
   > La versión de instalación del servidor de procesos debe ser igual o anterior a la versión del servidor de configuración que se esté ejecutando. Una forma sencilla de garantizar la compatibilidad de versiones consiste en usar el mismo instalador que usó recientemente para instalar o actualizar el servidor de configuración.

## <a name="install-from-the-ui"></a>Instalar desde la interfaz de usuario

Haga la instalación de la siguiente manera. Después de configurar el servidor, debe migrar las máquinas de origen para que lo utilicen.

[!INCLUDE [site-recovery-configuration-server-requirements](../../includes/site-recovery-add-process-server.md)]


## <a name="install-from-the-command-line"></a>Instalación desde la línea de comandos

Instálelos ejecutando el comando siguiente:

```
UnifiedSetup.exe [/ServerMode <CS/PS>] [/InstallDrive <DriveLetter>] [/MySQLCredsFilePath <MySQL credentials file path>] [/VaultCredsFilePath <Vault credentials file path>] [/EnvType <VMware/NonVMware>] [/PSIP <IP address to be used for data transfer] [/CSIP <IP address of CS to be registered with>] [/PassphraseFilePath <Passphrase file path>]
```

Los parámetros de la línea de comandos son los siguientes:

[!INCLUDE [site-recovery-unified-setup-parameters](../../includes/site-recovery-unified-installer-command-parameters.md)]

Por ejemplo:

```
MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:C:\Temp\Extracted
cd C:\Temp\Extracted
UNIFIEDSETUP.EXE /AcceptThirdpartyEULA /servermode "PS" /InstallLocation "D:\" /EnvType "VMware" /CSIP "10.150.24.119" /PassphraseFilePath "C:\Users\Administrator\Desktop\Passphrase.txt" /DataTransferSecurePort 443
```
### <a name="create-a-proxy-settings-file"></a>Creación de un archivo de configuración de proxy

Si tiene que configurar un servidor proxy, el parámetro ProxySettingsFilePath toma un archivo como entrada. Puede crear el archivo de la manera siguiente y pasarlo como parámetro de entrada ProxySettingsFilePath.

```
* [ProxySettings]
* ProxyAuthentication = "Yes/No"
* Proxy IP = "IP Address"
* ProxyPort = "Port"
* ProxyUserName="UserName"
* ProxyPassword="Password"
```

## <a name="next-steps"></a>Pasos siguientes
Más información sobre la [administración de configuración de servidores de procesos](vmware-azure-manage-process-server.md)
