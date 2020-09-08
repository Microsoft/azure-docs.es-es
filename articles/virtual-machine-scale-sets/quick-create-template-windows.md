---
title: 'Inicio rápido: Creación de un conjunto de escalado de máquinas virtuales Windows con una plantilla de Azure'
description: Aprenda a crear rápidamente un conjunto de escalado de máquinas virtuales con Windows con una plantilla de Azure Resource Manager que implementa una aplicación de ejemplo y configura reglas de escalado automático.
author: ju-shim
ms.author: jushiman
ms.topic: quickstart
ms.service: virtual-machine-scale-sets
ms.subservice: windows
ms.date: 03/27/2020
ms.reviewer: mimckitt
ms.custom: mimckitt, subject-armqs
ms.openlocfilehash: 03863171cb05c0557611fb8cd3c0ade667e7d79e
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88650084"
---
# <a name="quickstart-create-a-windows-virtual-machine-scale-set-with-an-arm-template"></a>Inicio rápido: Creación de un conjunto de escalado de máquinas virtuales Windows con una plantilla de Resource Manager

El conjunto de escalado de máquinas virtuales le permite implementar y administrar un conjunto de máquinas virtuales de escalado automático. Puede escalar el número de máquinas virtuales del conjunto de escalado manualmente o definir reglas de escalado automático según el uso de recursos tales como la CPU, la demanda de memoria o el tráfico de red. Un equilibrador de carga de Azure distribuirá el tráfico a las instancias de máquina virtual del conjunto de escalado. En este inicio rápido, creará un conjunto de escalado de máquinas virtuales e implementará una aplicación de ejemplo con una plantilla de Azure Resource Manager.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Las plantillas de Resource Manager permiten implementar grupos de recursos relacionados. En una única plantilla, puede crear el conjunto de escalado de máquinas virtuales, instalar aplicaciones y configurar reglas de escalado automático. Con el uso de variables y parámetros, esta plantilla se puede reutilizar para actualizar los conjuntos de escalado existentes o crear más. Puede implementar plantillas mediante Azure Portal, la CLI de Azure, Azure PowerShell, o bien desde las canalizaciones de integración continua o entrega continua (CI/CD).

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-vmss-windows-webapp-dsc-autoscale%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/201-vmss-windows-webapp-dsc-autoscale/).

:::code language="json" source="~/quickstart-templates/201-vmss-windows-webapp-dsc-autoscale/azuredeploy.json":::

Estos recursos se definen en estas plantillas:

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
| imageReference               | Imagen personalizada o plataforma que se usará para las instancias de VM | Microsoft Windows Server 2016 Datacenter  |
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

La plantilla usa la extensión DSC de PowerShell para instalar una aplicación ASP.NET MVC que se ejecuta en IIS.

Se descarga un script de instalación desde GitHub, tal como se define en *url*. A continuación, la extensión ejecuta *InstallIIS* desde el script *IISInstall.ps1*, tal como se define en *function* y *script*. La propia aplicación de ASP.NET se proporciona como un paquete de Web Deploy, que también se descarga desde GitHub, tal como se define en *WebDeployPackagePath*:

## <a name="deploy-the-template"></a>Implementación de la plantilla

Puede implementar la plantilla si selecciona el botón **Implementar en Azure**. Este botón abre Azure Portal, carga la plantilla completa y pide algunos parámetros, como un nombre para el conjunto de escalado, el recuento de instancias y las credenciales de administrador.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-vmss-windows-webapp-dsc-autoscale%2Fazuredeploy.json)

También puede implementar una plantilla de Resource Manager mediante Azure PowerShell:

```azurepowershell-interactive
# Create a resource group
New-AzResourceGroup -Name myResourceGroup -Location EastUS

# Deploy template into resource group
New-AzResourceGroupDeployment `
    -ResourceGroupName myResourceGroup `
    -TemplateURI https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vmss-windows-webapp-dsc-autoscale/azuredeploy.json

# Update the scale set and apply the extension
Update-AzVmss `
    -ResourceGroupName myResourceGroup `
    -VmScaleSetName myVMSS `
    -VirtualMachineScaleSet $vmssConfig
```

Responda a los avisos para proporcionar un nombre para el conjunto de escalado y las credenciales de administrador de las instancias de VM. El proceso de creación del conjunto de escalado y la aplicación de la extensión para configurar la aplicación puede tardar entre 10 y 15 minutos.

## <a name="validate-the-deployment"></a>Validación de la implementación

Para ver el conjunto de escalado en acción, acceda a la aplicación web de ejemplo en un explorador web. Obtenga la dirección IP pública del equilibrador de carga con [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) de la siguiente manera:

```azurepowershell-interactive
Get-AzPublicIpAddress -ResourceGroupName myResourceGroup | Select IpAddress
```

Escriba la dirección IP pública del equilibrador de carga en un explorador web con el formato *http:\//publicIpAddress/MyApp*. El equilibrador de carga distribuye el tráfico a una de las instancias de VM, como se muestra en el ejemplo siguiente:

![Ejecución del sitio de IIS](./media/virtual-machine-scale-sets-create-powershell/running-iis-site.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y el conjunto de escalado. El parámetro `-Force` confirma que desea eliminar los recursos sin pedir confirmación adicional. El parámetro `-AsJob` devuelve el control a la petición de confirmación sin esperar a que finalice la operación.

```azurepowershell-interactive
Remove-AzResourceGroup -Name "myResourceGroup" -Force -AsJob
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un conjunto de escalado de Windows con una plantilla de Resource Manager y ha usado la extensión DSC de PowerShell para instalar una aplicación ASP.NET básica en las instancias de máquina virtual. Para más información, continúe con el tutorial sobre cómo crear y administrar conjuntos de escalado de máquinas virtuales de Azure.

> [!div class="nextstepaction"]
> [Creación y administración de conjuntos de escalado de máquinas virtuales de Azure](tutorial-create-and-manage-powershell.md)
