---
title: Alerta sobre problemas en Azure Cloud Services mediante la integración de Azure Diagnostics con Azure Application Insights | Microsoft Docs
description: Monitor de problemas como errores de inicio, bloqueos y bucles de reciclaje de roles en Azure Cloud Services con Azure Application Insights
ms.topic: conceptual
ms.date: 06/07/2018
ms.reviewer: harelbr
ms.openlocfilehash: 1cdfc6dc3ac74997743512ee07f9293699e3ad10
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87309297"
---
# <a name="alert-on-issues-in-azure-cloud-services-using-the-azure-diagnostics-integration-with-azure-application-insights"></a>Alerta sobre problemas en Azure Cloud Services mediante la integración de Azure Diagnostics con Azure Application Insights

En este artículo se describirá cómo configurar reglas de alerta para supervisan problemas como errores de inicio, bloqueos y bucles de reciclaje de roles en Azure Cloud Services (roles web y de trabajo).

El método descrito en este artículo se basa en la [integración Azure Diagnostics con Application Insights](https://azure.microsoft.com/blog/azure-diagnostics-integration-with-application-insights/) y la funcionalidad de [alertas de registro para Application Insights](https://azure.microsoft.com/blog/log-alerts-for-application-insights-preview/) lanzada recientemente.

## <a name="define-a-base-query"></a>Definición de una consulta base

Para empezar, definiremos una consulta base que recupera los eventos del registro de eventos de Windows desde el canal de Microsoft Azure, que se capturan en Application Insights como registros de seguimiento.
Estos registros se pueden utilizar para detectar diversos problemas en Azure Cloud Services, como errores de inicio, errores en tiempo de ejecución y bucles de reciclaje.

> [!NOTE]
> La siguiente consulta base comprueba si hay problemas en un período de tiempo de treinta minutos y supone una latencia de diez minutos en la ingesta de los registros de telemetría. Estos valores predeterminados se pueden configurar como considere oportuno.

```
let window = 30m;
let endTime = ago(10m);
let EventLogs = traces
| where timestamp > endTime - window and timestamp < endTime
| extend channel = tostring(customDimensions.Channel), eventId = tostring(customDimensions.EventId)
| where channel == 'Windows Azure' and isnotempty(eventId)
| where tostring(customDimensions.DeploymentName) !contains 'deployment' // discard records captured from local machines
| project timestamp, channel, eventId, message, cloud_RoleInstance, cloud_RoleName, itemCount;
```

## <a name="check-for-specific-event-ids"></a>Comprobación de identificadores de eventos específicos

Después de recuperar los eventos del registro de eventos de Windows, se pueden detectar problemas específicos mediante la comprobación de sus propiedades de mensaje y de identificador de evento correspondientes (vea los ejemplos siguientes).
Simplemente combine la consulta base anterior con una de las consultas siguientes y use esa consulta combinada cuando defina la regla de alerta de registro.

> [!NOTE]
> En los ejemplos siguientes, se detectará un problema si se encuentran más de tres eventos durante el período de tiempo analizado. Este comportamiento predeterminado puede configurarse para cambiar la sensibilidad de la regla de alerta.

```
// Detect failures in the OnStart method
EventLogs
| where eventId == '2001'
| where message contains '.OnStart()'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

```
// Detect failures during runtime
EventLogs
| where eventId == '2001'
| where message contains '.Run()'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

```
// Detect failures when running a startup task
EventLogs
| where eventId == '1000'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

```
// Detect recycle loops
EventLogs
| where eventId == '1006'
| summarize Failures = sum(itemCount) by cloud_RoleInstance, cloud_RoleName
| where Failures > 3
```

## <a name="create-an-alert"></a>Crear una alerta

En el menú de navegación del recurso de Application Insights, vaya a **Alertas** y luego seleccione **Nueva regla de alertas**.

![Captura de pantalla de creación de regla](./media/proactive-cloud-services/001.png)

En la ventana **Crear regla**, en la sección **Definir condición de la alerta**, haga clic en **Agregar criterios** y luego seleccione **Búsqueda de registros personalizada**.

![Captura de pantalla de definición de los criterios de condición de alerta](./media/proactive-cloud-services/002.png)

En el cuadro **Consulta de búsqueda**, pegue la consulta combinada que preparó en el paso anterior.

Luego, siga con el cuadro **Umbral** y establezca su valor en 0. Si lo desea, puede ajustar los campos **Período** y **Frecuencia**.
Haga clic en **Done**(Listo).

![Captura de pantalla de la configuración de consulta de lógica de señal](./media/proactive-cloud-services/003.png)

En la sección **Definir detalles de la alerta**, proporcione un **nombre** y **descripción** a la regla de alerta y establezca su **gravedad**.
Además, asegúrese de que el botón **Habilitar regla tras la creación** se establece en **Sí**.

![Captura de pantalla de detalles de alerta](./media/proactive-cloud-services/004.png)

En la sección **Definir grupo de acciones**, puede seleccionar un **grupo de acciones** existente o crear uno nuevo.
Puede elegir que el grupo de acciones contenga varias acciones de distintos tipos.

![Captura de pantalla de grupo de acciones](./media/proactive-cloud-services/005.png)

Una vez que haya definido el grupo de acciones, confirme los cambios y haga clic en **Crear regla de alertas**.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la detección automática:

[Anomalías de error](./proactive-failure-diagnostics.md)
[Fugas de memoria](./proactive-potential-memory-leak.md)
[Anomalías de rendimiento](./proactive-performance-diagnostics.md)

