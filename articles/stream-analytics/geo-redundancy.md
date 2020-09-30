---
title: Obtención de redundancia geográfica para trabajos de Azure Stream Analytics
description: En este artículo se describe cómo lograr la redundancia geográfica de trabajos de Azure Stream Analytics en lugar de la conmutación por error geográfica.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 08/26/2020
ms.openlocfilehash: 44b0394773485f8054eddc94c7d7e9b91baebc6d
ms.sourcegitcommit: e69bb334ea7e81d49530ebd6c2d3a3a8fa9775c9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88950824"
---
# <a name="achieve-geo-redundancy-for-azure-stream-analytics-jobs"></a>Obtención de redundancia geográfica para trabajos de Azure Stream Analytics

Azure Stream Analytics no proporciona conmutación por error geográfica automática, pero puede lograr redundancia geográfica implementando trabajos de Stream Analytics idénticos en varias regiones de Azure. Cada trabajo se conecta a una entrada local y a orígenes de salida local. La aplicación es la responsable de enviar datos de entrada a las dos entradas regionales y de realizar la conciliación entre las dos salidas regionales. Los trabajos de Stream Analytics son dos entidades independientes.

En el diagrama siguiente se muestra una implementación de trabajo de Stream Analytics con redundancia geográfica con la entrada del centro de eventos y la salida de Azure Database.

:::image type="content" source="media/geo-redundancy/geo-redundant-jobs.png" alt-text="diagrama de trabajos de Stream Analytics con redundancia geográfica":::

## <a name="primarysecondary-strategy"></a>Estrategia primaria y secundaria

La aplicación necesita administrar qué base de datos de salida de la región se considera la principal y cuál se considera la secundaria. En un error en una región primaria, la aplicación cambia a la base de datos secundaria y comienza a leer las actualizaciones de esa base de datos. El mecanismo real que permite minimizar las lecturas duplicadas depende de la aplicación. Puede simplificar este proceso escribiendo información adicional en la salida. Por ejemplo, puede agregar una marca de tiempo o un identificador de secuencia a cada salida para que la omisión de filas duplicadas sea una operación trivial. Una vez que se restaure la región primaria, se pone al día con la base de datos secundaria mediante una mecánica similar.

Aunque los distintos tipos de entrada y salida permiten diferentes opciones de replicación geográfica, se recomienda usar el patrón que se describe en este artículo para lograr redundancia geográfica, ya que proporciona flexibilidad y control para los productores de eventos y los consumidores de eventos.

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión y administración de los trabajos de Azure Stream Analytics con PowerShell](stream-analytics-monitor-and-manage-jobs-use-powershell.md)
* [Depuración controlada por datos en Azure Stream Analytics](stream-analytics-job-diagram-with-metrics.md)