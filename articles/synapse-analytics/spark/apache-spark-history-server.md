---
title: Use el servidor de historial de Spark extendido para depurar aplicaciones
description: Uso del servidor de historial de Spark extendido para depurar y diagnosticar aplicaciones Spark en Azure Synapse Analytics.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 04/15/2020
ms.author: euang
ms.reviewer: euang
ms.openlocfilehash: e8dece3478e00c6f9279767e57e3bb8aca865f45
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87059982"
---
# <a name="use-extended-apache-spark-history-server-to-debug-and-diagnose-apache-spark-applications"></a>Uso del servidor de historial de Apache Spark extendido para depurar y diagnosticar aplicaciones Spark

En este artículo se proporcionan instrucciones sobre cómo usar el servidor de historial de Apache Spark para depurar y diagnosticar las aplicaciones Spark en ejecución y completadas.

La extensión incluye las pestañas Datos, Gráfico y Diagnóstico. Use la pestaña **Datos** para comprobar los datos de entrada y salida del trabajo de Spark. La pestaña **Gráfico** muestra el flujo de datos y reproduce el gráfico del trabajo. La pestaña **Diagnóstico** muestra la **Asimetría de datos**, el **Desfase horario** y el **Análisis de uso del ejecutor**.

## <a name="access-the-apache-spark-history-server"></a>Acceso al servidor de historial de Apache Spark

El servidor de historial de Apache Spark es la interfaz de usuario web para las aplicaciones Spark completadas y en ejecución. Puede abrir la interfaz web del servidor de historial de Apache Spark desde Azure Synapse Analytics.

### <a name="open-the-spark-history-server-web-ui-from-apache-spark-applications-node"></a>Apertura de la interfaz de usuario web del servidor de historial de Spark desde el nodo de aplicaciones Apache Spark

1. Abra [Azure Synapse Analytics](https://web.azuresynapse.net/).

2. Haga clic en **Supervisar** y, a continuación, seleccione la opción **Apache Spark Applications** (Aplicaciones Apache Spark).

    ![Haga clic en Supervisar y, a continuación, seleccione Aplicación Spark.](./media/apache-spark-history-server/click-monitor-spark-application.png)

3. Seleccione una aplicación y, a continuación, abra **Consulta de registro** haciendo clic en la opción.

    ![Abra una ventana de consulta de registro.](./media/apache-spark-history-server/open-application-window.png)

4. Seleccione **Servidor de historial de Spark** y, a continuación, se mostrará la interfaz de usuario web del servidor de historial de Spark.

    ![Abrir el servidor de historial de Spark.](./media/apache-spark-history-server/open-spark-history-server.png)

### <a name="open-the-spark-history-server-web-ui-from-data-node"></a>Apertura de la interfaz de usuario web del servidor de historial de Spark desde el nodo Datos

1. En el cuaderno de Azure Synapse Studio, seleccione **Servidor de historial de Spark** desde la celda de resultados de la ejecución del trabajo o desde el panel de estado en la parte inferior del documento del cuaderno. Seleccione **Detalles de la sesión**.

   ![Iniciar servidor de historial de Spark](./media/apache-spark-history-server/launch-history-server2.png "Iniciar servidor de historial de Spark")

2. Seleccione **Servidor de historial de Spark** en el panel deslizante.

   ![Iniciar servidor de historial de Spark](./media/apache-spark-history-server/launch-history-server.png "Iniciar servidor de historial de Spark")

## <a name="explore-the-data-tab-in-spark-history-server"></a>Examen de la pestaña Datos del servidor de historial de Spark

Seleccione el id. del trabajo que quiere ver. Luego, seleccione **Datos** en el menú de herramientas para obtener la vista de datos. En esta sección se muestra cómo realizar varias tareas desde la pestaña Datos.

* Seleccione cada pestaña para comprobar las **entradas**, las **salidas** y las **operaciones de tabla**.

    ![Datos de aplicaciones Spark: pestañas](./media/apache-spark-history-server/apache-spark-data-tabs.png)

* Seleccione la opción **Copiar** para copiar todas las filas.

    ![Datos de la aplicación Spark: copiar](./media/apache-spark-history-server/apache-spark-data-copy.png)

* Seleccione la opción **csv** para guardar todos los datos como un archivo CSV.

    ![Datos de la aplicación Spark: guardar](./media/apache-spark-history-server/apache-spark-data-save.png)

* Escriba palabras clave en el campo **Búsqueda** para realizar búsquedas. Los resultados de la búsqueda se muestran inmediatamente.

    ![Datos de la aplicación Spark: buscar](./media/apache-spark-history-server/apache-spark-data-search.png)

* Seleccione el encabezado de columna para ordenar la tabla, seleccione el signo más para expandir una fila y mostrar más detalles o seleccione el signo menos para contraer una fila.

    ![Datos de la aplicación Spark: tabla](./media/apache-spark-history-server/apache-spark-data-table.png)

* Descargue un solo archivo; para ello, seleccione **Descarga parcial**. El archivo seleccionado se descarga en la ubicación local. Si el archivo ya no existe, aparece una nueva pestaña con un mensaje de error.

    ![Datos de la aplicación Spark: descargar fila](./media/apache-spark-history-server/sparkui-data-download-row.png)

* Para copiar una ruta de acceso completa o una ruta de acceso relativa, seleccione la opción **Copiar ruta de acceso completa** o **Copiar ruta de acceso relativa** que se expande desde el menú desplegable. Para los archivos de Azure Data Lake Storage, **Abrir en Explorador de Azure Storage** iniciará Explorador de Azure Storage y buscará la carpeta en la que inició sesión.

    ![Datos de la aplicación Spark: copiar ruta de acceso](./media/apache-spark-history-server/sparkui-data-copy-path.png)

* Seleccione el número de página situado debajo de la tabla para desplazarse cuando en una página se muestran demasiadas filas.

    ![Datos de la aplicación Spark: página](./media/apache-spark-history-server/apache-spark-data-page.png)

* Mantenga el mouse sobre el signo de interrogación junto a **Datos** para mostrar la información sobre herramientas, o seleccione el signo de interrogación para obtener más información.

    ![Datos de la aplicación Spark: más información](./media/apache-spark-history-server/sparkui-data-more-info.png)

* Seleccione **Proporcione sus comentarios** para enviar comentarios sobre incidencias.

    ![El gráfico de Spark proporciona de nuevo comentarios](./media/apache-spark-history-server/sparkui-graph-feedback.png)

## <a name="graph-tab-in-apache-spark-history-server"></a>Pestaña Gráfico en el servidor de historial de Apache Spark

Seleccione el id. del trabajo que quiere ver. Después, seleccione **Gráfico** en el menú de herramientas para obtener la vista de gráfico del trabajo.

### <a name="overview"></a>Información general

Puede ver la información general del trabajo en el gráfico generado. De forma predeterminada, el gráfico muestra todos los trabajos. Puede filtrar esta vista por **Id. de trabajo**.

![Aplicación Spark y gráfico de trabajos: identificador del trabajo](./media/apache-spark-history-server/apache-spark-graph-jobid.png)

### <a name="display"></a>Pantalla

De forma predeterminada, está seleccionada la opción para mostrar **Progreso**. Puede comprobar el flujo de datos; para ello, seleccione **Read** (Leído) o **Written** (Escrito) en la lista desplegable **Mostrar**.

![Aplicación Spark y gráfico de trabajos: mostrar](./media/apache-spark-history-server/sparkui-graph-display.png)

El nodo de gráfico muestra los colores descritos en la leyenda del mapa térmico.

![Aplicación Spark y gráfico de trabajos: mapa térmico](./media/apache-spark-history-server/sparkui-graph-heatmap.png)

### <a name="playback"></a>Reproducción

Para reproducir el trabajo, seleccione **Reproducir**. Puede seleccionar **Detener** en cualquier momento para detener el proceso. Los colores de las tareas muestran distintos estados durante la reproducción:

|Color|Significado|
|-|-|
|Verde|Correcto: el trabajo se ha completado correctamente.|
|Naranja|Intentado de nuevo: las instancias de tareas que no se han realizado correctamente, pero que no afectan al resultado final del trabajo. Estas tareas tenían instancias duplicadas o de reintentos que pueden realizarse correctamente más tarde.|
|Azul|En ejecución: la tarea se está ejecutando.|
|Blanco|En espera u omitido: la tarea está esperando para ejecutarse o la fase se ha omitido.|
|Rojo|Con errores: error en la tarea.|

En la imagen siguiente se muestran los colores de estado verde, naranja y azul.

![Aplicación Spark y gráfico de trabajos: ejemplo de color, en ejecución](./media/apache-spark-history-server/sparkui-graph-color-running.png)

En la imagen siguiente se muestran los colores de estado blanco y verde.

![Aplicación Spark y gráfico de trabajos: ejemplo de color, omitir](./media/apache-spark-history-server/sparkui-graph-color-skip.png)

En la imagen siguiente se muestran los colores de estado verde y rojo.

![Aplicación Spark y gráfico de trabajos: ejemplo de color, error](./media/apache-spark-history-server/sparkui-graph-color-failed.png)

> [!NOTE]  
> Se permite la reproducción de cada trabajo. La reproducción no es compatible con los trabajos incompletos.

### <a name="zoom"></a>Zoom

Use la rueda del mouse para acercar y alejar el gráfico del trabajo o seleccione **Ajustar al tamaño** para ajustarlo a la pantalla.

![Aplicación Spark y gráfico de trabajos: zoom para ajustar](./media/apache-spark-history-server/sparkui-graph-zoom2fit.png)

### <a name="tooltips"></a>Información sobre herramientas

Mantenga el mouse sobre el nodo del gráfico para ver la información sobre herramientas cuando haya tareas con errores y seleccione una fase para abrir su página.

![Aplicación Spark y gráfico de trabajos: información sobre herramientas](./media/apache-spark-history-server/sparkui-graph-tooltip.png)

En la pestaña del gráfico de trabajo, las fases tienen información sobre herramientas y se muestra un icono pequeño si tienen tareas que cumplen las siguientes condiciones:

|Condición|Descripción|
|-|-|
|Asimetría de datos|Tamaño de lectura de datos > tamaño promedio de lectura de datos de todas las tareas de esta fase * 2 y tamaño de lectura de datos > 10 MB|
|Desfase horario|Tiempo de ejecución > tiempo promedio de ejecución de todas las tareas de esta fase * 2 y tiempo de ejecución > 2 minutos|
   
![Aplicación Spark y gráfico de trabajos: icono de asimetría](./media/apache-spark-history-server/sparkui-graph-skew-icon.png)

### <a name="graph-node-description"></a>Descripción del nodo de gráfico

El nodo de gráfico de trabajo muestra la siguiente información de cada fase:

  * Id.
  * Nombre o descripción.
  * Número de tarea total.
  * Datos leídos: la suma del tamaño de entrada y el tamaño de lectura aleatorio.
  * Datos escritos: suma del tamaño de salida y el tamaño de escrituras aleatorio.
  * Tiempo de ejecución: el tiempo entre la hora de inicio del primer intento y la hora de finalización del último intento.
  * Recuento de filas: la suma de los registros de entrada, los registros de salida, los registros de lectura aleatoria y los registros de escritura aleatoria.
  * Progreso.

    > [!NOTE]  
    > De forma predeterminada, el nodo del gráfico del trabajo muestra la información del último intento de cada fase (excepto en el tiempo de ejecución de la fase). Sin embargo, durante la reproducción, el nodo del gráfico muestra la información de cada intento.
    >  
    > El tamaño de los datos de lectura y escritura es 1 MB = 1000 KB = 1000 * 1000 bytes.

### <a name="provide-feedback"></a>Envío de comentarios

Seleccione **Proporcione sus comentarios** para enviar comentarios sobre incidencias.

![Aplicación Spark y gráfico de trabajos: comentarios](./media/apache-spark-history-server/sparkui-graph-feedback.png)

## <a name="explore-the-diagnosis-tab-in-apache-spark-history-server"></a>Examen de la pestaña Diagnóstico en el servidor de historial de Apache Spark

Para acceder a la pestaña Diagnóstico, seleccione un Id. de trabajo. Después, seleccione **Diagnóstico** en el menú de herramientas para obtener la vista Diagnóstico del trabajo. La pestaña de diagnóstico incluye **Asimetría de datos**, **Desfase horario** y **Análisis de uso del ejecutor**.

Para comprobar la **Asimetría de datos**, el **Desfase horario** y el **Análisis de uso del ejecutor**, seleccione las pestañas respectivamente.

![Diagnóstico de la interfaz de usuario de Spark: de nuevo la pestaña de asimetría de datos](./media/apache-spark-history-server/sparkui-diagnosis-tabs.png)

### <a name="data-skew"></a>Asimetría de datos

Cuando seleccione la pestaña **Asimetría de datos**, se mostrarán las tareas con sesgos según los parámetros especificados.

* **Especificar parámetros**: la primera sección muestra los parámetros que se usan para detectar la Asimetría de datos. La regla predeterminada es: La lectura de datos de tarea es tres veces mayor que la lectura de datos de tarea promedio y la lectura de datos de tarea es mayor que 10 MB. Si quiere definir su propia regla para las tareas con sesgos, puede elegir los parámetros y las secciones **Fase sesgada** y la sección **Gráfico de sesgo** se actualizarán en consecuencia.

* **Fase sesgada**: la segunda sección muestra las fases que tienen tareas con desfase que cumplen los criterios especificados anteriormente. Si hay más de una tarea sesgada en una fase, la tabla de fase sesgada solo muestra la tarea más sesgada (por ejemplo, los datos más grandes para la asimetría de datos).

    ![Diagnóstico de la interfaz de usuario de Spark: pestaña de asimetría de datos](./media/apache-spark-history-server/sparkui-diagnosis-dataskew-section2.png)

* **Gráfico de sesgo**: cuando se selecciona una fila de la tabla de fase sesgada, el gráfico de sesgo muestra más detalles de distribución de tareas en función de la lectura de datos y el tiempo de ejecución. Las tareas sesgadas se marcan en rojo y las tareas normales se marcan en azul. El gráfico muestra hasta 100 tareas de ejemplo y los detalles de la tarea se muestran en el panel inferior derecho.

    ![Interfaz de usuario de Spark: gráfico de desfase de la fase 10](./media/apache-spark-history-server/sparkui-diagnosis-dataskew-section3.png)

### <a name="time-skew"></a>Desfase horario

En la pestaña **Desfase horario** se muestran las tareas sesgadas en función del tiempo de ejecución de la tarea.

* **Especificar parámetros**: la primera sección muestra los parámetros que se usan para detectar el desfase horario. Los criterios predeterminados para detectar el desfase horario son: el tiempo de ejecución de la tarea es mayor que tres veces el tiempo de ejecución promedio y el tiempo de ejecución de la tarea es superior a 30 segundos. Puede cambiar los parámetros en función de sus necesidades. La **fase sesgada** y el **gráfico de sesgo** muestran información sobre las fases y las tareas correspondientes del mismo modo que la pestaña **Asimetría de datos**.

* Seleccione **Desfase horario** y, después, se muestran los resultados filtrados en la sección **Fase sesgada** según los parámetros establecidos en la sección **Especificar parámetros**. Seleccione un elemento de la sección **Fase sesgada** para mostrar el gráfico correspondiente en la sección 3 y los detalles de la tarea en el panel inferior derecho.

    ![Diagnóstico de la interfaz de usuario de Spark: sección de desfase horario](./media/apache-spark-history-server/sparkui-diagnosis-timeskew-section2.png)

### <a name="executor-usage-analysis"></a>Análisis de uso del ejecutor

El gráfico de uso del ejecutor visualiza el estado de ejecución y la asignación del ejecutor del trabajo de Spark.  

1. Seleccione **Análisis de uso del ejecutor** y se trazarán cuatro tipos de curvas sobre el uso del ejecutor, incluidas **Ejecutores asignados**, **Ejecutores en ejecución**, **Ejecutores sin uso** e **Instancias de ejecutor máximas**. En lo que respecta a los ejecutores asignados, cada evento "Ejecutor agregado" o "Ejecutor eliminado" aumenta o disminuye los ejecutores asignados. Puede comprobar la "escala de tiempo del evento" en la pestaña "Trabajos" para obtener más comparaciones.

   ![Diagnóstico de la interfaz de usuario de Spark: pestaña de ejecutores](./media/apache-spark-history-server/sparkui-diagnosis-executors.png)

2. Seleccione el icono de color para seleccionar o anular la selección del contenido correspondiente en todos los borradores.

    ![Diagnósticos de la interfaz de usuario de Spark: seleccionar gráfico](./media/apache-spark-history-server/sparkui-diagnosis-select-chart.png)

## <a name="known-issues"></a>Problemas conocidos

Los datos de entrada/salida mediante conjuntos de datos distribuidos resistentes (RDD) no se muestran en la pestaña Datos.

## <a name="next-steps"></a>Pasos siguientes

- [Azure Synapse Analytics](../overview-what-is.md)
- [Documentación de .NET para Apache Spark](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)

