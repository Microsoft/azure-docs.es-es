---
title: Habilitar Azure Monitor para VM para una sola máquina virtual o un conjunto de escalado de máquinas virtuales mediante Azure Portal
description: Obtenga información sobre cómo habilitar Azure Monitor para VM en una sola máquina virtual o un conjunto de escalado de máquinas virtuales de Azure mediante Azure Portal.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: ba6ae9262fb0e55c53e1b1421c075e555fae8d98
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87328032"
---
# <a name="enable-azure-monitor-for-single-virtual-machine-or-virtual-machine-scale-set-in-the-azure-portal"></a>Habilitar Azure Monitor para VM para una sola máquina virtual o un conjunto de escalado de máquinas virtuales mediante Azure Portal
En este artículo se describe cómo habilitar Azure Monitor para VM para una máquina virtual o un conjunto de escalado de máquinas virtuales mediante Azure Portal. Este procedimiento se puede usar para:

- Máquina virtual de Azure
- Conjunto de escalado de máquinas virtuales de Azure
- Máquina virtual híbrida conectada con Azure Arc

## <a name="prerequisites"></a>Requisitos previos

- [Cree y configure un área de trabajo de Log Analytics](vminsights-configure-workspace.md). Como alternativa, puede crear un área de trabajo durante este proceso.
- Vea [Sistemas operativos admitidos](vminsights-enable-overview.md#supported-operating-systems) para asegurarse de que el sistema operativo de la máquina virtual o el conjunto de escalado de máquinas virtuales que va a habilitar son compatibles. 

## <a name="enable-azure-monitor-for-vms"></a>Habilitar Azure Monitor para VM

En el Azure Portal, seleccione **Máquinas virtuales**, **Conjuntos de escalado de máquinas virtuales** o **Máquinas: Azure Arc** y seleccione un recurso de la lista. En la sección **Supervisión** del menú, seleccione **Insights** y, a continuación, **Habilitar**. En el ejemplo siguiente, se muestra una máquina virtual de Azure, pero el menú es similar para el conjunto de escalado de máquinas virtuales o Azure Arc.

![Habilitar Azure Monitor para VM para una máquina virtual](media/vminsights-enable-single-vm/enable-vminsights-vm-portal.png)

Si la máquina virtual aún no está conectada a un área de trabajo de Log Analytics, se le pedirá que seleccione una. Si anteriormente no ha [creado un área de trabajo](../../azure-monitor/learn/quick-create-workspace.md), puede seleccionar una predeterminada para la ubicación en la que se implementa la máquina virtual o el conjunto de escalado de máquinas virtuales en la suscripción. Se creará y configurará el área de trabajo si aún no existe. Si selecciona un área de trabajo existente, se configurará en Azure Monitor para VM si aún no está configurada.

> [!NOTE]
> Si selecciona un área de trabajo que no se configuró anteriormente en Azure Monitor para VM, se agregará el módulo de administración *VMInsights* a esta área de trabajo. Esto se aplicará a cualquier agente que ya esté conectado al área de trabajo, independientemente de si está habilitado o no en Azure Monitor para VM. Los datos de rendimiento se recopilarán de estas máquinas virtuales y se almacenarán en la tabla *InsightsMetrics*.

![Selección del área de trabajo](media/vminsights-configure-workspace/select-workspace.png)

Recibirá mensajes de estado a medida que se lleve a cabo la configuración.

>[!NOTE]
>Si usa un modelo de actualización manual para el conjunto de escalado de máquinas virtuales, actualice las instancias para completar la configuración. Puede iniciar las actualizaciones desde la página **Instancias**, en la sección **Configuración**.

![Habilitar el procesamiento de implementación de supervisión de Azure Monitor para VM](media/vminsights-enable-single-vm/onboard-vminsights-vm-portal-status.png)



## <a name="next-steps"></a>Pasos siguientes

* Vea [Uso de la característica de asignación de Azure Monitor para VM](vminsights-maps.md) para ver las dependencias de las aplicaciones detectadas. 
* Consulte [Ver el rendimiento de las máquinas virtuales de Azure](vminsights-performance.md) para identificar cuellos de botella, el uso general y el rendimiento de la máquina virtual.
