---
title: 'Visualización de datos en el Explorador de TSI: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Aprenda sobre las características y opciones disponibles en el Explorador de Azure Time Series Insights.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 08/27/2020
ms.custom: seodec18
ms.openlocfilehash: 31ed5f93fc7d1b44ae8a4afb1f4368d0b52eb2fd
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89227056"
---
# <a name="azure-time-series-insights-explorer"></a>Explorador de Azure Time Series Insights

En este artículo se describen las distintas características y opciones disponibles en el [entorno de demostración](https://insights.timeseries.azure.com/preview/demo) de Azure Time Series Insights Gen2.

## <a name="prerequisites"></a>Requisitos previos

Para empezar a usar el Explorador de Azure Time Series Insights, debe hacer lo siguiente:

* Tener aprovisionado un entorno de Azure Time Series Insights Gen2. Lea el tutorial de [Azure Time Series Insights Gen2](./time-series-insights-update-create-environment.md) para más información sobre el aprovisionamiento de una instancia.
* [Proporcionar acceso a datos](./time-series-insights-data-access.md) al entorno de Azure Time Series Insights Gen2 que creó para la cuenta. Puede proporcionar acceso al resto de usuarios y a usted mismo.
* Agregar un origen de eventos al entorno de Azure Time Series Insights Gen2 para insertar datos en el entorno:
  * Aprenda a [conectarse a un centro de eventos](./time-series-insights-how-to-add-an-event-source-eventhub.md)
  * Aprenda a [conectarse a un centro de IoT](./time-series-insights-how-to-add-an-event-source-iothub.md)

## <a name="explore-the-azure-time-series-insights-explorer"></a>Explore el Explorador de Azure Time Series Insights

El Explorador de Azure Time Series Insights consta de los siguientes siete elementos:

[![Introducción al Explorador de Azure Time Series Insights](media/v2-update-explorer/preview-explorer-overview.png)](media/v2-update-explorer/preview-explorer-overview.png#lightbox)

1. [Panel de entornos](#1-environment-panel): Muestra los entornos de Azure Time Series Insights Gen2.
1. [Barra de navegación](#2-navigation-bar): Le permite cambiar entre las páginas **Analizar** y **Modelo**.
1. [Árbol de jerarquía y panel de búsqueda](#3-hierarchy-tree-and-search-panel): Le permite seleccionar y buscar los elementos de datos específicos que se van a agregar al gráfico.
1. [Fuente de serie temporal](#4-time-series-well): Muestra los elementos de datos seleccionados actualmente.
1. [Panel del gráfico](#5-chart-panel): muestra el gráfico de trabajo actual.
1. [Escala de tiempo](#6-time-editor-panel): le permite modificar el intervalo de tiempo de trabajo.
1. [Barra de la aplicación](#7-app-bar): Contiene las opciones de administración de usuario, como el inquilino actual; además le permite cambiar la configuración del tema y el idioma.

## <a name="1-environment-panel"></a>1. Panel de entornos

El panel de entornos muestra todos los entornos de Azure Time Series Insights Gen2 a los que tiene acceso. La lista incluye tanto entornos de Gen2 como entornos de Gen1. Solo tiene que seleccionar el entorno que quiera usar para que se le dirija inmediatamente.

1. Seleccione la flecha desplegable situada junto a su entorno mostrado.

   [![Panel de entornos](media/v2-update-explorer/environment-panel.png)](media/v2-update-explorer/environment-panel.png#lightbox)

1. A continuación, seleccione el entorno que desee.

## <a name="2-navigation-bar"></a>2. Barra de navegación

  [![La barra de navegación](media/v2-update-explorer/tsi-preview-navigation-bar.png)](media/v2-update-explorer/tsi-preview-navigation-bar.png#lightbox)

Use la barra de navegación para seleccionar entre dos vistas:

* **Analizar**: se usa para trazar y realizar análisis exhaustivos en sus datos de serie temporal modelados o sin modelar.
* **Modelo**: se usa para insertar nuevos tipos, jerarquías e instancias de Azure Time Series Insights Gen2 en el modelo de serie temporal.

### <a name="model-authoring"></a>Creación de modelos

Azure Time Series Insights Gen2 admite operaciones completas de creación, lectura, actualización y eliminación (CRUD) en el modelo de serie temporal.

[![Panel de búsqueda de modelos](media/v2-update-explorer/model-search-panel.png)](media/v2-update-explorer/model-search-panel.png#lightbox)

* **Tipo de modelo de serie temporal**: puede usar tipos de modelos de serie temporal para definir variables o fórmulas para realizar cálculos. Dichos tipos están asociados a una instancia determinada del modelo de serie temporal. Un tipo puede tener una o más variables.
* **Jerarquía del modelo de serie temporal**: Las jerarquías son modos de organizar sus datos de forma sistemática. Las jerarquías representan las relaciones entre diferentes instancias del modelo de serie temporal.
* **Instancia del modelo de serie temporal**: Las instancias son la serie temporal en sí mismas. En la mayoría de los casos, son **DeviceID** o **AssetID**, que funcionan como el identificador único del recurso en el entorno.

Para obtener más información sobre el modelo de serie temporal, lea los [Modelos de serie temporal](./concepts-model-overview.md).

## <a name="3-hierarchy-tree-and-search-panel"></a>3. Árbol de jerarquía y panel de búsqueda

El árbol de jerarquía y el panel de búsqueda le permiten buscar y navegar fácilmente por su jerarquía de [modelos de Time Series](./concepts-model-overview.md) a fin de buscar las instancias de serie temporal específicas que quiera mostrar en su gráfico. Cuando selecciona las instancias, estas se agregan tanto al gráfico actual como a los datos.

[![Árbol de jerarquía y panel de búsqueda](media/v2-update-explorer/tsi-preview-explorer-hierarchy-search.png)](media/v2-update-explorer/tsi-preview-explorer-hierarchy-search.png#lightbox)

El panel Resultados de la búsqueda también le permite ver los resultados en una vista de jerarquía o en una vista de lista, lo que facilita la búsqueda de las instancias que desea mostrar.

## <a name="4-time-series-well"></a>4. Cuadro de serie temporal

El cuadro muestra los campos de instancia y otros metadatos asociados con las instancias del modelo de serie temporal seleccionadas. Mediante la selección de las casillas del lado derecho, puede ocultar o mostrar instancias específicas del gráfico actual.

  [![Cuadro de Gen2](media/v2-update-explorer/tsi-preview-ui-explorer-well.png)](media/v2-update-explorer/tsi-preview-ui-explorer-well.png#lightbox)

También puede eliminar elementos de datos específicos de su cuadro de datos actual seleccionando el control **Eliminar** (papelera de reciclaje) de color rojo a la izquierda del elemento. El cuadro le permite controlar el modo en que se muestra cada elemento en el gráfico. Puede optar por agregar sombras mínimas/máximas, puntos de datos, desplazar el elemento en el tiempo y visualizar la instancia de forma escalonada.

Además, el control exploraciones le permite crear fácilmente turnos de tiempo y gráficos de dispersión.  

  [![Opciones de diseño del cuadro](media/v2-update-explorer/well-layout-options.png)](media/v2-update-explorer/well-layout-options.png#lightbox)

> [!NOTE]
> Si se muestra el siguiente mensaje, quiere decir que la instancia no tiene ningún dato durante el intervalo de tiempo seleccionado. Para resolver el problema, aumente el intervalo de tiempo o confirme que la instancia está enviando datos.
>
> ![Sin notificación de datos](media/v2-update-explorer/tsi-preview-no-data-warning.png)

## <a name="5-chart-panel"></a>5. Panel del gráfico

El gráfico le permite mostrar las instancias de serie temporal como líneas. Asimismo, puede contraer el panel de entorno, el modelo de datos y el panel de control de intervalos de tiempo haciendo clic en los controles web para agrandar el gráfico.

  [![Introducción al gráfico de Gen2](media/v2-update-explorer/tsi-preview-chart-overview.png)](media/v2-update-explorer/tsi-preview-chart-overview.png#lightbox)

1. **Tipo de gráfico**: controla qué elementos de datos están disponibles para su visualización.

1. **Tamaño del intervalo**: La herramienta de control deslizante Interval Size (Tamaño del intervalo) permite acercar y alejar los intervalos del mismo intervalo de tiempo. Esto proporciona un control más preciso del movimiento entre intervalos de tiempo de gran tamaño que muestran tendencias uniformes hasta en intervalos tan pequeños como un milisegundo, lo que permite revisar segmentos granulares y de alta resolución de los datos. El punto inicial predeterminado del control deslizante está establecido como la vista más óptima de los datos de la selección, lo que permite equilibrar la resolución, la velocidad de la consulta y la granularidad.

1. **Acercar y desplazamiento lateral**: Seleccione este control para acercarse y desplazarse lateralmente por el gráfico.

1. **Control del eje Y**: recorre las opciones de vista del eje Y disponibles:

    * `Stacked`: cada línea tiene un eje Y individual.
    * `Overlap`: Úselo para apilar varias líneas en el mismo eje Y para que los datos del eje Y cambien en función de la línea seleccionada.
    * `Shared`: Todos los datos del eje Y se muestran juntos.

1. **Elemento de marcador**: es el elemento de datos seleccionado actualmente y sus detalles asociados.

Puede profundizar aún más en un segmento de datos específico si hace **clic con el botón izquierdo** en un punto de datos del gráfico actual, lo mantiene presionado y arrastra luego el área seleccionada al punto final que usted elija. **Haga clic con el botón derecho** en el área seleccionada de color azul y, a continuación, seleccione el **Zoom** tal como se muestra. También puede ver y descargar los eventos de telemetría en el intervalo de tiempo seleccionado.

  [![Zoom del gráfico de Gen2](media/v2-update-explorer/preview-chart-zoom.png)](media/v2-update-explorer/preview-chart-zoom.png#lightbox)

Después de realizar la acción de **Zoom**, se mostrará el conjunto de datos seleccionado. Seleccione el control de formato para desplazarse por las tres representaciones del eje Y de sus datos.

  [![Eje Y del gráfico de Gen2](media/v2-update-explorer/tsi-preview-explorer-standard-chart.png)](media/v2-update-explorer/tsi-preview-explorer-standard-chart.png#lightbox)

A continuación se proporciona un ejemplo de **gráfico superpuesto**:

  [![Opción de gráfico superpuesto](media/v2-update-explorer/tsi-preview-explorer-overlapping-chart.png)](media/v2-update-explorer/tsi-preview-explorer-overlapping-chart.png#lightbox)

El botón **Más acciones** se expande para mostrar las opciones **Descargar como CSV**, **Conectarse a Power BI**, **Mostrar datos de gráfico como tabla** y **Explorar eventos sin formato**.

  [![Opción de Más acciones](media/v2-update-explorer/more-actions-icon.png)](media/v2-update-explorer/more-actions-icon.png#lightbox)

Más información sobre la opción **Conectarse a Power BI** del [conector de Power BI](concepts-power-bi.md).

## <a name="6-time-editor-panel"></a>6. Panel del editor de tiempo

Cuando trabaje con Azure Time Series Insights Gen2, primero debe seleccionar un intervalo de tiempo. El intervalo de tiempo seleccionado controla el conjunto de datos que está disponible para la manipulación con los widgets de actualización de Azure Time Series Insights Gen2.

  [![Panel de selección de hora](media/v2-update-explorer/tsi-preview-explorer-timeline-element.png)](media/v2-update-explorer/tsi-preview-explorer-timeline-element.png#lightbox)

> [!TIP]
> Una parte de la escala de tiempo se resalta en color ámbar o naranja para indicar el intervalo de datos que está disponible en el almacén semiactivo.

Los siguientes controles web están disponibles en Azure Time Series Insights Gen2 para que pueda seleccionar el intervalo de tiempo de funcionamiento.

  [![Control de cuadro de exploración](media/v2-update-explorer/exploration-well-control.png)](media/v2-update-explorer/exploration-well-control.png#lightbox)

1. **Control deslizante de intervalos de fechas externas**: use los dos controles de punto de conexión arrastrándolos sobre el intervalo de tiempo que quiera usar. El control deslizante del intervalo de fechas externas se encarga de restringir el intervalo de fechas internas.

1. **Aumentar y disminuir los botones de intervalo de fechas**: puede aumentar o disminuir el intervalo de tiempo si selecciona cualquiera de los botones del intervalo que quiera.

1. **Control para contraer los intervalos de tiempo**: este control web le permite ocultar todos los controles, excepto la herramienta de control deslizante de intervalos de fechas internas.

1. **Control deslizante de intervalos de fechas externas**: use los controles del punto de conexión para seleccionar el intervalo de fechas externas que estará disponible para el control de intervalos de fechas internas.

1. **Control deslizante de intervalo de tiempo**: use esta opción para cambiar rápidamente entre las selecciones de intervalos de tiempo preestablecidos, como los últimos **30 minutos**, las **últimas 12 horas** o un **intervalo personalizado**. Si cambia este valor también cambiará los intervalos disponibles que se describen en la herramienta de control deslizante para el tamaño de intervalos.

   [![Panel de selección de fecha inicial y de finalización](media/v2-update-explorer/to-and-from-element.png)](media/v2-update-explorer/to-and-from-element.png#lightbox)

## <a name="7-app-bar"></a>7. Barra de la aplicación

En la parte superior de la aplicación, aparece el panel de navegación de Azure Time Series Insights Gen2. Ofrece la siguiente funcionalidad:

### <a name="current-session-share-link-control"></a>Control de vínculo del recurso compartido de la sesión actual

  [![Icono Compartir](media/v2-update-explorer/tsi-preview-explorer-share-icon.png)](media/v2-update-explorer/tsi-preview-explorer-share-icon.png#lightbox)

Seleccione el nuevo icono **Compartir** para compartir un vínculo URL con su equipo.

  [![Uso compartido de la dirección URL de su instancia](media/v2-update-explorer/tsi-preview-explorer-share-your-view.png)](media/v2-update-explorer/tsi-preview-explorer-share-your-view.png#lightbox)

### <a name="tenant-section"></a>Sección del inquilino

  [![Selección de inquilino](media/v2-update-explorer/tsi-preview-explorer-tenant-selection.png)](media/v2-update-explorer/tsi-preview-explorer-tenant-selection.png#lightbox)

* Muestra la información actual de la cuenta de inicio de sesión Azure Time Series Insights Gen2.
* Úsela para cambiar entre los temas disponibles.
* Úsela para ver el [entorno de demostración](https://insights.timeseries.azure.com/preview/demo) de Gen2.

### <a name="theme-selection"></a>Selección de temas

Para seleccionar un nuevo tema, seleccione el icono del perfil ubicado en la esquina superior derecha. A continuación, seleccione **Cambiar tema**.

  [![Selección de tema](media/v2-update-explorer/tsi-preview-theme-selection.png)](media/v2-update-explorer/tsi-preview-theme-selection.png#lightbox)

> [!TIP]
> La selección del idioma también está disponible si selecciona el icono del perfil.

El Explorador de Azure Time Series Insights admite dos temas:

* **Tema claro**: es el tema predeterminado que se muestra en este documento.
* **Tema oscuro**: representa el Explorador como se muestra aquí:

  [![Tema oscuro seleccionado](media/v2-update-explorer/tsi-preview-dark-theme-selected.png)](media/v2-update-explorer/tsi-preview-dark-theme-selected.png#lightbox)

## <a name="gen-1-environment-controls"></a>Controles del entorno de Gen1

### <a name="gen2-terms-panel"></a>Panel de términos de Gen2

Esta sección se aplica solo a entornos de Gen1 existentes que intentan usar el Explorador en la interfaz de usuario actualizada. Es posible que quiera usar una combinación de los productos Gen1 y Gen2. Aunque se han agregado algunas funcionalidades con respecto a la interfaz de usuario existente al Explorador actualizado, puede obtener la experiencia completa de la interfaz de usuario del entorno de Gen1 en el nuevo Explorador de Azure Time Series Insights.

En lugar de la jerarquía, se muestra el panel de términos de Azure Time Series Insights Gen2. El panel de términos permite definir consultas en el entorno. Úselo también para filtrar los datos según un predicado.

  [![Panel de consultas con la cláusula Where](media/v2-update-explorer/s1-s2-preview-query.png)](media/v2-update-explorer/s1-s2-preview-query.png#lightbox)

El panel del editor de términos de Azure Time Series Insights Gen2 acepta los parámetros siguientes:

**Where**: use la cláusula Where para filtrar rápidamente sus eventos mediante el conjunto de operandos que se enumeran en la siguiente tabla. Si realiza una búsqueda seleccionando un operando, el predicado se actualizará automáticamente según esa búsqueda. Los tipos de operando compatibles incluyen los siguientes:

| Operación    | Tipos admitidos    | Notas |
| --- | --- | --- |
| `<`, `>`, `<=`, `>=` | Double, DateTime, TimeSpan | |
| `=`, `!=`, `<>` | String, Bool, Double, DateTime, TimeSpan, NULL |
| `IN` | String, Bool, Double, DateTime, TimeSpan, NULL | Todos los operandos deben ser del mismo tipo o deben ser una constante NULL. |
| `HAS` | String | En el lado derecho solo se permiten literales de cadenas constantes. No se permiten cadenas vacías ni NULL. |

Para obtener más información sobre las operaciones de consulta y los tipos de datos admitidos, lea [Expresión de serie temporal (TSX)](https://docs.microsoft.com/rest/api/time-series-insights/reference-time-series-expression-syntax).

### <a name="examples-of-where-clauses"></a>Ejemplos de cláusulas Where

  [![Ejemplos de cláusulas Where](media/v2-update-explorer/tsi-preview-example-queries.png)](media/v2-update-explorer/tsi-preview-example-queries.png#lightbox)

**Medida**: lista desplegable que muestra todas las columnas numéricas (**Dobles**) que puede usar como elementos para su gráfico actual.

**Split by**: esta lista desplegable muestra todas las columnas categóricas disponibles (Cadenas) en el modelo por las que puede agrupar sus datos. Puede agregar hasta cinco términos para verlos en el mismo eje X. Escriba los parámetros que quiera usar y, a continuación, seleccione **Agregar** para agregar un término nuevo.

  [![Vista filtrada y consultada uno](media/v2-update-explorer/s1-s2-preview-filtered-view.png)](media/v2-update-explorer/s1-s2-preview-filtered-view.png#lightbox)

Puede mostrar y ocultar elementos en el panel del gráfico si selecciona el icono visible, tal como se muestra en la siguiente imagen. Para quitar las consultas completamente, seleccione la **X** de color rojo.

  [![Cancelar una opción consultada y filtrada](media/v2-update-explorer/s1-s2-preview-filtered-view-cancel.png)](media/v2-update-explorer/s1-s2-preview-filtered-view-cancel.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [la ingesta de datos](./concepts-ingestion-overview.md) en su entorno.

* Consulte el artículo sobre el [almacenamiento](concepts-storage.md).

* Lea sobre el [modelado de datos](./concepts-model-overview.md) en Azure Time Series Insights Gen2.

* Aprenda a [diagnosticar y solucionar problemas](./time-series-insights-update-how-to-troubleshoot.md) del entorno.
