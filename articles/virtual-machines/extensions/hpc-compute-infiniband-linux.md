---
title: 'Extensión del controlador InfiniBand: máquinas virtuales Linux de Azure'
description: Extensión de Microsoft Azure para instalar controladores InfiniBand en máquinas virtuales de proceso de las series H y N que ejecutan Linux.
services: virtual-machines-linux
documentationcenter: ''
author: vermagit
editor: ''
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 07/20/2020
ms.author: amverma
ms.openlocfilehash: eb06f98d1a6e9b76c321e3d202043d656a2d94eb
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87829057"
---
# <a name="infiniband-driver-extension-for-linux"></a>Extensión del controlador InfiniBand para Linux

Esta extensión instala los controladores InfiniBand OFED en máquinas virtuales de las series [H](../sizes-hpc.md) y [N](../sizes-gpu.md) habilitadas para InfiniBand y SR-IOV (tamaños "r") que ejecutan Linux. En función de la familia de máquinas virtuales, la extensión instala los controladores adecuados para la NIC de Connect-X.

Las instrucciones sobre la instalación manual de los controladores OFED están disponibles [aquí](../workloads/hpc/enable-infiniband.md#manual-installation).

También hay una extensión para instalar los controladores InfiniBand para [máquinas virtuales Windows](hpc-compute-infiniband-windows.md).

## <a name="prerequisites"></a>Prerrequisitos

### <a name="operating-system"></a>Sistema operativo

Esta extensión admite las siguientes distribuciones del sistema operativo, dependiendo de la compatibilidad de controladores para la versión específica del sistema operativo.

| Distribución | Versión |
|---|---|
| Linux: Ubuntu | 16.04 LTS, 18.04 LTS, 20.04 LTS |
| Linux: CentOS | 7.4, 7.5, 7.6, 7.7, 8.1, 8,2 |
| Linux: Red Hat Enterprise Linux | 7.4, 7.5, 7.6, 7.7, 8.1, 8,2 |

### <a name="internet-connectivity"></a>Conectividad de Internet

La extensión de Microsoft Azure para los controladores de InfiniBand requiere que la máquina virtual de destino esté conectada a Internet y tenga acceso.

## <a name="extension-schema"></a>Esquema de extensión

En el siguiente JSON, se muestra el esquema para la extensión.

```json
{
  "name": "<myExtensionName>",
  "type": "extensions",
  "apiVersion": "2015-06-15",
  "location": "<location>",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', <myVM>)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "InfiniBandDriverLinux",
    "typeHandlerVersion": "1.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="properties"></a>Propiedades

| Nombre | Valor / ejemplo | Tipo de datos |
| ---- | ---- | ---- |
| apiVersion | 2015-06-15 | date |
| publisher | Microsoft.HpcCompute | string |
| type | InfiniBandDriverLinux | string |
| typeHandlerVersion | 1.1 | int |



## <a name="deployment"></a>Implementación


### <a name="azure-resource-manager-template"></a>Plantilla de Azure Resource Manager 

Las extensiones de VM de Azure pueden implementarse con plantillas de Azure Resource Manager. Las plantillas resultan ideales al implementar una o varias máquinas virtuales que requieren configurarse tras la implementación.

La configuración JSON de una extensión de máquina virtual puede estar anidada en el recurso de máquina virtual o colocada en la raíz o nivel superior de una plantilla JSON de Resource Manager. La colocación de la configuración JSON afecta al valor del nombre y tipo del recurso. Para obtener más información, consulte el artículo sobre cómo [establecer el nombre y el tipo de recursos secundarios](../../azure-resource-manager/templates/child-resource-name-type.md). 

En el siguiente ejemplo se da por supuesto que la extensión está anidada dentro del recurso de máquina virtual. Cuando se anidan los recursos de extensión, la plantilla JSON se coloca en el objeto `"resources": []` de la máquina virtual.

```json
{
  "name": "myExtensionName",
  "type": "extensions",
  "location": "[resourceGroup().location]",
  "apiVersion": "2015-06-15",
  "dependsOn": [
    "[concat('Microsoft.Compute/virtualMachines/', myVM)]"
  ],
  "properties": {
    "publisher": "Microsoft.HpcCompute",
    "type": "InfiniBandDriverLinux",
    "typeHandlerVersion": "1.1",
    "autoUpgradeMinorVersion": true,
    "settings": {
    }
  }
}
```

### <a name="powershell"></a>PowerShell

```powershell
Set-AzVMExtension
    -ResourceGroupName "myResourceGroup" `
    -VMName "myVM" `
    -Location "southcentralus" `
    -Publisher "Microsoft.HpcCompute" `
    -ExtensionName "InfiniBandDriverLinux" `
    -ExtensionType "InfiniBandDriverLinux" `
    -TypeHandlerVersion 1.1 `
    -SettingString '{ `
    }'
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name InfiniBandDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.1 
```

### <a name="add-extension-to-a-virtual-machine-scale-set"></a>Incorporación de la extensión a un conjunto de escalado de máquinas virtuales

En el ejemplo siguiente se instala la versión más reciente de la extensión InfiniBandDriverLinux, la 1.1, en todas las máquinas virtuales compatibles con RDMA de un conjunto de escalado de máquinas virtuales existente denominado *myVMSS* implementado en el grupo de recursos denominado *myResourceGroup*:

  ```powershell
  $VMSS = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS"
  Add-AzVmssExtension -VirtualMachineScaleSet $VMSS -Name "InfiniBandDriverLinux" -Publisher "Microsoft.HpcCompute" -Type "InfiniBandDriverLinux" -TypeHandlerVersion "1.1"
  Update-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "MyVMSS" -VirtualMachineScaleSet $VMSS
  Update-AzVmssInstance -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS" -InstanceId "*"
```


## <a name="troubleshoot-and-support"></a>Solución de problemas y asistencia

### <a name="troubleshoot"></a>Solución de problemas

Los datos sobre el estado de las implementaciones de extensiones pueden recuperarse desde Azure Portal y mediante Azure PowerShell y la CLI de Azure. Para ver el estado de implementación de las extensiones de una VM determinada, ejecute el comando siguiente.

```powershell
Get-AzVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName
```

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

La salida de la ejecución de las extensiones se registra en el archivo siguiente: Consulte este archivo para realizar un seguimiento del estado de la instalación, así como para solucionar los errores.

```bash
/var/log/azure/ib-vmext-status
```

### <a name="exit-codes"></a>Códigos de salida

En la tabla siguiente se describe el significado y la acción recomendada en función de los códigos de salida del proceso de instalación de la extensión.

| Código de salida | Significado | Acción posible |
| :---: | --- | --- |
| 0 | Operación correcta |
| 1 | Uso incorrecto de la extensión. | Compruebe el registro de salida de ejecución |
| 10 | Linux Integration Services para Hyper-V y Azure no están disponibles o no están instalados | Comprobación de la salida de lspci |
| 11 | Mellanox InfiniBand no se encuentra en este tamaño de máquina virtual | Use un [tamaño de VM y SO admitido](../sizes-hpc.md) |
| 12 | Oferta de imagen no compatible |
| 13 | Tamaño de máquina virtual no compatible | Use una máquina virtual de las series [H](../sizes-hpc.md) y [N](../sizes-gpu.md) habilitada para InfiniBand (tamaño "r") para implementar |
| 14 | La operación no se realizó correctamente | Compruebe el registro de salida de ejecución |


### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/community/). De manera alternativa, puede notificar una incidencia de soporte técnico en el [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/). Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las máquinas virtuales habilitadas para InfiniBand (tamaños "r"), consulte las máquinas virtuales de las series [H](../sizes-hpc.md) y [N](../sizes-gpu.md).

> [!div class="nextstepaction"]
> [Más información sobre las extensiones y características se VM de Linux](features-linux.md)