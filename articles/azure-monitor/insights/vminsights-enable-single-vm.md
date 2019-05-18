---
title: Habilitar Monitor de Azure para máquinas virtuales (versión preliminar) para la evaluación | Microsoft Docs
description: En este artículo se describe cómo habilitar a Azure Monitor para las máquinas virtuales para una máquina virtual de Azure o un conjunto para fines de evaluación de escalado de máquina virtual.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2019
ms.author: magoedte
ms.openlocfilehash: 673f153b551e4b0c89a564c96d6bd9819ca26f5d
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65524091"
---
# <a name="enable-azure-monitor-for-vms-preview-for-evaluation"></a>Habilitar a Monitor de Azure para máquinas virtuales (versión preliminar) para la evaluación

Para evaluar a Azure Monitor para máquinas virtuales (versión preliminar) en un número pequeño de máquinas virtuales de Azure o una sola máquina virtual o el conjunto de escalado de máquinas virtuales, el enfoque más sencillo y más directo para habilitar la supervisión es desde el portal de Azure. Al final de este proceso, habrá comenzado correctamente supervisarlos y obtenga información sobre si experimentan los problemas de rendimiento o disponibilidad. 

Antes de obtener iniciado, no olvide revisar la [requisitos previos](vminsights-enable-overview.md) y comprobar la suscripción y los recursos cumplan los requisitos.  

## <a name="enable-monitoring-for-a-single-azure-vm"></a>Habilitar la supervisión de una sola máquina virtual de Azure
Para habilitar la supervisión de la VM de Azure en Azure Portal, siga estos pasos:

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).

1. Seleccione **Máquinas virtuales**.

1. En la lista, seleccione una máquina virtual.

1. En la página de la VM, en la sección **Supervisión**, seleccione **Insights (versión preliminar)**.

1. En la página **Insights (versión preliminar)**, seleccione **Probar ahora**.

    ![Habilitar Azure Monitor para VM para una máquina virtual](./media/vminsights-enable-single-vm/enable-vminsights-vm-portal-01.png)

1. En la página **Azure Monitor Insights Onboarding** (Incorporación a Insights de Azure Monitor), si tiene un área de trabajo de Log Analytics existente en la misma suscripción, selecciónela en la lista desplegable.  

    La lista preselecciona el área de trabajo y la ubicación predeterminadas en las que se implementa la máquina virtual en la suscripción. 

    >[!NOTE]
    >Si desea crear una nueva área de trabajo de Log Analytics para almacenar los datos de supervisión de la máquina virtual, siga las instrucciones de [crear un área de trabajo de Log Analytics](../../azure-monitor/learn/quick-create-workspace.md) en una de las regiones compatibles enumerados [aquí](vminsights-enable-overview.md#log-analytics).

Después de habilitar la supervisión, pueden pasar unos 10 minutos hasta que pueda ver la métrica de estado de la máquina virtual.

![Habilitar el procesamiento de implementación de supervisión de Azure Monitor para VM](./media/vminsights-enable-single-vm/onboard-vminsights-vm-portal-status.png)

## <a name="enable-monitoring-for-a-single-virtual-machine-scale-set"></a>Habilitar la supervisión de un conjunto de escalado de máquina virtual única

Para habilitar la supervisión de la escala de máquina virtual de Azure establecida en Azure portal, haga lo siguiente:

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).

2. Seleccione **conjuntos de escalado de máquinas virtuales**.

3. En la lista, seleccione un conjunto de escalado de máquinas virtuales.

4. En la máquina virtual de conjunto de escalado de página, en el **supervisión** sección, seleccione **Insights (versión preliminar)**.

5. En el **Insights (versión preliminar)** página, si tiene un área de trabajo de Log Analytics existente que desea usar, selecciónelo en la lista desplegable.

    La lista preselecciona el área de trabajo y la ubicación predeterminadas en las que se implementa la máquina virtual en la suscripción. 

    ![Habilitar a Monitor de Azure para máquinas virtuales para un conjunto de escalado de máquinas virtuales](./media/vminsights-enable-single-vm/enable-vminsights-vmss-portal-01.png)

    >[!NOTE]
    >Si desea crear una nueva área de trabajo de Log Analytics para almacenar los datos de supervisión de la máquina virtual, siga las instrucciones de [crear un área de trabajo de Log Analytics](../learn/quick-create-workspace.md) en una de las regiones compatibles enumerados [aquí](vminsights-enable-overview.md#log-analytics).

Después de habilitar la supervisión, tardará unos 10 minutos antes de poder ver los datos de supervisión para el conjunto de escalado.

>[!NOTE]
>Si usa un modelo de actualización manual para el conjunto de escalado deberá actualizar las instancias para completar la instalación.  Esto puede hacerse desde la página de las instancias en el **configuración** sección.

![Habilitar el procesamiento de implementación de supervisión de Azure Monitor para VM](./media/vminsights-enable-single-vm/onboard-vminsights-vmss-portal-status-01.png)

## <a name="next-steps"></a>Pasos siguientes

Ahora que la supervisión está habilitada para la máquina virtual o un conjunto de escalado de máquinas virtuales, esta información está disponible para su análisis con Azure Monitor para las máquinas virtuales. Para obtener información sobre cómo usar la característica de mantenimiento, consulte [Descripción del estado de las máquinas virtuales de Azure con Azure Monitor para VM (versión preliminar)](vminsights-health.md). Para ver las dependencias de las aplicaciones detectadas, consulte [Uso de la asignación de Azure Monitor para VM (versión preliminar) para conocer los componentes de una aplicación](vminsights-maps.md). Para identificar cuellos de botella y el uso general con el rendimiento de sus VM, consulte el artículo sobre cómo [ver el rendimiento de la VM de Azure](vminsights-performance.md), o bien, para ver las dependencias de la aplicación detectadas, consulte cómo [ver el Azure Monitor para la asignación de VM](vminsights-maps.md).