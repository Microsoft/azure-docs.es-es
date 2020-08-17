---
title: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/16/2020
ms.author: tamram
ms.openlocfilehash: 96d8100f2cffcfb001a693575128ce19e742225d
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534146"
---
Antes de asignar un rol de Azure a una entidad de seguridad, determine el ámbito de acceso que debería tener la entidad de seguridad. Los procedimientos recomendados dictan que siempre es mejor conceder únicamente el ámbito más restringido posible.

En la lista siguiente se describen los niveles en los que puede definir el ámbito de acceso a recursos de blob y cola de Azure, empezando por el ámbito más restringido:

- **Un contenedor individual.** En este ámbito, se aplica una asignación de roles a todos los blobs del contenedor, así como las propiedades y los metadatos del contenedor.
- **Una cola individual.** En este ámbito, se aplica una asignación de roles a los mensajes de la cola, así como las propiedades y los metadatos de la cola.
- **La cuenta de almacenamiento.** En este ámbito, se aplica una asignación de roles a todos los contenedores y sus blobs, o bien a todas las colas y sus mensajes.
- **El grupo de recursos.** En este ámbito, se aplica una asignación de roles a todos los contenedores o colas de todas las cuentas de almacenamiento del grupo de recursos.
- **La suscripción.** En este ámbito, se aplica una asignación de roles a todos los contenedores o las colas de todas las cuentas de almacenamiento de todos los grupos de recursos de la suscripción.
- **Un grupo de administración.** En este ámbito, se aplica una asignación de roles a todos los contenedores o colas de todas las cuentas de almacenamiento de todos los grupos de recursos de todas las suscripciones del grupo de administración.

Para más información sobre las asignaciones de roles de Azure y su ámbito, consulte [¿Qué es el control de acceso basado en rol de Azure (Azure RBAC)?](../articles/role-based-access-control/overview.md)
