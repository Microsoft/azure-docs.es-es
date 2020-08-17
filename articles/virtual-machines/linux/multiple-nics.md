---
title: Creación de una máquina virtual Linux en Azure con varias NIC
description: Aprenda a crear una máquina virtual Linux con varias NIC conectadas a ella mediante la CLI de Azure o plantillas de Resource Manager.
author: cynthn
ms.service: virtual-machines-linux
ms.subservice: networking
ms.topic: how-to
ms.workload: infrastructure
ms.date: 06/07/2018
ms.author: cynthn
ms.openlocfilehash: 86910ece57d8fb72ade0c67a9e6787023c4283f3
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87836928"
---
# <a name="how-to-create-a-linux-virtual-machine-in-azure-with-multiple-network-interface-cards"></a>Cómo crear una máquina virtual Linux en Azure con red varias tarjetas de interfaz de red


En este artículo se describe cómo crear una máquina virtual con varias NIC con la CLI de Azure.

## <a name="create-supporting-resources"></a>Creación de recursos de apoyo
Instale la última versión de la [CLI de Azure](/cli/azure/install-az-cli2) e inicie sesión en una cuenta de Azure con [az login](/cli/azure/reference-index).

En los ejemplos siguientes, reemplace los nombres de parámetros de ejemplo por los suyos propios. Los nombres de parámetro de ejemplo incluyen *myResourceGroup*, *mystorageaccount* y *myVM*.

En primer lugar, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli
az group create --name myResourceGroup --location eastus
```

Cree la red virtual con [az network vnet create](/cli/azure/network/vnet). En el ejemplo siguiente se crea una red virtual denominada *myVnet* y una subred *mySubnetFrontEnd*:

```azurecli
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefix 10.0.0.0/16 \
    --subnet-name mySubnetFrontEnd \
    --subnet-prefix 10.0.1.0/24
```

Cree una subred para el tráfico de back-end con [az network vnet subnet create](/cli/azure/network/vnet/subnet). En el ejemplo siguiente se crea una subred denominada *mySubnetBackEnd*:

```azurecli
az network vnet subnet create \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnetBackEnd \
    --address-prefix 10.0.2.0/24
```

Cree un grupo de seguridad de red con [az network nsg create](/cli/azure/network/nsg). En el ejemplo siguiente se crea un grupo de seguridad de red denominado *myNetworkSecurityGroup*:

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --name myNetworkSecurityGroup
```

## <a name="create-and-configure-multiple-nics"></a>Creación y configuración de varias NIC
Creación de dos NIC con [az network nic create](/cli/azure/network/nic). En el ejemplo siguiente se crean dos NIC, denominadas *myNic1* y *myNic2*, conectadas al grupo de seguridad de red, con una NIC conectada con cada subred:

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic1 \
    --vnet-name myVnet \
    --subnet mySubnetFrontEnd \
    --network-security-group myNetworkSecurityGroup
az network nic create \
    --resource-group myResourceGroup \
    --name myNic2 \
    --vnet-name myVnet \
    --subnet mySubnetBackEnd \
    --network-security-group myNetworkSecurityGroup
```

## <a name="create-a-vm-and-attach-the-nics"></a>Creación de una máquina virtual y conexión de las NIC
Cuando cree la máquina virtual, especifique las NIC creadas con `--nics`. También debe tener cuidado al seleccionar el tamaño de la máquina virtual. Existen límites para el número total de NIC que se pueden agregar a una máquina virtual. Más información sobre los [tamaños de máquina virtual Linux](../sizes.md).

Cree la máquina virtual con [az vm create](/cli/azure/vm). En el ejemplo siguiente se crea una máquina virtual denominada *myVM*:

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --size Standard_DS3_v2 \
    --admin-username azureuser \
    --generate-ssh-keys \
    --nics myNic1 myNic2
```

Agregue tablas de enrutamiento al sistema operativo invitado completando los pasos descritos en [Cómo crear una máquina virtual Linux en Azure con red varias tarjetas de interfaz de red](#configure-guest-os-for-multiple-nics).

## <a name="add-a-nic-to-a-vm"></a>Adición de una NIC a una máquina virtual
Los pasos anteriores crean una máquina virtual con varias NIC. También puede agregar varias NIC a una máquina virtual existente con la CLI de Azure. Diferentes [tamaños de máquina virtual](../sizes.md) admiten un número distinto de NIC, así que ajuste el tamaño de su máquina virtual teniendo esto en cuenta. Si es necesario, puede [cambiar el tamaño de una máquina virtual](change-vm-size.md).

Cree otra NIC con [az network nic create](/cli/azure/network/nic). En el ejemplo siguiente se crea una NIC denominada *myNic3* conectada a la subred de back-end y al grupo de seguridad de red creado en los pasos anteriores:

```azurecli
az network nic create \
    --resource-group myResourceGroup \
    --name myNic3 \
    --vnet-name myVnet \
    --subnet mySubnetBackEnd \
    --network-security-group myNetworkSecurityGroup
```

Para agregar una NIC a una máquina virtual existente, en primer lugar desasigne la máquina virtual con [az vm deallocate](/cli/azure/vm). En el ejemplo siguiente se desasigna la máquina virtual denominada *myVM*:


```azurecli
az vm deallocate --resource-group myResourceGroup --name myVM
```

Agregue la NIC con [az vm nic add](/cli/azure/vm/nic). En el ejemplo siguiente se agrega *myNic3* a *myVM*:

```azurecli
az vm nic add \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --nics myNic3
```

Inicie la máquina virtual con [az vm start](/cli/azure/vm):

```azurecli
az vm start --resource-group myResourceGroup --name myVM
```

Agregue tablas de enrutamiento al sistema operativo invitado completando los pasos descritos en [Cómo crear una máquina virtual Linux en Azure con red varias tarjetas de interfaz de red](#configure-guest-os-for-multiple-nics).

## <a name="remove-a-nic-from-a-vm"></a>Eliminación de una NIC de una máquina virtual
Para quitar una NIC de una máquina virtual existente, en primer lugar desasigne la máquina virtual con [az vm deallocate](/cli/azure/vm). En el ejemplo siguiente se desasigna la máquina virtual denominada *myVM*:

```azurecli
az vm deallocate --resource-group myResourceGroup --name myVM
```

Quite la NIC con [az vm nic remove](/cli/azure/vm/nic). En el ejemplo siguiente se quita *myNic3* de *myVM*:

```azurecli
az vm nic remove \
    --resource-group myResourceGroup \
    --vm-name myVM \
    --nics myNic3
```

Inicie la máquina virtual con [az vm start](/cli/azure/vm):

```azurecli
az vm start --resource-group myResourceGroup --name myVM
```


## <a name="create-multiple-nics-using-resource-manager-templates"></a>Creación de varias NIC con plantillas de Resource Manager
Las plantillas de Azure Resource Manager emplean archivos JSON declarativos para definir el entorno. Puede leer la [introducción a Azure Resource Manager](../../azure-resource-manager/management/overview.md). Las plantillas de Resource Manager ofrecen una manera de crear varias instancias de un recurso durante la implementación; por ejemplo, se pueden crear varias NIC. Utilizará el comando *copy* para especificar el número de instancias que se crearán:

```json
"copy": {
    "name": "multiplenics"
    "count": "[parameters('count')]"
}
```

Más información sobre la [creación de varias instancias mediante *copia*](../../azure-resource-manager/templates/copy-resources.md). 

También puede utilizar `copyIndex()` para anexar un número a un nombre de recurso, lo que le permite crear `myNic1`, `myNic2`, etc. A continuación se muestra un ejemplo de cómo anexar el valor de índice:

```json
"name": "[concat('myNic', copyIndex())]", 
```

Puede leer un ejemplo completo de [cómo crear varias NIC con plantillas de Resource Manager](../../virtual-network/template-samples.md).

Agregue tablas de enrutamiento al sistema operativo invitado completando los pasos descritos en [Cómo crear una máquina virtual Linux en Azure con red varias tarjetas de interfaz de red](#configure-guest-os-for-multiple-nics).

## <a name="configure-guest-os-for-multiple-nics"></a>Configuración del sistema operativo invitado para varias NIC

En los pasos anteriores se creó una red virtual y una subred, se conectaron las NIC y luego se creó una máquina virtual. No se creó ninguna regla de grupos de seguridad de red ni ninguna dirección IP pública que permitieran el tráfico SSH. Para configurar al sistema operativo invitado para varias NIC, tiene que permitir conexiones remotas y ejecutar comandos localmente en la máquina virtual.

Para permitir el tráfico SSH, cree una regla de grupo de seguridad de red con [az network nsg rule create](/cli/azure/network/nsg/rule#az-network-nsg-rule-create) de la manera siguiente:

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name allow_ssh \
    --priority 101 \
    --destination-port-ranges 22
```

Cree una dirección IP pública con [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) y asígnela a la primera NIC con [az network nic ip-config update](/cli/azure/network/nic/ip-config#az-network-nic-ip-config-update):

```azurecli
az network public-ip create --resource-group myResourceGroup --name myPublicIP

az network nic ip-config update \
    --resource-group myResourceGroup \
    --nic-name myNic1 \
    --name ipconfig1 \
    --public-ip myPublicIP
```

Para ver la dirección IP pública de la máquina virtual, use [az vm show](/cli/azure/vm#az-vm-show) de la manera siguiente:

```azurecli
az vm show --resource-group myResourceGroup --name myVM -d --query publicIps -o tsv
```

Conéctese ahora por SSH a la dirección IP pública de la máquina virtual. El nombre de usuario predeterminado indicado en el paso anterior era *azureuser*. Indique su propio nombre de usuario y la dirección IP pública:

```bash
ssh azureuser@137.117.58.232
```

Para enviar hacia o desde una interfaz de red secundaria, tiene que agregar manualmente rutas persistentes al sistema operativo para cada interfaz de red secundaria. En este artículo, *eth1* es la interfaz secundaria. Las instrucciones para agregar rutas persistentes al sistema operativo varían según la distribución. Vea la documentación correspondiente a su distribución para obtener instrucciones.

Al agregar una ruta al sistema operativo, la dirección de puerta de enlace es *.1* para la subred en la que se encuentre la interfaz de red. Por ejemplo, si a la interfaz de red se le asigna la dirección *10.0.2.4*, la puerta de enlace que se especifica para la ruta es *10.0.2.1*. Si quiere que todo el tráfico de la interfaz pase por la puerta de enlace especificada, puede definir una red concreta para el destino de la ruta o especificar un destino de *0.0.0.0*. La puerta de enlace para cada subred se administra mediante la red virtual.

Cuando haya agregado la ruta para una interfaz secundaria, compruebe que la ruta se encuentra en la tabla de rutas con `route -n`. La siguiente salida de ejemplo corresponde a la tabla de rutas en la que se han agregado las dos interfaces de red para la máquina virtual en este artículo:

```bash
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.1.1        0.0.0.0         UG    0      0        0 eth0
0.0.0.0         10.0.2.1        0.0.0.0         UG    0      0        0 eth1
10.0.1.0        0.0.0.0         255.255.255.0   U     0      0        0 eth0
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 eth1
168.63.129.16   10.0.1.1        255.255.255.255 UGH   0      0        0 eth0
169.254.169.254 10.0.1.1        255.255.255.255 UGH   0      0        0 eth0
```

Confirme que la ruta agregada se conserva entre un reinicio y el siguiente comprobando de nuevo la tabla de rutas tras un reinicio. Para probar la conectividad, puede escribir el siguiente comando, por ejemplo, donde *eth1* es el nombre de una interfaz de red secundaria:

```bash
ping bing.com -c 4 -I eth1
```

## <a name="next-steps"></a>Pasos siguientes
Revise los [tamaños de máquina virtual Linux](../sizes.md) al intentar crear una máquina virtual con varias NIC. Preste atención al número máximo de NIC que admite cada tamaño de máquina virtual.

Para proteger más las máquinas virtuales, use acceso Just-In-Time a la máquina virtual. Esta característica abre reglas del grupo de seguridad de red al tráfico SSH cuando es necesario y durante un período de tiempo definido. Para más información, consulte [Administrar el acceso a máquina virtual mediante Just-In-Time](../../security-center/security-center-just-in-time.md).