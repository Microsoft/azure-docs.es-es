---
title: Configuración de una dirección IP estática de salida
description: Configure Azure Firewall y las rutas definidas por el usuario para las cargas de trabajo de Azure Container Instances que utilizan la dirección IP pública del firewall para la entrada y salida
ms.topic: article
ms.date: 07/16/2020
ms.openlocfilehash: 497645b9fe7f908cc9b8b4d7ed0ba5e201570160
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89566575"
---
# <a name="configure-a-single-public-ip-address-for-outbound-and-inbound-traffic-to-a-container-group"></a>Configuración de una única dirección IP pública para el tráfico entrante y saliente a un grupo de contenedores

La configuración de un [grupo de contenedores](container-instances-container-groups.md) con una dirección IP externa permite a los clientes externos usar la dirección IP para tener acceso a un contenedor del grupo. Por ejemplo, un explorador puede acceder a una aplicación web que se ejecute en un contenedor. Sin embargo, actualmente un grupo de contenedores usa una dirección IP diferente para el tráfico saliente. Esta dirección IP de salida no se expone mediante programación, lo que hace que la supervisión del grupo de contenedores y la configuración de las reglas del firewall cliente sean más complejas.

En este artículo encontrará los pasos para configurar un grupo de contenedores en una [red virtual](container-instances-virtual-network-concepts.md) integrada en [Azure Firewall](../firewall/overview.md). Al configurar una ruta definida por el usuario para el grupo de contenedores y las reglas de firewall, puede enrutar e identificar el tráfico hacia y desde el grupo de contenedores. La entrada y la salida del grupo de contenedores usan la dirección IP pública del firewall. Una única dirección IP de salida puede usarse en varios grupos de contenedores implementados en la subred delegada en Azure Container Instances de dicha red virtual.

En este artículo se usa la CLI de Azure para crear los recursos para este escenario:

* Grupos de contenedores implementados en una subred delegada [en la red virtual](container-instances-vnet.md) 
* Una instancia de Azure Firewall implementada en la red con una dirección IP pública estática
* Una ruta definida por el usuario en la subred de los grupos de contenedores
* Una regla NAT para la entrada de firewall y una regla de aplicación para la salida

A continuación, validará la entrada y salida de los grupos de contenedores de ejemplo a través del firewall.

## <a name="deploy-aci-in-a-virtual-network"></a>Implementación de ACI en una red virtual

En un caso típico, es posible que ya tenga una instancia de Azure Virtual Network donde implementar un grupo de contenedores. Para fines de demostración, los siguientes comandos crean una red virtual y una subred cuando se crea el grupo de contenedores. La subred se delega en Azure Container Instances. 

El grupo de contenedores ejecuta una pequeña aplicación web desde la imagen `aci-helloworld`. Como se muestra en otros artículos de la documentación, esta imagen empaqueta una pequeña aplicación web escrita en Node.js que sirve una página HTML estática.

Si necesita un grupo de recursos de Azure, cree uno con el comando [az group create][az-group-create]. Por ejemplo:

```azurecli
az group create --name myResourceGroup --location eastus
```

Para simplificar los siguientes ejemplos de comandos, use una variable de entorno para el nombre del grupo de recursos:

```console
export RESOURCE_GROUP_NAME=myResourceGroup
```

Cree el grupo de contenedores con el comando [az container create][az-container-create]:

```azurecli
az container create \
  --name appcontainer \
  --resource-group $RESOURCE_GROUP_NAME \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --vnet aci-vnet \
  --vnet-address-prefix 10.0.0.0/16 \
  --subnet aci-subnet \
  --subnet-address-prefix 10.0.0.0/24
```

> [!TIP]
> Ajuste el valor de `--subnet address-prefix` para el espacio de direcciones IP que necesita en la subred. La subred compatible más pequeña es /29, que proporciona ocho direcciones IP. Ciertas direcciones IP están reservadas para uso por parte de Azure.

Para su uso en un paso posterior, obtenga la dirección IP privada del grupo de contenedores con el comando [az container show][az-container-show]:

```azurecli
ACI_PRIVATE_IP="$(az container show --name appcontainer \
  --resource-group $RESOURCE_GROUP_NAME \
  --query ipAddress.ip --output tsv)"
```

## <a name="deploy-azure-firewall-in-network"></a>Implementación de Azure Firewall en la red

En las secciones siguientes, use la CLI de Azure para implementar una instancia de Azure Firewall en la red virtual. Para obtener información general, consulte [Tutorial: Implementación y configuración de Azure Firewall mediante Azure Portal](../firewall/deploy-cli.md).

En primer lugar, use [az network vnet subnet create][az-network-vnet-subnet-create] para agregar una subred denominada AzureFirewallSubnet para el firewall. AzureFirewallSubnet es el nombre *obligatorio* de esta subred.

```azurecli
az network vnet subnet create \
  --name AzureFirewallSubnet \
  --resource-group $RESOURCE_GROUP_NAME \
  --vnet-name aci-vnet   \
  --address-prefix 10.0.1.0/26
```

Use los siguientes [comandos de la CLI de Azure](../firewall/deploy-cli.md) para crear un firewall en la subred.

Si aún no está instalada, agregue la extensión de firewall a la CLI de Azure mediante el comando [az extension add][az-extension-add]:

```azurecli
az extension add --name azure-firewall
```

Cree los recursos de firewall:

```azurecli
az network firewall create \
  --name myFirewall \
  --resource-group $RESOURCE_GROUP_NAME \
  --location eastus

az network public-ip create \
  --name fw-pip \
  --resource-group $RESOURCE_GROUP_NAME \
  --location eastus \
  --allocation-method static \
  --sku standard
    
az network firewall ip-config create \
  --firewall-name myFirewall \
  --name FW-config \
  --public-ip-address fw-pip \
  --resource-group $RESOURCE_GROUP_NAME \
  --vnet-name aci-vnet
```

Actualice la configuración del firewall mediante el comando [az network firewall update][az-network-firewall-update]:

```azurecli
az network firewall update \
  --name myFirewall \
  --resource-group $RESOURCE_GROUP_NAME
```

Para obtener la dirección IP privada del firewall, use el comando [az network firewall ip-config list][az-network-firewall-ip-config-list]. Esta dirección IP privada se usa en un comando posterior.


```azurecli
FW_PRIVATE_IP="$(az network firewall ip-config list \
  --resource-group $RESOURCE_GROUP_NAME \
  --firewall-name myFirewall \
  --query "[].privateIpAddress" --output tsv)"
```
Para obtener la dirección IP pública del firewall, use el comando [az network public-ip show][az-network-public-ip-show]. Esta dirección IP pública se usa en un comando posterior.

```azurecli
FW_PUBLIC_IP="$(az network public-ip show \
  --name fw-pip \
  --resource-group $RESOURCE_GROUP_NAME \
  --query ipAddress --output tsv)"
```

## <a name="define-user-defined-route-on-aci-subnet"></a>Definición de la ruta definida por el usuario en la subred de ACI

Defina una ruta definida por el uso en la subred de ACI para desviar el tráfico a Azure Firewall. Para obtener más información, consulte [Enrutamiento del tráfico de red](../virtual-network/tutorial-create-route-table-cli.md). 

### <a name="create-route-table"></a>Creación de una tabla de rutas

En primer lugar, ejecute el comando siguiente [az network route-table create][az-network-route-table-create] para crear la tabla de rutas. Cree la tabla de rutas en la misma región que la red virtual.

```azurecli
az network route-table create \
  --name Firewall-rt-table \
  --resource-group $RESOURCE_GROUP_NAME \
  --location eastus \
  --disable-bgp-route-propagation true
```

### <a name="create-route"></a>Creación de una ruta

Ejecute [az network route-table route create][az-network-route-table-route-create] para crear una ruta en la tabla de rutas. Para enrutar el tráfico al firewall, establezca el tipo de próximo salto en `VirtualAppliance` y pase la dirección IP privada del firewall como dirección del próximo salto.

```azurecli
az network route-table route create \
  --resource-group $RESOURCE_GROUP_NAME \
  --name DG-Route \
  --route-table-name Firewall-rt-table \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $FW_PRIVATE_IP
```

### <a name="associate-route-table-to-aci-subnet"></a>Asociación de una tabla de rutas a una subred de ACI

Ejecute el comando [az network vnet subnet update][az-network-vnet-subnet-update] para asociar la tabla de rutas con la subred delegada en Azure Container Instances.

```azurecli
az network vnet subnet update \
  --name aci-subnet \
  --resource-group $RESOURCE_GROUP_NAME \
  --vnet-name aci-vnet \
  --address-prefixes 10.0.0.0/24 \
  --route-table Firewall-rt-table
```

## <a name="configure-rules-on-firewall"></a>Configuración de reglas en el firewall

De forma predeterminada, Azure Firewall deniega (bloquea) el tráfico entrante y saliente. 

### <a name="configure-nat-rule-on-firewall-to-aci-subnet"></a>Configuración de la regla NAT en el firewall a la subred de ACI

Cree una [regla NAT](../firewall/rule-processing.md) en el firewall para traducir y filtrar el tráfico entrante de Internet al contenedor de la aplicación que inició anteriormente en la red. Para obtener más detalles, consulte [Filtrado del tráfico de entrada de Internet con la DNAT de Azure Firewall](../firewall/tutorial-firewall-dnat.md).

Cree una regla NAT y una colección mediante el comando [az network firewall nat-rule create][az-network-firewall-nat-rule-create]:

```azurecli
az network firewall nat-rule create \
  --firewall-name myFirewall \
  --collection-name myNATCollection \
  --action dnat \
  --name myRule \
  --protocols TCP \
  --source-addresses '*' \
  --destination-addresses $FW_PUBLIC_IP \
  --destination-ports 80 \
  --resource-group $RESOURCE_GROUP_NAME \
  --translated-address $ACI_PRIVATE_IP \
  --translated-port 80 \
  --priority 200
```

Agregue reglas NAT según sea necesario para filtrar el tráfico a otras direcciones IP de la subred. Por ejemplo, otros grupos de contenedores de la subred podrían exponer direcciones IP para el tráfico entrante, o bien se podrían asignar otras direcciones IP internas al grupo de contenedores después de un reinicio.

### <a name="create-outbound-application-rule-on-the-firewall"></a>Creación de una regla de aplicación de salida en el firewall

Ejecute el siguiente comando [az network firewall application-rule create][az-network-firewall-application-rule-create] para crear una regla de salida en el firewall. Esta regla de ejemplo permite el acceso desde la subred delegada en Azure Container Instances al FQDN `checkip.dyndns.org`. El acceso HTTP al sitio se usa en un paso posterior para confirmar la dirección IP de salida desde Azure Container Instances.

```azurecli
az network firewall application-rule create \
  --collection-name myAppCollection \
  --firewall-name myFirewall \
  --name Allow-CheckIP \
  --protocols Http=80 Https=443 \
  --resource-group $RESOURCE_GROUP_NAME \
  --target-fqdns checkip.dyndns.org \
  --source-addresses 10.0.0.0/24 \
  --priority 200 \
  --action Allow
```

## <a name="test-container-group-access-through-the-firewall"></a>Prueba del acceso del grupo de contenedores a través del firewall

En las secciones siguientes se verifica si la subred delegada en Azure Container Instances está configurada correctamente detrás de la instancia de Azure Firewall. En los pasos anteriores se enrutó el tráfico entrante a la subred y el tráfico saliente desde la subred a través del firewall.

### <a name="test-ingress-to-a-container-group"></a>Prueba de la entrada a un grupo de contenedores

Para probar el acceso de entrada al *appcontainer* que se ejecuta en la red virtual, vaya a la dirección IP pública del firewall. Anteriormente, almacenó la dirección IP pública en la variable $FW_PUBLIC_IP:

```bash
echo $FW_PUBLIC_IP
```

La salida es parecida a esta:

```console
52.142.18.133
```

Si la regla NAT en el firewall está configurada correctamente, verá lo siguiente al escribir la dirección IP pública del firewall en el explorador:

:::image type="content" source="media/container-instances-egress-ip-address/aci-ingress-ip-address.png" alt-text="Navegación a la dirección IP pública del firewall":::

### <a name="test-egress-from-a-container-group"></a>Prueba de la salida desde un grupo de contenedores


Implemente el siguiente contenedor de ejemplo en la red virtual. Cuando se ejecuta, envía una única solicitud HTTP a `http://checkip.dyndns.org`, que muestra la dirección IP del remitente (la dirección IP de salida). Si la regla de aplicación en el firewall está configurada correctamente, se devuelve la dirección IP pública del firewall.

```azurecli
az container create \
  --resource-group $RESOURCE_GROUP_NAME \
  --name testegress \
  --image mcr.microsoft.com/azuredocs/aci-tutorial-sidecar \
  --command-line "curl -s http://checkip.dyndns.org" \
  --restart-policy OnFailure \
  --vnet aci-vnet \
  --subnet aci-subnet
```

Vea los registros de contenedor para confirmar que la dirección IP sea la misma que la dirección IP pública del firewall.

```azurecli
az container logs \
  --resource-group $RESOURCE_GROUP_NAME \
  --name testegress 
```

La salida es parecida a esta:

```console
<html><head><title>Current IP Check</title></head><body>Current IP Address: 52.142.18.133</body></html>
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, configuró grupos de contenedores en una red virtual detrás de una instancia de Azure Firewall. Configuró una ruta definida por el usuario y reglas NAT y de aplicación en el firewall. Mediante esta configuración, configuró una única dirección IP estática para la entrada y salida desde Azure Container Instances.

Para obtener más información sobre cómo administrar el tráfico y proteger los recursos de Azure, consulte la documentación de [Azure Firewall](../firewall/index.yml).



[az-group-create]: /cli/azure/group#az-group-create
[az-container-create]: /cli/azure/container#az-container-create
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-create
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-network-firewall-update]: /cli/azure/ext/azure-firewall/network/firewall#ext-azure-firewall-az-network-firewall-update
[az-network-public-ip-show]: /cli/azure/network/public-ip/#az-network-public-ip-show
[az-network-route-table-create]:/cli/azure/network/route-table/#az-network-route-table-create
[az-network-route-table-route-create]: /cli/azure/network/route-table/route#az-network-route-table-route-create
[az-network-firewall-ip-config-list]: /cli/azure/ext/azure-firewall/network/firewall/ip-config#ext-azure-firewall-az-network-firewall-ip-config-list
[az-network-vnet-subnet-update]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-update
[az-container-exec]: /cli/azure/container#az-container-exec
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-vm-open-port]: /cli/azure/vm#az-vm-open-port
[az-vm-list-ip-addresses]: /cli/azure/vm#az-vm-list-ip-addresses
[az-network-firewall-application-rule-create]: /cli/azure/ext/azure-firewall/network/firewall/application-rule#ext-azure-firewall-az-network-firewall-application-rule-create
[az-network-firewall-nat-rule-create]: /cli/azure/ext/azure-firewall/network/firewall/nat-rule#ext-azure-firewall-az-network-firewall-nat-rule-create






