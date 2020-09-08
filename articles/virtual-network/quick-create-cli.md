---
title: 'Inicio rápido: Creación de una red virtual mediante la CLI de Azure'
titlesuffix: Azure Virtual Network
description: En esta guía de inicio rápido, aprenda a crear una red virtual mediante la CLI de Azure. Una red virtual permite que los recursos de Azure se comuniquen entre sí y con Internet.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
Customer intent: I want to create a virtual network so that virtual machines can communicate with privately with each other and with the internet.
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: quickstart
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 01/22/2019
ms.author: kumud
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1feae201738a560c4cdb56f703c4af9a38af86d1
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056795"
---
# <a name="quickstart-create-a-virtual-network-using-the-azure-cli"></a>Inicio rápido: Creación de una red virtual mediante la CLI de Azure

Una red virtual permite que los recursos de Azure, como las máquinas virtuales, se comuniquen de manera privada entre sí y con Internet. En esta guía de inicio rápido aprenderá a crear una red virtual. Después de crear una red virtual, implementará dos máquinas virtuales en la red virtual. Luego, conéctese a las máquinas virtuales desde Internet y, de manera privada, comuníquese a través de la red virtual nueva.
## <a name="prerequisites"></a>Requisitos previos
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) ahora.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si, en su lugar, decide instalar y usar la CLI de Azure en un entorno local, para esta guía de inicio rápido se necesita la versión 2.0.28 de la CLI de Azure o una versión posterior. Ejecute `az --version` para buscar la versión instalada. Consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) para obtener información sobre la instalación o actualización.

## <a name="create-a-resource-group-and-a-virtual-network"></a>Creación de un grupo de recursos y una red virtual

Para poder crear una red virtual, debe crear un grupo de recursos que hospede la red virtual. Cree un grupo de recursos con [az group create](/cli/azure/group). En este ejemplo, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Cree la red virtual con el comando [az network vnet create](/cli/azure/network/vnet). En este ejemplo se crea una red virtual predeterminada denominada *myVirtualNetwork* con una subred denominada *default*:

```azurecli-interactive
az network vnet create \
  --name myVirtualNetwork \
  --resource-group myResourceGroup \
  --subnet-name default
```

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree dos máquinas virtuales en la red virtual.

### <a name="create-the-first-vm"></a>Creación de la primera máquina virtual

Cree la máquina virtual con [az vm create](/cli/azure/vm). Si las claves SSH no existen en la ubicación de claves predeterminada, el comando las crea. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`. La opción `--no-wait` crea la máquina virtual en segundo plano, así que puede continuar con el siguiente paso. En este ejemplo se crea una máquina virtual llamada *myVm1*:

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm1 \
  --image UbuntuLTS \
  --generate-ssh-keys \
  --no-wait
```

### <a name="create-the-second-vm"></a>Creación de la segunda máquina virtual

Como usó la opción `--no-wait` en el paso anterior, puede avanzar y crear la segunda máquina virtual denominada *myVm2*.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm2 \
  --image UbuntuLTS \
  --generate-ssh-keys
```

### <a name="azure-cli-output-message"></a>Mensaje de salida de la CLI de Azure

Las máquinas virtuales tardan unos minutos en crearse. Una vez que Azure crea las máquinas virtuales, la CLI de Azure devuelve una salida similar a la siguiente:

```output
{
  "fqdns": "",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVm2",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.5",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
  "zones": ""
}
```

Anote el valor de **publicIpAddress**. Usará esta dirección para conectarse a la máquina virtual desde Internet en el paso siguiente.

## <a name="connect-to-a-vm-from-the-internet"></a>Conexión a una máquina virtual desde Internet

En este comando, reemplace `<publicIpAddress>` por la dirección IP pública de la máquina virtual *myVm2*:

```bash
ssh <publicIpAddress>
```

## <a name="communicate-between-vms"></a>Comunicarse entre máquinas virtuales

Para confirmar la comunicación privada entre las máquinas virtuales *myVm2* y *myVm1*, escriba este comando:

```bash
ping myVm1 -c 4
```

Recibirá cuatro respuestas de *10.0.0.4*.

Cierre la sesión SSH con la máquina virtual *myVm2*.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede utilizar [az group delete](/cli/azure/group) para eliminar el grupo de recursos y todos los recursos que contiene:

```azurecli-interactive
az group delete --name myResourceGroup --yes
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado una red virtual predeterminada y dos máquinas virtuales. Se ha conectado a una máquina virtual desde Internet y se ha comunicado de forma privada entre las dos máquinas virtuales.
Azure permite que haya comunicación privada sin restricciones entre las máquinas virtuales. De manera predeterminada, Azure solo permite conexiones de Escritorio remoto entrantes a las máquinas virtuales Windows desde Internet. Para más información sobre cómo configurar distintos tipos de comunicaciones de red de VM, vaya al siguiente artículo:
> [!div class="nextstepaction"]
> [Filtrado del tráfico de red](tutorial-filter-network-traffic.md)
