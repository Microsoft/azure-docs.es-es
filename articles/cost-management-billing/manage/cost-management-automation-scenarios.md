---
title: Escenarios de automatización para la administración de los costos y la facturación de Azure
description: Obtenga información sobre cómo se asignan escenarios comunes de administración de costos y facturación a diferentes API.
author: bandersmsft
ms.reviewer: adwise
tags: billing
ms.service: cost-management-billing
ms.subservice: common
ms.topic: reference
ms.date: 02/12/2020
ms.author: banders
ms.openlocfilehash: 5ee544a307be26c737a05e2147298a31a9ef7d1a
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88689990"
---
# <a name="automation-scenarios-for-billing-and-cost-management"></a>Escenarios de automatización de administración de costos y facturación

En este artículo se enumeran los escenarios comunes para la administración de costos y facturación de Azure. A estos escenarios se les asignan las API que puede usar. En cada asignación de escenario a API, puede encontrar un resumen de las API y la funcionalidad que ofrecen.

## <a name="common-scenarios"></a>Escenarios frecuentes

Puede usar las API de administración de costos y facturación en distintos escenarios para responder a las preguntas relacionadas con los costos y el uso. A continuación se proporciona un resumen de los escenarios comunes:

- **Conciliación de facturas**: ¿Microsoft me cobró el importe correcto?  ¿Cuál es mi factura y puedo calcularla yo mismo?

- **Cargos cruzados**: ahora que ya sé cuándo se me va a cobrar, ¿quién debe encargarse de pagar en mi organización?

- **Optimización de costos**: sé cuánto se me ha cobrado. ¿Cómo puedo obtener mayor rendimiento del dinero que gasto en Azure?

- **Seguimiento de costos**: me gustaría consultar lo que gasto y uso en Azure a lo largo del tiempo. ¿Cuáles son las tendencias? ¿Cómo puedo hacerlo mejor?

- **Gasto mensual en Azure**: ¿cuánto llevo gastado este mes? ¿Debo realizar algún cambio en lo que gasto o uso en Azure? ¿En qué momento del mes registro más consumo de Azure?

- **Alertas**: ¿cómo puedo configurar alertas basadas en el gasto o el consumo de recursos?

## <a name="scenario-to-api-mapping"></a>Asignación de escenario a API

|         API        | Conciliación de facturas    | Cargos cruzados    | Optimización de costos    | Seguimiento de costos    | Gasto a mitad de mes    | Alertas    |
|:---------------------------:|:-------------------------:|:----------------:|:--------------------:|:----------------:|:------------------:|:---------:|
| Presupuestos                     |                           |                  |           X          |                  |                    |     X     |
| Cargos de Marketplace                |             X             |         X        |           X          |         X        |          X         |     X     |
| Hoja de precios                 |             X             |         X        |           X          |         X        |          X         |           |
| Recomendaciones de reserva |                           |                  |           X          |                  |                    |           |
| Detalles de la reserva         |                           |                  |           X          |         X        |                    |           |
| Resúmenes de reservas       |                           |                  |           X          |         X        |                    |           |
| Detalles de uso               |             X             |         X        |           X          |         X        |          X         |     X     |
| Períodos de facturación             |             X             |         X        |           X          |         X        |                    |           |
| Facturas                    |             X             |         X        |           X          |         X        |                    |           |
| RateCard                    |             X             |                  |           X          |         X        |          X         |           |
| Uso sin calificar               |             X             |                  |           X          |                  |          X         |           |

> [!NOTE]
> La asignación de escenario a API no incluye las API de consumo Enterprise. Siempre que sea posible, use las API de consumo generales para los nuevos escenarios de desarrollo.

## <a name="api-summaries"></a>Resúmenes de API

### <a name="consumption"></a>Consumo
Los clientes de Web Direct y Enterprise pueden usar todas las API siguientes, excepto donde se indique:

-   [Budgets API](https://docs.microsoft.com/rest/api/consumption/budgets) (*solo clientes Enterprise*): cree presupuestos de costos o uso para recursos, grupos de recursos o medidores de facturación. Cuando haya creado los presupuestos, puede configurar alertas para informarle cuando supere los umbrales de presupuesto definidos. También puede configurar acciones que deben realizarse cuando alcance las cantidades del presupuesto.

-   [Marketplace Charges API](https://docs.microsoft.com/rest/api/consumption/marketplaces): obtenga los datos de los cobros y el uso de todos los recursos de Azure Marketplace (ofertas de asociados de Azure). Puede utilizar estos datos para sumar los costos de todos los recursos de Marketplace o investigar los costos o el uso de recursos específicos.

-   [Price Sheet API](https://docs.microsoft.com/rest/api/consumption/pricesheet) (*solo clientes Enterprise*): obtenga precios personalizados para todos los medidores. Las empresas pueden utilizar estos datos en combinación con los detalles de uso y la información de uso de las instancias de Marketplace para calcular los costos en función de los datos de uso y de Marketplace.

-   [Reservation Recommendations API](https://docs.microsoft.com/rest/api/consumption/reservationrecommendations): obtenga recomendaciones de compra de instancias reservadas de máquina virtual. Las recomendaciones ayudarán a analizar las cantidades de compra y los ahorros en costos esperados. Para más información, consulte [APIs for Azure reservation automation](../reservations/reservation-apis.md) (API de automatización de reservas de Azure).

-   [Reservation Details API](https://docs.microsoft.com/rest/api/consumption/reservationsdetails): consulte la información sobre las reservas de máquinas virtuales adquiridas anteriormente, como cuánto consumo se ha reservado y cuánto se ha utilizado realmente. Puede consultar datos con detalles de cada máquina virtual. Para más información, consulte [APIs for Azure reservation automation](../reservations/reservation-apis.md) (API de automatización de reservas de Azure).

-   [Reservation Summaries API](https://docs.microsoft.com/rest/api/consumption/reservationssummaries): vea información agregada sobre las reservas de máquina virtual que adquirió su organización, como cuánto consumo está reservado en comparación con la cantidad usada del agregado. Para más información, consulte [APIs for Azure reservation automation](../reservations/reservation-apis.md) (API de automatización de reservas de Azure).

-   [Usage Details API](https://docs.microsoft.com/rest/api/consumption/usagedetails): obtenga información sobre los cargos y el uso de todos los recursos de Azure en Microsoft. La información presenta la forma de registros de detalles de uso, que actualmente se emiten una vez por cada medidor al día. Puede utilizar la información para sumar los costos de todos los recursos o investigar los costos o el uso de recursos específicos.

-   [RateCard API](/previous-versions/azure/reference/mt219005(v=azure.100)): obtenga las tasas de los medidores si es un cliente de Web Direct. Posteriormente, puede usar la información devuelta con la información de uso de los recursos para calcular manualmente la factura esperada.

-   [Unrated Usage API](/previous-versions/azure/reference/mt219003(v=azure.100)): obtenga información sobre el uso sin procesar antes de que Azure realice cualquier medición o aplique algún cargo.

### <a name="billing"></a>Facturación
-   [Billing Periods API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-billing-periods): determine un período de facturación para analizar, junto con el identificador de la factura de ese período. Puede usar los identificadores de las facturas con Invoices API.

-   [Invoices API](/rest/api/billing/2019-10-01-preview/invoices): obtenga la dirección URL de descarga de una factura para un período de facturación en formato PDF.

### <a name="enterprise-consumption"></a>Consumo de Enterprise
Las siguientes API son solo para Enterprise:

-   [Balance Summary API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-balance-summary): obtenga un resumen mensual de la información sobre los saldos, las nuevas compras, los gastos de los servicios de Azure Marketplace, los ajustes y los gastos de uso por encima del límite. Puede obtener esta información para el período de facturación actual o cualquier período en el pasado. Las empresas pueden utilizar estos datos para compararlos con los cargos del resumen calculados manualmente. Esta API no proporciona información específica de recursos o una vista agregada de los costos.

-   [Usage Details API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-usage-detail): obtenga información sobre el uso de Azure (en ofertas de Microsoft) para el mes actual, un período de facturación específico o un período de fechas personalizado. Las empresas pueden usar estos datos para calcular manualmente la factura en función de la velocidad y el consumo. Las empresas también pueden utilizar la información de departamento u organización para los costos de atributo entre organizaciones. Los datos proporcionan una vista específica de recursos de uso/costo.

-   [Marketplace Store Charge API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-marketplace-storecharge): obtenga información sobre el uso de Azure (en ofertas de asociados) para el mes actual, un período de facturación específico o un período de fechas personalizado. Las empresas pueden usar estos datos para calcular manualmente la factura en función de la velocidad y el consumo. Las empresas también pueden utilizar la información de departamento u organización para los costos de atributo entre organizaciones. Esta API proporciona una vista específica de recursos de uso/costo.

-   [Price Sheet API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-pricesheet): obtenga la tasa aplicable de cada medidor para la inscripción y el período de facturación determinados. Puede usar esta información de tasa junto con los detalles de uso y la información de uso de Marketplace para calcular manualmente la factura esperada.

-   [Billing Periods API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-billing-periods): obtenga una lista de períodos de facturación. La API también proporciona una propiedad que apunta a la ruta de la API para los cuatro conjuntos de datos de la API de Enterprise que pertenecen a dicho período de facturación: BalanceSummary, UsageDetails, Marketplace Charges y PriceSheet.

-   [Reserved Instance Recommendations API](https://docs.microsoft.com/rest/api/billing/enterprise/billing-enterprise-api-reserved-instance-recommendation): examine el uso de las máquinas virtuales correspondiente a 7 días, 30 días o 60 días y obtenga recomendaciones de compra única y compartida. Puede usar esta API para analizar las cantidades de compra recomendadas y los ahorros en costos esperados. Para más información, consulte [APIs for Azure reservation automation](../reservations/reservation-apis.md) (API de automatización de reservas de Azure).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="whats-the-difference-between-the-enterprise-reporting-apis-and-the-consumption-apis-when-should-i-use-each"></a>¿Cuál es la diferencia entre Enterprise Reporting APIs y Consumption APIs? ¿Cuándo se debe usar cada una?
Estas API tienen un conjunto similar de funcionalidades y pueden responder al mismo conjunto amplio de preguntas en el ámbito de administración de costos y facturación. Se dirigen a distintas audiencias:

- Enterprise Reporting APIs están disponibles para los clientes que han firmado un Contrato Enterprise con Microsoft que les concede acceso a compromisos monetarios negociados y precios personalizados. Las API necesitan una clave que se obtiene de [Enterprise Portal](https://ea.azure.com). Para obtener una descripción de estas API, vea [Información general de API de informes para clientes de Enterprise](enterprise-api.md).

- Consumption APIs están disponibles para todos los clientes, con algunas excepciones. Para más información, consulte [Información general sobre la API de consumo de Azure](consumption-api-overview.md) y la [referencia de la API de consumo de Azure](https://docs.microsoft.com/rest/api/consumption/). Se recomiendan las API proporcionadas como la solución para los escenarios de desarrollo más recientes.

### <a name="whats-the-difference-between-the-usage-details-api-and-the-usage-api"></a>¿Cuál es la diferencia entre Usage Details API y Usage API?
Estas API ofrecen básicamente distintos datos:

- [Usage Details API](https://docs.microsoft.com/rest/api/consumption/usagedetails) proporciona información sobre los costos y el uso de Azure por instancia del medidor. Los datos proporcionados ya han pasado por el sistema de medición de costos de Azure y se les habían aplicado costos junto con otros posibles cambios:

   - Cambios que tienen en cuenta el uso de compromisos monetarios de prepago
   - Cambios que tienen en cuenta las discrepancias de uso detectadas por Azure

- [Usage API](/previous-versions/azure/reference/mt219003(v=azure.100)) proporciona información de uso de Azure sin procesar antes de pasar por el sistema de medición de costos de Azure. Estos datos no pueden tener ninguna correlación con la cantidad de uso o el importe de los cargos que se ven después del sistema de medición de cargos de Azure.

### <a name="whats-the-difference-between-the-invoice-api-and-the-usage-details-api"></a>¿Cuál es la diferencia entre Invoice API y Usage Details API?
Estas API ofrecen una vista diferente de los mismos datos:

- [Invoice API](/rest/api/billing/2019-10-01-preview/invoices) es solamente para los clientes de Web Direct. Proporciona una acumulación mensual de su factura según los cargos agregados por cada tipo de medidor.

- [Usage Details API](https://docs.microsoft.com/rest/api/consumption/usagedetails) proporciona una vista detallada de los registros de uso y costo diarios. La pueden usar tanto clientes Enterprise como Web Direct.

### <a name="whats-the-difference-between-the-price-sheet-api-and-the-ratecard-api"></a>¿Cuál es la diferencia entre Price Sheet API y RateCard API?
Estas API ofrecen conjuntos similares de datos, pero tienen distintas audiencias:

- [Price Sheet API](https://docs.microsoft.com/rest/api/consumption/pricesheet) proporciona los precios personalizados que se han negociado para un cliente Enterprise.

- [RateCard API](/previous-versions/azure/reference/mt219005(v=azure.100)) proporciona los precios públicos que se aplican a los clientes de Web Direct.

## <a name="next-steps"></a>Pasos siguientes

- Para información sobre el uso de API de Azure para consultar información detallada mediante programación, consulte [Información general sobre la API de consumo de Azure](consumption-api-overview.md) e [Información general sobre la API de facturación](usage-rate-card-overview.md).

- Para comparar la factura con el archivo de uso diario detallado y los informes de administración de costos en Azure Portal, consulte [Comprender la factura de Microsoft Azure](../understand/review-individual-bill.md).

- Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).
