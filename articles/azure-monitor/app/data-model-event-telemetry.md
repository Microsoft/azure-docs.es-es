---
title: 'Modelo de datos de telemetría de Azure Application Insights: telemetría de eventos | Microsoft Docs'
description: Modelo de datos de Application Insights para la telemetría de eventos
ms.topic: conceptual
ms.date: 04/25/2017
ms.reviewer: sergkanz
ms.openlocfilehash: 69685afa14352a22b58bccbea342038e4273696e
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320619"
---
# <a name="event-telemetry-application-insights-data-model"></a>Telemetría de eventos: Modelo de datos de Application Insights

Puede crear elementos de telemetría de eventos (en [Application Insights](./app-insights-overview.md)) para representar un evento producido en la aplicación. Normalmente es una interacción del usuario como un clic de botón o la desprotección de un pedido. También puede ser un evento de ciclo de vida de la aplicación como la actualización de la inicialización o la configuración. 

Semánticamente, los eventos pueden estar correlacionados o no con las solicitudes. Pero si se usa correctamente, la telemetría de eventos es más importante que las solicitudes o los seguimientos. Los eventos representan la telemetría de negocio y deberían estar sujetos a un [muestreo](./api-filtering-sampling.md) independiente, menos agresivo.

## <a name="name"></a>Nombre

Nombre del evento. Para permitir la agrupación adecuada y métricas útiles, restrinja la aplicación para que genere un pequeño número de nombres de evento independientes. Por ejemplo, no utilice un nombre diferente para cada instancia generada de un evento.

Longitud máxima: 512 caracteres

## <a name="custom-properties"></a>Propiedades personalizadas

[!INCLUDE [application-insights-data-model-properties](../../../includes/application-insights-data-model-properties.md)]

## <a name="custom-measurements"></a>Medidas personalizadas

[!INCLUDE [application-insights-data-model-measurements](../../../includes/application-insights-data-model-measurements.md)]

## <a name="next-steps"></a>Pasos siguientes

- Consulte [modelo de datos](data-model.md) para los tipos y el modelo de datos de Application Insights.
- [Escritura de telemetría de eventos personalizada](./api-custom-events-metrics.md#trackevent)
- Consulte las [plataformas](./platforms.md) compatibles con Application Insights.

