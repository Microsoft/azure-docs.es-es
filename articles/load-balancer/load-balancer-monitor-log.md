---
title: Supervisión de operaciones, eventos y contadores para una instancia pública de Load Balancer
titleSuffix: Azure Load Balancer
description: Aprenda como habilitar eventos de alerta y el registro del estado del sondeo de estado una instancia pública de Basic Load Balancer
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/05/2020
ms.author: allensu
ms.openlocfilehash: 42ec5a661bd7b42ba5de5bfa99b3898291cc60fa
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935609"
---
# <a name="azure-monitor-logs-for-public-basic-load-balancer"></a>Registros de Azure Monitor para el equilibrador de carga básica público

Puede usar diferentes tipos de registros en Azure para administrar y solucionar problemas de Basic Load Balancer. Se puede acceder a algunos de estos registros a través del portal. Los registros se pueden transmitir a un centro de eventos o a un área de trabajo de Log Analytics. Se pueden extraer todos los registros desde Azure Blob Storage y visualizarse en distintas herramientas, como Excel y Power BI.  Puede obtener más información acerca de los diferentes tipos de registros en la lista siguiente.

* **Registros de actividad:** Puede usar [Visualización de registros de actividad para supervisar acciones sobre recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit) para ver toda la actividad que se está enviado a las suscripciones de Azure y sus estados. Los registros de actividad están habilitados de manera predeterminada y se pueden ver en Azure Portal.
* **Registro de eventos de alerta:** puede utilizar este registro para las alertas generadas por el equilibrador de carga. El estado del equilibrador de carga se recopila cada cinco minutos. Este registro se escribe solo si se produce un evento de alerta del equilibrador de carga.
* **Registro de sondeo de estado:** puede utilizar este registro para ver los problemas detectados por el sondeo de estado, como el número de instancias en el grupo back-end que no reciben las solicitudes del equilibrador de carga debido a errores de sondeo de estado. Este registro se escribe cuando se produce un cambio en el estatus del sondeo de estado.

> [!IMPORTANT]
> **Los registros de eventos de sondeo de estado no funcionan en la actualidad y aparecen listados en los [problemas conocidos de Azure Load Balancer](whats-new.md#known-issues).** Los registros solo están disponibles para los recursos implementados en el modelo de implementación del Administrador de recursos. No puede usar los registros de recursos del modelo de implementación clásica. Para más información sobre estos modelos de implementación, consulte [Understanding Resource Manager deployment and classic deployment](../azure-resource-manager/management/deployment-models.md) (Descripción de la implementación de Resource Manager y la implementación clásica).

## <a name="enable-logging"></a>Habilitar registro

El registro de actividades se habilita automáticamente para todos los recursos de Resource Manager. Habilite el registro de eventos y de sondeos de estado para iniciar la recopilación de los datos disponibles a través de esos registros. Para habilitar el registro, realice los siguientes pasos.

Inicie sesión en [Azure Portal](https://portal.azure.com). Si aún no tiene un equilibrador de carga, [cree uno](https://docs.microsoft.com/azure/load-balancer/quickstart-create-basic-load-balancer-portal) antes de continuar.

1. En el portal, haga clic en **Grupos de recursos**.
2. Seleccione el grupo de recursos **\<resource-group-name>** donde esté el equilibrador de carga.
3. Seleccione el equilibrador de carga.
4. Seleccione **Registro de actividades** > **Configuración de diagnóstico**.
5. En el panel **Configuración de diagnóstico**, en **Configuración de diagnóstico**, seleccione **+ Agregar configuración de diagnóstico**.
6. En el panel de creación **Configuración de diagnóstico**, escriba **myLBDiagnostics** en el campo **Nombre**.
7. Tiene tres opciones para la **Configuración de diagnóstico**.  Puede elegir una, dos o las tres y configurar cada una de ellas según sus requisitos:
   * **Archivar en una cuenta de almacenamiento**
   * **Transmisión a un centro de eventos**
   * **Enviar a Log Analytics**

    ### <a name="archive-to-a-storage-account"></a>Archivar en una cuenta de almacenamiento
    Necesitará una cuenta de almacenamiento ya creada para este proceso.  Para crear una cuenta de almacenamiento, consulte [Creación de una cuenta de almacenamiento](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal).

    1. Active la casilla junto a **Archivar en una cuenta de almacenamiento**.
    2. Seleccione **Configurar** para abrir el panel **Selección de una cuenta de almacenamiento**.
    3. Seleccione la **Suscripción** donde se creó la cuenta de almacenamiento en el cuadro desplegable.
    4. Seleccione el nombre de la cuenta de almacenamiento en **Cuenta de almacenamiento** en el cuadro desplegable.
    5. Seleccione Aceptar.

    ### <a name="stream-to-an-event-hub"></a>Transmitir a un centro de eventos
    Necesitará un centro de eventos ya creado para este proceso.  Para crear un centro de eventos, consulte [Inicio rápido: Creación de un centro de eventos mediante Azure Portal](https://docs.microsoft.com/azure/event-hubs/event-hubs-create).

    1. Active la casilla situada junto a **Transmitir a un centro de eventos**.
    2. Seleccione **Configurar** para abrir el panel **Select event hub** (Selección del centro de eventos).
    3. Seleccione la **Suscripción** donde se creó el centro de eventos en el cuadro desplegable.
    4. **Seleccione el espacio de nombres del centro de eventos** en el cuadro desplegable.
    5. **Seleccione el nombre de la directiva del centro de eventos** en el cuadro desplegable.
    6. Seleccione Aceptar.

    ### <a name="send-to-log-analytics"></a>Enviar a Log Analytics
    Deberá tener un área de trabajo de Log Analytics creada y configurada para este proceso.  Para crear un área de trabajo de Log Analytics, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace).

    1. Active la casilla junto a **Enviar a Log Analytics**.
    2. Seleccione la **Suscripción** donde se encuentra el área de trabajo de Log Analytics en el cuadro desplegable.
    3. Seleccione el **área de trabajo de Log Analytics** en el cuadro desplegable.


8. En la sección **LOG** del panel **Configuración de diagnóstico**, active la casilla junto a:
   * **LoadBalancerAlertEvent**
   * **LoadBalancerProbeHealthStatus**

9.  En la sección **METRIC** del panel **Configuración de diagnóstico**, active la casilla junto a:
   * **AllMetrics**

11. Compruebe que todo esté correcto y haga clic en **Guardar** en la parte superior del panel de creación de **Configuración de diagnóstico**.

## <a name="activity-log"></a>Registro de actividades

El registro de actividad se genera de manera predeterminada. Los registros se conservan durante 90 días en el almacén de registros de eventos de Azure. Obtenga más información sobre estos registros en el artículo [Visualización de registros de actividad para supervisar acciones sobre recursos](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-audit).

## <a name="archive-to-storage-account-logs"></a>Archivado en los registros de la cuenta de almacenamiento

### <a name="alert-event-log"></a>Registro de eventos de alerta

Este registro solo se genera si lo habilitó para cada uno de los equilibradores de carga. Los eventos se registran en formato JSON y se almacenan en la cuenta de almacenamiento que especificó cuando habilitó el registro. El ejemplo siguiente es de un evento.

```json
{
    "time": "2016-01-26T10:37:46.6024215Z",
    "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
    "category": "LoadBalancerAlertEvent",
    "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
    "operationName": "LoadBalancerProbeHealthStatus",
    "properties": {
        "eventName": "Resource Limits Hit",
        "eventDescription": "Ports exhausted",
        "eventProperties": {
            "public ip address": "40.117.227.32"
        }
    }
}
```

El resultado de JSON muestra la propiedad *eventname* que describirá el motivo de creación de una alerta por parte del equilibrador de carga. En este caso, la alerta generada se debió al agotamiento de puertos TCP causado por los límites de IP NAT de origen (SNAT).

### <a name="health-probe-log"></a>Registro de sondeo de estado

Este registro solo se genera si lo habilitó para cada uno de los equilibradores de carga, tal como se indicó anteriormente. Los datos se almacenan en la cuenta de almacenamiento que especificó cuando habilitó el registro. Se crea un contenedor denominado "insights-logs-loadbalancerprobehealthstatus" y se registran los datos siguientes:

```json
{
    "records":[
    {
        "time": "2016-01-26T10:37:46.6024215Z",
        "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
        "category": "LoadBalancerProbeHealthStatus",
        "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
        "operationName": "LoadBalancerProbeHealthStatus",
        "properties": {
            "publicIpAddress": "40.83.190.158",
            "port": "81",
            "totalDipCount": 2,
            "dipDownCount": 1,
            "healthPercentage": 50.000000
        }
    },
    {
        "time": "2016-01-26T10:37:46.6024215Z",
        "systemId": "32077926-b9c4-42fb-94c1-762e528b5b27",
        "category": "LoadBalancerProbeHealthStatus",
        "resourceId": "/SUBSCRIPTIONS/XXXXXXXXXXXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXX/RESOURCEGROUPS/RG7/PROVIDERS/MICROSOFT.NETWORK/LOADBALANCERS/WWEBLB",
        "operationName": "LoadBalancerProbeHealthStatus",
        "properties": {
            "publicIpAddress": "40.83.190.158",
            "port": "81",
            "totalDipCount": 2,
            "dipDownCount": 0,
            "healthPercentage": 100.000000
        }
    }]
}
```

El resultado JSON muestra en el campo de propiedades la información básica del estado de mantenimiento del sondeo. La propiedad *dipDownCount* muestra el número total de instancias en el back-end que no están recibiendo tráfico de red debido a las respuestas de sondeo con error.

### <a name="view-and-analyze-the-activity-log"></a>Visualización y análisis del registro de actividades

Puede ver y analizar los datos del registro de actividades mediante el uso de cualquiera de los métodos siguientes:

* **Herramientas de Azure:** puede recuperar información de los registros de actividad mediante Azure PowerShell, la interfaz de la línea de comandos (CLI) de Azure, la API REST de Azure o Azure Portal. En el artículo [Operaciones de auditoría con el Administrador de recursos](../azure-resource-manager/management/view-activity-logs.md) se detallan instrucciones paso a paso de cada método.
* **Power BI:** si todavía no tiene una cuenta de [Power BI](https:// .microsoft.com/pricing), puede probarlo gratis. Con el [paquete de contenido de los registros de auditoría de Azure para Power BI](https:// .microsoft.com/documentation/ -content-pack-azure-audit-logs) puede analizar los datos con los paneles preconfigurados o puede personalizar las vistas para que se adapten a sus necesidades.

### <a name="view-and-analyze-the-health-probe-and-event-log"></a>Visualización y análisis del registro de eventos y de sondeos de estado

Conéctese a la cuenta de almacenamiento y recupere las entradas del registro JSON para los registros de eventos y de sondeos de estado. Cuando descargue los archivos JSON, se pueden convertir a CSV y consultarlos en Excel, Power BI o cualquier otra herramienta de visualización de datos.

> [!TIP]
> Si está familiarizado con Visual Studio y con los conceptos básicos de cambio de los valores de constantes y variables de C#, puede usar las [herramientas convertidoras de registros](https://github.com/Azure-Samples/networking-dotnet-log-converter) que encontrará en GitHub.

## <a name="stream-to-an-event-hub"></a>Transmitir a un centro de eventos
Cuando se transmite información de diagnóstico a un centro de eventos, se puede usar para el análisis de registros centralizado en una herramienta SIEM de terceros con la integración de Azure Monitor. Para más información, consulte [Transmisión de datos de supervisión de Azure a un centro de eventos](../azure-monitor/platform/stream-monitoring-data-event-hubs.md#partner-tools-with-azure-monitor-integration).

## <a name="send-to-log-analytics"></a>Enviar a Log Analytics
Los recursos de Azure pueden hacer que su información de diagnóstico se envíe directamente a un área de trabajo de Log Analytics, donde se pueden ejecutar consultas complejas en la información para la solución de problemas y el análisis.  Para más información, consulte [Recopilación de registros de recursos de Azure en el área de trabajo de Log Analytics en Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/platform/resource-logs-collect-workspace)

## <a name="next-steps"></a>Pasos siguientes

[Descripción de los sondeos del equilibrador de carga](load-balancer-custom-probe-overview.md)
