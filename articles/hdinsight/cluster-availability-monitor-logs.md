---
title: Supervisión de la disponibilidad de un clúster con los registros Azure Monitor en HDInsight
description: Aprenda a usar los registros de Azure Monitor para supervisar la disponibilidad y estado de un clúster.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 08/12/2020
ms.openlocfilehash: 19e3f1a157ee2c042dfebfc96c9b51c3c4698ebc
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88163737"
---
# <a name="how-to-monitor-cluster-availability-with-azure-monitor-logs-in-hdinsight"></a>Supervisión de la disponibilidad de un clúster con los registros Azure Monitor en HDInsight

Los clústeres de HDInsight incluyen la integración de registros de Azure Monitor, lo que proporciona registros y métricas que se pueden consultar, así como alertas configurables. En este artículo se muestra cómo usar Azure Monitor para supervisar un clúster.

## <a name="azure-monitor-logs-integration"></a>Integración de los registros de Azure Monitor

Los registros de Azure Monitor permiten la recolección de los datos generados por varios recursos, como clústeres de HDInsight, y su agregación en un solo lugar para lograr una experiencia de supervisión unificada.

Como requisito previo, necesitará un área de trabajo de Log Analytics para almacenar los datos recopilados. Si aún no ha creado una, puede seguir estas instrucciones: [Crear un área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace).

## <a name="enable-hdinsight-azure-monitor-logs-integration"></a>Habilitar la integración de registros de Azure Monitor de HDInsight

En la página de recursos del clúster de HDInsight, en el portal, seleccione **Azure Monitor**. Luego, seleccione **Habilitar** y seleccione el área de trabajo de Log Analytics de la lista desplegable.

![HDInsight Operations Management Suite](media/cluster-availability-monitor-logs/azure-portal-monitoring.png)

De forma predeterminada, esto instala el agente de OMS en todos los nodos del clúster, excepto en los nodos perimetrales. Dado que no se instala ningún agente de OMS en los nodos perimetrales del clúster, no hay telemetría en los nodos perimetrales presentes en Log Analytics de forma predeterminada.

## <a name="query-metrics-and-logs-tables"></a>Consultar tablas de métricas y registros

Cuando se haya habilitado la integración de registros de Azure Monitor (esto puede tardar unos minutos), vaya al recurso **Área de trabajo de Log Analytics** y seleccione **Registros**.

![Registros del área de trabajo de Log Analytics](media/cluster-availability-monitor-logs/hdinsight-portal-logs.png)

La hoja Registros muestra un número de consultas de ejemplo, como:

| Nombre de la consulta                      | Descripción                                                               |
|---------------------------------|---------------------------------------------------------------------------|
| Disponibilidad de los equipos hoy en día    | Crear un gráfico del número de equipos que envían registros, cada hora                     |
| Enumerar latidos                 | Lista de todos los latidos del equipo de la última hora                           |
| Último latido de cada equipo | Mostrar el último latido enviado por cada equipo                             |
| Equipos desconectados           | Lista de todos los equipos conocidos que no enviaron un latido en las últimas 5 horas |
| Índice de disponibilidad               | Calcular el índice de disponibilidad de cada equipo conectado                |

Por ejemplo, ejecute la consulta de ejemplo **Índice de disponibilidad**. Para ello, seleccione **Ejecutar** en esa consulta, tal como se muestra en la captura de pantalla anterior. Esto mostrará el índice de disponibilidad de cada nodo del clúster como un porcentaje. Si habilitó varios clústeres de HDInsight para enviar métricas a la misma área de trabajo de Log Analytics, verá el índice de disponibilidad para todos los nodos (salvo los perimetrales) en los clústeres que se muestran.

![Consulta de ejemplo de "índice de disponibilidad" de los registros del área de trabajo de Log Analytics](media/cluster-availability-monitor-logs/portal-availability-rate.png)

> [!NOTE]  
> El índice de disponibilidad se mide en un período de 24 horas, por lo que el clúster tendrá que ejecutarse durante al menos 24 horas antes de poder ver índice de disponibilidad precisos.

Puede anclar esta tabla a un panel compartido al hacer clic en **Anclar** de la esquina superior derecha. Si no tiene ningún panel compartido en el que se puede escribir, puede ver cómo crear uno aquí: [Creación y uso compartido de paneles en Azure Portal](https://docs.microsoft.com/azure/azure-portal/azure-portal-dashboards#publish-and-share-a-dashboard).

## <a name="azure-monitor-alerts"></a>Alertas de Azure Monitor

También puede configurar alertas de Azure Monitor que se desencadenarán cuando el valor de una métrica o los resultados de una consulta cumplan ciertas condiciones. Por ejemplo, vamos a crear una alerta para enviar un correo electrónico cuando uno o varios nodos no hayan enviado un latido en 5 horas (es decir, se supone que no está disponible).

En la hoja **Registros**, ejecute la consulta de ejemplo **Equipos no disponibles**. Para ello, seleccione **Ejecutar** en esa consulta, tal como se muestra a continuación.

![Ejemplo de registros de "equipos no disponibles" del área de trabajo de Log Analytics](media/cluster-availability-monitor-logs/portal-unavailable-computers.png)

Si todos los nodos están disponibles, esta consulta debería devolver 0 resultados por ahora. Haga clic en **Nueva regla de alertas** para comenzar a configurar la alerta para esta consulta.

![Nueva regla de alertas del área de trabajo de Log Analytics](media/cluster-availability-monitor-logs/portal-logs-new-alert-rule.png)

Una alerta tiene tres componentes: el *recursos* para el que crear la regla (en este caso, el área de Log Analytics), la *condición* para desencadenar la alerta y los *grupos de acciones*  que determinan lo que sucederá cuando se desencadene la alerta.
Haga clic en el **título de la condición**, tal como se muestra abajo, para finalizar la configuración de la lógica de señal.

![Condición de regla de creación de alertas del portal](media/cluster-availability-monitor-logs/portal-condition-title.png)

Se abrirá **Configurar lógica de señal**.

Establezca la sección **Lógica de alerta** tal como se indica abajo:

*Basado en: número de resultados, Condición: Mayor que, Umbral: 0.*

Dado que esta consulta devuelve solo nodos no disponibles como resultado, si el número de resultados es mayor que 0, se debería desencadenar la alerta.

En la sección **Se evaluó basándose en**, establezca el **período** y la **frecuencia** según la frecuencia con la que desea comprobar si hay nodos no disponibles.

Para esta alerta, debería asegurarse de que **Período=Frecuencia**. Puede encontrar más información sobre el período, la frecuencia y otros parámetros de alerta [aquí](https://docs.microsoft.com/azure/azure-monitor/platform/alerts-unified-log#log-search-alert-rule---definition-and-types).

Seleccione **Listo** cuando haya terminado de configurar la lógica de señal.

![Configuración de la lógica de señal en la regla de alertas](media/cluster-availability-monitor-logs/portal-configure-signal-logic.png)

Si no dispone de un grupo de acciones existente, haga clic en **Crear nuevo** en la sección **Grupos de acciones**.

![Creación de un nuevo grupo de acciones en la regla de alertas](media/cluster-availability-monitor-logs/portal-create-new-action-group.png)

Se abrirá **Agregar grupo de acciones**. Elija un valor de **Nombre del grupo de acciones**, **Nombre corto**, **Suscripción** y **Grupo de recursos**. En la sección **Acciones**, elija un valor de **Nombre de acción** y seleccione **Correo electrónico/SMS/Inserción/Voz** como el valor de **Tipo de acción.**

> [!NOTE]
> Hay muchas otras acciones que una alerta puede desencadenar además de correo electrónico, SMS, inserción o voz, como una función de Azure, LogicApp, Webhook, ITSM y Automation Runbook. [Más información](https://docs.microsoft.com/azure/azure-monitor/platform/action-groups#action-specific-information).

Se abrirá **Correo electrónico/SMS/Inserción/Voz**. Elija un valor de **Nombre** para el destinatario, **active** la casilla **Correo electrónico** y escriba una dirección de correo electrónico a la que enviar la alerta. Seleccione **Aceptar** en **Correo electrónico/SMS/Inserción/Voz** y luego en **Agregar grupo de acciones** para terminar de configurar el grupo de acciones.

![Incorporación de un grupo de acciones en la regla de alertas](media/cluster-availability-monitor-logs/portal-add-action-group.png)

Después de que se cierren estas hojas, debería ver el grupo de acciones en la sección **Grupos de acciones**. Por último, complete la sección **Detalles de alerta**. Para ello, escriba un valor en **Nombre de la regla de alertas** y en **Descripción**, y luego elija **Gravedad**. Haga clic en **Crear regla de alerta** para finalizar.

![Finalización después de la creación de una regla de alertas del portal](media/cluster-availability-monitor-logs/portal-create-alert-rule-finish.png)

> [!TIP]
> La capacidad de especificar el valor de **Gravedad** es una herramienta eficaz que puede usarse al crear varias alertas. Por ejemplo, podría crear una alerta para generar una advertencia (gravedad 1) si un único nodo principal se queda inactivo y otra alerta crítica (gravedad 0) en el improbable caso de que ambos nodos principales dejen de funcionar.

Cuando se cumple la condición de la alerta, esta última se desencadena y recibirá un correo electrónico con los detalles de la misma, que tendrá el aspecto siguiente:

![Ejemplo de correo electrónico de alerta de Azure Monitor](media/cluster-availability-monitor-logs/portal-oms-alert-email.png)

También puede ver todas las alertas desencadenadas, agrupadas por gravedad. Para ello, vaya a **Alertas** en su **Área de trabajo de Log Analytics**.

![Alertas del área de trabajo de Log Analytics](media/cluster-availability-monitor-logs/hdi-portal-oms-alerts.png)

Al seleccionar una agrupación de gravedad (por ejemplo, las de **Gravedad 1** tal como se ha mencionado anteriormente), se mostrarán los registros de todas las alertas de esta gravedad que se hayan desencadenado, como se muestra a continuación:

![Alertas de gravedad 1 del área de trabajo de Log Analytics](media/cluster-availability-monitor-logs/portal-oms-alerts-sev1.png)

## <a name="next-steps"></a>Pasos siguientes

* [Disponibilidad del clúster: Apache Ambari](./hdinsight-cluster-availability.md)
* [Uso de registros de Azure Monitor](hdinsight-hadoop-oms-log-analytics-tutorial.md)
