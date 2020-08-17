---
title: Solución Agent Health en Azure Monitor | Microsoft Docs
description: Este artículo está pensado para ayudarle a entender cómo usar esta solución para supervisar el mantenimiento de los agentes que informan directamente a Log Analytics o System Center Operations Manager.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 02/06/2020
ms.openlocfilehash: 4f14f006283b7430458d67d2bd3bee787c08411d
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326025"
---
#  <a name="agent-health-solution-in-azure-monitor"></a>Solución Agent Health en Azure Monitor
La solución Agent Health en Azure le ayuda a entender, para todos los agentes que informan directamente al área de trabajo de Log Analytics en Azure Monitor o a un grupo de administración de System Center Operations Manager conectado a Azure Monitor, cuáles no responden y cuáles envían datos operativos.  También puede realizar un seguimiento del número de agentes que se implementan, dónde están distribuidos geográficamente y llevar a cabo otras consultas para mantener el conocimiento de la distribución de los agentes implementados en Azure, en otros entornos de nube o en un entorno local.    

## <a name="prerequisites"></a>Prerrequisitos
Antes de implementar esta solución, confirme que tiene actualmente [agentes de Windows](../platform/agent-windows.md) compatibles que informan al área de trabajo de Log Analytics o a un [grupo de administración de Operations Manager](../platform/om-agents.md) integrado con el área de trabajo.

## <a name="solution-components"></a>Componentes de soluciones
Esta solución consta de los siguientes recursos que se agregan a su área de trabajo y a los agentes directamente conectados o al grupo de administración conectado de Operations Manager.

### <a name="management-packs"></a>Módulos de administración
Si el grupo de administración de System Center Operations Manager está conectado al área de trabajo de Log Analytics, se instalarán los siguientes módulos de administración en Operations Manager.  Estos módulos de administración también se instalan en equipos Windows directamente conectados después de agregar esta solución. No hay nada para configurar o administrar en estos módulos de administración.

* Intelligence Pack HealthAssessment Direct Channel de Microsoft System Center Advisor (Microsoft.IntelligencePacks.HealthAssessmentDirect)
* Intelligence Pack HealthAssessment Server Channel de Microsoft System Center Advisor (Microsoft.IntelligencePacks.HealthAssessmentViaServer)  

Para obtener más información sobre cómo se actualizan los módulos de administración de soluciones, consulte [Conexión de Operations Manager con Log Analytics](../platform/om-agents.md).

## <a name="configuration"></a>Configuración
Agregue la solución Agent Health al área de trabajo de Log Analytics mediante el proceso descrito en [Incorporación de soluciones](solutions.md). No es necesario realizar ninguna configuración más.


## <a name="data-collection"></a>datos, recopilación
### <a name="supported-agents"></a>Agentes admitidos
En la tabla siguiente se describen los orígenes conectados que son compatibles con esta solución.

| Origen conectado | Compatible | Descripción |
| --- | --- | --- |
| Agentes de Windows | Sí | Se recopilan eventos de latido de agentes directos de Windows.|
| Grupo de administración de System Center Operations Manager | Sí | Se recopilan eventos de latido de agentes que informan al grupo de administración cada 60 segundos y después se reenvían a Azure Monitor. No se requiere ninguna conexión directa entre los agentes de Operations Manager y Azure Monitor. Los datos de eventos de latido se reenvían del grupo de administración al área de trabajo de Log Analytics.|

## <a name="using-the-solution"></a>Uso de la solución
Al agregar la solución al área de trabajo de Log Analytics, se agrega el icono **Agent Health** al panel. Este icono muestra el número total de agentes y el número de agentes que no responden en las últimas 24 horas.<br><br> ![Icono de la solución Agent Health en el panel](./media/solution-agenthealth/agenthealth-solution-tile-homepage.png)

Haga clic en el icono **Agent Health** para abrir el panel **Agent Health**.  El panel incluye las columnas de la tabla siguiente. Cada columna muestra los diez principales eventos por recuento que coinciden con los criterios de esa columna para el intervalo de tiempo especificado. Puede ejecutar una búsqueda de registros que proporcione toda la lista si selecciona **Ver todo** en la parte inferior derecha de la columna o hace clic en el encabezado de columna.

| Columna | Descripción |
|--------|-------------|
| Agent count over time (Número de agentes a lo largo del tiempo) | Una tendencia del número de agentes durante un período de siete días para agentes de Linux y Windows.|
| Count of unresponsive agents (Número de agentes que no responden) | Una lista de agentes que no ha enviado ningún latido en las últimas 24 horas.|
| Distribution by OS Type (Distribución por tipo de sistema operativo) | Una división de cuántos agentes de Windows y Linux tiene en su entorno.|
| Distribution by Agent Version (Distribución por versión del agente) | Una división de las diferentes versiones de agente instaladas en su entorno y el número de cada una de ellas.|
| Distribution by Agent Category (Distribución por categoría del agente) | Una división de las diferentes categorías de agentes que envían eventos de latido: agentes directos, agentes de OpsMgr o el servidor de administración de OpsMgr.|
| Distribution by Management Group (Distribución por grupo de administración) | División de los diferentes grupos de administración de Operations Manager en su entorno.|
| Geo-location of Agents (Geolocalización de los agentes) | División de los diferentes países o regiones donde tiene agentes y número total de agentes instalados en cada país o región.|
| Count of Gateways Installed (Número de puertas de enlace instaladas) | Número de servidores que tienen instalada la puerta de enlace de Log Analytics y una lista de estos servidores.|

![Ejemplo de panel de la solución Agent Health](./media/solution-agenthealth/agenthealth-solution-dashboard.png)  

## <a name="azure-monitor-log-records"></a>Registros de Azure Monitor
La solución crea un tipo de registro en el área de trabajo de Log Analytics.  

### <a name="heartbeat-records"></a>Registros de latidos
Se crea un registro del tipo **Heartbeat**.  Estos registros tienen las propiedades de la tabla siguiente.  

| Propiedad | Descripción |
| --- | --- |
| `Type` | *Heartbeat*|
| `Category` | El valor es *Direct Agent*, *SCOM Agent* o *SCOM Management Server*.|
| `Computer` | Nombre del equipo.|
| `OSType` | Sistema operativo Windows o Linux.|
| `OSMajorVersion` | Versión principal del sistema operativo.|
| `OSMinorVersion` | Versión secundaria del sistema operativo.|
| `Version` | Versión del agente de Log Analytics o de Operations Manager.|
| `SCAgentChannel` | El valor es *Direct* o *SCManagementServer*.|
| `IsGatewayInstalled` | Si la puerta de enlace de Log Analytics está instalada, el valor es *true*; en caso contrario, es *false*.|
| `ComputerIP` | Dirección IP pública del equipo. En máquinas virtuales de Azure, se mostrará la dirección IP pública si hay alguna disponible. En el caso de las máquinas virtuales que usan direcciones IP privadas, se mostrará la dirección SNAT de Azure (no la dirección IP privada). |
| `RemoteIPCountry` | Ubicación geográfica donde el equipo está implementado.|
| `ManagementGroupName` | Nombre del grupo de administración de Operations Manager.|
| `SourceComputerId` | Identificador único del equipo.|
| `RemoteIPLongitude` | Longitud de la ubicación geográfica del equipo.|
| `RemoteIPLatitude` | Latitud de la ubicación geográfica del equipo.|

Cada agente que informa a un servidor de administración de Operations Manager enviará dos latidos y el valor de la propiedad SCAgentChannel incluirá tanto **Direct** como **SCManagementServer**, en función de qué soluciones y orígenes de datos haya habilitado en su suscripción. Si recuerda, los datos de soluciones se envían directamente desde un servidor de administración de Operations Manager a Azure Monitor, o bien, debido al volumen de los datos recopilados en el agente, se envían directamente del agente a Azure Monitor. Para los eventos de latido que tienen el valor **SCManagementServer**, el valor de ComputerIP es la dirección IP del servidor de administración, ya que es el que realmente carga los datos.  Para los latidos donde SCAgentChannel esté establecido en **Direct**, es la dirección IP pública del agente.  

## <a name="sample-log-searches"></a>Búsquedas de registros de ejemplo
En la tabla siguiente se proporcionan búsquedas de registros de ejemplo para los registros recopilados por esta solución.

| Consultar | Descripción |
|:---|:---|
| Heartbeat &#124; distinct Computer |Número total de agentes |
| Heartbeat &#124; summarize LastCall = max(TimeGenerated) by Computer &#124; where LastCall < ago(24h) |Número de agentes que no responden en las últimas 24 horas |
| Heartbeat &#124; summarize LastCall = max(TimeGenerated) by Computer &#124; where LastCall < ago(15m) |Número de agentes que no responden en los últimos 15 minutos |
| Heartbeat &#124; where TimeGenerated > ago(24h) and Computer in ((Heartbeat &#124; where TimeGenerated > ago(24h) &#124; distinct Computer)) &#124; summarize LastCall = max(TimeGenerated) by Computer |Equipos en línea (en las últimas 24 horas) |
| Heartbeat &#124; where TimeGenerated > ago(24h) and Computer !in ((Heartbeat &#124; where TimeGenerated > ago(30m) &#124; distinct Computer)) &#124; summarize LastCall = max(TimeGenerated) by Computer |Total de agentes sin conexión en los últimos 30 minutos (para las últimas 24 horas) |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by OSType |Obtener una tendencia de número de agentes a lo largo del tiempo por tipo de sistema operativo|
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by OSType |Distribution by OS Type (Distribución por tipo de sistema operativo) |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by Version |Distribution by Agent Version (Distribución por versión del agente) |
| Heartbeat &#124; summarize AggregatedValue = count() by Category |Distribution by Agent Category (Distribución por categoría del agente) |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by ManagementGroupName | Distribution by Management Group (Distribución por grupo de administración) |
| Heartbeat &#124; summarize AggregatedValue = dcount(Computer) by RemoteIPCountry |Geo-location of Agents (Geolocalización de los agentes) |
| Heartbeat &#124; where iff(isnotnull(toint(IsGatewayInstalled)), IsGatewayInstalled == true, IsGatewayInstalled == "true") == true &#124; distinct Computer |Número de puertas de enlace de Log Analytics instaladas |




## <a name="next-steps"></a>Pasos siguientes

* Consulte las [información general sobre las alertas en Azure Monitor](../platform/alerts-overview.md) para obtener más detalles sobre la generación de alertas desde consultas de registro. 

