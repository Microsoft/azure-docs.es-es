---
title: 'Conceptos: seguridad en Azure Kubernetes Service (AKS)'
description: Obtenga información acerca de la seguridad en Azure Kubernetes Service (AKS), incluidos los secretos de Kubernetes, las directivas de red, y la comunicación con el maestro y los nodos.
services: container-service
author: mlearned
ms.topic: conceptual
ms.date: 07/01/2020
ms.author: mlearned
ms.openlocfilehash: e5f137808bb5e4c6876206bca7950117edb85aab
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005676"
---
# <a name="security-concepts-for-applications-and-clusters-in-azure-kubernetes-service-aks"></a>Conceptos de seguridad de las aplicaciones y los clústeres en Azure Kubernetes Service (AKS)

Para proteger los datos de los clientes a medida que ejecuta cargas de trabajo de aplicaciones en Azure Kubernetes Service (AKS), es importante tener en cuenta la seguridad del clúster. Kubernetes incluye componentes de seguridad tales como *directivas de red* y *secretos*. Posteriormente, Azure agrega componentes, como grupos de seguridad de red y actualizaciones de clúster organizadas. Estos componentes de seguridad se combinan para que el clúster de AKS siga ejecutando las actualizaciones de seguridad del sistema operativo y las versiones de Kubernetes más recientes, con tráfico de pods seguro y acceso a credenciales confidenciales.

En este artículo se presentan los conceptos básicos para proteger sus aplicaciones en AKS:

- [Conceptos de seguridad de las aplicaciones y los clústeres en Azure Kubernetes Service (AKS)](#security-concepts-for-applications-and-clusters-in-azure-kubernetes-service-aks)
  - [Seguridad de componentes maestros](#master-security)
  - [Seguridad de nodos](#node-security)
    - [Aislamiento de proceso](#compute-isolation)
  - [Actualización de un clúster de Service Fabric](#cluster-upgrades)
    - [Acordonamiento y purga](#cordon-and-drain)
  - [Seguridad de las redes](#network-security)
    - [Grupos de seguridad de red de Azure](#azure-network-security-groups)
  - [Secretos de Kubernetes](#kubernetes-secrets)
  - [Pasos siguientes](#next-steps)

## <a name="master-security"></a>Seguridad de componentes maestros

En AKS, los componentes maestros de Kubernetes forman parte del servicio administrado que proporciona Microsoft. Cada clúster de AKS tiene su propio maestro de Kubernetes dedicado con un solo inquilino para proporcionar el servidor de API, Scheduler, etc. Microsoft administra y mantiene este maestro.

De forma predeterminada, el servidor de API de Kubernetes utiliza una dirección IP pública y un nombre de dominio completo (FQDN). Puede limitar el acceso al punto de conexión del servidor de API mediante los [intervalos IP autorizados][authorized-ip-ranges]. También puede crear un [clúster privado][private-clusters] por completo para limitar el acceso del servidor de la API a la red virtual.

Puede regular el acceso al servidor de API mediante el control de acceso basado en rol (RBAC) de Kubernetes y Azure Active Directory. Para más información, consulte el artículo de [integración de Azure AD con AKS][aks-aad].

## <a name="node-security"></a>Seguridad de nodos

Los nodos de AKS son máquinas virtuales de Azure de los que usted realiza la administración y el mantenimiento. Los nodos de Linux ejecutan una distribución Ubuntu optimizada con el tiempo de ejecución del contenedor de Moby. Los nodos de Windows Server ejecutan una versión de Windows Server 2019 y también, el entorno de ejecución del contenedor de Moby. Cuando se crea o se escala verticalmente un clúster de AKS, los nodos se implementan automáticamente con las actualizaciones de seguridad del sistema operativo y las configuraciones más recientes.

La plataforma Azure aplica automáticamente las revisiones de seguridad del sistema operativo a los nodos de Linux del clúster durante la noche. Si una actualización de seguridad del sistema operativo de Linux requiere un reinicio del host, este no se realiza automáticamente. Puede reiniciar manualmente los nodos de Linux o bien optar por un enfoque común que consiste en usar [Kured][kured], un demonio de reinicio de código abierto para Kubernetes. Kured se ejecuta como un elemento [DaemonSet][aks-daemonsets] y supervisa cada nodo para comprobar la presencia de un archivo que indique que hace falta un reinicio. Los reinicios se administran en el clúster con el mismo [proceso de acordonar y purgar](#cordon-and-drain) que una actualización de clúster.

Para los nodos de Windows Server, Windows Update no ejecuta ni aplica las actualizaciones más recientes de manera automática. En una programación normal del ciclo de versiones de Windows Update y su proceso de validación propio, debe realizar una actualización de los grupos de nodos de Windows Server en el clúster de AKS. Este proceso de actualización crea nodos que ejecutan la imagen y las revisiones más recientes de Windows Server y elimina los nodos anteriores. Para obtener más información sobre este proceso, consulte [Actualización de un grupo de nodos en AKS][nodepool-upgrade].

Los nodos se implementan en una subred de una red privada virtual, sin ninguna dirección IP pública asignada. Para fines de administración y solución de problemas, SSH está habilitado de forma predeterminada. El acceso de SSH solo está disponible mediante la dirección IP interna.

Para proporcionar almacenamiento, los nodos usan Azure Managed Disks. Para la mayoría de los tamaños de nodo de máquina virtual, estos son los discos Premium respaldados por SSD de alto rendimiento. Los datos almacenados en discos administrados se cifran automáticamente en reposo dentro de la plataforma Azure. Para mejorar la redundancia, estos discos también se replican de forma segura en el centro de datos de Azure.

Los entornos de Kubernetes en AKS o en cualquier otro lugar no están completamente seguros en este momento ante la utilización de multiinquilinos hostiles. Otras características de seguridad, como las *directivas de seguridad de pods* o un control de acceso basado en rol (RBAC) más detallado para los nodos, dificultan las vulnerabilidades. Sin embargo, para que la seguridad resulte efectiva cuando se ejecutan cargas de trabajo multiinquilino hostiles, el hipervisor es el único nivel de seguridad en el que debe confiar. El dominio de seguridad de Kubernetes se convierte en todo el clúster, no en un nodo específico. En el caso de estos tipos de cargas de trabajo multiinquilino hostiles, debe usar clústeres que estén físicamente aislados. Para obtener más información sobre las formas de aislar las cargas de trabajo, consulte [Procedimientos recomendados para el aislamiento de clústeres en AKS][cluster-isolation].

### <a name="compute-isolation"></a>Aislamiento de proceso

 Ciertas cargas de trabajo pueden requerir un alto grado de aislamiento de otras cargas de trabajo del cliente debido a los requisitos normativos o de cumplimiento. Para estas cargas de trabajo, Azure proporciona [máquinas virtuales aisladas](../virtual-machines/isolation.md), que se pueden usar como nodos de agente en un clúster de AKS. Estas máquinas virtuales aisladas están aisladas de un tipo de hardware específico y están dedicadas a un solo cliente. 

 Para usar estas máquinas virtuales aisladas con un clúster de AKS, seleccione uno de los tamaños de máquinas virtuales aisladas que aparecen [aquí](../virtual-machines/isolation.md) como **tamaño de nodo** al crear un clúster de AKS o agregar un grupo de nodos.


## <a name="cluster-upgrades"></a>Actualizaciones de clústeres

Por motivos de seguridad y cumplimiento, o para usar las características más recientes, Azure proporciona herramientas para orquestar la actualización de un clúster y de componentes de AKS. La orquestación de esta actualización incluye los componentes tanto de maestro como de agente de Kubernetes. Puede ver una [lista de las versiones de Kubernetes disponibles](supported-kubernetes-versions.md) para su clúster de AKS. Para iniciar el proceso de actualización, especifique una de estas versiones disponibles. A continuación, Azure acordona y purga con seguridad cada uno de los nodos de AKS y lleva a cabo la actualización.

### <a name="cordon-and-drain"></a>Acordonar y purgar

Durante el proceso de actualización, los nodos de AKS se acordonan individualmente desde el clúster de modo que no se programen nuevos pods en ellos. A continuación, los nodos se purgan y actualizan de la siguiente manera:

- Se implementa un nuevo nodo en el grupo de nodos. Este nodo ejecuta la imagen de sistema operativo y las revisiones más recientes.
- Se identifica uno de los nodos existentes para la actualización. Los pods de este nodo finalizan correctamente y se programan en los otros nodos del grupo.
- Este nodo existente se elimina del clúster de AKS.
- El siguiente nodo del clúster se acordona y purga mediante el mismo proceso hasta que todos los nodos se reemplazan correctamente como parte de la actualización.

Para más información, consulte [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade-cluster].

## <a name="network-security"></a>Seguridad de las redes

Por motivos de conectividad y seguridad con las redes locales, puede implementar el clúster de AKS en subredes de red virtual de Azure existentes. Estas redes virtuales pueden presentar una conexión de VPN de sitio a sitio o ExpressRoute de Azure de vuelta a su red local. Los controladores de entrada de Kubernetes pueden definirse con direcciones IP internas privadas, de modo que los servicios solo sean accesibles a través de esta conexión de red interna.

### <a name="azure-network-security-groups"></a>Grupos de seguridad de red de Azure

Para filtrar el flujo del tráfico en las redes virtuales, Azure usa reglas de grupo de seguridad de red. Estas reglas definen los intervalos, los puertos y los protocolos de IP de origen y destino, a los que se permite o deniega el acceso a los recursos. Se crean reglas predeterminadas para permitir el tráfico TLS al servidor de API de Kubernetes. A medida que crea servicios con equilibradores de carga, asignaciones de puertos o rutas de entrada, AKS modifica automáticamente el grupo de seguridad de red para que el tráfico fluya adecuadamente.

En los casos en los que proporcione su propia subred para el clúster de AKS y quiera modificar el flujo de tráfico, no modifique el grupo de seguridad de red del nivel de subred que administra AKS. Puede crear grupos de seguridad de red del nivel de subred adicionales para modificar el flujo de tráfico siempre y cuando no interfieran con el tráfico necesario para administrar el clúster, como el acceso al equilibrador de carga, la comunicación con el plano de control y la [salida][aks-limit-egress-traffic].

### <a name="kubernetes-network-policy"></a>Directiva de red de Kubernetes

Para limitar el tráfico de red entre los pods del clúster, AKS ofrece compatibilidad con las [directivas de red de Kubernetes][network-policy]. Con las directivas de red, puede optar por permitir o denegar rutas de acceso de red específicas en el clúster en función de los espacios de nombres y los selectores de etiquetas.

## <a name="kubernetes-secrets"></a>Secretos de Kubernetes

Un *secreto* de Kubernetes se usa para insertar datos confidenciales en pods, como credenciales de acceso o claves. En primer lugar, cree un secreto mediante la API de Kubernetes. Al definir el pod o la implementación, puede solicitar un secreto específico. Los secretos solo se proporcionan a los nodos que tienen un pod programado que lo requiere. El secreto se almacena en el sistema *tmpfs*, no se escribe en el disco. Cuando se elimina el último pod en un nodo que requiere un secreto, el secreto se elimina del sistema tmpfs del nodo. Los secretos se almacenan en un espacio de nombres determinado y solo son accesibles para los pods del mismo espacio de nombres.

El uso de secretos reduce la información confidencial que se define en el pod o el manifiesto YAML del servicio. En su lugar, solicite el secreto almacenado en el servidor de API de Kubernetes como parte de su manifiesto YAML. Este enfoque proporciona acceso al secreto solo al pod específico. Nota: Los archivos de manifiesto de secreto sin formato contienen los datos secretos en formato base64 (consulte la [documentación oficial][secret-risks] para más detalles). Por lo tanto, este archivo debe considerarse como información confidencial y no confirmarse en el control de código fuente.

Los secretos de Kubernetes se almacenan en etcd, un almacén distribuido de pares clave-valor. El almacén etcd está totalmente administrado por AKS y [los datos se cifran en reposo dentro de la plataforma de Azure][encryption-atrest]. 

## <a name="next-steps"></a>Pasos siguientes

Para empezar a proteger los clústeres de AKS, consulte [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade-cluster].

Para los procedimientos recomendados asociados, consulte [Procedimientos recomendados para la seguridad de clústeres y las actualizaciones en AKS][operator-best-practices-cluster-security] y [Procedimientos recomendados para la seguridad de pod en AKS][developer-best-practices-pod-security].

Para obtener más información sobre los conceptos básicos de Kubernetes y AKS, consulte los artículos siguientes:

- [Clústeres y cargas de trabajo de Kubernetes/AKS][aks-concepts-clusters-workloads]
- [Identidad de Kubernetes/AKS][aks-concepts-identity]
- [Redes virtuales de Kubernetes/AKS][aks-concepts-network]
- [Almacenamiento de Kubernetes/AKS][aks-concepts-storage]
- [Escala de Kubernetes/AKS][aks-concepts-scale]

<!-- LINKS - External -->
[kured]: https://github.com/weaveworks/kured
[kubernetes-network-policies]: https://kubernetes.io/docs/concepts/services-networking/network-policies/
[secret-risks]: https://kubernetes.io/docs/concepts/configuration/secret/#risks
[encryption-atrest]: ../security/fundamentals/encryption-atrest.md

<!-- LINKS - Internal -->
[aks-daemonsets]: concepts-clusters-workloads.md#daemonsets
[aks-upgrade-cluster]: upgrade-cluster.md
[aks-aad]: ./azure-ad-integration-cli.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-identity]: concepts-identity.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-network]: concepts-network.md
[aks-limit-egress-traffic]: limit-egress-traffic.md
[cluster-isolation]: operator-best-practices-cluster-isolation.md
[operator-best-practices-cluster-security]: operator-best-practices-cluster-security.md
[developer-best-practices-pod-security]:developer-best-practices-pod-security.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[authorized-ip-ranges]: api-server-authorized-ip-ranges.md
[private-clusters]: private-clusters.md
[network-policy]: use-network-policies.md
