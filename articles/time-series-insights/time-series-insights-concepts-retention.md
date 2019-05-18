---
title: Descripción de la retención de datos en el entorno de Azure Time Series Insights | Microsoft Docs
description: En este artículo se describen dos parámetros de configuración que controlan la retención de datos en el entorno Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: ashannon7
ms.author: anshan
manager: cshankar
ms.reviewer: jasonh, kfile, anshan
ms.workload: big-data
ms.topic: conceptual
ms.date: 04/30/2019
ms.custom: seodec18
ms.openlocfilehash: 443599e1b2876012bcbdf720bef7762a24e1ff90
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790427"
---
# <a name="understand-data-retention-in-azure-time-series-insights"></a>Descripción de la retención de datos en Azure Time Series Insights

En este artículo se describe dos opciones de configuración que afectan a la retención de datos en su entorno Azure Time Series Insights.

## <a name="video"></a>Vídeo

### <a name="the-following-video-summarizes-time-series-insights-data-retention-and-how-to-plan-for-itbr"></a>El vídeo siguiente resume la retención de datos de Time Series Insights y cómo planear para él.</br>

> [!VIDEO https://www.youtube.com/embed/03x6zKDQ6DU]

Cada uno de los entornos de Azure Time Series tiene un valor que controla **tiempo de retención de datos**. El valor abarca de 1 a 400 días. Los datos se eliminan según la capacidad de almacenamiento de entorno o la duración de retención, lo que ocurra primero.

Además, el entorno de Azure Time Series tiene un **comportamiento ha superado el límite de almacenamiento** configuración. Controles de entrada y purgar el comportamiento cuando se alcanza la capacidad máxima de un entorno. Hay dos comportamientos para elegir cuando se configura:

- **Purgar datos antiguos** (valor predeterminado)  
- **Pausar entradas**

> [!NOTE]
> De forma predeterminada, al crear un nuevo entorno, el período de retención está configurado en **Purgar datos antiguos**. Esta configuración se puede activar o desactivar según sea necesario después de la hora de creación mediante Azure portal, en la **configurar** página del entorno de Time Series Insights.

Para obtener información acerca de cómo cambiar los comportamientos de retención, consulte [Configuring retention in Time Series Insights](time-series-insights-how-to-configure-retention.md) (Configuración de la retención en Time Series Insights).

Compare el comportamiento de retención de datos:

## <a name="purge-old-data"></a>Purgar datos antiguos

- Este comportamiento es el comportamiento predeterminado para los entornos de Time Series Insights.  
- Este comportamiento es preferible cuando los usuarios desean ver siempre sus *datos más recientes* en su entorno de Time Series Insights.
- Este comportamiento *purga* los datos una vez que se alcanzan los límites del entorno (tiempo de retención, tamaño o cantidad, lo que llegue antes). La retención está establecida en 30 días de forma predeterminada.
- Los datos introducidos hace más tiempo se purgan primero (enfoque FIFO [PEPS]).

### <a name="example-one"></a>Ejemplo uno

Piense ahora en un entorno de ejemplo con un comportamiento de retención en el modo de **continuar entrada y purgar datos antiguos**:

**Tiempo de retención de datos** está establecido en 400 días. La **capacidad** es una unidad S1, que contiene 30 GB de capacidad total.   Supongamos que los datos entrantes acumulan 500 MB cada día por término medio. Este entorno solo puede retener datos de 60 días dada la tasa de datos entrantes, ya que la capacidad máxima se alcanza a los 60 días. Los datos entrantes se acumulan de este modo: 500 MB cada día x 60 días = 30 GB.

El día 61, el entorno muestra los datos más recientes, pero purga los datos más antiguos, más de 60 días. La purga genera espacio para el flujo entrante de nuevos datos, de manera que puedan seguir explorándose nuevos datos. Si el usuario desea retener los datos durante más tiempo, puede aumentar el tamaño del entorno mediante la adición de unidades adicionales o bien insertar menos datos.  

### <a name="example-two"></a>Segundo ejemplo

Piense ahora en un entorno que tiene configurado también el comportamiento de retención en el modo de **continuar entrada y purgar datos antiguos**. En este ejemplo, el **tiempo de retención de datos** está configurado e en un valor inferior de 180 días. La **capacidad** es una unidad S1, que contiene 30 GB de capacidad total. Con el fin de almacenar los datos de los 180 días completos, la entrada diaria no puede ser superior a 0,166 GB (166 MB) por día.  

En caso de que la tasa de entrada diaria del entorno sea superior a 0,166 GB por día, los datos no se podrán almacenar durante 180 días, ya que algunos datos se purgarán. Piense en este mismo entorno durante un período de tiempo ocupado. Supongamos que la tasa de entrada del entorno puede aumentar hasta un promedio de 0,189 por día. En ese período de tiempo ocupado, se retienen unos 158 días de datos (30GB/0,189 = 158,73 días de retención). Este tiempo es inferior al periodo de tiempo de retención de datos deseado.

## <a name="pause-ingress"></a>Pausar entrada

- El **pausar entradas** está diseñado para garantizar que no se purgan los datos si se alcanzan los límites de tamaño y recuento antes de su período de retención.  
- **Pausar entradas** proporciona más tiempo para que los usuarios aumentar la capacidad de su entorno antes de purgan datos debido a que superan el período de retención
- Ayuda a protegerse frente a pérdidas de datos, pero puede crear una oportunidad para que la pérdida de los datos más recientes, si la entrada se pausa más allá del período de retención del origen del evento.
- Sin embargo, una vez que se alcanza la capacidad máxima del entorno, el entorno pausa la entrada de datos hasta que se producen las siguientes acciones adicionales:

   - Aumentar la capacidad máxima del entorno para agregar más unidades de escalado como se describe en [cómo escalar el entorno de Time Series Insights](time-series-insights-how-to-scale-your-environment.md).
   - Se alcanza el período de retención de datos y se purgan datos, llevar al entorno por debajo de su capacidad máxima.

### <a name="example-three"></a>Ejemplo 3

Piense en un entorno con el comportamiento de retención configurado en **pausar entradas**. En este ejemplo, el **período de retención de datos** está configurado en 60 días. **Capacidad** está establecido en tres (3) las unidades de S1. Supongamos que este entorno tiene una entrada de datos de 2 GB cada día. En este entorno, la entrada se pausa una vez que se alcance la capacidad máxima.

En ese momento, el entorno muestra el mismo conjunto de datos hasta que reanude la entrada o hasta que **continuar entrada** está habilitada (que desea purgar los datos más antiguos para dejar espacio para los nuevos datos).

Cuando se reanuda la entrada:

- Los datos fluyen en el orden en que se recibieron por el origen del evento.
- Los eventos se indexan en función de su marca de tiempo, a menos que se hayan superado las directivas de retención en el origen del evento. Para obtener más información sobre la configuración de la retención del origen del evento, consulte [Preguntas frecuentes sobre Event Hubs](../event-hubs/event-hubs-faq.md).

> [!IMPORTANT]
> Debe establecer alertas para proporcionar un aviso que ayude a evitar que el ingreso se pause. Es posible que se pierdan datos, ya que el tiempo predeterminado de retención es de 1 día para orígenes de eventos de Azure. Por lo tanto, cuando se pausa la entrada, es probable que pierdan los datos más recientes, a menos que se realice alguna acción adicional. Debe aumentar la capacidad, o cambiar el comportamiento a **Purgar datos antiguos** para evitar la posible pérdida de datos.

En los Event Hubs afectados, considere la posibilidad de ajustar la propiedad **Retención de mensajes** para minimizar la pérdida de datos cuando se pausa la entrada en Time Series Insights.

[![Retención de mensajes del centro de eventos.](media/time-series-insights-contepts-retention/event-hub-retention.png)](media/time-series-insights-contepts-retention/event-hub-retention.png#lightbox)

Si no hay propiedades se configuran en el origen del evento (`timeStampPropertyName`), Time Series Insights tiene como valor predeterminado la marca de tiempo de llegada al event hub como eje x. Si `timeStampPropertyName` está configurada para ser algo más, el entorno busca configurado `timeStampPropertyName` en el paquete de datos al analizar eventos.

Si necesita escalar su entorno para dar cabida a una capacidad adicional o para aumentar la duración de retención, vea [Escalado de su entorno de Time Series Insights](time-series-insights-how-to-scale-your-environment.md) para obtener más información.  

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre la configuración o cambiar la configuración de retención de datos, revise [configurar la retención en Time Series Insights](time-series-insights-how-to-configure-retention.md).
