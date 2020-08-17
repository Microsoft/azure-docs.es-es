---
title: 'Planeamiento de un entorno de Gen2: Azure Time Series Insights | Microsoft Docs'
description: Procedimientos recomendados para configurar, administrar, planear e implementar el entorno de Azure Time Series Insights Gen2.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 06/30/2020
ms.custom: seodec18
ms.openlocfilehash: 76e49393b1d26e6db85146a204911ba164d3ffc0
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289915"
---
# <a name="plan-your-azure-time-series-insights-gen2-environment"></a>Planeamiento del entorno de Azure Time Series Insights Gen2

En este artículo se describen los procedimientos recomendados para planear y comenzar a usar rápidamente la versión preliminar de Azure Time Series Insights Gen2.

## <a name="best-practices-for-planning-and-preparation"></a>Procedimientos recomendados para el planeamiento y la preparación

En los artículos siguientes se describen con más detalles los procedimientos recomendados para planear y preparar el entorno:

* Lo que se obtiene cuando se [aprovisiona un entorno de Azure Time Series Insights Gen2](#the-gen2-environment).
* Cuáles son las [propiedades IDs y Timestamp de Times Series](#configure-time-series-ids-and-timestamp-properties).
* Cuál es el nuevo [modelo de Times Series](#understand-the-time-series-model) y cómo crear el suyo.
* Cómo [enviar eventos de forma eficaz en JSON](#shape-your-events).
* [Opciones comerciales de recuperación ante desastres](#business-disaster-recovery) de Azure Time Series Insights.

Azure Time Series Insights emplea un modelo comercial de pago por uso. Para más información sobre la capacidad y los cargos, lea [Precios de Azure Time Series Insights](https://azure.microsoft.com/pricing/details/time-series-insights/).

## <a name="the-gen2-environment"></a>El entorno de Gen2

Cuando se aprovisiona entorno de Azure Time Series Insights Gen2, se crean también dos recursos de Azure:


* Un entorno de Azure Time Series Insights Gen2
* Una cuenta de Azure Storage.

Como parte del proceso de aprovisionamiento, especifique si desea habilitar un almacenamiento intermedio. El almacenamiento intermedio proporciona una experiencia de consulta en niveles. Cuando esté habilitado, debe especificar un período de retención entre 7 y 30 días. Las consultas que se ejecutan dentro del período de retención del almacenamiento intermedio generalmente proporcionan tiempos de respuesta más rápidos. Cuando una consulta abarca el período de retención de almacenamiento intermedio, se sirve desde el almacenamiento en reposo.

Las consultas que están en el almacenamiento intermedio son gratuitas, mientras que las del almacenamiento en reposo generan costos. Es importante comprender los modelos de consulta y planear la configuración del almacenamiento intermedio correspondiente. Se recomienda que los análisis interactivos de los datos más recientes residan en el almacenamiento intermedio y el análisis de patrones y las tendencias a largo plazo residan en reposo.

> [!NOTE]
> Para obtener más información sobre cómo consultar los datos semiactivos, lea la [referencia de API](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute#uri-parameters).

Para empezar, necesita tres elementos adicionales:

* Un [modelo de Time Series](./concepts-model-overview.md)
* Un [origen de eventos conectado a Time Series Insights](./concepts-streaming-ingestion-event-sources.md)
* [Eventos que fluyen hacia el origen del evento](./time-series-insights-send-events.md) que se asignan al modelo y están en formato JSON válido

## <a name="review-azure-time-series-insights-gen2-limits"></a>Revisión de los límites de Azure Time Series Insights Gen2

[!INCLUDE [Review Azure Time Series Insights Gen2 limits](../../includes/time-series-insights-preview-limits.md)]

## <a name="configure-time-series-ids-and-timestamp-properties"></a>Configurar las propiedades IDs y Timestamp de Time Series

Para crear un nuevo entorno de Azure Time Series Insights, seleccione un identificador de Time Series. Al hacer esto se crea una partición lógica para los datos. Como se indicó, asegúrese de tener listos los identificadores de serie temporal.

> [!IMPORTANT]
> Los identificadores de serie temporal *no se pueden cambiar más adelante*. Compruebe cada uno de ellos antes de pasar a la selección final y el primer uso.

Puede seleccionar hasta tres claves para diferenciar los recursos de forma única. Para más información, consulte [Procedimientos recomendados para elegir un identificador de serie temporal](./time-series-insights-update-how-to-id.md) y [Almacenamiento y entrada](concepts-json-flattening-escaping-rules.md).

La propiedad **Timestamp** también es importante. Esta propiedad se puede designar al agregar orígenes de eventos. Cada origen de eventos tiene una propiedad Timestamp opcional que se usa para realizar el seguimiento de los orígenes de eventos con el tiempo. Los valores de Timestamp distinguen mayúsculas de minúsculas y su formato debe seguir la especificación de cada origen de evento.

> [!TIP]
> Compruebe los requisitos de formato y análisis de los orígenes de eventos.

Si se deja en blanco, la hora de puesta en cola del evento se usa como marca de tiempo del evento. Si envía datos históricos o eventos por lotes, personalizar la propiedad Timestamp es más útil que la hora predeterminada de puesta en cola del evento. Para más información, lea acerca de cómo [agregar orígenes de eventos en Azure IoT Hub](./time-series-insights-how-to-add-an-event-source-iothub.md).

## <a name="understand-the-time-series-model"></a>El modelo de serie temporal

Ahora puede configurar el modelo de serie temporal del entorno de Azure Time Series Insights. El nuevo modelo facilita la búsqueda y el análisis de los datos de IoT. Además, permite la protección, el mantenimiento y el enriquecimiento de datos de serie temporal y ayuda a preparar los conjuntos de datos listos para el consumidor. El modelo usa identificadores de Time Series, que se asignan a una instancia que asocia el recurso único con variables, conocidas como tipos, y jerarquías. Lea la información general sobre el [modelo de serie temporal](./concepts-model-overview.md) para más detalles.

El modelo es dinámico, por lo que se puede compilar en cualquier momento. Para empezar a trabajar rápidamente, compílelo y cárguelo antes de insertar datos en Azure Time Series Insights. Para compilar el modelo, lea [Modelado de datos en la versión preliminar de Azure Time Series Insights](/azure/time-series-insights/concepts-model-overview).

Para muchos clientes, el modelo de serie temporal se asigna a un modelo de recursos existente o a un sistema ERP ya implantado. Si no tiene un modelo existente, se [proporciona](https://github.com/Microsoft/tsiclient) una experiencia de usuario ya creada para ponerse en marcha rápidamente. Para saber cómo puede ayudarle un modelo, vea el [entorno de demostración de ejemplo](https://insights.timeseries.azure.com/preview/demo).

## <a name="shape-your-events"></a>Moldear los eventos

Puede comprobar el modo en que los eventos se envían a Azure Time Series Insights. Lo ideal es que los eventos se desnormalicen bien y de manera eficaz.

Una buena regla general es la siguiente:

* Almacenar los metadatos en el modelo de Time Series.
* Asegúrese de que el modo de serie temporal, los campos de instancia y los eventos incluyen únicamente la información necesaria, como los identificadores de serie temporal o la propiedad Timestamp.

Para más información, lea [Moldeo de eventos](./time-series-insights-send-events.md#supported-json-shapes).

[!INCLUDE [business-disaster-recover](../../includes/time-series-insights-business-recovery.md)]

## <a name="next-steps"></a>Pasos siguientes

- Revise [Azure Advisor](../advisor/advisor-overview.md) para planear las opciones de configuración de recuperación de negocio.
- Revise [Azure Advisor](../advisor/advisor-overview.md) para planear las opciones de configuración de recuperación de negocio.
- Obtenga más información sobre la [ingesta de datos](./concepts-ingestion-overview.md) en Azure Time Series Insights Gen2.
- Revise el artículo sobre el [almacenamiento de datos](./concepts-storage.md) en Azure Time Series Insights Gen2.
- Obtenga información sobre el [modelado de datos](./concepts-model-overview.md) en Azure Time Series Insights Gen2.
