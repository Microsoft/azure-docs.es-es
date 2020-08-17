---
title: 'Inicio rápido: Creación de un conjunto de escalado de máquinas virtuales con la CLI de Azure'
description: Para empezar a trabajar con las implementaciones, aprenda a crear rápidamente un conjunto de escalado de máquinas virtuales con la CLI de Azure.
author: ju-shim
ms.author: jushiman
ms.topic: quickstart
ms.service: virtual-machine-scale-sets
ms.subservice: cli
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: 0dc98ba242d3eb5fba79605dae9f8eadc56affd7
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87504439"
---
# <a name="quickstart-create-a-virtual-machine-scale-set-with-the-azure-cli"></a>Inicio rápido: Creación de un conjunto de escalado de máquinas virtuales con la CLI de Azure
El conjunto de escalado de máquinas virtuales le permite implementar y administrar un conjunto de máquinas virtuales de escalado automático. Puede escalar el número de máquinas virtuales del conjunto de escalado manualmente o definir reglas de escalado automático según el uso de recursos tales como la CPU, la demanda de memoria o el tráfico de red. Un equilibrador de carga de Azure distribuirá el tráfico a las instancias de máquina virtual del conjunto de escalado. En esta guía de inicio rápido creará un conjunto de escalado de máquinas virtuales e implementará una aplicación de ejemplo con la CLI de Azure.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure 2.0.29 o versiones posteriores. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado
Antes de poder crear un conjunto de escalado, cree un grupo de recursos con [az group create](/cli/azure/group). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Ahora, cree un conjunto de escalado de máquinas virtuales con [az vmss create](/cli/azure/vmss). En el ejemplo siguiente se crea un conjunto de escalado llamado *myScaleSet* que se establece para actualizarse automáticamente a medida que se aplican los cambios, y se generan claves SSH si no existen en *~/.ssh/id_rsa*. Estas claves SSH se usan si necesita iniciar sesión en las instancias de máquina virtual. Para usar un conjunto existente de claves SSH, utilice en su lugar el parámetro `--ssh-key-value` y especifique la ubicación de las claves.

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


## <a name="deploy-sample-application"></a>Implementación de una aplicación de ejemplo
Para probar el conjunto de escalado, instale una aplicación web básica. La extensión de script personalizado de Azure se usa para descargar y ejecutar un script que instala una aplicación en las instancias de máquina virtual. Esta extensión es útil para la configuración posterior a la implementación, la instalación de software o cualquier otra tarea de configuración o administración. Para obtener más información, consulte [Información general de la extensión de script personalizado](../virtual-machines/extensions/custom-script-linux.md).

Use la extensión de script personalizado para instalar un servidor web de NGINX básico. Aplique la extensión de script personalizado que instala NGINX con [az vmss extension set](/cli/azure/vmss/extension) como se indica a continuación:

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/automate_nginx.sh"],"commandToExecute":"./automate_nginx.sh"}'
```


## <a name="allow-traffic-to-application"></a>Permitir tráfico a la aplicación
Cuando se creó el conjunto de escalado, se implementó automáticamente un equilibrador de carga de Azure. El equilibrador de carga distribuye el tráfico a las instancias de máquina virtual del conjunto de escalado. Para permitir que el tráfico llegue a la aplicación web de ejemplo, cree una regla del equilibrador de carga con [az network lb rule create](/cli/azure/network/lb/rule). En el ejemplo siguiente, se crea una regla denominada *myLoadBalancerRuleWeb*:

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


## <a name="test-your-scale-set"></a>Prueba del conjunto de escalado
Para ver el conjunto de escalado en acción, acceda a la aplicación web de ejemplo en un explorador web. Obtenga la dirección IP pública del equilibrador de carga con [az network public-ip show](/cli/azure/network/public-ip). En el ejemplo siguiente se obtiene la dirección IP de *myLoadBalancerRuleWeb* que se ha creado como parte del conjunto de escalado:

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroup \
  --name myScaleSetLBPublicIP \
  --query '[ipAddress]' \
  --output tsv
```

Escriba la dirección IP pública del equilibrador de carga en un explorador web. El equilibrador de carga distribuye el tráfico a una de las instancias de VM, como se muestra en el ejemplo siguiente:

![Página web predeterminada de NGINX](media/virtual-machine-scale-sets-create-cli/running-nginx-site.png)


## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando ya no lo necesite, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, el conjunto de escalado y todos los recursos relacionados como se indica a continuación. El parámetro `--no-wait` devuelve el control a la petición de confirmación sin esperar a que finalice la operación. El parámetro `--yes` confirma que desea eliminar los recursos sin pedir confirmación adicional.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```


## <a name="next-steps"></a>Pasos siguientes
En esta guía de inicio rápido, creó un conjunto de escalado básico y usó la extensión de script personalizado para instalar un servidor web NGINX básico en las instancias de máquina virtual. Para más información, continúe con el tutorial sobre cómo crear y administrar conjuntos de escalado de máquinas virtuales de Azure.

> [!div class="nextstepaction"]
> [Creación y administración de conjuntos de escalado de máquinas virtuales de Azure](tutorial-create-and-manage-cli.md)
