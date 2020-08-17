---
title: Preguntas más frecuentes sobre Azure Kubernetes Service (AKS)
description: Encuentre respuestas a algunas de las preguntas comunes sobre Azure Kubernetes Service (AKS).
ms.topic: conceptual
ms.date: 07/31/2020
ms.openlocfilehash: 6d30bc1faa6a896233d09f9247fe8c56f82371fa
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87485633"
---
# <a name="frequently-asked-questions-about-azure-kubernetes-service-aks"></a>Preguntas más frecuentes sobre Azure Kubernetes Service (AKS)

En este artículo se abordan las preguntas más frecuentes sobre Azure Kubernetes Service (AKS).

## <a name="which-azure-regions-currently-provide-aks"></a>¿Qué regiones de Azure proporcionan actualmente AKS?

Para obtener una lista completa de las regiones disponibles, consulte [AKS regions and availability][aks-regions] (Regiones y disponibilidad de AKS).

## <a name="can-i-spread-an-aks-cluster-across-regions"></a>¿Puedo propagar un clúster de AKS entre regiones?

No. Los clústeres de AKS son recursos regionales y no pueden abarcar regiones. Consulte [Procedimientos recomendados de continuidad empresarial y recuperación ante desastres][bcdr-bestpractices] para obtener instrucciones sobre cómo crear una arquitectura que incluya varias regiones.

## <a name="can-i-spread-an-aks-cluster-across-availability-zones"></a>¿Puedo propagar un clúster de AKS entre zonas de disponibilidad?

Sí. Puede implementar un clúster de AKS en una o más [zonas de disponibilidad][availability-zones] de [regiones que las admitan][az-regions].

## <a name="can-i-limit-who-has-access-to-the-kubernetes-api-server"></a>¿Puedo limitar quién tiene acceso al servidor de API de Kubernetes?

Sí. Hay dos opciones para limitar el acceso al servidor de API:

- Use [intervalos de direcciones IP autorizadas del servidor de API][api-server-authorized-ip-ranges] si quiere mantener un punto de conexión público para el servidor de API, pero restringir el acceso a un conjunto de intervalos de direcciones IP de confianza.
- Use [un clúster privado][private-clusters] si quiere limitar el servidor de API para que *solo* sea accesible desde dentro de la red virtual.

## <a name="can-i-have-different-vm-sizes-in-a-single-cluster"></a>¿Puedo tener diferentes tamaños de máquina virtual en un único clúster?

Sí, puede usar diferentes tamaños de máquinas virtuales en el clúster de AKS mediante la creación de [varios grupos de nodos][multi-node-pools].

## <a name="are-security-updates-applied-to-aks-agent-nodes"></a>¿Se aplican las actualizaciones de seguridad a los nodos de agente de AKS?

Azure aplica automáticamente revisiones de seguridad a los nodos de Linux del clúster siguiendo una programación nocturna. Sin embargo, es responsabilidad suya asegurarse de que los nodos de Linux se reinician según sea necesario. Cuenta con varias opciones para reiniciar los nodos:

- Manualmente, mediante Azure Portal o la CLI de Azure.
- Mediante la actualización del clúster de AKS. Las actualizaciones del clúster [acordonan y purgan los nodos][cordon-drain] automáticamente, y luego ponen un nuevo nodo en línea con la imagen de Ubuntu más reciente y una nueva versión de revisión o una versión secundaria de Kubernetes. Para más información, consulte [Actualización de un clúster de Azure Kubernetes Service (AKS)][aks-upgrade].
- Mediante [Kured](https://github.com/weaveworks/kured), un demonio de reinicio de código abierto para Kubernetes. Kured se ejecuta como un elemento [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/) y supervisa en todos los nodos la presencia de un archivo que indique que hace falta un reinicio. En el clúster, los reinicios del sistema operativo se administran con el mismo [proceso de acordonar y purgar][cordon-drain] que una actualización de clúster.

Para obtener más información sobre el uso de Kured, consulte [Apply security and kernel updates to nodes in AKS][node-updates-kured] (Aplicación de actualizaciones de kernel y de seguridad en los nodos de AKS).

### <a name="windows-server-nodes"></a>Nodos de Windows Server

Para los nodos de Windows Server, Windows Update no ejecuta ni aplica las actualizaciones más recientes de manera automática. En una programación normal del ciclo de versiones de Windows Update y su proceso de validación propio, debe realizar una actualización en el clúster y los grupos de nodos de Windows Server en el clúster de AKS. Este proceso de actualización crea nodos que ejecutan la imagen y las revisiones más recientes de Windows Server y elimina los nodos anteriores. Para obtener más información sobre este proceso, consulte [Actualización de un grupo de nodos en AKS][nodepool-upgrade].

## <a name="why-are-two-resource-groups-created-with-aks"></a>¿Por qué se crean dos grupos de recursos con AKS?

AKS se basa en diversos recursos de infraestructura de Azure, como los conjuntos de escalado de máquinas virtuales, las redes virtuales y los discos administrados. Esto le permite utilizar muchas de las funcionalidades básicas de la plataforma de Azure en el entorno de Kubernetes administrado que proporciona AKS. Por ejemplo, la mayoría de los tipos de máquinas virtuales de Azure se pueden usar directamente con AKS, y Azure Reservations se puede usar para recibir descuentos automáticamente en esos recursos.

Para habilitar esta arquitectura, cada implementación de AKS abarca dos grupos de recursos:

1. Debe cree el primer grupo de recursos. Este grupo solo contiene el recurso del servicio de Kubernetes. El proveedor de recursos de AKS crea automáticamente el segundo grupo de recursos durante la implementación. Un ejemplo del segundo grupo de recursos es *MC_myResourceGroup_myAKSCluster_eastus*. Para obtener información sobre cómo especificar el nombre de este segundo grupo de recursos, consulte la sección siguiente.
1. El segundo grupo de recursos, conocido como el *grupo de recursos del nodo*, contiene todos los recursos de infraestructura asociados con el clúster. Estos recursos incluyen las máquinas virtuales de nodos de Kubernetes, las redes virtuales y el almacenamiento. De forma predeterminada, el grupo de recursos del nodo tiene un nombre como *MC_myResourceGroup_myAKSCluster_eastus*. AKS elimina automáticamente el recurso del nodo cada vez que se elimina el clúster, por lo que solo se debe usar para los recursos que comparten el ciclo de vida del clúster.

## <a name="can-i-provide-my-own-name-for-the-aks-node-resource-group"></a>¿Puedo proporcionar mi propio nombre para el grupo de recursos del nodo de AKS?

Sí. De forma predeterminada, AKS asignará el nombre *MC_resourcegroupname_clustername_location* al grupo de recursos del nodo, pero también puede proporcionar su propio nombre.

Para especificar un nombre de su elección para el grupo de recursos, instale la versión de la extensión de la CLI de Azure [aks-preview][aks-preview-cli]*0.3.2* o una posterior. Cuando cree un clúster de AKS mediante el comando [az aks create][az-aks-create], use el parámetro *--node-resource-group* y especifique un nombre para el grupo de recursos. Si [usa una plantilla de Azure Resource Manager][aks-rm-template] para implementar un clúster de AKS, puede definir el nombre del grupo de recursos mediante la propiedad *nodeResourceGroup*.

* El proveedor de recursos de Azure crea automáticamente el grupo de recursos secundario en su propia suscripción.
* Solo puede especificar un nombre personalizado para el grupo de recursos cuando cree el clúster.

Mientras trabaje con el grupo de recursos del nodo, tenga en cuenta que no puede realizar lo siguiente:

* Especificar un grupo de recursos existente para el grupo de recursos del nodo.
* Especificar otra suscripción para el grupo de recursos del nodo.
* Cambiar el nombre del grupo de recursos del nodo una vez se haya creado el clúster.
* Especificar los nombres de los recursos administrados del grupo de recursos del nodo.
* Modificar o eliminar las etiquetas creadas en Azure de los recursos administrados del grupo de recursos del nodo. (Puede ver la información adicional en la sección siguiente).

## <a name="can-i-modify-tags-and-other-properties-of-the-aks-resources-in-the-node-resource-group"></a>¿Puedo modificar etiquetas y otras propiedades de los recursos de AKS en el grupo de recursos del nodo?

Si modifica o elimina etiquetas creadas por Azure y otras propiedades de recursos en el grupo de recursos del nodo, es posible que obtenga resultados inesperados, como errores de escalado y actualización. AKS permite crear y modificar etiquetas personalizadas generadas por usuarios finales, que se pueden agregar al [crear un grupo de nodos](use-multiple-node-pools.md#specify-a-taint-label-or-tag-for-a-node-pool). Es posible que quiera crear o modificar etiquetas personalizadas, por ejemplo, para asignar un centro de costos o una unidad de negocio. Esto también puede conseguirse creando directivas de Azure cuyo ámbito sea el grupo de recursos administrado.

Pero la modificación de las **etiquetas creadas por Azure** en los recursos del grupo de recursos del nodo en el clúster de AKS es una acción no admitida que interrumpe el objetivo de nivel de servicio (SLO). Para obtener más información, consulte [¿AKS ofrece un contrato de nivel de servicio?](#does-aks-offer-a-service-level-agreement)

## <a name="what-kubernetes-admission-controllers-does-aks-support-can-admission-controllers-be-added-or-removed"></a>¿Qué controladores de admisión de Kubernetes admite AKS? ¿Se pueden agregar o eliminar los controladores de admisión?

AKS admite los siguientes [controladores de admisión][admission-controllers]:

- *NamespaceLifecycle*
- *LimitRanger*
- *ServiceAccount*
- *DefaultStorageClass*
- *DefaultTolerationSeconds*
- *MutatingAdmissionWebhook*
- *ValidatingAdmissionWebhook*
- *ResourceQuota*

Actualmente, no puede modificar la lista de controladores de admisión en AKS.

## <a name="can-i-use-admission-controller-webhooks-on-aks"></a>¿Puedo utilizar webhooks de controlador de admisión en AKS?

Sí, puede usar webhooks de controlador de admisión en AKS. Se recomienda excluir los espacios de nombres internos de AKS que estén marcados con la **etiqueta de plano de control**. Por ejemplo, al agregar lo siguiente a la configuración de webhook:

```
namespaceSelector:
    matchExpressions:
    - key: control-plane
      operator: DoesNotExist
```

## <a name="can-admission-controller-webhooks-impact-kube-system-and-internal-aks-namespaces"></a>¿Los webhooks de controlador de admisión pueden afectar a los espacios de nombres internos de kube-system y de AKS?

Para proteger la estabilidad del sistema y evitar que los controladores de admisión personalizados afecten a los servicios internos de kube-system, el espacio de nombres de AKS tiene un **ejecutor de admisiones**, que excluye automáticamente los espacios de nombres internos de kube-system y de AKS. Este servicio garantiza que los controladores de admisión personalizados no afecten a los servicios que se ejecutan en kube-system.

Si tiene un caso de uso crítico por tener algo implementado en kube-system (no recomendado) que necesita que esté incluido en el webhook de admisión personalizado, puede agregar la siguiente etiqueta o anotación para que el factor de admisión lo omita.

Etiqueta: ```"admissions.enforcer/disabled": "true"``` o anotación: ```"admissions.enforcer/disabled": true```

## <a name="is-azure-key-vault-integrated-with-aks"></a>¿Azure Key Vault se integra con AKS?

AKS no está integrado de forma nativa con Azure Key Vault en estos momentos. Sin embargo, el [proveedor de Azure Key Vault para el almacén de secretos de CSI][csi-driver] permite realizar una integración directa desde pods de Kubernetes a los secretos de Key Vault.

## <a name="can-i-run-windows-server-containers-on-aks"></a>¿Puedo ejecutar contenedores de Windows Server en AKS?

Sí, los contenedores de Windows Server están disponibles en AKS. Para ejecutar los contenedores de Windows Server en AKS, cree un grupo de nodos que ejecute Windows Server como sistema operativo invitado. Los contenedores de Windows Server solo pueden usar Windows Server 2019. Para empezar, vea [Creación de un clúster de AKS con un grupo de nodos de Windows Server][aks-windows-cli].

La compatibilidad de Windows Server con el grupo de nodos incluye algunas limitaciones que forman parte de la versión de Windows Server ascendente en el proyecto de Kubernetes. Para obtener más información sobre estas limitaciones, consulte [Windows Server containers in AKS limitations][aks-windows-limitations] (Limitaciones de los contenedores de Windows Server en AKS).

## <a name="does-aks-offer-a-service-level-agreement"></a>¿AKS ofrece un Acuerdo de Nivel de Servicio?

AKS proporciona garantías de Acuerdo de Nivel de Servicio como una característica opcional de complemento con [Acuerdo de Nivel de Servicio de tiempo de actividad][uptime-sla].

## <a name="can-i-apply-azure-reservation-discounts-to-my-aks-agent-nodes"></a>¿Puedo aplicar descuentos de las reservas de Azure en mis nodos de agente de AKS?

Los nodos de agente de AKS se facturan como máquinas virtuales de Azure estándares, por lo que, si ha comprado [reservas Azure][reservation-discounts] para el tamaño de máquina virtual que está usando en AKS, los descuentos se aplican automáticamente.

## <a name="can-i-movemigrate-my-cluster-between-azure-tenants"></a>¿Puedo mover o migrar mi clúster entre inquilinos de Azure?

Actualmente, no puede mover el clúster de AKS entre inquilinos.

## <a name="can-i-movemigrate-my-cluster-between-subscriptions"></a>¿Puedo mover o migrar mi clúster entre suscripciones?

Actualmente no se admite el movimiento de clústeres entre suscripciones.

## <a name="can-i-move-my-aks-clusters-from-the-current-azure-subscription-to-another"></a>¿Puedo trasladar los clústeres de AKS de la suscripción actual de Azure a otra? 

No se admite el traslado de clústeres de AKS y de los recursos asociados entre suscripciones de Azure.

## <a name="can-i-move-my-aks-cluster-or-aks-infrastructure-resources-to-other-resource-groups-or-rename-them"></a>¿Puedo mover el clúster de AKS o los recursos de infraestructura de AKS a otros grupos de recursos o cambiarles el nombre?

No se admite mover el clúster de AKS y sus recursos asociados ni cambiarles el nombre.

## <a name="why-is-my-cluster-delete-taking-so-long"></a>¿Por qué tarda tanto la eliminación del clúster? 

La mayoría de los clústeres se eliminan en el momento de la solicitud del usuario; en algunos casos, especialmente en aquellos en los que los clientes traen su propio grupo de recursos o se están realizando tareas entre grupos de recursos, la eliminación puede tardar más tiempo o producir un error. Si tiene un problema con las eliminaciones, compruebe que no tiene bloqueos en el grupo de recursos, que los recursos situados fuera del grupo están desasociados de este, etc.

## <a name="if-i-have-pod--deployments-in-state-nodelost-or-unknown-can-i-still-upgrade-my-cluster"></a>Si tengo un pod o implementaciones con el estado "NodeLost" (Nodo perdido) o "Unknown" (Desconocido), ¿puedo actualizar el clúster?

Se puede, pero AKS no lo recomienda. Lo ideal es realizar las actualizaciones si el estado del clúster es conocido o correcto.

## <a name="if-i-have-a-cluster-with-one-or-more-nodes-in-an-unhealthy-state-or-shut-down-can-i-perform-an-upgrade"></a>Si tengo un clúster con uno o varios nodos en un estado incorrecto o apagado, ¿puedo realizar una actualización?

No, elimine o quite los nodos con un estado erróneo antes de actualizar.

## <a name="i-ran-a-cluster-delete-but-see-the-error-errno-11001-getaddrinfo-failed"></a>Ejecuté una eliminación de clúster, pero me aparece el error `[Errno 11001] getaddrinfo failed` 

Normalmente, esto se debe a usuarios que tienen uno o varios grupos de seguridad de red (NSG) todavía en uso y asociados al clúster.  Quítelos e intente de nuevo la eliminación.

## <a name="i-ran-an-upgrade-but-now-my-pods-are-in-crash-loops-and-readiness-probes-fail"></a>Ejecuté una actualización, pero ahora mis pods están en bucles de bloqueo y se producen errores en los sondeos de preparación.

Confirme que la entidad de servicio no ha expirado.  Consulte: [Entidad de servicio de AKS](./kubernetes-service-principal.md) y [Credenciales de actualización de AKS](./update-credentials.md).

## <a name="my-cluster-was-working-but-suddenly-cannot-provision-loadbalancers-mount-pvcs-etc"></a>Mi clúster estaba funcionando, pero, de repente, no puede aprovisionar LoadBalancers, montar PVC, etc. 

Confirme que la entidad de servicio no ha expirado.  Consulte: [Entidad de servicio de AKS](./kubernetes-service-principal.md) y [Credenciales de actualización de AKS](./update-credentials.md).

## <a name="can-i-use-the-virtual-machine-scale-set-apis-to-scale-manually"></a>¿Puedo usar las API del conjunto de escalado de máquinas virtuales para escalar manualmente?

No, no se admiten las operaciones de escalado mediante el uso de las API del conjunto de escalado de máquinas virtuales. Use las API de AKS (`az aks scale`).

## <a name="can-i-use-virtual-machine-scale-sets-to-manually-scale-to-0-nodes"></a>¿Puedo usar conjuntos de escalado de máquinas virtuales para escalar manualmente a 0 nodos?

No, no se admiten las operaciones de escalado mediante el uso de las API del conjunto de escalado de máquinas virtuales.

## <a name="can-i-stop-or-de-allocate-all-my-vms"></a>¿Puedo detener o cancelar la asignación de todas las máquinas virtuales?

Aunque AKS tiene mecanismos de resistencia para admitir este tipo de configuración y recuperarse de él, no es una configuración recomendada.

## <a name="can-i-use-custom-vm-extensions"></a>¿Puedo usar extensiones de máquina virtual personalizadas?

No, AKS es un servicio administrado y no se admite la manipulación de los recursos de IaaS. Para instalar componentes personalizados, etc. Aproveche las API y los mecanismos de Kubernetes. Por ejemplo, aproveche DaemonSets para instalar los componentes necesarios.

## <a name="does-aks-store-any-customer-data-outside-of-the-clusters-region"></a>¿AKS guarda datos de los clientes fuera de la región del clúster?

No. Todos los datos que se crean en un clúster de AKS se mantienen en la región del clúster.

<!-- LINKS - internal -->

[aks-upgrade]: ./upgrade-cluster.md
[aks-cluster-autoscale]: ./cluster-autoscaler.md
[aks-advanced-networking]: ./configure-azure-cni.md
[aks-rbac-aad]: ./azure-ad-integration-cli.md
[node-updates-kured]: node-updates-kured.md
[aks-preview-cli]: /cli/azure/ext/aks-preview/aks
[az-aks-create]: /cli/azure/aks#az-aks-create
[aks-rm-template]: /azure/templates/microsoft.containerservice/2019-06-01/managedclusters
[aks-cluster-autoscaler]: cluster-autoscaler.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[aks-windows-cli]: windows-container-cli.md
[aks-windows-limitations]: windows-node-limitations.md
[reservation-discounts]:../cost-management-billing/reservations/save-compute-costs-reservations.md
[api-server-authorized-ip-ranges]: ./api-server-authorized-ip-ranges.md
[multi-node-pools]: ./use-multiple-node-pools.md
[availability-zones]: ./availability-zones.md
[private-clusters]: ./private-clusters.md
[bcdr-bestpractices]: ./operator-best-practices-multi-region.md#plan-for-multiregion-deployment
[availability-zones]: ./availability-zones.md
[az-regions]: ../availability-zones/az-region.md
[uptime-sla]: ./uptime-sla.md

<!-- LINKS - external -->
[aks-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[auto-scaler]: https://github.com/kubernetes/autoscaler
[cordon-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/
[admission-controllers]: https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/
[private-clusters-github-issue]: https://github.com/Azure/AKS/issues/948
[csi-driver]: https://github.com/Azure/secrets-store-csi-driver-provider-azure
[vm-sla]: https://azure.microsoft.com/support/legal/sla/virtual-machines/
