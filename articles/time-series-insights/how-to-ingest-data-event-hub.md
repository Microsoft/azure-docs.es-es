---
title: 'Adición de un origen de eventos de Event Hubs: Azure Time Series Insights | Microsoft Docs'
description: Aprenda a agregar un origen de eventos de Azure Event Hubs a un entorno de Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: v-mamcge, jasonh, kfile
ms.workload: big-data
ms.topic: conceptual
ms.date: 06/30/2020
ms.custom: seodec18
ms.openlocfilehash: ee94a36ea27a15067cbcbab22b10629bc4b37634
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095558"
---
# <a name="add-an-event-hub-event-source-to-your-azure-time-series-insights-environment"></a>Incorporación de un origen de eventos de Event Hubs al entorno de Azure Time Series Insights

Este artículo describe cómo usar Azure Portal para agregar un origen de eventos que lee datos de Azure Event Hubs en su entorno de Azure Time Series Insights.

> [!NOTE]
> Los pasos que se describen en este artículo son válidos tanto en entornos de Azure Time Series Insights G1 como en entornos de Azure Time Series Insights G2.

## <a name="prerequisites"></a>Requisitos previos

- Cree un entorno de Azure Time Series Insights según lo explicado en [Creación de un entorno de Azure Time Series Insights](./time-series-insights-update-create-environment.md).
- Cree un centro de eventos. Lea [Creación de un espacio de nombres de Event Hubs y un centro de eventos con Azure Portal](../event-hubs/event-hubs-create.md).
- El centro de eventos tiene que tener eventos de mensajes activos que se le hayan enviado. Encontrará más información en [Envío de eventos a Azure Event Hubs mediante .NET Framework](../event-hubs/event-hubs-dotnet-framework-getstarted-send.md).
- Cree un grupo de consumidores dedicado en el centro de eventos para que el entorno de Azure Time Series Insights los consuma. Cada origen del evento de Azure Time Series Insights debe tener su propio grupo de consumidores dedicado que no se comparte con otros consumidores. Si varios lectores consumen eventos desde el mismo grupo de consumidores, es probable que todos los lectores exhiban errores. Hay un límite de 20 grupos de consumidores por centro de eventos. Si desea información más detallada, lea la [Guía de programación de Event Hubs](../event-hubs/event-hubs-programming-guide.md).

### <a name="add-a-consumer-group-to-your-event-hub"></a>Incorporación de un grupo de consumidores al centro de eventos

Las aplicaciones usan grupos de consumidores para extraer datos de Azure Event Hubs. Para leer datos de manera confiable del centro de eventos, proporcione un grupo de consumidores dedicado, solo para su uso en este entorno de Azure Time Series Insights.

Para agregar un nuevo grupo de consumidores al centro de eventos:

1. En [Azure Portal](https://portal.azure.com), busque y abra la instancia del centro de eventos desde el panel **Información general** del espacio de nombres del centro de eventos. Seleccione **Entidades > Event Hubs** o busque su instancia en **Nombre**.

    [![Apertura del espacio de nombres del centro de eventos](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-connect-event-hub-namespace.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-connect-event-hub-namespace.png#lightbox)

1. En la instancia del centro de eventos, seleccione **Entidades > Grupos de consumidores**. Luego, seleccione **+ Grupo de consumidores** para agregar un nuevo grupo de consumidores. 

   [![Centro de eventos: Incorporación de un grupo de consumidores](media/time-series-insights-how-to-add-an-event-source-eventhub/add-event-hub-consumer-group.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/add-event-hub-consumer-group.png#lightbox)

   En caso contrario, seleccione un grupo de consumidores existente y vaya a la sección siguiente.

1. En la página **Grupos de consumidores**, proporcione un nuevo valor exclusivo para **Nombre**.  Use este mismo nombre al crear un nuevo origen del evento en el entorno de Azure Time Series Insights.

1. Seleccione **Crear**.

## <a name="add-a-new-event-source"></a>Adición de un nuevo origen del evento

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Busque su entorno de Azure Time Series Insights existente. En el menú de la izquierda, seleccione **Todos los recursos** y después el entorno de Azure Time Series Insights.

1. Seleccione **Orígenes de eventos** y, luego, seleccione **Agregar**.

   [![En Orígenes de eventos, seleccione el botón Agregar](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-add-an-event-source.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-add-an-event-source.png#lightbox)

1. Especifique un valor para **Nombre de origen del evento** que sea exclusivo para este entorno de Azure Time Series Insights, como `Contoso-TSI-Gen 1-Event-Hub-ES`.

1. En **Origen**, seleccione **Centro de eventos**.

1. Seleccione los valores apropiados para **Opción de importación**:

   * Cuando ya tenga un centro de eventos en una de sus suscripciones seleccione **Use Event Hub from available subscriptions** (Usar el centro de eventos de las suscripciones disponibles). Esta opción presenta el enfoque más sencillo.

     [![Selección de una opción de importación de origen de eventos](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-event-hub-select-import-option.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-event-hub-select-import-option.png#lightbox)

    *  En la siguiente tabla se explican las propiedades necesarias para la opción **Use Event Hub from available subscriptions** (Usar el centro de eventos de las suscripciones disponibles):

       [![Detalles del centro de eventos y la suscripción](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-configure-create-confirm.png)](media/time-series-insights-how-to-add-an-event-source-eventhub/tsi-configure-create-confirm.png#lightbox)

       | Propiedad | Descripción |
       | --- | --- |
       | Suscripción | La suscripción a la que pertenece la instancia y el espacio de nombres del centro de eventos deseado. |
       | Espacio de nombres del centro de eventos | El espacio de nombres del centro de eventos al que pertenece la instancia de centro de eventos deseada. |
       | Nombre del centro de eventos | Nombre de la instancia de centro de eventos deseada. |
       | Valor de la directiva del centro de eventos | Seleccione la directiva de acceso compartido deseada. Puede crear la directiva de acceso compartido en el centro de eventos en la pestaña **Configurar**. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. La directiva de acceso compartido para el origen de eventos *debe* tener permisos de **lectura**. |
       | Clave de la directiva del Centro de eventos | Se rellena previamente a partir del valor de la directiva del centro de eventos seleccionado. |

    * Si el centro de eventos es externo a sus suscripciones o si quiere elegir opciones avanzadas, seleccione **Indicar manualmente la configuración del Centro de eventos**.

       En la siguiente tabla se explican las propiedades necesarias para la opción **Indicar manualmente la configuración del Centro de eventos**:
 
       | Propiedad | Descripción |
       | --- | --- |
       | Id. de suscripción | La suscripción a la que pertenece la instancia y el espacio de nombres del centro de eventos deseado. |
       | Resource group | El grupo de recursos al que pertenece la instancia y el espacio de nombres del centro de eventos deseado. |
       | Espacio de nombres del centro de eventos | El espacio de nombres del centro de eventos al que pertenece la instancia de centro de eventos deseada. |
       | Nombre del centro de eventos | Nombre de la instancia de centro de eventos deseada. |
       | Valor de la directiva del centro de eventos | Seleccione la directiva de acceso compartido deseada. Puede crear la directiva de acceso compartido en el centro de eventos en la pestaña **Configurar**. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. La directiva de acceso compartido para el origen de eventos *debe* tener permisos de **lectura**. |
       | Clave de la directiva del Centro de eventos | La clave de acceso compartido que se usa para autenticar el acceso al espacio de nombres de Service Bus. Especifique la clave principal o secundaria aquí. |

    * Ambas opciones comparten las siguientes opciones de configuración:

       | Propiedad | Descripción |
       | --- | --- |
       | Grupo de consumidores de Event Hubs | El grupo de consumidores que lee eventos del centro de eventos. Es muy recomendable usar un grupo de consumidores dedicado para el origen del evento. |
       | Formato de serialización de eventos | Actualmente, JSON es el único formato de serialización disponible. Los mensajes de eventos tienen que estar en este formato, de lo contrario no se podrá leer ningún dato. |
       | Nombre de la propiedad de marca de tiempo | Para determinar este valor, es necesario que comprenda el formato de mensaje de los datos del mensaje que se envía al centro de eventos. Este valor es el **nombre** de la propiedad específica del evento en los datos del mensaje que quiere usar como marca de tiempo del evento. El valor distingue mayúsculas de minúsculas. Si se deja en blanco, se usa la **hora de puesta en cola del evento** como marca de tiempo del evento en el origen del evento. |

1. Agregue el nombre del grupo de consumidores dedicado de Azure Time Series Insights que agregó al centro de eventos.

1. Seleccione **Crear**.

   Una vez que se haya creado el origen del evento, Azure Time Series Insights iniciará automáticamente la transmisión de datos al entorno.

## <a name="next-steps"></a>Pasos siguientes

* [Defina las directivas de acceso a datos](time-series-insights-data-access.md) para proteger los datos.

* Realice el [envío de eventos](time-series-insights-send-events.md) al origen del evento.

* Acceda al entorno en el [explorador de Azure Time Series Insights](https://insights.timeseries.azure.com).
