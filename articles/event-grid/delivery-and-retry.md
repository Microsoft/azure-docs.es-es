---
title: Entrega y reintento de entrega de Azure Event Grid
description: Describe cómo Azure Event Grid entrega eventos y cómo administra los mensajes no entregados.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: fe7574d7e17b1763afb2292c15007dd87b056ef1
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087618"
---
# <a name="event-grid-message-delivery-and-retry"></a>Entrega y reintento de entrega de mensajes de Event Grid

En este artículo se describe cómo Azure Event Grid administra los eventos cuando no se confirma la entrega.

Event Grid ofrece entrega duradera. Entrega cada mensaje por lo menos una vez en cada suscripción. Los eventos se envían inmediatamente al punto de conexión registrado de cada suscripción. Si un punto de conexión no acusa recibo de un evento, Event Grid reintenta la entrega del evento.

## <a name="batched-event-delivery"></a>Entrega de eventos por lotes

De forma predeterminada, Event Grid envía cada evento individualmente a los suscriptores. El suscriptor recibe una matriz con un solo evento. Puede configurar Event Grid para procesar por lotes los eventos para su entrega con el fin de mejorar el rendimiento HTTP en escenarios de alto rendimiento.

La entrega por lotes tiene dos opciones:

* **Número máximo de eventos por lote**: es el número máximo de eventos que Event Grid entregará por lote. No se superará nunca este número; sin embargo, se pueden entregar menos eventos si no hay otros eventos disponibles en el momento de la publicación. Event Grid no retrasa los eventos para crear un lote si hay menos eventos disponibles. Debe estar entre 1 y 5 000.
* **Tamaño de lote preferido en kilobytes**: es el límite superior de destino para el tamaño de lote en kilobytes. Al igual que el número máximo de eventos, el tamaño del lote puede ser menor si no hay más eventos disponibles en el momento de la publicación. Es posible que un lote sea mayor que el tamaño de lote preferido *si* un solo evento es mayor que el tamaño preferido. Por ejemplo, si el tamaño preferido es 4 KB y se inserta un evento de 10 KB en Event Grid, el evento de 10 KB se seguirá entregando en su propio lote en lugar de ser eliminado.

La entrega por lotes se configura en función de una suscripción por evento mediante el portal, la CLI, PowerShell o los SDK.

### <a name="azure-portal"></a>Azure Portal: 
![Configuración de la entrega por lotes](./media/delivery-and-retry/batch-settings.png)

### <a name="azure-cli"></a>Azure CLI
Al crear una suscripción a eventos, use los parámetros siguientes: 

- **max-events-per-batch**: número máximo de eventos en un lote. Debe ser un número entre 1 y 5000.
- **preferred-batch-size-in-kilobytes**: tamaño de lote preferido en kilobytes. Debe ser un número entre 1 y 1024.

```azurecli
storageid=$(az storage account show --name <storage_account_name> --resource-group <resource_group_name> --query id --output tsv)
endpoint=https://$sitename.azurewebsites.net/api/updates

az eventgrid event-subscription create \
  --resource-id $storageid \
  --name <event_subscription_name> \
  --endpoint $endpoint \
  --max-events-per-batch 1000 \
  --preferred-batch-size-in-kilobytes 512
```

Para más información sobre el uso de la CLI de Azure con Event Grid, consulte [Enrutamiento de eventos de almacenamiento a un punto de conexión web con la CLI de Azure](../storage/blobs/storage-blob-event-quickstart.md).

## <a name="retry-schedule-and-duration"></a>Programación y duración de los reintentos

Event Grid espera 30 segundos para obtener una respuesta después de entregar un mensaje. Después de 30 segundos, si el punto de conexión no ha respondido, el mensaje se pone en cola para volver a intentarlo. Event Grid usa una directiva de reintentos de retroceso exponencial para la entrega de eventos. Event Grid reintenta la entrega en el siguiente horario y cuando sea el mejor momento:

- 10 segundos
- 30 segundos
- 1 minuto.
- 5 minutos
- 10 minutos
- 30 minutos
- 1 hora
- Cada hora, hasta 24 horas

Si el punto de conexión responde en 3 minutos, Event Grid intentará eliminar el evento de la cola de reintentos en el mejor momento posible, pero aún se pueden recibir duplicados.

Event Grid agrega una pequeña selección aleatoria en todos los pasos de reintento y puede omitir oportunamente ciertos reintentos si un punto de conexión es incorrecto, está inactivo durante un largo período o parece estar demasiado ocupado.

Para un comportamiento determinista, establezca el tiempo que estará activo el evento y los intentos de entrega máximos en las [directivas de reintentos de suscripción](manage-event-delivery.md).

De forma predeterminada, Event Grid expira todos los eventos que no se entregan en 24 horas. Puede [personalizar la directiva de reintentos](manage-event-delivery.md) al crear una suscripción al evento. Proporcione el número máximo de intentos de entrega (el valor predeterminado es 30) y el periodo de vida de los eventos (el valor predeterminado es 1440 minutos).

## <a name="delayed-delivery"></a>Entrega retrasada

Cuando un punto de conexión experimenta errores de entrega, Event Grid comienza a retrasar la entrega y a reintentar los eventos a ese punto de conexión. Por ejemplo, si se produce un error en los 10 primeros eventos publicados en un punto de conexión, Event Grid asumirá que el punto de conexión está experimentando problemas y retrasará todos los reintentos posteriores y las *nuevas entregas* durante algún tiempo; en algunos casos, hasta varias horas.

El propósito funcional de la entrega retrasada es proteger los puntos de conexión incorrectos, así como el sistema Event Grid. Sin los mecanismos de retroceso y el retraso de la entrega a puntos de conexión incorrectos, la directiva de reintentos de Event Grid y las funcionalidades del volumen pueden sobrecargar fácilmente un sistema.

## <a name="dead-letter-events"></a>Eventos fallidos
Cuando Event Grid no puede entregar un evento en un período de tiempo determinado o después de intentar entregarlo un número concreto de veces, puede enviar el evento sin entregar a una cuenta de almacenamiento. Este proceso se conoce como **colas de mensajes fallidos**. Event Grid pone en la cola de mensajes fallidos un evento cuando se cumple **una de las siguientes** condiciones. 

- El evento no se entrega en el período de tiempo de vida
- El número de intentos de entrega del evento ha superado el límite

Si se cumple alguna de las condiciones, el evento se quita o pone en la cola de mensajes fallidos.  De forma predeterminada, Event Grid no tiene activada esta opción. Para habilitarla, debe especificar una cuenta de almacenamiento para contener los eventos no entregados al crear la suscripción a eventos. Puede extraer eventos de esta cuenta de almacenamiento para resolver las entregas.

Event Grid envía un evento a la ubicación de la cola de mensajes fallidos cuando ha intentado todos los reintentos. Si Event Grid recibe un código de respuesta 400 (solicitud incorrecta) o 413 (entidad de solicitud demasiado grande), envía inmediatamente el evento al punto de conexión de la cola de mensajes fallidos. Estos códigos de respuesta indican que la entrega del evento nunca se realizará correctamente.

Hay un retraso de cinco minutos entre el último intento de entregar un evento y su entrega a la ubicación de mensajes fallidos. Este retraso está pensado para reducir el número de operaciones de almacenamiento de blobs. Si la ubicación de la cola de mensajes fallidos no está disponible durante cuatro horas, se elimina el evento.

Antes de establecer la ubicación de mensajes fallidos, debe tener una cuenta de almacenamiento con un contenedor. Puede proporcionar el punto de conexión de este contenedor al crear la suscripción a eventos. El punto de conexión tiene este formato: `/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.Storage/storageAccounts/<storage-name>/blobServices/default/containers/<container-name>`

Es posible que desee recibir una notificación cuando un evento se envía a la ubicación de la cola de mensajes fallidos. Para utilizar Event Grid para responder a eventos no entregados, [cree una suscripción a eventos](../storage/blobs/storage-blob-event-quickstart.md?toc=%2fazure%2fevent-grid%2ftoc.json) para el almacenamiento de blobs de los eventos fallidos. Cada vez que el almacenamiento de blobs de eventos fallidos recibe un evento no entregado, Event Grid lo notifica al controlador. El controlador responderá con las acciones que debe realizar para conciliar los eventos no entregados.

Para obtener un ejemplo de cómo configurar una ubicación de la cola de mensajes fallidos, consulte [Cola de mensajes fallidos y directivas de reintento](manage-event-delivery.md).

## <a name="message-delivery-status"></a>Estado de entrega de mensajes

Event Grid usa códigos de respuesta HTTP para acusar recibo de eventos. 

### <a name="success-codes"></a>Códigos de éxito

Event Grid **solo** considera entregas correctas los siguientes códigos de respuesta HTTP. El resto de los códigos de estado se consideran entregas con errores y se reintentarán o procesarán como entregas devueltas. Al recibir un código de estado correcto, Event Grid considera que la entrega ha finalizado.

- 200 OK
- 201 Creado
- 202 - Aceptado
- 203 Información no autoritativa
- 204 No Content

### <a name="failure-codes"></a>Códigos de error

Todos los demás códigos que no están en el conjunto anterior (200-204) se consideran errores y se reintentarán. Algunos tienen directivas de reintento específicas, que se describen a continuación y todos las demás siguen el modelo de interrupción exponencial estándar. Es importante recordar que, debido a la naturaleza altamente paralela de la arquitectura de Event Grid, el comportamiento de reintento no es determinista. 

| status code | Comportamiento de reintento |
| ------------|----------------|
| 400 - Solicitud incorrecta | Reintentar después de 5 minutos o más (procesar inmediatamente como entrega devuelta si está configurada) |
| 401 No autorizado | Reintentar después de 5 minutos o más |
| 403 Prohibido | Reintentar después de 5 minutos o más |
| 404 No encontrado | Reintentar después de 5 minutos o más |
| Tiempo de espera de solicitud 408 | Reintentar después de 2 minutos o más |
| 413 Entidad de solicitud demasiado larga | Reintentar después de 10 segundos o más (procesar inmediatamente como entrega devuelta si está configurada) |
| Servicio no disponible 503 | Reintentar después de 30 segundos o más |
| Todos los demás | Reintentar después de 10 segundos o más |


## <a name="next-steps"></a>Pasos siguientes

* Para ver el estado de las entregas de eventos, consulte [Supervisar la entrega de mensajes de Event Grid](monitor-event-delivery.md).
* Para personalizar las opciones de entrega de eventos, consulte [Cola de mensajes fallidos y directivas de reintento](manage-event-delivery.md).
