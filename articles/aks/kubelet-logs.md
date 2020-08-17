---
title: Visualización de registros de kubelet desde Azure Kubernetes Service (AKS)
description: Aprenda a visualizar la información de solución de problemas en los registros de kubelet desde los nodos de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/05/2019
ms.openlocfilehash: 2ef38aa76f4ef9152e7bc55a1d74c84ef426f0ac
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87056751"
---
# <a name="get-kubelet-logs-from-azure-kubernetes-service-aks-cluster-nodes"></a>Obtención de registros de kubelet desde nodos de clúster de Azure Kubernetes Service (AKS)

Como parte del funcionamiento de un clúster de AKS, puede que deba revisar los registros para solucionar un problema. Azure Portal incorpora la posibilidad de ver los registros de los [componentes maestros de AKS][aks-master-logs] o de los [contenedores de un clúster AKS][azure-container-logs]. En ocasiones, necesitará obtener registros de *kubelet* de un nodo de AKS para solucionar problemas.

En este artículo se muestra cómo puede usar `journalctl` para ver los registros de *kubelet* en un nodo de AKS.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

## <a name="create-an-ssh-connection"></a>Creación de una conexión SSH

En primer lugar, cree una conexión SSH con el nodo en el que necesita ver los registros de *kubelet*. Esta operación se describe en el documento [Conexión SSH con los nodos de un clúster de Azure Kubernetes Service (AKS)][aks-ssh].

## <a name="get-kubelet-logs"></a>Obtención de registros de kubelet

Una vez que se ha conectado al nodo, ejecute el siguiente comando para extraer los registros de *kubelet*:

```console
sudo journalctl -u kubelet -o cat
```

> [!NOTE]
> En el caso de los nodos de Windows, los datos de registro están en `C:\k` y se pueden ver con el comando *more*:
> ```
> more C:\k\kubelet.log
> ```

La siguiente salida de ejemplo muestra los datos del registro de *kubelet*:

```
I0508 12:26:17.905042    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:26:27.943494    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:26:28.920125    8672 server.go:796] GET /stats/summary: (10.370874ms) 200 [[Ruby] 10.244.0.2:52292]
I0508 12:26:37.964650    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:26:47.996449    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:26:58.019746    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:05.107680    8672 server.go:796] GET /stats/summary/: (24.853838ms) 200 [[Go-http-client/1.1] 10.244.0.3:44660]
I0508 12:27:08.041736    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:18.068505    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:28.094889    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:38.121346    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:44.015205    8672 server.go:796] GET /stats/summary: (30.236824ms) 200 [[Ruby] 10.244.0.2:52588]
I0508 12:27:48.145640    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:27:58.178534    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:05.040375    8672 server.go:796] GET /stats/summary/: (27.78503ms) 200 [[Go-http-client/1.1] 10.244.0.3:44660]
I0508 12:28:08.214158    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:18.242160    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:28.274408    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:38.296074    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:48.321952    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
I0508 12:28:58.344656    8672 kubelet_node_status.go:497] Using Node Hostname from cloudprovider: "aks-agentpool-11482510-0"
```

## <a name="next-steps"></a>Pasos siguientes

Si necesita información adicional desde el maestro de Kubernetes, consulte [Habilitación y revisión de los registros del nodo maestro de Kubernetes en Azure Kubernetes Service (AKS)][aks-master-logs].

<!-- LINKS - internal -->
[aks-ssh]: ssh.md
[aks-master-logs]: view-master-logs.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[aks-master-logs]: view-master-logs.md
[azure-container-logs]: ../azure-monitor/insights/container-insights-overview.md
