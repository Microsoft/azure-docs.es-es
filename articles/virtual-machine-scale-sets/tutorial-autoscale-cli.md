---
title: 'Tutorial: Escalado automático de un conjunto de escalado con la CLI de Azure'
description: Aprenda a usar la CLI de Azure para escalar automáticamente un conjunto de escalado de máquinas virtuales a medida que la demanda de la CPU aumenta y disminuye
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 05/18/2018
ms.reviewer: avverma
ms.custom: avverma, devx-track-azurecli
ms.openlocfilehash: 5e1cc5512eeaec058a9ce3030517391a54a7707f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501668"
---
# <a name="tutorial-automatically-scale-a-virtual-machine-scale-set-with-the-azure-cli"></a>Tutorial: Escalado automático de un conjunto de escalado de máquinas virtuales con la CLI de Azure

Al crear un conjunto de escalado, puede definir el número de instancias de máquina virtual que quiere ejecutar. A medida que cambia la demanda de las aplicaciones, puede aumentar o reducir automáticamente el número de estas instancias. La posibilidad de realizar el escalado automático le permite satisfacer la demanda del cliente o responder a los cambios de rendimiento de la aplicación a lo largo del ciclo de vida de esta. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar el escalado automático con un conjunto de escalado
> * Crear y usar reglas de escalado automático
> * Realizar pruebas de esfuerzo de instancias de máquina virtual y desencadenar reglas de escalado automático
> * Reducir el escalado automáticamente cuando se reduzca la demanda

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si elige instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.32 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado

Cree un grupo de recursos con [az group create](/cli/azure/group), como sigue:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [az vmss create](/cli/azure/vmss). En el ejemplo siguiente se crea un conjunto de escalado con *2* instancias y se generan claves SSH (si aún no existen):

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --instance-count 2 \
  --admin-username azureuser \
  --generate-ssh-keys
```

## <a name="define-an-autoscale-profile"></a>Definición de un perfil de escalado automático

Para habilitar el escalado automático en un conjunto de escalado, primero debe definir un perfil de escalado automático. Este perfil define la capacidad predeterminada, mínima y máxima del conjunto de escalado. Estos límites le permiten controlar el costo al no crear continuamente instancias de máquina virtual, y equilibrar un rendimiento aceptable con un número mínimo de instancias que permanecen en un evento de reducción horizontal. Cree un perfil de escalado automático con [az monitor autoscale create](/cli/azure/monitor/autoscale#az-monitor-autoscale-create). En el ejemplo siguiente se establece la capacidad predeterminada y la mínima para *2* instancias de máquina virtual y un máximo de *10*:

```azurecli-interactive
az monitor autoscale create \
  --resource-group myResourceGroup \
  --resource myScaleSet \
  --resource-type Microsoft.Compute/virtualMachineScaleSets \
  --name autoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2
```

## <a name="create-a-rule-to-autoscale-out"></a>Creación de una regla de escalado automático horizontal

Si aumenta la demanda de la aplicación, la carga de las instancias de máquina virtual del conjunto de escalado aumenta. Si este aumento de la carga es continuado, en lugar de ser algo puntual, puede configurar reglas de escalado automático para aumentar el número de instancias de máquina virtual en el conjunto de escalado. Cuando se crean estas instancias de máquina virtual y se implementan las aplicaciones, el conjunto de escalado empieza a distribuir el tráfico entre ellas mediante el equilibrador de carga. Puede controlar qué métricas se deben supervisar como, por ejemplo, la CPU o el disco, cuánto tiempo debe cumplir la carga de la aplicación un límite determinado y cuántas instancias de máquina virtual se deben agregar al conjunto de escalado.

Vamos a crear una regla con [az monitor autoscale rule create](/cli/azure/monitor/autoscale/rule#az-monitor-autoscale-rule-create) que aumenta el número de instancias de máquina virtual de un conjunto de escalado cuando la carga media de la CPU es superior al 70 % durante un período de más de 5 minutos. Cuando se desencadena la regla, aumenta el número de instancias de máquina virtual en tres.

```azurecli-interactive
az monitor autoscale rule create \
  --resource-group myResourceGroup \
  --autoscale-name autoscale \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 3
```

## <a name="create-a-rule-to-autoscale-in"></a>Creación de una regla de escalado automático horizontal de reducción

La demanda de la aplicación puede reducirse por las tardes o durante los fines de semana. Si esta reducción es constante a lo largo de un período, puede configurar reglas de escalado automático para reducir el número de instancias de máquina virtual del conjunto de escalado. Esta acción de reducción horizontal permite rebajar el costo de ejecutar el conjunto de escalado ya que solo se ejecuta el número de instancias necesario para satisfacer la demanda actual.

Cree otra regla con [az monitor autoscale rule create](/cli/azure/monitor/autoscale/rule#az-monitor-autoscale-rule-create) que reduzca el número de instancias de máquina virtual de un conjunto de escalado cuando la carga media de la CPU descienda por debajo del 30 % durante un período de más de 5 minutos. En el ejemplo siguiente se define la regla para reducir horizontalmente el número de instancias de máquina virtual a una:

```azurecli-interactive
az monitor autoscale rule create \
  --resource-group myResourceGroup \
  --autoscale-name autoscale \
  --condition "Percentage CPU < 30 avg 5m" \
  --scale in 1
```

## <a name="generate-cpu-load-on-scale-set"></a>Generación de carga de CPU en el conjunto de escalado

Para probar las reglas de escalado automático, genere cargas de CPU en las instancias de máquina virtual del conjunto de escalado. Esta carga de CPU simulada hace que el escalado automático escale y reduzca horizontalmente el número de instancias de máquina virtual. Como la carga simulada de la CPU se reduce, las reglas de escalado automático reducen horizontalmente el número de instancias de máquina virtual.

Primero, enumere las direcciones y los puertos para conectarse a las instancias de máquina virtual en un conjunto de escalado con [az vmss list-instance-connection-info](/cli/azure/vmss):

```azurecli-interactive
az vmss list-instance-connection-info \
  --resource-group myResourceGroup \
  --name myScaleSet
```

La salida del ejemplo siguiente muestra el nombre de instancia, la dirección IP pública del equilibrador de carga y el número de puerto al que las reglas de traducción de direcciones de red (NAT) reenvían el tráfico:

```json
{
  "instance 1": "13.92.224.66:50001",
  "instance 3": "13.92.224.66:50003"
}
```

Conéctese mediante SSH a la primera instancia de máquina virtual. Especifique su propia dirección IP pública y el número de puerto con el parámetro `-p`, tal como se muestra en el comando anterior:

```console
ssh azureuser@13.92.224.66 -p 50001
```

Una vez iniciada la sesión, instale la utilidad **stress**. Inicie *10* trabajos de **stress** que generen carga de CPU. Estos trabajos se ejecutan durante *420* segundos, lo suficiente para que las reglas de escalado automático implementen la acción deseada.

```console
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 420 &
```

Cuando **stress** muestra una salida similar a *stress: info: [2688] dispatching hogs: 10 cpu, 0 io, 0 vm, 0 hdd*, presione la tecla *Entrar* para volver al símbolo del sistema.

Para confirmar que **stress** genera carga de CPU, examine la carga del sistema activa con la utilidad **top**:

```console
top
```

Salga de la utilidad **top** y cierre la conexión a la instancia de máquina virtual. **Stress** continuará ejecutándose en la instancia de máquina virtual.

```console
Ctrl-c
exit
```

Conéctese a la segunda instancia de máquina virtual con el número de puerto de la lista [az vmss list-instance-connection-info](/cli/azure/vmss) anterior:

```console
ssh azureuser@13.92.224.66 -p 50003
```

Instale y ejecute la utilidad **stress** e inicie diez trabajos en esta segunda instancia de máquina virtual.

```console
sudo apt-get -y install stress
sudo stress --cpu 10 --timeout 420 &
```

De nuevo, cuando **stress** muestra una salida similar a *stress: info: [2713] dispatching hogs: 10 cpu, 0 io, 0 vm, 0 hdd*, presione la tecla *Entrar* para volver al símbolo del sistema.

Cierre la conexión a esta instancia de máquina virtual. **Stress** continuará ejecutándose en la instancia de máquina virtual.

```console
exit
```

## <a name="monitor-the-active-autoscale-rules"></a>Supervisión de las reglas de escalado automático activas

Para supervisar el número de instancias de máquina virtual del conjunto de escalado, use la utilidad **watch**. Las reglas de escalado automático tardan 5 minutos en empezar el proceso de escalado horizontal en respuesta a la carga de CPU generada por **stress** en cada instancia de máquina virtual:

```azurecli-interactive
watch az vmss list-instances \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --output table
```

Cuando se alcanza el umbral de la CPU, las reglas de escalado automático aumentan el número de instancias de máquina virtual en el conjunto de escalado. La siguiente salida muestra tres máquinas virtuales que se crean durante el escalado horizontal automático:

```output
Every 2.0s: az vmss list-instances --resource-group myResourceGroup --name myScaleSet --output table

  InstanceId  LatestModelApplied    Location    Name          ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  ------------  -------------------  ---------------  ------------------------------------
           1  True                  eastus      myScaleSet_1  Succeeded            myResourceGroup  4f92f350-2b68-464f-8a01-e5e590557955
           2  True                  eastus      myScaleSet_2  Succeeded            myResourceGroup  d734cd3d-fb38-4302-817c-cfe35655d48e
           4  True                  eastus      myScaleSet_4  Creating             myResourceGroup  061b4c90-0d73-49fc-a066-19eab0b3d95c
           5  True                  eastus      myScaleSet_5  Creating             myResourceGroup  4beff8b9-4e65-40cb-9652-43899309da27
           6  True                  eastus      myScaleSet_6  Creating             myResourceGroup  9e4133dd-2c57-490e-ae45-90513ce3b336
```

Cuando se detiene **stress** en las instancias de máquina virtual iniciales, la carga media de CPU vuelve al estado normal. Después de otros 5 minutos, las reglas de escalado automático reducen horizontalmente el número de instancias de máquina virtual. Con las acciones de reducción horizontal se eliminan instancias de máquina virtual en orden, empezando por los identificadores mayores. Cuando un conjunto de escalado usa conjuntos de disponibilidad o zonas de disponibilidad, las acciones de reducción horizontal se distribuyen por igual entre esas instancias de máquina virtual. En la salida de ejemplo siguiente se muestra la eliminación de una instancia de máquina virtual con la reducción horizontal del conjunto de escalado:

```output
           6  True                  eastus      myScaleSet_6  Deleting             myResourceGroup  9e4133dd-2c57-490e-ae45-90513ce3b336
```

Salga de la utilidad *watch* con `Ctrl-c`. El conjunto de escalado continúa la reducción horizontal cada 5 minutos y quita una instancia de máquina virtual hasta que se alcanza el mínimo de dos instancias.

## <a name="clean-up-resources"></a>Limpieza de recursos

Para quitar el conjunto de escalado y los recursos adicionales, elimine el grupo de recursos y todos sus recursos con [az group delete](/cli/azure/group). El parámetro `--no-wait` devuelve el control a la petición de confirmación sin esperar a que finalice la operación. El parámetro `--yes` confirma que desea eliminar los recursos sin pedir confirmación adicional.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a escalar y reducir horizontalmente un conjunto de escalado de forma automática con la CLI de Azure:

> [!div class="checklist"]
> * Usar el escalado automático con un conjunto de escalado
> * Crear y usar reglas de escalado automático
> * Realizar pruebas de esfuerzo de instancias de máquina virtual y desencadenar reglas de escalado automático
> * Reducir el escalado automáticamente cuando se reduzca la demanda
