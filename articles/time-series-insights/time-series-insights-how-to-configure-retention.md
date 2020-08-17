---
title: Configuración de la retención en el entorno, Azure Time Series Insights | Microsoft Docs
description: Aprenda a configurar la retención en un entorno de Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.topic: conceptual
ms.date: 06/30/2020
ms.custom: seodec18
ms.openlocfilehash: 9ee06501134515d9369e98e724e55a66f040fffa
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86495132"
---
# <a name="configuring-retention-in-azure-time-series-insights-gen1"></a>Configuración de la retención en Azure Time Series Insights Gen1

En este artículo se describe cómo configurar el **Tiempo de retención de datos** y el **Comportamiento correspondiente a un exceso del límite de almacenamiento** en Azure Time Series Insights.

## <a name="summary"></a>Resumen

Cada entorno de Azure Time Series Insights tiene una opción para configurar el **tiempo de retención de los datos**. El valor abarca de 1 a 400 días. Los datos se eliminan según la capacidad de almacenamiento del entorno o la duración de la retención (1-400), lo que se agote antes.

Cada entorno de Azure Time Series Insights tiene una opción adicional de **Comportamiento correspondiente a un exceso del límite de almacenamiento**. Esta configuración controla el comportamiento de entrada y de purga cuando se alcanza la capacidad máxima de un entorno. Hay dos comportamientos para elegir:

- **Purgar datos antiguos** (valor predeterminado)
- **Pausar entradas**

Para obtener información detallada para comprender mejor estas opciones de configuración, consulte [Descripción de la retención de datos en Azure Time Series Insights](time-series-insights-concepts-retention.md).  

## <a name="configure-data-retention"></a>Configuración de la retención de datos

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Busque su entorno de Azure Time Series Insights existente. Seleccione **Todos los recursos** en el menú izquierdo de Azure Portal. Seleccione el entorno de Azure Time Series Insights.

1. En el encabezado **Configuración**, seleccione **Configuración de almacenamiento**.

    [![En Configuración, seleccione Configuración de almacenamiento](media/data-retention/configure-data-retention.png)](media/data-retention/configure-data-retention.png#lightbox)

1. Seleccione el **tiempo de retención de los datos (en días)** para configurar el período de retención; use para ello la barra de control deslizante o escriba un número en el cuadro de texto.

1. Tenga en cuenta la opción de configuración **Capacidad**, ya que este valor afecta a la cantidad máxima de eventos de datos y la capacidad de almacenamiento total para almacenar los datos.

1. Alterne la opción de configuración **Comportamiento correspondiente a un exceso del límite de almacenamiento**. Seleccione el comportamiento **Purgar datos antiguos** o **Pausar entradas**.

    [![Pausar entradas: aceptar y guardar.](media/data-retention/pause-ingress-accept-and-save.png)](media/data-retention/pause-ingress-accept-and-save.png#lightbox)

1. Revise la documentación para conocer los posibles riesgos de pérdida de datos. Seleccione **Guardar** para configurar los cambios.

## <a name="next-steps"></a>Pasos siguientes

- Para más información, consulte [Descripción de la retención de datos en Azure Time Series Insights](time-series-insights-concepts-retention.md).

- Aprenda a [escalar el entorno de Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).

- Aprenda sobre el [planeamiento de su entorno](time-series-insights-environment-planning.md).
