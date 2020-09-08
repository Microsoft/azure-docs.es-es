---
title: Procedimientos recomendados para el aislamiento de clústeres
titleSuffix: Azure Kubernetes Service
description: Obtenga más información acerca de los procedimientos recomendados del operador de clústeres para el aislamiento en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: conceptual
ms.date: 11/26/2018
ms.openlocfilehash: cdeecabf569e3c6f9b280e6b0179e5378f5b1c95
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88003105"
---
# <a name="best-practices-for-cluster-isolation-in-azure-kubernetes-service-aks"></a>Procedimientos recomendados para el aislamiento de clústeres en Azure Kubernetes Service (AKS)

A medida que administra los clústeres en Azure Kubernetes Service (AKS), a menudo necesita aislar los equipos y las cargas de trabajo. AKS proporciona flexibilidad acerca de cómo puede ejecutar clústeres multiempresa y aislar recursos. Para maximizar la inversión en Kubernetes, deben comprenderse e implementarse estas funciones multiempresa y de aislamiento.

Este artículo de procedimientos recomendados se centra en el aislamiento de los operadores de clústeres. En este artículo aprenderá a:

> [!div class="checklist"]
> * Planificar clústeres multiempresa y separación de recursos.
> * Usar el aislamiento lógico o físico de los clústeres de AKS.

## <a name="design-clusters-for-multi-tenancy"></a>Diseño de clústeres para la configuración multiempresa

Kubernetes proporciona características que permiten aislar de forma lógica los equipos y las cargas de trabajo en el mismo clúster. El objetivo debería consistir en proporcionar el menor número de privilegios, con el ámbito definido según los recursos que necesita cada equipo. Un [espacio de nombres][k8s-namespaces] en Kubernetes crea un límite de aislamiento lógico. A continuación, se incluyen otras características y consideraciones de Kubernetes sobre el aislamiento y la arquitectura multiinquilino:

* La **programación** incluye el uso de características básicas, como las cuotas de recursos y los presupuestos de interrupciones del pod. Para más información acerca de estas características, consulte [Procedimientos recomendados para características básicas del programador en Azure Kubernetes Service (AKS)][aks-best-practices-scheduler].
  * Las características más avanzadas del programador incluyen manchas y tolerancias, selectores de nodo y afinidad o falta de afinidad del nodo y el pod. Para más información acerca de estas características, consulte [Procedimientos recomendados para características avanzadas del programador en Azure Kubernetes Service (AKS)][aks-best-practices-advanced-scheduler].
* Las **redes** incluyen el uso de directivas de red para controlar el flujo de tráfico de entrada y salida de pods.
* La **autenticación y autorización** incluyen el uso del control de acceso basado en rol (RBAC) y la integración de Azure Active Directory (AD), las identidades de pods y los secretos de Azure Key Vault. Para más información sobre estas características, consulte [Procedimientos recomendados para la autenticación y autorización en Azure Kubernetes Service (AKS)][aks-best-practices-identity].
* **Containers** incluye el complemento de Azure Policy para AKS para exigir la seguridad de los pods, el uso de contextos de seguridad de pods y el análisis de las imágenes y el entorno de ejecución en busca de vulnerabilidades. Esto también implica el uso de AppArmor o Seccomp (informática segura) para restringir el acceso del contenedor al nodo subyacente.

## <a name="logically-isolate-clusters"></a>Aislamiento de clústeres de forma lógica

**Guía de procedimientos recomendados**: use el aislamiento lógico para separar los equipos y los proyectos. Intente minimizar el número de clústeres físicos de AKS que implementa para aislar los equipos o las aplicaciones.

Con el aislamiento lógico, un único clúster de AKS puede utilizarse para varios entornos, equipos o cargas de trabajo. Los [espacios de nombres][k8s-namespaces] de Kubernetes forman el límite de aislamiento lógico para las cargas de trabajo y los recursos.

![Aislamiento lógico de un clúster de Kubernetes en AKS](media/operator-best-practices-cluster-isolation/logical-isolation.png)

Normalmente, una separación lógica de los clústeres proporciona una mayor densidad de pods que los clústeres aislados físicamente. Hay menos exceso de capacidad del proceso que se encuentra inactiva en el clúster. Cuando se combina con el escalador automático de clúster de Kubernetes, puede escalar vertical u horizontalmente el número de nodos para satisfacer las necesidades. Este enfoque de procedimiento recomendado para el escalado automático le permite ejecutar solo el número de nodos necesario y minimiza los costos.

Los entornos de Kubernetes, tanto en AKS como en cualquier otro lugar, no están completamente seguros ante el uso de varios inquilinos hostiles. En un entorno multiinquilino, varios inquilinos trabajan en una infraestructura compartida común. Como resultado, si no se puede confiar en todos los inquilinos, debe realizar una planeación adicional para evitar que un inquilino afecte a la seguridad y el servicio de otro. El uso de características de seguridad adicionales, como la *directiva de seguridad de pod* y el control de acceso basado en rol (RBAC), más pormenorizado, en los nodos, hace que sea más difícil aprovechar las vulnerabilidades. Sin embargo, para que la seguridad resulte efectiva cuando se ejecutan cargas de trabajo multiinquilino hostiles, el hipervisor es el único nivel de seguridad en el que debe confiar. El dominio de seguridad de Kubernetes se convierte en todo el clúster, no en un nodo específico. En el caso de estos tipos de cargas de trabajo multiinquilino hostiles, debe usar clústeres que estén físicamente aislados.

## <a name="physically-isolate-clusters"></a>Aislamiento de clústeres de forma física

**Guía de procedimientos recomendados**: minimiza el uso del aislamiento físico para cada equipo independiente o una implementación de la aplicación. En su lugar, use el aislamiento *lógico*, como se describe en la sección anterior.

Un enfoque común para el aislamiento de clústeres consiste en usar clústeres de AKS físicamente independientes. En este modelo de aislamiento, los equipos o las cargas de trabajo se asignan a su propio clúster de AKS. A menudo, este enfoque es similar a la manera más fácil de aislar las cargas de trabajo o los equipos, pero agrega sobrecarga financiera y administración adicional. Ahora tiene que mantener estos clústeres, así como proporcionar acceso y asignar permisos de forma individual. También se le cobrará por todos los nodos individuales.

![Aislamiento físico de clústeres individuales de Kubernetes en AKS](media/operator-best-practices-cluster-isolation/physical-isolation.png)

Los clústeres físicamente independientes suelen tener una densidad de pods baja. Dado que cada equipo o carga de trabajo tiene su propio clúster de AKS, el clúster a menudo se aprovisiona en exceso con recursos de proceso. Con frecuencia, se programa un pequeño número de pods en esos nodos. La capacidad no utilizada en los nodos no se puede usar para aplicaciones o servicios que están desarrollando otros equipos. Estos recursos en exceso contribuyen a los costos adicionales en los clústeres físicamente independientes.

## <a name="next-steps"></a>Pasos siguientes

Este artículo se centra en el aislamiento de clústeres. Para obtener más información acerca de las operaciones de clúster en AKS, consulte los siguientes procedimientos recomendados:

* [Características básicas del programador de Kubernetes][aks-best-practices-scheduler]
* [Características avanzadas del programador de Kubernetes][aks-best-practices-advanced-scheduler]
* [Autenticación y autorización][aks-best-practices-identity]

<!-- EXTERNAL LINKS -->

<!-- INTERNAL LINKS -->
[k8s-namespaces]: concepts-clusters-workloads.md#namespaces
[aks-best-practices-scheduler]: operator-best-practices-scheduler.md
[aks-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[aks-best-practices-identity]: operator-best-practices-identity.md
