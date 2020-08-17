---
title: Creación de un clúster privado de Azure Kubernetes Service
description: Aprenda a crear un clúster privado de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 7/17/2020
ms.openlocfilehash: 10cbd58807c213418a88b42887cdb76868eac34e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015656"
---
# <a name="create-a-private-azure-kubernetes-service-cluster"></a>Creación de un clúster privado de Azure Kubernetes Service

En un clúster privado, el servidor de la API o el plano de control tienen direcciones IP internas que se definen en el documento [RFC1918 sobre la asignación de direcciones para conexiones privadas](https://tools.ietf.org/html/rfc1918). Mediante el uso de un clúster privado, puede asegurarse de que el tráfico entre el servidor de API y los grupos de nodos permanece solo en la red privada.

El plano de control o el servidor de la API están en una suscripción de Azure administrada mediante Azure Kubernetes Service (AKS). El grupo de clústeres o nodos de un cliente está en la suscripción del cliente. El servidor y el grupo de clústeres o nodos pueden comunicarse entre sí a través del [servicio de Azure Private Link][private-link-service] en la red virtual del servidor de la API y de un punto de conexión privado expuesto en la subred del clúster de AKS del cliente.

## <a name="region-availability"></a>Disponibilidad en regiones

El clúster privado está disponible en las regiones públicas en las que [se admite AKS](https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service).

* Azure China 21Vianet no se admite actualmente.
* Actualmente no se admite US Gov Texas debido a la falta de compatibilidad con Private Link.

## <a name="prerequisites"></a>Prerrequisitos

* CLI de Azure, versión 2.2.0 o cualquier versión posterior

## <a name="create-a-private-aks-cluster"></a>Creación de un clúster privado de AKS

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Cree un grupo de recursos o seleccione uno existente en el clúster de AKS.

```azurecli-interactive
az group create -l westus -n MyResourceGroup
```

### <a name="default-basic-networking"></a>Redes básicas predeterminadas 

```azurecli-interactive
az aks create -n <private-cluster-name> -g <private-cluster-resource-group> --load-balancer-sku standard --enable-private-cluster  
```
Donde *--enable-private-cluster* es una marca obligatoria para un clúster privado. 

### <a name="advanced-networking"></a>Redes avanzadas  

```azurecli-interactive
az aks create \
    --resource-group <private-cluster-resource-group> \
    --name <private-cluster-name> \
    --load-balancer-sku standard \
    --enable-private-cluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24 
```
Donde *--enable-private-cluster* es una marca obligatoria para un clúster privado. 

> [!NOTE]
> Si la dirección CIDR del puente de Docker (172.17.0.1/16) entra en conflicto con el CIDR de la subred, cambie la dirección del puente de Docker.

## <a name="options-for-connecting-to-the-private-cluster"></a>Opciones para conectarse al clúster privado

El punto de conexión del servidor de la API no tiene ninguna dirección IP pública. Para administrar el servidor de API, necesitará una máquina virtual que tenga acceso a la red virtual de Azure (VNet) del clúster de AKS. Hay varias opciones para establecer la conectividad de red con el clúster privado.

* Crear una máquina virtual en la misma red virtual de Azure (VNet) que el clúster de AKS.
* Usar una máquina virtual de una red diferente y configurar el [emparejamiento de red virtual][virtual-network-peering].  Consulte la sección siguiente para más información sobre esta opción.
* Usar una conexión de [ExpressRoute o VPN][express-route-or-VPN].

La opción más sencilla es crear una máquina virtual en la misma red virtual que el clúster de AKS.  ExpressRoute y las VPN incrementan los costos y requieren redes más complejas.  Para utilizar el emparejamiento de red virtual, debe planear los intervalos CIDR de la red para asegurarse de que no haya intervalos superpuestos.

## <a name="virtual-network-peering"></a>Emparejamiento de redes virtuales de Azure

Tal y como se ha dicho, el emparejamiento de red virtual es un mecanismo para acceder a un clúster privado. Si desea usar el emparejamiento de red virtual, tiene que configurar un vínculo entre la red virtual y la zona DNS privada.
    
1. Vaya al grupo de recursos del nodo en Azure Portal.  
2. Seleccione la zona DNS privada.   
3. En el panel izquierdo, seleccione el vínculo **red virtual**.  
4. Cree un nuevo vínculo para agregar la red virtual de la máquina virtual a la zona DNS privada. El vínculo de la zona DNS puede tardar unos minutos en estar disponible.  
5. En Azure Portal, vaya al grupo de recursos que contiene la red virtual del clúster.  
6. En el panel derecho, seleccione la red virtual. El nombre de la red virtual tiene el formato *aks-vnet-\** .  
7. En el panel izquierdo, seleccione **Emparejamientos**.  
8. Seleccione **Agregar**, agregue la red virtual de la máquina virtual y, después, cree el emparejamiento.  
9. Vaya a la red virtual en la que tiene la máquina virtual, seleccione **Emparejamientos**, seleccione la red virtual de AKS y, después, cree el emparejamiento. Si los intervalos de direcciones de la red virtual de AKS y de la red virtual de la máquina virtual entran en conflicto, se produce un error de emparejamiento. Para más información, vea el artículo [Emparejamiento de redes virtuales][virtual-network-peering].

## <a name="hub-and-spoke-with-custom-dns"></a>Concentrador y radio con DNS personalizado

[Las arquitecturas de concentrador y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke) suelen usarse para implementar redes en Azure. En muchas de estas implementaciones, los valores de DNS en las redes virtuales de radios están configurados para hacer referencia a un reenviador de DNS central con el fin de permitir la resolución de DNS local y basada en Azure. Al implementar un clúster de AKS en un entorno de red de este tipo, hay algunas consideraciones especiales que se deben tener en cuenta.

![Concentrador y radio de clúster privado](media/private-clusters/aks-private-hub-spoke.png)

1. De forma predeterminada, cuando se aprovisiona un clúster privado, se crean un punto de conexión privado (1) y una zona DNS privada (2) en el grupo de recursos administrados del clúster. El clúster usa un registro A en la zona privada a fin de resolver la dirección IP del punto de conexión privado para la comunicación con el servidor de la API.

2. La zona DNS privada solo está vinculada a la red virtual a la que están adjuntados los nodos del clúster (3). Esto significa que el punto de conexión privado solo lo pueden resolver los hosts de esa red virtual vinculada. En escenarios en los que no haya ningún DNS personalizado configurado en la red virtual (valor predeterminado), esto funciona sin incidencias, ya que los hosts apuntan a 168.63.129.16 para DNS, que puede resolver registros en la zona DNS privada debido al vínculo.

3. En escenarios en los que la red virtual que contiene el clúster tenga una configuración de DNS personalizada (4), se produce un error en la implementación del clúster a menos que la zona DNS privada esté vinculada a la red virtual que contiene las resoluciones de DNS personalizadas (5). Este vínculo se puede crear manualmente después de crear la zona privada, durante el aprovisionamiento del clúster, o a través de la automatización si se detecta la creación de la zona usando mecanismos de implementación basados en eventos (por ejemplo, Azure Event Grid y Azure Functions).

## <a name="dependencies"></a>Dependencias  

* El servicio Azure Private Link solo se admite en Standard Azure Load Balancer. No se admite en Basic Azure Load Balancer.  
* Para usar un servidor DNS personalizado, agregue la IP 168.63.129.16 de Azure DNS como servidor DNS ascendente en el servidor DNS personalizado.

## <a name="limitations"></a>Limitaciones 
* Los intervalos autorizados de direcciones IP no se pueden aplicar al punto de conexión del servidor de API privada. Solo se aplican al servidor de API pública.
* Las [zonas de disponibilidad][availability-zones] se admiten actualmente en determinadas regiones. 
* Las [limitaciones del servicio Azure Private Link][private-link-service] aplican a los clústeres privados.
* No se admiten agentes hospedados por Microsoft en Azure DevOps con clústeres privados. Considere la posibilidad de usar [agentes autohospedados][devops-agents]. 
* En el caso de los clientes que necesitan habilitar Azure Container Registry para trabajar con instancias privadas de AKS, la red virtual de Container Registry debe estar emparejada con la red virtual del clúster del agente.
* Actualmente no hay compatibilidad con Azure Dev Spaces.
* No se admite la conversión de clústeres de AKS existentes en clústeres privados.
* La eliminación o modificación del punto de conexión privado en la subred del cliente hará que el clúster deje de funcionar. 
* Actualmente no existe compatibilidad con los datos en directo de Azure Monitor para contenedores.
* El contrato de nivel de servicio de tiempo de actividad no se admite actualmente.


<!-- LINKS - internal -->
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register
[az-feature-list]: /cli/azure/feature?view=azure-cli-latest#az-feature-list
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[private-link-service]: ../private-link/private-link-service-overview.md#limitations
[virtual-network-peering]: ../virtual-network/virtual-network-peering-overview.md
[azure-bastion]: ../bastion/bastion-create-host-portal.md
[express-route-or-vpn]: ../expressroute/expressroute-about-virtual-network-gateways.md
[devops-agents]: /azure/devops/pipelines/agents/agents?view=azure-devops
[availability-zones]: availability-zones.md
