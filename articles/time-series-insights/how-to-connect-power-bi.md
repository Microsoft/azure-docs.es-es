---
title: 'Conexión de un entorno a Power BI: Azure Time Series Insights | Microsoft Docs'
description: Aprenda a conectar Azure Time Series Insights a Power BI para compartir, representar y mostrar datos en toda una organización.
author: deepakpalled
ms.author: dpalled
manager: diviso
services: time-series-insights
ms.service: time-series-insights
ms.topic: conceptual
ms.date: 08/26/2020
ms.openlocfilehash: f15686cf07bae4aee41095c970cd8a474724c2ed
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230983"
---
# <a name="visualize-data-from-azure-time-series-insights-in-power-bi"></a>Visualización de datos de Azure Time Series Insights en Power BI

Azure Time Series Insights es una plataforma para almacenar, administrar, consultar y visualizar datos de series temporales en la nube. [Power BI](https://powerbi.microsoft.com) es una herramienta de análisis de negocio con funcionalidades de visualización enriquecidas que permiten compartir información y resultados en toda la organización. Ahora se pueden integrar ambos servicios para obtener lo mejor de las funcionalidades de visualización inherentes de Azure Time Series Insights y de Power BI.

Aprenderá a:

* Conectar Azure Time Series Insights a Power BI mediante el conector de nube
* Crear objetos visuales con los datos en Power BI
* Publicar el informe en Power BI y compartirlo con el resto de la organización

Al final, aprenderá a visualizar datos de series temporales mediante Azure Time Series Insights y a mejorarlos con las potentes funcionalidades de visualización de datos y de fácil uso compartido de Power BI.

Asegúrese de que se registra para obtener una [suscripción gratuita a Azure](https://azure.microsoft.com/free/), en caso de que no tenga ninguna.

## <a name="prerequisites"></a>Requisitos previos

* Descargar e instalar la versión más reciente de [Power BI Desktop](https://powerbi.microsoft.com/downloads/)
* Tener o crear un [entorno de Azure Time Series Insights Gen2](time-series-insights-update-how-to-manage.md)

> [!IMPORTANT]
>
> * El conector se admite actualmente en entornos de Azure Time Series Insights Gen2 configurados con **Warm Store only** (Solo almacén intermedio).
> * Si tiene acceso de invitado al entorno de Azure Time Series Insights Gen2 desde otro inquilino de Azure AD, no podrá tener acceso al conector. Lea sobre las [directivas de acceso al entorno](./concepts-access-policies.md).

## <a name="connect-data-from-azure-time-series-insights-to-power-bi"></a>Conexión de datos de Azure Time Series Insights a Power BI

Para conectar un entorno de Azure Time Series Insights a Power BI, siga estos pasos:

1. Abra el Explorador de Azure Time Series Insights.
1. Exporte los datos en forma de consulta o de datos sin procesar
1. Abra Power BI Desktop
1. Cargue desde una consulta personalizada

### <a name="export-data-into-power-bi-desktop"></a>Exporte los datos en Power BI Desktop

Primeros pasos:

1. Abra el Explorador de Azure Time Series Insights y realice el mantenimiento de los datos.
1. Una vez que haya creado una vista con la que esté conforme, vaya al menú desplegable **Más acciones** y seleccione **Conectar con Power BI**.

    [![Exportación del Explorador de Azure Time Series Insights](media/how-to-connect-power-bi/time-series-insights-export-option.png)](media/how-to-connect-power-bi/time-series-insights-export-option.png#lightbox)

1. Establezca los parámetros en esta pestaña:

   1. Especifique el periodo concreto que va a ver. Si no le gusta su vista existente, deje la opción **Periodo existente**.

   1. Elija entre **Agregado** y **Eventos sin formato**.

       > [!NOTE]
       > Siempre puede agregar los datos posteriormente en Power BI, pero no puede realizar la reversión a datos sin formato después de la agregación.

       > [!NOTE]
       > Hay un límite de recuento de eventos de 250 000 para los datos de nivel de evento sin formato.

       [![Conectar](media/how-to-connect-power-bi/connect-to-power-bi.png)](media/how-to-connect-power-bi/connect-to-power-bi.png#lightbox)

   1. Si no ha configurado el entorno de Azure Time Series Insights con **almacenamiento intermedio**, recibirá una advertencia.

       [![Advertencia de almacenamiento intermedio](media/how-to-connect-power-bi/connect-to-power-bi-warning.png)](media/how-to-connect-power-bi/connect-to-power-bi-warning.png#lightbox)

       > [!TIP]
       > Puede configurar la instancia existente para el **almacenamiento intermedio** en Azure Portal.

1. Seleccione **Copiar consulta en el Portapapeles**.
1. Ahora, inicie Power BI Desktop.
1. En Power BI Desktop, en la pestaña **Inicio**, seleccione **Obtener datos** en la esquina superior izquierda y luego **Más**.

    [![Menú desplegable de inicio](media/how-to-connect-power-bi/power-bi-home-drop-down.png)](media/how-to-connect-power-bi/power-bi-home-drop-down.png#lightbox)

1. Busque **Azure Time Series Insights**, seleccione **Azure Time Series Insights (Beta)** y, después, **Conectar**.

    [![Conexión de Power BI a Azure Time Series Insights](media/how-to-connect-power-bi/connect-to-time-series-insights.png)](media/how-to-connect-power-bi/connect-to-time-series-insights.png#lightbox)

    También puede ir a la pestaña **Azure**, seleccionar **Azure Time Series Insights (Beta)** y, después, **Conectar**.

1. Se mostrará un cuadro de diálogo en el que se le pedirá permiso para conectarse a recursos de terceros. Seleccione **Continuar**.

    [![Elegir Creación de una consulta personalizada](media/how-to-connect-power-bi/confirm-the-connection.png)](media/how-to-connect-power-bi/confirm-the-connection.png#lightbox)

1. En el menú desplegable que hay en **Origen de datos**, elija **Create custom query** (Crear consulta personalizada). Pegue desde el portapapeles en el campo **Consulta personalizada (opcional)** siguiente y, después, presione **Aceptar**.

    [![Pasar la consulta personalizada y seleccionar Aceptar](media/how-to-connect-power-bi/custom-query-load.png)](media/how-to-connect-power-bi/custom-query-load.png#lightbox)  

1. Se cargará la tabla de datos. Pulse **Cargar** para realizar la carga en Power BI.

    [![Examinar los datos cargados en la tabla y seleccionar Cargar](media/how-to-connect-power-bi/review-the-loaded-data-table.png)](media/how-to-connect-power-bi/review-the-loaded-data-table.png#lightbox)  

Si ha completado estos pasos, vaya a la sección siguiente.

## <a name="create-a-report-with-visuals"></a>Crear un informe con objetos visuales

Ahora que ha importado los datos en Power BI, es el momento de crear un informe con objetos visuales.

1. En el lado izquierdo de la ventana, asegúrese de que ha seleccionado la vista **Informe**.

    [![Seleccionar la vista Informe](media/how-to-connect-power-bi/select-the-report-view.png)](media/how-to-connect-power-bi/select-the-report-view.png#lightbox)

1. En la columna **Visualizaciones**, seleccione el objeto visual que prefiera. Por ejemplo, seleccione **Gráfico de líneas**. Se agregará un gráfico de líneas en blanco al lienzo.

1. En la lista **Campos**, seleccione **_Marca de tiempo** y arrástrelo hasta el campo **Eje** para mostrar los elementos en el eje X. Asegúrese de cambiar a **_Marca de tiempo** como el valor de **Eje** (el valor predeterminado es **Jerarquía de fechas**).

    [![Seleccionar la vista Informe](media/how-to-connect-power-bi/select-timestamp.png)](media/how-to-connect-power-bi/select-timestamp.png#lightbox)

1. En la lista **Campos**, seleccione **TimeSeriesId** y arrástrelo hasta el campo **Valores** para mostrar los elementos en el eje Y.

    [![Crear un gráfico de líneas](media/how-to-connect-power-bi/power-bi-line-chart.png)](media/how-to-connect-power-bi/power-bi-line-chart.png#lightbox)

1. Para agregar otro gráfico al lienzo, seleccione cualquier lugar del lienzo fuera del gráfico de líneas y repita este proceso.

    [![Crear gráficos adicionales para compartir](media/how-to-connect-power-bi/power-bi-additional-charts.png)](media/how-to-connect-power-bi/power-bi-additional-charts.png#lightbox)

Una vez creado el informe, puede publicarlo en Reporting Services de Power BI.

## <a name="advanced-editing"></a>Edición avanzada

Si ya ha cargado un conjunto de datos en Power BI pero desea modificar la consulta (por ejemplo, los parámetros de fecha y hora o del identificador del entorno), puede hacerlo a través de la funcionalidad Editor avanzado de Power BI. Para más información, consulte la [documentación de Power BI](https://docs.microsoft.com/power-bi/desktop-query-overview).

A modo de información general:

1. En Power BI Desktop, seleccione **Editar consultas**.
1. Presione **Editor avanzado**.

    [![Editar consultas en el Editor avanzado](media/how-to-connect-power-bi/power-bi-advanced-query-editing.png)](media/how-to-connect-power-bi/power-bi-advanced-query-editing.png#lightbox)

1. Modifique la carga de JSON según sea necesario.
1. Seleccione **Listo** y, después, **Cerrar y aplicar** en la **ventana del editor Power Query**.

Ahora, en la interfaz se reflejarán los cambios que haya aplicado.  

## <a name="next-steps"></a>Pasos siguientes

* Más información acerca de los [conceptos de Power BI Connector](https://docs.microsoft.com/power-bi/desktop-query-overview) para Azure Time Series Insights.

* Más información acerca de [Power BI Desktop](https://docs.microsoft.com/power-bi/desktop-query-overview).
