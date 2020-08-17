---
title: Descripción de la retención de datos en el entorno, Azure Time Series Insights | Microsoft Docs
description: En este artículo se describen dos parámetros de configuración que controlan la retención de datos en el entorno Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: jasonh, kfile
ms.workload: big-data
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: seodec18
ms.openlocfilehash: 005725acf7270ff87ac9418f27941bdb205ae986
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87059423"
---
# <a name="understand-data-retention-in-azure-time-series-insights-gen1"></a>Descripción de la retención de datos en Azure Time Series Insights Gen1

En este artículo se describen dos parámetros de configuración principales que tienen un impacto en la retención de datos en el entorno Azure Time Series Insights.

## <a name="video"></a>Vídeo

### <a name="the-following-video-summarizes-azure-time-series-insights-data-retention-and-how-to-plan-for-itbr"></a>En el vídeo siguiente se resume la retención de datos de Azure Time Series Insights y cómo planearla.</br>

> [!VIDEO https://www.youtube.com/embed/03x6zKDQ6DU]

Cada uno de los entornos de Azure Time Series Insights tiene una configuración que controla el **tiempo de retención de los datos**. El valor abarca de 1 a 400 días. Los datos se eliminan según la capacidad de almacenamiento del entorno o la duración de la retención, lo que se agote antes.

Además, el entorno de Azure Time Series Insights tiene una configuración de **comportamiento superado de límite de almacenamiento**. Controla el comportamiento de entrada y de purga cuando se alcanza la capacidad máxima de un entorno. Para configurarlo se puede elegir entre dos comportamientos:

- **Purgar datos antiguos** (valor predeterminado)  
- **Pausar entradas**

> [!NOTE]
> De forma predeterminada, al crear un nuevo entorno, el período de retención está configurado en **Purgar datos antiguos**. Esta opción se puede activar o desactivar según sea necesario después de la creación mediante Azure Portal, en la página **Configure** (Configurar) del entorno de Azure Time Series Insights.
> * Para información sobre cómo configurar las directivas de retención, lea [Configuración de la retención en Azure Time Series Insights](time-series-insights-how-to-configure-retention.md).

A continuación se describen con más detalle ambas directivas de retención de datos.

## <a name="purge-old-data"></a>Purgar datos antiguos

- **Purgar datos antiguos** es la configuración predeterminada para los entornos de Azure Time Series Insights.  
- **Purgar datos antiguos** es la opción preferida cuando los usuarios quieren tener siempre los *datos más recientes* en su entorno de Azure Time Series Insights.
- La configuración **Purgar datos antiguos** *purga* los datos una vez que se alcanzan los límites del entorno (tiempo de retención, tamaño o cantidad, lo que llegue antes). La retención está establecida en 30 días de forma predeterminada.
- Los datos ingeridos hace más tiempo se purgan primero (enfoque "primero en entrar, primero en salir").

### <a name="example-one"></a>Ejemplo 1

Piense ahora en un entorno de ejemplo con un comportamiento de retención en el modo de **continuar entrada y purgar datos antiguos**:

El **tiempo de retención de datos** está configurado en un valor de 400 días. La **capacidad** es una unidad S1, que contiene 30 GB de capacidad total. Supongamos que los datos entrantes acumulan 500 MB cada día por término medio. Este entorno solo puede retener datos de 60 días dada la tasa de datos entrantes, ya que la capacidad máxima se alcanza a los 60 días. Los datos entrantes se acumulan de este modo: 500 MB cada día x 60 días = 30 GB.

Al día 61.º, el entorno muestra los datos más actuales, pero purga los más antiguos (los que tengan más de 60 días). La purga genera espacio para el flujo entrante de nuevos datos, de manera que puedan seguir explorándose nuevos datos. Si el usuario desea retener los datos durante más tiempo, puede aumentar el tamaño del entorno mediante la adición de unidades adicionales o bien insertar menos datos.  

### <a name="example-two"></a>Ejemplo 2

Piense ahora en un entorno que tiene configurado también el comportamiento de retención en el modo de **continuar entrada y purgar datos antiguos**. En este ejemplo, el **tiempo de retención de datos** está configurado e en un valor inferior de 180 días. La **capacidad** es una unidad S1, que contiene 30 GB de capacidad total. Con el fin de almacenar los datos de los 180 días completos, la entrada diaria no puede ser superior a 0,166 GB (166 MB) por día.  

En caso de que la tasa de entrada diaria del entorno sea superior a 0,166 GB por día, los datos no se podrán almacenar durante 180 días, ya que algunos datos se purgarán. Piense en este mismo entorno durante un período de tiempo ocupado. Supongamos que la tasa de entrada del entorno puede aumentar hasta un promedio de 0,189 por día. En ese período de tiempo ocupado, se retienen unos 158 días de datos (30GB/0,189 = 158,73 días de retención). Este tiempo es inferior al periodo de tiempo de retención de datos deseado.

## <a name="pause-ingress"></a>Pausar entradas

- Este valor **Pausar entradas** está diseñado para velar por que no se purguen los datos si los límites de tamaño y cantidad se alcanzan con anterioridad a su período de retención.  
- **Pausar entradas** proporciona más tiempo para que los usuarios aumenten la capacidad de su entorno antes de que los datos se purguen una vez que se supere el periodo de retención.
- Ayuda a proteger contra la pérdida de datos, pero puede generar una oportunidad para la pérdida de los datos más recientes si la entrada se pausa una vez alcanzado el periodo de retención del origen del evento.
- Pero una vez alcanzada la capacidad máxima de un entorno, el entorno pausa la entrada de datos hasta que ocurran las acciones adicionales siguientes:

   - Se aumenta la capacidad máxima del entorno para agregar más unidades de escalado, como se describe en [Escalado del entorno de Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).
   - Se alcanza el período de retención de datos y estos se purgan, poniendo al entorno por debajo de su capacidad máxima.

### <a name="example-three"></a>Ejemplo 3

Piense en un entorno con el comportamiento de retención configurado en **pausar entradas**. En este ejemplo, el **período de retención de datos** está configurado en 60 días. La **capacidad** se establece en tres (3) unidades de S1. Supongamos que este entorno tiene una entrada de datos de 2 GB cada día. En este entorno, la entrada se pausa una vez que se alcance la capacidad máxima.

En ese momento, el entorno muestra el mismo conjunto de datos hasta que se reanude la entrada o hasta que se habilite **continuar entrada** (lo que purgaría los datos más antiguos para dejar sitio a los nuevos).

Cuando se reanuda la entrada:

- Los datos fluyen en el orden en que se recibieron por el origen del evento.
- Los eventos se indexan en función de su marca de tiempo, a menos que se hayan superado las directivas de retención en el origen del evento. Para obtener más información sobre la configuración de la retención del origen del evento, consulte [Preguntas frecuentes sobre Event Hubs](../event-hubs/event-hubs-faq.md).

> [!IMPORTANT]
> Debe establecer alertas para proporcionar un aviso que ayude a evitar que el ingreso se pause. Es posible que se pierdan datos, ya que el tiempo predeterminado de retención es de 1 día para orígenes de eventos de Azure. Por lo tanto, cuando se pausa la entrada, es probable que pierdan los datos más recientes, a menos que se realice alguna acción adicional. Debe aumentar la capacidad, o cambiar el comportamiento a **Purgar datos antiguos** para evitar la posible pérdida de datos.

En los centros de eventos afectadas, considere la posibilidad de ajustar la propiedad **Retención de mensajes** para minimizar la pérdida de datos cuando se pausa la entrada en Azure Time Series Insights.

[![Retención de mensajes de Event Hub.](media/time-series-insights-concepts-retention/event-hub-retention.png)](media/time-series-insights-concepts-retention/event-hub-retention.png#lightbox)

Si no se configuran propiedades en el origen de eventos (`timeStampPropertyName`), Azure Time Series Insights se establece de manera predeterminada en la marca de tiempo de llegada al centro de eventos como el eje X. Si se configura `timeStampPropertyName` en algo diferente, el entorno busca el valor `timeStampPropertyName` configurado en el paquete de datos al analizar eventos.

Para escalar su entorno a fin de disponer de capacidad adicional o de aumentar la duración de la retención, lea [Escalado del entorno de Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).

## <a name="next-steps"></a>Pasos siguientes

- Para información sobre la configuración o el cambio de los valores de retención de datos, consulte [Configuración de la retención en Azure Time Series Insights](time-series-insights-how-to-configure-retention.md).

- Más información sobre [Mitigación de la latencia en Azure Time Series Insights](time-series-insights-environment-mitigate-latency.md).
