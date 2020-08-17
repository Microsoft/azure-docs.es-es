---
title: Introducción a Azure Diagnostics Extension
description: Use Diagnósticos de Azure para realizar tareas de depuración, medición de rendimiento, supervisión y análisis de tráfico en servicios en la nube, en máquinas virtuales y en Service Fabric
ms.subservice: diagnostic-extension
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/14/2020
ms.openlocfilehash: 6201a4e0551f0f75dde65b2bc4b8b560a0f5ea20
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87008003"
---
# <a name="azure-diagnostics-extension-overview"></a>Introducción a Azure Diagnostics Extension
Azure Diagnostics Extension es un [agente de Azure Monitor](agents-overview.md) que recopila datos de supervisión del sistema operativo invitado de los recursos de proceso de Azure, máquinas virtuales incluidas. En este artículo se proporciona información general sobre Azure Diagnostics Extension, incluida la funcionalidad específica que admite y las opciones de instalación y configuración. 

> [!NOTE]
> Azure Diagnostics Extension es uno de los agentes disponibles para recopilar datos de supervisión del sistema operativo invitado y de los recursos de proceso de Azure. Consulte [Introducción a los agentes de Azure Monitor](agents-overview.md) para ver una descripción de los distintos agentes e instrucciones sobre cómo seleccionar los que necesite.

## <a name="primary-scenarios"></a>Escenarios principales
Los escenarios principales abordados por la extensión de diagnóstico son:

- Recopilación de métricas de invitado en métricas de Azure Monitor.
- Envío de registros y métricas de invitado a Azure Storage para su archivado.
- Envío de registros y métricas de invitado a Azure Event Hubs para enviarlos fuera de Azure.


## <a name="comparison-to-log-analytics-agent"></a>Comparación con el agente de Log Analytics
El agente de Log Analytics en Azure Monitor también sirve para recopilar datos de supervisión del sistema operativo invitado de máquinas virtuales. Puede optar por usar uno o ambos, según sus necesidades. Consulte [Introducción a los agentes de Azure Monitor](agents-overview.md) para ver una comparación detallada entre los agentes de Azure Monitor. 

Las principales diferencias que debe tener en cuenta son:

- Azure Diagnostics Extension solo se puede usar con máquinas virtuales de Azure. El agente de Log Analytics se puede usar con máquinas virtuales de Azure, de otras nubes y del entorno local.
- Azure Diagnostics Extension envía datos a Azure Storage, a las [métricas de Azure Monitor](data-platform-metrics.md) (solo en Windows) y a Event Hubs. El agente de Log Analytics recopila los datos en los [registros en Azure Monitor](data-platform-logs.md).
- El agente de Log Analytics es necesario para las [soluciones](../monitor-reference.md#insights-and-core-solutions), [Azure Monitor para VM](../insights/vminsights-overview.md) y otros servicios, como [Azure Security Center](../../security-center/index.yml).

## <a name="costs"></a>Costos
La extensión de Azure Diagnostics no cuesta nada, pero puede incurrir en cargos por los datos ingeridos. Compruebe la página [Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) del destino en el que va a recopilar los datos.

## <a name="data-collected"></a>Datos recopilados
En las tablas siguientes se enumeran los datos que se pueden recopilar con la extensión Diagnostics de Windows y de Linux.

### <a name="windows-diagnostics-extension-wad"></a>Extensión Diagnostics de Windows (WAD)

| Origen de datos | Descripción |
| --- | --- |
| Registros de eventos de Windows   | Eventos del registro de eventos de Windows. |
| Contadores de rendimiento | Valores numéricos que miden el rendimiento de diferentes aspectos del sistema operativo y las cargas de trabajo. |
| Registros IIS             | Información sobre el uso de los sitios web de IIS que se ejecutan en el sistema operativo invitado. |
| Registros de aplicación     | Seguimiento de mensajes escritos por la aplicación. |
| Registros de .NET EventSource |Eventos de escritura de código mediante la clase [EventSource](/dotnet/api/system.diagnostics.tracing.eventsource?view=netcore-3.1) de .NET |
| [Registros de ETW basado en manifiesto](/windows/desktop/etw/about-event-tracing) |Seguimiento de eventos para Windows generados por cualquier proceso. |
| Volcados de memoria (registros)   | Información sobre el estado del proceso en caso de bloqueo de una aplicación. |
| Registros basados en archivos    | Archivos creados por su aplicación o servicio. |
| Registros de diagnóstico del agente | Información sobre Azure Diagnostics. |


### <a name="linux-diagnostics-extension-lad"></a>Extensión Diagnostics de Linux (LAD)

| Origen de datos | Descripción |
| --- | --- |
| syslog | Eventos enviados al sistema de registro de eventos de Linux.   |
| Contadores de rendimiento  | Valores numéricos que miden el rendimiento de diferentes aspectos del sistema operativo y las cargas de trabajo. |
| Archivos de registro | Entradas enviadas a un registro basado en archivos.  |

## <a name="data-destinations"></a>Destinos de datos
La extensión Azure Diagnostics para Windows y Linux siempre recopila datos en una cuenta de Azure Storage. Consulte el artículo acerca de [Instalación y configuración de Azure Diagnostics Extension de Windows (WAD)](diagnostics-extension-windows-install.md) y [Uso de la extensión Diagnostics de Linux para supervisar métricas y registros](../../virtual-machines/extensions/diagnostics-linux.md) para una lista de las tablas y los blobs donde se recopilan estos datos.

Configure uno o varios *receptores de datos* para enviar datos a otros destinos adicionales. En las secciones siguientes se enumeran los receptores disponibles para la extensión Diagnostics de Windows y Linux.

### <a name="windows-diagnostics-extension-wad"></a>Extensión Diagnostics de Windows (WAD)

| Destination | Descripción |
|:---|:---|
| Métricas de Azure Monitor | Recopila datos de rendimiento en Métricas en Azure Monitor. Consulte el artículo [Envío de métricas de sistema operativo invitado a la base de datos de métricas de Azure Monitor](collect-custom-metrics-guestos-resource-manager-vm.md).  |
| Centros de eventos | Usa Azure Event Hubs para enviar datos fuera de Azure. Consulte [Transmisión de datos de Azure Diagnostics a Event Hubs](diagnostics-extension-stream-event-hubs.md). |
| Blobs de Azure Storage | Escribe los datos en blobs de Azure Storage además de en tablas. |
| Application Insights | Recopila datos de las aplicaciones que se ejecutan en la máquina virtual en Application Insights para integrarlos con la supervisión de otras aplicaciones. Consulte el artículo sobre el [Envío de datos de diagnóstico a Application Insights](diagnostics-extension-to-application-insights.md). |

También puede recopilar datos de WAD del almacenamiento en un área de trabajo de Log Analytics para analizarlos con Registros en Azure Monitor, aunque normalmente se usa el agente de Log Analytics para esta funcionalidad. Puede enviar datos directamente a un área de trabajo de Log Analytics y admite soluciones e información que proporcionan funcionalidades adicionales.  Consulte el artículo sobre la [Recopilación de registros de diagnóstico de Azure desde Azure Storage](diagnostics-extension-logs.md). 


### <a name="linux-diagnostics-extension-lad"></a>Extensión Diagnostics de Linux (LAD)
LAD escribe los datos en tablas en Azure Storage. Es compatible con los receptores de la tabla siguiente.

| Destination | Descripción |
|:---|:---|
| Centros de eventos | Usa Azure Event Hubs para enviar datos fuera de Azure. |
| Blobs de Azure Storage | Escribe los datos en blobs de Azure Storage además de en tablas. |
| Métricas de Azure Monitor | Instala el agente de Telegraf además de LAD. Consulte [Recopilación de métricas personalizadas para una máquina virtual Linux con el agente de InfluxData Telegraf](collect-custom-metrics-linux-telegraf.md).


## <a name="installation-and-configuration"></a>Instalación y configuración
La extensión Diagnostics se implementa como una [extensión de máquina virtual](../../virtual-machines/extensions/overview.md) en Azure, por lo que admite las mismas opciones de instalación con las plantillas de Resource Manager, PowerShell y la CLI. Consulte [Características y extensiones de máquina virtual de Windows](../../virtual-machines/extensions/features-windows.md) y [Características y extensiones de máquina virtual de Linux](../../virtual-machines/extensions/features-linux.md) para información general sobre la instalación y el mantenimiento de extensiones de máquina virtual.

También puede instalar y configurar la extensión Diagnostics de Windows y Linux en Azure Portal en **Configuración de diagnóstico** en la sección **Supervisión** del menú de la máquina virtual.

Consulte los artículos siguientes para más información sobre la instalación y la configuración de la extensión Diagnostics de Windows y de Linux.

- [Instalación y configuración de Azure Diagnostics Extension de Windows (WAD)](diagnostics-extension-windows-install.md)
- [Uso de la extensión Diagnostics de Linux para supervisar métricas y registros](../../virtual-machines/extensions/diagnostics-linux.md)

## <a name="other-documentation"></a>Otra documentación

###  <a name="azure-cloud-service-classic-web-and-worker-roles"></a>Roles web y de trabajo del servicio en la nube de Azure (clásico)
- [Introducción a la supervisión de servicios en la nube](../../cloud-services/cloud-services-how-to-monitor.md)
- [Habilitación de Azure Diagnostics en Azure Cloud Services](../../cloud-services/cloud-services-dotnet-diagnostics.md)
- [Application Insights para Azure Cloud Services](../app/cloudservices.md)<br>[Seguimiento del flujo en una aplicación de Cloud Services  con Diagnósticos de Azure](../../cloud-services/cloud-services-dotnet-diagnostics-trace-flow.md) 

### <a name="azure-service-fabric"></a>Azure Service Fabric
- [Supervisión y diagnóstico de servicios en una configuración de desarrollo de máquina local](../../service-fabric/service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md)

## <a name="next-steps"></a>Pasos siguientes


* Aprenda a [utilizar los contadores de rendimiento en Diagnósticos de Azure](../../cloud-services/diagnostics-performance-counters.md).
* Si tiene problemas al iniciar los diagnósticos o al buscar datos en las tablas de Azure Storage, consulte [Solución de problemas de Azure Diagnostics](diagnostics-extension-troubleshooting.md).
