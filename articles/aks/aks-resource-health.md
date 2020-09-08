---
title: Comprobación de eventos de Resource Health que afectan al clúster de AKS (versión preliminar)
description: Compruebe el estado de mantenimiento del clúster de AKS mediante Azure Resource Health.
services: container-service
author: yunjchoi
ms.topic: troubleshooting
ms.date: 08/18/2020
ms.author: yunjchoi
ms.openlocfilehash: d14ed7e714eac175dc7a17bbccd2150f9bb01ef9
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88752699"
---
# <a name="check-for-resource-health-events-impacting-your-aks-cluster-preview"></a>Comprobación de eventos de Resource Health que afectan al clúster de AKS (versión preliminar)


Al ejecutar las cargas de trabajo de contenedor en AKS, quiere asegurarse de que puede solucionar y corregir problemas en cuanto surjan para minimizar su impacto en la disponibilidad de las cargas de trabajo. [Azure Resource Health](https://docs.microsoft.com/azure/service-health/resource-health-overview) le ofrece visibilidad sobre diversos eventos de mantenimiento que pueden hacer que el clúster de AKS deje de estar disponible.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="open-resource-health"></a>Apertura de Resource Health

### <a name="to-access-resource-health-for-your-aks-cluster"></a>Para acceder a Resource Health para el clúster de AKS:

- Vaya al clúster de AKS en [Azure Portal](https://portal.azure.com).
- Seleccione **Resource Health** en el panel de navegación izquierdo.

### <a name="to-access-resource-health-for-all-clusters-on-your-subscription"></a>Para tener acceso a Resource Health en todos los clústeres de su suscripción:

- Busque **Service Health** en [Azure Portal](https://portal.azure.com) y navegue hasta dicha opción.
- Seleccione **Resource Health** en el panel de navegación izquierdo.
- Seleccione su suscripción y establezca el tipo de recurso en Azure Kubernetes Service (AKS).

![comprobación-del-estado-de-recursos](./media/aks-resource-health/resource-health-check.png)

## <a name="check-the-health-status"></a>Comprobación del estado de mantenimiento

Azure Resource Health ayuda a diagnosticar problemas en los servicios que afectan a los recursos de Azure y a obtener soporte técnico para resolverlos. Resource Health informa sobre el mantenimiento actual y anterior de los recursos y le ayuda a determinar si el problema se debe a una acción iniciada por el usuario o a un evento de la plataforma.

Resource Health recibe señales del clúster administrado con el fin de determinar el estado de mantenimiento del mismo. Examina el estado de mantenimiento del clúster de AKS e informa de las acciones necesarias para cada señal de mantenimiento. Estas señales van desde problemas de resolución automática, actualizaciones planeadas, eventos de mantenimiento no planeados y no disponibilidad generados por acciones iniciadas por el usuario. Estas señales se clasifican mediante el estado de mantenimiento de Azure Resource Health: *Disponible*, *No disponible*, *Desconocido* y *Degradado*.

- **Disponible**: cuando no hay ningún problema conocido que afecte al mantenimiento del clúster, Resource Health informará que el clúster está *Disponible*.

- **No disponible**: cuando hay un evento de la plataforma o ajeno a esta que afecta al mantenimiento del clúster, Resource Health informa que el clúster está *No disponible*.

- **Desconocido**: cuando se produce una pérdida temporal de conexión a las métricas de mantenimiento del clúster, Resource Health informa que el clúster tiene un estado *Desconocido*.

- **Degradado**: cuando hay un problema de mantenimiento que requiere una acción del usuario, Resource Health informa que el clúster está *Degradado*.

Para obtener información adicional sobre lo que indica cada estado de mantenimiento, visite [Introducción a Resource Health](https://docs.microsoft.com/azure/service-health/resource-health-overview#health-status).

### <a name="view-historical-data"></a>Visualización de datos históricos

También puede ver los últimos 30 días de información histórica de Resource Health en la sección **Historial de estado**.

## <a name="next-steps"></a>Pasos siguientes

Ejecute comprobaciones en el clúster para solucionar más problemas del clúster mediante los [diagnósticos de AKS](https://docs.microsoft.com/azure/aks/concepts-diagnostics).
