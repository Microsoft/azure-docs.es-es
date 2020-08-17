---
title: 'Extensión de controlador de InfiniBand: máquinas virtuales Windows de Azure'
description: Extensión de Microsoft Azure para instalar controladores de InfiniBand en máquinas virtuales de proceso de las series H y N que ejecutan Windows.
services: virtual-machines-windows
documentationcenter: ''
author: vermagit
editor: ''
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 07/20/2020
ms.author: amverma
ms.openlocfilehash: ccc9df8078bb7fec8be7d72b0ae18ed416bb10ca
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095141"
---
# <a name="infiniband-driver-extension-for-windows"></a>Extensión del controlador de InfiniBand para Windows

Esta extensión instala los controladores ND (para máquinas virtuales no habilitadas para SR-IOV) y controladores OFED (para máquinas virtuales habilitadas para SR-IOV) de InfiniBand (tamaños "r") máquinas virtuales de las series [H](../sizes-hpc.md) y [N](../sizes-gpu.md) que ejecutan Windows. En función de la familia de máquinas virtuales, la extensión instala los controladores adecuados para la NIC de Connect-X.

También hay disponible una extensión para instalar los controladores de InfiniBand para [máquinas virtuales Linux](hpc-compute-infiniband-linux.md).

## <a name="prerequisites"></a>Prerrequisitos

### <a name="operating-system"></a>Sistema operativo

Esta extensión admite las siguientes distribuciones del sistema operativo, dependiendo de la compatibilidad de controladores para la versión específica del sistema operativo.

| Distribución | Versión |
|---|---|
| Windows 10 | Core |
| Windows Server 2019 | Core |
| Windows Server 2016 | Core |
| Windows Server 2012 R2 | Core |
| Windows Server 2012 | Core |

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
    "type": "InfiniBandDriverWindows",
    "typeHandlerVersion": "1.2",
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
| type | InfiniBandDriverWindows | string |
| typeHandlerVersion | 1.2 | int |



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
    "type": "InfiniBandDriverWindows",
    "typeHandlerVersion": "1.2",
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
    -ExtensionName "InfiniBandDriverWindows" `
    -ExtensionType "InfiniBandDriverWindows" `
    -TypeHandlerVersion 1.2 `
    -SettingString '{ `
    }'
```

### <a name="azure-cli"></a>Azure CLI

```azurecli
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name InfiniBandDriverWindows \
  --publisher Microsoft.HpcCompute \
  --version 1.2 
```

### <a name="add-extension-to-a-virtual-machine-scale-set"></a>Incorporación de la extensión a un conjunto de escalado de máquinas virtuales

El ejemplo siguiente instala la versión más reciente de la extensión InfiniBandDriverWindows, la versión 1.2, en todas las VM compatibles con RDMA en un conjunto de escalado de máquinas virtuales denominado *myVMSS* implementado en el grupo de recursos denominado *myResourceGroup*:

  ```powershell
  $VMSS = Get-AzVmss -ResourceGroupName "myResourceGroup" -VMScaleSetName "myVMSS"
  Add-AzVmssExtension -VirtualMachineScaleSet $VMSS -Name "InfiniBandDriverWindows" -Publisher "Microsoft.HpcCompute" -Type "InfiniBandDriverWindows" -TypeHandlerVersion "1.2"
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

```cmd
C:\WindowsAzure\Logs\Plugins\Microsoft.HpcCompute.InfiniBandDriverWindows\
```

### <a name="exit-codes"></a>Códigos de salida

En la tabla siguiente se describe el significado y la acción recomendada en función de los códigos de salida del proceso de instalación de la extensión.

| Código de error | Significado | Acción posible |
| :---: | --- | --- |
| 0 | Operación correcta |
| 3010 | Operación correcta. Se requiere reiniciar. |
| 100 | La operación no es compatible o no se pudo completar. | Causas posibles: No se admite la versión de PowerShell, el tamaño de la VM no corresponde a una VM habilitada para InfiniBand, no se pudieron descargar los datos. Compruebe los archivos de registro para determinar la causa del error. |
| 240, 840 | Tiempo de espera de la operación. | Reintentar operación. |
| -1 | Se produjo una excepción. | Comprobar los archivos de registro para determinar la causa de la excepción. |

### <a name="support"></a>Soporte técnico

Si necesita más ayuda con cualquier aspecto de este artículo, puede ponerse en contacto con los expertos de Azure en los [foros de MSDN Azure o Stack Overflow](https://azure.microsoft.com/support/community/). De manera alternativa, puede notificar una incidencia de soporte técnico en el [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/). Para obtener información sobre el uso del soporte técnico, lea las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/).

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre las máquinas virtuales habilitadas para InfiniBand (tamaños "r"), consulte las máquinas virtuales de las series [H](../sizes-hpc.md) y [N](../sizes-gpu.md).

> [!div class="nextstepaction"]
> [Más información sobre las extensiones y características se VM de Linux](features-linux.md)