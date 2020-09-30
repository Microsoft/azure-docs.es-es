---
title: Recomendaciones de reservas de Azure
description: Obtenga información acerca de las recomendaciones de reservas de Azure.
author: banders
ms.author: banders
ms.reviewer: yashar
ms.service: cost-management-billing
ms.subservice: reservations
ms.topic: conceptual
ms.date: 09/02/2020
ms.openlocfilehash: 869009d7dd26685842da3c948fbdc058a38feb21
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89398433"
---
# <a name="reservation-recommendations"></a>Recomendaciones de reserva

Las recomendaciones de compra de instancias reservadas de Azure (RI) se proporcionan mediante la [API de recomendación de reservas](/rest/api/consumption/reservationrecommendations) de Azure Consumption, [Azure Advisor](../../advisor/advisor-cost-recommendations.md#buy-reserved-virtual-machine-instances-to-save-money-over-pay-as-you-go-costs) y a través de la experiencia de compra de reservas de Azure Portal.

En los pasos siguientes se define cómo se calculan las recomendaciones:

1. El motor de recomendaciones evalúa el uso por hora de los recursos en el ámbito determinado de los últimos 7, 30 y 60 días.
2. Según los datos de uso, el motor simula los costos con y sin reservas.
3. Los costos se simulan para distintas cantidades y se recomienda la cantidad que maximiza el ahorro.
4. Si los recursos se cierran con regularidad, la simulación no encontrará ningún ahorro y no se proporcionará ninguna recomendación de compra.
5. El cálculo de recomendaciones incluye todos los descuentos especiales que pueda tener en las tarifas de uso a petición.

## <a name="recommendations-in-the-azure-portal"></a>Recomendaciones en Azure Portal

Las recomendaciones para compra de reservas también se muestran en Azure Portal, en la experiencia de compra y aparecen con una **Cantidad recomendada**. Cuando se adquiere, la cantidad que recomienda Azure ofrece el máximo ahorro posible. Aunque puede comprar cualquier cantidad que quiera, si compra una cantidad diferente, su ahorro no será óptimo.

Veamos algunos ejemplos del porqué.

En la siguiente imagen de ejemplo, Azure recomienda comprar una cantidad de 6 instancias para la recomendación seleccionada.

:::image type="content" source="./media/reserved-instance-purchase-recommendations/recommended-quantity.png" alt-text="Ejemplo que muestra una recomendación de compra de reservas" lightbox="./media/reserved-instance-purchase-recommendations/recommended-quantity.png" :::

Al seleccionar el vínculo **Ver detalles** se muestra más información sobre la recomendación. En la imagen siguiente se muestran los detalles de la recomendación. La cantidad recomendada se calcula para el uso más elevado posible en función del uso histórico. Es posible que la recomendación no sea para un uso del 100 % si ha tenido un uso incoherente. En el ejemplo, observe que el uso ha fluctuado a lo largo del tiempo. Se muestran el costo de la reserva, el ahorro posible y el porcentaje de uso.

:::image type="content" source="./media/reserved-instance-purchase-recommendations/recommended-quantity-details.png" alt-text="Ejemplo que muestra los detalles de una recomendación de compra de reservas" :::

Al aumentar o disminuir la reserva de la cantidad por encima o por debajo de la recomendación, el gráfico y los valores estimados cambiarán. Al aumentar la cantidad de reserva, se reducirá el ahorro, ya que se infrautilizarán las reservas. En otras palabras, pagará por reservas que no se aprovecharán al máximo.

Si reduce la cantidad de reserva, también se reducirá el ahorro. Aunque habrá aumentado el uso, es probable que haya períodos en los que las reservas no cubran plenamente sus necesidades. El uso más allá de la cantidad de reserva se cubrirá con recursos de pago por uso más caros. En el ejemplo siguiente se ilustra este punto. Hemos reducido manualmente la cantidad de reserva a 4. Se aumenta el uso de la reserva, pero el ahorro general se reduce porque están presentes los costos de pago por uso.

:::image type="content" source="./media/reserved-instance-purchase-recommendations/recommended-quantity-details-changed.png" alt-text="Ejemplo que muestra los detalles de una recomendación de compra de reservas modificada" :::

Para maximizar el ahorro con las reservas, intente adquirir una cantidad de reservas lo más cercana posible a la recomendación.

## <a name="recommendations-in-azure-advisor"></a>Recomendaciones en Azure Advisor

Las recomendaciones de compra de reservas están disponibles en Azure Advisor. Tenga en cuenta los siguientes puntos:

- Advisor solo tiene recomendaciones de ámbito de una sola suscripción. Si desea ver recomendaciones de todo el ámbito de facturación (cuenta de facturación o perfil de facturación), vaya a Azure Portal > Reservas > Agregar y seleccionar el tipo cuyas recomendaciones desea ver.
- Recomendaciones disponibles en Advisor considerada la tendencia de uso de 30 días.
- La cantidad y ahorro de las recomendaciones son para una reserva de 3 años, siempre que esté disponible. Si no se vende una reserva de 3 años para el servicio, la recomendación se calculará según el precio de una reserva de 1 año.
- Los cálculos de recomendaciones incluyen todos los descuentos especiales que pueda tener en las tarifas de uso a petición.
- Si compra una reserva de ámbito compartido, las recomendaciones de compra de reservas de Advisor pueden tardar hasta 5 días en desaparecer.

## <a name="other-expected-api-behavior"></a>Otro comportamiento esperado de la API

- Si usa un período retrospectivo de siete días, es posible que no obtenga recomendaciones cuando las máquinas virtuales estén apagadas durante más de un día.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [Aplicación del descuento por reserva de Azure en las máquinas virtuales](../manage/understand-vm-reservation-charges.md).
