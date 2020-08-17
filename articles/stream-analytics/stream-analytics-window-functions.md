---
title: Introducción a las funciones de ventana de Azure Stream Analytics
description: En este artículo se describen cuatro funciones de ventana (saltos de tamaño constante, salto, deslizante y sesión) que se usan en los trabajos de Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 07/10/2020
ms.openlocfilehash: fd741a9401a3936ec02939562e8e85046e829d31
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87075941"
---
# <a name="introduction-to-stream-analytics-windowing-functions"></a>Introducción a las funciones de ventana de Stream Analytics

En los escenarios de streaming en tiempo real, realizar operaciones en los datos contenidos en ventanas temporales es un patrón común. Stream Analytics ofrece compatibilidad nativa para las funciones de ventana, lo que permite a los desarrolladores crear trabajos de procesamiento de flujo complejos con un mínimo esfuerzo.

Hay cuatro tipos de ventanas temporales para elegir: ventanas de [**saltos de tamaño constante**](https://docs.microsoft.com/stream-analytics-query/tumbling-window-azure-stream-analytics), de [**salto**](https://docs.microsoft.com/stream-analytics-query/hopping-window-azure-stream-analytics), [**deslizante**](https://docs.microsoft.com/stream-analytics-query/sliding-window-azure-stream-analytics) y de [**sesión**](https://docs.microsoft.com/stream-analytics-query/session-window-azure-stream-analytics).  Utilice las funciones de ventana en la cláusula [**GROUP BY**](https://docs.microsoft.com/stream-analytics-query/group-by-azure-stream-analytics) de la sintaxis de consulta en los trabajos de Stream Analytics. También puede agregar eventos a través de varias ventanas mediante la función [**Windows()** ](https://docs.microsoft.com/stream-analytics-query/windows-azure-stream-analytics).

Todas las operaciones de [ventana](https://docs.microsoft.com/stream-analytics-query/windowing-azure-stream-analytics) generan resultados al **final** de la ventana. La salida de la ventana será un solo evento basado en la función agregada que se usa. El evento de salida tendrá la marca de tiempo del final de la ventana y todas las funciones de ventana están definidas con una longitud fija. 

![Conceptos de las funciones de ventana de Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-conceptual.png)

## <a name="tumbling-window"></a>Ventana de saltos de tamaño constante
Las funciones de ventana de saltos de tamaño constante se usan para segmentar una transmisión de datos en segmentos de tiempo distintos y realizar una función con ellos, como en el ejemplo siguiente. Los diferenciadores clave de una ventana de saltos de tamaño constante son que se repiten, no se superponen y un evento no puede pertenecer a más de una ventana de saltos de tamaño constante.

![Ventana de saltos de tamaño constante de Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-tumbling-intro.png)

## <a name="hopping-window"></a>Ventana de salto
Las funciones de ventana de salto saltan hacia adelante en el tiempo un período fijo. Es fácil pensar en ellas como ventanas de salto que se pueden superponer, por lo que los eventos pueden pertenecer a más de un conjunto de resultados de ventana de salto. Para hacer que una ventana de salto sea igual que una ventana de saltos de tamaño constante, especifique el tamaño de salto de forma que coincida con el tamaño de la ventana. 

![Ventana de salto de Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-hopping-intro.png)

## <a name="sliding-window"></a>Ventana deslizante

Las ventanas deslizantes, a diferencia de las ventanas de salto o de salto de tamaño constante, solo generan eventos para puntos en el tiempo cuando el contenido de la ventana cambia realmente. En otras palabras, cuando un evento entra o sale de la ventana. Cada ventana tiene al menos un evento, como en el caso de las ventanas de salto, y los eventos pueden pertenecer a más de una ventana deslizante.

![Ventana deslizante de Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-sliding-intro.png)

## <a name="session-window"></a>Ventana de sesión
Las funciones de ventana de sesión agrupan eventos que llegan a la misma hora, filtrando los periodos en los que no hay ningún dato. Tiene tres parámetros principales: tiempo de espera, duración máxima y clave de partición (opcional).

![Ventana de sesión de Stream Analytics](media/stream-analytics-window-functions/stream-analytics-window-functions-session-intro.png)

Una ventana de sesión se inicia cuando se produce el primer evento. Si se produce otro evento en el tiempo de espera especificado desde el último evento ingerido, la ventana se amplía para incluir el nuevo evento. En caso contrario, si no se produce ningún evento en el tiempo de espera, se cierra la ventana en el tiempo de espera.

Si se siguen produciendo eventos en el tiempo de espera especificado, la ventana de sesión se sigue ampliando hasta que se alcanza la duración máxima. Los intervalos de comprobación de la duración máxima se establecen para que tengan el mismo tamaño que la duración máxima especificada. Por ejemplo, si la duración máxima es 10, las comprobaciones, si la ventana supera la duración máxima, se producirán en t = 0, 10, 20, 30, etc.

Cuando se proporciona una clave de partición, los eventos se agrupan por clave, y la ventana de sesión se aplica independientemente a cada grupo. Esta creación de particiones es útil para los casos donde son necesarias distintas ventanas de sesión para distintos usuarios o dispositivos.

## <a name="snapshot-window"></a>Ventana de instantánea

Las ventanas de instantánea agrupan los eventos que tienen la misma marca de tiempo. A diferencia de otros tipos de ventanas, que requieren una función de ventana específica (como [SessionWindow()](https://docs.microsoft.com/stream-analytics-query/session-window-azure-stream-analytics), puede aplicar una ventana de instantánea si agrega System.Timestamp() a la cláusula GROUP BY.

![Ventana de instantánea de Stream Analytics](media/stream-analytics-window-functions/snapshot.png)

## <a name="next-steps"></a>Pasos siguientes
* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalado de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)

