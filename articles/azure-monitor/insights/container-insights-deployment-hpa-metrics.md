---
title: Implementación y métricas de HPA con Azure Monitor para contenedores | Microsoft Docs
description: En este artículo se describe qué métricas de implementación y HPA (Horizontal Pod Autoscaler) se recopilan con Azure Monitor para contenedores.
ms.topic: conceptual
ms.date: 08/09/2020
ms.openlocfilehash: ee1f0d4849a8382a898aaca84956ff78166e138f
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89570413"
---
# <a name="deployment--hpa-metrics-with-azure-monitor-for-containers"></a>Implementación y métricas de HPA con Azure Monitor para contenedores

A partir de la versión de agente *ciprod08072020*, el agente integrado de Azure Monitor para contenedores ahora recopila métricas para las implementaciones y las instancias de HPA.

## <a name="deployment-metrics"></a>Métricas de implementación

Azure Monitor para contenedores inicia automáticamente la supervisión de las implementaciones mediante la recopilación de las métricas siguientes a intervalos de 60 s y su almacenamiento en la tabla **InsightMetrics**:

|Nombre de métrica |Dimensión de métrica (etiquetas) |Descripción |
|------------|------------------------|------------|
|kube_deployment_status_replicas_ready |container.azm.ms/clusterId, container.azm.ms/clusterName, creationTime, deployment, deploymentStrategy, k8sNamespace, spec_replicas, status_replicas_available, status_replicas_updated (status.updatedReplicas) | Número total de pods preparados que esta implementación tiene como destino (status.readyReplicas). A continuación se muestran las dimensiones de esta métrica. <ul> <li> deployment: nombre de la implementación. </li> <li> k8sNamespace: espacio de nombres de Kubernetes para la implementación. </li> <li> deploymentStrategy: estrategia de implementación que se usa para reemplazar los pods por otros nuevos (spec.strategy.type).</li><li> creationTime: marca de tiempo de creación de la implementación. </li> <li> spec_replicas: número de pods deseados (spec.replicas). </li> <li>status_replicas_available: número total de pods disponibles (preparados durante al menos minReadySeconds) que esta implementación tiene como destino (status.availableReplicas).</li><li>status_replicas_updated: número total de pods no terminados que esta implementación tiene como destino y que tienen la especificación de plantilla deseada (status.updatedReplicas). </li></ul>|

## <a name="hpa-metrics"></a>Métricas de HPA

Azure Monitor para contenedores inicia automáticamente la supervisión de las instancias de HPA mediante la recopilación de las métricas siguientes a intervalos de 60 s y su almacenamiento en la tabla **InsightMetrics**:

|Nombre de métrica |Dimensión de métrica (etiquetas) |Descripción |
|------------|------------------------|------------|
|kube_hpa_status_current_replicas |container.azm.ms/clusterId, container.azm.ms/clusterName, creationTime, hpa, k8sNamespace, lastScaleTime, spec_max_replicas, spec_min_replicas, status_desired_replicas, targetKind, targetName | Número actual de réplicas de pods administradas por este escalador automático (status.currentReplicas). A continuación se muestran las dimensiones de esta métrica. <ul> <li> hpa: nombre de la instancia de HPA. </li> <li> k8sNamespace: espacio de nombres de Kubernetes para la instancia de HPA. </li> <li> lastScaleTime: última vez que la instancia de HPA escaló el número de pods (status.lastScaleTime).</li><li> creationTime: marca de tiempo de creación de la instancia de HPA. </li> <li> spec_max_replicas: límite superior del número de pods que puede establecer el escalador automático (spec.maxReplicas). </li> <li> spec_min_replicas: límite inferior del número de réplicas al que se puede reducir verticalmente el escalador automático (spec.minReplicas). </li><li>status_desired_replicas: número deseado de réplicas de pods administradas por este escalador automático (status.desiredReplicas).</li><li>targetKind: tipo del destino de la instancia de HPA (spec.scaleTargetRef.kind). </li><li>targetName: nombre del destino de la instancia de HPA (spec.scaleTargetRef.name). </li></ul>|

## <a name="deployment--hpa-charts"></a>Gráficos de implementación y HPA 

Azure Monitor para contenedores incluye gráficos preconfigurados para las métricas indicadas anteriormente en la tabla como libro para cada clúster. Puede encontrar el libro de implementaciones y HPA **Implementaciones y HPA** directamente de un clúster de AKS. Para ello, seleccione **Libros** en el panel izquierdo y después selecciónelo en la lista desplegable **Ver libros** de Conclusión.

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Métricas de estado de Kube en Kubernetes](https://github.com/kubernetes/kube-state-metrics/tree/master/docs) para obtener más información sobre las métricas de estado de Kube.