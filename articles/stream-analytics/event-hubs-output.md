---
title: Salida de Event Hubs desde Azure Stream Analytics
description: En este artículo se describe cómo generar datos desde Azure Stream Analytics a Azure Event Hubs.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: d18d4aa4bf9306bcdd667faa53f0d888c090e2fd
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88875538"
---
# <a name="event-hubs-output-from-azure-stream-analytics"></a>Salida de Event Hubs desde Azure Stream Analytics

El servicio [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/) es un agente de ingesta de eventos de publicación-suscripción altamente escalable. Puede recopilar millones de eventos por segundo. Un uso del centro de eventos como salida es cuando la salida de un trabajo de Stream Analytics se convierte en la entrada de otro trabajo de streaming. Para obtener información sobre el tamaño máximo del mensaje y la optimización del tamaño del lote, consulte la sección [Tamaño de lote de salida](#output-batch-size).

## <a name="output-configuration"></a>Configuración de salida

La tabla siguiente contiene los parámetros necesarios para configurar las transmisiones de datos desde Event Hubs como salida.

| Nombre de la propiedad | Descripción |
| --- | --- |
| Alias de salida | Nombre descriptivo usado en las consultas para dirigir la salida de la consulta a este centro de eventos. |
| Espacio de nombres del centro de eventos | Contenedor para un conjunto de entidades de mensajería. Cuando crea un nuevo centro de eventos, también se crea un espacio de nombres de centro de eventos. |
| Nombre del centro de eventos | Nombre de la salida del centro de eventos. |
| Nombre de la directiva del centro de eventos | Directiva de acceso compartido, que puede crear en la pestaña **Configurar** del centro de eventos. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. |
| Clave de la directiva de centro de eventos | Clave de acceso compartido usada para autenticar el acceso al espacio de nombres del centro de eventos. |
| Columna de clave de partición | Opcional. Columna que contiene la clave de partición para la salida del centro de eventos. |
| Formato de serialización de eventos | Formato de serialización para los datos de salida. Se admiten JSON, CSV y Avro. |
| Encoding | Por el momento, UTF-8 es el único formato de codificación compatible para CSV y JSON. |
| Delimitador | Solo se aplica para la serialización de CSV. Stream Analytics admite un número de delimitadores comunes para la serialización de datos en formato CSV. Los valores admitidos son la coma, punto y coma, espacio, tabulador y barra vertical. |
| Formato | Solo se aplica para la serialización de JSON. La opción **Separado por líneas** especifica que en el formato de la salida cada objeto JSON está separado por una línea nueva. Si selecciona **Separado por líneas**, el objeto JSON se lee un objeto a la vez. El contenido total en sí mismo no sería un JSON válido. La opción **Matriz** especifica que el formato de la salida es una matriz de objetos JSON.  |
| Columnas de propiedades | Opcional. Columnas separadas por comas que necesitan agregarse como propiedades de usuario del mensaje saliente, en lugar de la carga útil. Puede obtener más información sobre esta característica en la sección [Propiedades de metadatos personalizadas para la salida](#custom-metadata-properties-for-output). |

## <a name="partitioning"></a>Creación de particiones

La creación de particiones varía según alineación de particiones. Cuando la clave de partición para la salida del centro de eventos está alineada equitativamente con el paso de consulta ascendente (anterior), el número de sistemas de escritura es el mismo que el número de particiones en la salida del centro de eventos. Cada sistema de escritura usa la [clase EventHubSender](/dotnet/api/microsoft.servicebus.messaging.eventhubsender?view=azure-dotnet) para enviar eventos a la partición específica. Cuando la clave de partición para la salida del centro de eventos no está alineada con el paso de consulta ascendente (anterior), el número de sistemas de escritura es el mismo que el número de particiones del paso anterior. Cada sistema de escritura usa [la clase SendBatchAsync](/dotnet/api/microsoft.servicebus.messaging.eventhubclient.sendasync?view=azure-dotnet) de **EventHubClient** para enviar eventos a todas las particiones de salida. 

## <a name="output-batch-size"></a>Tamaño de lote de salida

El tamaño máximo del mensaje es 256 KB o 1 MB por mensaje. Para obtener más información, consulte los [límites de Event Hubs](../event-hubs/event-hubs-quotas.md). Cuando las particiones de entrada-salida no se alinean, cada evento se empaqueta individualmente en `EventData` y se envía en un lote que tiene como límite el tamaño máximo de mensaje. Esto también sucede si se usan las [propiedades de metadatos personalizadas](#custom-metadata-properties-for-output). Cuando las particiones de entrada-salida se alinean, varios eventos se empaquetan en una misma instancia de `EventData` con el tamaño máximo de mensaje como límite y se envían.

## <a name="custom-metadata-properties-for-output"></a>Propiedades de metadatos personalizadas para la salida

Puede asociar las columnas de la consulta como propiedades de usuario a los mensajes salientes. Estas columnas no se envían a la carga útil. Las propiedades están presentes como diccionario en el mensaje de salida. *Key* es el nombre de columna y *value* es el valor de columna en el diccionario de propiedades. Se admiten todos los tipos de datos de Stream Analytics, excepto los tipos de registro y matriz.  

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure](quick-create-azure-cli.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante una plantilla de ARM](quick-create-azure-resource-manager.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-vs-code.md)
