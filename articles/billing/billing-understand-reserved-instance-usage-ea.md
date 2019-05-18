---
title: Entender el uso de las reservas de Azure para contratos Enterprise
description: Aprenda a interpretar los datos de uso para comprender cómo se aplican las reservas de Azure a las inscripciones Enterprise.
author: bandersmsft
manager: yashar
tags: billing
ms.service: billing
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/07/2019
ms.author: banders
ms.openlocfilehash: 09242eaa6058229226062801f5f71f2bf4c7a9e8
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789387"
---
# <a name="get-enterprise-agreement-reservation-costs-and-usage"></a>Obtener el uso y costos de reserva de contrato Enterprise

Datos de uso y costos de reserva están disponibles para los clientes con contrato Enterprise en Azure portal y API de REST. Este artículo le ayuda:

- Obtener datos de compra de reserva
- Obtener datos de infrautilización de reserva
- Amortizar los costos de reserva
- Anulación de la utilización de la reserva
- Calcular el ahorro de reserva

Los cargos de Marketplace se consolidan en datos de uso. Ver los cargos de uso de la primera entidad, el uso de marketplace y compras desde un único origen de datos.

## <a name="reservation-charges-in-azure-usage-data"></a>Cargos de reserva en los datos de uso de Azure

Datos se dividen en dos conjuntos de datos independientes: _Costo real_ y _Amortized Cost_. ¿Cómo se diferencian estos dos conjuntos de datos:

**Costo real** : proporciona datos para conciliar con su factura mensual. Estos datos tienen reserva los costos de compra. Tiene cero EffectivePrice para el uso que recibió el descuento de reserva.

**Amortizar el costo** -el recurso EffectiveCost que obtiene el descuento de reserva es el costo de la instancia reservada prorrateado. El conjunto de datos también tiene costos de reserva sin usar. La suma del costo de reserva y no utilizado reserva proporciona el costo amortizado diario de la reserva.

Comparación de dos conjuntos de datos:

| Datos | Conjunto de datos de costo real | Conjunto de datos de costo amortizado |
| --- | --- | --- |
| Compras de reserva | Está disponible en esta vista.<br>  Para obtener este filtro de datos en ChargeType = &quot;compra&quot;. <br> Consulte ReservationID o ReservationName saber qué es el cargo de reserva.  | No es aplicable a esta vista. <br> Los costos de compra no se proporcionan en datos amortizados. |
| effectivePrice | El valor es cero para el uso que se obtiene el descuento de reserva. | El valor es el costo de con un prorrateo por hora de la reserva para uso con el descuento de reserva. |
| Reservas sin usar (proporciona el número de horas que no se usaron la reserva en un día y el valor monetario de los residuos) | No es aplicable en esta vista. | Está disponible en esta vista.<br> Para obtener estos datos, filtre por ChargeType = &quot;UnusedReservation&quot;.<br>  Consulte ReservationID o ReservationName saber qué reserva se infrautilizados. Se trata de la cantidad de la reserva se desperdicia en para el día.  |
| UnitPrice (precio del recurso de la hoja de precios) | Disponible | Disponible |

Otra información disponible en los datos de uso de Azure ha cambiado:

- Información de productos y medidor - Azure no reemplaza el medidor consumido originalmente con el valor de ReservationId y ReservationName, tal como lo hizo anteriormente.
- ReservationId y ReservationName - son sus propios campos de los datos. Anteriormente, solían estar disponibles solo en AdditionalInfo.
- ProductOrderId - el identificador de pedido de reserva, agregado como su propio campo.
- ProductOrderName - el nombre del producto de la reserva adquirida.
- Término - 12 meses o 36 meses.
- RINormalizationRatio - disponible bajo AdditionalInfo. Ésta es la relación donde la reserva se aplica al registro de uso. Si la flexibilidad de tamaño de instancia está habilitada en para la reserva, puede aplicar a otros tamaños. El valor se muestra la relación que se aplicó la reserva para el registro de uso.

## <a name="get-azure-consumption-and-reservation-usage-data-using-api"></a>Obtengan datos de uso de reserva y el consumo de Azure mediante API

Puede obtener los datos mediante la API o descargarlo desde Azure portal.

Se llama a la [API de detalles de uso](/rest/api/consumption/usagedetails/list) con la versión de API &quot;2019-04-01-preview&quot; para obtener los nuevos datos. Para obtener más información acerca de la terminología, consulte [términos de uso](billing-understand-your-usage.md). El llamador debe ser un administrador de empresa para el contrato empresarial mediante el [portal EA](https://ea.azure.com). Los administradores de organización de solo lectura también puede obtener los datos.

Los datos no están disponibles en [API de informes para clientes de Enterprise - detalles de uso](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail).

Este es un ejemplo de llamada a la API:

```
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{enrollmentId}/providers/Microsoft.Billing/billingPeriods/{billingPeriodId}/providers/Microsoft.Consumption/usagedetails?metric={metric}&amp;api-version=2019-04-01-preview&amp;$filter={filter}
```

Para obtener más información acerca de {enrollmentId} y {billingPeriodId}, vea el [lista obtener detalles de uso](https://docs.microsoft.com/rest/api/consumption/usagedetails/list) artículo de la API.

Información en la tabla siguiente sobre la métrica y filtro puede ayudar a resolver problemas comunes de reserva.

| **Tipo de datos de API** | Acción de llamada de API |
| --- | --- |
| **Todos los cargos (uso y las compras)** | Reemplace {metric} ActualCost |
| **Uso de la que se obtuvo el descuento de reserva** | Reemplace {metric} ActualCost<br>Reemplace {filter} con: properties/reservationId%20ne%20 |
| **Uso que no obtuvieron el descuento de reserva** | Reemplace {metric} ActualCost<br>Reemplace {filter} con: properties/reservationId%20eq%20 |
| **Cargos amortizados (uso y las compras)** | Reemplace {metric} AmortizedCost |
| **Informe de reserva sin usar** | Reemplace {metric} AmortizedCost<br>Reemplace {filter} con: properties/ChargeType%20eq%20'UnusedReservation' |
| **Compras de reserva** | Reemplace {metric} por {filter} ActualCostReplace por: properties/ChargeType%20eq%20'Purchase'  |
| **Reembolsos** | Reemplace {metric} ActualCost<br>Reemplace {filter} con: properties/ChargeType%20eq%20'Refund' |

## <a name="download-the-usage-csv-file-with-new-data"></a>Descargue el archivo CSV de uso con nuevos datos

Si es un administrador EA, puede descargar el archivo CSV que contiene los nuevos datos de uso de Azure portal. Estos datos no están disponibles desde el [portal EA](https://ea.azure.com).

En el portal de Azure, vaya a [administración de costos + facturación](https://portal.azure.com/#blade/Microsoft_Azure_Billing/ModernBillingMenuBlade/BillingAccounts).

1. Seleccione la cuenta de facturación.
2. Haga clic en **uso + cargos**.
3. Haga clic en **Descargar**.  
![Ejemplo que muestra dónde debe descargar el archivo de datos de uso CSV en el portal de Azure](./media/billing-understand-reserved-instance-usage-ea/portal-download-csv.png)
4. En **descargar uso + cargos** , en **detalles del uso de la versión 2** , seleccione **todos los cargos (uso y las compras)** y, a continuación, haga clic en descargar. Repita el proceso para **amortizado cargos (uso y las compras)**.

Los archivos CSV que descarga contienen los costos reales y los costos amortizados.

## <a name="common-cost-and-usage-tasks"></a>Tareas comunes de uso y costo

Las secciones siguientes son las tareas comunes que se suele usa para ver sus datos de uso y costo de reserva.

### <a name="get-reservation-purchase-costs"></a>Obtener los costos de compra de reserva

Los costos de compra de reserva están disponibles en los datos de costo real. Filtrar por _ChargeType = compra_. Consulte ProductOrderID para determinar qué es la compra de pedido de reserva.

### <a name="get-underutilized-reservation-quantity-and-costs"></a>Obtener la cantidad de reserva infrautilizados y los costos

Obtener datos de costo amortizado y filtre por _ChargeType_ _= UnusedReservation_. Obtiene la cantidad diaria de reserva sin usar y el costo. Puede filtrar los datos de una reserva o un pedido de reserva con _ReservationId_ y _ProductOrderId_ campos, respectivamente. Si una reserva era utiliza al 100%, el registro tiene una cantidad de 0.

### <a name="amortize-reservation-costs"></a>Amortizar los costos de reserva

Obtener datos de costo amortizado y filtro para un pedido de reserva con _ProductOrderID_ obtener amortizados diarios para una reserva.

### <a name="chargeback-for-a-reservation"></a>Anulación de una reserva

Puede usar de reserva de anulación a otras organizaciones por suscripción, grupos de recursos o etiquetas. Datos de costo amortizado proporcionan valor monetario de utilización de una reserva en los siguientes tipos de datos:

- Recursos (por ejemplo, una máquina virtual)
- Grupos de recursos
- `Tags`
- Subscription

### <a name="get-the-blended-rate-for-chargeback"></a>Obtener la velocidad combinada de anulación

Para determinar la velocidad combinada, obtener los datos de los costos amortizados y el costo total de agregado. Las máquinas virtuales, puede usar la información de nombre del medidor o ServiceType datos AdditionalInfo JSON. Dividir el coste total por la cantidad utilizada para obtener la velocidad combinada.

### <a name="audit-optimum-reservation-use-for-instance-size-flexibility"></a>Reserva de auditoría óptimo usar, por ejemplo, flexibilidad de tamaño

La cantidad múltiple con el _RINormalizationRatio_, desde AdditionalInfo. Los resultados indican cuántas horas de uso de reserva se aplicó al registro de uso.

### <a name="determine-reservation-savings"></a>Determinar los ahorros de reserva

Obtener los datos de los costos de Amortized y filtrar los datos para una instancia reservada. A continuación:

1. Calcule el costo de pago por uso estimado. Multiplique el _UnitPrice_ valor con _cantidad_ estimado de valores para obtener los costos de pago por uso, si no se ha aplicado el descuento de reserva para el uso.
2. Obtener los costos de reserva. Suma el _costo_ valores para obtener el valor monetario de lo que ha pagado por la instancia reservada. Incluye los costos utilizados y no utilizados de la reserva.
3. Restar los costos de reserva de costos estimados de pago por uso para obtener el ahorro estimado derivado.

## <a name="reservation-purchases-and-amortization-in-azure-cost-analysis"></a>Las compras de reserva y amortización en el análisis de costos de Azure

Costo de las instancias reservadas está disponible en [modo de vista previa de análisis de costos de Azure](https://preview.portal.azure.com/?feature.canmodifystamps=true&amp;microsoft_azure_costmanagement=stage2&amp;Microsoft_Azure_CostManagement_arm_canary=true&amp;Microsoft_Azure_CostManagement_apiversion=2019-04-01-preview&amp;Microsoft_Azure_CostManagement_amortizedCost=true#blade/Microsoft_Azure_CostManagement/Menu/costanalysis). De forma predeterminada, la vista de datos de costo es de costo real. Puede cambiar al costo amortizado. Este es un ejemplo.

![Ejemplo que muestra donde debe seleccionar el costo amortizado en análisis de costos](./media/billing-understand-reserved-instance-usage-ea/portal-cost-analysis-amortized-view.png)

Aplicar filtros para ver los cargos por un tipo de reserva o la batería. Agrupar por nombre de reserva para ver los costos desglosados por las reservas de direcciones.

## <a name="need-help-contact-us"></a>¿Ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [crear una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

- [¿Qué es Azure Reservations?](billing-save-compute-costs-reservations.md)
- [Pago por adelantado de máquinas virtuales con Azure Reserved VM Instances](../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Pago por adelantado de los recursos de proceso de SQL Database con capacidad reservada de Azure SQL Database](../sql-database/sql-database-reserved-capacity.md)
- [Administración de Azure Reservations](billing-manage-reserved-vm-instance.md)
- [Información sobre cómo se aplica el descuento por la reserva](billing-understand-vm-reservation-charges.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](billing-understand-reserved-instance-usage.md)
- [Costos de software de Windows no incluidos con Reservations](billing-reserved-instance-windows-software-costs.md)
