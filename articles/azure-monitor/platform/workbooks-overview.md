---
title: Introducción a los libros de Azure Monitor
description: Simplifique la creación de informes complejos con libros parametrizados creados previamente y personalizados.
author: mrbullwinkle
manager: carmonm
services: azure-monitor
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 07/23/2020
ms.author: mbullwin
ms.openlocfilehash: 1d85f011289bfa27bcf1e94d8da8cd7a32092c2c
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87799572"
---
# <a name="azure-monitor-workbooks"></a>Libros de Azure Monitor

Los libros proporcionan un lienzo flexible para el análisis de datos y la creación de informes visuales completos en el Azure Portal. Permiten acceder a varios orígenes de datos desde Azure y combinarlos en experiencias interactivas unificadas. 

Este es un tutorial en vídeo sobre la creación de libros.

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE4B4Ap]

## <a name="data-sources"></a>Orígenes de datos

Los libros pueden consultar datos de varios orígenes dentro de Azure. Los autores de los libros pueden transformar estos datos para proporcionar información sobre la disponibilidad, el rendimiento, el uso y el estado general de los componentes subyacentes. Por ejemplo, el análisis de los registros de rendimiento de las máquinas virtuales para identificar grandes instancias de CPU o de memoria insuficiente y mostrar los resultados como una cuadrícula en un informe interactivo.
  
Pero la eficacia real de los libros es la capacidad de combinar datos de orígenes dispares dentro de un único informe. Esto permite la creación de vistas de recursos compuestas o combinaciones en los recursos, lo que permite obtener datos e información más completos que, de lo contrario, serían imposibles.

Los libros son compatibles actualmente con los siguientes orígenes de datos:

* [Registros](workbooks-data-sources.md#logs)
* [Métricas](workbooks-data-sources.md#metrics)
* [Azure Resource Graph](workbooks-data-sources.md#azure-resource-graph)
* [Alertas (versión preliminar)](workbooks-data-sources.md#alerts-preview)
* [Mantenimiento de la carga de trabajo](workbooks-data-sources.md#workload-health)
* [Azure Resource Health](workbooks-data-sources.md#azure-resource-health)
* [Azure Data Explorer](workbooks-data-sources.md#azure-data-explorer)

## <a name="visualizations"></a>Visualizaciones

Los libros proporcionan un amplio conjunto de funciones para visualizar los datos. Para obtener ejemplos detallados de cada tipo de visualización, puede consultar los siguientes vínculos de ejemplo:

* [Texto](workbooks-visualizations.md#text)
* [Gráficos](workbooks-visualizations.md#charts)
* [Cuadrículas](workbooks-visualizations.md#grids)
* [Iconos](workbooks-visualizations.md#tiles)
* [Árboles](workbooks-visualizations.md#trees)
* [Gráficos](workbooks-visualizations.md#graphs)

![Ejemplo de visualizaciones de libros](./media/workbooks-overview/visualizations.png)

## <a name="getting-started"></a>Introducción

Para explorar la experiencia de los libros, navegue primero hasta el servicio Azure Monitor. Para ello, escriba **monitor** en el cuadro de búsqueda de Azure Portal.

A continuación, seleccione **Libros**.

![Captura de pantalla con el botón de Workbooks resaltado en un cuadro rojo](./media/workbooks-overview/workbooks.png)

### <a name="gallery"></a>Galería

Esto le llevará a la galería de libros:

![Captura de pantalla de la galería de libros de Azure Monitor](./media/workbooks-overview/gallery.png)

### <a name="workbooks-versus-workbook-templates"></a>Libros frente a plantillas de libro

Puede ver un _libro_ en verde y varias _plantillas de libro_ en color púrpura. Las plantillas sirven como informes seleccionados que están diseñados para que varios usuarios y equipos puedan reutilizarlos de forma flexible. Al abrir una plantilla se crea un libro transitorio que se rellena con el contenido de la plantilla. 

Puede ajustar los parámetros del libro basado en la plantilla y realizar el análisis sin miedo a interrumpir la futura experiencia de informes para colegas. Si abre una plantilla, realice algunos ajustes y luego seleccione el icono guardar para guardar la plantilla como un libro que se mostrará en verde y la plantilla original quedará intacta. 

En el fondo, las plantillas también difieren de los libros guardados. Al guardar un libro, se crea un recurso de Azure Resource Manager asociado, mientras que el libro transitorio que se crea al abrir una plantilla no tiene asociado un recurso único. Para obtener más información sobre cómo se administra el control de acceso en los libros, consulte el [artículo control de acceso de los libros](workbooks-access-control.md).

### <a name="exploring-a-workbook-template"></a>Exploración de una plantilla de libro

Seleccione **Análisis de los errores de la aplicación** para ver una de las plantillas de libro de la aplicación predeterminadas.

![Captura de pantalla de la plantilla de análisis de los errores de la aplicación](./media/workbooks-overview/failure-analysis.png)

Como se indicó anteriormente, al abrir la plantilla se crea un libro temporal para poder interactuar con él. De forma predeterminada, el libro se abre en modo de lectura, lo que muestra solo la información de la experiencia de análisis previsto creada por el autor original de la plantilla.

En el caso de este libro concreto, la experiencia es interactiva. Puede ajustar la suscripción, las aplicaciones de destino y el intervalo de tiempo de los datos que desea mostrar. Una vez realizadas esas selecciones, la cuadrícula de solicitudes HTTP también es interactiva, por lo que seleccionar una fila individual cambiará los datos que se representan en los dos gráficos en la parte inferior del informe.

### <a name="editing-mode"></a>Modo de edición

Para comprender cómo se agrupa esta plantilla de libro, debe cambiar al modo de edición seleccionando **Editar**. 

![Captura de pantalla de la plantilla de análisis de los errores de la aplicación](./media/workbooks-overview/edit.png)

Una vez que haya cambiado al modo de edición, observará que aparece un número de cuadros de **Editar** a la derecha correspondiente con cada aspecto individual del libro.

![Captura de pantalla del botón de edición](./media/workbooks-overview/edit-mode.png)

Si seleccionamos el botón Editar inmediatamente debajo de la cuadrícula de los datos de la solicitud, podemos ver que esta parte del libro consta de una consulta Kusto en los datos de un recurso de Application Insights.

![Captura de pantalla de la consulta Kusto subyacente](./media/workbooks-overview/kusto.png)

Al hacer clic en los botones otros **Editar** de la derecha, se mostrarán varios componentes principales que conforman los libros, como los [cuadros de texto](workbooks-visualizations.md#text) basados en marcado, los elementos de la interfaz de usuario de la [selección de parámetros](workbooks-parameters.md) y otros [tipos de gráficos y visualizaciones](workbooks-visualizations.md). 

Explorar las plantillas predefinidas en modo de edición y después modificarlas para adaptarlas a sus necesidades y guardar su propio libro personalizado es una excelente manera de empezar a obtener información sobre lo que es posible con los libros de Azure Monitor.

## <a name="pinning-visualizations"></a>Anclado de visualizaciones

Los pasos de texto, consulta y métricas de un libro se pueden anclar mediante el botón de anclaje en esos elementos, mientras el libro está en modo de anclaje, o bien, si el autor del libro tiene habilitada la configuración para ese elemento con el fin de que el icono para anclar esté visible. 

Para acceder al modo de anclaje, haga clic en **editar** para entrar en el modo de edición, luego seleccione el icono para anclar azul en la barra superior. Luego aparecerá un icono de anclaje individual por encima de cada cuadro de *Editar* de la parte correspondiente del libro en el lado derecho de la pantalla.

![Experiencia de anclaje](./media/workbooks-overview/pin-experience.png)

> [!NOTE]
> El estado del libro se guarda en el momento del anclaje, y los libros anclados en un panel no se actualizarán si se modifica el libro subyacente. Para actualizar una parte del libro anclada, deberá eliminarla y volver a anclarla.

## <a name="dashboard-time-ranges"></a>Intervalos de tiempo del panel

Los elementos de consulta del libro anclado respetarán el intervalo de tiempo del panel si el elemento anclado está configurado para usar un parámetro de *Intervalo de tiempo*. El valor del intervalo de tiempo del panel se usará como el valor del parámetro de intervalo de tiempo y cualquier cambio del intervalo de tiempo del panel hará que el elemento anclado se actualice. Si una parte anclada usa el intervalo de tiempo del panel, verá el subtítulo de la actualización de la parte anclada para mostrar el intervalo de tiempo del panel cada vez que cambie el intervalo de tiempo. 

Además, los elementos del libro anclados con un parámetro de intervalo de tiempo se actualizarán automáticamente a una velocidad determinada por el intervalo de tiempo del panel. La última vez que se ejecutó la consulta aparecerá en el subtítulo de la parte anclada.

Si un paso anclado tiene un intervalo de tiempo establecido explícitamente (no usa un parámetro de intervalo de tiempo), ese intervalo de tiempo se usará siempre para el panel, independientemente de la configuración del panel. El subtítulo de la parte anclada no mostrará el intervalo de tiempo del panel y la consulta no se actualizará automáticamente en el panel. El subtítulo mostrará la última vez que se ejecutó la consulta.

> [!NOTE]
> Actualmente no se admiten consultas con el origen de datos de *combinación* al anclar a los paneles.

## <a name="sharing-workbook-templates"></a>Compartir plantillas de libro

Una vez que empiece a crear sus propias plantillas de libro, es posible que quiera compartirla con la comunidad. Para obtener más información y explorar otras plantillas que no forman parte de la vista predeterminada de la galería de Azure Monitor, visite nuestro [repositorio GitHub](https://github.com/Microsoft/Application-Insights-Workbooks/blob/master/README.md). Para examinar los libros existentes, visite la [biblioteca de libros](https://github.com/microsoft/Application-Insights-Workbooks/tree/master/Workbooks) en GitHub.

## <a name="next-step"></a>Paso siguiente

* [Comience](workbooks-visualizations.md) a aprender más sobre las muchas opciones de visualizaciones enriquecidas de los libros.
* [Controle](workbooks-access-control.md) y comparta el acceso a los recursos del libro.
