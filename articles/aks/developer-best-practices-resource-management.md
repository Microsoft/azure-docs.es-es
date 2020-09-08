---
title: Procedimientos recomendados de administración de recursos
titleSuffix: Azure Kubernetes Service
description: Obtenga más información acerca de los procedimientos recomendados para desarrolladores de aplicaciones para la administración de recursos en Azure Kubernetes Services (AKS).
services: container-service
author: zr-msft
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: zarhoads
ms.openlocfilehash: 4882fadcc2f05e4047366d8d097a3918091035bb
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005319"
---
# <a name="best-practices-for-application-developers-to-manage-resources-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para desarrolladores de aplicaciones para administrar recursos en Azure Kubernetes Services (AKS)

A medida que desarrolla y ejecuta aplicaciones en Azure Kubernetes Service (AKS), hay algunas áreas clave a tener en cuenta. El modo de administrar las implementaciones de aplicaciones puede repercutir negativamente en la experiencia del usuario final de los servicios que proporciona. A fin de tener éxito, tenga en cuenta algunos procedimientos recomendados que puede seguir a medida que desarrolla y ejecuta aplicaciones en AKS.

Este artículo de procedimientos recomendados se centra en cómo ejecutar el clúster y las cargas de trabajo desde la perspectiva de un desarrollador de aplicaciones. Para obtener información acerca de los procedimientos recomendados, consulte [Procedimientos recomendados para el aislamiento y la administración de recursos en Azure Kubernetes Service (AKS)][operator-best-practices-isolation]. En este artículo, aprenderá lo siguiente:

> [!div class="checklist"]
> * Cuáles son los límites y solicitudes de recursos del pod
> * Formas de desarrollar e implementar aplicaciones con Dev Spaces y Visual Studio Code
> * Cómo usar la herramienta `kube-advisor` para comprobar si existen problemas con las implementaciones

## <a name="define-pod-resource-requests-and-limits"></a>Definición de los límites y solicitudes de recursos del pod

**Guía de procedimientos recomendados**: configure las solicitudes y los límites de pods en todos los pods de los manifiestos de YAML. Si el clúster de AKS usa *cuotas de recursos*, se puede rechazar la implementación si no define estos valores.

Una manera principal de administrar los recursos de proceso dentro de un clúster de AKS consiste en usar solicitudes y límites de pods. Estos límites y solicitudes permiten que el programador de Kubernetes sepa qué recursos de proceso deben asignarse a un pod.

* Las **solicitudes de CPU/memoria del pod** definen una cantidad establecida de CPU y memoria que el pod necesita regularmente.
    * Cuando el programador de Kubernetes intenta colocar un pod en un nodo, las solicitudes de pods se usan para determinar qué nodo tiene suficientes recursos disponibles para la programación.
    * Si no configura una solicitud de pod, el valor predeterminado será el límite definido.
    * Es muy importante supervisar el rendimiento de su aplicación para ajustar estas solicitudes. Si se realizan solicitudes insuficientes, su aplicación puede recibir un rendimiento degradado debido a la programación excesiva de un nodo. Si se sobreestiman las solicitudes, su aplicación puede tener mayores dificultades para ser programada.
* Los **límites de CPU/memoria del pod** equivalen a la cantidad máxima de CPU y memoria que puede usar un pod. Los límites de memoria ayudan a definir qué pods deben eliminarse en caso de inestabilidad de nodos debido a recursos insuficientes. Sin los límites adecuados, los pods establecidos se terminarán hasta que se levante la presión de los recursos. Un pod puede o no superar el límite de CPU durante un período de tiempo, pero el pod no se terminará cuando se supere el límite de CPU. 
    * Los límites de pod ayudan a definir cuándo ha perdido un pod el control del consumo de recursos. Cuando se supera un límite, el pod tiene prioridad de terminar para mantener el estado del nodo y minimizar el impacto en los pods que comparten el nodo.
    * Al no establecer un límite de pod, el valor predeterminado es el valor más alto disponible en un nodo determinado.
    * No establezca un límite de pods superior al que los nodos pueden admitir. Cada nodo de AKS reserva una cierta cantidad de CPU y memoria para los componentes básicos de Kubernetes. La aplicación puede intentar consumir demasiados recursos en el nodo para que otros pods se ejecuten correctamente.
    * Nuevamente, es muy importante supervisar el rendimiento de su aplicación en diferentes momentos durante el día o la semana. Determine cuándo se produce la máxima demanda y alinee los límites de pods para que los recursos necesarios satisfagan las necesidades máximas de la aplicación.

En las especificaciones de su pod, es un **procedimiento recomendado y muy importante**  definir estas solicitudes y límites en función de la información anterior. Si no incluye estos valores, el scheduler de Kubernetes no puede tener en cuenta los recursos que requieren sus aplicaciones para ayudarlo a tomar decisiones de programación.

Si el Scheduler coloca un Pod en un nodo con recursos insuficientes, el rendimiento de la aplicación se degradará. Se recomienda encarecidamente a los administradores del clúster establecer *cuotas de recursos* en un espacio de nombres que requiera que establezca solicitudes y límites de recursos. Para obtener más información, consulte [cuotas de recursos en clústeres de AKS][resource-quotas].

Al definir un límite o solicitud de CPU, el valor se mide en unidades de CPU. 
* CPU *1.0* equivale a un núcleo de CPU virtual subyacente en el nodo. 
* Se usa la misma unidad de medida para las GPU.
* Puede definir fracciones medidas en milinúcleos. Por ejemplo, *100 m* es *0,1* de un núcleo de vCPU subyacente.

En el siguiente ejemplo básico para un solo pod de NGINX, el pod solicita *100 m* de tiempo de CPU y *128 Mi* de memoria. Los límites de recursos para el pod se definen en *250 m* de CPU y *256 Mi* de memoria:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: nginx:1.15.5
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
```

Para obtener más información acerca de las asignaciones y medidas de recursos, consulte [Administración de recursos de proceso para contenedores][k8s-resource-limits].

## <a name="develop-and-debug-applications-against-an-aks-cluster"></a>Desarrollo y depuración de aplicaciones en un clúster de AKS

**Guía de procedimientos recomendados**: los equipos de desarrollo deben realizar la implementación y depuración en un clúster de AKS mediante Dev Spaces. Este modelo de desarrollo garantiza el cumplimiento de los requisitos de almacenamiento, red o control de acceso basado en rol antes de implementar la aplicación en el entorno de producción.

Con Azure Dev Spaces, desarrolle, depure y pruebe aplicaciones directamente en un clúster de AKS. Los desarrolladores de un equipo colaboran para realizar compilaciones y pruebas a lo largo del ciclo de vida de la aplicación. Puede seguir usando herramientas existentes, como Visual Studio o Visual Studio Code. Se instala una extensión para Dev Spaces que ofrece una opción para ejecutar y depurar la aplicación en un clúster de AKS.

Este proceso de desarrollo y pruebas integrado con Dev Spaces reduce la necesidad de entornos de prueba locales, como [minikube][minikube]. En su lugar, desarrolle y haga pruebas con un clúster de AKS. Este clúster se puede proteger y aislar como se indicó en la sección anterior sobre el uso de espacios de nombres para aislar un clúster de forma lógica. Cuando las aplicaciones estén listas para implementarse en producción, podrá implementarlas de forma segura, ya que el desarrollo se realizó totalmente en un clúster de AKS real.

Azure Dev Spaces está pensado para su uso con aplicaciones que se ejecutan en nodos y pods de Linux.

## <a name="use-the-visual-studio-code-extension-for-kubernetes"></a>Uso de la extensión de Visual Studio Code para Kubernetes

**Guía de procedimientos recomendados**: instale y use la extensión de VS Code para Kubernetes al escribir manifiestos de YAML. También puede usar la extensión para la solución de implementación integrada, lo que puede ayudar a los propietarios de aplicaciones que interactúan con poca frecuencia con el clúster de AKS.

La [extensión de Visual Studio Code para Kubernetes][vscode-kubernetes] le ayuda a desarrollar e implementar aplicaciones en AKS. La extensión proporciona IntelliSense para recursos de Kubernetes, así como para gráficos y plantillas de Helm. También puede explorar, implementar y modificar recursos de Kubernetes desde dentro de VS Code. La extensión también proporciona una comprobación de IntelliSense para los límites o solicitudes de recursos que se van a establecer en las especificaciones del pod:

![Advertencia de la extensión de VS Code para Kubernetes que indica que faltan los límites de memoria.](media/developer-best-practices-resource-management/vs-code-kubernetes-extension.png)

## <a name="regularly-check-for-application-issues-with-kube-advisor"></a>Comprobación de forma periódica de problemas de aplicaciones con kube-advisor

**Orientación con procedimientos recomendados**: Ejecute de forma periódica la versión más reciente de la herramienta de código abierto `kube-advisor` para detectar problemas en el clúster. Si aplica cuotas de recursos en un clúster de AKS existente, en primer lugar, ejecute `kube-advisor` para buscar los pods que no tienen definidos los límites y las solicitudes de recursos.

La herramienta [kube-advisor][kube-advisor] es un proyecto de código abierto de AKS asociado que explora un clúster de Kubernetes e informa sobre los problemas que encuentra. Una comprobación útil consiste en identificar los pods que no tienen preparados los límites y las solicitudes de recursos.

La herramienta kube-advisor puede informar sobre la solicitud de recursos y la falta de límites en PodSpecs para las aplicaciones Windows, así como las aplicaciones de Linux, pero la propia herramienta kube-advisor debe programarse en un pod de Linux. Puede programar un pod para que se ejecute en un grupo de nodos con un sistema operativo específico mediante un [selector de nodo][k8s-node-selector] en la configuración del pod.

En un clúster de AKS que hospeda muchos equipos y aplicaciones de desarrollo, puede ser difícil realizar un seguimiento de los pods sin definir estos límites y solicitudes de recursos. Como procedimiento recomendado, ejecute `kube-advisor` de forma periódica en los clústeres de AKS.

## <a name="next-steps"></a>Pasos siguientes

Este artículo de procedimientos recomendados se centra en cómo ejecutar el clúster y las cargas de trabajo desde la perspectiva de un operador de clústeres. Para obtener información acerca de los procedimientos recomendados, consulte [Procedimientos recomendados para el aislamiento y la administración de recursos en Azure Kubernetes Service (AKS)][operator-best-practices-isolation].

Para implementar algunos de estos procedimientos recomendados, consulte los artículos siguientes:

* [Desarrollo con Dev Spaces][dev-spaces]
* [Comprobación de problemas con kube-advisor][aks-kubeadvisor]

<!-- EXTERNAL LINKS -->
[k8s-resource-limits]: https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/
[vscode-kubernetes]: https://github.com/Azure/vscode-kubernetes-tools
[kube-advisor]: https://github.com/Azure/kube-advisor
[minikube]: https://kubernetes.io/docs/setup/minikube/

<!-- INTERNAL LINKS -->
[aks-kubeadvisor]: kube-advisor-tool.md
[dev-spaces]: /visualstudio/containers/overview-local-process-kubernetes
[operator-best-practices-isolation]: operator-best-practices-cluster-isolation.md
[resource-quotas]: operator-best-practices-scheduler.md#enforce-resource-quotas
[k8s-node-selector]: concepts-clusters-workloads.md#node-selectors
