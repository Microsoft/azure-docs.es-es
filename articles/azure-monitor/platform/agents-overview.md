---
title: Introducción a los agentes de supervisión de Azure | Microsoft Docs
description: En este artículo se proporciona una descripción detallada de los agentes de Azure disponibles que permiten supervisar las máquinas virtuales hospedadas en Azure o el entorno híbrido.
services: azure-monitor
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/05/2020
ms.openlocfilehash: e6a4c7fe739bd517646f8401e5c812a557441e9f
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88076904"
---
# <a name="overview-of-azure-monitor-agents"></a>Información general sobre los agentes de Azure Monitor

Las máquinas virtuales y otros recursos de proceso requieren un agente para recopilar datos de supervisión para medir el rendimiento y la disponibilidad de sus cargas de trabajo y sistema operativo invitado. En este artículo se describen los agentes utilizados por Azure Monitor y se ayuda a determinar qué se necesita para cumplir con los requisitos del entorno concreto.

> [!NOTE]
> Actualmente, Azure Monitor tiene varios agentes, debido a la reciente consolidación de Azure Monitor y Log Analytics. Aunque puede haber superposición en sus características, cada uno tiene funcionalidades únicas. En función de los requisitos, puede que necesite uno o varios de los agentes en las máquinas virtuales. 

Puede tener un conjunto específico de requisitos que no se puede cumplir por completo con un solo agente para una máquina virtual concreta. Por ejemplo, puede que quiera usar alertas de métricas que requieran la extensión de Azure Diagnostics, pero que también quieran aprovechar la funcionalidad de Azure Monitor para VM que requiere el agente de Log Analytics y Dependency Agent. En este tipo de casos, puede usar varios agentes; se trata de un escenario común en clientes que requieren funcionalidad de cada uno de ellos.

## <a name="summary-of-agents"></a>Resumen de los agentes

En las tablas siguientes se proporciona una comparación rápida de los agentes de Azure Monitor para Windows y Linux. En la sección siguiente se proporcionan más detalles sobre cada uno de ellos. 

### <a name="windows-agents"></a>Agentes de Windows

| | Agente de Azure Monitor (versión preliminar) | Diagnóstico<br>extensión (WAD) | Log Analytics<br>agente | Dependencia<br>agente |
|:---|:---|:---|:---|:---|
| **Entornos compatibles** | Azure<br>Otra nube<br>Local | Azure | Azure<br>Otra nube<br>Local | Azure<br>Otra nube<br>Local | 
| **Requisitos del agente**  | None | None | None | Requiere el agente de Log Analytics |
| **Datos recopilados** | Registros de eventos<br>Rendimiento | Registros de eventos<br>Eventos de ETW<br>Rendimiento<br>Registros basados en archivos<br>Registros IIS<br>Registros de aplicaciones .NET<br>Volcados de memoria<br>Registros de diagnósticos del agente | Registros de eventos<br>Rendimiento<IIS logs><br>Registros basados en archivos<br>Conclusiones y soluciones<br>Otros servicios | Detalles y dependencias de proceso<br>Métricas de conexión de red |
| **Destinatario de los datos** | Registros de Azure Monitor<br>Métricas de Azure Monitor<br>Azure Storage<br>Centro de eventos | Azure Storage<br>Métricas de Azure Monitor<br>Centro de eventos | Registros de Azure Monitor | Registros de Azure Monitor |


### <a name="linux-agents"></a>Agentes de Linux

| | Agente de Azure Monitor (versión preliminar) | Diagnóstico<br>extensión (WAD) | Telegraf<br>agente | Log Analytics<br>agente | Dependencia<br>agente |
|:---|:---|:---|:---|:---|:---|
| **Entornos compatibles** | Azure | Azure | Azure<br>Otra nube<br>Local | Azure<br>Otra nube<br>Local | Azure<br>Otra nube<br>Local |
| **Requisitos del agente**  | None | None | None | None | Requiere el agente de Log Analytics |
| **Datos recopilados** | syslog<br>Rendimiento | syslog<br>Rendimiento | Rendimiento | syslog<br>Rendimiento| Detalles y dependencias de proceso<br>Métricas de conexión de red |
| **Destinatario de los datos** | Registros de Azure Monitor<br>Azure Storage<br>Métricas de Azure Monitor<br>Centro de eventos | Azure Storage<br>Centro de eventos | Métricas de Azure Monitor | Registros de Azure Monitor | Registros de Azure Monitor |

## <a name="azure-monitor-agent-preview"></a>Agente de Azure Monitor (versión preliminar)
El [agente de Azure Monitor ](azure-monitor-agent-overview.md) se encuentra actualmente en versión preliminar y reemplazará al agente de Log Analytics, la extensión de diagnósticos y el agente de Telegraf para máquinas virtuales Windows y Linux. Puede enviar datos a los registros y métricas de Azure Monitor y usar las [reglas de recopilación de datos (DCR)](data-collection-rule-overview.md) que proporcionan un método más escalable para configurar la recopilación de datos y los destinos de cada agente.

Utilice el agente de Azure Monitor si necesita:

- Recopilar los registros y las métricas de los invitados de cualquier máquina virtual en Azure, en otras nubes o en el entorno local. (Azure solo en versión preliminar).
- Enviar datos a registros y métricas de Azure Monitor para su análisis con Azure Monitor. 
- Enviar datos a Azure Storage para su archivado.
- Enviar datos a herramientas de terceros mediante [Azure Event Hubs](diagnostics-extension-stream-event-hubs.md).
- Administrar la seguridad de las máquinas virtuales mediante [Azure Security Center](../../security-center/security-center-intro.md) o [Azure Sentinel](../../sentinel/overview.md). (No disponible en versión preliminar).

Las limitaciones del agente de Azure Monitor incluyen:

- Actualmente en versión preliminar pública. Consulte las [limitaciones actuales](azure-monitor-agent-overview.md#current-limitations) para obtener una lista de las limitaciones durante la versión preliminar pública.

## <a name="log-analytics-agent"></a>Agente de Log Analytics

El [agente de Log Analytics](log-analytics-agent.md) recopila datos de supervisión del sistema operativo invitado y de las cargas de trabajo de las máquinas virtuales de Azure, otros proveedores de nube y el entorno local. Recopila los datos en un área de trabajo de Log Analytics. El agente de Log Analytics es el mismo agente que se usa en System Center Operations Manager y en los equipos de agente de host múltiple para comunicarse con el grupo de administración y Azure Monitor simultáneamente. Este agente también es necesario en ciertas conclusiones y soluciones de Azure Monitor.


> [!NOTE]
> El agente de Log Analytics para Windows a menudo se conoce como Microsoft Monitoring Agent (MMA). El agente de Log Analytics para Linux se conoce a menudo como agente de OMS.



Use el agente de Log Analytics si necesita:

* Recopilar registros y datos de rendimiento de máquinas virtuales o físicas fuera de Azure. 
* Enviar datos a un área de trabajo de Log Analytics para aprovechar las características compatibles con los [registros de Azure Monitor](data-platform-logs.md#what-can-you-do-with-azure-monitor-logs), como las [consultas de registro](../log-query/log-query-overview.md).
* Use [Azure Monitor para VM](../insights/vminsights-overview.md), que le permite supervisar las máquinas virtuales a escala y supervisa sus procesos y dependencias en otros recursos y procesos externos.  
* Administrar la seguridad de las máquinas virtuales mediante [Azure Security Center](../../security-center/security-center-intro.md) o [Azure Sentinel](../../sentinel/overview.md).
* Usar [Update Management de Azure Automation](../../automation/update-management/update-mgmt-overview.md), [State Configuration de Azure Automation](../../automation/automation-dsc-overview.md) o [Change Tracking e Inventario de Azure Automation](../../automation/change-tracking.md) para ofrecer una administración completa de las VM de Azure.
* Usar diferentes [soluciones](../monitor-reference.md#insights-and-core-solutions) para supervisar un servicio o una aplicación determinados.

Las limitaciones del agente de Log Analytics incluyen:

- No se pueden enviar datos a métricas de Azure Monitor, Azure Storage ni Azure Event Hubs.

## <a name="azure-diagnostics-extension"></a>Extensión de Diagnósticos de Azure

La [extensión Azure Diagnostics](diagnostics-extension-overview.md) recopila datos de supervisión del sistema operativo invitado y de las cargas de trabajo de las máquinas virtuales de Azure y otros recursos de proceso. Principalmente, recopila datos en Azure Storage, pero también permite definir receptores de datos para enviar datos a otros destinos, como métricas de Azure Monitor y Azure Event Hubs.

Use la extensión Azure Diagnostics si necesita:

- Enviar datos a Azure Storage para archivarlos o analizarlos con herramientas como [Explorador de Azure Storage](../../vs-azure-tools-storage-manage-with-storage-explorer.md).
- Enviar datos a [métricas de Azure Monitor](data-platform-metrics.md) para analizarlos con el [explorador de métricas](metrics-getting-started.md) y para aprovechar las característica, como las [alertas de métricas](./alerts-metric-overview.md) casi en tiempo real y la [escalabilidad automática](autoscale-overview.md) (solo Windows).
- Enviar datos a herramientas de terceros mediante [Azure Event Hubs](diagnostics-extension-stream-event-hubs.md).
- Recopilar [diagnósticos de arranque](../../virtual-machines/troubleshooting/boot-diagnostics.md) para investigar los problemas de arranque de VM.

Las limitaciones de la extensión Azure Diagnostics incluyen:

- Solo se puede utilizar con recursos de Azure.
- Tiene capacidad limitada para enviar datos a los registros de Azure Monitor.

## <a name="telegraf-agent"></a>Agente Telegraf

El [agente de InfluxData Telegraf](collect-custom-metrics-linux-telegraf.md) se usa para recopilar datos de rendimiento de equipos Linux en métricas de Azure Monitor.

Use el agente Telegraf si necesita:

* Enviar datos a [métricas de Azure Monitor](data-platform-metrics.md) para analizarlos con el [explorador de métricas](metrics-getting-started.md) y para aprovechar las característica, como las [alertas de métricas](./alerts-metric-overview.md) casi en tiempo real y la [escalabilidad automática](autoscale-overview.md) (solo Linux). 



## <a name="dependency-agent"></a>Dependency Agent

Dependency Agent recopila datos detectados acerca de los procesos que se ejecutan en las dependencias de las máquinas virtuales y los procesos externos. 

Use Dependency Agent si necesita:

* Usar la característica de asignación para [Azure Monitor para VM](../insights/vminsights-overview.md) o la solución [Service Map](../insights/service-map.md).

Tenga en cuenta lo siguiente al usar Dependency Agent:

- Dependency Agent requiere que el agente de Log Analytics se instale en la misma máquina virtual.
- En las máquinas virtuales de Linux, el agente de Log Analytics debe instalarse antes que la extensión Azure Diagnostics.

## <a name="extensions-compared-to-agents"></a>Comparación de extensiones y agentes

La extensión Log Analytics para [Windows](../../virtual-machines/extensions/oms-windows.md) y [Linux](../../virtual-machines/extensions/oms-linux.md) instala el agente de Log Analytics en máquinas virtuales de Azure. La extensión Dependency de Azure Monitor para [Windows](../../virtual-machines/extensions/agent-dependency-windows.md) y [Linux](../../virtual-machines/extensions/agent-dependency-linux.md) instala Dependency Agent en máquinas virtuales de Azure. Se trata de los mismos agentes descritos anteriormente, pero permiten administrarlos a través de [extensiones de máquinas virtuales](../../virtual-machines/extensions/overview.md). Debe usar extensiones para instalar y administrar los agentes siempre que sea posible.


## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cada uno de los agentes en los siguientes artículos:

- [Introducción al agente de Log Analytics](log-analytics-agent.md)
- [Introducción a la extensión Azure Diagnostics](diagnostics-extension-overview.md)
- [Recopilación de métricas personalizadas para una máquina virtual Linux con el agente de InfluxData Telegraf](collect-custom-metrics-linux-telegraf.md)

