---
title: 'Introducción a la ingesta: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Aprenda sobre la ingesta de datos en Azure Time Series Insights Gen2.
author: lyrana
ms.author: lyhughes
manager: deepakpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: seodec18
ms.openlocfilehash: 33cafd058e55951f7da4e925a603c2c442d4aed1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87077665"
---
# <a name="azure-time-series-insights-gen2-data-ingestion-overview"></a>Introducción a la ingesta de datos en Azure Time Series Insights Gen2

El entorno de Azure Time Series Insights Gen2 contiene un *motor de ingesta* para recopilar, procesar y almacenar datos de series temporales de streaming. A medida que los datos llegan a los orígenes de eventos, Azure Time Series Insights Gen2 consumirá y almacenará los datos casi en tiempo real.

[![Introducción a la ingesta](media/concepts-ingress-overview/ingress-overview.png)](media/concepts-ingress-overview/ingress-overview.png#lightbox)

## <a name="ingestion-topics"></a>Temas de ingesta de datos

Los siguientes artículos tratan a fondo del procesamiento de datos, incluidos los procedimientos recomendados que se deben seguir:

* Obtenga información sobre [orígenes de eventos](./concepts-streaming-ingestion-event-sources.md) y orientación sobre cómo seleccionar una marca de tiempo de origen de eventos.

* Revisar los [tipos de datos](./concepts-supported-data-types.md) admitidos

* Comprender cómo el motor de ingesta aplicará un conjunto de [reglas](./concepts-json-flattening-escaping-rules.md) a las propiedades JSON para crear las columnas de la cuenta de almacenamiento.

* Revise en el entorno las [limitaciones de rendimiento](./concepts-streaming-ingress-throughput-limits.md) para planificar sus necesidades de escala.

## <a name="next-steps"></a>Pasos siguientes

* Continúe para aprender sobre los [orígenes de eventos](./concepts-streaming-ingestion-event-sources.md) en el entorno de Azure Time Series Insights Gen2. 
