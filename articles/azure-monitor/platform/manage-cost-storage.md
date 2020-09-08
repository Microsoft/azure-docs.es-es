---
title: Administración del uso y los costos con los registros de Azure Monitor
description: Aprenda cómo cambiar el plan de precios y cómo administrar el volumen de datos y la directiva de retención para el área de trabajo de Log Analytics en Azure Monitor.
services: azure-monitor
documentationcenter: azure-monitor
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: bwren
ms.subservice: ''
ms.openlocfilehash: 84a5b1cd7b2229defd4e38a227f75cfbf9ebdd95
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88933671"
---
# <a name="manage-usage-and-costs-with-azure-monitor-logs"></a>Administrar el uso y los costos con los registros de Azure Monitor    

> [!NOTE]
> En este artículo se describe cómo entender y controlar los costos de los registros de Azure Monitor. En un artículo relacionado, [Supervisión del uso y costos estimados](usage-estimated-costs.md), se describe cómo ver el uso y los costos estimados mediante varias características de supervisión de Azure para los distintos modelos de precios. Todos los precios y costos que se muestran en este artículo son solo para fines de ejemplo. 

Los registros de Azure Monitor están diseñados para escalar y admitir la recopilación, indexación y almacenamiento de grandes cantidades de datos por día provenientes de cualquier origen dentro de su empresa o implementados en Azure.  Aunque esto puede ser un factor clave principal para su organización, la rentabilidad es en última instancia el factor clave subyacente. Con este fin, es importante comprender que el costo de un área de trabajo de Log Analytics no se basa simplemente en el volumen de datos recopilados; también depende del plan seleccionado y de cuánto tiempo se decida almacenar los datos generados a partir de los orígenes conectados.  

En este artículo se revisa cómo supervisar de forma activa el crecimiento del volumen y el almacenamiento de los datos ingeridos, y definir los límites para controlar esos costos asociados. 

## <a name="pricing-model"></a>Modelo de precios

Los precios predeterminados de Log Analytics son de un modelo de **Pago por uso**, que se basa en el volumen de datos ingeridos y, opcionalmente, permite obtener una mayor retención de datos. El volumen de datos se mide como el tamaño de los datos que se almacenarán, en GB (10^9 bytes). Cada área de trabajo de Log Analytics se cobra como un servicio independiente y contribuye a la factura de la suscripción a Azure. La cantidad de ingesta de datos puede ser considerable dependiendo de los factores siguientes: 

  - Número de soluciones de administración habilitadas y su configuración
  - Número de máquinas virtuales supervisadas
  - Tipo de datos recopilados de cada máquina virtual supervisada 
  
Además del modelo de pago por uso, Log Analytics tiene niveles de **Reserva de capacidad** que le permiten ahorrar hasta un 25 % en comparación con el precio de pago por uso. Los precios de la reserva de capacidad permiten comprar una reserva a partir de 100 GB/día. Cualquier uso por encima del nivel de reserva se facturará según la tarifa de pago por uso. Los niveles de Reserva de capacidad tienen un período de compromiso de 31 días. Durante el período de compromiso, puede cambiar a un nivel de Reserva de capacidad de nivel superior (que reiniciará el período de compromiso de 31 días), pero no podrá volver a la versión de pago por uso o a un nivel de Reserva de capacidad inferior hasta que el período de compromiso finalice. La facturación de los niveles de reserva de capacidad se realiza cada día. [Más información](https://azure.microsoft.com/pricing/details/monitor/) sobre los precios de Pago por uso y de Reserva de capacidad de Log Analytics. 

En todos los planes de tarifa, el tamaño de los datos de un evento se calcula a partir de una representación de cadena de las propiedades que se almacenan en Log Analytics para ese evento, tanto si los datos se envían desde un agente como si se agregan durante el proceso de ingesta. Esto incluye cualquier [campo personalizado](custom-fields.md) que se agregue a medida que se recopilen datos y luego se almacenen en Log Analytics. Varias propiedades comunes a todos los tipos de datos, incluidas algunas [propiedades estándar de Log Analytics](log-standard-properties.md), se excluyen del cálculo del tamaño del evento. Esto incluye `_ResourceId`, `_ItemId`, `_IsBillable`, `_BilledSize` y `Type`. Todas las demás propiedades almacenadas en Log Analytics se incluyen en el cálculo del tamaño del evento. Algunos tipos de datos están libres de los cargos de ingesta de datos, por ejemplo, los tipos AzureActivity, Latido y Uso. Para determinar si un evento se ha excluido de la facturación relacionada con la ingesta de datos, puede usar la propiedad `_IsBillable`, como se muestra [más adelante](#data-volume-for-specific-events). El uso se informa en GB (1,0E9 bytes). 

Además, tenga en cuenta que algunas soluciones, como [Azure Security Center](https://azure.microsoft.com/pricing/details/security-center/), [Azure Sentinel](https://azure.microsoft.com/pricing/details/azure-sentinel/) y [Administración de configuración](https://azure.microsoft.com/pricing/details/automation/) tienen sus propios modelos de precios. 

### <a name="log-analytics-dedicated-clusters"></a>Clústeres dedicados de Log Analytics

Los clústeres dedicados de Log Analytics son colecciones de áreas de trabajo en un único clúster de Azure Data Explorer administrado para permitir escenarios avanzados, como las [claves administradas por el cliente](customer-managed-keys.md).  Los clústeres dedicados de Log Analytics solo admiten un modelo de precios de reserva de capacidad a partir de 1000 GB/día con un descuento del 25 % en comparación con los precios de pago por uso. Cualquier uso por encima del nivel de reserva se facturará según la tarifa de pago por uso. La reserva de capacidad del clúster tiene un período de compromiso de 31 días después de aumentar el nivel de reserva. Durante el período de compromiso, no se puede reducir el nivel de reserva de capacidad, pero se puede aumentar en cualquier momento. Más información sobre la [creación de clústeres de Log Analytics](customer-managed-keys.md#create-cluster-resource) y la [asociación de áreas de trabajo a ellos](customer-managed-keys.md#workspace-association-to-cluster-resource).  

El nivel de reserva de capacidad del clúster se configura mediante programación con Azure Resource Manager con el parámetro `Capacity` en `Sku`. El parámetro `Capacity` se especifica en unidades de GB y puede tener valores de 1000 GB/día o más en incrementos de 100 GB/día. Esto se detalla en [Clave administrada por el cliente de Azure Monitor](customer-managed-keys.md#create-cluster-resource). Si el clúster necesita una reserva superior a 2000 GB/día, póngase en contacto con nosotros en [LAIngestionRate@microsoft.com](mailto:LAIngestionRate@microsoft.com).

Hay dos modos de facturación para el uso en un clúster. Estos pueden especificarse mediante el parámetro `billingType` al [configurar el clúster](customer-managed-keys.md#cmk-management). Los dos modos son: 

1. **Clúster**: en este caso (que es el modo predeterminado), la facturación de los datos ingeridos se realiza en el nivel de clúster. Las cantidades de datos ingeridas desde cada área de trabajo asociada a un clúster se suman para calcular la factura diaria del clúster. Tenga en cuenta las asignaciones por nodo de [Azure Security Center](https://docs.microsoft.com/azure/security-center/) que se aplican en el nivel de área de trabajo antes de esta agregación de datos agregados en todas las áreas de trabajo del clúster. 

2. **Áreas de trabajo**: los costos de reserva de capacidad para el clúster se atribuyen proporcionalmente a las áreas de trabajo del clúster (después de tener en cuenta las asignaciones por nodo de [Azure Security Center](https://docs.microsoft.com/azure/security-center/) para cada área de trabajo). Si el volumen total de datos ingeridos en un área de trabajo durante un día es menor que la reserva de capacidad, cada área de trabajo se factura por sus datos ingeridos con la tasa de reserva de capacidad por GB efectiva y la facturación de una fracción de la reserva de capacidad. La parte sin usar de la reserva de capacidad se factura al recurso de clúster. Si el volumen total de datos ingeridos en un área de trabajo durante un día es superior al de la reserva de capacidad, se factura a cada área de trabajo una fracción de la reserva de capacidad en función de su fracción de la cantidad de datos ingeridos ese día, y también una fracción de los datos ingeridos por encima de la reserva de capacidad. No se factura nada al recurso de clúster si el volumen total de datos ingeridos en un área de trabajo durante un día supera la reserva de capacidad.

En las opciones de facturación del clúster, la retención de datos se factura en el nivel del área de trabajo. Tenga en cuenta que la facturación del clúster comienza cuando se crea este, independientemente de si las áreas de trabajo se han asociado al clúster. Además, tenga en cuenta que las áreas de trabajo asociadas a un clúster ya no tienen un plan de tarifa.

## <a name="estimating-the-costs-to-manage-your-environment"></a>Estimación de los costos de administración del entorno 

Si aún no usa los registros de Azure Monitor, puede usar la [calculadora de precios de Azure Monitor](https://azure.microsoft.com/pricing/calculator/?service=monitor) para calcular el costo del uso de Log Analytics. Comience por escribir "Azure Monitor" en el cuadro de búsqueda y haga clic en el icono de Azure Monitor resultante. Desplácese hacia abajo en la página hasta Azure Monitor y seleccione Log Analytics en la lista desplegable Tipo.  Aquí puede especificar el número de máquinas virtuales y los GB de datos que espera recopilar de cada máquina virtual. Normalmente se ingieren de 1 a 3 GB de datos al mes en una máquina virtual típica de Azure. Si ya está evaluando los registros de Azure Monitor, puede usar las estadísticas de datos de su propio entorno. Consulte a continuación cómo determinar el [número de máquinas virtuales supervisadas](#understanding-nodes-sending-data) y el [volumen de los datos que ingiere el área de trabajo](#understanding-ingested-data-volume). 

## <a name="understand-your-usage-and-estimate-costs"></a>Información útil del uso y los costos estimados

Si ya usa los registros de Azure Monitor ahora, es fácil comprender cuáles serán probablemente los costos según los patrones de uso recientes. Para ello, utilice **Uso y costos estimados de Log Analytics** a fin de revisar y analizar el uso de datos. Muestra la cantidad de datos que recopila cada solución, la cantidad de datos que se retienen y una estimación de los costos según la cantidad de datos ingeridos y cualquier retención adicional más allá de la cantidad incluida.

![Uso y costos estimados](media/manage-cost-storage/usage-estimated-cost-dashboard-01.png)

Para explorar los datos más detalladamente, haga clic en el icono situado en el extremo superior derecho de cualquiera de los gráficos de la página **Uso y costos estimados**. Ahora puede trabajar con esta consulta para explorar más detalles sobre su uso.  

![Visualización de registros](media/manage-cost-storage/logs.png)

En la página **Uso y costos estimados** puede revisar el volumen de datos del mes. Esto incluye todos los datos recibidos y retenidos facturables en el área de trabajo de Log Analytics.  
 
Los cargos de Log Analytics se agregarán a la factura de Azure. Puede consultar los detalles de su factura de Azure en la sección de facturación de Azure Portal o en el [Portal de facturación de Azure](https://account.windowsazure.com/Subscriptions).  

## <a name="viewing-log-analytics-usage-on-your-azure-bill"></a>Visualización del uso de Log Analytics en la factura de Azure 

Azure proporciona una gran cantidad de funcionalidades útiles en el centro [Azure Cost Management + Facturación](https://docs.microsoft.com/azure/cost-management/quick-acm-cost-analysis?toc=/azure/billing/TOC.json). Por ejemplo, la funcionalidad de "Análisis de costos" le permite ver los gastos de los recursos de Azure. En primer lugar, agregar un filtro por "tipo de recurso" (a microsoft.operationalinsights/workspace para Log Analytics y microsoft.operationalinsights/workspace para los clústeres de Log Analytics) le permitirá realizar un seguimiento del gasto de Log Analytics. Después, para "Agrupar por", seleccione "Categoría del medidor" o "Medidor".  Tenga en cuenta que otros servicios, como Azure Security Center y Azure Sentinel, también se facturan por uso frente a los recursos del área de trabajo de Log Analytics. Para ver la asignación al nombre del servicio, puede seleccionar la vista de tabla en lugar de ver un gráfico. 

Puede obtener información sobre el uso mediante la [descarga del uso desde Azure Portal](https://docs.microsoft.com/azure/billing/billing-download-azure-invoice-daily-usage-date#download-usage-in-azure-portal). En la hoja de cálculo descargada puede ver el uso por recurso de Azure (por ejemplo, área de trabajo de Log Analytics) al día. En esta hoja de cálculo de Excel, el uso de las áreas de trabajo de Log Analytics se puede encontrar filtrando, en primer lugar, la columna "Categoría del medidor" para mostrar "Log Analytics", "Insights y Analytics" (utilizado por algunos planes de tarifa heredados) y "Azure Monitor" (usado por los planes de tarifa de reserva de capacidad) y, a continuación, agregando un filtro en la columna "Id. de instancia", que es "contiene área de trabajo" o "contiene clúster" (este último para incluir el uso de clústeres de Log Analytics). El uso se muestra en la columna "Cantidad consumida" y la unidad de cada entrada se muestra en la columna "Unidad de medida".  Hay más detalles disponibles para ayudarle a [entender la factura de Microsoft Azure](https://docs.microsoft.com/azure/billing/billing-understand-your-bill). 

## <a name="changing-pricing-tier"></a>Actualización del plan de tarifa

Para cambiar el plan de tarifa de Log Analytics del área de trabajo, 

1. en Azure Portal, abra **Uso y costos estimados** en el área de trabajo donde verá una lista de cada uno de los planes de tarifa disponibles para esta área de trabajo.

2. Revise los costos estimados para cada uno de los planes de tarifa. Esta estimación se basa en los últimos 31 días de uso, por lo que esta estimación de costos se basa en los últimos 31 días lo que es representativo de su uso habitual. En el ejemplo siguiente puede ver cómo, en función de los patrones de datos de los últimos 31 días, esta área de trabajo costaría menos en el nivel de pago por uso (n.° 1) en comparación con el nivel de Reserva de capacidad de 100 GB/día (n.° 2).  

    ![Planes de tarifa](media/manage-cost-storage/pricing-tier-estimated-costs.png)

3. Después de revisar los costos estimados en función de los últimos 31 días de uso, si decide cambiar el plan de tarifa, haga clic en **Seleccionar**.  

También puede [establecer el plan de tarifa mediante Azure Resource Manager](template-workspace-configuration.md#configure-a-log-analytics-workspace) con el parámetro `sku` (`pricingTier` en la plantilla de Azure Resource Manager). 

## <a name="legacy-pricing-tiers"></a>Planes de tarifa heredados

Las suscripciones que contenían un área de trabajo de Log Analytics o un recurso de Application Insights antes del 2 de abril de 2018, o que están vinculadas a un Contrato Enterprise que comenzó antes del 1 de febrero de 2019, continuarán teniendo acceso a los planes de tarifa heredados: **Gratuito**,**Independiente (por GB)** y **Por nodo (OMS)** .  Las áreas de trabajo en el plan de tarifa Gratis tendrán una ingesta diaria de datos limitada a 500 MB (excepto los tipos de datos de seguridad que recopile [Azure Security Center](https://docs.microsoft.com/azure/security-center/)) y la retención de datos se limitará a 7 días. El plan de tarifa gratuito está destinado solo para fines de evaluación. Las áreas de trabajo en los planes de tarifa independientes o por nodo tienen una retención configurable para el usuario de 30 a 730 días.

El uso en el plan de tarifa independiente se factura por el volumen de datos ingerido. Se indica en el servicio **Log Analytics** y el medidor se denomina "Datos analizados". 

Los cargos del plan de tarifa por nodo y por VM supervisada (nodo) en una granularidad de hora. En cada nodo supervisado, se asignan al área de trabajo 500 MB de datos al día que no se facturan. Esta asignación se agrega en el nivel de área de trabajo. Los datos ingeridos por encima de la asignación de datos diaria agregada se facturan por GB como datos con un uso por encima del límite. Tenga en cuenta que, en su factura, el servicio será **Insight and Analytics** para el uso de Log Analytics si el área de trabajo se encuentra en el plan de tarifa por nodo. El uso se muestra en tres medidores:

1. Nodo: se usa para el número de nodos supervisados (VM) en unidades de nodo*meses.
2. Data Overage per Node (Datos por encima del límite por nodo): es el número de GB de datos ingeridos en exceso de la asignación de datos agregados.
3. Data Included per Node (Datos incluidos por nodo): es la cantidad de datos ingeridos incluidos en la asignación de datos agregados. Este medidor también se usa cuando el área de trabajo está en todos los planes de tarifa para mostrar la cantidad de datos incluidos en Azure Security Center.

> [!TIP]
> Si el área de trabajo tiene acceso al plan de tarifa **por nodo**, pero se pregunta si sería menos costoso un plan de tarifa de pago por uso, puede [usar la siguiente consulta](#evaluating-the-legacy-per-node-pricing-tier) para obtener fácilmente una recomendación. 

Las áreas de trabajo creadas antes de abril de 2016 también tienen acceso a los planes de tarifa **Estándar** y **Premium** originales, que tienen una retención de datos fija de 30 y 365 días, respectivamente. No se pueden crear áreas de trabajo en los planes de tarifa **Estándar** o **Premium**, y si un área de trabajo se saca de estos niveles, no puede regresar a ellos. Los medidores de ingesta de datos para estos niveles heredados se denominan "Datos analizados".

También hay algunos comportamientos entre el uso de los niveles de Log Analytics heredados y cómo se factura el uso en [Azure Security Center](https://docs.microsoft.com/azure/security-center/). 

1. Si el área de trabajo se encuentra en el nivel heredado Estándar o Premium, Azure Security Center se facturará solo por la ingesta de datos de Log Analytics, no por nodo.
2. Si el área de trabajo se encuentra en el nivel heredado Por nodo, Azure Security Center se facturará según el [modelo actual de precios basado en nodos de Azure Security Center](https://azure.microsoft.com/pricing/details/security-center/). 
3. En otros planes de tarifa (incluidas las reservas de capacidad), si Azure Security Center se habilitó antes del 19 de junio de 2017, Azure Security Center se facturará solo por la ingesta de datos de Log Analytics. De lo contrario, Azure Security Center se facturará según el modelo actual de precios basado en nodos de Azure Security Center.

Puede encontrar más detalles sobre las limitaciones de los planes de tarifa en [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md#log-analytics-workspaces).

Ninguno de los planes de tarifa heredados tiene precios basados en la región.  

> [!NOTE]
> Para usar los derechos que proceden de la adquisición de OMS E1 Suite, OMS E2 Suite o un complemento de OMS para System Center, elija el plan de tarifa *Por nodo* de Log Analytics.

## <a name="change-the-data-retention-period"></a>Cambio del período de retención de datos

Los pasos siguientes describen cómo configurar cuánto tiempo se conservan los datos de registro en el área de trabajo. La retención de datos se puede configurar entre 30 y 730 días (2 años) para todas las áreas de trabajo a menos que usen el plan de tarifa Gratis heredado. [Más información](https://azure.microsoft.com/pricing/details/monitor/) sobre los precios para una retención de datos más prolongada. 

### <a name="default-retention"></a>Retención predeterminada

Para establecer la retención predeterminada del área de trabajo, 
 
1. En Azure Portal, en el área de trabajo, seleccione **Uso y costos estimados** en el panel izquierdo.
2. En la página **Uso y costos estimados**, haga clic en **Retención de datos** en la parte superior de la página.
3. En el panel, mueva el control deslizante para aumentar o disminuir el número de días y, luego, haga clic en **Aceptar**.  Si el nivel es *gratuito*, no podrá modificar el período de retención de datos y tendrá que actualizar al nivel de pago para controlar esta configuración.

    ![Cambio de la configuración de retención de datos del área de trabajo](media/manage-cost-storage/manage-cost-change-retention-01.png)

Cuando se reduce la retención, hay un período de gracia de varios días antes de que se quiten los datos más antiguos que la nueva configuración de retención. 

La retención también se puede [configurar a través de Azure Resource Manager](template-workspace-configuration.md#configure-a-log-analytics-workspace) mediante el parámetro `retentionInDays`. Al configurar la retención de datos en 30 días, puede desencadenar una purga inmediata de los datos más antiguos mediante el parámetro `immediatePurgeDataOn30Days`, que elimina el período de gracias de varios días. Esto puede ser útil para escenarios relacionados con el cumplimiento en los que la eliminación de datos inmediata es imperativa. Esta funcionalidad de purga inmediata solo está expuesta a través de Azure Resource Manager. 

Las áreas de trabajo con una retención de 30 días en realidad pueden conservar los datos durante 31 días. Si es imperativo que los datos se conserven solo durante 30 días, use Azure Resource Manager para establecer la retención en 30 días y con el parámetro `immediatePurgeDataOn30Days`.  

Dos tipos de datos (`Usage` y `AzureActivity`) se conservan durante un mínimo de 90 días de forma predeterminada, y no se aplica ningún cargo por esta retención de 90 días. Si la retención del área de trabajo se aumenta por encima de los 90 días, también se aumentará la retención de estos tipos de datos.  Estos tipos de datos también están libres de los cargos de ingesta de datos. 

De forma predeterminada, los tipos de datos de los recursos de Application Insights basados en áreas de trabajo (`AppAvailabilityResults`, `AppBrowserTimings`, `AppDependencies`, `AppExceptions`, `AppEvents`, `AppMetrics`, `AppPageViews`, `AppPerformanceCounters`, `AppRequests`, `AppSystemEvents` y `AppTraces`) también se conservan durante 90 días, y no se aplica ningún cargo por esta retención de 90 días. Su retención se puede ajustar mediante la funcionalidad de retención por tipo de datos. 

Tenga en cuenta que la [API de purga](https://docs.microsoft.com/rest/api/loganalytics/workspacepurge/purge) de Log Analytics no afecta a la facturación de la retención y está pensada para usarse en casos muy limitados. Para reducir la factura de retención, el período de retención se debe reducir para el área de trabajo o para tipos de datos específicos. 

### <a name="retention-by-data-type"></a>Retención por tipo de datos

También es posible especificar diferentes configuraciones de retención para tipos de datos individuales de 30 a 730 días (excepto para áreas de trabajo del plan de tarifa gratuito heredado). Cada tipo de datos es un subrecurso del área de trabajo. Por ejemplo, la tabla SecurityEvent se puede tratar en [Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview) como:

```
/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables/SecurityEvent
```

Tenga en cuenta que el tipo de datos (tabla) distingue mayúsculas de minúsculas.  Para obtener la configuración actual de retención por tipo de datos de un tipo de datos determinado (en este ejemplo, SecurityEvent), use:

```JSON
    GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables/SecurityEvent?api-version=2017-04-26-preview
```

Para obtener la configuración actual de retención por tipo de datos para todos los tipos de datos del área de trabajo, solo tiene que omitir el tipo de datos específico, por ejemplo:

```JSON
    GET /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables?api-version=2017-04-26-preview
```

Para establecer la retención de un tipo de datos determinado (en este ejemplo, SecurityEvent) en 730 días, haga

```JSON
    PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables/SecurityEvent?api-version=2017-04-26-preview
    {
        "properties": 
        {
            "retentionInDays": 730
        }
    }
```

Los valores válidos para `retentionInDays` oscilan entre 30 y 730.

Los tipos de datos `Usage` y `AzureActivity` no se pueden establecer con retención personalizada. Adoptarán el máximo de la retención predeterminada del área de trabajo o 90 días. 

Una herramienta excelente para conectarse directamente a Azure Resource Manager para establecer la retención por tipo de datos es la herramienta OSS [ARMclient](https://github.com/projectkudu/ARMClient).  Más información sobre ARMclient en los artículos de [David Ebbo](http://blog.davidebbo.com/2015/01/azure-resource-manager-client.html) y [Daniel Bowbyes](https://blog.bowbyes.co.nz/2016/11/02/using-armclient-to-directly-access-azure-arm-rest-apis-and-list-arm-policy-details/).  A continuación se muestra un ejemplo de uso de ARMClient, estableciendo los datos de SecurityEvent con una retención de 730 días:

```
armclient PUT /subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/MyResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/MyWorkspaceName/Tables/SecurityEvent?api-version=2017-04-26-preview "{properties: {retentionInDays: 730}}"
```

> [!TIP]
> El establecimiento de la retención de tipos de datos individuales se puede usar para reducir los costos de la retención de datos.  Para los datos recopilados a partir de octubre de 2019 (fecha de publicación de esta característica), la reducción de la retención para algunos tipos de datos puede reducir el costo de retención con el tiempo.  Para los datos recopilados anteriormente, el establecimiento de una retención más baja para un tipo individual no afectará a los costos de retención.  

## <a name="manage-your-maximum-daily-data-volume"></a>Administración del volumen de datos diario máximo

Puede configurar un límite diario y limitar la ingesta diaria para el área de trabajo, pero tenga cuidado de establecer el límite diario como el objetivo al que llegar.  En caso contrario, perderá los datos para el resto del día, lo que puede afectar a otros servicios y soluciones de Azure cuya funcionalidad puede depender de la disponibilidad de datos actualizados en el área de trabajo.  Como resultado, esto afecta a la capacidad de observar y recibir alertas cuando cambian las condiciones de mantenimiento de los recursos que respaldan los servicios de TI.  El límite diario está pensado para usarse como una manera de administrar un **aumento inesperado** del volumen de datos de los recursos administrados y permanecer dentro de su límite, o simplemente cuando quiera limitar los cargos no planeados para el área de trabajo. No es adecuado para establecer un límite diario para que se cumpla cada día en un área de trabajo.

A cada área de trabajo se le aplica su límite diario en una hora diferente del día. La hora de restablecimiento se muestra en la página **Límite diario** (consulte más adelante). Esta hora de restablecimiento no se puede configurar. 

Poco después de alcanzar el límite diario, la recopilación de tipos de datos facturables se detiene durante el resto del día. La latencia inherente a la aplicación del límite diario significa que el límite no se aplica con precisión al nivel de límite diario especificado. Un mensaje emergente de advertencia aparece en la parte superior de la página del área de trabajo de Log Analytics seleccionada, y se envía un evento de operación para la tabla *Operación* en la categoría **LogManagement**. La recopilación de datos se reanuda después de que se restablezca el tiempo definido en *Daily limit will be set at* (El límite diario se establecerá en). Se recomienda definir una regla de alerta en función de este evento de operación que esté configurada para notificar cuando se ha alcanzado el límite diario de datos. 

> [!NOTE]
> El límite diario no puede detener la recopilación de datos con la misma precisión que el nivel de límite especificado y pueden esperarse datos sobrantes, especialmente si el área de trabajo recibe grandes volúmenes de datos.  

> [!WARNING]
> El límite diario no detiene la recopilación de datos de Azure Sentinel o Azure Security Center, excepto en el caso de las áreas de trabajo en las que Azure Security Center se instaló antes del 19 de junio de 2017. 

### <a name="identify-what-daily-data-limit-to-define"></a>Identificación del límite diario de datos para definir

Revise el [uso de Log Analytics y costos estimados](usage-estimated-costs.md) para comprender la tendencia de ingesta de datos y cuál es el límite de volumen diario para definir. Se debe examinar con detenimiento, ya que, una vez alcanzado el límite, no podrá supervisar los recursos. 

### <a name="set-the-daily-cap"></a>Establecer el límite diario

Los pasos siguientes describen cómo configurar un límite para administrar el volumen de datos que el área de trabajo de Log Analytics ingiere por día.  

1. En el área de trabajo seleccione **Usage and estimated costs** (Uso y costos estimados) en el panel izquierdo.
2. En la página **Usage and estimated costs** (Uso y costos estimados) del área de trabajo seleccionada, haga clic en **Límite de datos** en la parte superior de la página. 
3. ¿Está el límite diario **desactivado** de forma predeterminada? Haga clic en **ON** (ACTIVADO) para habilitarlo y, luego, establezca el límite de volumen diario en GB/día.

    ![Configuración del límite de datos con Log Analytics](media/manage-cost-storage/set-daily-volume-cap-01.png)
    
El límite diario se puede configurar mediante ARM si se establece el parámetro `dailyQuotaGb` en `WorkspaceCapping`, como se describe en [Áreas de trabajo: creación o actualización](https://docs.microsoft.com/rest/api/loganalytics/workspaces/createorupdate#workspacecapping). 

### <a name="alert-when-daily-cap-reached"></a>Alerta cuando se alcanza el límite diario

Aunque Azure Portal presenta una indicación visual cuando se alcanza el umbral de límite de datos, este comportamiento no tiene que alinearse necesariamente con la forma en la que el usuario administra los problemas de funcionamiento que requieren atención inmediata.  Para recibir una notificación de alerta, puede crear una nueva regla de alerta en Azure Monitor.  Para obtener más información, consulte [Cómo crear, ver y administrar alertas](alerts-metric.md).

Para comenzar, esta es la configuración recomendada para la alerta:

- Destino: seleccione el recurso de Log Analytics
- Criterios: 
   - Nombre de señal: Búsqueda de registros personalizada
   - Consulta de búsqueda: Operación | donde Detalle tiene "OverQuota"
   - Basado en: Número de resultados
   - Condición: Mayor que
   - Umbral: 0
   - Período: 5 (minutos)
   - Frecuencia: 5 (minutos)
- Nombre de regla de alertas: Límite de datos diario alcanzado
- Gravedad: advertencia (gravedad 1)

Una vez que se define la alerta y se alcanza el límite, se desencadena una alerta que realiza la respuesta que se define en el grupo de acciones. Puede notificar a su equipo a través de mensajes de correo electrónico y de texto, o automatizar acciones mediante webhooks, runbooks de Automation o [integrar con una solución de ITSM externa](itsmc-overview.md#create-itsm-work-items-from-azure-alerts). 

## <a name="troubleshooting-why-usage-is-higher-than-expected"></a>Solución del problema de un uso mayor de lo esperado

Un mayor uso está provocado por una de estas causas o por ambas:
- Más nodos de lo previsto que envían datos al área de trabajo de Log Analytics.
- Se enviaron más datos de los esperados al área de trabajo de Log Analytics (quizás debido a que empiece a usarse una solución nueva o a un cambio de configuración).

## <a name="understanding-nodes-sending-data"></a>Descripción de nodos que envían datos

Para conocer el número de equipos que notificaron latidos todos los días del último mes, use lo siguiente:

```kusto
Heartbeat 
| where TimeGenerated > startofday(ago(31d))
| summarize nodes = dcount(Computer) by bin(TimeGenerated, 1d)    
| render timechart
```
Para obtener un recuento de los nodos que han enviado datos en las últimas 24 horas, use la consulta: 

```kusto
find where TimeGenerated > ago(24h) project Computer
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize nodes = dcount(computerName)
```

Para obtener una lista de nodos que envían datos (y la cantidad de datos que envía cada uno) se puede usar la consulta siguiente:

```kusto
find where TimeGenerated > ago(24h) project _BilledSize, Computer
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize TotalVolumeBytes=sum(_BilledSize) by computerName
```

> [!TIP]
> Use estas consultas `find` con moderación, ya que la ejecución de exámenes entre tipos de datos [consume muchos recursos](https://docs.microsoft.com/azure/azure-monitor/log-query/query-optimization#query-performance-pane). Si no necesita resultados **por equipo**, consulte el tipo de datos Usage (vea a continuación).

## <a name="understanding-ingested-data-volume"></a>Descripción del volumen de datos ingeridos

En la página **Uso y costos estimados**, el gráfico *Ingesta de datos por solución*  muestra el volumen total de los datos enviados y la cantidad que envía cada solución. Esto le permite determinar tendencias tales como si el uso global de los datos (o el uso de una solución en particular) crece, permanece constante o disminuye. 

### <a name="data-volume-for-specific-events"></a>Volumen de datos para eventos específicos

Para ver el tamaño de los datos ingeridos de un conjunto determinado de eventos, puede consultar la tabla específica (en este ejemplo, `Event`) y, a continuación, restringir la consulta a los eventos de interés (en este ejemplo, identificador de evento 5145 o 5156):

```kusto
Event
| where TimeGenerated > startofday(ago(31d)) and TimeGenerated < startofday(now()) 
| where EventID == 5145 or EventID == 5156
| where _IsBillable == true
| summarize count(), Bytes=sum(_BilledSize) by EventID, bin(TimeGenerated, 1d)
``` 

Tenga en cuenta que la cláusula `where _IsBillable = true` filtra los tipos de datos de determinadas soluciones para las que no hay ningún cargo de ingesta. [Más información](log-standard-properties.md#_isbillable) sobre `_IsBillable`.

### <a name="data-volume-by-solution"></a>Data volume by solution (Volumen de datos por solución)

La consulta que se usa para ver el volumen de datos facturable por solución en el último mes (excepto el último día parcial) es:

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), Solution | render barchart
```

La cláusula con `TimeGenerated` solo se utiliza para asegurarse de que la experiencia de consulta en Azure Portal examine más allá del periodo predeterminado de 24 horas. Al utilizar el tipo de datos de uso, `StartTime` y `EndTime` representan los períodos de tiempo de los que se presentan resultados. 

### <a name="data-volume-by-type"></a>Volumen de datos por tipo

Puede profundizar más para ver las tendencias de datos por tipo de datos:

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) / 1000. by bin(StartTime, 1d), DataType | render barchart
```

O bien, para ver una tabla por solución y tipo durante el último mes,

```kusto
Usage 
| where TimeGenerated > ago(32d)
| where StartTime >= startofday(ago(31d)) and EndTime < startofday(now())
| where IsBillable == true
| summarize BillableDataGB = sum(Quantity) by Solution, DataType
| sort by Solution asc, DataType asc
```

### <a name="data-volume-by-computer"></a>Volumen de datos por equipo

El tipo de datos `Usage` no incluye información en el nivel de equipo. Para ver el **tamaño** de los datos ingeridos por cada equipo, use la [propiedad](log-standard-properties.md#_billedsize) `_BilledSize`, que proporciona el tamaño en bytes:

```kusto
find where TimeGenerated > ago(24h) project _BilledSize, _IsBillable, Computer
| where _IsBillable == true 
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| summarize BillableDataBytes = sum(_BilledSize) by  computerName 
| sort by BillableDataBytes nulls last
```

La [propiedad](log-standard-properties.md#_isbillable) `_IsBillable` especifica si los datos ingeridos generarán gastos. 

Para ver el **recuento** de eventos facturables que ingirió cada equipo, use: 

```kusto
find where TimeGenerated > ago(24h) project _IsBillable, Computer
| where _IsBillable == true 
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| summarize eventCount = count() by computerName  
| sort by eventCount nulls last
```

> [!TIP]
> Use estas consultas `find` con moderación, ya que la ejecución de exámenes entre tipos de datos [consume muchos recursos](https://docs.microsoft.com/azure/azure-monitor/log-query/query-optimization#query-performance-pane). Si no necesita resultados **por equipo**, consulte el tipo de datos Usage.

### <a name="data-volume-by-azure-resource-resource-group-or-subscription"></a>Volumen de datos por recurso de Azure, grupo de recursos o suscripción

Para los datos de los nodos hospedados en Azure, puede obtener el **tamaño** de los eventos ingeridos __por cada equipo__; para ello, use la [propiedad](log-standard-properties.md#_resourceid) _ResourceId, que proporciona la ruta completa al recurso:

```kusto
find where TimeGenerated > ago(24h) project _ResourceId, _BilledSize, _IsBillable
| where _IsBillable == true 
| summarize BillableDataBytes = sum(_BilledSize) by _ResourceId | sort by BillableDataBytes nulls last
```

Para los datos de los nodos hospedados en Azure, puede obtener el **tamaño** de los datos ingeridos __por cada suscripción de Azure__; obtenga el id. de suscripción de la propiedad `_ResourceId` como:

```kusto
find where TimeGenerated > ago(24h) project _ResourceId, _BilledSize, _IsBillable
| where _IsBillable == true 
| summarize BillableDataBytes = sum(_BilledSize) by _ResourceId
| extend subscriptionId = tostring(split(_ResourceId, "/")[2]) 
| summarize BillableDataBytes = sum(BillableDataBytes) by subscriptionId | sort by BillableDataBytes nulls last
```

Del mismo modo, para obtener el volumen de datos por grupo de recursos, sería:

```kusto
find where TimeGenerated > ago(24h) project _ResourceId, _BilledSize, _IsBillable
| where _IsBillable == true 
| summarize BillableDataBytes = sum(_BilledSize) by _ResourceId
| extend resourceGroup = tostring(split(_ResourceId, "/")[4] )
| summarize BillableDataBytes = sum(BillableDataBytes) by resourceGroup | sort by BillableDataBytes nulls last
```

Además, de ser necesario, puede analizar la propiedad `_ResourceId` más atentamente con:

```Kusto
| parse tolower(_ResourceId) with "/subscriptions/" subscriptionId "/resourcegroups/" 
    resourceGroup "/providers/" provider "/" resourceType "/" resourceName   
```

> [!TIP]
> Use estas consultas `find` con moderación, ya que la ejecución de exámenes entre tipos de datos [consume muchos recursos](https://docs.microsoft.com/azure/azure-monitor/log-query/query-optimization#query-performance-pane). Si no necesita resultados por suscripción, grupo de recursos ni nombre de recurso, consulte el tipo de datos Usage.

> [!WARNING]
> Algunos de los campos del tipo de datos de uso, aunque siguen en el esquema, han quedado en desuso y ya no se rellenarán sus valores. Estos son **Computer**, además de los campos relacionados con la ingesta (**TotalBatches**, **BatchesWithinSla**, **BatchesOutsideSla**, **BatchesCapped** y **AverageProcessingTimeMs**.


### <a name="querying-for-common-data-types"></a>Consulta de los tipos de datos comunes

Para profundizar en el origen de datos de un tipo de datos concreto, estas son algunas consultas de ejemplo útiles:

+ Recursos de **Application Insights basados en áreas de trabajo**
  - Obtenga más información en [Administración del uso y los costos de Application Insights](../app/pricing.md#data-volume-for-workspace-based-application-insights-resources)
+ Solución **Security**
  - `SecurityEvent | summarize AggregatedValue = count() by EventID`
+ Solución **Log Management**
  - `Usage | where Solution == "LogManagement" and iff(isnotnull(toint(IsBillable)), IsBillable == true, IsBillable == "true") == true | summarize AggregatedValue = count() by DataType`
+ Tipo de datos **Perf**
  - `Perf | summarize AggregatedValue = count() by CounterPath`
  - `Perf | summarize AggregatedValue = count() by CounterName`
+ Tipo de datos **Event**
  - `Event | summarize AggregatedValue = count() by EventID`
  - `Event | summarize AggregatedValue = count() by EventLog, EventLevelName`
+ Tipo de datos **Syslog**
  - `Syslog | summarize AggregatedValue = count() by Facility, SeverityLevel`
  - `Syslog | summarize AggregatedValue = count() by ProcessName`
+ Tipo de datos de **AzureDiagnostics**
  - `AzureDiagnostics | summarize AggregatedValue = count() by ResourceProvider, ResourceId`

## <a name="tips-for-reducing-data-volume"></a>Sugerencias para reducir el volumen de datos

Algunas sugerencias para reducir el volumen de registros recopilados incluyen:

| Origen del mayor volumen de datos | Cómo reducir el volumen de datos |
| -------------------------- | ------------------------- |
| Container Insights         | [Configure Container Insights](https://docs.microsoft.com/azure/azure-monitor/insights/container-insights-cost#controlling-ingestion-to-reduce-cost) para recopilar solo los datos necesarios. |
| Eventos de seguridad            | Seleccione los [eventos de seguridad común o mínima](https://docs.microsoft.com/azure/security-center/security-center-enable-data-collection#data-collection-tier). <br> Cambie la directiva de auditoría de seguridad para recopilar únicamente los eventos necesarios. En particular, revise la necesidad de recopilar eventos para <br> - [auditar plataforma de filtrado](https://technet.microsoft.com/library/dd772749(WS.10).aspx) <br> - [auditar registro](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd941614(v%3dws.10))<br> - [auditar sistema de archivos](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772661(v%3dws.10))<br> - [auditar objeto de kernel](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd941615(v%3dws.10))<br> - [auditar manipulación de identificadores](https://docs.microsoft.com/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/dd772626(v%3dws.10))<br> - auditar almacenamiento extraíble |
| Contadores de rendimiento       | Cambie la [configuración de los contadores de rendimiento](data-sources-performance-counters.md) para: <br> - Reducir la frecuencia de la colección <br> - Reducir el número de contadores de rendimiento |
| Registros de eventos                 | Cambie la [configuración del registro de eventos](data-sources-windows-events.md) para: <br> - Reducir el número de registros de eventos recopilados <br> - Recopilar solo los niveles de eventos necesarios Por ejemplo, no recopile eventos de nivel de *información*. |
| syslog                     | Cambie la [configuración de syslog](data-sources-syslog.md) para: <br> - Reducir el número de instalaciones recopiladas <br> - Recopilar solo los niveles de eventos necesarios Por ejemplo, no recopile eventos de nivel de *información* y *depuración*. |
| AzureDiagnostics           | Cambie la colección de registros de recursos para: <br> - Reducir el número de registros de recursos enviados a Log Analytics <br> - Recopilar solo los registros necesarios |
| Datos de la solución procedentes de equipos que no necesitan la solución | Use la [selección de destino de solución](../insights/solution-targeting.md) para recopilar datos solo de los grupos de equipos necesarios. |

### <a name="getting-nodes-as-billed-in-the-per-node-pricing-tier"></a>Obtención de nodos facturados en el plan de tarifa Por nodo

Para obtener una lista de equipos que se facturarán como nodos si el área de trabajo se encuentra en el plan de tarifa heredado de cada nodo, busque los nodos que envían **tipos de datos facturados** (algunos tipos de datos son gratuitos). Para hacer esto, use la [propiedad](log-standard-properties.md#_isbillable) `_IsBillable` y el campo situado más a la izquierda del nombre de dominio completo. De este modo se devuelve el recuento de equipos con datos facturados por hora (que es la granularidad con la que se contabilizan y se facturan los nodos):

```kusto
find where TimeGenerated > ago(24h) project Computer, TimeGenerated
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize billableNodes=dcount(computerName) by bin(TimeGenerated, 1h) | sort by TimeGenerated asc
```

### <a name="getting-security-and-automation-node-counts"></a>Obtener recuentos de nodos de seguridad y automatización

Si se encuentra en el plan de tarifa "Por nodo (OMS)", se le cobrará en función del número de nodos y soluciones que utilice y el número de nodos de Insight and Analytics por los que se le factura se mostrará en la tabla de la página **Uso y costos estimados**.  

Para ver el número de nodos de seguridad distintos, puede utilizar la consulta:

```kusto
union
(
    Heartbeat
    | where (Solutions has 'security' or Solutions has 'antimalware' or Solutions has 'securitycenter')
    | project Computer
),
(
    ProtectionStatus
    | where Computer !in~
    (
        (
            Heartbeat
            | project Computer
        )
    )
    | project Computer
)
| distinct Computer
| project lowComputer = tolower(Computer)
| distinct lowComputer
| count
```

Para ver el número de nodos de Automation distintos, utilice la consulta:

```kusto
 ConfigurationData 
 | where (ConfigDataType == "WindowsServices" or ConfigDataType == "Software" or ConfigDataType =="Daemons") 
 | extend lowComputer = tolower(Computer) | summarize by lowComputer 
 | join (
     Heartbeat 
       | where SCAgentChannel == "Direct"
       | extend lowComputer = tolower(Computer) | summarize by lowComputer, ComputerEnvironment
 ) on lowComputer
 | summarize count() by ComputerEnvironment | sort by ComputerEnvironment asc
```

## <a name="evaluating-the-legacy-per-node-pricing-tier"></a>Evaluación del plan de tarifa por nodo heredado

La decisión de si las áreas de trabajo con acceso a los planes de tarifas **por nodo** heredados están mejor en ese plan o en otro de **pago por uso** o de **reserva de capacidad** suele resultar difícil de evaluar para los clientes.  Implica conocer el equilibrio entre el costo fijo por nodo supervisado en el plan de tarifa por nodo y su asignación de datos incluida de 500 MB/nodo/día, y el costo que supone pagar solo por los datos ingeridos en la tarifa de pago por uso (por GB). 

Para facilitar esta valoración, se puede usar la consulta siguiente a fin de realizar una recomendación para el plan de tarifa óptimo en función de los patrones de uso de un área de trabajo.  Esta consulta examina los nodos supervisados y los datos ingeridos en un área de trabajo los últimos siete días y evalúa para cada día qué plan de tarifa habría sido el óptimo. Para usar la consulta, debe especificar

1. si el área de trabajo usa Azure Security Center estableciendo `workspaceHasSecurityCenter` en `true` o `false`; 
2. actualizar los precios si tiene descuentos específicos;
3. especificar el número de días que se van a buscar y analizar estableciendo `daysToEvaluate`. Esto resulta útil si la consulta tarda demasiado en buscar datos para 7 días. 

Esta es la consulta de recomendación del nivel de precios:

```kusto
// Set these parameters before running query
let workspaceHasSecurityCenter = true;  // Specify if the workspace has Azure Security Center
let PerNodePrice = 15.; // Enter your montly price per monitored nodes
let PerNodeOveragePrice = 2.30; // Enter your price per GB for data overage in the Per Node pricing tier
let PerGBPrice = 2.30; // Enter your price per GB in the Pay-as-you-go pricing tier
let daysToEvaluate = 7; // Enter number of previous days look at (reduce if the query is taking too long)
// ---------------------------------------
let SecurityDataTypes=dynamic(["SecurityAlert", "SecurityBaseline", "SecurityBaselineSummary", "SecurityDetection", "SecurityEvent", "WindowsFirewall", "MaliciousIPCommunication", "LinuxAuditLog", "SysmonEvent", "ProtectionStatus", "WindowsEvent", "Update", "UpdateSummary"]);
let StartDate = startofday(datetime_add("Day",-1*daysToEvaluate,now()));
let EndDate = startofday(now());
union * 
| where TimeGenerated >= StartDate and TimeGenerated < EndDate
| extend computerName = tolower(tostring(split(Computer, '.')[0]))
| where computerName != ""
| summarize nodesPerHour = dcount(computerName) by bin(TimeGenerated, 1h)  
| summarize nodesPerDay = sum(nodesPerHour)/24.  by day=bin(TimeGenerated, 1d)  
| join kind=leftouter (
    Heartbeat 
    | where TimeGenerated >= StartDate and TimeGenerated < EndDate
    | where Computer != ""
    | summarize ASCnodesPerHour = dcount(Computer) by bin(TimeGenerated, 1h) 
    | extend ASCnodesPerHour = iff(workspaceHasSecurityCenter, ASCnodesPerHour, 0)
    | summarize ASCnodesPerDay = sum(ASCnodesPerHour)/24.  by day=bin(TimeGenerated, 1d)   
) on day
| join (
    Usage 
    | where TimeGenerated >= StartDate and TimeGenerated < EndDate
    | where IsBillable == true
    | extend NonSecurityData = iff(DataType !in (SecurityDataTypes), Quantity, 0.)
    | extend SecurityData = iff(DataType in (SecurityDataTypes), Quantity, 0.)
    | summarize DataGB=sum(Quantity)/1000., NonSecurityDataGB=sum(NonSecurityData)/1000., SecurityDataGB=sum(SecurityData)/1000. by day=bin(StartTime, 1d)  
) on day
| extend AvgGbPerNode =  NonSecurityDataGB / nodesPerDay
| extend PerGBDailyCost = iff(workspaceHasSecurityCenter,
             (NonSecurityDataGB + max_of(SecurityDataGB - 0.5*ASCnodesPerDay, 0.)) * PerGBPrice,
             DataGB * PerGBPrice)
| extend OverageGB = iff(workspaceHasSecurityCenter, 
             max_of(DataGB - 0.5*nodesPerDay - 0.5*ASCnodesPerDay, 0.), 
             max_of(DataGB - 0.5*nodesPerDay, 0.))
| extend PerNodeDailyCost = nodesPerDay * PerNodePrice / 31. + OverageGB * PerNodeOveragePrice
| extend Recommendation = iff(PerNodeDailyCost < PerGBDailyCost, "Per Node tier", 
             iff(NonSecurityDataGB > 85., "Capacity Reservation tier", "Pay-as-you-go (Per GB) tier"))
| project day, nodesPerDay, ASCnodesPerDay, NonSecurityDataGB, SecurityDataGB, OverageGB, AvgGbPerNode, PerGBDailyCost, PerNodeDailyCost, Recommendation | sort by day asc
//| project day, Recommendation // Comment this line to see details
| sort by day asc
```

Esta consulta no es una replicación exacta de cómo se calcula el uso, pero funcionará para proporcionar recomendaciones de plan de tarifa en la mayoría de los casos.  

> [!NOTE]
> Para usar los derechos que proceden de la adquisición de OMS E1 Suite, OMS E2 Suite o un complemento de OMS para System Center, elija el plan de tarifa *Por nodo* de Log Analytics.

## <a name="create-an-alert-when-data-collection-is-high"></a>Creación de una alerta cuando la colección de datos es mayor

En esta sección se describe cómo crear una alerta si el volumen de datos en las últimas 24 horas ha superado una cantidad especificada, mediante [Alertas de registro](alerts-unified-log.md) de Azure Monitor. 

Para generar una alerta si la ingesta del volumen de datos facturable en las últimas 24 horas fue superior a 50 GB, siga estos pasos: 

- **Definición de la condición de alerta**: especifique el área de trabajo de Log Analytics como el destino del recurso.
- **Criterios de alerta**: especifique lo siguiente:
   - **Nombre de señal**: seleccione **Custom log search** (Búsqueda de registros personalizada)
   - **Consulta de búsqueda** en `Usage | where IsBillable | summarize DataGB = sum(Quantity / 1000.) | where DataGB > 50`. 
   - El valor de **Lógica de alerta** está **Basado en** el *número de resultados*, mientras que el valor de **Condición** es *Mayor que* un **Umbral**  de *0*.
   - **Período de tiempo** de *1440* minutos y **Frecuencia de alerta** cada *1440* minutos para ejecutarse una vez al día.
- **Definición de los detalles de la alerta**: especifique lo siguiente:
   - **Nombre** en *Billable data volume greater than 50 GB in 24 hours* (Volumen de datos facturable mayor que 50 GB en 24 horas)
   - **Gravedad** en *Advertencia*

Especifique un [grupo de acciones](action-groups.md) existente o cree uno nuevo para que cuando la alerta de registro coincida con criterios, se le notifique.

Cuando se recibe una alerta, siga los pasos de las secciones siguientes sobre cómo solucionar el problema del uso mayor de lo esperado.

## <a name="data-transfer-charges-using-log-analytics"></a>Cargos por transferencia de datos mediante Log Analytics

Al enviar datos a Log Analytics se pueden aplicar ciertos cargos debido al ancho de banda de datos. Tal como se describe en la [página de precios de Azure Bandwidth](https://azure.microsoft.com/pricing/details/bandwidth/), la transferencia de datos entre los servicios de Azure ubicados en dos regiones se cobra como transferencia de datos salientes a precio normal. La transferencia de datos entrantes es gratuita. Sin embargo, este cargo es muy pequeño (un tanto por ciento mínimo) en comparación con los costos de la ingesta de datos de Log Analytics. Por lo tanto, el control de los costos de Log Analytics tiene que centrarse en el [volumen de datos ingerido](#understanding-ingested-data-volume). 


## <a name="troubleshooting-why-log-analytics-is-no-longer-collecting-data"></a>Solucionar que Log Analytics ya no recopile datos

Si tiene el plan de tarifa gratuito heredado y ha enviado más de 500 MB de datos en un día, la recopilación de datos se detiene durante el resto del día. Alcanzar el límite diario es un motivo frecuente de que Log Analytics deje de recopilar datos o de que parezca que faltan datos.  Log Analytics crea un evento de tipo Operación cuando la recopilación de datos se inicia y se detiene. Ejecute la siguiente consulta en la búsqueda para comprobar si ha alcanzado el límite diario y faltan datos: 

```kusto
Operation | where OperationCategory == 'Data Collection Status'
```

Cuando se detiene la recopilación de datos, el valor de OperationStatus es **Advertencia**. Cuando se inicia la recopilación de datos, el valor de OperationStatus es **Correcto**. La tabla siguiente describe los motivos por los que se detiene la recopilación de datos y una acción recomendada para reanudarla:  

|Motivo por el que se detiene la recopilación| Solución| 
|-----------------------|---------|
|Se alcanzó el límite diario del área de trabajo|Espere para que se reinicie automáticamente la recopilación o aumente el límite diario de volumen de datos que se describe en Administración del volumen de datos diario máximo El tiempo de restablecimiento del límite diario se muestra en la página **Límite diario**. |
| El área de trabajo ha alcanzado la [tasa de volumen de ingesta de datos](https://docs.microsoft.com/azure/azure-monitor/service-limits#log-analytics-workspaces). | En las áreas de trabajo se aplica un umbral de velocidad de volumen de ingesta de datos de 500 MB (comprimidos), que es aproximadamente **6 GB/min** sin comprimir (el tamaño real puede variar entre los tipos de datos en función de la longitud del registro y su relación de compresión). Este umbral se aplica a todos los datos ingeridos, tanto si se envían desde recursos de Azure mediante la [configuración de diagnóstico](diagnostic-settings.md), [Data Collector API](data-collector-api.md) o agentes. Cuando se envían datos a un área de trabajo a una velocidad superior al 80 % del umbral configurado en el área de trabajo, se envía un evento a la tabla *Operación* del área de trabajo cada 6 horas mientras se siga superando el umbral. Cuando la velocidad de ingesta del volumen supera el umbral, se quitan algunos datos y se envía un evento a la tabla *Operación* del área de trabajo cada 6 horas mientras se siga superando el umbral. Si la velocidad de ingesta del sigue superando el umbral o prevé que lo va a alcanzar pronto, puede abrir una solicitud de soporte técnico para solicitar su aumento en su área de trabajo. Para recibir una notificación de este evento en el área de trabajo, cree una [regla de alerta de registro](alerts-log.md) mediante la siguiente consulta que utiliza lógica de alerta basada en un número de resultados superior a cero, un periodo de evaluación de 5 minutos y una frecuencia de 5 minutos. La velocidad de ingesta del volumen ha alcanzado el 80 % del umbral: `Operation | where OperationCategory == "Ingestion" | where Detail startswith "The data ingestion volume rate crossed 80% of the threshold"`. La velocidad de ingesta del volumen ha alcanzado el umbral: `Operation | where OperationCategory == "Ingestion" | where Detail startswith "The data ingestion volume rate crossed the threshold"`. |
|Se alcanzó el límite diario del plan de tarifa Gratis heredado |Espere hasta el día siguiente para que la recopilación se reinicie automáticamente, o cambie a un plan de tarifa de pago.|
|La suscripción de Azure está en estado suspendido debido a:<br> Prueba gratuita finalizada<br> Pase para Azure expirado<br> Se ha alcanzado el límite de gasto mensual (por ejemplo, en una suscripción de MSDN o Visual Studio)|Cambie a una suscripción de pago<br> Quite el límite o espere a que se restablezca|

Para recibir una notificación cuando se detenga la recopilación de datos, siga los pasos descritos en la alerta *Crear límite de datos diario* para recibir una notificación cuando se detenga la recopilación de datos. Siga los pasos descritos en [Crear un grupo de acciones](action-groups.md) para configurar una acción de correo electrónico, de webhook o de runbook para la regla de alerta. 

## <a name="limits-summary"></a>Resumen de límites

Existen algunas limitaciones adicionales de Log Analytics, algunas de los cuales dependen del plan de tarifa de Log Analytics. Se documentan en [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md#log-analytics-workspaces).


## <a name="next-steps"></a>Pasos siguientes

- Consulte [Log searches in Azure Monitor Logs](../log-query/log-query-overview.md) (Búsquedas de registros en el registro de Azure Monitor) para obtener información sobre cómo usar el lenguaje de búsqueda. Puede utilizar las consultas de búsqueda para realizar análisis adicionales sobre los datos de uso.
- Siga los pasos explicados en [Crear una nueva alerta de registro](alerts-metric.md) para recibir una notificación cuando se cumplan los criterios de búsqueda.
- Use la [selección de destino de solución](../insights/solution-targeting.md) para recopilar datos solo de los grupos de equipos necesarios.
- Para configurar una directiva eficaz de recopilación de eventos, revise la [Directiva de filtrado de Azure Security Center](../../security-center/security-center-enable-data-collection.md).
- Cambie la [configuración de los contadores de rendimiento](data-sources-performance-counters.md).
- Para modificar la configuración de recopilación de eventos, revise la [configuración de registros de eventos](data-sources-windows-events.md).
- Para modificar la configuración de la recopilación de syslog, revise la [configuración de syslog](data-sources-syslog.md).
