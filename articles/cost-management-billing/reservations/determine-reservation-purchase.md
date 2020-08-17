---
title: Determinación de qué reserva de Azure se debe comprar
description: Este artículo le ayuda a determinar qué reserva debe comprar.
author: bandersmsft
ms.reviewer: yashar
ms.service: cost-management-billing
ms.topic: how-to
ms.date: 08/04/2020
ms.author: banders
ms.openlocfilehash: 909d712e0c8e8ea242c27ab93f16a36e22d36aae
ms.sourcegitcommit: 5a37753456bc2e152c3cb765b90dc7815c27a0a8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87759907"
---
# <a name="determine-what-reservation-to-purchase"></a>Determinación de qué reserva comprar

Todas las reservas, excepto Azure Databricks, se aplican por hora. Las reservas se deben comprar en función del uso base constante. Hay varias maneras de determinar qué reservas comprar y este artículo le va a ayudar a ello.

La compra de una capacidad superior al uso histórico da lugar a una reserva infrautilizada. Siempre que sea posible, debe evitarlo. La capacidad reservada no utilizada no se acumula de una hora a la siguiente. El uso por encima de la cantidad reservada se cobra con las tarifas de pago por uso más costosas.

## <a name="analyze-usage-data"></a>Análisis de los datos de uso

Utilice las secciones siguientes como ayuda para analizar los datos de uso diario a fin de determinar el uso de línea base y la reserva que se va a comprar.

### <a name="analyze-usage-for-a-vm-reserved-instance-purchase"></a>Análisis de uso para la compra de instancias reservadas de máquina virtual

Identifique el tamaño correcto de la máquina virtual para la compra. Por ejemplo, si se compra una reserva para máquinas virtuales de la serie ES, no se aplica a las máquinas virtuales de la serie E y viceversa.

Las máquinas virtuales de la serie promocional no obtienen un descuento por reserva, por lo que debe quitarlas del análisis.

Para ceñirse al uso de las máquinas virtuales válidas, aplique los siguientes filtros en los datos de uso:

- Filtre **MeterCategory** por **Máquinas virtuales**.
- Obtenga la información de **ServiceType** de **AdditionalInfo**. La información sugiere el tamaño correcto de la máquina virtual. Por ejemplo, E32 estándar.
- Use el campo **ResourceLocation** para determinar el centro de datos de uso.

Omita los recursos que tengan menos de 24 horas de uso en un día.

Si quiere basar el análisis en el nivel de la familia de tamaños de instancia, puede obtener los valores de flexibilidad de tamaño de instancia en [https://isfratio.blob.core.windows.net/isfratio/ISFRatio.csv](https://isfratio.blob.core.windows.net/isfratio/ISFRatio.csv). Combine los valores con los datos para realizar el análisis. Para más información sobre la flexibilidad de tamaño de instancia, vea [Flexibilidad en el tamaño de las máquinas virtuales con Azure Reserved VM Instances](../../virtual-machines/windows/reserved-vm-instance-size-flexibility.md).

### <a name="analyze-usage-for-an-azure-synapse-analytics-reserved-instance-purchase"></a>Análisis del uso de la compra de una instancia reservada de Azure Synapse Analytics

La capacidad reservada se aplica a los precios de DWU de Azure Synapse Analytics. No se aplica al costo de la licencia de Azure Synapse Analytics ni a ningún otro costo que no sea un proceso.

Para ceñirse al uso válido, aplique los filtros siguientes a los datos de uso:


- Filtre **MeterCategory** por **SQL Database**.
- Filtre **MeterName** por **Núcleo virtual**.
- Filtre **MeterSubCategory** por todos los registros de uso que tengan _Proceso_ en el nombre.

En **AdditionalInfo**, obtenga el valor de **Núcleos virtuales**. Este valor indica el número de núcleos virtuales que se usaron. La cantidad de **Núcleos virtuales** se multiplica por el número de horas que se usó la base de datos.

Los datos le informan sobre el uso constante de:

- Combinación de tipos de bases de datos. Por ejemplo, instancia administrada o grupo elástico por base de datos única.
- Nivel de servicio. Por ejemplo, uso general o crítico para la empresa.
- Generación. Por ejemplo, Gen 5.
- Ubicación del recurso

### <a name="analysis-for-azure-synapse-analytics"></a>Análisis de Azure Synapse Analytics

La capacidad reservada se aplica al uso de DWU de Azure Synapse Analytics y se compra en incrementos de 100 DWU. Para ceñirse al uso válido, aplique los filtros siguientes a los datos de uso:

- Filtre **MeterName** por **100 DWU**.
- Filtre **Subcategoría de medición** por **Compute Optimized Gen2** (Optimizado para proceso Gen 2).

Use el campo **Ubicación de recursos** para determinar el uso de Azure Synapse Analytics en una región.

El uso de Azure Synapse Analytics se puede escalar o reducir verticalmente a lo largo del día. Hable con el equipo que administró la instancia de Azure Synapse Analytics para conocer el uso base.

Vaya a Reservas en Azure Portal y compre la capacidad reservada de Azure Synapse Analytics en múltiplos de 100 DWU.

## <a name="reservation-purchase-recommendations"></a>Recomendaciones de compra de reservas

Las recomendaciones de compra de reservas se calculan mediante el análisis de los datos de uso por hora en los últimos 7, 30 y 60 días. Azure calcula cuáles hubieran sido los costos si tuviera una reserva y los compara con los costos de pago por uso reales en los que se incurre a lo largo del tiempo. El cálculo se realiza para cada cantidad que se ha usado durante el período de tiempo. Se recomienda la cantidad que maximice el ahorro.

Por ejemplo, puede que la mayoría de las veces use 500 máquinas virtuales, pero que en ocasiones el uso ascienda a 700 máquinas virtuales. En este ejemplo, Azure calcula el ahorro para las cantidades de 500 y 700 máquinas virtuales. Dado que el uso de 700 máquinas virtuales es esporádico, el cálculo de la recomendación determina que el ahorro se maximiza para una compra de reserva de 500 máquinas virtuales y se proporciona la recomendación para la cantidad de 500.

Tenga en cuenta los siguientes puntos:

- Las recomendaciones de reserva se calculan mediante las tarifas de uso a petición que se aplican en su caso.
- Las recomendaciones se calculan para tamaños individuales, no para la familia de tamaños de la instancia.
- La cantidad recomendada para un ámbito se reduce el mismo día en que se compran las reservas para este.
    - Sin embargo, una actualización para la recomendación de la cantidad de reserva entre ámbitos puede tardar hasta 25 días. Por ejemplo, si compra según las recomendaciones de ámbito compartido, las recomendaciones de ámbito de una sola suscripción pueden tardar hasta 25 días en ajustarse.

## <a name="recommendations-in-the-azure-portal"></a>Recomendaciones en Azure Portal

Las compras de reservas calculadas por el motor de recomendaciones se muestran en la pestaña **Recomendado** de [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Reservations/CreateBlade/referrer/docs). A continuación se muestra una imagen de ejemplo.

![Imagen que muestra recomendaciones](./media/determine-reservation-purchase/select-product-ri.png)

Más información sobre las [recomendaciones](reserved-instance-purchase-recommendations.md#recommendations-in-the-azure-portal).

## <a name="recommendations-in-the-cost-management-power-bi-app"></a>Recomendaciones en la aplicación de Power BI de Cost Management

Los clientes de Contrato Enterprise y de Contrato de cliente de Microsoft pueden usar los informes de cobertura de instancia reservada de máquina virtual y las recomendaciones de compra. Los informes de cobertura muestran el uso total y el uso que está cubierto por las instancias reservadas.

1. Obtenga la [aplicación Cost Management](https://appsource.microsoft.com/product/power-bi/costmanagement.azurecostmanagementapp).
2. Vaya al informe de cobertura de instancia reservada de máquina virtual, ámbito único o compartido, en función del ámbito en el que desee comprar.
3. Seleccione la región y la familia de tamaños de instancia para ver el uso, la cobertura de instancia reservada y la recomendación de compra del filtro seleccionado.

## <a name="recommendations-in-azure-advisor"></a>Recomendaciones en Azure Advisor

Las recomendaciones de compra de reservas están disponibles en [Azure Advisor](https://portal.azure.com/#blade/Microsoft_Azure_Expert/AdvisorMenuBlade/overview).

- Advisor solo tiene recomendaciones de ámbito de una sola suscripción.
- Las recomendaciones de Advisor se calculan con el período retrospectivo de 30 días. El ahorro previsto es para un período de reserva de 3 años.
- Si adquiere una reserva de ámbito compartido, las recomendaciones de compra de reservas de Advisor pueden tardar hasta 30 días en desaparecer.

## <a name="recommendations-using-apis"></a>Recomendaciones mediante API

Use las [recomendaciones de reserva](/rest/api/consumption/reservationrecommendations/list) de la API REST para ver las recomendaciones mediante programación.

## <a name="next-steps"></a>Pasos siguientes

- [Administración de Azure Reservations](manage-reserved-vm-instance.md)
- [Información sobre el uso de reservas de Azure para suscripciones de pago por uso](understand-reserved-instance-usage.md)
- [Información sobre el uso de reservas para la inscripción Enterprise](understand-reserved-instance-usage-ea.md)
- [Costos de software de Windows no incluidos con reservas](reserved-instance-windows-software-costs.md)