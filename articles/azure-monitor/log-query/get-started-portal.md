---
title: 'Tutorial: Introducción a las consultas de Log Analytics'
description: En este tutorial aprenderá a escribir y administrar consultas de registro de Azure Monitor mediante Log Analytics en Azure Portal.
ms.subservice: logs
ms.topic: tutorial
author: bwren
ms.author: bwren
ms.date: 03/17/2020
ms.openlocfilehash: 8917c73474b81ee76e6acdf8c6f274e962ee1215
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87874238"
---
# <a name="tutorial-get-started-with-log-analytics-queries"></a>Tutorial: Introducción a las consultas de Log Analytics

En este tutorial se indica cómo usar Log Analytics para escribir, ejecutar y administrar consultas de registro de Azure Monitor en Azure Portal. Las consultas de Log Analytics se pueden usar para buscar términos, identificar tendencias, analizar patrones y proporcionar muchas otras conclusiones basadas en los datos. 

En este tutorial, aprenderá a usar Log Analytics para:

> [!div class="checklist"]
> * Comprender el esquema de datos del registro
> * Escribir y ejecutar consultas sencillas, y modificar el intervalo de tiempo de las consultas
> * Filtrar, ordenar y agrupar los resultados de las consultas
> * Ver, modificar y compartir los objetos visuales de los resultados de las consultas
> * Guardar, cargar, exportar y copiar consultas y resultados

Para obtener más información sobre las consultas de registro, vea [Análisis de datos de registro en Azure Monitor](log-query-overview.md).<br/>
Para ver un tutorial detallado sobre cómo escribir consultas de registro, consulte [Introducción a las consultas de registro en Azure Monitor](get-started-queries.md).

## <a name="open-log-analytics"></a>Apertura de Log Analytics
Para usar Log Analytics debe iniciar sesión en una cuenta de Azure. Si no tiene una cuenta de Azure, [cree una gratis](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

Para completar la mayoría de los pasos de este tutorial, puede usar [este entorno de demostración](https://ms.portal.azure.com/#blade/Microsoft_Azure_Monitoring_Logs/DemoLogsBlade) que incluye gran cantidad de datos de ejemplo. En este entorno, no podrá guardar consultas ni anclar resultados en un panel.

También puede usar su propio entorno si va a utilizar Azure Monitor para recopilar datos de registro en al menos un recurso de Azure. Para abrir un área de trabajo de Log Analytics, seleccione **Registros** en el panel de navegación izquierdo de Azure Monitor. 

## <a name="understand-the-schema"></a>Información sobre el esquema
 
Un *esquema* es una colección de tablas que se agrupan en categorías lógicas. El esquema de demostración incluye varias categorías de soluciones de supervisión. Por ejemplo, la categoría **LogManagement** contiene eventos, datos de rendimiento y latidos del agente de Windows y Syslog.

Las tablas del esquema aparecen en la pestaña **Tablas** del área de trabajo de Log Analytics. Las tablas contienen columnas, cada una con un tipo de datos que muestra el icono situado junto al nombre de la columna. Por ejemplo, la tabla **Eventos** contiene columnas de texto como **Equipo** y columnas numéricas como **EventCategory**.

![Schema](media/get-started-portal/schema.png)

## <a name="write-and-run-basic-queries"></a>Escritura y ejecución de consultas básicas

Log Analytics se abre con una nueva consulta en blanco en el **Editor de consultas**.

![Log Analytics](media/get-started-portal/homepage.png)

### <a name="write-a-query"></a>Escriba una consulta.

Las consultas de registro de Azure Monitor usan una versión del lenguaje de consulta de Kusto. Las consultas pueden comenzar por un nombre de tabla o un comando [search](/azure/kusto/query/searchoperator). 

La siguiente consulta recupera todos los registros de la tabla **Eventos**:

```Kusto
Event
```

El carácter de barra vertical (|) separa los comandos, de manera que la salida del primer comando es la entrada del siguiente. A una consulta se puede agregar cualquier cantidad de comandos. La consulta siguiente recupera los registros de la tabla **Eventos** y, a continuación, busca en ellos el término **error** en todas las propiedades:

```Kusto
Event 
| search "error"
```

Un solo salto de línea hace que las consultas sean más fáciles de leer. Más de un salto de línea divide la consulta en consultas independientes.

Esta es otra forma de escribir la misma consulta:

```Kusto
search in (Event) "error"
```

En el segundo ejemplo, el comando **search** solo busca en los registros de la tabla **Eventos** el término **error**.

De forma predeterminada, Log Analytics limita las consultas a un intervalo de tiempo de las últimas 24 horas. Para establecer un intervalo de tiempo diferente, puede agregar un filtro **TimeGenerated** explícito a la consulta o utilizar el control **Intervalo de tiempo**.

### <a name="use-the-time-range-control"></a>Uso del control Intervalo de tiempo
Para utilizar el control **Intervalo de tiempo** selecciónelo en la barra superior y, después, seleccione un valor de la lista desplegable o seleccione **Personalizado** para crear un intervalo de tiempo personalizado.

![Selector de hora](media/get-started-portal/time-picker.png)

- Los valores del intervalo de tiempo están en formato UTC, el cual podría ser diferente de su zona horaria local.
- Si la consulta establece explícitamente un filtro para **TimeGenerated**, en el control selector de hora aparece **Establecer en la consulta** y se deshabilita para evitar un conflicto.

### <a name="run-a-query"></a>Ejecución de una consulta
Para ejecutar una consulta, sitúe el cursor en algún lugar de esta y seleccione **Ejecutar** en la barra superior o presione **Mayús**+**Entrar**. La consulta se ejecuta hasta que encuentra una línea en blanco.

## <a name="filter-results"></a>Filtrar resultados
Log Analytics limita los resultados a un máximo de 10 000 registros. Una consulta general como `Event` devuelve demasiados resultados para ser útil. Puede filtrar los resultados mediante la restricción de los elementos de la tabla de la consulta o agregando explícitamente un filtro a los resultados. El filtrado mediante elementos de tabla devuelve un nuevo conjunto de resultados mientras que un filtro explícito se aplica al conjunto de resultados ya existente.

### <a name="filter-by-restricting-table-elements"></a>Filtro mediante la restricción de elementos de tabla
Para filtrar los resultados de la consulta de `Event` por eventos de **Error** mediante la restricción de elementos de tabla de la consulta:

1. En los resultados de la consulta, seleccione la flecha desplegable situada junto a cualquier registro que contenga **Error** en la columna **EventLevelName**. 
   
1. En los detalles expandidos, pase el puntero y seleccione el icono **...** situado junto a **EventLevelName** y, después, seleccione **Incluir "Error"** . 
   
   ![Agregar filtro a consulta](media/get-started-portal/add-filter.png)
   
1. Observe que la consulta del **Editor de consultas** ha cambiado ahora a:
   
   ```Kusto
   Event
   | where EventLevelName == "Error"
   ```
   
1. Seleccione **Ejecutar** para ejecutar la nueva consulta.

### <a name="filter-by-explicitly-filtering-results"></a>Filtro mediante el filtrado explícito de los resultados
Para filtrar los resultados de la consulta de `Event` por eventos de **Error** mediante el filtrado de los resultados de la consulta:

1. En los resultados de la consulta, seleccione el icono de **Filtro** situado junto al encabezado de la columna **EventLevelName**. 
   
1. En el primer campo de la ventana emergente, seleccione **Es igual que** y, en el siguiente campo, escriba *error*. 
   
1. Seleccione **Filtro**.
   
   ![Filter](media/get-started-portal/filter.png)

## <a name="sort-group-and-select-columns"></a>Ordenación, agrupación y selección de columnas
Para ordenar los resultados de una consulta por una columna específica, como **TimeGenerated [UTC]** , seleccione el encabezado de la columna. Seleccione el encabezado de nuevo para cambiar entre un orden ascendente o descendente.

![Ordenación de la columna](media/get-started-portal/sort-column.png)

Otra manera de organizar los resultados es por grupos. Para agrupar resultados por una columna específica, arrastre el encabezado de la columna a la barra situada sobre la tabla de resultados con la etiqueta **Drag a column header and drop it here to group by that column** (Arrastrar un encabezado de columna y soltarlo aquí para agrupar por esa columna). Para crear subgrupos, arrastre otras columnas a la barra superior. Puede reorganizar la jerarquía y ordenar los grupos y subgrupos en la barra.

![Grupos](media/get-started-portal/groups.png)

Para ocultar o mostrar las columnas de los resultados, seleccione **Columnas** situado encima de la tabla y, a continuación, seleccione o anule la selección de las columnas que desee en la lista desplegable.

![Select columns](media/get-started-portal/select-columns.png)

## <a name="view-and-modify-charts"></a>Visualización y modificación de gráficos
También puede ver los resultados de la consulta en formatos visuales. Escriba la siguiente consulta como ejemplo:

```Kusto
Event 
| where EventLevelName == "Error" 
| where TimeGenerated > ago(1d) 
| summarize count() by Source 
```

De forma predeterminada, los resultados aparecen en una tabla. Seleccione **Gráfico** en la parte superior de la tabla para ver los resultados en una vista gráfica.

![Gráfico de barras](media/get-started-portal/bar-chart.png)

Los resultados aparecen en un gráfico de barras apiladas. Seleccione otras opciones como **Columna apilada** o **Circular** para mostrar otras vistas de los resultados.

![Gráfico circular](media/get-started-portal/pie-chart.png)

Las propiedades de la vista, como los ejes "x" e "y", o las preferencias de agrupación y división, se pueden cambiar manualmente en la barra de control.

También puede establecer la vista que prefiera en la propia consulta mediante el operador de [representación](/azure/kusto/query/renderoperator).

## <a name="pin-results-to-a-dashboard"></a>Anclaje de resultados a un panel

Para anclar una tabla o gráfico de resultados desde Log Analytics en un panel compartido de Azure, seleccione **Anclar al panel** en la barra superior. 

![Anclar al panel](media/get-started-portal/pin-dashboard.png)

En el panel **Anclar a otro panel**, seleccione un panel al que realizar el anclaje, o créelo, y seleccione **Aplicar**. La tabla o gráfico aparece en el panel de Azure seleccionado.

![Gráfico anclado al panel](media/get-started-portal/pin-dashboard2.png)

La tablas o gráficos que ancle a un panel compartido tendrán las siguientes limitaciones: 

- Los datos se limitan a los últimos 14 días.
- Una tabla solo muestra un máximo de cuatro columnas y las siete primeras filas.
- Los gráficos con muchas categorías discretas agrupan automáticamente las categorías menos pobladas en una única categoría **otros**.

## <a name="save-load-or-export-queries"></a>Guardar, cargar o exportar consultas

Después de crear una consulta, puede guardarla o compartir la consulta o los resultados con otros. 

### <a name="save-queries"></a>Guardado de consultas

Para guardar una consulta:

1. Seleccione **Guardar** en la barra superior.
   
1. En el cuadro de diálogo **Guardar**, asigne a la consulta un **Nombre** mediante los caracteres a – z, A – Z, 0-9, espacio, guión, carácter de subrayado, punto, paréntesis o barra vertical. 
   
1. Seleccione si desea guardar la consulta como **Consulta** o como **Función**. Las funciones son consultas a las que pueden hacer referencia otras consultas. 
   
   Para guardar una consulta como una función, proporcione un **Alias de función**, que es un nombre corto que otras consultas usan para llamar a esta consulta.
   
1. Si está en un área de trabajo de Log Analytics, proporcione una **categoría** para el **Explorador de consultas** que se usará para la consulta. (Las categorías no están disponibles para las consultas de Applications Insights)
   
1. Seleccione **Guardar**.
   
   ![Función Guardar](media/get-started-portal/save-function.png)

### <a name="load-queries"></a>Carga de consultas
Para cargar una consulta guardada, seleccione **Explorador de consultas** en la esquina superior derecha. Se abre el panel **Explorador de consultas**, donde se muestran todas las consultas por categoría. Expanda las categorías o escriba un nombre de consulta en la barra de búsqueda y, a continuación, seleccione una consulta para cargarla en el **Editor de consultas**. Para marcar una consulta como **Favorita**, seleccione la estrella situada junto al nombre de la consulta.

![Explorador de consultas](media/get-started-portal/query-explorer.png)

### <a name="export-and-share-queries"></a>Exportar y compartir consultas
Para exportar una consulta, seleccione **Exportar** en la barra superior y, después, seleccione **Exportar a CSV: todas las columnas**, **Exportar a CSV: columnas mostradas** o **Exportar a Power BI (M Query)** en la lista desplegable.

En el vídeo siguiente se muestra cómo integrar Log Analytics con Excel.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4Asme]

Para compartir un vínculo a una consulta, seleccione **Copiar vínculo** en la barra superior y, a continuación, seleccione **Copiar vínculo a la consulta**, **Copiar texto de la consulta** o **Copiar resultados de la consulta** para copiarlos en el portapapeles. Puede enviar el vínculo de la consulta a otros usuarios que tengan acceso a la misma área de trabajo.

## <a name="next-steps"></a>Pasos siguientes

Prosiga con el siguiente tutorial para más información sobre la escritura de consultas de registro de Azure Monitor.
> [!div class="nextstepaction"]
> [Escritura de consultas de registros de Azure Monitor](get-started-queries.md)
