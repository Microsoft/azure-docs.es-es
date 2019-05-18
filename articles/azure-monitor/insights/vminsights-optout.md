---
title: Deshabilitación de la supervisión con Azure Monitor para VM (versión preliminar) | Microsoft Docs
description: En este artículo se describe cómo puede interrumpir la supervisión de máquinas virtuales con Azure Monitor para VM.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/05/2018
ms.author: magoedte
ms.openlocfilehash: 0f35ea3e35277ee7f1afd8278a31f45ed20c6995
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522117"
---
# <a name="how-to-disable-monitoring-of-your-virtual-machines-with-azure-monitor-for-vms-preview"></a>Deshabilitación de la supervisión de las máquinas virtuales con Azure Monitor para VM (versión preliminar)

Si después de habilitar la supervisión de las máquinas virtuales decide que ya no desea supervisarlas con Azure Monitor para VM, puede deshabilitar la supervisión. En este artículo se muestra cómo realizar esta tarea para una o varias máquinas virtuales.  

Actualmente, Azure Monitor para VM no permite deshabilitar de forma selectiva la supervisión de las máquinas virtuales. Si el área de trabajo de Log Analytics está configurada para admitir esta solución y otras soluciones, así como para recopilar otros datos de supervisión, es importante que comprenda el impacto y los métodos descritos a continuación antes de continuar.

Azure Monitor para VM se basa en los siguientes componentes para ofrecer su experiencia:

* Un área de trabajo de Log Analytics, que almacena los datos de supervisión recogidos de las máquinas virtuales y otros orígenes.
* La colección de contadores de rendimiento configurados en el área de trabajo, que actualiza la configuración de supervisión de todas las máquinas virtuales conectadas al área de trabajo.
* Dos soluciones de supervisión configuradas en el área de trabajo, **InfrastructureInsights** y **ServiceMap**, que actualizan la configuración de supervisión en todas las máquinas virtuales conectadas al área de trabajo.
* Dos extensiones de máquinas virtuales de Azure, **MicrosoftMonitoringAgent** y **DependencyAgent**, que recopilan y envían datos al área de trabajo.

Considere lo siguiente al preparar la deshabilitación de la supervisión de las máquinas virtuales con Azure Monitor para VM:

* Si realiza la evaluación con una sola máquina virtual y ha aceptado el área de trabajo de Log Analytics predeterminada preseleccionada, puede deshabilitar la supervisión mediante la desinstalación del agente de dependencia de la máquina virtual y la desconexión del agente de Log Analytics de esta área de trabajo. Este enfoque es adecuado si tiene la intención de utilizar la máquina virtual para otros fines y decide posteriormente volver a conectarla a un área de trabajo diferente.
* Si utiliza el área de trabajo de Log Analytics para admitir otras soluciones de supervisión y recopilación de datos de otros orígenes, puede quitar los componentes de la solución Azure Monitor para VM del área de trabajo sin interrupción ni impacto en el área de trabajo.  

>[!NOTE]
> Después de quitar los componentes de la solución del área de trabajo, puede continuar viendo el estado de mantenimiento de las máquinas virtuales de Azure; específicamente, datos de rendimiento y de mapa cuando se desplaza a cualquiera de las dos vistas en el portal. Eventualmente, los datos dejarán de aparecer en la vista de rendimiento y mapa después de algún tiempo; sin embargo, la vista de mantenimiento continuará mostrando el estado de mantenimiento de las máquinas virtuales. La opción **Probar ahora** estará disponible en la máquina virtual de Azure seleccionada para que pueda volver a habilitar la supervisión en el futuro.  

## <a name="complete-removal-of-azure-monitor-for-vms"></a>Eliminación completa de Azure Monitor para VM

Los pasos siguientes describen cómo quitar por completo Azure Monitor para VM si aún necesita el área de trabajo de Log Analytics. Va a quitar las soluciones **InfrastructureInsights** y **ServiceMap** del área de trabajo.  

>[!NOTE]
>Si estaba usando la solución de supervisión de Service Map antes de habilitar Azure Monitor para VM y todavía confía en ella, no quite esa solución como se describe en el paso 6 siguiente.  
>

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. En Azure Portal, haga clic en **Todos los servicios**. En la lista de recursos, escriba Log Analytics. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Log Analytics**.
3. En la lista de áreas de trabajo de Log Analytics, seleccione el área de trabajo que ha elegido al incorporarse a Azure Monitor para VM.
4. En el panel izquierdo, seleccione **Soluciones**.  
5. En la lista de soluciones, seleccione **InfrastructureInsights(nombre del área de trabajo)** y, después, en la página **Información general** para la solución, haga clic en **Eliminar**.  Cuando se le solicite confirmar, haga clic en **Sí**.  
6. En la lista de soluciones, seleccione **ServiceMap(nombre del área de trabajo)** y, después, en la página **Información general** para la solución, haga clic en **Eliminar**.  Cuando se le solicite confirmar, haga clic en **Sí**.  

Si antes de incorporarse a Azure Monitor para VM, no estaba [recopilando los contadores de rendimiento habilitados](vminsights-enable-overview.md#performance-counters-enabled) para las máquinas virtuales basadas en Windows o Linux en el área de trabajo, necesita deshabilitar esas reglas siguiendo los pasos descritos [aquí](../platform/data-sources-performance-counters.md#configuring-performance-counters) para Windows y para Linux.

## <a name="disable-monitoring-for-an-azure-vm-and-retain-workspace"></a>Deshabilitación de la supervisión de una máquina virtual de Azure y conservación del área de trabajo  

Los pasos siguientes describen cómo deshabilitar la supervisión de una máquina virtual que se habilitó para evaluar Azure Monitor para VM, pero el área de trabajo de Log Analytics sigue siendo necesario para admitir la supervisión desde otros orígenes. Si se trata de una máquina virtual de Azure, va a quitar la extensión de máquina virtual del agente de dependencias y la extensión de máquina virtual del agente de Log Analytics para Windows o Linux directamente de la máquina virtual. 

>[!NOTE]
>Si la máquina virtual la administra Azure Automation para orquestar los procesos, administrar la configuración o administrar las actualizaciones, o la administra Azure Security Center para la administración de seguridad y detección de amenazas, no se debe quitar el agente de Log Analytics. De lo contrario, evitará que esos servicios y soluciones administren de forma proactiva la máquina virtual. 

1. Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com). 
2. En Azure Portal, seleccione **Virtual Machines**. 
3. En la lista, seleccione una máquina virtual. 
4. En el panel izquierdo, seleccione **Extensiones** y en la página **Extensiones**, seleccione **DependencyAgent**.
5. En la página de propiedades de la extensión, haga clic en **Desinstalar**.
6. En la página **Extensiones**, seleccione **MicrosoftMonitoringAgent** y, en la página de propiedades de la extensión, haga clic en **Desinstalar**.  
