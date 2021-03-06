---
title: Procedimientos recomendados para Azure Kubernetes Service (AKS)
description: Colección de procedimientos recomendados de operadores de clúster y desarrolladores para crear y administrar aplicaciones en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 03/09/2021
ms.openlocfilehash: f4bd109ce6827de26b1649380b6f6a2ba7c10e17
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107105908"
---
# <a name="cluster-operator-and-developer-best-practices-to-build-and-manage-applications-on-azure-kubernetes-service-aks"></a>Procedimientos recomendados de operadores de clúster y desarrolladores para crear y administrar aplicaciones en Azure Kubernetes Service (AKS)

La creación y ejecución correctas de aplicaciones en Azure Kubernetes Service (AKS) requieren la comprensión y la implementación de algunas consideraciones clave, entre las que se incluyen:
* Características multiinquilino y programador
* Seguridad de clúster y pod
* Continuidad empresarial y recuperación ante desastres 


El grupo de productos de AKS, los equipos de ingeniería y los equipos de campo (incluidos cinturones negros globales [GBB]) han escrito y agrupado los procedimientos recomendados siguientes y los artículos conceptuales, así como han contribuido a ellos. Su propósito es ayudar a los operadores y desarrolladores de clústeres a comprender las consideraciones anteriores e implementar las características adecuadas.


## <a name="cluster-operator-best-practices"></a>Procedimientos recomendados para operadores de clúster

Como operador de clúster, trabaje en conjunto con los propietarios y desarrolladores de aplicaciones para conocer sus necesidades. A continuación, puede usar los siguientes procedimientos recomendados para configurar los clústeres de AKS según sea necesario.

**Servicios multiinquilino**

* [Procedimientos recomendados para el aislamiento de clústeres](operator-best-practices-cluster-isolation.md)
    * Incluye los componentes principales de los servicios multiinquilino y el aislamiento lógico con espacios de nombres.
* [Procedimientos recomendados para las características básicas del programador](operator-best-practices-scheduler.md)
    * Incluye el uso de las cuotas de recursos y los presupuestos de interrupciones de pod.
* [Procedimientos recomendados para las características avanzadas del programador](operator-best-practices-advanced-scheduler.md)
    * Incluye el uso de defectos y tolerancias, los selectores de nodo y afinidad, y la afinidad y falta de afinidad entre pods.
* [Procedimientos recomendados para autenticación y autorización](operator-best-practices-identity.md)
    * Incluye la integración en Azure Active Directory, el uso del control de acceso basado en rol de Kubernetes (RBAC de Kubernetes), el uso de Azure RBAC y el uso de identidades de pod.

**Seguridad**

* [Procedimientos recomendados para actualizaciones y seguridad de clústeres](operator-best-practices-cluster-security.md)
    * Incluye la protección del acceso al servidor de API, limitar el acceso a los contenedores y la administración de las actualizaciones y los reinicios de nodos.
* [Procedimientos recomendados para la administración y la protección de las imágenes de contenedor](operator-best-practices-container-image-management.md)
    * Incluye la protección de la imagen y los runtime y compilaciones automatizadas en actualizaciones de la imagen base.
* [Procedimientos recomendados para la seguridad de pods](developer-best-practices-pod-security.md)
    * Incluye la protección del acceso a los recursos, la limitación de la exposición de las credenciales y el uso de identidades de pods y almacenes de claves digitales.

**Red y almacenamiento**

* [Procedimientos recomendados para la conectividad de red](operator-best-practices-network.md)
    * Incluye distintos modelos de red, el uso de firewalls de aplicaciones web (WAF) y de entrada, y la protección del acceso SSH a los nodos.
* [Procedimientos recomendados para almacenamiento y copias de seguridad](operator-best-practices-storage.md)
    * Incluye la elección del tipo de almacenamiento y el tamaño de nodo apropiados, el aprovisionamiento dinámico de volúmenes, y copias de seguridad de datos.

**Ejecución de cargas de trabajo preparadas para la empresa**

* [Procedimientos recomendados para continuidad empresarial y recuperación ante desastres](operator-best-practices-multi-region.md)
    * Incluye el uso de pares de regiones, varios clústeres con Azure Traffic Manager y la replicación geográfica de las imágenes de contenedor.

## <a name="developer-best-practices"></a>Procedimientos recomendados para desarrolladores

Como desarrollador o propietario de una aplicación, puede simplificar su experiencia de desarrollo y definir las necesidades de rendimiento de la aplicación.

* [Procedimientos recomendados para desarrolladores de aplicaciones para administrar recursos](developer-best-practices-resource-management.md)
    * Incluye la definición de las solicitudes y los límites de recursos de pod, la configuración de las herramientas de desarrollo y la comprobación de problemas de la aplicación.
* [Procedimientos recomendados para la seguridad de pods](developer-best-practices-pod-security.md)
    * Incluye la protección del acceso a los recursos, la limitación de la exposición de las credenciales y el uso de identidades de pods y almacenes de claves digitales.

## <a name="kubernetes--aks-concepts"></a>Conceptos de Kubernetes/AKS

Para ayudar a comprender algunas de las características y componentes de estos procedimientos recomendados, también puede consultar los siguientes artículos conceptuales para clústeres en Azure Kubernetes Service (AKS):

* [Conceptos básicos de Kubernetes](concepts-clusters-workloads.md)
* [Acceso e identidad](concepts-identity.md)
* [Conceptos de seguridad](concepts-security.md)
* [Conceptos de red](concepts-network.md)
* [Opciones de almacenamiento](concepts-storage.md)
* [Opciones de escalado](concepts-scale.md)

## <a name="next-steps"></a>Pasos siguientes

Si necesita empezar a trabajar con AKS, siga uno de los tutoriales de inicio rápido para implementar un clúster de Azure Kubernetes Service (AKS) con la [CLI de Azure](kubernetes-walkthrough.md) o [Azure Portal](kubernetes-walkthrough-portal.md).
