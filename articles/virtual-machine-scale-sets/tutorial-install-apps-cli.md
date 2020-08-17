---
title: 'Tutorial: Instalación de aplicaciones en un conjunto de escalado con la CLI de Azure'
description: Aprenda a usar la CLI de Azure para instalar aplicaciones en conjuntos de escalado de máquinas virtuales con la extensión de script personalizado.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: cli
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 92bd5bb6e9c8a00b72ace5f81d1a1cedccfa95e5
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87503674"
---
# <a name="tutorial-install-applications-in-virtual-machine-scale-sets-with-the-azure-cli"></a>Tutorial: Instalación de aplicaciones en conjuntos de escalado de máquinas virtuales con la CLI de Azure
Para ejecutar aplicaciones en las instancias de máquinas virtuales (VM) de un conjunto de escalado, primero debe instalar los componentes de la aplicación y los archivos necesarios. En un tutorial anterior, aprendió a crear y usar una imagen de máquina virtual personalizada para implementar las instancias de máquina virtual. Esta imagen personalizada incluía instalaciones y configuraciones manuales de aplicaciones. También puede automatizar la instalación de aplicaciones en un conjunto de escalado después de implementar cada instancia de máquina virtual, o actualizar una aplicación que ya se ejecuta en un conjunto de escalado. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Instalar automáticamente aplicaciones en un conjunto de escalado
> * Usar la extensión de script personalizado de Azure
> * Actualizar una aplicación en ejecución en un conjunto de escalado

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure 2.0.29 o versiones posteriores. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 


## <a name="what-is-the-azure-custom-script-extension"></a>¿Qué es la extensión de script personalizado de Azure?
La extensión de script personalizado descarga y ejecuta scripts en máquinas virtuales de Azure. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración. Los scripts se pueden descargar desde Azure Storage o GitHub, o se pueden proporcionar a Azure Portal en el tiempo de ejecución de la extensión.

La extensión de script personalizado se integra con las plantillas de Azure Resource Manager y también se puede usar con la CLI de Azure, Azure PowerShell, Azure Portal o la API REST. Para obtener más información, consulte [Información general de la extensión de script personalizado](../virtual-machines/extensions/custom-script-linux.md).

Para usar la extensión de script personalizado con la CLI de Azure, puede crear un archivo JSON que defina qué archivos se van a obtener y qué comandos se van a ejecutar. Estas definiciones JSON se pueden volver a usar en implementaciones del conjunto de escalado para aplicar instalaciones de la aplicación coherentes.


## <a name="create-custom-script-extension-definition"></a>Creación de una definición de extensión de script personalizado
Para ver la extensión de script personalizado en acción, vamos a crear un conjunto de escalado que instala el servidor web de NGINX y genera el nombre de host de la instancia de máquina virtual del conjunto de escalado. La siguiente definición de extensión de script personalizado descarga un script de ejemplo de GitHub, instala los paquetes necesarios y, luego, escribe el nombre de host de la instancia de máquina virtual en una página HTML básica.

En el shell actual, cree un archivo denominado *customConfig.json* y pegue la siguiente configuración. Por ejemplo, cree el archivo en Cloud Shell, no en la máquina local. Puede utilizar el editor que prefiera. Escriba `sensible-editor customConfig.json` en Cloud Shell para crear el archivo y ver una lista de editores disponibles.

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],
  "commandToExecute": "./automate_nginx.sh"
}
```


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado
Cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [az vmss create](/cli/azure/vmss). En el ejemplo siguiente se crea un conjunto de escalado denominado *myScaleSet* y se generan claves SSH si estas no existen aún:

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --generate-ssh-keys
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.


## <a name="apply-the-custom-script-extension"></a>Aplicación de la extensión de script personalizado
Aplique la configuración de la extensión de script personalizado a las instancias de máquinas virtuales del conjunto de escalado con [az vmss extension set](/cli/azure/vmss/extension). En el siguiente ejemplo se aplica la configuración *customConfig.json* a las instancias de máquina virtual *myScaleSet* del grupo de recursos denominado *myResourceGroup*:

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings @customConfig.json
```

Cada instancia de máquina virtual del conjunto de escalado descarga y ejecuta el script de GitHub. En un ejemplo más complejo, se podrían instalar varios componentes y archivos de aplicaciones. Si el conjunto de escalado se escala verticalmente, las nuevas instancias de máquina virtual aplican automáticamente la misma definición de extensión de script personalizado e instalan la aplicación necesaria.


## <a name="test-your-scale-set"></a>Prueba del conjunto de escalado
Para permitir que el tráfico llegue al servidor web, cree una regla del equilibrador de carga con [az network lb rule create](/cli/azure/network/lb/rule). En el ejemplo siguiente, se crea una regla denominada *myLoadBalancerRuleWeb*:

```azurecli-interactive
az network lb rule create \
  --resource-group myResourceGroup \
  --name myLoadBalancerRuleWeb \
  --lb-name myScaleSetLB \
  --backend-pool-name myScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp
```

Para ver el servidor web en acción, obtenga la dirección IP pública del equilibrador de carga con [az network public-ip show](/cli/azure/network/public-ip). En el ejemplo siguiente se obtiene la dirección IP de *myLoadBalancerRuleWeb* que se ha creado como parte del conjunto de escalado:

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query [ipAddress] \
  --output tsv
```

Escriba la dirección IP pública del equilibrador de carga en un explorador web. El equilibrador de carga distribuye el tráfico a una de las instancias de VM, como se muestra en el ejemplo siguiente:

![Página web básica de NGINX](media/tutorial-install-apps-cli/running-nginx.png)

Deje el explorador web abierto para que pueda ver una versión actualizada en el paso siguiente.


## <a name="update-app-deployment"></a>Actualización de la implementación de aplicaciones
A lo largo del ciclo de vida de un conjunto de escalado, puede que deba implementar una versión actualizada de la aplicación. Con la extensión de script personalizado, puede hacer referencia a un script de implementación actualizado y, luego, volver a aplicar la extensión al conjunto de escalado. Cuando se creó el conjunto de escalado en un paso anterior, el parámetro `--upgrade-policy-mode` se estableció en *automatic*. Esta configuración permite que las instancias de máquina virtual del conjunto de escalado se actualicen y apliquen automáticamente la versión más reciente de la aplicación.

En el shell actual, cree un archivo denominado *customConfig2.json* y pegue la siguiente configuración. Esta definición ejecuta una versión *v2* actualizada del script de instalación de la aplicación:

```json
{
  "fileUris": ["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx_v2.sh"],
  "commandToExecute": "./automate_nginx_v2.sh"
}
```

Aplique de nuevo la configuración de la extensión de script personalizado a las instancias de máquina virtual del conjunto de escalado con [az vmss extension set](/cli/azure/vmss/extension). *customConfigv2.json* se usa para aplicar la versión actualizada de la aplicación:

```azurecli-interactive
az vmss extension set \
    --publisher Microsoft.Azure.Extensions \
    --version 2.0 \
    --name CustomScript \
    --resource-group myResourceGroup \
    --vmss-name myScaleSet \
    --settings @customConfigv2.json
```

Todas las instancias de máquina virtual del conjunto de escalado se actualizan automáticamente con la versión más reciente de la página web de ejemplo. Para ver la versión actualizada, actualice el sitio web en el explorador:

![Página web actualizada de NGINX](media/tutorial-install-apps-cli/running-nginx-updated.png)


## <a name="clean-up-resources"></a>Limpieza de recursos
Para quitar el conjunto de escalado y los recursos adicionales, elimine el grupo de recursos y todos sus recursos con [az group delete](/cli/azure/group). El parámetro `--no-wait` devuelve el control a la petición de confirmación sin esperar a que finalice la operación. El parámetro `--yes` confirma que desea eliminar los recursos sin pedir confirmación adicional.

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, aprendió a instalar y actualizar automáticamente aplicaciones en un conjunto de escalado con la CLI de Azure:

> [!div class="checklist"]
> * Instalar automáticamente aplicaciones en un conjunto de escalado
> * Usar la extensión de script personalizado de Azure
> * Actualizar una aplicación en ejecución en un conjunto de escalado

Vaya al siguiente tutorial para aprender cómo se escala automáticamente el conjunto de escalado.

> [!div class="nextstepaction"]
> [Escalado automático de los conjuntos de escalado](tutorial-autoscale-cli.md)
