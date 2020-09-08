---
title: Salida de temas de Service Bus desde Azure Stream Analytics
description: En este artículo se describen los temas de Service Bus como salida de Azure Stream Analytics.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 71a41a3a8d137fdf9d4f41c975e7be6c148078e1
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88875480"
---
# <a name="service-bus-topics-output-from-azure-stream-analytics"></a>Salida de temas de Service Bus desde Azure Stream Analytics

Las colas de Service Bus proporcionan un método de comunicación uno a uno del emisor al receptor. Los [temas de Service Bus](https://msdn.microsoft.com/library/azure/hh367516.aspx) proporcionan una forma de comunicación de uno a varios.

En la siguiente tabla se enumeran los nombres de propiedad y su descripción para crear una salida de tema de Service Bus.

| Nombre de propiedad | Descripción |
| --- | --- |
| Alias de salida |Nombre descriptivo usado en las consultas para dirigir la salida de la consulta a este tema de Service Bus. |
| Espacio de nombres de Service Bus |Contenedor para un conjunto de entidades de mensajería. Al crear un nuevo centro de eventos, también se crea un espacio de nombres de Service Bus. |
| Nombre del tema |Los temas son entidades de mensajería, similares a los centros de eventos y las colas. Están diseñados para recopilar secuencias de eventos de dispositivos y servicios. Cuando se crea un tema, se proporciona también un nombre específico. Los mensajes enviados a un tema no están disponibles a menos que se cree una suscripción, así que asegúrese de que hay una o varias suscripciones en el tema. |
| Nombre de la directiva de temas |Cuando crea un tema de Service Bus, también puede crear directivas de acceso compartido en la pestaña **Configurar** del tema. Cada directiva de acceso compartido tiene un nombre, los permisos establecidos y las claves de acceso. |
| Clave de la directiva de temas |La clave de acceso compartido que se usa para autenticar el acceso al espacio de nombres de Service Bus. |
| Formato de serialización de eventos |Formato de serialización para los datos de salida. Se admiten JSON, CSV y Avro. |
| Encoding |Si usa el formato CSV o JSON, debe especificar una codificación. Por el momento, UTF-8 es el único formato de codificación compatible. |
| Delimitador |Solo se aplica para la serialización de CSV. Stream Analytics admite un número de delimitadores comunes para la serialización de datos en formato CSV. Los valores admitidos son la coma, punto y coma, espacio, tabulador y barra vertical. |
| Columnas de propiedades | Opcional. Columnas separadas por comas que necesitan agregarse como propiedades de usuario del mensaje saliente, en lugar de la carga útil. Puede obtener más información sobre esta característica en la sección [Propiedades de metadatos personalizadas para la salida](#custom-metadata-properties-for-output). |
| Columnas de propiedades del sistema | Opcional. Pares de clave-valor de las propiedades del sistema y los nombres de columna correspondientes que se deben adjuntar al mensaje saliente en lugar de a la carga. |

El número de particiones se [basa en la SKU y el tamaño de Service Bus](../service-bus-messaging/service-bus-partitioning.md). La clave de partición es un valor entero único para cada partición.

## <a name="partitioning"></a>Creación de particiones

La creación de particiones se elige automáticamente. El número de particiones se basa en [la SKU y el tamaño de Service Bus](../service-bus-messaging/service-bus-partitioning.md). La clave de partición es un valor entero único para cada partición. El número de escritores de salida es el mismo que el número de particiones en el tema de salida.

## <a name="output-batch-size"></a>Tamaño de lote de salida

El tamaño máximo del mensaje es 256 KB por mensaje para el nivel Estándar y 1 MB para el nivel Premium. Para más información, consulte los [límites de Service Bus](../service-bus-messaging/service-bus-quotas.md). Para optimizar, use un evento único por mensaje.

## <a name="custom-metadata-properties-for-output"></a>Propiedades de metadatos personalizadas para la salida

Puede asociar las columnas de la consulta como propiedades de usuario a los mensajes salientes. Estas columnas no se envían a la carga útil. Las propiedades están presentes como diccionario en el mensaje de salida. *Key* es el nombre de columna y *value* es el valor de columna en el diccionario de propiedades. Se admiten todos los tipos de datos de Stream Analytics, excepto los tipos de registro y matriz.

## <a name="system-properties"></a>Propiedades del sistema

Puede adjuntar columnas de consulta como [propiedades del sistema](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage?view=azure-dotnet#properties) a los mensajes salientes de cola y tema de Service Bus. Estas columnas no entran en la carga; en su lugar, la [propiedad del sistema](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage?view=azure-dotnet#properties) BrokeredMessage correspondiente se rellena con los valores de la columna de consulta.
Se admiten estas propiedades del sistema: `MessageId, ContentType, Label, PartitionKey, ReplyTo, SessionId, CorrelationId, To, ForcePersistence, TimeToLive, ScheduledEnqueueTimeUtc`.

Los valores de cadena de estas columnas se analizan como el tipo de valor de propiedad del sistema correspondiente y los errores de análisis se tratan como errores de datos.
Este campo se proporciona como un formato de objeto JSON. Los detalles sobre este formato son los siguientes:

* Rodeado de llaves {}.
* Escrito en pares de clave-valor.
* Las claves y los valores deben ser cadenas.
* La clave es el nombre de la propiedad del sistema y el valor es el nombre de la columna de consulta.
* Las claves y los valores se separan por el signo de dos puntos.
* Los pares de clave-valor están separados entre ellos por una coma.

A continuación se muestra cómo utilizar esta propiedad:

* Consulta: `select *, column1, column2 INTO queueOutput FROM iotHubInput`
* Columnas de propiedades del sistema: `{ "MessageId": "column1", "PartitionKey": "column2"}`

Esto establece `MessageId` en los mensajes de cola de Service Bus con los valores de `column1` y PartitionKey se establece con los valores de `column2`.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante la CLI de Azure](quick-create-azure-cli.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics mediante una plantilla de ARM](quick-create-azure-resource-manager.md)
* [Inicio rápido: Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics con Visual Studio](stream-analytics-quick-create-vs.md)
* [Inicio rápido: Creación de un trabajo de Azure Stream Analytics en Visual Studio Code](quick-create-vs-code.md)
