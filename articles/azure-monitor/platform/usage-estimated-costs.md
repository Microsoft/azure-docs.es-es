---
title: Supervisión del uso y costos estimados en Azure Monitor
description: Introducción al proceso de uso de la página Uso y costos estimados de Azure Monitor
author: dalekoetke
services: azure-monitor
ms.topic: conceptual
ms.date: 10/28/2019
ms.author: mbullwin
ms.reviewer: Dale.Koetke
ms.subservice: ''
ms.openlocfilehash: eb96537f67c61fb31759da020068f784d0e89993
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323407"
---
# <a name="monitoring-usage-and-estimated-costs-in-azure-monitor"></a>Supervisión del uso y costos estimados en Azure Monitor

> [!NOTE]
> En este artículo se describe cómo ver el uso y los costos estimados a través de varias características de supervisión de Azure. Los artículos relacionados con componentes específicos de Azure Monitor incluyen:
> - En [Administrar el uso y los costos con los registros de Azure Monitor](manage-cost-storage.md) se describe cómo controlar los costos cambiando el período de retención de datos y cómo analizar y alertar sobre el uso de los datos.
> - En [Administración del uso y los costos de Application Insights](../app/pricing.md) se describe cómo analizar el uso de datos en Application Insights.

## <a name="azure-monitor-pricing-model"></a>Modelo de precios de Azure Monitor

El modelo de facturación básico de Azure Monitor consta de precios basados en la nube y el consumo ("pago por uso"). Pague solo por lo que usa. Hay detalles disponibles sobre los precios para [alertas, métricas y notificaciones](https://azure.microsoft.com/pricing/details/monitor/), [Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/) y [Application Insights](https://azure.microsoft.com/pricing/details/application-insights/). 

Además del modelo de pago por uso para los datos de registros, Log Analytics tiene niveles de Reserva de capacidad que le permiten ahorrar hasta un 25 % en comparación con el precio de pago por uso. Los precios de la reserva de capacidad permiten comprar una reserva a partir de 100 GB/día. Cualquier uso por encima del nivel de reserva se facturará según la tarifa de pago por uso. [Obtenga más información](https://azure.microsoft.com/pricing/details/monitor/) sobre los precios de la reserva de capacidad.

Algunos clientes tendrán acceso a los [planes de tarifa de Log Analytics heredados](./manage-cost-storage.md#legacy-pricing-tiers) y el [plan de tarifa de Enterprise Application Insights heredado](../app/pricing.md#legacy-enterprise-per-node-pricing-tier). 

## <a name="understanding-your-azure-monitor-costs"></a>Información sobre los costos de Azure Monitor

Hay dos fases para comprender los costos. La primera es al momento de plantearse Azure Monitor como solución de supervisión. 

### <a name="estimating-the-costs-to-manage-your-environment"></a>Estimación de los costos de administración del entorno

Si aún no usa los registros de Azure Monitor, puede usar la [calculadora de precios de Azure Monitor](https://azure.microsoft.com/pricing/calculator/?service=monitor) para calcular el costo del uso de Azure Monitor. Comience por escribir "Azure Monitor" en el cuadro de búsqueda y haga clic en el icono de Azure Monitor resultante. Desplácese hacia abajo en la página hasta Azure Monitor y seleccione una de las opciones de la lista desplegable Tipo:

- Consultas y alertas de métricas  
- Log Analytics
- Application Insights

En cada una de ellas, la calculadora de precios le ayudará a calcular los costos más probables en función del uso esperado.

Por ejemplo, con Log Analytics puede especificar el número de máquinas virtuales y los GB de datos que espera recopilar de cada máquina virtual. Normalmente se ingieren de 1 GB a 3 GB de datos al mes en una máquina virtual típica de Azure. Si ya está evaluando los registros de Azure Monitor, puede usar las estadísticas de datos de su propio entorno. Consulte a continuación cómo determinar el [número de máquinas virtuales supervisadas](./manage-cost-storage.md#understanding-nodes-sending-data) y el [volumen de los datos que ingiere el área de trabajo](./manage-cost-storage.md#understanding-ingested-data-volume).

De igual forma, para Application Insights, si habilita la funcionalidad "Estimar el volumen de datos en función de la actividad de la aplicación", puede proporcionar entradas sobre la aplicación (solicitudes por mes y vistas de página al mes, en caso de que se recopile la telemetría del lado cliente) y, a continuación, la calculadora le indicará el valor medio y la cantidad de percentil 90 de los datos recopilados por aplicaciones similares. Estas aplicaciones abarcan el intervalo de configuración de Application Insights (por ejemplo, algunas tienen el muestreo predeterminado, otras no tienen muestreo, etc.), por lo que todavía tiene el control para reducir el volumen de datos que ingiere muy por debajo del nivel medio con muestreo. Pero se trata de un punto de partida para comprender lo que ven otros clientes similares. [Más información](../app/pricing.md#estimating-the-costs-to-manage-your-application) sobre la estimación de costos de Application Insights.

### <a name="understanding-your-usage-and-estimated-costs"></a>Información sobre el uso y los costos estimados

Es importante comprender y hacer un seguimiento del uso una vez que comience a usar Azure Monitor y hay un conjunto completo de herramientas para facilitar esta tarea. 

Azure proporciona una gran cantidad de funcionalidades útiles en el centro [Azure Cost Management + Facturación](../../cost-management-billing/costs/quick-acm-cost-analysis.md?toc=/azure/billing/TOC.json). Después de abrir el centro **Azure Cost Management y facturación**, haga clic en **Cost Management** y seleccione el [ámbito](../../cost-management-billing/costs/understand-work-scopes.md) (el conjunto de recursos que se van a investigar). 

A continuación, para consultar los costos de Azure Monitor de los últimos 30 días, haga clic en el icono **Costos diarios**, elija "Últimos 30 días" en Fechas relativas y agregue un filtro que seleccione los nombres de servicio:

1. Azure Monitor
2. Application Insights
3. Log Analytics
4. Insight and Analytics

Esto da como resultado una vista como la siguiente:

![Captura de pantalla de Azure Cost Management](./media/usage-estimated-costs/010.png)

Desde aquí, puede profundizar en este resumen de costos acumulados para obtener detalles más precisos en la vista "Costo por recurso". En los planes de tarifa actuales, los datos de registro de Azure se cobran según el mismo conjunto de medidores independientemente de si se originan en Log Analytics o Application Insights. Para separar los costos del uso de Log Analytics o Application Insights, puede agregar un filtro en **Tipo de recurso**. Para ver todos los costos de Application Insights, filtre el tipo de recurso por "microsoft.insights/components", y para los costos de Log Analytics, filtre el tipo de recurso por "microsoft.operationalinsights/workspaces". 

Puede consultar más detalles sobre el uso mediante la [descarga del uso desde Azure Portal](../../cost-management-billing/manage/download-azure-invoice-daily-usage-date.md#download-usage-in-azure-portal). En la hoja de cálculo descargada puede ver el uso por recurso de Azure al día. En esta hoja de cálculo de Excel, el uso de los recursos de Application Insights se puede encontrar filtrando, en primer lugar, la columna "Categoría de medición" para mostrar "Application Insights" y "Log Analytics" y, a continuación, agregando un filtro en la columna "Id. de instancia", que es "contiene microsoft.insights/components".  La mayor parte del uso de Application Insights se muestra en medidores con la Categoría de medición de Log Analytics, ya que hay un back-end de registros único para todos los componentes de Azure Monitor.  Con una Categoría de medición de Application Insights, solo se muestran recursos de Application Insights de los planes de tarifa heredados y las pruebas web de varios pasos.  El uso se muestra en la columna "Cantidad consumida" y la unidad de cada entrada se muestra en la columna "Unidad de medida".  Hay más detalles disponibles para ayudarle a [entender la factura de Microsoft Azure](../../cost-management-billing/understand/review-individual-bill.md). 

> [!NOTE]
> El uso de **Cost Management** en el centro de **Administración de costos y facturación de Azure** es el mejor método para comprender de forma general los costos de supervisión.  Las experiencias de **Uso y costos estimados** de [Log Analytics](./manage-cost-storage.md#understand-your-usage-and-estimate-costs) y [Application Insights](../app/pricing.md#understand-your-usage-and-estimate-costs) proporcionan información más detallada sobre cada una de las partes de Azure Monitor.

Otra opción para ver el uso de Azure Monitor es la página de **Uso y costos estimados** en el centro de supervisión. Esto muestra el uso de características de supervisión principales como [alertas, métricas, notificaciones](https://azure.microsoft.com/pricing/details/monitor/), [Azure Log Analytics](https://azure.microsoft.com/pricing/details/log-analytics/) y [Azure Application Insights](https://azure.microsoft.com/pricing/details/application-insights/). Para los clientes en los planes de precios disponibles antes de abril de 2018, esto también incluye el uso de Log Analytics adquirido a través de la oferta Insights y Analytics.

En esta página, los usuarios pueden ver el uso de sus recursos durante los últimos 31 días, agregados por suscripción. `Drill-ins` muestra las tendencias de uso durante el período de 31 días. Es necesario que una gran cantidad de datos se reúna para este cálculo, por lo que agradecemos su paciencia mientras se carga la página.

En este ejemplo se muestra la supervisión del uso y una estimación de los costos resultantes:

![Captura de pantalla de la página Uso y costos estimados del portal](./media/usage-estimated-costs/001.png)

Seleccione el vínculo de la columna de uso mensual para abrir un gráfico que muestra las tendencias de uso en el último período de 31 días: 

![Captura de la pantalla del gráfico de barras Incluido por nodo](./media/usage-estimated-costs/002.png)

## <a name="operations-management-suite-subscription-entitlements"></a>Derechos de suscripción de Operations Management Suite

Los clientes que compraron Microsoft Operations Management Suite E1 y E2 son aptos para los derechos de ingesta de datos por nodo para [Log Analytics](https://www.microsoft.com/cloud-platform/operations-management-suite) y [Application Insights](../app/pricing.md). Para recibir estos derechos para las áreas de trabajo de Log Analytics o los recursos de Application Insights en una suscripción determinada: 

- Las áreas de trabajo de Log Analytics deben usar el plan de tarifa "por nodo (OMS)".
- Los recursos de Application Insights deben utilizar el plan de tarifa "Enterprise".

En función del número de nodos del conjunto de aplicaciones que adquiriera la organización, puede ser de utilidad mover alguna suscripción al plan de tarifa de pago por uso (por GB), pero le recomendamos que antes lo piense detenidamente.

> [!WARNING]
> Si su organización tiene las Microsoft Operations Management Suite E1 y E2 actuales, normalmente es mejor mantener sus áreas de trabajo de Log Analytics en el plan de tarifa "por nodo (OMS)" y los recursos de Application Insights en el plan de tarifa "Enterprise". 
>

