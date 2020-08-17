---
title: Adición de un origen de eventos de IoT Hub a Azure Time Series Insights | Microsoft Docs
description: Aprenda a agregar un origen del evento de IoT Hub a su entorno de Azure Time Series Insights.
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
ms.openlocfilehash: e963c092b968476d20e25482cbe165234f7e86f0
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86528188"
---
# <a name="add-an-iot-hub-event-source-to-your-azure-time-series-insight-environment"></a>Adición de un origen del evento de IoT Hub a su entorno de Azure Time Series Insights

Este artículo describe cómo usar Azure Portal para agregar un origen de eventos que lee datos de Azure IoT Hub en su entorno de Time Series Insights.

> [!NOTE]
> Las instrucciones de este artículo se aplican a los entornos de Azure Time Series Insights Gen 1 y de Azure Time Series Insight Gen 2.

## <a name="prerequisites"></a>Requisitos previos

* Creación de un [entorno de Azure Time Series Insights](time-series-insights-update-create-environment.md).
* [Creación de una instancia de IoT Hub mediante Azure Portal](../iot-hub/iot-hub-create-through-portal.md).
* El centro de IoT Hub debe tener eventos de mensajes activos en proceso de envío.
* Cree un grupo de consumidores dedicado en IoT Hub del que el entorno de Time Series Insight pueda consumir. Cada origen del evento de Azure Time Series Insights debe tener su propio grupo de consumidores dedicado que no se comparte con otros consumidores. Si varios lectores consumen eventos desde el mismo grupo de consumidores, es probable que todos los lectores exhiban errores. Para más información, lea la [Guía para desarrolladores de IoT Hub](../iot-hub/iot-hub-devguide.md).

### <a name="add-a-consumer-group-to-your-iot-hub"></a>Adición de un grupo de consumidores a IoT Hub

Las aplicaciones usan grupos de consumidores para extraer datos de Azure IoT Hub. Para leer datos de manera confiable de IoT Hub, especifique un grupo de consumidores dedicado que use solo este entorno de Azure Time Series Insights.

Para agregar un nuevo grupo de consumidores a la instancia de IoT Hub:

1. En [Azure Portal](https://portal.azure.com), busque y abra el centro de IoT Hub.

1. En el menú, en **Configuración**, seleccione **Puntos de conexión integrados** y, luego, seleccione el punto de conexión **Eventos**.

   [![Selección del botón Eventos en la página Puntos de conexión integrados](media/time-series-insights-how-to-add-an-event-source-iothub/tsi-connect-iot-hub.png)](media/time-series-insights-how-to-add-an-event-source-iothub/tsi-connect-iot-hub.png#lightbox)

1. En **Grupos de consumidores**, escriba un nombre único para el grupo de consumidores. Use este mismo nombre en el entorno de Azure Time Series Insights al crear un origen del evento.

1. Seleccione **Guardar**.

## <a name="add-a-new-event-source"></a>Adición de un nuevo origen del evento

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En el menú izquierdo, seleccione **Todos los recursos**. Seleccione el entorno de Azure Time Series Insights.

1. En **Configuración**, seleccione **Orígenes de eventos** y luego **Agregar**.

   [![Selección de Orígenes de eventos y del botón Agregar](media/time-series-insights-how-to-add-an-event-source-iothub/tsi-add-event-source.png)](media/time-series-insights-how-to-add-an-event-source-iothub/tsi-add-event-source.png#lightbox)

1. En el panel **Nuevo origen del evento**, en **Nombre del origen del evento**, escriba un nombre que sea exclusivo para este entorno de Azure Time Series Insights. Por ejemplo, escriba **event-stream**.

1. En **Origen**, seleccione **IoT Hub**.

1. Seleccione un valor para **Opción de importación**:

   * Si ya tiene un centro de IoT en una de sus suscripciones, seleccione **Usar IoT Hub desde las suscripciones disponibles**. Esta opción presenta el enfoque más sencillo.
   
     [![Selección de opciones en el panel Nuevo origen del evento](media/time-series-insights-how-to-add-an-event-source-iothub/tsi-select-an-import-option.png)](media/time-series-insights-how-to-add-an-event-source-iothub/tsi-select-an-import-option.png#lightbox)

    * En la siguiente tabla se explican las propiedades necesarias para la opción **Usar IoT Hub desde las suscripciones disponibles**:

       [![Panel Nuevo origen del evento: propiedades que hay que establecer en la opción Usar IoT Hub desde la opción de suscripciones disponibles](media/time-series-insights-how-to-add-an-event-source-iothub/tsi-create-configure-confirm.png)](media/time-series-insights-how-to-add-an-event-source-iothub/tsi-create-configure-confirm.png#lightbox)

       | Propiedad | Descripción |
       | --- | --- |
       | Suscripción | La suscripción a la que pertenece el centro de IoT deseado. |
       | Nombre de IoT Hub | El nombre del centro de IoT seleccionado. |
       | Nombre de la directiva de IoT Hub | Seleccione la directiva de acceso compartido. Puede encontrar esta directiva en la pestaña Configuración de centro de IoT. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. La directiva de acceso compartido para el origen de eventos *debe* tener permisos de **conexión de servicios**. |
       | Clave de la directiva de IoT Hub | La clave se rellena previamente. |

    * Si el centro de IoT es externo a sus suscripciones, o si desea elegir opciones avanzadas, seleccione **Proporcionar configuración del centro de IoT de forma manual**.

      En la siguiente tabla se explican las propiedades necesarias para la opción **Proporcionar configuración del Centro de IoT de forma manual**:

       | Propiedad | Descripción |
       | --- | --- |
       | Id. de suscripción | La suscripción a la que pertenece el centro de IoT deseado. |
       | Resource group | El nombre del grupo de recursos en el que se creó este centro de IoT. |
       | Nombre de IoT Hub | El nombre del centro de IoT. Cuando creó el centro de IoT especificó un nombre para este. |
       | Nombre de la directiva de IoT Hub | La directiva de acceso compartido. Puede crear esta directiva en la pestaña Configuración de centro de IoT. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. La directiva de acceso compartido para el origen de eventos *debe* tener permisos de **conexión de servicios**. |
       | Clave de la directiva de IoT Hub | La clave de acceso compartido que se usa para autenticar el acceso al espacio de nombres de Azure Service Bus. Especifique la clave principal o secundaria aquí. |

    * Ambas opciones comparten las siguientes opciones de configuración:

       | Propiedad | Descripción |
       | --- | --- |
       | Grupo de consumidores de IoT Hub | El grupo de consumidores que lee eventos del centro de IoT. Es muy recomendable usar un grupo de consumidores dedicado para el origen del evento. |
       | Formato de serialización de eventos | Actualmente, JSON es el único formato de serialización disponible. Los mensajes de eventos deberán estar en este formato o no se podrá leer ningún dato. |
       | Nombre de la propiedad de marca de tiempo | Para determinar este valor, es necesario que comprenda el formato de mensaje de los datos del mensaje que se envía al centro de IoT. Este valor es el **nombre** de la propiedad específica del evento en los datos del mensaje que quiere usar como marca de tiempo del evento. El valor distingue mayúsculas de minúsculas. Si se deja en blanco, se usa la **hora de puesta en cola del evento** como marca de tiempo del evento en el origen del evento. |


1. Agregue el nombre del grupo de consumidores de Azure Time Series Insight dedicado que agregó a su centro de IoT.

1. Seleccione **Crear**.

1. Una vez que haya creado el origen del evento, Azure Time Series Insights inicia automáticamente la transmisión de datos al entorno.

## <a name="next-steps"></a>Pasos siguientes

* [Defina las directivas de acceso a datos](time-series-insights-data-access.md) para proteger los datos.

* Realice el [envío de eventos](time-series-insights-send-events.md) al origen del evento.

* Acceso al entorno en el [explorador de Azure Time Series Insights](https://insights.timeseries.azure.com).
