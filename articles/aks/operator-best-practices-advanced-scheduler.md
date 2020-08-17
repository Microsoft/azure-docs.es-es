---
title: Procedimientos recomendados para las características del programador
titleSuffix: Azure Kubernetes Service
description: Conozca las prácticas recomendadas de operador de clúster para usar características avanzadas de programador como taints y tolerations, los selectores de nodo y la afinidad o falta de afinidad entre pods en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 11/26/2018
ms.openlocfilehash: b8077a772d6fdc4b911fabdfa893a15dcd7615db
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87530068"
---
# <a name="best-practices-for-advanced-scheduler-features-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para características avanzadas del programador en Azure Kubernetes Service (AKS)

A medida que administra los clústeres en Azure Kubernetes Service (AKS), a menudo necesita aislar los equipos y las cargas de trabajo. El programador de Kubernetes proporciona características avanzadas que permiten controlar qué pods se pueden programar en determinados nodos o cómo se distribuyen las aplicaciones de varios pods apropiadamente en el clúster. 

Este artículo de procedimientos recomendados se centra en las características avanzadas de Kubernetes para operadores de clúster. En este artículo aprenderá a:

> [!div class="checklist"]
> * Usar taints y tolerations para limitar los pods que pueden programarse en nodos
> * Dar prioridad a los pods para ejecutarse en determinados nodos con los selectores de nodo o la afinidad de nodo
> * Dividir o agrupar pods con afinidad o falta de afinidad entre pods

## <a name="provide-dedicated-nodes-using-taints-and-tolerations"></a>Proporcionar nodos dedicados con taints y tolerations

**Orientación con procedimientos recomendados**: limite el acceso a aplicaciones que consumen muchos recursos, como los controladores de entrada, a nodos específicos. Mantenga recursos de nodo disponibles para las cargas de trabajo que los necesite y no permita la programación de otras cargas de trabajo en los nodos.

Al crear el clúster de AKS, puede implementar los nodos con compatibilidad con GPU o un gran número de CPU eficaces. Estos nodos se utilizan a menudo para cargas de trabajo de procesamiento de datos grandes, como el aprendizaje automático o la inteligencia artificial. Como este tipo de hardware suele ser un recurso de nodo costoso para implementar, limite las cargas de trabajo que se pueden programar en esos nodos. En su lugar, puede dedicar algunos nodos del clúster a ejecutar servicios de entrada y evitar otras cargas de trabajo.

Esta compatibilidad para distintos nodos se proporciona mediante el uso de varios grupos de nodos. Un clúster de AKS proporciona uno o varios grupos de nodos.

El programador de Kubernetes puede utilizar taints y tolerations para limitar las cargas de trabajo que se pueden ejecutar en los nodos.

* Un valor **taint** se aplica a un nodo que indica que solo se pueden programar pods específicos en él.
* Luego se aplica un valor **toleration** a un pod que le permite *tolerar* el valor taint de un nodo.

Al implementar un pod en un clúster de AKS, Kubernetes programa solo pods en nodos donde un valor toleration se alinea con el valor taint. Por ejemplo, suponga que tiene un grupo de nodos en el clúster de AKS para nodos con compatibilidad con GPU. Define el nombre (por ejemplo, *gpu*) y, a continuación, un valor para la programación. Si establece este valor en *NoSchedule*, el programador de Kubernetes no puede programar pods en el nodo si el pod no define el valor toleration correspondiente.

```console
kubectl taint node aks-nodepool1 sku=gpu:NoSchedule
```

Una vez aplicado un valor taint a los nodos, puede definir un valor toleration en la especificación del pod que permite la programación en los nodos. En el ejemplo siguiente se define `sku: gpu` y `effect: NoSchedule` para tolerar el valor taint aplicado al nodo en el paso anterior:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: microsoft/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  tolerations:
  - key: "sku"
    operator: "Equal"
    value: "gpu"
    effect: "NoSchedule"
```

Cuando se implemente este pod, como el uso de `kubectl apply -f gpu-toleration.yaml`, Kubernetes puede programar correctamente el pod en los nodos donde se ha aplicado el valor taint. Este aislamiento lógico le permite controlar el acceso a los recursos dentro de un clúster.

Al aplicar valores taint, el trabajo con los desarrolladores y propietarios de su aplicación les permite definir las tolerancias necesarias en sus implementaciones.

Para obtener más información sobre cómo usar varios grupos de nodos en AKS, consulte [Creación y administración de varios grupos de nodos para un clúster en AKS][use-multiple-node-pools].

### <a name="behavior-of-taints-and-tolerations-in-aks"></a>Comportamiento de taints y tolerations en AKS

Al actualizar un grupo de nodos en AKS, taints y tolerations siguen un patrón de conjunto a medida que se aplican a los nuevos nodos:

- **Clústeres predeterminados que utilizan conjuntos de escalado de máquinas virtuales**
  - Puede [agregar intolerancias a un grupo de nodos][taint-node-pool] desde la API de AKS para que los nuevos nodos que se escalen horizontalmente reciban las intolerancias de nodo especificadas por la API.
  - Supongamos que tiene un clúster de dos nodos: *node1* y *node2*. Actualice el grupo de nodos.
  - Se crean dos nodos adicionales, *node3* y *node4*, y se pasan los valores taints respectivamente.
  - Las versiones originales de *node1* y *node2* se eliminan.

- **Clústeres sin compatibilidad con conjuntos de escalado de máquinas virtuales**
  - Supongamos de nuevo que tiene un clúster de dos nodos: *node1* y *node2*. Al actualizar, se crea un nodo adicional (*node3*).
  - Los valores taints de *node1* se aplican a *node3* y luego *node1* se elimina.
  - Se crea otro nuevo nodo (llamado *node1*, ya que el anterior *node1* se ha eliminado) y los valores taints de *node2* se aplican al nuevo *node1*. Luego, *node2* se elimina.
  - Básicamente, *node1* se convierte en *node3* y *node2* se convierte en *node1*.

Al escalar un grupo de nodos en AKS, los valores taints y tolerations no se transfieren mediante el diseño.

## <a name="control-pod-scheduling-using-node-selectors-and-affinity"></a>Control de la programación del pod mediante selectores de nodo y afinidad

**Orientación con procedimientos recomendados**: controle la programación de pods en nodos mediante selectores de nodo, la afinidad de nodo y la afinidad entre nodos. Esta configuración permite que el programador de Kubernetes aísle lógicamente las cargas de trabajo, como el hardware en el nodo.

Se usan valores taint y toleration para aislar lógicamente los recursos con un corte radical: si el pod no tolera el valor taint de un nodo, no se programa en el nodo. Otra posibilidad es usar los selectores de nodo. Etiqueta los nodos, como para indicar el almacenamiento SSD adjunto localmente o una gran cantidad de memoria, y luego define en la especificación del pod un selector de nodo. Kubernetes, a continuación, programa los pods en un nodo coincidente. A diferencia de las tolerancias, los pods sin un selector de nodo coincidente pueden programarse en nodos etiquetados. Este comportamiento permite que los recursos no utilizados en los nodos se consuman, pero da prioridad a los pods que definen el selector de nodo correspondiente.

Veamos un ejemplo de nodos con una gran cantidad de memoria. Estos nodos pueden dar preferencia a los pods que solicitan una gran cantidad de memoria. Para asegurarse de que los recursos no estén inactivos, también permiten que se ejecuten otros pods.

```console
kubectl label node aks-nodepool1 hardware=highmem
```

Una especificación de pod, a continuación, agrega la propiedad `nodeSelector` para definir un selector de nodo que coincide con la etiqueta configurada en un nodo:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: microsoft/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  nodeSelector:
      hardware: highmem
```

Cuando utilice estas opciones del programador, trabaje con los desarrolladores y propietarios de las aplicaciones para permitirles definir correctamente sus especificaciones de pods.

Para obtener más información sobre el uso de selectores de nodo, vea [Assigning Pods to Nodes][k8s-node-selector] (Asignación de pods a nodos).

### <a name="node-affinity"></a>Afinidad de nodos

Un selector de nodo es una forma básica de asignar pods a un nodo determinado. Hay más flexibilidad disponible al utilizar la *afinidad de nodo*. Con la afinidad de nodo, se define lo que sucede si no se puede asociar el pod con un nodo. Puede *exigir* que el programador de Kubernetes coincida con un pod con un host etiquetado. O bien, puede *preferir* una coincidencia, pero permitir que el pod se programe en un host diferente si no hay coincidencias disponibles.

En el ejemplo siguiente se establece la afinidad de nodo en *requiredDuringSchedulingIgnoredDuringExecution*. Esta afinidad requiere que la programación de Kubernetes utilice un nodo con una etiqueta coincidente. Si ningún nodo está disponible, el pod tiene que esperar una programación para continuar. Para permitir que el pod se programe en un nodo diferente, puede establecer el valor en *preferredDuringSchedulingIgnoreDuringExecution*:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: tf-mnist
spec:
  containers:
  - name: tf-mnist
    image: microsoft/samples-tf-mnist-demo:gpu
    resources:
      requests:
        cpu: 0.5
        memory: 2Gi
      limits:
        cpu: 4.0
        memory: 16Gi
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: hardware
            operator: In
            values: highmem
```

La parte *IgnoredDuringExecution* de la configuración indica que, si cambian las etiquetas del nodo, el pod no debe expulsarse del nodo. El programador de Kubernetes usa solo las etiquetas de nodo actualizadas para los nuevos pods que se programan, no para los pods que ya están programados en los nodos.

Para obtener más información, consulte [Affinity and anti-affinity][k8s-affinity] (Afinidad y falta de afinidad).

### <a name="inter-pod-affinity-and-anti-affinity"></a>Afinidad y falta de afinidad entre pods

Un último enfoque para que el programador de Kubernetes aísle lógicamente las cargas de trabajo es usar la afinidad o la falta de afinidad entre pods. La configuración define que los pods *no deben* estar programados en un nodo que tenga un pod coincidente existente, o que *deben* estar programados. De forma predeterminada, el programador de Kubernetes intenta programar varios pods en un conjunto de réplica entre nodos. Puede definir reglas más específicas sobre este comportamiento.

Un buen ejemplo es una aplicación web que también usa una instancia de Azure Cache for Redis. Puede usar reglas de falta de afinidad de pods para solicitar que el programador de Kubernetes distribuya réplicas entre nodos. Después, puede usar las reglas de afinidad para asegurarse de que cada componente de aplicación web se programe en el mismo host como una caché correspondiente. La distribución de pods en todos los nodos es similar al ejemplo siguiente:

| **Nodo 1** | **Nodo 2** | **Nodo 3** |
|------------|------------|------------|
| aplicación web-1   | aplicación web-2   | aplicación web-3   |
| caché-1    | caché-2    | caché-3    |

Este ejemplo es una implementación más compleja que el uso de selectores de nodo o afinidad de nodo. La implementación le permite controlar cómo programa Kubernetes los pods en los nodos y puede aislar los recursos de forma lógica. Para obtener un ejemplo completo de esta aplicación web con el ejemplo de la instancia de Azure Redis Cache, consulte [Colocación de pods en el mismo nodo][k8s-pod-affinity].

## <a name="next-steps"></a>Pasos siguientes

Este artículo se ha centrado en las características avanzadas del programador de Kubernetes. Para obtener más información acerca de las operaciones de clúster en AKS, consulte los siguientes procedimientos recomendados:

* [Aislamiento de clúster y de multiinquilinato][aks-best-practices-scheduler]
* [Características básicas del programador de Kubernetes][aks-best-practices-scheduler]
* [Autenticación y autorización][aks-best-practices-identity]

<!-- EXTERNAL LINKS -->
[k8s-taints-tolerations]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[k8s-node-selector]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[k8s-affinity]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
[k8s-pod-affinity]: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#always-co-located-in-the-same-node

<!-- INTERNAL LINKS -->
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-cluster-isolation]: operator-best-practices-cluster-isolation.md
[aks-best-practices-identity]: operator-best-practices-identity.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[taint-node-pool]: use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool
