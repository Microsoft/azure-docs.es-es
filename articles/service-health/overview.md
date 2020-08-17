---
title: ¿Qué es Azure Service Health?
description: Información personalizada sobre cómo las aplicaciones de Azure se ven afectadas por el mantenimiento y los problemas de servicios de Azure actuales y futuros.
ms.topic: overview
ms.date: 05/10/2019
ms.openlocfilehash: cb1eb8eb75a18a73e8f024f02217df37f45981fe
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499779"
---
# <a name="what-is-azure-service-health"></a>¿Qué es Azure Service Health?

Azure ofrece un conjunto de experiencias para mantenerle informado sobre el estado de los recursos en la nube. Esta información incluye los problemas actuales y futuros como eventos que afectan al servicio, el mantenimiento planeado y otros cambios que pueden afectar a la disponibilidad.

Azure Service Health es una combinación de tres servicios independientes más pequeños.

[Estado de Azure](azure-status-overview.md) informa de las interrupciones de servicio en Azure en la **[página de estado de Azure](https://status.azure.com)** . La página es una vista global del estado de todos los servicios y regiones de Azure. La página de estado es una buena referencia para incidentes con un impacto masivo, pero se recomienda encarecidamente a los usuarios de Azure actuales que aprovechan Azure Service Health para mantenerse informados sobre los incidentes y el mantenimiento de Azure.

[Service Health](service-health-overview.md) proporciona una vista personalizada del estado de los servicios y regiones de Azure que usa. Es el mejor lugar para buscar comunicaciones que afecten a los servicios relativas a interrupciones, actividades de mantenimiento planeado y otros avisos de mantenimiento, ya que tras la autenticación, Service Health conoce los servicios y recursos que usa en la actualidad. La mejor forma de usar Service Health es configurar sus alertas para que le envíen notificaciones a través de sus canales de comunicación preferidos cuando los problemas del servicio, el mantenimiento planeado u otros cambios puedan afectar a los servicios y regiones de Azure que utilice.

[Resource Health](resource-health-overview.md) proporciona información sobre el estado de los recursos en la nube individuales, como una instancia de máquina virtual concreta. Mediante Azure Monitor también puede configurar alertas que le notifiquen los cambios de disponibilidad de los recursos en la nube. Resource Health, junto con las notificaciones de Azure Monitor, le ayudarán a estar mejor informado acerca de la disponibilidad de los recursos minuto a minuto y a evaluar rápidamente si un problema se le puede achacar a usted o está relacionado con un evento de plataforma de Azure.

Conjuntamente, estas experiencias proporcionan una vista completa del estado de Azure con el nivel de detalle relevante.

**Vea una introducción de la página Estado de Azure, Azure Service Health y Azure Resource Health**

>[!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2OgX6]

[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]