---
author: SnehaGunda
ms.service: cosmos-db
ms.topic: include
ms.date: 8/12/2020
ms.author: sngun
ms.openlocfilehash: 5c5ed4e1bbe54c642202c19e1beb61de94b4f751
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88515197"
---
**1. ¿Cómo se notificará a los clientes la retirada del SDK?**

Microsoft notificará con 12 meses de antelación la finalización del soporte técnico del SDK que se retira para facilitar una transición sin problemas a un SDK compatible. Además, se notificará a los clientes a través de diversos canales de comunicación: Azure Portal, actualizaciones de Azure y comunicación directa a los administradores de servicios asignados.

**2. ¿Pueden los clientes crear aplicaciones con un SDK de Azure Cosmos DB de "posible" retirada durante el período de 12 meses?** 

Sí, los clientes tendrán pleno acceso para crear, implementar y modificar aplicaciones mediante el SDK de Azure Cosmos DB de "posible" retirada durante el período de aviso de 12 meses. Durante este período de aviso, se recomienda a los clientes migrar a una versión compatible más reciente del SDK de Azure Cosmos DB, según corresponda. 

**3. Tras la fecha de retirada, ¿qué ocurre con las aplicaciones en ejecución que usan el SDK de Azure Cosmos DB no compatible?** 

Después de la fecha de retirada, Azure Cosmos DB ya no hará correcciones de errores, agregará nuevas características ni proporcionará soporte técnico para las versiones del SDK retiradas. Si prefiere no realizar la actualización, el servicio Azure Cosmos DB seguirá atendiendo las solicitudes enviadas desde las versiones retiradas del SDK. 

**4. ¿Qué versiones del SDK tendrán las características y actualizaciones más recientes?**

Las nuevas características y actualizaciones solo se agregarán a la versión secundaria más reciente de la versión del SDK principal compatible más reciente. Se recomienda usar siempre la versión más reciente para aprovechar las ventajas de las nuevas características, mejoras de rendimiento y correcciones de errores. Si usa una versión anterior del SDK que no se haya retirado, las solicitudes realizadas en Azure Cosmos DB seguirán funcionando, pero no se tendrá acceso a las nuevas funcionalidades.  

**5. ¿Qué debo hacer si no puedo actualizar la aplicación antes de una fecha límite?**

Se recomienda que actualice al SDK más reciente tan pronto como sea posible. Una vez etiquetado un SDK para la retirada, tendrá 12 meses para actualizar la aplicación. Si no puede realizar la actualización en la fecha de retirada, Azure Cosmos DB seguirá atendiendo las solicitudes enviadas desde las versiones retiradas del SDK, por lo que las aplicaciones en ejecución seguirán funcionando. Sin embargo, Azure Cosmos DB ya no hará correcciones de errores, agregará nuevas características ni proporcionará soporte técnico para las versiones del SDK retiradas. 

Si tiene un plan de soporte técnico y requiere soporte técnico, [póngase en contacto con nosotros](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) abriendo una incidencia de soporte técnico.
    


