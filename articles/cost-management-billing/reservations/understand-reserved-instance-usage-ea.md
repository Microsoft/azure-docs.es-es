---
title: Información sobre el uso de Azure Reservations para Contratos Enterprise
description: Aprenda a interpretar los datos de uso para comprender cómo se aplican las reservas de Azure a las inscripciones Enterprise.
author: bandersmsft
ms.reviewer: yashar
tags: billing
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 02/13/2020
ms.author: banders
ms.openlocfilehash: 20a9fb6a158134ffc18dc5bbb7eddd34d2b79562
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682015"
---
# <a name="get-enterprise-agreement-reservation-costs-and-usage"></a>Obtención del uso y los costos de reservas de Contrato Enterprise

Los datos de uso y costos de reservas están disponibles para los clientes con Contrato Enterprise en Azure Portal y API REST. Este artículo le ayudará a realizar las siguientes acciones:

- Obtener los datos de compra de la reserva.
- Saber qué suscripción, grupo de recursos o recurso usó la reserva
- Anular la utilización de la reserva.
- Calcular el ahorro de la reserva.
- Obtener los datos de infrautilización de la reserva.
- Amortización de los costos de la reserva

Los cargos de Marketplace se consolidan en datos de uso. Consulte los cargos de uso de la primera entidad, el uso de Marketplace y las compras desde un único origen de datos.

## <a name="reservation-charges-in-azure-usage-data"></a>Cargos de reserva en los datos de uso de Azure

Los datos se dividen en dos conjuntos de datos independientes: _Costo real_ y _Costo amortizado_. Diferencias entre estos dos conjuntos de datos:

**Costo real** : proporciona datos para conciliar con la factura mensual. Estos datos tienen costos de compra de reserva y detalles de aplicación de reserva. Con estos datos, puede saber qué suscripción o grupo de recursos o recurso ha recibido el descuento de reserva en un día concreto. El valor EffectivePrice para el uso que recibe el descuento de la reserva es cero.

**Costo amortizado**: este conjunto de datos es similar al conjunto de datos de Costo real, salvo que el valor EffectivePrice para el uso que obtiene el descuento de reserva es el costo prorrateado de la reserva (en lugar de ser cero). Esto le ayuda a conocer el valor monetario del consumo de reserva de una suscripción, un grupo de recursos o un recurso, y puede ayudarle realizar cargos al usuario por el uso de la reserva internamente. El conjunto de datos también tiene horas de reserva sin usar. El conjunto de archivos no tiene registros de compra de reservas.

Comparación de dos conjuntos de datos:

| data | Conjunto de datos de costo real | Conjunto de datos de costo amortizado |
| --- | --- | --- |
| Compras de la reserva | Está disponible en esta vista.<br><br>  Para obtener estos datos, filtre según ChargeType = &quot;Compra&quot;. <br><br> Consulte ReservationID o ReservationName para saber para qué es el cargo de la reserva.  | No es aplicable a esta vista. <br><br> Los costos de compra no se proporcionan en los datos amortizados. |
| EffectivePrice | El valor es cero para el uso que obtiene el descuento de la reserva. | El valor es el costo prorrateado por hora de la reserva para el uso que tiene el descuento de la reserva. |
| Reservas sin usar (proporciona el número de horas que no se usó la reserva en un día y el valor monetario de los residuos) | No es aplicable en esta vista. | Está disponible en esta vista.<br><br> Para obtener estos datos, filtre según ChargeType = &quot;UnusedReservation&quot;.<br><br>  Consulte ReservationID o ReservationName para saber qué reserva estaba infrautilizada. Se trata de la cantidad de la reserva que se desperdició durante el día.  |
| UnitPrice (precio del recurso de la hoja de precios) | Disponible | Disponible |

Otra información disponible en los datos de uso de Azure ha cambiado:

- Información de productos y medidor: Azure no reemplaza el medidor consumido originalmente con el valor de ReservationId y ReservationName, ya que lo hizo anteriormente.
- ReservationId y ReservationName: son sus propios campos de los datos. Anteriormente, solían estar disponibles solo en AdditionalInfo.
- ProductOrderId: identificador de pedido de reserva, agregado como su propio campo.
- ProductOrderName: nombre del producto de la reserva adquirida.
- Término: 12 meses o 36 meses.
- RINormalizationRatio: disponible en AdditionalInfo. Esta es la relación donde la reserva se aplica al registro de uso. Si la flexibilidad de tamaño de instancia está habilitada para la reserva, puede aplicarse a otros tamaños. El valor muestra la relación a la que se aplicó la reserva para el registro de uso.

[Consulte la definición del campo](https://docs.microsoft.com/rest/api/consumption/usagedetails/list#definitions)

## <a name="get-azure-consumption-and-reservation-usage-data-using-api"></a>Obtención de datos de uso de reserva y consumo de Azure mediante API

Puede obtener los datos mediante la API o descargarlos desde Azure Portal.

Para obtener los datos nuevos, llame a [Usage Details API](/rest/api/consumption/usagedetails/list). Para obtener más información acerca de la terminología, consulte los [términos de uso](../understand/understand-usage.md). El autor de la llamada debe ser un administrador de Enterprise del Contrato Enterprise mediante [EA Portal](https://ea.azure.com). Los administradores de Enterprise de solo lectura también pueden obtener los datos.

Tenga en cuenta que estos datos no están disponibles en [Reporting APIs for Enterprise customers - Usage Details](/rest/api/billing/enterprise/billing-enterprise-api-usage-detail) (API de informes para clientes de Enterprise: Usage Details).

Este es un ejemplo de llamada a Usage Details API:

```
https://management.azure.com/providers/Microsoft.Billing/billingAccounts/{enrollmentId}/providers/Microsoft.Billing/billingPeriods/{billingPeriodId}/providers/Microsoft.Consumption/usagedetails?metric={metric}&amp;api-version=2019-05-01&amp;$filter={filter}
```

Para obtener más información acerca de {enrollmentId} y {billingPeriodId}, vea el artículo de la API [Usage Details – List](https://docs.microsoft.com/rest/api/consumption/usagedetails/list) (Detalles de uso: lista).

La información de la tabla siguiente sobre la métrica y los filtros puede ayudar a resolver problemas comunes de reserva.

| **Tipo de datos de API** | Acción de llamada a la API |
| --- | --- |
| **Todos los cargos (uso y compras)** | Reemplace {metric} con ActualCost |
| **Uso que obtuvo el descuento de reserva** | Reemplace {metric} con ActualCost<br><br>Reemplace {filter} con properties/reservationId%20ne%20 |
| **Uso que no obtuvo el descuento de reserva** | Reemplace {metric} con ActualCost<br><br>Reemplace {filter} con properties/reservationId%20eq%20 |
| **Cargos amortizados (uso y compras)** | Reemplace {metric} con AmortizedCost |
| **Informe de reserva sin usar** | Reemplace {metric} con AmortizedCost<br><br>Reemplace {filter} con properties/ChargeType%20eq%20'UnusedReservation' |
| **Compras de la reserva** | Reemplace {metric} con ActualCost<br><br>Reemplace {filter} con properties/ChargeType%20eq%20'Purchase'  |
| **Reembolsos** | Reemplace {metric} con ActualCost<br><br>Reemplace {filter} con properties/ChargeType%20eq%20'Refund' |

## <a name="download-the-usage-csv-file-with-new-data"></a>Descarga del archivo CSV de uso con los nuevos datos

Si es administrador de EA, puede descargar el archivo CSV que contiene los nuevos datos de uso de Azure Portal. Estos datos no están disponibles en el portal de EA (ea.azure.com), debe descargar el archivo de uso de Azure Portal (portal.azure.com) para ver los nuevos datos.

En Azure Portal, vaya a [Administración de costos + facturación](https://portal.azure.com/#blade/Microsoft_Azure_Billing/ModernBillingMenuBlade/BillingAccounts).

1. Seleccione el nombre de la cuenta de facturación.
2. Haga clic en **Uso + cargos**.
3. Haga clic en **Descargar**.  
![Ejemplo que muestra dónde debe descargar el archivo de datos de uso CSV en Azure Portal](./media/understand-reserved-instance-usage-ea/portal-download-csv.png)
4. En **Descargar uso y cargos**, en **Usage Details Version 2** (Versión 2 de detalles del uso), seleccione **All Charges (usage and purchases)** [Todos los cargos (uso y compras)] y, a continuación, haga clic en Descargar. Repita los pasos para **Cargos amortizados (uso y compras)** .

Los archivos CSV que descarga contienen los costos reales y los costos amortizados.

## <a name="common-cost-and-usage-tasks"></a>Tareas comunes de uso y costo

Las secciones siguientes son las tareas comunes que la mayoría de los usuarios utiliza para ver sus datos de uso y costo de reserva.

### <a name="get-reservation-purchase-costs"></a>Obtención de los costos de compra de la reserva

Los costos de compra de reserva están disponibles en los datos de Costo real. Filtre por _ChargeType = Compra_. Consulte ProductOrderID para determinar para qué pedido de reserva es la compra.

### <a name="get-underutilized-reservation-quantity-and-costs"></a>Obtención de los costos y la cantidad de reserva infrautilizada

Obtenga los datos de costo amortizado y filtre por _ChargeType_ _= UnusedReservation_. De esta forma, obtiene el costo y la cantidad diaria de reserva sin usar. Puede filtrar los datos de una reserva o un pedido de reserva mediante los campos _ReservationId_ y _ProductOrderId_, respectivamente. Si una reserva era utiliza al 100%, el registro tiene una cantidad de 0.

### <a name="amortize-reservation-costs"></a>Amortización de los costos de la reserva

Obtenga los datos de Costo amortizado y filtre por un pedido de reserva mediante _ProductOrderID_ para obtener los costos amortizados diarios de una reserva.

### <a name="chargeback-for-a-reservation"></a>Anulación de una reserva

Puede anular el uso de una reserva para otras organizaciones por suscripción, grupos de recursos o etiquetas. Los datos de costo amortizado proporcionan el valor monetario de utilización de una reserva en los siguientes tipos de datos:

- Recursos (por ejemplo, una VM)
- Resource group
- Etiquetas
- Subscription

### <a name="get-the-blended-rate-for-chargeback"></a>Obtención de la tarifa combinada de anulación

Para determinar la tarifa combinada, obtenga los datos de los costos amortizados y agregue el costo total. En el caso de VM, puede usar la información de MeterName o ServiceType de los datos JSON de AdditionalInfo. Divida el coste total por la cantidad usada para obtener la tarifa combinada.

### <a name="audit-optimum-reservation-use-for-instance-size-flexibility"></a>Uso de reserva óptimo de auditoría para flexibilidad de tamaño de instancia

Multiplique la cantidad por _RINormalizationRatio_ de AdditionalInfo. Los resultados indican cuántas horas de uso de reserva se aplicaron al registro de uso.

### <a name="determine-reservation-savings"></a>Determinación del ahorro de la reserva

Obtenga los datos de los costos amortizados y filtre los datos para una instancia reservada. A continuación:

1. Obtenga los costos de pago por uso estimado. Multiplique el valor _UnitPrice_ por los valores de _cantidad_ para obtener los costos de pago por uso estimado, si no se ha aplicado el descuento de reserva al uso.
2. Obtenga los costos de la reserva. Sume los valores de _costo_ para obtener el valor monetario de lo que ha pagado por la instancia reservada. Se incluyen los costos utilizados y no utilizados de la reserva.
3. Reste los costos de reserva de los costos de pago por uso estimado para obtener el ahorro estimado.

## <a name="reservation-purchases-and-amortization-in-cost-analysis"></a>Compras de reserva y amortización en el análisis de costos

Los costos de reserva están disponibles en el [análisis de costos](https://aka.ms/costanalysis). De forma predeterminada, el análisis de costos muestra **Costo real**, que es cómo se mostrarán los costos en la factura. Para ver las compras de reserva desglosadas y asociadas a los recursos que usaron la ventaja, cambie a **Costo amortizado**:

![Ejemplo que muestra donde debe seleccionar el costo amortizado en análisis de costos](./media/understand-reserved-instance-usage-ea/portal-cost-analysis-amortized-view.png)

Agrupe por tipo de cargo para ver un desglose del uso, las compras y los reembolsos. También puede agrupar por reserva para un desglose de los costos de reserva y a petición. Recuerde que los únicos costos de reserva que verá en el caso del costo real serán las compras, pero, en el caso del costo amortizado, los costos se asignarán a los recursos individuales que usaron la ventaja. También verá un nuevo tipo de cargo **UnusedReservation** cuando se fije en el costo amortizado.

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de Azure Reservations, consulte los siguientes artículos:

- [¿Qué es Azure Reservations?](save-compute-costs-reservations.md)
- [Pago por adelantado de máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/windows/prepay-reserved-vm-instances.md)
- [Pago por adelantado de los recursos de proceso de SQL Database con capacidad reservada de Azure SQL Database](../../azure-sql/database/reserved-capacity-overview.md)
- [Administración de Azure Reservations](manage-reserved-vm-instance.md)
- [Información sobre cómo se aplica el descuento por la reserva](../manage/understand-vm-reservation-charges.md)
- [Información sobre el uso de reservas para suscripciones de pago por uso](understand-reserved-instance-usage.md)
- [Costos de software de Windows no incluidos con Reservations](reserved-instance-windows-software-costs.md)
