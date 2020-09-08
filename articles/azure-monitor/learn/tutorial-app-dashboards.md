---
title: Creación de paneles personalizados en Azure Application Insights | Microsoft Docs
description: Tutorial para crear paneles de indicadores clave de rendimiento (KPI) personalizados con Azure Application Insights.
ms.subservice: application-insights
ms.topic: tutorial
author: lgayhardt
ms.author: lagayhar
ms.date: 07/3/2019
ms.custom: mvc
ms.openlocfilehash: 1b6fdbdfaef601751961d022858545d731fe62a8
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88856014"
---
# <a name="create-custom-kpi-dashboards-using-azure-application-insights"></a>Creación de paneles de indicadores clave de rendimiento (KPI) personalizados con Azure Application Insights

Puede crear varios paneles en Azure Portal de manera que cada uno incluya iconos en los que se visualicen datos procedentes de varios recursos de Azure en diferentes suscripciones y grupos de recursos.  Puede anclar distintos gráficos y vistas de visión de Azure Application Insights para crear paneles personalizados que le proporcionen una descripción completa del estado y el rendimiento de la aplicación. Este tutorial le guía a través de la creación de un panel personalizado que incluye varios tipos de datos y visualizaciones de Azure Application Insights.

 Aprenderá a:

> [!div class="checklist"]
> * Crear un panel personalizado en Azure
> * Agregar un icono desde la Galería de iconos
> * Agregar métricas estándar de Application Insights al panel
> * Agregar un gráfico de métricas personalizado de Application Insights al panel
> * Agregar los resultados de una consulta de Registros (Analytics) al panel

## <a name="prerequisites"></a>Prerequisites

Para completar este tutorial:

- Implemente una aplicación de .NET en Azure y [habilite el SDK de Application Insights](../app/asp-net.md).

> [!NOTE]
> Los permisos necesarios para trabajar con paneles se describen en el artículo [Descripción del control de acceso para los paneles](../../azure-portal/azure-portal-dashboard-share-access.md#understanding-access-control-for-dashboards).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-new-dashboard"></a>Creación de un nuevo panel
> [!WARNING]
> Si mueve el recurso de Application Insights a un grupo de recursos o una suscripción diferentes, tendrá que actualizar manualmente el panel; para ello, deberá quitar los iconos antiguos y anclar otros nuevos del mismo recurso de Application Insights en la nueva ubicación. 

Un solo panel puede contener recursos de varias aplicaciones, grupos de recursos y suscripciones.  Comience el tutorial creando un nuevo panel para la aplicación.  

1. En el panel, seleccione **Nuevo panel**.

   ![Nuevo panel](media/tutorial-app-dashboards/1newdashboard.png)

1. Escriba un nombre para el panel.
1. Eche un vistazo a la **Galería de iconos** para ver la variedad de iconos que se pueden agregar al panel.  Además de agregar iconos de la galería, puede anclar gráficos y otras vistas directamente desde Application Insights al panel.
1. Busque el icono **Markdown** y arrástrelo hasta el panel.  Este icono le permite agregar texto en formato Markdown, que es idóneo para agregar texto descriptivo al panel.
1. Agregue texto a las propiedades del icono y cambie su tamaño en el lienzo del panel.
    
    ![Edición del mosaico de Markdown](media/tutorial-app-dashboards/2dashboard-text.png)

1. Haga clic en **Personalización finalizada** en la parte superior de la pantalla para salir del modo de personalización de iconos.

## <a name="add-health-overview"></a>Adición de información general de estado
Un panel con texto estático no resulta muy interesante; así pues, agregue ahora un icono de Application Insights para mostrar información acerca de la aplicación.  Puede agregar iconos de Application Insights desde la Galería de iconos, o puede anclarlos directamente desde las pantallas de Application Insights.  Esto le permite configurar gráficos y vistas con las que ya está familiarizado antes de anclarlos al panel.  Empiece por agregar la información general de estado estándar para la aplicación.  Esto no requiere configuración y permite una personalización mínima en el panel.


1. Seleccione el recurso de **Application Insights** en la pantalla de inicio.
2. En el panel **Información general**, haga clic en el icono de la chincheta ![icono de chincheta](media/tutorial-app-dashboards/pushpin.png) para agregar el icono al último panel que ha visto.  
 
3. En la esquina superior derecha aparecerá una notificación que indica que el icono estaba anclado al panel. Haga clic en **Anclada al panel** en la notificación para volver al panel o use el panel.
4. El icono ya se ha agregado al panel. Seleccione **Editar** para cambiar el posicionamiento del icono. Haga clic y arrástrelo hasta su posición y haga clic en **Personalización finalizada**. Ahora, el panel tiene un icono con información útil.

    ![Panel con Escala de tiempo con información general](media/tutorial-app-dashboards/4dashboard-edit.png)

## <a name="add-custom-metric-chart"></a>Adición del gráfico de métricas personalizado
El panel **Métricas** panel le permite representar una métrica recopilada por Application Insights a lo largo del tiempo con filtros opcionales y agrupación.  Al igual que todo lo demás en Application Insights, puede agregar este gráfico al panel.  Para ello se requiere antes un poco de personalización.

1. Seleccione el recurso de **Application Insights** en la pantalla de inicio.
1. Seleccione **Métricas**.  
2. Ya se ha creado un gráfico vacío, y se le pedirá que agregue una métrica.  Agregue una métrica al gráfico y, opcionalmente, agregue un filtro y una agrupación.  En el ejemplo siguiente se muestra el número de solicitudes de servidor agrupadas según se hayan completado correctamente o no.  De este modo se obtiene una vista de ejecución de las solicitudes correctas e incorrectas.

    ![Adición de métrica](media/tutorial-app-dashboards/metrics.png)

4. Seleccione **Anclar al panel** a la derecha. De este modo, se agrega la vista al último panel con el que estaba trabajando.

3.  En la esquina superior derecha aparecerá una notificación que indica que el icono estaba anclado al panel. Haga clic en **Anclada al panel** en la notificación para volver al panel o use el panel del panel.

4. El icono ya se ha agregado al panel. Seleccione **Editar** para cambiar el posicionamiento del icono. Haga clic y arrástrelo hasta su posición y haga clic en **Personalización finalizada**.

## <a name="add-logs-analytics-query"></a>Adición de consultas de Registros (Analytics)
Registros (Analytics) de Azure Application Insights proporciona un lenguaje de consulta completo que le permite analizar todos los datos recopilados por Application Insights. Al igual que los gráficos y otras vistas, puede agregar el resultado de una consulta de registros al panel.

Dado que Logs (Analytics) de Azure Applications Insights es un servicio independiente, debe compartir el panel para que incluya una consulta de registros. Al compartir un panel de Azure, se publica como un recurso de Azure que puede poner a disposición de otros usuarios y recursos.  

1. En la parte superior de la pantalla del panel, haga clic en **Compartir**.

    ![Publicación del panel](media/tutorial-app-dashboards/8dashboard-share.png)

2. Mantenga el mismo **Nombre del panel** y seleccione el **Nombre de la suscripción** para compartir el panel.  Haga clic en **Publicar**.  El panel ahora está disponible para otros servicios y suscripciones.  Si lo desea, puede definir los usuarios específicos que deben tener acceso al panel.
1. Seleccione el recurso de **Application Insights** en la pantalla de inicio.
2. Haga clic en **Registros (Analytics)** en la parte izquierda debajo de Supervisión para abrir el portal de Registros (Analytics).
3. Escriba la siguiente consulta, que devuelve las 10 páginas más solicitadas y su número de solicitudes:

    ``` Kusto
    requests
    | summarize count() by name
    | sort by count_ desc
    | take 10
    ```

4. Haga clic en **Ejecutar** para validar los resultados de la consulta.
5. Hacer clic en el icono de chincheta ![Icono de chincheta](media/tutorial-app-dashboards/pushpin.png) y seleccione el nombre del panel. La razón por la que esta opción le solicita que seleccione un panel, a diferencia de los pasos anteriores en los que se utilizaba el último panel, es que la consola de Registros (Analytics) es un servicio independiente y debe seleccionar entre todos los paneles compartidos disponibles.

5. Antes de volver al panel, agregue otra consulta, pero esta vez represéntela como un gráfico para ver las distintas maneras de visualizar una consulta de registros en un panel. Comience con la siguiente consulta, que resume las 10 operaciones principales con la mayoría de las excepciones.

    ``` Kusto
    exceptions
    | summarize count() by operation_Name
    | sort by count_ desc
    | take 10
    ```

6. Seleccione **Gráfico** y, a continuación, cambie a **Anillo** para visualizar el resultado.

    ![Gráfico de Registros (Analytics)](media/tutorial-app-dashboards/11querychart.png)

6. Hacer clic en el icono de chincheta ![Icono de chincheta](media/tutorial-app-dashboards/pushpin.png) en la parte superior derecha para anclar el gráfico al panel y, esta vez, seleccione el vínculo para volver al panel.
4. Los resultados de las consultas se agregan ahora al panel en el formato que seleccionó.  Haga clic todos ellos y arrástrelos hasta su posición, y haga clic en **Personalización finalizada**.
5. Seleccione el icono de lápiz ![Icono de lápiz](media/tutorial-app-dashboards/pencil.png) en cada título para darles un título descriptivo.

5. Seleccione **Compartir** para volver a publicar los cambios en el panel, que ahora incluye una serie de gráficos y visualizaciones de Application Insights.


## <a name="next-steps"></a>Pasos siguientes
Ahora que ha aprendido a crear paneles personalizados, eche un vistazo al resto de la documentación de Application Insights, que incluye un caso práctico.

> [!div class="nextstepaction"]
> [Diagnósticos detallados](../app/devops.md)
