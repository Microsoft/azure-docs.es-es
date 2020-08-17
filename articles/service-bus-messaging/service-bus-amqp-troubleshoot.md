---
title: Solución de errores de AMQP en Azure Service Bus | Microsoft Docs
description: Proporciona una lista de errores de AMQP que pueden aparecer al usar Azure Service Bus y la causa de esos errores.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 88b10940e0b910f50e6ccf7f8c53134fa7f0ba2f
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88064356"
---
# <a name="amqp-errors-in-azure-service-bus"></a>Errores de AMQP en Azure Service Bus
En este artículo se indican algunos de los errores que aparecen al usar AMQP con Azure Service Bus. Corresponden todos a comportamientos estándar del servicio. Para evitarlos, puede efectuar llamadas de envío o recepción en la conexión o el vínculo, lo que hace que estos se vuelvan a crear automáticamente.

## <a name="link-is-closed"></a>Se cierra el vínculo 
Aparecerá el siguiente error cuando la conexión de AMQP y el vínculo estén activos pero no se realicen llamadas (por ejemplo, envío o recepción) con el vínculo durante 10 minutos. Esto hace que se cierre el vínculo. La conexión sigue abierta.

```
amqp:link:detach-forced:The link 'G2:7223832:user.tenant0.cud_00000000000-0000-0000-0000-00000000000000' is force detached by the broker due to errors occurred in publisher(link164614). Detach origin: AmqpMessagePublisher.IdleTimerExpired: Idle timeout: 00:10:00. TrackingId:00000000000000000000000000000000000000_G2_B3, SystemTracker:mynamespace:Topic:MyTopic, Timestamp:2/16/2018 11:10:40 PM
```

## <a name="connection-is-closed"></a>Se cierra la conexión
Aparecerá el siguiente error en la conexión de AMQP cuando se hayan cerrado todos los vínculos de la conexión porque no ha habido ninguna actividad (inactivo) y no se haya creado un nuevo vínculo en 5 minutos.

```
Error{condition=amqp:connection:forced, description='The connection was inactive for more than the allowed 300000 milliseconds and is closed by container 'LinkTracker'. TrackingId:00000000000000000000000000000000000_G21, SystemTracker:gateway5, Timestamp:2019-03-06T17:32:00', info=null}
```

## <a name="link-is-not-created"></a>No se crea el vínculo 
Aparecerá este error cuando se cree una nueva conexión de AMQP, pero no se cree un vínculo antes de que haya transcurrido 1 minuto.

```
Error{condition=amqp:connection:forced, description='The connection was inactive for more than the allowed 60000 milliseconds and is closed by container 'LinkTracker'. TrackingId:0000000000000000000000000000000000000_G21, SystemTracker:gateway5, Timestamp:2019-03-06T18:41:51', info=null}
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre AMQP y Service Bus, visite los siguientes vínculos:

* [Información general sobre AMQP para Service Bus]
* [Guía del protocolo AMQP 1.0]
* [AMQP de Service Bus para Windows Server]

[Información general sobre AMQP para Service Bus]: service-bus-amqp-overview.md
[Guía del protocolo AMQP 1.0]: service-bus-amqp-protocol-guide.md
[AMQP de Service Bus para Windows Server]: /previous-versions/service-bus-archive/dn282144(v=azure.100)