---
title: 'Diagnóstico y solución de problemas de un entorno Gen2: Azure Time Series Insights | Microsoft Docs'
description: Aprenda a diagnosticar y solucionar problemas de un entorno de Azure Time Series Insights Gen2.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 06/30/2020
ms.custom: seodec18
ms.openlocfilehash: d9a4f7aa270aa4ed2b02e61da984e14379a241a9
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289922"
---
# <a name="diagnose-and-troubleshoot-an-azure-time-series-insights-gen2-environment"></a>Diagnóstico y solución de problemas de un entorno de Azure Time Series Insights Gen2

Este artículo resume varios problemas comunes que pueden surgir al trabajar con el entorno de Azure Time Series Insights Gen2. El artículo también describe posibles causas y soluciones para cada problema.

## <a name="problem-i-cant-find-my-environment-in-the-gen2-explorer"></a>Problema: no encuentro mi entorno en el explorador de Gen2

Este problema puede producirse si no tiene los permisos necesarios para acceder al entorno de Time Series Insights. El usuario necesita un rol de acceso de nivel lector para ver su entorno de Time Series Insights. Para comprobar los niveles de acceso actuales y conceder acceso adicional, vaya a la sección **Directivas de acceso a datos** del recurso de Time Series Insights en [Azure Portal](https://portal.azure.com/).

  [![Comprobación de las directivas de acceso a los datos.](media/preview-troubleshoot/verify-data-access-policies.png)](media/preview-troubleshoot/verify-data-access-policies.png#lightbox)

## <a name="problem-no-data-is-seen-in-the-gen2-explorer"></a>Problema: no se muestran datos en el explorador de Gen2

Hay varias razones por las cuales es posible que los datos no aparezcan en el [explorador de Azure Time Series Insights Gen2](https://insights.timeseries.azure.com/preview).

- Puede que el origen del evento no esté recibiendo datos.

    Compruebe que el origen del evento, que es un centro de eventos o un centro de IoT, está recibiendo datos de las etiquetas o las instancias. Para comprobarlo, vaya a la página de información general de Azure Portal.

    [![Revise la información general sobre las métricas del panel de información.](media/preview-troubleshoot/verify-dashboard-metrics.png)](media/preview-troubleshoot/verify-dashboard-metrics.png#lightbox)

- Los datos de origen del evento no tienen formato JSON.

    Time Series Insights solo admite datos JSON. Para ver ejemplos de JSON, lea [Formas de JSON admitidas](./how-to-shape-query-json.md).

- A la clave de origen del evento le falta un permiso necesario.

  * Para IoT Hub, debe proporcionar la clave con el permiso de **conexión de servicio**.

    [![Verifique los permisos de IoT Hub](media/preview-troubleshoot/verify-correct-permissions.png)](media/preview-troubleshoot/verify-correct-permissions.png#lightbox).

    * Las directivas **iothubowner** y el **servicio** funcionan porque tienen permiso **Conexión de servicios**.

  * Para una instancia de Event Hubs, debe proporcionar la clave con el permiso de **escucha**.
  
    [![Revise los permisos del centro de eventos.](media/preview-troubleshoot/verify-eh-permissions.png)](media/preview-troubleshoot/verify-eh-permissions.png#lightbox)

    * Las directivas **Leer** y **Administrar** funcionan porque tienen el permiso **Escuchar**.

- El grupo de consumidores proporcionado no es exclusivo de Time Series Insights.

    Durante el registro de un centro de IoT o un centro de eventos, se especifica el grupo de consumidores que se usa para leer los datos. Este grupo de consumidores debe ser único para cada entorno. Si el grupo de consumidor es compartido, el centro de eventos subyacente desconectará automáticamente a uno de los lectores de forma aleatoria. Proporcione un grupo de consumidor exclusivo del que Time Series Insights pueda leer.

- La propiedad Time Series ID especificada en el momento del aprovisionamiento es incorrecta, falta o es nula.

    Este problema puede producirse si la propiedad Time Series ID se configura incorrectamente en el momento de aprovisionamiento del entorno. Para más información, lea [Procedimientos recomendados para elegir un identificador de Time Series](./time-series-insights-update-how-to-id.md). En la actualidad, no se puede actualizar un entorno de Time Series Insights existente para usar un identificador de Time Series distinto.

## <a name="problem-some-data-shows-but-some-is-missing"></a>Problema: Se muestran algunos datos, pero faltan otros

Podría estar enviando datos sin el identificador de Time Series.

- Este problema puede producirse cuando se envían los eventos sin el campo de identificador de Time Series en la carga útil. Para más información, lea el artículo sobre las [formas compatibles de JSON](./how-to-shape-query-json.md).
- Este problema puede producirse porque se está limitando su entorno.

    > [!NOTE]
    > En este momento, Time Series Insights admite una tasa máxima de ingesta de 6 Mbps.

## <a name="problem-data-was-showing-but-now-ingestion-has-stopped"></a>Problema: Los datos se mostraban, pero la ingesta ahora se detuvo

- Es posible que se haya regenerado la clave de origen del evento y que el entorno Gen2 necesite la clave de origen del evento nueva.

Este problema se produce cuando la clave proporcionada al crear el origen del evento ya no es válida. Vería la telemetría en el centro, pero ningún mensaje de entrada recibido en Time Series Insights. Si no está seguro de si se ha regenerado la clave, puede buscar "crear o actualizar reglas de autorización de espacio de nombres" o "crear o actualizar recursos de IotHub" para IoT Hub en el registro de actividad de Event Hubs. 

Para actualizar el entorno de Time Series Insights Gen2 con la clave nueva, abra el recurso del centro en Azure Portal y copie la nueva clave. Vaya al recurso de TSI y haga clic en Orígenes de eventos. 

   [![Actualización de la clave.](media/preview-troubleshoot/update-hub-key-step-1.png)](media/preview-troubleshoot/update-hub-key-step-1.png#lightbox)

Seleccione los orígenes de eventos en los que se ha detenido la ingesta, pegue la nueva clave y haga clic en Guardar.

   [![Actualización de la clave.](media/preview-troubleshoot/update-hub-key-step-2.png)](media/preview-troubleshoot/update-hub-key-step-2.png#lightbox)

## <a name="problem-my-event-sources-timestamp-property-name-doesnt-work"></a>Problema: mi nombre de propiedad Timestamp del origen del evento no funciona

Asegúrese de que el nombre y el valor se ajustan a las reglas siguientes:

* El nombre de la propiedad Timestamp distingue mayúsculas de minúsculas.
* El valor de propiedad Timestamp que procede del origen del evento, como una cadena JSON, tiene el formato `yyyy-MM-ddTHH:mm:ss.FFFFFFFK`. Un ejemplo de este tipo de cadena es `"2008-04-12T12:53Z"`.

La manera más fácil de asegurarse de que el nombre de la propiedad timestamp se captura y funciona correctamente consiste en utilizar el explorador de Time Series Insights Gen2. En el explorador de Time Series Insights Gen2, utilice el gráfico para seleccionar un período de tiempo después de que proporcionara el nombre de la propiedad Timestamp. Haga clic con el botón derecho en la selección y elija la opción de **exploración de eventos**. El primer encabezado de columna es el nombre de propiedad Timestamp. Debe tener `($ts)` junto a la palabra `Timestamp`, en lugar de:

* `(abc)`, que indica que Time Series Insights lee los valores de datos como cadenas.
* El icono de **calendario**, que indica que Time Series Insights lee los valores de datos como datetime.
* `#`, que indica que Time Series Insights lee los valores de datos como un entero.

Si no se especifica explícitamente la propiedad Timestamp, se utiliza como marca de tiempo predeterminado la hora de puesta en cola de un centro de IoT o centro de eventos de un evento.

## <a name="problem-i-cant-view-data-from-my-warm-store-in-the-explorer"></a>Problema: no puedo ver los datos de mi almacenamiento intermedio en el explorador

- Es posible que haya aprovisionado recientemente el almacenamiento intermedio y que los datos sigan fluyendo.
- Es posible que haya eliminado el almacenamiento intermedio, en cuyo caso habría perdido datos.

## <a name="problem-i-cant-view-or-edit-my-time-series-model"></a>Problema: no puedo ver ni editar mi modelo de Time Series

- Puede que esté accediendo a un entorno Time Series Insights S1 o S2.

   Los modelos de Time Series solo se admiten en entornos de pago por uso. Para más información sobre cómo acceder a su entorno S1 o S2 desde el explorador de Time Series Insights Gen2, lea [Visualización de datos en el explorador](./time-series-insights-update-explorer.md).

   [![No hay eventos en el entorno.](media/preview-troubleshoot/troubleshoot-no-events.png)](media/preview-troubleshoot/troubleshoot-no-events.png#lightbox)

- Puede que no tenga permisos para ver y editar el modelo.

   Los usuarios necesitan acceso de nivel de colaborador para editar y ver su modelo de Time Series. Para comprobar los niveles de acceso actuales y conceder acceso adicional, vaya a la sección **Directivas de acceso a datos** del recurso de Time Series Insights en Azure Portal.

## <a name="problem-all-my-instances-in-the-gen2-explorer-lack-a-parent"></a>Problema: todas las instancias del explorador de Gen2 carecen de un elemento primario

Este problema puede producirse si el entorno no tiene una jerarquía de modelo de Time Series definida. Para más información, consulte cómo [trabajar con modelos de Time Series](/azure/time-series-insights/time-series-insights-overview).

  [![Las instancias no primarias mostrarán una advertencia.](media/preview-troubleshoot/unparented-instances.png)](media/preview-troubleshoot/unparented-instances.png#lightbox)

## <a name="problem-power-bi-connector-shows-unable-to-connect"></a>Problema: el conector de Power BI muestra "No se puede conectar"

Este problema se puede producir si no usa la versión más reciente del conector de Power BI en Power BI Desktop.

[![Las instancias no primarias mostrarán una advertencia.](media/preview-troubleshoot/power-bi-unable-to-connect.png)](media/preview-troubleshoot/power-bi-unable-to-connect.png#lightbox)

* Compruebe la versión de Power BI Desktop y asegúrese de que está usando la versión de julio de 2020. Si no es así, actualice Power BI Desktop y vuelva a ejecutar el conector. 

## <a name="next-steps"></a>Pasos siguientes

- Información sobre cómo [trabajar con modelos de Time Series](/azure/time-series-insights/time-series-insights-overview).

- Más información sobre las [formas JSON admitidas](./how-to-shape-query-json.md).

- Revise el [planeamiento y los límites](./time-series-insights-update-plan.md) en Azure Time Series Insights Gen2.
