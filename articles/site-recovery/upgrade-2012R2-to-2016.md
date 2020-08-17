---
title: 'Actualización de Windows Server o System Center VMM 2012 R2 a Windows Server 2016: Azure Site Recovery'
description: Aprenda a configurar la recuperación ante desastres en Azure para máquinas virtuales de Azure Stack con el servicio Azure Site Recovery.
services: site-recovery
author: Sharmistha-Rai
manager: gaggupta
ms.topic: conceptual
ms.service: site-recovery
ms.date: 12/03/2018
ms.author: sharrai
ms.openlocfilehash: 82e178d37074b22cd170d6b2478525dd2ea7ddf1
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87424331"
---
# <a name="upgrade-windows-server-serversystem-center-2012-r2-vmm-to-windows-servervmm-2016"></a>Actualización de Windows Server o System Center 2012 R2 VMM a Windows Server o VMM 2016 

Este artículo muestra cómo actualizar los hosts de Windows Server 2012 R2 y SCVMM 2012 R2 que están configurados con Azure Site Recovery para Windows Server 2016 y SCVMM 2016

Site Recovery contribuye a su estrategia de continuidad empresarial y recuperación ante desastres (BCDR). El servicio garantiza que las cargas de trabajo de máquina virtual sigan estando disponibles cuando se produzcan interrupciones esperadas e inesperadas.

> [!IMPORTANT]
> Cuando actualiza los hosts de Windows Server 2012 R2 que ya están configurados para la replicación con Azure Site Recovery, debe seguir los pasos mencionados en este documento. Cualquier ruta de acceso alternativa elegida para la actualización puede provocar estados no admitidos y puede provocar una interrupción en la replicación o en la capacidad de realizar la conmutación por error.


En este artículo, aprenderá cómo actualizar las siguientes configuraciones en su entorno:

> [!div class="checklist"]
> * **Hosts de Windows Server 2012 R2 que no están administrados por SCVMM** 
> * **Hosts de Windows Server 2012 R2 que se administran mediante un servidor de SCVMM 2012 R2 independiente** 
> * **Hosts de Windows Server 2012 R2 que se administran mediante un servidor de SCVMM 2012 R2 de alta disponibilidad**


## <a name="prerequisites--factors-to-consider"></a>Requisitos previos y factores a considerar

Antes de realizar la actualización, tenga en cuenta lo siguiente:

- Si tiene hosts de Windows Server 2012 R2 que no administra SCVMM, y es una configuración de entorno independiente, habrá una interrupción en la replicación si intenta realizar la actualización.
- Si ha seleccionado "*not store my Keys in Active Directory under Distributed Key Management*" (no almacenar mis claves en Active Directory en Distributed Key Management) durante la instalación de SCVMM 2012 R2, las actualizaciones no se completarán correctamente.

- Si usa System Center 2012 R2 VMM, 

    - Compruebe la información de la base de datos de VMM: **Consola VMM** -> **Configuración** -> **General** -> **Conexión de base de datos**
    - Comprobación de las cuentas de servicio que se utilizan para el servicio System Center Virtual Machine Manager Agent
    - Asegúrese de tener una copia de seguridad de la base de datos de VMM.
    - Anote el nombre de la base de datos de los servidores SCVMM implicados. Esto puede hacerse yendo a **Consola VMM** -> **Configuración** -> **General** -> **Conexión de base de datos**.
    - Anote el identificador de VMM de los servidores VMM principales y de recuperación 2012R2. El identificador de VMM se puede encontrar en el Registro "HKLM:\NSOFTWARE\Microsoft\Microsoft System Center Virtual Machine Manager Server\Setup".
    - Asegúrese de que los nuevos SCVMM que agregue al clúster tengan los mismos nombres que antes. 

- Si está replicando entre dos de sus sitios administrados por SCVMM en ambos lados, asegúrese de actualizar primero su lado de recuperación antes de actualizar el lado principal.
  > [!WARNING]
  > Mientras actualiza la SCVMM 2012 R2, en Administración de claves distribuida, seleccione la opción para **almacenar claves de cifrado en Active Directory**. Elija cuidadosamente la configuración de la cuenta de servicio y administración de claves distribuida. Según su selección, es posible que los datos cifrados, como las contraseñas de las plantillas, no estén disponibles después de la actualización, y pueden afectar potencialmente a la replicación con Azure Site Recovery.

> [!IMPORTANT]
> Consulte la documentación detallada de SCVMM de [requisitos previos](/system-center/vmm/upgrade-vmm?view=sc-vmm-2016#requirements-and-limitations).

## <a name="windows-server-2012-r2-hosts-which-arent-managed-by-scvmm"></a>Hosts de Windows Server 2012 R2 que no están administrados por SCVMM 
La lista de pasos mencionados a continuación se aplica a la configuración de usuario de los [hosts de Hyper-V a Azure](./hyper-v-azure-architecture.md) que se ejecuta siguiendo este [tutorial](./hyper-v-prepare-on-premises-tutorial.md).

> [!WARNING]
> Como se menciona en los requisitos previos, estos pasos solo se aplican a un escenario de entorno en clústeres, y no en una configuración de host de Hyper-V independiente.

1. Siga los pasos para realizar la [actualización gradual de clústeres](/windows-server/failover-clustering/cluster-operating-system-rolling-upgrade#cluster-os-rolling-upgrade-process). para ejecutar el proceso de actualización gradual de clústeres.
2. Con cada nuevo host de Windows Server 2016 que se introduzca en el clúster, quite la referencia de un host de Windows Server 2012 R2 de Azure Site Recovery siguiendo los pasos mencionados [aquí]. Debe ser el host que ha elegido para purgar y expulsar del clúster.
3. Una vez que el comando *Update-VMVersion* se ha ejecutado para todas las máquinas virtuales, las actualizaciones se han completado. 
4. Utilice los pasos mencionados [aquí](./hyper-v-azure-tutorial.md#set-up-the-source-environment) para registrar el nuevo host de Windows Server 2016 en Azure Site Recovery. Tenga en cuenta que el sitio Hyper-V ya está activo y solo necesita registrar el nuevo host en el clúster. 
5.  Vaya a Azure Portal y compruebe el estado replicado dentro de Recovery Services.

## <a name="upgrade-windows-server-2012-r2-hosts-managed-by-stand-alone-scvmm-2012-r2-server"></a>Actualización de los hosts de Windows Server 2012 R2 administrados por el servidor de SCVMM 2012 R2 independiente
Antes de actualizar los hosts de Windows Sever 2012 R2, necesita actualizar el SCVMM 2012 R2 a SCVMM 2016. Siga los pasos siguientes:

**Actualización independiente de SCVMM 2012 R2 a SCVMM 2016**

1.  Desinstale el proveedor de ASR en Panel de control -> Programas -> Programas y características -> Microsoft Azure Site Recovery y haga clic en Desinstalar.
2. [Conserve la base de datos SCVMM y actualice el sistema operativo](/system-center/vmm/upgrade-vmm?view=sc-vmm-2016#back-up-and-upgrade-the-operating-system)
3. En **Agregar o quitar programas**, seleccione **VMM** > **Desinstalar**. b. Seleccione **Quitar características** y, a continuación, seleccione **Servidor de administración VMM y Consola VMM**. c. En **Opciones de base de datos**, seleccione **Conservar base de datos**. d. Revise el resumen y haga clic en **Desinstalar**.

4. [Instale VMM 2016](/system-center/vmm/upgrade-vmm?view=sc-vmm-2016#install-vmm-2016).
5. Inicie SCVMM y compruebe el estado de cada host en la pestaña **Tejidos**. Haga clic en **Actualizar** para obtener el estado más reciente. Debería ver el estado como "Necesita atención". 
17. Instale el [proveedor de Microsoft Azure Site Recovery](https://aka.ms/downloaddra) más reciente en SCVMM.
16. Instale el [agente de Microsoft Azure Recovery Service (MARS)](https://aka.ms/latestmarsagent) más reciente en cada host del clúster. Actualice para asegurarse de que SCVMM es capaz de consultar correctamente los hosts.

**Actualización de hosts de Windows Server 2012 R2 a Windows Server 2016**

1. Siga los pasos mencionados [aquí](/windows-server/failover-clustering/cluster-operating-system-rolling-upgrade#cluster-os-rolling-upgrade-process) para ejecutar el proceso de actualización gradual de clústeres. 
2. Después de agregar el nuevo host al clúster, actualice el host desde la consola SCVMM para instalar el agente VMM en este host actualizado.
3. Ejecute *Update-VMVersion* para actualizar las versiones de las máquinas virtuales. 
4.  Vaya a Azure Portal y compruebe el estado replicado de las máquinas virtuales dentro del almacén de Recovery Services. 

## <a name="upgrade-windows-server-2012-r2-hosts-are-managed-by-highly-available-scvmm-2012-r2-server"></a>Actualización de hosts de Windows Server 2012 R2 que se administran mediante un servidor de SCVMM 2012 R2 de alta disponibilidad
Antes de actualizar los hosts de Windows Sever 2012 R2, necesita actualizar el SCVMM 2012 R2 a SCVMM 2016. Los siguientes modos de actualización son compatibles con la actualización de los servidores SCVMM 2012 R2 configurados con Azure Site Recovery-modo mixto sin servidores VMM adicionales y modo mixto con servidores VMM adicionales.

**Actualización de SCVMM 2012 R2 a SCVMM 2016**

1.  Desinstale el proveedor de ASR en Panel de control -> Programas -> Programas y características -> Microsoft Azure Site Recovery y haga clic en Desinstalar.
2. Siga los pasos mencionados [aquí](/system-center/vmm/upgrade-vmm?view=sc-vmm-2016#upgrade-a-standalone-vmm-server) según el modo de actualización que desea ejecutar.
3. Inicie la consola de SCVMM y compruebe el estado de cada host en la pestaña **Tejidos**. Haga clic en **Actualizar** para obtener el estado más reciente. Debería ver el estado como "Necesita atención".
4. Instale el [proveedor de Microsoft Azure Site Recovery](https://aka.ms/downloaddra) más reciente en SCVMM.
5. Actualice el [agente de Microsoft Azure Recovery Service (MARS)](https://aka.ms/latestmarsagent) más reciente en cada host del clúster. Actualice para asegurarse de que SCVMM es capaz de consultar correctamente los hosts.


**Actualización de hosts de Windows Server 2012 R2 a Windows Server 2016**

1. Siga los pasos mencionados [aquí](/windows-server/failover-clustering/cluster-operating-system-rolling-upgrade#cluster-os-rolling-upgrade-process) para ejecutar el proceso de actualización gradual de clústeres.
2. Después de agregar el nuevo host al clúster, actualice el host desde la consola SCVMM para instalar el agente VMM en este host actualizado.
3. Ejecute *Update-VMVersion* para actualizar las versiones de las máquinas virtuales. 
4.  Vaya a Azure Portal y compruebe el estado replicado de las máquinas virtuales dentro del almacén de Recovery Services. 

## <a name="next-steps"></a>Pasos siguientes
Cuando se realiza la actualización de los hosts, puede realizar una [conmutación por error de prueba](tutorial-dr-drill-azure.md) para comprobar el estado de la replicación y el estado de la recuperación ante desastres.

