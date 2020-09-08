---
author: baanders
description: archivo de inclusión para comprobar la asignación de roles en la configuración de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/22/2020
ms.author: baanders
ms.openlocfilehash: fdb3bd034d93e623037be9fa0721a805924af5c3
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88864843"
---
Una manera de comprobar que configuró correctamente la asignación de roles es consultar las asignaciones de roles para la instancia de Azure Digital Twins en [Azure Portal](https://portal.azure.com). Vaya a la instancia de Azure Digital Twins en Azure Portal (puede buscarla en la página de [instancias de Azure Digital Twins](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) o buscar su nombre en la barra de búsqueda del portal).

Luego, consulte todos sus roles asignados en *Control de acceso (IAM) > Asignaciones de roles*. El usuario debe aparecer en la lista con un rol de *Propietario de Azure Digital Twins (versión preliminar)* . 

:::image type="content" source="../articles/digital-twins/media/how-to-set-up-instance/portal/verify-role-assignment.png" alt-text="Vista de las asignaciones de roles para una instancia de Azure Digital Twins en Azure Portal":::