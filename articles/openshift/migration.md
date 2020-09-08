---
title: Migración de Azure Red Hat OpenShift 3.11 a Azure Red Hat OpenShift 4
description: Migración de Azure Red Hat OpenShift 3.11 a Azure Red Hat OpenShift 4
author: sakthi-vetrivel
ms.author: suvetriv
ms.service: container-service
ms.topic: conceptual
ms.date: 08/13/2020
keywords: migración, aro, openshift, red hat
ms.openlocfilehash: 68e8be24273dfab490d2423b76c372f2ff9f2b38
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88513067"
---
# <a name="migrate-from-azure-red-hat-openshift-311-to-azure-red-hat-openshift-4"></a>Migración de Red Hat OpenShift en Azure 3.11 a Red Hat OpenShift en Azure 4

Red Hat OpenShift en Azure en OpenShift 4 incluye Kubernetes 1.16 en Red Hat Core OS, clústeres privados, compatibilidad con la opción de traer su propia red virtual y un rol de administrador de clúster completo. Además, hay disponibles muchas características nuevas, como la compatibilidad con Operator Framework, Operator Hub y OpenShift Service Mesh.

Para realizar correctamente la transición de Red Hat OpenShift en Azure 3.11 a Red Hat OpenShift en Azure 4, asegúrese de revisar las [diferencias en el almacenamiento, las redes, el registro, la seguridad y la supervisión](https://docs.openshift.com/container-platform/4.4/migration/migrating_3_4/planning-migration-3-to-4.html).

En este artículo, mostraremos cómo migrar de un clúster de Red Hat OpenShift en Azure 3.11 a un clúster de Red Hat en Azure 4.

> [!NOTE]
> Herramientas de migración de Red Hat OpenShift como la de asistencia a la migración del plano de control y la de migración de aplicaciones de clúster (CAM) no pueden usarse con los clústeres de Red Hat OpenShift en Azure 3.11.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se da por supuesto que tiene un clúster existente de la versión 3.11 de Red Hat OpenShift en Azure.

## <a name="create-a-target-azure-red-hat-openshift-4-cluster"></a>Creación de un clúster de Red Hat OpenShift en Azure 4 de destino

En primer lugar, [cree el clúster de Red Hat OpenShift en Azure 4](tutorial-create-cluster.md) que quiere usar como clúster de destino. Aquí usaremos la configuración básica. Si está interesado en una configuración diferente, consulte el [tutorial Creación de un clúster de la versión 4 de Red Hat OpenShift en Azure](tutorial-create-cluster.md).

Cree una red virtual con dos subredes vacías para los nodos maestro y de trabajo.

```azurecli-interactive
    az network vnet create \
    --resource-group $RESOURCEGROUP \
    --name aro-vnet \
    --address-prefixes 10.0.0.0/22

    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name master-subnet \
    --address-prefixes 10.0.0.0/23 \
    --service-endpoints Microsoft.ContainerRegistry

    az network vnet subnet create \
    --resource-group $RESOURCEGROUP \
    --vnet-name aro-vnet \
    --name worker-subnet \
    --address-prefixes 10.0.2.0/23 \
    --service-endpoints Microsoft.ContainerRegistry
```

A continuación, use el comando siguiente para crear el clúster.

```azurecli-interactive
az aro create \
  --resource-group $RESOURCEGROUP \
  --name $CLUSTER \
  --vnet aro-vnet \
  --master-subnet master-subnet \
  --worker-subnet worker-subnet \
  # --domain foo.example.com # [OPTIONAL] custom domain
  # --pull-secret @pull-secret.txt # [OPTIONAL]
```

## <a name="configure-the-target-openshift-4-cluster"></a>Configuración del clúster OpenShift 4 de destino

### <a name="authentication"></a>Authentication

Para que los usuarios interactúen con Red Hat OpenShift en Azure, primero deben autenticarse en el clúster. El nivel de autenticación identifica al usuario asociado a las solicitudes a la API de Red Hat OpenShift en Azure. A continuación, el nivel de autorización usa información sobre el usuario solicitante para determinar si se permite la solicitud.

Al crearse un clúster de Red Hat OpenShift en Azure 4, se crea un usuario administrativo temporal. [Conéctese al clúster](tutorial-connect-cluster.md), agregue usuarios y grupos, y [configure los permisos adecuados](https://docs.openshift.com/aro/4/authentication/understanding-authentication.html) para ambos.

### <a name="networking"></a>Redes

Red Hat OpenShift en Azure 4 usa algunos operadores diferentes para configurar la red en el clúster: el [operador de red de clústeres](https://docs.openshift.com/aro/4/networking/cluster-network-operator.html#nw-cluster-network-operator_cluster-network-operator), el [operador DNS](https://docs.openshift.com/aro/4/networking/dns-operator.html) y el [operador de entrada](https://docs.openshift.com/aro/4/networking/ingress-operator.html). Para obtener más información sobre cómo configurar las redes en un clúster de Red Hat OpenShift en Azure 4, consulte el [diagrama de redes](concepts-networking.md) y la [descripción de las mismas](https://docs.openshift.com/aro/4/networking/understanding-networking.html).

### <a name="storage"></a>Storage
Red Hat OpenShift en Azure 4 admite los siguientes complementos de PersistentVolume:

- Elastic Block Store (EBS) de AWS
- Azure Disk
- Archivo de Azure
- Disco persistente del GCE
- HostPath
- iSCSI
- Volumen local
- NFS
- Red Hat OpenShift Container Storage

Para obtener información sobre la configuración de estos tipos de almacenamiento, consulte [Configuración de almacenamiento persistente](https://access.redhat.com/documentation/azure_red_hat_openshift/4/html/storage/configuring-persistent-storage).

### <a name="registry"></a>Registro

Red Hat OpenShift en Azure 4 puede compilar imágenes a partir del código fuente, implementarlas y administrar su ciclo de vida. Para habilitar esto, Red Hat OpenShift en Azure 4 proporciona un [registro de imágenes de contenedor integrado e interno](https://docs.openshift.com/aro/4/registry/registry-options.html) que se puede implementar en su entorno de Red Hat OpenShift en Azure para administrar las imágenes de forma local.

Si usa registros externos como [Azure Container Registry](https://docs.microsoft.com/azure/container-registry/), [registros de Red Hat Quay](https://docs.openshift.com/aro/4/registry/registry-options.html#registry-quay-overview_registry-options) o un [registro de Red Hat habilitado para la autenticación](https://docs.openshift.com/aro/4/registry/registry-options.html#registry-authentication-enabled-registry-overview_registry-options), siga los pasos para proporcionar credenciales al clúster con el fin de que este tenga acceso a los repositorios.

### <a name="monitoring"></a>Supervisión

Red Hat OpenShift en Azure incluye una pila de supervisión preconfigurada, preinstalada y de auto-actualización que se basa en el proyecto de código abierto Prometheus y su ecosistema de mayor amplitud. Proporciona la supervisión de componentes del clúster e incluye un conjunto de alertas para notificar inmediatamente al administrador de clústeres de cualquier problema que pueda producirse y un conjunto de paneles de Grafana. La pila de supervisión de clústeres solo se admite para la supervisión de clústeres de Red Hat OpenShift en Azure. Para obtener más información, consulte la [supervisión de clústeres para Red Hat OpenShift en Azure](https://docs.openshift.com/aro/4/monitoring/cluster_monitoring/about-cluster-monitoring.html).

Si ha usado [Azure Monitor para contenedores para Red Hat OpenShift en Azure 3.11](../azure-monitor/insights/container-insights-azure-redhat-setup.md), también puede habilitar Azure Monitor para contenedores para [Red Hat OpenShift en Azure 4](../azure-monitor/insights/container-insights-azure-redhat4-setup.md) y seguir utilizando la misma área de trabajo de Log Analytics.

## <a name="move-your-dns-or-load-balancer-configuration-to-the-new-cluster"></a>Traslado de la configuración del equilibrador de carga o DNS al nuevo clúster

Si usa Azure Traffic Manager, agregue puntos de conexión para hacer referencia al clúster de destino y priorice dichos puntos.

## <a name="deploy-application-to-your-target-cluster"></a>Implementación de una aplicación en el clúster de destino

Una vez que tenga configurado correctamente el clúster de destino para la carga de trabajo, [conéctese al clúster](tutorial-connect-cluster.md) y cree las aplicaciones, los componentes o los servicios necesarios para los proyectos. Red Hat OpenShift en Azure permite crearlos a partir de Git, imágenes de contenedor, el catálogo de desarrolladores de Red Hat, un Dockerfile, una definición de YAML/JSON, o bien seleccionando un servicio de base de datos del catálogo.

## <a name="delete-your-source-cluster"></a>Eliminación del clúster de origen
Una vez que haya confirmado que el clúster de Red Hat OpenShift en Azure 4 está configurado correctamente, elimine el clúster de Red Hat OpenShift en Azure 3.11.

```
az openshift delete --name $CLUSTER_NAME
                    --resource-group $RESOURCE_GROUP
                    [--no-wait]
                    [--subscription]
                    [--yes]
```
## <a name="next-steps"></a>Pasos siguientes
Consulte la documentación de Red Hat OpenShift en Azure que proporciona Red Hat [aquí](https://docs.openshift.com/aro/4/welcome/index.html).

