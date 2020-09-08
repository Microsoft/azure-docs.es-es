---
title: Introducción a Azure Kubernetes Service
description: Aprenda las características y ventajas de Azure Kubernetes Service para implementar y administrar aplicaciones basadas en contenedor en Azure.
services: container-service
ms.topic: overview
ms.date: 05/06/2019
ms.custom: mvc
ms.openlocfilehash: f4c1e96a0603caa8e026f1968299fa24b8755a42
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88003202"
---
# <a name="azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS)

Azure Kubernetes Service (AKS) simplifica la implementación de los clústeres de Kubernetes administrados en Azure. AKS reduce la complejidad y la sobrecarga operativa de la administración de Kubernetes al descargar gran parte de esa responsabilidad en Azure. Como servicio hospedado de Kubernetes, Azure maneja tareas críticas como la supervisión del estado y el mantenimiento para usted. Azure administra los maestros de Kubernetes. Usted solo administra y mantiene los nodos del agente. Como servicio de Kubernetes administrado, AKS es gratuito: solo tiene que pagar los nodos de agente incluidos en sus clústeres, no los maestros.

Puede crear un clúster de AKS en Azure Portal con la CLI de Azure, o bien opciones de implementación controladas por plantillas como las plantillas de Resource Manager y Terraform. Al implementar un clúster de AKS, el maestro y todos los nodos de Kubernetes se implementan y configuran automáticamente. Otras características como redes avanzadas, integración con Azure Active Directory y supervisión se pueden configurar también durante el proceso de implementación. Se admiten contenedores de Windows Server en AKS.

Para más información sobre los conceptos básicos de Kubernetes, consulte [Conceptos básicos de Kubernetes para AKS][concepts-clusters-workloads].

Para empezar, complete el inicio rápido de AKS [en Azure Portal][aks-portal] o [con la CLI de Azure][aks-cli].

[!INCLUDE [azure-lighthouse-supported-service](../../includes/azure-lighthouse-supported-service.md)]

## <a name="access-security-and-monitoring"></a>Acceso, seguridad y supervisión

Para mejorar la seguridad y la administración, AKS le permite integrarse con Azure Active Directory y usar controles de acceso basados en rol (RBAC) de Kubernetes. También puede supervisar el mantenimiento del clúster y recursos.

### <a name="identity-and-security-management"></a>Administración de identidades y seguridad

Para limitar el acceso a los recursos de clúster, AKS admite un [control de acceso basado en rol (RBAC) de Kubernetes][kubernetes-rbac]. RBAC le permite controlar el acceso a los recursos y espacios de nombres de Kubernetes, así como los permisos a esos recursos. También puede configurar un clúster de AKS para integrarse con Azure Active Directory (AD). Con la integración con Azure AD, se puede configurar el acceso a Kubernetes en función de la identidad y la pertenencia a grupos existentes. Se puede proporcionar acceso a los recursos de AKS a sus usuarios y grupos de Azure AD existentes, además de una experiencia de inicio de sesión integrada.

Para más información sobre la identidad, consulte [Opciones de acceso e identidad para AKS][concepts-identity].

Para proteger sus clústeres de AKS, consulte [Integrate Azure Active Directory with AKS][aks-aad] (Integración de Azure Active Directory con AKS).

### <a name="integrated-logging-and-monitoring"></a>Supervisión y registro integrados

Para conocer el rendimiento de su clúster de AKS y las aplicaciones implementadas, Azure Monitor para el estado de los contenedores recopila métricas de procesador y memoria de contenedores, nodos y controladores. Los registros de contenedor están disponibles y también puede [revisar los registros maestros de Kubernetes][aks-master-logs]. Estos datos de supervisión se almacenan en el área de trabajo de Azure Log Analytics y están disponibles mediante Azure Portal, la CLI de Azure o un punto de conexión de REST.

Para más información, consulte [Supervisión del mantenimiento de contenedores de Azure Kubernetes Service][container-health].

## <a name="clusters-and-nodes"></a>Clústeres y nodos

Los nodos de AKS se ejecutan en máquinas virtuales de Azure. Puede conectar el almacenamiento a nodos y pods, actualizar los componentes de clúster y usar GPU. AKS admite clústeres de Kubernetes que ejecutan varios grupos de nodos, con el fin de que se puedan usar sistemas operativos mixtos y contenedores de Windows Server. Los nodos de Linux ejecutan una imagen personalizada del sistema operativo Ubuntu y los nodos de Windows Server ejecutan una imagen personalizada del sistema operativo Windows Server 2019.

### <a name="cluster-node-and-pod-scaling"></a>Escalado de pods y nodos de clúster

A medida que cambia la demanda de recursos, el número de pods o nodos de clúster que ejecutan sus servicios puede escalarse vertical u horizontalmente. Puede usar tanto el escalador automático de pod horizontal como el escalador automático de clúster. Este enfoque de escalado permite al clúster de AKS ajustarse automáticamente a las demandas y ejecutar solo los recursos necesarios.

Para más información, consulte [Escalado de un clúster de Azure Kubernetes Service (AKS)][aks-scale].

### <a name="cluster-node-upgrades"></a>Actualizaciones de nodos de clúster

Azure Kubernetes Service ofrece varias versiones de Kubernetes. A medida que hay nuevas versiones disponibles en AKS, el clúster se puede actualizar mediante Azure Portal o la CLI de Azure. Durante el proceso de actualización, los nodos se acordonan y vacían minuciosamente para minimizar las interrupciones en las aplicaciones en ejecución.

Para obtener más información sobre las versiones del ciclo de vida, vea [Versiones de Kubernetes compatibles en Azure Kubernetes Service (AKS)][aks-supported versions]. Para obtener información sobre los pasos de actualización, vea [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade].

### <a name="gpu-enabled-nodes"></a>Nodos habilitados para GPU

AKS admite la creación de grupos de nodo habilitados para GPU. Azure proporciona actualmente única o varias máquinas virtuales habilitadas para GPU. Las máquinas virtuales habilitadas para GPU están diseñadas para cargas de trabajo de proceso intensivo, uso intensivo de gráficos y visualización.

Para más información, consulte [Uso de GPU en AKS][aks-gpu].

### <a name="storage-volume-support"></a>Compatibilidad con volúmenes de almacenamiento

Para admitir las cargas de trabajo de la aplicación, puede montar volúmenes de almacenamiento para los datos persistentes. Se pueden usar volúmenes estáticos y dinámicos. Dependiendo de cuántos pods conectados están destinados a compartir el almacenamiento, puede usar almacenamiento respaldado por Azure Disks para el acceso a un solo pod o Azure Files para el acceso a varios pods de forma simultánea.

Para más información, consulte [Opciones de almacenamiento para aplicaciones en AKS][concepts-storage].

Empiece con volúmenes persistentes dinámicos mediante [Azure Disks][azure-disk] o [Azure Files][azure-files].

## <a name="virtual-networks-and-ingress"></a>Redes virtuales y entrada

Un clúster de AKS se puede implementar en una red virtual existente. En esta configuración, cada pod del clúster tiene asignada una dirección IP en la red virtual y puede comunicarse directamente con los otros pods del clúster y otros nodos de la red virtual. Un pod se puede conectar también con otros servicios de una red virtual emparejada y con redes locales a través de ExpressRoute o conexiones VPN de sitio a sitio (S2S).

Para más información, consulte [Conceptos de red para aplicaciones en AKS][aks-networking].

Para empezar con el tráfico de entrada, consulte [Enrutamiento de aplicación HTTP][aks-http-routing].

### <a name="ingress-with-http-application-routing"></a>Entrada con el enrutamiento de aplicación HTTP

El complemento de enrutamiento de aplicación HTTP facilita el acceso a las aplicaciones implementadas en el clúster de AKS. Cuando está habilitada, la solución de enrutamiento de aplicaciones HTTP configura un controlador de entrada en el clúster de AKS. A medida que se implementan las aplicaciones, los nombres DNS con acceso público se configuran automáticamente. El enrutamiento de aplicación HTTP configura una zona DNS y la integra con el clúster de AKS. A continuación, puede implementar recursos de entrada de Kubernetes de la forma habitual.

Para empezar con el tráfico de entrada, consulte [Enrutamiento de aplicación HTTP][aks-http-routing].

## <a name="development-tooling-integration"></a>Integración de herramientas de desarrollo

Kubernetes cuenta con un completo ecosistema de herramientas de desarrollo y administración, como Helm y la extensión de Kubernetes para Visual Studio Code. Estas herramientas funcionan perfectamente con AKS.

Además, Azure Dev Spaces proporciona una experiencia de desarrollo rápida e iterativa para los equipos. Con una configuración mínima, puede ejecutar y depurar contenedores directamente en AKS. Para empezar, consulte [Azure Dev Spaces][azure-dev-spaces].

El proyecto de Azure DevOps ofrece una solución sencilla para llevar a Azure el código y el repositorio de Git existentes. El proyecto de DevOps crea automáticamente recursos de Azure, como AKS, una canalización de versión en Azure DevOps Services que incluye una canalización de compilación para CI, configura una canalización de versión para CD y crea un recurso de Azure Application Insights para la supervisión.

Para más información, consulte [Proyecto de Azure DevOps][azure-devops].

## <a name="docker-image-support-and-private-container-registry"></a>Compatibilidad con imágenes de Docker y registro de contenedor privado

AKS admite el formato de imagen de Docker. Para el almacenamiento privado de las imágenes de Docker, puede realizar la integración de AKS con Azure Container Registry (ACR).

Para crear un almacén de imágenes privado, consulte [Azure Container Registry][acr-docs].

## <a name="kubernetes-certification"></a>Certificación de Kubernetes

Azure Kubernetes Service (AKS) tiene el certificado CNCF de cumplimiento con Kubernetes.

## <a name="regulatory-compliance"></a>Cumplimiento de normativas

Azure Kubernetes Service (AKS) cumple con SOC, ISO, PCI DSS y HIPAA. Para más información, consulte [Introducción al cumplimiento de Microsoft Azure][compliance-doc].

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la implementación y administración de AKS con la guía de inicio rápido de la CLI de Azure.

> [!div class="nextstepaction"]
> [Inicio rápido de AKS][aks-cli]

<!-- LINKS - external -->
[aks-engine]: https://github.com/Azure/aks-engine
[kubectl-overview]: https://kubernetes.io/docs/user-guide/kubectl-overview/
[compliance-doc]: https://gallery.technet.microsoft.com/Overview-of-Azure-c1be3942

<!-- LINKS - internal -->
[acr-docs]: ../container-registry/container-registry-intro.md
[aks-aad]: ./azure-ad-integration-cli.md
[aks-cli]: ./kubernetes-walkthrough.md
[aks-gpu]: ./gpu-cluster.md
[aks-http-routing]: ./http-application-routing.md
[aks-networking]: ./concepts-network.md
[aks-portal]: ./kubernetes-walkthrough-portal.md
[aks-scale]: ./tutorial-kubernetes-scale.md
[aks-upgrade]: ./upgrade-cluster.md
[azure-dev-spaces]: ../dev-spaces/index.yml
[azure-devops]: ../devops-project/overview.md
[azure-disk]: ./azure-disks-dynamic-pv.md
[azure-files]: ./azure-files-dynamic-pv.md
[container-health]: ../azure-monitor/insights/container-insights-overview.md
[aks-master-logs]: view-master-logs.md
[aks-supported versions]: supported-kubernetes-versions.md
[concepts-clusters-workloads]: concepts-clusters-workloads.md
[kubernetes-rbac]: concepts-identity.md#kubernetes-role-based-access-control-rbac
[concepts-identity]: concepts-identity.md
[concepts-storage]: concepts-storage.md
