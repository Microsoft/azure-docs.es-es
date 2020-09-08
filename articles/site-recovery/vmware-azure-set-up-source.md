---
title: Configuración de opciones de origen para la recuperación ante desastres de VMware en Azure con Azure Site Recovery.
description: En este artículo se describe cómo configurar el entorno local para replicar máquinas virtuales de VMware en Azure con Azure Site Recovery.
services: site-recovery
author: Rajeswari-Mamilla
manager: rochakm
ms.service: site-recovery
ms.topic: article
ms.date: 04/14/2019
ms.author: ramamill
ms.openlocfilehash: afd3979690b8952c915a49099ee04b3d416031fd
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88189728"
---
# <a name="set-up-the-source-environment-for-vmware-to-azure-replication"></a>Configuración del entorno de origen para la replicación de VMware en Azure

En este artículo se describe cómo configurar el entorno local de origen para replicar máquinas virtuales de VMware en Azure. Este artículo incluye los pasos para seleccionar el escenario de replicación, configurar una máquina local como el servidor de configuración de Site Recovery y detectar automáticamente las máquinas virtuales locales.

## <a name="prerequisites"></a>Prerrequisitos

En este artículo se supone que ya ha seguido estos pasos:

- Ha planeado la implementación con la ayuda de [Azure Site Recovery Deployment Planner](site-recovery-deployment-planner.md). Esto le ayuda a asignar suficiente ancho de banda, en función de la frecuencia diaria de intercambio de datos, para cumplir el objetivo de punto de recuperación (RPO) deseado.
- [Configurar los recursos](tutorial-prepare-azure.md) en [Azure Portal](https://portal.azure.com).
- [Configurar VMware local](vmware-azure-tutorial-prepare-on-premises.md), incluida una cuenta dedicada para la detección automática.

## <a name="choose-your-protection-goals"></a>Selección de los objetivos de protección

1. En **Almacenes de Recovery Services**, seleccione el nombre del almacén. En este escenario se usa **ContosoVMVault**.
2. En **Introducción**, seleccione Site Recovery. Luego, seleccione **Preparar infraestructura**.
3. En **Objetivo de protección** >  **¿Dónde están ubicadas las máquinas?** , seleccione **Local**.
4. En **¿A dónde quiere replicar las máquinas?** , seleccione **En Azure**.
5. En **¿Las máquinas están virtualizadas?** , seleccione **Yes, with VMware vSphere Hypervisor** (Sí, con VMware vSphere Hypervisor). Después, seleccione **Aceptar**.

## <a name="set-up-the-configuration-server"></a>Configuración del servidor de configuración

Puede configurar el servidor de configuración como una máquina virtual de VMware local mediante una plantilla de Open Virtualization Application (OVA). [Más información](./vmware-azure-architecture.md) sobre los componentes que se instalarán en la máquina virtual VMware.

1. Obtenga más información sobre los [requisitos previos](vmware-azure-deploy-configuration-server.md#prerequisites) para la implementación del servidor de configuración.
2. [Compruebe los números que representan la capacidad](vmware-azure-deploy-configuration-server.md#sizing-and-capacity-requirements) para la implementación.
3. [Descargue](vmware-azure-deploy-configuration-server.md#download-the-template) e [importe](vmware-azure-deploy-configuration-server.md#import-the-template-in-vmware) la plantilla de OVA para configurar una máquina virtual de VMware local que se ejecute en el servidor de configuración. La licencia proporcionada con la plantilla es una licencia de evaluación y es válida durante 180 días. Tras este periodo, el cliente debe activar Windows con una licencia adquirida.
4. Encienda la máquina virtual VMware y [regístrela](vmware-azure-deploy-configuration-server.md#register-the-configuration-server-with-azure-site-recovery-services) en el almacén de Recovery Services.

## <a name="azure-site-recovery-folder-exclusions-from-antivirus-program"></a>Exclusiones de carpetas de Azure Site Recovery de un programa antivirus

### <a name="if-antivirus-software-is-active-on-source-machine"></a>Si hay algún antivirus activo en la máquina de origen

Si el equipo de origen contiene algún software antivirus, la carpeta de instalación debe excluirse. Por consiguiente, para que no surja ningún problema durante la replicación, es preciso excluir la carpeta *C:\ProgramData\ASR\agent*.

### <a name="if-antivirus-software-is-active-on-configuration-server"></a>Si hay algún antivirus activo en el servidor de configuración

Para que la replicación se realice sin problema alguno y para evitar problemas de conectividad, excluya las siguientes carpetas del software antivirus

- C:\Archivos de programa\Microsoft Azure Recovery Services Agent
- C:\Archivos de programa\Microsoft Azure Site Recovery Provider
- C:\Archivos de programa\Microsoft Azure Site Recovery Configuration Manager 
- C:\Archivos de programa\Microsoft Azure Site Recovery Error Collection Tool 
  - C:\thirdparty
  - C:\Temp
  - C:\strawberry
  - C:\ProgramData\MySQL
  - C:\Archivos de programa (x86)\MySQL
  - C:\ProgramData\ASR
  - C:\ProgramData\Microsoft Azure Site Recovery
  - C:\ProgramData\ASRLogs
  - C:\ProgramData\ASRSetupLogs
  - C:\ProgramData\LogUploadServiceLogs
  - C:\inetpub
  - Directorio de instalación del servidor de Site Recovery. Por ejemplo: E:\Program Files (x86)\Microsoft Azure Site Recovery

### <a name="if-antivirus-software-is-active-on-scale-out-process-servermaster-target"></a>Si el software antivirus está activo en el servidor de procesos o destino maestro de la escalabilidad horizontal

Excluya las siguientes carpetas del software antivirus

1. C:\Archivos de programa\Microsoft Azure Recovery Services Agent
2. C:\ProgramData\ASR
3. C:\ProgramData\ASRLogs
4. C:\ProgramData\ASRSetupLogs
5. C:\ProgramData\LogUploadServiceLogs
6. C:\ProgramData\Microsoft Azure Site Recovery
7. directorio de instalación del servidor de procesos con equilibrio de carga de Azure Site Recovery. Ejemplo: C:\Program Files (x86)\Microsoft Azure Site Recovery

### <a name="if-antivirus-software-is-active-on-the-linux-master-target"></a>Si el software antivirus está activo en el destino maestro de Linux

Excluya las siguientes carpetas del software antivirus

1.  /usr/local/ASR
2.  /usr/local/InMage
3.  /var/log/vxlogs
4.  /var/log
5.  /var/log/ApplicationPolicyLogs
6.  /var/log/ASRsetuptelemetry
7.  /var/log/ASRsetuptelemetry_uploaded


## <a name="next-steps"></a>Pasos siguientes
[Configure el entorno de destino](./vmware-azure-set-up-target.md) 
