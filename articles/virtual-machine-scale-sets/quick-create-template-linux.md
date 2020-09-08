---
title: 'Inicio rápido: Creación de un conjunto de escalado de máquinas virtuales Linux con una plantilla de Azure Resource Manager'
description: Aprenda a crear rápidamente un conjunto de escalado de máquinas virtuales con Linux con una plantilla de Azure Resource Manager que implementa una aplicación de ejemplo y configura reglas de escalado automático.
author: ju-shim
ms.author: jushiman
ms.topic: quickstart
ms.service: virtual-machine-scale-sets
ms.subservice: linux
ms.date: 03/27/2020
ms.reviewer: mimckitt
ms.custom: mimckitt, subject-armqs
ms.openlocfilehash: f6ab030b7f807a884b5d05487724fc9c66a6de87
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88648639"
---
# <a name="quickstart-create-a-linux-virtual-machine-scale-set-with-an-arm-template"></a>Inicio rápido: Creación de un conjunto de escalado de máquinas virtuales Linux con una plantilla de Resource Manager

El conjunto de escalado de máquinas virtuales le permite implementar y administrar un conjunto de máquinas virtuales de escalado automático. Puede escalar el número de máquinas virtuales del conjunto de escalado manualmente o definir reglas de escalado automático según el uso de recursos tales como la CPU, la demanda de memoria o el tráfico de red. Un equilibrador de carga de Azure distribuirá el tráfico a las instancias de máquina virtual del conjunto de escalado. En este inicio rápido, creará un conjunto de escalado de máquinas virtuales e implementará una aplicación de ejemplo con una plantilla de Azure Resource Manager.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Las plantillas de Resource Manager permiten implementar grupos de recursos relacionados. En una única plantilla, puede crear el conjunto de escalado de máquinas virtuales, instalar aplicaciones y configurar reglas de escalado automático. Con el uso de variables y parámetros, esta plantilla se puede reutilizar para actualizar los conjuntos de escalado existentes o crear más. Puede implementar plantillas mediante Azure Portal, la CLI de Azure o Azure PowerShell, o bien desde las canalizaciones de integración continua o entrega continua (CI/CD).

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-vmss-bottle-autoscale%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/201-vmss-bottle-autoscale/).

:::code language="json" source="~/quickstart-templates/201-vmss-bottle-autoscale/azuredeploy.json":::

Estos recursos se definen en la plantilla:

- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
- [**Microsoft.Network/publicIPAddresses**](/azure/templates/microsoft.network/publicipaddresses)
- [**Microsoft.Network/loadBalancers**](/azure/templates/microsoft.network/loadbalancers)
- [**Microsoft.Compute/virtualMachineScaleSets**](/azure/templates/microsoft.compute/virtualmachinescalesets)
- [**Microsoft.Insights/autoscaleSettings**](/azure/templates/microsoft.insights/autoscalesettings)

### <a name="define-a-scale-set"></a>Definición de un conjunto de escalado

Para crear una escala con una plantilla, debe definir los recursos adecuados. Las partes principales del tipo de recurso del conjunto de escalado de máquinas virtuales son:

| Propiedad                     | Descripción de la propiedad                                  | Valor de la plantilla de ejemplo                    |
|------------------------------|----------------------------------------------------------|-------------------------------------------|
| type                         | Tipo de recurso de Azure que se creará                            | Microsoft.Compute/virtualMachineScaleSets |
| name                         | Nombre del conjunto de escalado                                       | myScaleSet                                |
| ubicación                     | Ubicación donde se creará el conjunto de escalado                     | Este de EE. UU.                                   |
| sku.name                     | Tamaño de VM para cada instancia de conjunto de escalado                  | Standard_A1                               |
| sku.capacity                 | Número de instancias de VM que se crearán inicialmente           | 2                                         |
| upgradePolicy.mode           | Modo de actualización de la instancia de VM cuando se producen cambios              | Automático                                 |
| imageReference               | Imagen personalizada o plataforma que se usará para las instancias de VM | Canonical Ubuntu Server 16.04-LTS         |
| osProfile.computerNamePrefix | Prefijo de nombre para cada instancia de VM                     | myvmss                                    |
| osProfile.adminUsername      | Nombre de usuario para cada instancia de VM                        | azureuser                                 |
| osProfile.adminPassword      | Contraseña para cada instancia de VM                        | P@ssw0rd!                                 |

Para personalizar una plantilla de conjunto de escalado, puede cambiar el tamaño de la máquina virtual o la capacidad inicial. Otra opción es usar una plataforma diferente o una imagen personalizada.

### <a name="add-a-sample-application"></a>Adición de una aplicación de ejemplo

Para probar el conjunto de escalado, instale una aplicación web básica. Cuando se implementa un conjunto de escalado, las extensiones de VM pueden proporcionar tareas de configuración y automatización posteriores a la implementación, como la instalación de una aplicación. Los scripts se pueden descargar desde Azure Storage o GitHub, o se pueden proporcionar a Azure Portal en el tiempo de ejecución de la extensión. Para aplicar una extensión al conjunto de escalado, agregue la sección *extensionProfile* en el ejemplo anterior del recurso. Normalmente, el perfil de extensión define las propiedades siguientes:

- Tipo de extensión
- Editor de la extensión
- Versión de la extensión
- Ubicación de los scripts de configuración o instalación
- Comandos que se ejecutarán en las instancias de VM

La plantilla usa la extensión de script personalizada para instalar [Bottle](https://bottlepy.org/docs/dev/), una plataforma web de Python y un servidor HTTP sencillo.

Se definen dos scripts en **fileUris** - *installserver.sh* y *workserver.py*. Estos archivos se descargan desde GitHub y, a continuación, *commandToExecute* ejecuta `bash installserver.sh` para instalar y configurar la aplicación.

## <a name="deploy-the-template"></a>Implementación de la plantilla

Puede implementar la plantilla si selecciona el botón siguiente **Implementar en Azure**. Este botón abre Azure Portal, carga la plantilla completa y pide algunos parámetros, como un nombre para el conjunto de escalado, el recuento de instancias y las credenciales de administrador.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-vmss-bottle-autoscale%2Fazuredeploy.json)

También puede implementar una plantilla de Resource Manager mediante la CLI de Azure:

```azurecli-interactive
# Create a resource group
az group create --name myResourceGroup --location EastUS

# Deploy template into resource group
az group deployment create \
    --resource-group myResourceGroup \
    --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-bottle-autoscale/azuredeploy.json
```

Responda a los avisos para proporcionar un nombre para el conjunto de escalado, el recuento de instancias y las credenciales de administrador de las instancias de VM. El proceso de creación del conjunto de escalado y de los recursos compatibles tarda unos minutos.

## <a name="validate-the-deployment"></a>Validación de la implementación

Para ver el conjunto de escalado en acción, acceda a la aplicación web de ejemplo en un explorador web. Obtenga la dirección IP pública del equilibrador de carga con [az network public-ip list](/cli/azure/network/public-ip), tal y como se indica a continuación:

```azurecli-interactive
az network public-ip list \
    --resource-group myResourceGroup \
    --query [*].ipAddress -o tsv
```

Escriba la dirección IP pública del equilibrador de carga en un explorador web con el formato *http:\//publicIpAddress:9000/do_work*. El equilibrador de carga distribuye el tráfico a una de las instancias de VM, como se muestra en el ejemplo siguiente:

![Página web predeterminada de NGINX](media/virtual-machine-scale-sets-create-template/running-python-app.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos, el conjunto de escalado y todos los recursos relacionados como se indica a continuación. El parámetro `--no-wait` devuelve el control a la petición de confirmación sin esperar a que finalice la operación. El parámetro `--yes` confirma que desea eliminar los recursos sin pedir confirmación adicional.

```azurecli-interactive
az group delete --name myResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, creó un conjunto de escalado de Linux con una plantilla de Resource Manager y usó la extensión de script personalizado para instalar un servidor web de Python básico en las instancias de máquina virtual. Para más información, continúe con el tutorial sobre cómo crear y administrar conjuntos de escalado de máquinas virtuales de Azure.

> [!div class="nextstepaction"]
> [Creación y administración de conjuntos de escalado de máquinas virtuales de Azure](tutorial-create-and-manage-cli.md)
