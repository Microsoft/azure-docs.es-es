---
title: 'Escalado del entorno: Azure Time Series Insights | Microsoft Docs'
description: Aprenda a escalar el entorno de Azure Time Series Insights mediante Azure Portal.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 06/30/2020
ms.custom: seodec18
ms.openlocfilehash: a552f03c8a8fa05ed7d2c6eb87374d4e7e17838d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87070086"
---
# <a name="how-to-scale-your-azure-time-series-insights-gen1-environment"></a>Escalado del entorno de Azure Time Series Insights Gen1

En este artículo se describe cómo cambiar la capacidad de su entorno de Azure Time Series Insights mediante [Azure Portal](https://portal.azure.com). La capacidad es el multiplicador que se aplica a la tasa de entrada, la capacidad de almacenamiento y el costo asociado a la SKU seleccionada.

Puede usar Azure Portal para aumentar o disminuir la capacidad en una SKU de precios determinada.

Sin embargo, el plan de tarifa de la SKU no se puede cambiar. Por ejemplo, un entorno con una SKU de precios de S1 no se puede convertir en una de S2 o viceversa.

## <a name="ga-limits"></a>Límites de la disponibilidad general

[!INCLUDE [Azure Time Series Insights GA limits](../../includes/time-series-insights-ga-limits.md)]

## <a name="change-the-capacity-of-your-environment"></a>Cambio de la capacidad del entorno

1. En Azure Portal, localice y seleccione el entorno de Azure Time Series Insights.

1. En el menú de su entorno de Azure Time Series Insights, seleccione **Configuración de almacenamiento**.

   [![Configuración de la capacidad de Azure Time Series Insights](media/scale-your-environment/scale-your-environment-configure.png)](media/scale-your-environment/scale-your-environment-configure.png#lightbox)

1. Utilice el control deslizante **Capacidad** para seleccionar la que cumpla los requisitos para la velocidad de entrada y la capacidad de almacenamiento. Tenga en cuenta la **velocidad de entrada**, **la capacidad de almacenamiento** y las actualizaciones de la **estimación de costos** dinámicamente para mostrar el efecto del cambio.

   [![Configuración del entorno con el control deslizante de capacidad](media/scale-your-environment/scale-your-environment-slider.png)](media/scale-your-environment/scale-your-environment-slider.png#lightbox)

   También puede escribir número del multiplicador de capacidad en el cuadro de texto de la derecha del control deslizante.

1. Seleccione **Guardar** para escalar el entorno. El indicador de progreso se muestra hasta que el cambio se confirma, un tiempo breve.

1. Compruebe que la nueva capacidad sea [suficiente para evitar la limitación](time-series-insights-diagnose-and-solve-problems.md).

## <a name="next-steps"></a>Pasos siguientes

- Para más información, consulte [Descripción de la retención de datos en Azure Time Series Insights](time-series-insights-concepts-retention.md).

- Aprenda a [configurar la retención de datos en Azure Time Series Insights](time-series-insights-how-to-configure-retention.md).

- Aprenda sobre el [planeamiento de su entorno](time-series-insights-environment-planning.md).
