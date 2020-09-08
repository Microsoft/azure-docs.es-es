---
title: 'Tutorial: Escalado automático de un conjunto de escalado con plantillas de Azure'
description: Aprenda a usar plantillas de Azure Resource Manager para escalar automáticamente un conjunto de escalado de máquinas virtuales a medida que la demanda de la CPU aumenta y disminuye.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 03/27/2018
ms.reviewer: avverma
ms.custom: avverma
ms.openlocfilehash: fae86e13be624d7a5304aa04b82432e1163b1244
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "84629548"
---
# <a name="tutorial-automatically-scale-a-virtual-machine-scale-set-with-an-azure-template"></a>Tutorial: Escalado automático de conjuntos de escalado de máquinas virtuales con una plantilla de Azure
Al crear un conjunto de escalado, puede definir el número de instancias de máquina virtual que quiere ejecutar. A medida que cambia la demanda de las aplicaciones, puede aumentar o reducir automáticamente el número de estas instancias. La posibilidad de realizar el escalado automático le permite satisfacer la demanda del cliente o responder a los cambios de rendimiento de la aplicación a lo largo del ciclo de vida de esta. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Usar el escalado automático con un conjunto de escalado
> * Crear y usar reglas de escalado automático
> * Realizar pruebas de esfuerzo de instancias de máquina virtual y desencadenar reglas de escalado automático
> * Reducir el escalado automáticamente cuando se reduzca la demanda

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure 2.0.29 o versiones posteriores. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 


## <a name="define-an-autoscale-profile"></a>Definición de un perfil de escalado automático
Se define un perfil de escalado automático en una plantilla de Azure con el proveedor de recursos *Microsoft.Insights/autoscalesettings*. Un *perfil* establece la capacidad del conjunto de escalado y las reglas asociadas. En el ejemplo siguiente se define un perfil llamado *Escalado automático por porcentaje según el uso de CPU* y se establece una capacidad mínima predeterminada de *2* instancias de máquina virtual y un máximo de *10*:

```json
{
"type": "Microsoft.insights/autoscalesettings",
"name": "Autoscale",
"apiVersion": "2015-04-01",
"location": "[variables('location')]",
"scale": null,
"properties": {
  "profiles": [
    {
      "name": "Autoscale by percentage based on CPU usage",
      "capacity": {
        "minimum": "2",
        "maximum": "10",
        "default": "2"
      }
    }
  ]
}
```


## <a name="define-a-rule-to-autoscale-out"></a>Definición de una regla de escalado horizontal automático
Si aumenta la demanda de la aplicación, la carga de las instancias de máquina virtual del conjunto de escalado aumenta. Si este aumento de la carga es continuado, en lugar de ser algo puntual, puede configurar reglas de escalado automático para aumentar el número de instancias de máquina virtual en el conjunto de escalado. Cuando se crean estas instancias de máquina virtual y se implementan las aplicaciones, el conjunto de escalado empieza a distribuir el tráfico entre ellas mediante el equilibrador de carga. Puede controlar qué métricas se deben supervisar como, por ejemplo, la CPU o el disco, cuánto tiempo debe cumplir la carga de la aplicación un límite determinado y cuántas instancias de máquina virtual se deben agregar al conjunto de escalado.

En el ejemplo siguiente, se define una regla que aumenta el número de instancias de máquina virtual de un conjunto de escalado cuando la carga promedio de la CPU es superior al 70 % durante un período de más de 5 minutos. Cuando se desencadena la regla, aumenta el número de instancias de máquina virtual en tres.

Los siguientes parámetros se utilizan para esta regla:

| Parámetro         | Explicación                                                                                                         | Value           |
|-------------------|---------------------------------------------------------------------------------------------------------------------|-----------------|
| *metricName*      | La métrica de rendimiento del conjunto de escalado sobre el que realizar las acciones de supervisión y aplicación.                                                   | Porcentaje de CPU  |
| *timeGrain*       | Frecuencia de recopilación de las métricas para el análisis.                                                                   | 1 minuto.        |
| *timeAggregation* | Define cómo se deben agregar las métricas recopiladas para el análisis.                                                | Average         |
| *timeWindow*      | El periodo durante el que se realiza la supervisión antes de que se comparen los valores de métricas y umbrales.                                   | 5 minutos       |
| *operator*        | El operador que se utiliza para comparar los datos de las métricas con los umbrales.                                                     | Mayor que    |
| *threshold*       | El valor que hace que la regla de escalado automático desencadene una acción.                                                      | 70%             |
| *direction*       | Define si el conjunto de escalado debe escalarse o reducirse horizontalmente al aplicar la regla.                                              | Aumento        |
| *type*            | Indica que el número de instancias de máquina virtual debe cambiarse por un valor especificado.                                    | Nº de cambios    |
| *value*           | Número de instancias de máquina virtual que se debe escalar o reducir horizontalmente al aplicar la regla.                                             | 3               |
| *cooldown*        | El periodo que hay que esperar hasta que la regla se vuelva a aplicar, para que las acciones de escalado automático tengan tiempo de surtir efecto. | 5 minutos       |

Se debe agregar la siguiente regla a la sección profile del proveedor de recursos *Microsoft.Insights/autoscalesettings* de la sección anterior:

```json
"rules": [
  {
    "metricTrigger": {
      "metricName": "Percentage CPU",
      "metricNamespace": "",
      "metricResourceUri": "[concat('/subscriptions/',subscription().subscriptionId, '/resourceGroups/',  resourceGroup().name, '/providers/Microsoft.Compute/virtualMachineScaleSets/', variables('vmssName'))]",
      "metricResourceLocation": "[variables('location')]",
      "timeGrain": "PT1M",
      "statistic": "Average",
      "timeWindow": "PT5M",
      "timeAggregation": "Average",
      "operator": "GreaterThan",
      "threshold": 70
    },
    "scaleAction": {
      "direction": "Increase",
      "type": "ChangeCount",
      "value": "3",
      "cooldown": "PT5M"
    }
  }
]
```


## <a name="define-a-rule-to-autoscale-in"></a>Definición de una regla de reducción horizontal automática
La demanda de la aplicación puede reducirse por las tardes o durante los fines de semana. Si esta reducción es constante a lo largo de un período, puede configurar reglas de escalado automático para reducir el número de instancias de máquina virtual del conjunto de escalado. Esta acción de reducción horizontal permite rebajar el costo de ejecutar el conjunto de escalado ya que solo se ejecuta el número de instancias necesario para satisfacer la demanda actual.

En el ejemplo siguiente, se define una regla que reduce el número de instancias de máquina virtual en uno cuando la carga promedio de la CPU es inferior al 30 % durante un período de más de 5 minutos. Esta regla se debe agregar al perfil de escalado automático después de la regla anterior de escalado horizontal:

```json
{
  "metricTrigger": {
    "metricName": "Percentage CPU",
    "metricNamespace": "",
    "metricResourceUri": "[concat('/subscriptions/',subscription().subscriptionId, '/resourceGroups/',  resourceGroup().name, '/providers/Microsoft.Compute/virtualMachineScaleSets/', variables('vmssName'))]",
    "metricResourceLocation": "[variables('location')]",
    "timeGrain": "PT1M",
    "statistic": "Average",
    "timeWindow": "PT5M",
    "timeAggregation": "Average",
    "operator": "LessThan",
    "threshold": 30
  },
  "scaleAction": {
    "direction": "Decrease",
    "type": "ChangeCount",
    "value": "1",
    "cooldown": "PT5M"
  }
}
```


## <a name="create-an-autoscaling-scale-set"></a>Creación de un conjunto de escalado automático
Vamos a usar una plantilla de ejemplo para crear un conjunto de escalado y aplicar reglas de escalado automático. También puede [revisar la plantilla completa](https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/scale_sets/autoscale.json) o [consultar la sección *Microsoft.Insights/autoscalesettings* del proveedor de recursos](https://github.com/Azure-Samples/compute-automation-configurations/blob/master/scale_sets/autoscale.json#L220) de la plantilla.

En primer lugar, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [az group deployment create](/cli/azure/group/deployment). Cuando se le solicite, proporcione su propio nombre de usuario, por ejemplo, *azureuser*, y la contraseña que se utilizan como credenciales para cada instancia de máquina virtual:

```azurecli-interactive
az group deployment create \
  --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/scale_sets/autoscale.json
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.


## <a name="generate-cpu-load-on-scale-set"></a>Generación de carga de CPU en el conjunto de escalado
Para probar las reglas de escalado automático, genere cargas de CPU en las instancias de máquina virtual del conjunto de escalado. Esta carga de CPU simulada hace que las reglas de escalado automático escalen horizontalmente y aumenten el número de instancias de máquina virtual. Como la carga simulada de la CPU se reduce, las reglas de escalado automático reducen horizontalmente el número de instancias de máquina virtual.

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
sudo apt-get update
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
           1  True                  eastus      myScaleSet_1  Succeeded            MYRESOURCEGROUP  4f92f350-2b68-464f-8a01-e5e590557955
           2  True                  eastus      myScaleSet_2  Succeeded            MYRESOURCEGROUP  d734cd3d-fb38-4302-817c-cfe35655d48e
           4  True                  eastus      myScaleSet_4  Creating             MYRESOURCEGROUP  061b4c90-0d73-49fc-a066-19eab0b3d95c
           5  True                  eastus      myScaleSet_5  Creating             MYRESOURCEGROUP  4beff8b9-4e65-40cb-9652-43899309da27
           6  True                  eastus      myScaleSet_6  Creating             MYRESOURCEGROUP  9e4133dd-2c57-490e-ae45-90513ce3b336
```

Cuando se detiene **stress** en las instancias de máquina virtual iniciales, la carga media de CPU vuelve al estado normal. Después de otros 5 minutos, las reglas de escalado automático reducen horizontalmente el número de instancias de máquina virtual. Con las acciones de reducción horizontal se eliminan instancias de máquina virtual en orden, empezando por los identificadores mayores. Cuando un conjunto de escalado usa conjuntos de disponibilidad o zonas de disponibilidad, las acciones de reducción horizontal se distribuyen por igual entre esas instancias de máquina virtual. En la salida de ejemplo siguiente se muestra la eliminación de una instancia de máquina virtual con la reducción horizontal del conjunto de escalado:

```output
           6  True                  eastus      myScaleSet_6  Deleting             MYRESOURCEGROUP  9e4133dd-2c57-490e-ae45-90513ce3b336
```

Salga de la utilidad *watch* con `Ctrl-c`. El conjunto de escalado continúa la reducción horizontal cada 5 minutos y quita una instancia de máquina virtual hasta que se alcanza el mínimo de dos instancias.


## <a name="clean-up-resources"></a>Limpieza de recursos
Para quitar el conjunto de escalado y los recursos adicionales, elimine el grupo de recursos y todos sus recursos con [az group delete](/cli/azure/group):

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
