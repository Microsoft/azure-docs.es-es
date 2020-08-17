---
title: Administración de conjuntos de escalado de máquinas virtuales con la CLI de Azure
description: Comandos comunes de la CLI de Azure para administrar conjuntos de escalado de máquinas virtuales, por ejemplo, cómo iniciar y detener una instancia, o cambiar la capacidad del conjunto de escalado.
author: ju-shim
ms.author: jushiman
ms.topic: how-to
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 05/29/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 02f868417ef9feea1771174e62152708c1257425
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502909"
---
# <a name="manage-a-virtual-machine-scale-set-with-the-azure-cli"></a>Administración de un conjunto de escalado de máquinas virtuales con la CLI de Azure
Durante el ciclo de vida de la máquina virtual, es posible que deba ejecutar una o varias tareas de administración. Además, puede crear scripts para automatizar varias tareas de ciclo de vida. En este artículo se detallan algunos de los comandos comunes de la CLI de Azure que le permiten realizar estas tareas.

Para completar estas tareas de administración, se requiere la CLI de Azure más reciente. Para obtener información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). Si necesita crear un conjunto de escalado de máquinas virtuales, puede [crear un conjunto de escalado en la CLI de Azure](quick-create-cli.md).


## <a name="view-information-about-a-scale-set"></a>Visualización de información acerca de un conjunto de escalado
Para ver información general sobre un conjunto de escalado, use [az vmss show](/cli/azure/vmss). En el ejemplo siguiente se obtiene información sobre el conjunto de escalado denominado *myScaleSet* del grupo de recursos *myResourceGroup*. Escriba sus propios nombres, como se indica a continuación:

```azurecli
az vmss show --resource-group myResourceGroup --name myScaleSet
```


## <a name="view-vms-in-a-scale-set"></a>Visualización de máquinas virtuales en un conjunto de escalado
Para ver una lista de las instancias de VM en un conjunto de escalado, use [az vmss list-instances](/cli/azure/vmss). En el ejemplo siguiente se enumeran todas las instancias de máquina virtual en el conjunto de escalado denominado *myScaleSet* en el grupo de recursos *myResourceGroup*. Proporcione sus propios valores para estos nombres:

```azurecli
az vmss list-instances \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --output table
```

Para ver información adicional acerca de una instancia específica de la máquina virtual, agregue el parámetro `--instance-id` a [az vmss get-instance-view](/cli/azure/vmss) y especifique una instancia para verla. En el ejemplo siguiente se puede consultar información acerca de la instancia de VM *0* en el conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Escriba sus propios nombres, como se indica a continuación:

```azurecli
az vmss get-instance-view \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --instance-id 0
```


## <a name="list-connection-information-for-vms"></a>Lista de la información de conexión para máquinas virtuales
Para conectarse a las máquinas virtuales de un conjunto de escalado, establezca una conexión SSH o RDP a una dirección IP pública y un número de puerto asignados. De manera predeterminada, las reglas de traducción de direcciones de red (NAT) se agregan a la instancia de Azure Load Balancer que reenvía el tráfico de conexión remota a cada máquina virtual. Para mostrar las direcciones y los puertos para conectarse a las instancias de VM en un conjunto de escalado, use [az vmss list-instance-connection-info](/cli/azure/vmss). En el ejemplo siguiente, se muestra la información de conexión de las instancias de máquina virtual en el conjunto de escalado denominado *myScaleSet* y en el grupo de recursos *myResourceGroup*. Proporcione sus propios valores para estos nombres:

```azurecli
az vmss list-instance-connection-info \
    --resource-group myResourceGroup \
    --name myScaleSet
```


## <a name="change-the-capacity-of-a-scale-set"></a>Cambio de la capacidad de un conjunto de escalado
Los comandos anteriores muestran información acerca del conjunto de escalado y las instancias de VM. Para aumentar o disminuir el número de instancias en el conjunto de escalado, puede cambiar la capacidad. El conjunto de escalado crea o quita el número necesario de máquinas virtuales y luego configura las máquinas virtuales para recibir tráfico de la aplicación.

Para ver el número de instancias que tiene actualmente en un conjunto de escalado, use [az vmss show](/cli/azure/vmss) y realice consultas a *sku.capacity*:

```azurecli
az vmss show \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --query [sku.capacity] \
    --output table
```

A continuación, puede aumentar o reducir manualmente el número de máquinas virtuales del conjunto de escalado con [az vmss scale](/cli/azure/vmss). En el ejemplo siguiente se establece el número de máquinas virtuales en el conjunto de escalado en *5*:

```azurecli
az vmss scale \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --new-capacity 5
```

La capacidad del conjunto de escalado tarda unos minutos en actualizarse. Si reduce la capacidad de un conjunto de escalado, se quitan primero las máquinas virtuales con los identificadores de instancia más altos.


## <a name="stop-and-start-vms-in-a-scale-set"></a>Detención e inicio de las máquinas virtuales de un conjunto de escalado
Para detener una o más máquinas virtuales en un conjunto de escalado, use [az vmss stop](/cli/azure/vmss#az-vmss-stop). El parámetro `--instance-ids` le permite especificar una o más máquinas virtuales para que se detengan. Si no especifica un identificador de instancia, se detienen todas las máquinas virtuales del conjunto de escalado. Para detener varias máquinas virtuales, separe cada identificador de instancia con un espacio.

En el ejemplo siguiente se detiene la instancia *0* del conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Proporcione sus propios valores, como se indica a continuación:

```azurecli
az vmss stop --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```

Las máquinas virtuales detenidas permanecen asignadas y siguen acumulando cargos por procesos. Si en su lugar desea desasignar las máquinas virtuales y solo acumular cargos por procesos, use [az vmss deallocate](/cli/azure/vmss). Para desasignar varias máquinas virtuales, separe cada identificador de instancia con un espacio. En el ejemplo siguiente se detiene y desasigna la instancia *0* del conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Proporcione sus propios valores, como se indica a continuación:

```azurecli
az vmss deallocate --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


### <a name="start-vms-in-a-scale-set"></a>Inicio de máquinas virtuales en un conjunto de escalado
Para iniciar una o más máquinas virtuales en un conjunto de escalado, use [az vmss start](/cli/azure/vmss). El parámetro `--instance-ids` le permite especificar una o más máquinas virtuales para que se inicien. Si no especifica un identificador de instancia, se inician todas las máquinas virtuales del conjunto de escalado. Para iniciar varias máquinas virtuales, separe cada identificador de instancia con un espacio.

En el ejemplo siguiente se inicia la instancia *0* del conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Proporcione sus propios valores, como se indica a continuación:

```azurecli
az vmss start --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


## <a name="restart-vms-in-a-scale-set"></a>Reinicio de máquinas virtuales en un conjunto de escalado
Para reiniciar una o más máquinas virtuales en un conjunto de escalado, use [az vmss restart](/cli/azure/vmss). El parámetro `--instance-ids` le permite especificar una o más máquinas virtuales para que se reinicien. Si no especifica un identificador de instancia, se reinician todas las máquinas virtuales del conjunto de escalado. Para reiniciar varias máquinas virtuales, separe cada identificador de instancia con un espacio.

En el ejemplo siguiente se reinicia la instancia *0* del conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Proporcione sus propios valores, como se indica a continuación:

```azurecli
az vmss restart --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


## <a name="remove-vms-from-a-scale-set"></a>Eliminación de las máquinas virtuales de un conjunto de escalado
Para quitar una o más máquinas virtuales en un conjunto de escalado, use [az vmss delete-instances](/cli/azure/vmss). El parámetro `--instance-ids` le permite especificar una o más máquinas virtuales para que se eliminen. Si especifica * para el identificador de instancia, se quitarán todas las máquinas virtuales del conjunto de escalado. Para quitar varias máquinas virtuales, separe cada identificador de instancia con un espacio.

En el ejemplo siguiente se quita la instancia *0* del conjunto de escalado denominado *myScaleSet* y el grupo de recursos *myResourceGroup*. Proporcione sus propios valores, como se indica a continuación:

```azurecli
az vmss delete-instances --resource-group myResourceGroup --name myScaleSet --instance-ids 0
```


## <a name="next-steps"></a>Pasos siguientes
Entre otras tareas comunes para conjuntos de escalado se incluye cómo [implementar una aplicación](virtual-machine-scale-sets-deploy-app.md) y cómo [actualizar instancias de máquinas virtuales](virtual-machine-scale-sets-upgrade-scale-set.md). También puede usar la CLI de Azure para [configurar reglas de escalado automático](virtual-machine-scale-sets-autoscale-overview.md).
