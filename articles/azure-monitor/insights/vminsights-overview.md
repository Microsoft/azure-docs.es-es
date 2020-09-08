---
title: ¿Qué es Azure Monitor para máquinas virtuales?
description: Información general de Azure Monitor para VM, que se encarga de supervisar el estado y el rendimiento de las VM de Azure, y detecta y asigna automáticamente los componentes de la aplicación y sus dependencias.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/22/2020
ms.openlocfilehash: 5c3cb13d0b2da9370f402083d82397679f2c9343
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022503"
---
# <a name="what-is-azure-monitor-for-vms"></a>¿Qué es Azure Monitor para máquinas virtuales?

Azure Monitor para VM supervisa el rendimiento y el mantenimiento de las máquinas virtuales y los conjuntos de escalado de máquinas virtuales, incluidos los procesos en ejecución y las dependencias de otros recursos. Puede ayudar a ofrecer de forma eficaz un rendimiento predecible y disponibilidad de las aplicaciones vitales mediante la identificación de cuellos de botella de rendimiento y problemas de red, así como ayudar a comprender si un problema está relacionado con otras dependencias.

Azure Monitor para VM admite sistemas operativos Windows y Linux en:

- Azure Virtual Machines
- Conjuntos de escalado de máquinas virtuales de Azure
- Máquinas virtuales híbridas conectadas con Azure Arc
- Máquinas virtuales locales
- Máquinas virtuales hospedadas en otro entorno de nube
  



Azure Monitor para VM almacena sus datos en registros de Azure Monitor, lo que le permite ofrecer una agregación y un filtrado eficaces, y el análisis de las tendencias de los datos a lo largo del tiempo. Puede ver estos datos directamente en una sola VM desde la máquina virtual, o puede usar a Azure Monitor para ofrecer una vista agregada de varias VM.

![Perspectiva de conclusiones de la máquina virtual desde Azure Portal](media/vminsights-overview/vminsights-azmon-directvm.png)


## <a name="pricing"></a>Precios
No hay ningún costo directo por Azure Monitor para VM, pero se le cobrará por la actividad en el área de trabajo de Log Analytics. Según los precios que se publican en la [página de precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/), Azure Monitor para VM se factura según:

- Datos ingeridos de agentes y almacenados en el área de trabajo.
- Las reglas de alertas basadas en el registro y los datos de estado.
- Las notificaciones enviadas desde reglas de alertas.

El tamaño del registro varía en función de las longitudes de cadena de los contadores de rendimiento y puede aumentar con el número de discos lógicos y adaptadores de red asignados a la máquina virtual. Si ya usa Service Map, el único cambio que verá son los datos de rendimiento adicionales que se envían al tipo de datos `InsightsMetrics` de Azure Monitor.


## <a name="configuring-azure-monitor-for-vms"></a>Configuración de Azure Monitor para VM
Los pasos para configurar Azure Monitor para VM son los siguientes. Siga cada vínculo para obtener instrucciones detalladas sobre cada paso:

- [Creación de un área de trabajo de Log Analytics.](vminsights-configure-workspace.md#create-log-analytics-workspace)
- [Incorporación de la solución VMInsights al área de trabajo.](vminsights-configure-workspace.md#add-vminsights-solution-to-workspace)
- [Instalación de agentes en la máquina virtual y el conjunto de escalado de máquinas virtuales que se van a supervisar.](vminsights-enable-overview.md)



## <a name="next-steps"></a>Pasos siguientes

- Consulte [Implementación de Azure Monitor para VM](vminsights-enable-overview.md) para conocer los requisitos y los métodos para habilitar la supervisión de máquinas virtuales.

