---
title: Control de los reinicios de nodos de Linux con kured
titleSuffix: Azure Kubernetes Service
description: Aprenda a actualizar los nodos de Linux y a reiniciarlos automáticamente con Kured en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 02/28/2019
ms.openlocfilehash: 35c9e76c234e4b09fbb090eda363506ee3e11130
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88164247"
---
# <a name="apply-security-and-kernel-updates-to-linux-nodes-in-azure-kubernetes-service-aks"></a>Aplicación de actualizaciones de kernel y seguridad a los nodos de Linux en Azure Kubernetes Service (AKS)

Para proteger los clústeres, las actualizaciones de seguridad se aplican automáticamente a los nodos de Linux en AKS. Estas actualizaciones incluyen las revisiones de seguridad del sistema operativo o las actualizaciones del kernel. Algunas de estas actualizaciones requieren un reinicio del nodo para completar el proceso. AKS no reinicia automáticamente estos nodos de Linux para completar el proceso de actualización.

El proceso para mantener actualizados los nodos de Windows Server difiere ligeramente. Los nodos de Windows Server no reciben actualizaciones diarias. En su lugar, debe actualizar AKS para implementar nuevos nodos con la imagen base de Windows Server más reciente y las revisiones. Para los clústeres de AKS que usan los nodos de Windows Server, consulte [Actualización de un grupo de nodos de AKS][nodepool-upgrade].

En este artículo se muestra cómo usar el código abierto de [Kured (demonio de reinicio de Kubernetes)][kured] para buscar nodos de Linux que requieran un reinicio y, a continuación, administrar automáticamente la reprogramación de pods en ejecución y el proceso de reinicio de los nodos.

> [!NOTE]
> `Kured` es un proyecto de código abierto de Weaveworks. La asistencia para este proyecto en AKS se proporciona dentro de lo posible. Se puede encontrar soporte técnico adicional en el canal de Slack #weave-community.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

También es preciso que esté instalada y configurada la versión 2.0.59 de la CLI de Azure u otra versión posterior. Ejecute  `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte  [Install Azure CLI][install-azure-cli] (Instalación de la CLI de Azure).

## <a name="understand-the-aks-node-update-experience"></a>Información sobre la experiencia de actualización del nodo AKS

En un clúster de AKS, los nodos de Kubernetes se ejecutan como máquinas virtuales de Azure. Estas máquinas virtuales basadas en Linux, usan una imagen de Ubuntu, con el sistema operativo configurado para buscar automáticamente actualizaciones cada noche. Si hay disponibles actualizaciones de kernel o de seguridad, estas se descargan e instalan automáticamente.

![Actualización de nodo AKS y proceso de reinicio con Kured](media/node-updates-kured/node-reboot-process.png)

Algunas actualizaciones de seguridad, como las actualizaciones del kernel, requieren un reinicio del nodo para finalizar el proceso. Un nodo de Linux que requiere reinicio crea un archivo denominado */var/run/reboot-required*. Este proceso de reinicio no se realiza automáticamente.

Puede usar sus propios flujos de trabajo y procesos para controlar los reinicios de nodo, o usar `kured` para coordinar el proceso. Con `kured`, se implementa un valor de [DaemonSet][DaemonSet] que ejecuta un pod en cada nodo de Linux del clúster. Estos pods en DaemonSet comprueban si existe el archivo */var/run/reboot-required* y, a continuación, inician un proceso para reiniciar los nodos.

### <a name="node-upgrades"></a>Actualizaciones de nodo

Hay un proceso adicional en AKS que le permite *actualizar* un clúster. Una actualización consiste generalmente en el cambio a una versión más reciente de Kubernetes, no solo en aplicar las actualizaciones de seguridad de nodo. Una actualización de AKS realiza las acciones siguientes:

* Se implementa un nuevo nodo con las últimas actualizaciones de seguridad y se aplica la versión de Kubernetes.
* Se acordona y vacía un nodo anterior.
* Los pods se programan en el nuevo nodo.
* El nodo anterior se elimina.

No puede permanecer en la misma versión de Kubernetes durante un evento de actualización. Tiene que especificar una versión más reciente de Kubernetes. Para actualizar a la versión más reciente de Kubernetes, puede [actualizar el clúster de AKS][aks-upgrade].

## <a name="deploy-kured-in-an-aks-cluster"></a>Implementación de Kured en un clúster de AKS

Para implementar el valor `kured` de DaemonSet, instale el siguiente gráfico oficial de Kured Helm. Esta acción crea un rol y un rol de clúster, enlaces y una cuenta de servicio y, luego, implementa DaemonSet con `kured`.

```console
# Add the Kured Helm repository
helm repo add kured https://weaveworks.github.io/kured

# Update your local Helm chart repository cache
helm repo update

# Create a dedicated namespace where you would like to deploy kured into
kubectl create namespace kured

# Install kured in that namespace with Helm 3 (only on Linux nodes, kured is not working on Windows nodes)
helm install kured kured/kured --namespace kured --set nodeSelector."beta\.kubernetes\.io/os"=linux
```

También puede configurar parámetros adicionales para `kured`, como la integración con Prometheus o Slack. Para más información sobre los parámetros de configuración adicionales, consulte el [gráfico de Kured Helm][kured-install].

## <a name="update-cluster-nodes"></a>Actualización de nodos de clúster

De forma predeterminada, los nodos de Linux en AKS buscan actualizaciones por la noche. Si no desea esperar, puede realizar manualmente una actualización para comprobar que `kured` se ejecuta correctamente. En primer lugar, siga los pasos para [establecer conexión mediante SSH con uno de los nodos de AKS][aks-ssh]. Una vez que tenga una conexión SSH al nodo de Linux, compruebe si hay actualizaciones y aplíquelas como sigue:

```console
sudo apt-get update && sudo apt-get upgrade -y
```

Si se aplicaron actualizaciones que requieren un reinicio del nodo, se escribe un archivo en */var/run/reboot-required*. `Kured` comprueba si hay nodos que requieran un reinicio cada 60 minutos de forma predeterminada.

## <a name="monitor-and-review-reboot-process"></a>Supervisión y revisión del proceso de reinicio

Cuando una de las réplicas en el DaemonSet ha detectado que se requiere el reinicio de un nodo, se coloca un bloqueo en el nodo mediante la API de Kubernetes. Este bloqueo impide que se programen pods adicionales en el nodo. El bloqueo también indica que solo un nodo debe reiniciarse cada vez. Con el nodo acordonado, los pods en ejecución se purgan desde el nodo y este se reinicia.

Puede supervisar el estado de los nodos mediante el comando [kubectl get nodes][kubectl-get-nodes]. La salida del ejemplo siguiente muestra un nodo con el estado *SchedulingDisabled* mientras el nodo se prepara para el proceso de reinicio:

```
NAME                       STATUS                     ROLES     AGE       VERSION
aks-nodepool1-28993262-0   Ready,SchedulingDisabled   agent     1h        v1.11.7
```

Una vez completado el proceso de actualización, puede ver el estado de los nodos mediante el comando [kubectl get nodes][kubectl-get-nodes] con el parámetro `--output wide`. Esta salida adicional le permite ver la diferencia en *KERNEL-VERSION* de los nodos subyacentes, como se muestra en la siguiente salida de ejemplo. El nodo *aks-nodepool1-28993262-0* se actualizó en un paso anterior y muestra la versión de kernel *4.15.0-1039-azure*. El nodo *aks-nodepool1-28993262-1* que no ha sido actualizado muestra la versión de kernel *4.15.0-1037-azure*.

```
NAME                       STATUS    ROLES     AGE       VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION      CONTAINER-RUNTIME
aks-nodepool1-28993262-0   Ready     agent     1h        v1.11.7   10.240.0.4    <none>        Ubuntu 16.04.6 LTS   4.15.0-1039-azure   docker://3.0.4
aks-nodepool1-28993262-1   Ready     agent     1h        v1.11.7   10.240.0.5    <none>        Ubuntu 16.04.6 LTS   4.15.0-1037-azure   docker://3.0.4
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha detallado cómo utilizar `kured` para reiniciar nodos de Linux automáticamente como parte del proceso de actualización de seguridad. Para actualizar a la versión más reciente de Kubernetes, puede [actualizar el clúster de AKS][aks-upgrade].

Para los clústeres de AKS que usan los nodos de Windows Server, consulte [Actualización de un grupo de nodos de AKS][nodepool-upgrade].

<!-- LINKS - external -->
[kured]: https://github.com/weaveworks/kured
[kured-install]: https://github.com/weaveworks/kured/tree/master/charts/kured
[kubectl-get-nodes]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[DaemonSet]: concepts-clusters-workloads.md#statefulsets-and-daemonsets
[aks-ssh]: ssh.md
[aks-upgrade]: upgrade-cluster.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
