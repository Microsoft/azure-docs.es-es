---
title: 'Envío de eventos a un entorno: Azure Time Series Insights | Microsoft Docs'
description: Aquí aprenderá a configurar un centro de eventos, a ejecutar una aplicación de ejemplo y a enviar eventos a un entorno de Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 08/12/2020
ms.custom: seodec18
ms.openlocfilehash: 3a5ee1cc8efead7c29dadaf64adb8e2686a10621
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88168089"
---
# <a name="send-events-to-an-azure-time-series-insights-gen1-environment-by-using-an-event-hub"></a>Envío de eventos a un entorno de Azure Time Series Insights Gen1 mediante un centro de eventos

En este artículo se explica cómo crear y configurar un centro de eventos en Azure Event Hubs. También se describe cómo ejecutar una aplicación de ejemplo para insertar eventos en Azure Time Series Insights desde Event Hubs. Si tiene un centro de eventos con eventos en formato JSON, pase por alto este tutorial y vea su entorno en [Azure Time Series Insights](./time-series-insights-update-create-environment.md).

## <a name="configure-an-event-hub"></a>Configuración de un centro de eventos

1. Para aprender a crear un centro de eventos, lea la [documentación de Event Hubs](https://docs.microsoft.com/azure/event-hubs/).
1. En el cuadro de búsqueda, busque **Event Hubs**. Haga clic en **Event Hubs** en la lista devuelta.
1. Seleccione su centro de eventos.
1. Cuando se crea un centro de eventos, se crea un espacio de nombres del centro de eventos. Si todavía no ha creado un centro de eventos en el espacio de nombres, vaya al menú y, en **Entities** (Entidades), cree un centro de eventos.  

    [![Lista de centros de eventos](media/send-events/tsi-connect-event-hub-namespace.png)](media/send-events/tsi-connect-event-hub-namespace.png#lightbox)

1. Después de crear un centro de eventos, selecciónelo en la lista de centros de eventos.
1. En el menú, en **Entities** (Entidades), seleccione **Event Hubs**.
1. Seleccione el nombre del centro de eventos para configurarlo.
1. En **Introducción**, seleccione **Grupos de consumidores** y, a continuación, **Grupo de consumidores**.

    [![Creación de un grupo de consumidores](media/send-events/add-event-hub-consumer-group.png)](media/send-events/add-event-hub-consumer-group.png#lightbox)

1. Asegúrese de crear un grupo de consumidores que se use exclusivamente con el origen de eventos de Azure Time Series Insights.

    > [!IMPORTANT]
    > Asegúrese de que ningún otro servicio usa este grupo de consumidores, como un trabajo de Azure Stream Analytics u otro entorno de Azure Time Series Insights. Si otros servicios usan el grupo de consumidores, las operaciones de lectura se ven afectadas negativamente en este entorno y en los otros servicios. Si usa **$Default** como grupo de consumidores, otros lectores podrían volver a usar el grupo de consumidores.

1. En el menú, en **Settings** (Configuración), seleccione **Shared access policies** (Directivas de acceso compartido) y, luego, **Add** (Agregar).

    [![Seleccionar las directivas de acceso compartido y luego del botón Add (Agregar)](media/send-events/add-shared-access-policy.png)](media/send-events/add-shared-access-policy.png#lightbox)

1. En el panel **Add new shared access policy** (Agregar nueva directiva de acceso compartido), cree un acceso compartido denominado **MySendPolicy**. Usará esta directiva de acceso compartido para enviar eventos en los ejemplos de C# más adelante en este artículo.

    [![Escribir un nombre en el cuadro Policy name (Nombre de la directiva)](media/send-events/configure-shared-access-policy-confirm.png)](media/send-events/configure-shared-access-policy-confirm.png#lightbox)

1. En **Claim** (Reclamar), active la casilla **Send** (Enviar).

## <a name="add-an-azure-time-series-insights-instance"></a>Adición de una instancia de Azure Time Series Insights

En Azure Time Series Insights Gen2 se pueden agregar datos contextuales a la telemetría entrante mediante el modelo de serie temporal (TSM). En TSM, las etiquetas o señales se denominan *instancias*, y se pueden almacenar los datos contextuales en los *campos de instancia*. Los datos se unen en el momento de la consulta mediante un **id. de serie temporal**. El **identificador de serie temporal** del proyecto de ejemplo de los molinos de viento que se usará más adelante en este artículo es `id`. Para más información sobre el almacenamiento de datos en campos de instancia, lea la información general acerca del [modelo de serie temporal](./concepts-model-overview.md).

### <a name="create-an-azure-time-series-insights-event-source"></a>Creación de un origen del evento de Azure Time Series Insights

1. Complete los pasos para [crear un origen de eventos](https://docs.microsoft.com/azure/time-series-insights/time-series-insights-how-to-add-an-event-source-eventhub) si aún no ha creado uno.

1. Establezca el valor de `timeSeriesId`. Para más información sobre el **id. de serie temporal**, lea [Modelos de Time Series](./concepts-model-overview.md).

### <a name="push-events-to-windmills-sample"></a>Eventos push del ejemplo de los molinos de viento

1. Busque **Event Hubs** en la barra de búsqueda. Haga clic en **Event Hubs** en la lista devuelta.

1. Seleccione su instancia de centro de eventos.

1. Vaya a **Directivas de acceso compartido** > **MySendPolicy**. Copie el valor de **Connection string-primary key**.

    [![Copiar el valor de la cadena de conexión de la clave principal](media/send-events/configure-sample-code-connection-string.png)](media/send-events/configure-sample-code-connection-string.png#lightbox)

1. Ir a <https://tsiclientsample.azurewebsites.net/windFarmGen.html>. La dirección URL crea y ejecuta dispositivos simulados de molinos de viento.
1. En el cuadro **Event Hub Connection String** (Cadena de conexión del centro de eventos) de la página web, pegue la cadena de conexión que copió en el [campo de entrada de molinos de viento](#push-events-to-windmills-sample).
  
    [![Pegar la cadena de conexión de la clave principal en el cuadro de la cadena de conexión del centro de eventos](media/send-events/configure-wind-mill-sim.png)](media/send-events/configure-wind-mill-sim.png#lightbox)

1. Seleccione **Click to start** (Haga clic para iniciar).

    > [!TIP]
    > El simulador de molino de viento también crea código JSON que se puede usar como carga útil con las [API de consulta de Azure Time Series Insights GA](https://docs.microsoft.com/rest/api/time-series-insights/gen1-query).

    > [!NOTE]
    > El simulador seguirá enviando datos hasta que se cierre la pestaña del explorador.

1. Vuelva a su centro de eventos en Azure Portal. En la página **Información general**, se muestran los eventos nuevos que recibe el centro de eventos.

    [![Una página de información general del centro de eventos que muestra las métricas del centro de eventos](media/send-events/review-windmill-telemetry.png)](media/send-events/review-windmill-telemetry.png#lightbox)

## <a name="supported-json-shapes"></a>Formas de JSON admitidas

### <a name="example-one"></a>Ejemplo 1

* **Entrada**: Un objeto JSON simple.

    ```JSON
    {
        "id":"device1",
        "timestamp":"2016-01-08T01:08:00Z"
    }
    ```

* **Salida**: Un evento.

    |id|timestamp|
    |--------|---------------|
    |device1|2016-01-08T01:08:00Z|

### <a name="example-two"></a>Ejemplo 2

* **Entrada**: Una matriz JSON con dos objetos JSON. Cada objeto JSON se convierte en un evento.

    ```JSON
    [
        {
            "id":"device1",
            "timestamp":"2016-01-08T01:08:00Z"
        },
        {
            "id":"device2",
            "timestamp":"2016-01-17T01:17:00Z"
        }
    ]
    ```

* **Salida**: Dos eventos.

    |id|timestamp|
    |--------|---------------|
    |device1|2016-01-08T01:08:00Z|
    |device2|2016-01-08T01:17:00Z|

### <a name="example-three"></a>Ejemplo 3

* **Entrada**: Un objeto JSON con una matriz JSON anidada que contiene dos objetos JSON.

    ```JSON
    {
        "location":"WestUs",
        "events":[
            {
                "id":"device1",
                "timestamp":"2016-01-08T01:08:00Z"
            },
            {
                "id":"device2",
                "timestamp":"2016-01-17T01:17:00Z"
            }
        ]
    }
    ```

* **Salida**: Dos eventos. La propiedad **location** se copia en cada uno de los eventos.

    |ubicación|events.id|events.timestamp|
    |--------|---------------|----------------------|
    |WestUs|device1|2016-01-08T01:08:00Z|
    |WestUs|device2|2016-01-08T01:17:00Z|

### <a name="example-four"></a>Ejemplo 4

* **Entrada**: Un objeto JSON con una matriz JSON anidada que contiene dos objetos JSON. Esta entrada muestra que las propiedades globales se pueden representar mediante el objeto JSON complejo.

    ```JSON
    {
        "location":"WestUs",
        "manufacturer":{
            "name":"manufacturer1",
            "location":"EastUs"
        },
        "events":[
            {
                "id":"device1",
                "timestamp":"2016-01-08T01:08:00Z",
                "data":{
                    "type":"pressure",
                    "units":"psi",
                    "value":108.09
                }
            },
            {
                "id":"device2",
                "timestamp":"2016-01-17T01:17:00Z",
                "data":{
                    "type":"vibration",
                    "units":"abs G",
                    "value":217.09
                }
            }
        ]
    }
    ```

* **Salida**: Dos eventos.

    |ubicación|manufacturer.name|manufacturer.location|events.id|events.timestamp|events.data.type|events.data.units|events.data.value|
    |---|---|---|---|---|---|---|---|
    |WestUs|manufacturer1|EastUs|device1|2016-01-08T01:08:00Z|presión|psi|108.09|
    |WestUs|manufacturer1|EastUs|device2|2016-01-08T01:17:00Z|vibration|abs G|217.09|

## <a name="next-steps"></a>Pasos siguientes

* [Vea el entorno](https://insights.timeseries.azure.com) en el explorador de Azure Time Series Insights.

* Más información sobre los [mensajes del dispositivo de IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-construct)
