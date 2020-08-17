---
title: Configuración de los orígenes de datos de agente en Azure Monitor | Microsoft Docs
description: Los orígenes de datos definen los datos de registro que Azure Monitor recopila de agentes y otros orígenes conectados.  En este artículo se describe el concepto de cómo Azure Monitor usa los orígenes de datos, se explican los detalles de cómo configurarlos y se brinda un resumen de los distintos orígenes de datos disponibles.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 11/28/2018
ms.openlocfilehash: a183589c3e5274cf747164cdc33d46044f95e716
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073684"
---
# <a name="agent-data-sources-in-azure-monitor"></a>Orígenes de datos de agente en Azure Monitor
Los orígenes de datos que configura definen los datos que recopila Azure Monitor de los agentes.  Los datos de los agentes se almacenan como [datos de registro](data-platform-logs.md) con un conjunto de registro.  Cada origen de datos crea registros de un tipo determinado, donde cada tipo tiene su propio conjunto de propiedades.

![Recopilación de datos de registro](media/agent-data-sources/overview.png)

## <a name="summary-of-data-sources"></a>Resumen de orígenes de datos
En la tabla siguiente se enumeran los orígenes de datos de agente que actualmente se encuentran disponibles en Azure Monitor.  Cada uno de ellos tiene un vínculo a un artículo independiente, donde se proporcionan detalles con respecto al origen de datos determinado.   También se proporciona información sobre el método y la frecuencia de recopilación. 


| Origen de datos | Plataforma | Agente de Log Analytics | Agente de Operations Manager | Almacenamiento de Azure | ¿Se requiere Operations Manager? | Se envían los datos del agente de Operations Manager a través del grupo de administración | Frecuencia de recopilación |
| --- | --- | --- | --- | --- | --- | --- | --- |
| [Registros personalizados](data-sources-custom-logs.md) | Windows |&#8226; |  | |  |  | a la llegada |
| [Registros personalizados](data-sources-custom-logs.md) | Linux   |&#8226; |  | |  |  | a la llegada |
| [Registros de IIS](data-sources-iis-logs.md) | Windows |&#8226; |&#8226; |&#8226; |  |  |depende de la configuración de la sustitución incremental de archivos de registro |
| [Contadores de rendimiento](data-sources-performance-counters.md) | Windows |&#8226; |&#8226; |  |  |  |mínimo de 10 segundos, según lo programado |
| [Contadores de rendimiento](data-sources-performance-counters.md) | Linux |&#8226; |  |  |  |  |mínimo de 10 segundos, según lo programado |
| [Syslog](data-sources-syslog.md) | Linux |&#8226; |  |  |  |  |Del almacenamiento de Azure: 10 minutos; del agente: a la llegada |
| [Registros de eventos de Windows](data-sources-windows-events.md) |Windows |&#8226; |&#8226; |&#8226; |  |&#8226; | a la llegada |


## <a name="configuring-data-sources"></a>Configuración de orígenes de datos
Los orígenes de datos se configuran en el menú **Datos** en **Configuración avanzada** del área de trabajo.  Cualquier configuración se proporciona a todos los orígenes conectados del área de trabajo.  Actualmente, no puede excluir ningún agente de esta configuración.

![Configurar eventos de Windows](media/agent-data-sources/configure-events.png)

1. En Azure Portal, seleccione las **áreas de trabajo de Log Analytics**  > su área de trabajo > **Configuración avanzada**.
2. Seleccione **Datos**.
3. Haga clic en el origen de datos que quiere configurar.
4. Siga el vínculo a la documentación correspondiente a cada origen de datos de la tabla anterior para obtener detalles sobre su configuración.


## <a name="data-collection"></a>datos, recopilación
Las configuraciones de orígenes de datos se entregan en cuestión de minutos a los agentes que están directamente conectados con Azure Monitor.  Los datos especificados se recopilan desde el agente y se entregan directamente a Azure Monitor a intervalos específicos a cada origen de datos.  Consulte la documentación de cada origen de datos para ver estas especificaciones.

En el caso de los agentes de System Center Operations Manager de un grupo de administración conectado, la configuración de origen de datos se convierte en módulos de administración y se proporciona al grupo de administración cada 5 minutos, de manera predeterminada.  El agente descarga el módulo de administración de la forma habitual y recopila los datos especificados. En función del origen de datos, los datos se enviarán a un servidor de administración que los reenvía a Azure Monitor o el agente los enviará a Azure Monitor sin pasar por el servidor de administración. Consulte [Detalles de la recopilación de datos para las soluciones de supervisión en Azure](../monitor-reference.md) para más información.  Puede leer detalles sobre cómo conectar Operations Manager y Azure Monitor y modificar la frecuencia con que la configuración se proporciona en [Configuración de integración con System Center Operations Manager](om-agents.md).

Si el agente no puede conectarse a Azure Monitor u Operations Manager, continuará recopilando datos para entregarlos cuando se establezca la conexión.  Si el volumen de datos alcanza el tamaño máximo de la memoria caché del cliente o si el agente no es capaz de establecer conexión en un plazo de 24 horas, los datos pueden perderse.

## <a name="log-records"></a>Entradas de registros
Todos los datos que recopila Azure Monitor se almacenan en el área de trabajo como registros.  Los registros que recopilan distintos orígenes de datos tendrán su propio conjunto de propiedades y se identificar por su propiedad **Type** .  Consulte la documentación de cada origen de datos y solución para obtener detalles sobre cada tipo de registro.

## <a name="next-steps"></a>Pasos siguientes
* Conozca las [soluciones de supervisión](../insights/solutions.md) que agregan funcionalidad a Azure Monitor y que también recopilan datos en el área de trabajo.
* Obtenga información acerca de las [consultas de registros](../log-query/log-query-overview.md) para analizar los datos recopilados de soluciones de supervisión y orígenes de datos.  
* Configure [alertas](alerts-overview.md) que le notifiquen de manera proactiva acerca de los datos críticos recopilados de soluciones de supervisión y orígenes de datos.
