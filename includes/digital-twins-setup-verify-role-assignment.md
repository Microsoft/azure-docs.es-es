---
author: baanders
description: archivo de inclusión para comprobar la asignación de roles en la configuración de Azure Digital Twins
ms.service: digital-twins
ms.topic: include
ms.date: 7/22/2020
ms.author: baanders
ms.openlocfilehash: e651b02bf72ced58b6cba637a68ace3258514176
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405602"
---
Una manera de comprobar que configuró correctamente la asignación de roles es consultar las asignaciones de roles para la instancia de Azure Digital Twins en [Azure Portal](https://portal.azure.com). Vaya a la página de las [instancias de Azure Digital Twins](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.DigitalTwins%2FdigitalTwinsInstances) del portal (puede usar este vínculo o buscarla en la barra de búsqueda del portal) y seleccione el nombre de la instancia que quiere comprobar. 

Luego, consulte todos sus roles asignados en *Control de acceso (IAM) > Asignaciones de roles*. El usuario debe aparecer en la lista con un rol de *Propietario de Azure Digital Twins (versión preliminar)* . 

:::image type="content" source="../articles/digital-twins/media/how-to-set-up-instance/portal/verify-role-assignment.png" alt-text="Vista de las asignaciones de roles para una instancia de Azure Digital Twins en Azure Portal":::