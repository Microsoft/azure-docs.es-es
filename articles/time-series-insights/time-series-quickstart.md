---
title: 'Inicio rápido: Explorador de Azure Time Series Insights: Azure Time Series Insights | Microsoft Docs'
description: Más información sobre cómo empezar a trabajar con el explorador de Azure Time Series Insights. Visualice grandes volúmenes de datos de IoT y conozca las características clave de su entorno.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.topic: quickstart
ms.workload: big-data
ms.custom: mvc seodec18
ms.date: 06/30/2020
ms.openlocfilehash: 2140cbbc43af7f80c0c5fc2d7cea85cb31bd7bfe
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87059350"
---
# <a name="quickstart-explore-azure-time-series-insights-gen1"></a>Inicio rápido: Exploración de Azure Time Series Insights Gen1

Esta guía de inicio rápido del explorador de Azure Time Series Insights muestra cómo empezar a trabajar con Azure Time Series Insights en un entorno de demostración gratuito. Con él, aprenderá a usar el explorador web para visualizar grandes volúmenes de datos de IoT y recorrerá las características clave que están disponibles de forma general en la actualidad.

Azure Time Series Insights es un servicio de análisis, almacenamiento y administración totalmente administrado que facilita el análisis y la exploración de miles de millones de eventos de IoT de forma simultánea. Proporciona una visión global de los datos, con el fin de que se pueda validar rápidamente la solución de IoT y evitar el costoso tiempo de inactividad de los dispositivos críticos. Azure Time Series Insights facilita la detección de las tendencias ocultas, descubrir anomalías y realizar análisis de causa principal casi en tiempo real.

Para mayor flexibilidad, Azure Time Series Insights se puede agregar a una aplicación existente previamente mediante sus eficaces [API REST](./concepts-query-overview.md) y [SDK de cliente](https://github.com/microsoft/tsiclient). Las API se pueden usar para almacenar, consultar y consumir datos de series temporales de la aplicación cliente que prefiera. También se puede usar el SDK de cliente para agregar los componentes de la interfaz de usuario a la aplicación existente.

En esta guía de inicio rápido del explorador de Azure Time Series Insights se ofrece un recorrido guiado por las características.

> [!IMPORTANT]
> Si no tiene ninguna, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

## <a name="prepare-the-demo-environment"></a>Preparación del entorno de demostración

1. En el explorador, vaya a la [demo de Gen1](https://insights.timeseries.azure.com/demo).

1. Si se le solicita, inicie sesión en el explorador de Azure Time Series Insights mediante las credenciales de la cuenta de Azure.

1. Aparece una página con un recorrido rápido por Azure Time Series Insights. Seleccione **Siguiente** para comenzar el recorrido rápido.

   [![Bienvenida del inicio rápido: seleccione Siguiente](media/quickstart/quickstart-welcome.png)](media/quickstart/quickstart-welcome.png#lightbox)

## <a name="explore-the-demo-environment"></a>Exploración del entorno de demostración

1. Aparece el **panel de selección de hora**. Use este panel para seleccionar un período de tiempo que se va a visualizar.

   [![Panel de selección de hora](media/quickstart/quickstart-time-selection-panel.png)](media/quickstart/quickstart-time-selection-panel.png#lightbox)

1. Seleccione un período de tiempo y arrástrelo en la región. Luego, seleccione **Buscar**.

   [![Selección de un período de tiempo](media/quickstart/quickstart-select-time.png)](media/quickstart/quickstart-select-time.png#lightbox)

   Azure Time Series Insights muestra una visualización de gráfico para el período de tiempo que especificó. Puede realizar diversas acciones en el gráfico de líneas. Por ejemplo, puede filtrar, anclar, ordenar y apilar.

   Para volver al **panel de selección de hora**, seleccione la flecha hacia abajo como se muestra a continuación:

   [![Gráfico](media/quickstart/quickstart-select-down-arrow.png)](media/quickstart/quickstart-select-down-arrow.png#lightbox)

1. Seleccione **Agregar** en el **panel de términos** para agregar un término de búsqueda nuevo.

   [![Agregar el panel de términos de búsqueda](media/quickstart/quickstart-add-terms.png)](media/quickstart/quickstart-add-terms.png#lightbox)

1. En el gráfico, puede seleccionar una región, hacer clic con el botón derecho en la región y seleccionar **Explorar eventos**.

   [![Explorar eventos](media/quickstart/quickstart-explore-events.png)](media/quickstart/quickstart-explore-events.png#lightbox)

   Se muestra una cuadrícula de los datos sin procesar de la región que explora.

   [![Explorar eventos: vista de datos de cuadrícula](media/quickstart/quickstart-explore-events-grid-data.png)](media/quickstart/quickstart-explore-events-grid-data.png#lightbox)

## <a name="select-and-filter-data"></a>Seleccionar y filtrar datos

1. Edite los términos para cambiar los valores del gráfico. Agregue otro término para establecer una correlación entre distintos tipos de valores.

   [![Agregar un término](media/quickstart/quickstart-add-a-term.png)](media/quickstart/quickstart-add-a-term.png#lightbox)

1. Deje en blanco el cuadro **Filtrar serie** para ver todos los términos de búsqueda seleccionados o escriba un término de filtro en el cuadro **Filtrar serie** para el filtrado de series improvisado.

   [![Filtrar series](media/quickstart/quickstart-filter-series.png)](media/quickstart/quickstart-filter-series.png#lightbox)

   En esta guía de inicio rápido, escriba **Station5** para correlacionar la temperatura y la presión de esa estación.

Cuando termine la guía de inicio rápido, podrá experimentar con el conjunto de datos de ejemplo para crear visualizaciones distintas.

## <a name="clean-up-resources"></a>Limpieza de recursos

Ahora que ha completado el tutorial, limpie los recursos que ha creado:

1. En el menú de la izquierda de [Azure Portal](https://portal.azure.com), seleccione **Todos los recursos** y busque el grupo de recursos de Azure Time Series Insights.
1. Seleccione **Eliminar** para eliminar todo el grupo de recursos (y todos los recursos que contiene), o quite los recursos de uno en uno.

## <a name="next-steps"></a>Pasos siguientes

* Ya está listo para crear su propio entorno de Azure Time Series Insights. Consulte la [planeación del entorno de Azure Time Series Insights](time-series-insights-environment-planning.md).
