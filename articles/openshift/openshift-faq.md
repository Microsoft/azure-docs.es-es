---
title: Preguntas frecuentes sobre Red Hat OpenShift en Azure
description: Aquí tiene respuestas a las preguntas habituales sobre Red Hat OpenShift en Azure.
author: jimzim
ms.author: jzim
ms.service: container-service
ms.topic: conceptual
ms.date: 07/31/2020
ms.openlocfilehash: a8b5ec48b64341ad9eabd087d7ee20bb703198c6
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816242"
---
# <a name="azure-red-hat-openshift-faq"></a>Preguntas más frecuentes de Red Hat OpenShift en Azure

En este artículo se responde a las preguntas más frecuentes (P+F) sobre Red Hat OpenShift en Microsoft Azure.

## <a name="installation-and-upgrade"></a>Instalación y actualización

### <a name="which-azure-regions-are-supported"></a>¿Qué regiones de Azure se admiten?

A fin de obtener una lista de las regiones admitidas para Red Hat OpenShift en Azure 4.x, vea [Regiones disponibles](https://docs.openshift.com/aro/4/welcome/index.html#available-regions).

A fin de obtener una lista de las regiones admitidas para Red Hat OpenShift en Azure 3.11, vea [Productos disponibles por región](supported-resources.md#azure-regions).

### <a name="what-virtual-machine-sizes-can-i-use"></a>¿Qué tamaños de máquina virtual puedo usar?

A fin de obtener una lista de los tamaños de máquina virtual admitidos para Red Hat OpenShift en Azure 4, vea [Recursos compatibles con Red Hat OpenShift en Azure 4](support-policies-v4.md).

A fin de obtener una lista de los tamaños de máquina virtual admitidos para Red Hat OpenShift en Azure 3.11, vea [Recursos compatibles con Red Hat OpenShift en Azure 3.11](supported-resources.md).

### <a name="what-is-the-maximum-number-of-pods-in-an-azure-red-hat-openshift-cluster--what-is-the-maximum-number-of-pods-per-node-in-azure-red-hat-openshift"></a>¿Cuál es el número máximo de pods en un clúster de Red Hat OpenShift en Azure?  ¿Cuál es el número máximo de pods por nodo de Red Hat OpenShift en Azure?

El número real de pods admitidos depende de los requisitos de memoria, CPU y almacenamiento de una aplicación.

Red Hat OpenShift en Azure 4.x tiene un límite de 250 pods por nodo y otro de 100 nodos de ejecución. Esto establece el número máximo de pods admitido en un clúster a 250&times;100 = 25 000.

Red Hat OpenShift en Azure 3.11 tiene un límite de 50 pods por nodo y otro de 20 nodos de ejecución. Esto establece el número máximo de pods admitido en un clúster a 50&times;20 = 1000.

### <a name="can-a-cluster-have-compute-nodes-across-multiple-azure-regions"></a>¿Un clúster puede tener nodos de proceso en varias regiones de Azure?

No. Todos los nodos de un clúster de Red Hat OpenShift en Azure se deben originar en la misma región de Azure.

### <a name="can-a-cluster-be-deployed-across-multiple-availability-zones"></a>¿Se puede implementar un clúster en varias zonas de disponibilidad?

Sí. Esto sucede automáticamente si el clúster se implementa en una región de Azure que admite zonas de disponibilidad. Para más información, consulte [Zonas de disponibilidad](../availability-zones/az-overview.md#availability-zones).

### <a name="are-control-plane-nodes-abstracted-away-as-they-are-with-azure-kubernetes-service-aks"></a>¿Los nodos de plano de control se abstraen como lo hacen con Azure Kubernetes Service (AKS)?

No. Todos los recursos, incluidos los nodos de clúster maestro, se ejecutan en su suscripción de cliente. Estos tipos de recursos se colocan en un grupo de recursos de solo lectura.

### <a name="does-the-cluster-reside-in-a-customer-subscription"></a>¿El clúster se encuentra en una suscripción de cliente? 

La aplicación administrada de Azure reside en un grupo de recursos bloqueado con la suscripción del cliente. Los clientes pueden ver los objetos de ese grupo de recursos, pero no modificarlos.

### <a name="is-there-any-element-in-azure-red-hat-openshift-shared-with-other-customers-or-is-everything-independent"></a>¿Hay algún elemento en Red Hat OpenShift en Azure compartido con otros clientes? ¿O todo es independiente?

Cada clúster de Red Hat OpenShift en Azure está dedicado a un cliente determinado y vive en la suscripción del cliente. 

### <a name="are-infrastructure-nodes-available"></a>¿Están disponibles los nodos de infraestructura?

En los clústeres de Red Hat OpenShift en Azure 4.x, los nodos de infraestructura no están disponibles actualmente.

En los clústeres de Red Hat OpenShift en Azure 3.11, los nodos de infraestructura están incluidos de forma predeterminada.

## <a name="how-do-i-handle-cluster-upgrades"></a>¿Cómo administrar las actualizaciones del clúster?

Para información sobre las actualizaciones, el mantenimiento y las versiones admitidas, consulte la [guía del ciclo de vida de soporte técnico](support-lifecycle.md).

### <a name="how-will-the-host-operating-system-and-openshift-software-be-updated"></a>¿Cómo se actualizará el sistema operativo host y el software OpenShift?

Los sistemas operativos host y el software OpenShift se actualizan a medida que Red Hat OpenShift en Azure consume versiones secundarias y revisiones de OpenShift Container Platform ascendente.

### <a name="whats-the-process-to-reboot-the-updated-node"></a>¿Cuál es el proceso de reinicio del nodo actualizado?

Los nodos se reinician como parte de una actualización.

## <a name="cluster-operations"></a>Operaciones del clúster

### <a name="can-i-use-prometheus-to-monitor-my-applications"></a>¿Se puede usar Prometheus para supervisar las aplicaciones?

Prometheus viene preinstalado y configurado para los clústeres de Red Hat OpenShift en Azure 4.x. Obtenga más información sobre la [supervisión de clústeres](https://docs.openshift.com/container-platform/3.11/install_config/prometheus_cluster_monitoring.html).

En el caso de los clústeres de Red Hat OpenShift en Azure 3.11, se puede implementar Prometheus en su espacio de nombres y supervisar las aplicaciones en él. Para obtener más información, vea [Implementación de una instancia de Prometheus en un clúster de Red Hat OpenShift en Azure](howto-deploy-prometheus.md).

### <a name="can-i-use-prometheus-to-monitor-metrics-related-to-cluster-health-and-capacity"></a>¿Puedo usar Prometheus para supervisar las métricas relacionadas con la capacidad y el mantenimiento del clúster?

En Red Hat OpenShift en Azure 4.x: Sí.

En Red Hat OpenShift en Azure 3.11: No.

### <a name="can-logs-of-underlying-vms-be-streamed-out-to-a-customer-log-analysis-system"></a>¿Se pueden transmitir los registros de máquinas virtuales subyacentes a un sistema de análisis de registros de cliente?

Los registros de las máquinas virtuales subyacentes se controlan mediante el servicio administrado y no se exponen a los clientes.

### <a name="how-can-a-customer-get-access-to-metrics-like-cpumemory-at-the-node-level-to-take-action-to-scale-debug-issues-etc-i-cannot-seem-to-run-kubectl-top-on-an-azure-red-hat-openshift-cluster"></a>¿Cómo puede un cliente obtener acceso a métricas como CPU y memoria en el nivel de nodo para tomar medidas a fin de escalar, depurar incidencias, etc.? No parece que se pueda ejecutar kubectl en un clúster de Red Hat OpenShift en Azure.

En el caso de los clústeres de Red Hat OpenShift en Azure 4.x, la consola web de OpenShift contiene todas las métricas en el nivel de nodo. Para obtener más información, vea la documentación de Red Hat sobre [vista de la información del clúster](https://docs.openshift.com/aro/4/web_console/using-dashboard-to-get-cluster-information.html).

En el caso de los clústeres de Red Hat OpenShift en Azure 3.11, los clientes pueden acceder a las métricas de CPU o memoria en el nivel de nodo mediante el comando `oc adm top nodes` o `kubectl top nodes` con el rol de clúster customer-admin. Los clientes también pueden acceder a las métricas de CPU o memoria de `pods` con el comando `oc adm top pods` o `kubectl top pods`.

### <a name="if-we-scale-up-the-deployment-how-do-azure-fault-domains-map-into-pod-placement-to-ensure-all-pods-for-a-service-do-not-get-knocked-out-by-a-failure-in-a-single-fault-domain"></a>Si escalamos verticalmente la implementación, ¿cómo se asignan los dominios de error de Azure en la colocación de los pods para tener la seguridad de que todos los pods de un servicio no se han visto afectados por un error en un único dominio de error?

De forma predeterminada, hay cinco dominios de error cuando se usan conjuntos de escalado de máquinas virtuales en Azure. Cada instancia de máquina virtual de un conjunto de escalado se colocará en uno de estos dominios de error. Esto garantiza que las aplicaciones implementadas en los nodos de proceso de un clúster se colocarán en distintos dominios de error.

A fin de obtener más información, vea [Elección del número correcto de dominios de error para el conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-manage-fault-domains.md).

### <a name="is-there-a-way-to-manage-pod-placement"></a>¿Hay alguna manera de administrar la colocación de los pods?

Los clientes tienen la posibilidad de obtener nodos y ver etiquetas con el rol customer-admin. De esta forma, podrán dirigirse a cualquier máquina virtual del conjunto de escalado.

Es necesario tener cuidado al usar etiquetas específicas:

- No se debe usar el nombre de host. El nombre de host se rota con frecuencia con las actualizaciones y es seguro que cambiará.
- Si el cliente tiene una solicitud de etiquetas específicas o una estrategia de implementación, se podría hacer, pero harían falta labores de ingeniería, y esto no se admite actualmente.

Para obtener más información, vea [Control de la selección de ubicación del pod](https://docs.openshift.com/aro/4/nodes/scheduling/nodes-scheduler-about.html).

### <a name="is-the-image-registry-available-externally-so-i-can-use-tools-such-as-jenkins"></a>¿Está disponible externamente el registro de imágenes para poder usar herramientas como Jenkins?

En el caso de los clústeres 4.x, debe exponer un registro seguro y configurar la autenticación. Para obtener más información, vea la documentación de Red Hat siguiente:

- [Exposición de un registro](https://docs.openshift.com/aro/4/registry/securing-exposing-registry.html)
- [Acceso al registro](https://docs.openshift.com/aro/4/registry/accessing-the-registry.html)

En el caso de los clústeres 3.11, el registro de imágenes de Docker está disponible. El registro de Docker está disponible en `https://docker-registry.apps.<clustername>.<region>.azmosa.io/`. También se puede usar Azure Container Registry.

## <a name="networking"></a>Redes

### <a name="can-i-deploy-a-cluster-into-an-existing-virtual-network"></a>¿Puedo implementar un clúster en una red virtual existente?

En los clústeres 4.x, se puede implementar un clúster en una red virtual existente.

En los clústeres 3.11, no se puede implementar un clúster en una red virtual existente. Pero se puede conectar un clúster de Red Hat OpenShift en Azure 3.11 en una red virtual existente mediante emparejamiento.

### <a name="is-cross-namespace-networking-supported"></a>¿Se admiten las redes entre espacios de nombres?

Los administradores de proyectos de cliente e individuales pueden personalizar las redes entre espacios de nombres (incluido denegarlas) por proyecto con objetos `NetworkPolicy`.

### <a name="i-am-trying-to-peer-into-a-virtual-network-in-a-different-subscription-but-getting-failed-to-get-vnet-cidr-error"></a>Estoy intentando mirar una red virtual en otra suscripción pero recibo el error Failed to get VNet CIDR (Error al obtener el CIDR de red virtual).

En la suscripción que tiene la red virtual, asegúrese de registrar el proveedor `Microsoft.ContainerService` con el comando siguiente: `az provider register -n Microsoft.ContainerService --wait`.

### <a name="can-we-specify-ip-ranges-for-deployment-on-the-private-vnet-avoiding-clashes-with-other-corporate-vnets-once-peered"></a>¿Se pueden especificar intervalos IP para la implementación en la red virtual privada y evitar los conflictos con otras redes virtuales corporativas una vez emparejadas?

En los clústeres 4.x, puede especificar sus propios intervalos IP.

En los clústeres 3.11, Red Hat OpenShift en Azure admite el emparejamiento de red virtual y permite al cliente proporcionar una red virtual con la que emparejar y un CIDR de red virtual en el que funcionará la red OpenShift.

La red virtual que ha creado Red Hat OpenShift en Azure se protegerá y no aceptará cambios de configuración. La red virtual que está emparejada la controla el cliente y reside en su suscripción.

### <a name="is-the-software-defined-network-module-configurable"></a>¿Se puede configurar el módulo Red definida por el software?

La Red definida por el software es `openshift-ovs-networkpolicy` y no se puede configurar.

### <a name="what-azure-load-balancer-is-used-by-azure-red-hat-openshift--is-it-standard-or-basic-and-is-it-configurable"></a>¿Qué Equilibrador de carga de Azure usa Red Hat OpenShift en Azure?  ¿Es Estándar o Básico y es configurable?

Red Hat OpenShift en Azure usa Azure Load Balancer Estándar y no es configurable.

## <a name="permissions"></a>Permisos

### <a name="can-an-admin-manage-users-and-quotas"></a>¿Puede un administrador administrar usuarios y cuotas?

Sí. Un administrador de Red Hat OpenShift en Azure puede administrar usuarios y cuotas, además de acceder a todos los proyectos creados por los usuarios.

### <a name="can-i-restrict-a-cluster-to-only-certain-azure-ad-users"></a>¿Puedo restringir un clúster a solo determinados usuarios de Azure AD?

Sí. Puede restringir qué usuarios de Azure AD pueden iniciar sesión en un clúster configurando la aplicación de Azure AD. Para más información, consulte [Procedimientos para: restringir la aplicación a un conjunto de usuarios](../active-directory/develop/howto-restrict-your-app-to-a-set-of-users.md).

### <a name="can-i-restrict-users-from-creating-projects"></a>¿Puedo impedir que los usuarios creen proyectos?

Sí. Inicie sesión en el clúster como administrador y ejecute este comando:

```
oc adm policy \
    remove-cluster-role-from-group self-provisioner \
    system:authenticated:oauth
```

Para obtener más información, vea la documentación de OpenShift sobre la deshabilitación del aprovisionamiento automático para la versión del clúster:

- [Deshabilitación del aprovisionamiento automático en clústeres 4.3](https://docs.openshift.com/aro/4/applications/projects/configuring-project-creation.html#disabling-project-self-provisioning_configuring-project-creation)
- [Deshabilitación del aprovisionamiento automático en clústeres 3.11](https://docs.openshift.com/container-platform/3.11/admin_guide/managing_projects.html#disabling-self-provisioning)

### <a name="which-unix-rights-in-iaas-are-available-for-mastersinfraapp-nodes"></a>¿Qué derechos de UNIX (en IaaS) están disponibles para los nodos maestros, infraestructura o aplicaciones?

En el caso de los clústeres 4.x, el acceso al nodo está disponible a través del rol de administrador de clústeres. Para obtener más información, vea [Información general de RBAC](https://docs.openshift.com/container-platform/4.3/authentication/using-rbac.html).

En el caso de los clústeres 3.11, el acceso al nodo está prohibido.

### <a name="which-ocp-rights-do-we-have-cluster-admin-project-admin"></a>¿Qué derechos de OCP tenemos? ¿Administrador del clúster? ¿Administrador de proyectos?

En el caso de los clústeres 4.x, el rol de administrador del clúster está disponible. Para obtener más información, vea [Información general de RBAC](https://docs.openshift.com/container-platform/4.3/authentication/using-rbac.html).

En el caso de los clústeres 3.11, vea la [información general de administración del clúster](https://docs.openshift.com/aro/admin_guide/index.html) para obtener más detalles.

### <a name="which-identity-providers-are-available"></a>¿Qué proveedores de identidades están disponibles?

En el caso de los clústeres 4.x, va a configurar su propio proveedor de identidades. Para obtener más información, vea la documentación de Red Hat sobre [configuración de los proveedores de identidades](https://docs.openshift.com/aro/4/authentication/identity_providers/configuring-ldap-identity-provider.html).

En el caso de los clústeres 3.11, se puede usar la integración de Azure AD. 

## <a name="storage"></a>Storage

### <a name="is-data-on-my-cluster-encrypted"></a>¿Están cifrados los datos de mi clúster?

De forma predeterminada, los datos se cifran en reposo. La plataforma de Azure Storage cifra automáticamente los datos antes de almacenarlos y los descifra antes de recuperarlos. Para más información, consulte [Cifrado del servicio Azure Storage para datos en reposo](../storage/common/storage-service-encryption.md).

### <a name="is-data-stored-in-etcd-encrypted-on-azure-red-hat-openshift"></a>¿Los datos almacenados en etcd están cifrados en Red Hat OpenShift en Azure?

En el caso de los clústeres de Red Hat OpenShift en Azure 4, los datos no se cifran de forma predeterminada, pero tiene la opción de habilitar el cifrado. Para obtener más información, vea la guía sobre [cifrado de etcd](https://docs.openshift.com/container-platform/4.3/authentication/encrypting-etcd.html).

En el caso de los clústeres 3.11, los datos no se cifran en el nivel de etcd. Actualmente no se admite la opción para activar el cifrado. OpenShift es compatible con esta característica, pero se requieren labores de ingeniería para crearla en la hoja de ruta. Los datos se cifran en el nivel de disco. Consulte [Encrypting Data at Datastore Layer](https://docs.openshift.com/container-platform/3.11/admin_guide/encrypting_data.html) (Cifrado de datos en la capa de almacén de datos) para más información.

### <a name="can-we-choose-any-persistent-storage-solution-like-ocs"></a>¿Podemos elegir cualquier solución de almacenamiento persistente, como OCS? 

En el caso de los clústeres 4.x, Azure Disk (Premium_LRS) se configura como la clase de almacenamiento predeterminada. Para obtener más información sobre los proveedores de almacenamiento y los detalles de configuración (incluido Azure File), vea la documentación de Red Hat sobre [almacenamiento persistente](https://docs.openshift.com/aro/4/storage/understanding-persistent-storage.html).

En el caso de los clústeres 3.11, se proporcionan dos clases de almacenamiento de forma predeterminada: una para Azure Disk (Premium_LRS) y otra para Azure File.

## <a name="does-aro-store-any-customer-data-outside-of-the-clusters-region"></a>¿ARO guarda datos de los clientes fuera de la región del clúster?

No. Todos los datos que se crean en un clúster de ARO se mantienen en la región del clúster.