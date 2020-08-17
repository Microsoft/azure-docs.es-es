---
title: Creación de una máquina virtual Linux con zonas con la CLI de Azure
description: Creación de una máquina virtual Linux en una zona de disponibilidad con la CLI de Azure
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 04/05/2018
ms.author: cynthn
ms.openlocfilehash: 964d2b5d89fd91aca68e9b47b0049529fe284848
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87829006"
---
# <a name="create-a-linux-virtual-machine-in-an-availability-zone-with-the-azure-cli"></a>Creación de una máquina virtual Linux en una zona de disponibilidad con la CLI de Azure

En este artículo se describe el uso de la CLI de Azure para crear una máquina virtual Linux en una zona de disponibilidad de Azure. Una [zona de disponibilidad](../../availability-zones/az-overview.md) es una zona separada físicamente en una región de Azure. Use zonas de disponibilidad para proteger sus datos y aplicaciones de la improbable pérdida o error de todo un centro de datos.

Para usar una zona de disponibilidad, cree la máquina virtual [en una región de Azure compatible](../../availability-zones/az-region.md).

Asegúrese de que ha instalado la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) y que ha iniciado sesión en una cuenta de Azure con [az login](/cli/azure/reference-index).


## <a name="check-vm-sku-availability"></a>Comprobación de la disponibilidad del SKU de la máquina virtual
La disponibilidad de tamaños de máquinas virtuales o SKU puede variar según la región y la zona. Para ayudarle a planear el uso de las Zonas de disponibilidad, puede enumerar las SKU de la máquina virtual disponibles por región de Azure y zona. Con esta funcionalidad se asegura de elegir un tamaño apropiado de máquina virtual y obtener la resistencia deseada en las distintas zonas. Para más información sobre los diferentes tipos y tamaños de máquina virtual, consulte [Introducción a los tamaños de máquina virtual](../sizes.md).

Puede ver las SKU de máquina virtual disponibles con el comando [az vm list-skus](/cli/azure/vm). En el ejemplo siguiente se enumeran las SKU de máquina virtual disponibles en la región *eastus2*:

```azurecli
az vm list-skus --location eastus2 --output table
```

La salida es similar al siguiente ejemplo reducido, en el que se muestran las Zonas de disponibilidad en las que está disponible cada tamaño de máquina virtual:

```output
ResourceType      Locations  Name               [...]    Tier       Size     Zones
----------------  ---------  -----------------           ---------  -------  -------
virtualMachines   eastus2    Standard_DS1_v2             Standard   DS1_v2   1,2,3
virtualMachines   eastus2    Standard_DS2_v2             Standard   DS2_v2   1,2,3
[...]
virtualMachines   eastus2    Standard_F1s                Standard   F1s      1,2,3
virtualMachines   eastus2    Standard_F2s                Standard   F2s      1,2,3
[...]
virtualMachines   eastus2    Standard_D2s_v3             Standard   D2_v3    1,2,3
virtualMachines   eastus2    Standard_D4s_v3             Standard   D4_v3    1,2,3
[...]
virtualMachines   eastus2    Standard_E2_v3              Standard   E2_v3    1,2,3
virtualMachines   eastus2    Standard_E4_v3              Standard   E4_v3    1,2,3
```


## <a name="create-resource-group"></a>Creación de un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group).  

Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Se debe crear un grupo de recursos antes de una máquina virtual. En este ejemplo, se crea un grupo de recursos denominado *myResourceGroupVM* en la región *eastus2*. La región Este de EE. UU. 2 es una de las regiones de Azure que admite zonas de disponibilidad.

```azurecli 
az group create --name myResourceGroupVM --location eastus2
```

El grupo de recursos se especifica al crear o modificar una máquina virtual, lo que se puede a lo largo de este artículo.

## <a name="create-virtual-machine"></a>Crear máquina virtual

Cree la máquina virtual con el comando [az vm create](/cli/azure/vm). 

Al crear una máquina virtual, están disponibles varias opciones, como la imagen de sistema operativo, tamaño de disco y credenciales administrativas. En este ejemplo, se crea una máquina virtual llamada *myVM* que se ejecuta en Ubuntu. La máquina virtual se crea en la zona de disponibilidad *1*. De forma predeterminada, la máquina virtual se crea con el tamaño *Standard_DS1_v2*.

```azurecli-interactive
az vm create --resource-group myResourceGroupVM --name myVM --location eastus2 --image UbuntuLTS --generate-ssh-keys --zone 1
```

La creación de la máquina virtual puede tardar unos minutos. Una vez creada la máquina virtual, la CLI de Azure ofrece como salida información sobre la máquina virtual. Tome nota del valor de `zones`, que indica la zona de disponibilidad en la que se ejecuta la máquina virtual. 

```output
{
  "fqdns": "",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus2",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "52.174.34.95",
  "resourceGroup": "myResourceGroupVM",
  "zones": "1"
}
```

## <a name="confirm-zone-for-managed-disk-and-ip-address"></a>Confirmación de zona para disco administrado y dirección IP

Cuando la máquina virtual se implementa en una zona de disponibilidad, se crea un disco administrado para la máquina virtual en la misma zona de disponibilidad. De forma predeterminada, también se crea una dirección IP pública en dicha zona. En los ejemplos siguientes se obtiene información acerca de estos recursos.

Para comprobar que el disco administrado de la máquina virtual está en la zona de disponibilidad, utilice el comando [az vm show](/cli/azure/vm) para devolver el identificador de disco. En este ejemplo, el identificador de disco se almacena en una variable que se usa en un paso posterior. 

```azurecli-interactive
osdiskname=$(az vm show -g myResourceGroupVM -n myVM --query "storageProfile.osDisk.name" -o tsv)
```
Ahora puede obtener información sobre el disco administrado:

```azurecli-interactive
az disk show --resource-group myResourceGroupVM --name $osdiskname
```

La salida muestra que el disco administrado está en la misma zona de disponibilidad que la máquina virtual:

```output
{
  "creationData": {
    "createOption": "FromImage",
    "imageReference": {
      "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westeurope/Publishers/Canonical/ArtifactTypes/VMImage/Offers/UbuntuServer/Skus/16.04-LTS/Versions/latest",
      "lun": null
    },
    "sourceResourceId": null,
    "sourceUri": null,
    "storageAccountId": null
  },
  "diskSizeGb": 30,
  "encryptionSettings": null,
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/disks/osdisk_761c570dab",
  "location": "eastus2",
  "managedBy": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Compute/virtualMachines/myVM",
  "name": "myVM_osdisk_761c570dab",
  "osType": "Linux",
  "provisioningState": "Succeeded",
  "resourceGroup": "myResourceGroupVM",
  "sku": {
    "name": "Premium_LRS",
    "tier": "Premium"
  },
  "tags": {},
  "timeCreated": "2018-03-05T22:16:06.892752+00:00",
  "type": "Microsoft.Compute/disks",
  "zones": [
    "1"
  ]
}
```

Use el comando [az vm list-ip-addresses](/cli/azure/vm) para devolver el nombre del recurso de dirección IP pública en *myVM*. En este ejemplo, el nombre se almacena en una variable que se usa en un paso posterior.

```azurecli
ipaddressname=$(az vm list-ip-addresses -g myResourceGroupVM -n myVM --query "[].virtualMachine.network.publicIpAddresses[].name" -o tsv)
```

Ahora puede obtener información sobre la dirección IP:

```azurecli
az network public-ip show --resource-group myResourceGroupVM --name $ipaddressname
```

El resultado muestra que la dirección IP está en la misma zona de disponibilidad que la máquina virtual:

```output
{
  "dnsSettings": null,
  "etag": "W/\"b7ad25eb-3191-4c8f-9cec-c5e4a3a37d35\"",
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Network/publicIPAddresses/myVMPublicIP",
  "idleTimeoutInMinutes": 4,
  "ipAddress": "52.174.34.95",
  "ipConfiguration": {
    "etag": null,
    "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroupVM/providers/Microsoft.Network/networkInterfaces/myVMVMNic/ipConfigurations/ipconfigmyVM",
    "name": null,
    "privateIpAddress": null,
    "privateIpAllocationMethod": null,
    "provisioningState": null,
    "publicIpAddress": null,
    "resourceGroup": "myResourceGroupVM",
    "subnet": null
  },
  "location": "eastUS2",
  "name": "myVMPublicIP",
  "provisioningState": "Succeeded",
  "publicIpAddressVersion": "IPv4",
  "publicIpAllocationMethod": "Dynamic",
  "resourceGroup": "myResourceGroupVM",
  "resourceGuid": "8c70a073-09be-4504-0000-000000000000",
  "tags": {},
  "type": "Microsoft.Network/publicIPAddresses",
  "zones": [
    "1"
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear una máquina virtual en una zona de disponibilidad. Aprenda más sobre la [disponibilidad](../availability.md) de las máquinas virtuales de Azure.