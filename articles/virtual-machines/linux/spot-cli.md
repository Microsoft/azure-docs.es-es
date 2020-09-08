---
title: Uso de la CLI para implementar máquinas virtuales de Azure Spot
description: Aprenda a usar la CLI para implementar máquinas virtuales de Azure Spot para ahorrar costos.
author: cynthn
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 06/26/2020
ms.author: cynthn
ms.reviewer: jagaveer
ms.openlocfilehash: 165a2c6221c8d3c14f71134deef962d0859bb438
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816718"
---
# <a name="deploy-spot-vms-using-the-azure-cli"></a>Implementación de máquinas virtuales de Spot con la CLI de Azure

El uso de [máquinas virtuales de Azure Spot](spot-vms.md) permite aprovechar las ventajas de nuestra capacidad no utilizada con un importante ahorro en los costos. Siempre que Azure necesite recuperar la capacidad, su infraestructura expulsará las máquinas virtuales de Spot. Por lo tanto, estas son excelentes para cargas de trabajo que soportan interrupciones, como los trabajos de procesamiento por lotes, los entornos de desarrollo y pruebas, las grandes cargas de trabajo de proceso, etc.

Los precios de las máquinas virtuales de Spot varían en función de la región y la SKU. Para más información, consulte precios de las máquinas virtuales para [Linux](https://azure.microsoft.com/pricing/details/virtual-machines/linux/) y [Windows](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). 

Puede establecer el precio máximo por hora que esté dispuesto por la máquina virtual. El precio máximo de una máquina virtual de Spot se puede establecer en dólares estadounidenses (USD), con un máximo de 5 decimales. Por ejemplo, el valor `0.98765` correspondería a un precio máximo de 0,98765 USD por hora. Si establece el precio máximo en `-1`, la máquina virtual no se expulsará por precio. El precio de la máquina virtual será el actual de Spot o el de una máquina virtual estándar, el menor de los dos, siempre que haya capacidad y cuota disponibles. Para más información sobre la configuración del precio máximo, consulte [Máquinas virtuales de Spot - Precios](../spot-vms.md#pricing).

El proceso de creación de una máquina virtual con la CLI de Azure es el mismo que el que se detalla en el [artículo de inicio rápido](./quick-create-cli.md). Tan solo agregue el parámetro "--priority Spot", establezca `--eviction-policy` en Desasignar (este es el valor predeterminado) o en `Delete` e indique un precio máximo o `-1`. 


## <a name="install-azure-cli"></a>Instalación de la CLI de Azure

Para crear máquinas virtuales de Spot, debe ejecutar la versión 2.0.74 o posterior de la CLI de Azure. Para saber qué versión tiene, ejecute el comando **az --version**. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

Inicie sesión en Azure mediante [az login](/cli/azure/reference-index#az-login).

```azurecli
az login
```

## <a name="create-a-spot-vm"></a>Creación de una máquina virtual de Spot

En este ejemplo se muestra cómo implementar una máquina virtual de Spot en Linux que no se expulse por precio. La directiva de expulsión se establece para desasignar la VM, de modo que se pueda reiniciar en otro momento. Si quiere eliminar la VM y el disco subyacente cuando se expulsa la VM, establezca `--eviction-policy` en `Delete`.

```azurecli
az group create -n mySpotGroup -l eastus
az vm create \
    --resource-group mySpotGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --priority Spot \
    --max-price -1 \
    --eviction-policy Deallocate
```



Una vez creada la máquina virtual, puede realizar una consulta para ver el precio máximo de facturación de todas las máquinas virtuales del grupo de recursos.

```azurecli
az vm list \
   -g mySpotGroup \
   --query '[].{Name:name, MaxPrice:billingProfile.maxPrice}' \
   --output table
```

**Pasos siguientes**

También puede crear una VM de Spot con [Azure PowerShell](../windows/spot-powershell.md), el [portal](../windows/spot-portal.md) o una [plantilla](spot-template.md).

Si se produce un error, consulte [Códigos de error](../error-codes-spot.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
