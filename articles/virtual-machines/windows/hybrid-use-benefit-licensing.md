---
title: Ventaja para uso híbrido de Azure para Windows Server
description: Descubra cómo maximizar las ventajas de Software Assurance de Windows para incorporar licencias locales a Azure
author: xujing-ms
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 4/22/2018
ms.author: xujing
ms.openlocfilehash: 1c87c517a0013fa31ba63ef1f78677fe85f5d287
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87074266"
---
# <a name="azure-hybrid-benefit-for-windows-server"></a>Ventaja para uso híbrido de Azure para Windows Server
Para los clientes con Software Assurance, la ventaja para uso híbrido de Azure para Windows Server le permite usar las licencias de Windows Server locales y ejecutar máquinas virtuales de Windows en Azure a bajo costo. Puede usar la Ventaja híbrida de Azure para Windows Server para implementar nuevas máquinas virtuales con el SO Windows. En este artículo se recorren los pasos necesarios para implementar nuevas máquinas virtuales con la Ventaja híbrida de Azure para Windows Server y para actualizar las máquinas virtuales en funcionamiento existentes. Para obtener más información acerca de los ahorros de costos y la concesión de licencias de la ventaja para uso híbrido para Azure para Windows Server, vea la [página de concesión de licencias de la ventaja para uso híbrido de Azure para Windows Server](https://azure.microsoft.com/pricing/hybrid-use-benefit/).

Cada una de las licencias de dos procesadores o cada uno de los conjuntos de licencias de 16 núcleos tienen derecho a dos instancias de hasta ocho núcleos o a una instancia de hasta 16 núcleos. La Ventaja híbrida de Azure para licencias de Standard Edition solo podrá usarse una vez en el entorno local o en Azure. Las ventajas de Datacenter Edition permiten el uso simultáneo de forma local y en Azure.

Ahora se admite el uso de la Ventaja híbrida de Azure para Windows Server con cualquier máquina virtual que ejecute el sistema operativo Windows Server en todas las regiones, incluidas las máquinas virtuales con software adicional como SQL Server o el software de catálogos de soluciones de terceros. 


## <a name="classic-vms"></a>Máquinas virtuales clásicas

Para las VM clásicas, solo se admite la implementación de una nueva VM desde imágenes personalizadas locales. Para aprovechar las ventajas de las funcionalidades admitidas en este artículo, primero debe migrar las máquinas virtuales clásicas al modelo de Resource Manager.

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]
 

## <a name="ways-to-use-azure-hybrid-benefit-for-windows-server"></a>Formas de usar la ventaja para uso híbrido de Azure para Windows Server
Hay varias maneras de utilizar máquinas virtuales de Windows con la ventaja para uso híbrido de Azure:

1. Puede implementar máquinas virtuales desde una de las imágenes de Windows Server en Azure Marketplace proporcionadas.
2. Puede cargar una máquina virtual personalizada e implementar mediante una plantilla de Resource Manager o Azure PowerShell.
3. Puede cambiar y convertir la máquina virtual existente entre la ejecución con la Ventaja híbrida de Azure o el pago del coste adicional por Windows Server
4. También puede aplicar la Ventaja híbrida de Azure para Windows Server en un conjunto de escalado de máquinas virtuales.


## <a name="create-a-vm-with-azure-hybrid-benefit-for-windows-server"></a>Crear una máquina virtual con la Ventaja híbrida de Azure para Windows Server
Todas las imágenes basadas en sistemas operativos de Windows Server son compatibles con la Ventaja híbrida de Azure para Windows Server. Puede usar imágenes compatibles con la plataforma de Azure o cargar sus propias imágenes personalizadas de Windows Server. 

### <a name="portal"></a>Portal
Para crear una máquina virtual con la Ventaja híbrida de Azure para Windows Server, use el botón de alternancia de la sección "Ahorrar dinero".

### <a name="powershell"></a>PowerShell


```powershell
New-AzVm `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVM" `
    -Location "East US" `
    -ImageName "Win2016Datacenter" `
    -LicenseType "Windows_Server"
```

### <a name="cli"></a>CLI
```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --location eastus \
    --license-type Windows_Server
```

### <a name="template"></a>Plantilla
En las plantillas de Resource Manager, se debe especificar un parámetro `licenseType` adicional. Puede obtener más información sobre la [creación de plantillas de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).
```json
"properties": {
    "licenseType": "Windows_Server",
    "hardwareProfile": {
        "vmSize": "[variables('vmSize')]"
    }
```

## <a name="convert-an-existing-vm-using-azure-hybrid-benefit-for-windows-server"></a>Conversión de una máquina virtual existente con la Ventaja híbrida de Azure para Windows Server
Si tiene una máquina virtual existente que le gustaría convertir para aprovechar la Ventaja híbrida de Azure para Windows Server, puede actualizar el tipo de licencia de dicha máquina siguiendo las instrucciones que se indican a continuación.

> [!NOTE]
> Cuando se cambia el tipo de licencia de una máquina virtual, el sistema no se reinicia ni se interrumpe el servicio.  Se trata simplemente de una actualización de una marca de metadatos.
> 

### <a name="portal"></a>Portal
En la hoja de la máquina virtual del portal, puede actualizar la máquina virtual para usar la Ventaja híbrida de Azure; para ello, seleccione la opción "Configuración" y cambie a la opción "Ventaja híbrida de Azure".

### <a name="powershell"></a>PowerShell
- Conversión de máquinas virtuales existentes de Windows Server a la Ventaja híbrida de Azure para Windows Server

    ```powershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "Windows_Server"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```
    
- Conversión de nuevo de las máquinas virtuales de Windows Server con la ventaja al pago por uso

    ```powershell
    $vm = Get-AzVM -ResourceGroup "rg-name" -Name "vm-name"
    $vm.LicenseType = "None"
    Update-AzVM -ResourceGroupName rg-name -VM $vm
    ```
    
### <a name="cli"></a>CLI
- Conversión de máquinas virtuales existentes de Windows Server a la Ventaja híbrida de Azure para Windows Server

    ```azurecli
    az vm update --resource-group myResourceGroup --name myVM --set licenseType=Windows_Server
    ```

### <a name="how-to-verify-your-vm-is-utilizing-the-licensing-benefit"></a>Cómo comprobar que la máquina virtual está usando la ventaja de licencia
Cuando haya implementado la máquina virtual mediante PowerShell, la plantilla de Resource Manager o el portal, puede comprobar el valor en los siguientes métodos.

### <a name="portal"></a>Portal
En la hoja de máquinas virtuales del portal, puede ver el botón de alternancia de la Ventaja híbrida de Azure para Windows Server si selecciona la pestaña "Configuración".

### <a name="powershell"></a>PowerShell
En el ejemplo siguiente, se muestra el tipo de licencia de una sola máquina virtual
```powershell
Get-AzVM -ResourceGroup "myResourceGroup" -Name "myVM"
```

Salida:
```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              : Windows_Server
```

Este resultado se contrasta con la siguiente máquina virtual implementada sin licencias para la ventaja para uso híbrido de Azure para Windows Server:
```powershell
Type                     : Microsoft.Compute/virtualMachines
Location                 : westus
LicenseType              :
```

### <a name="cli"></a>CLI
```azurecli
az vm get-instance-view -g MyResourceGroup -n MyVM --query "[?licenseType=='Windows_Server']" -o table
```

> [!NOTE]
> Cuando se cambia el tipo de licencia de una máquina virtual, el sistema no se reinicia ni se interrumpe el servicio. Se trata simplemente de una marca de licencia de metadatos.
>

## <a name="list-all-vms-with-azure-hybrid-benefit-for-windows-server-in-a-subscription"></a>Enumeración de todas las máquinas virtuales con la Ventaja híbrida de Azure para Windows Server en una suscripción
Para ver y contar todas las máquinas virtuales implementadas con la ventaja para uso híbrido de Azure para Windows Server, puede ejecutar el siguiente comando desde su suscripción:

### <a name="portal"></a>Portal
En la hoja de recursos de la máquina virtual o de los conjuntos de escalado de máquinas virtuales, puede ver una lista de todas las máquinas virtuales y el tipo de licencia si configura la columna de la tabla de forma que incluya "Ventaja híbrida de Azure". El valor de la máquina virtual puede estar en los siguientes estados: "Habilitado", "No habilitado" o "No compatible".

### <a name="powershell"></a>PowerShell
```powershell
$vms = Get-AzVM
$vms | ?{$_.LicenseType -like "Windows_Server"} | select ResourceGroupName, Name, LicenseType
```

### <a name="cli"></a>CLI
```azurecli
az vm list --query "[?licenseType=='Windows_Server']" -o table
```

## <a name="deploy-a-virtual-machine-scale-set-with-azure-hybrid-benefit-for-windows-server"></a>Implementación de un conjunto de escalado de máquinas virtuales con la Ventaja híbrida de Azure para Windows Server
En las plantillas de Resource Manager del conjunto de escalado de máquinas virtuales, se debe especificar un parámetro `licenseType` adicional en la propiedad VirtualMachineProfile. Puede hacerlo durante la creación o actualización del conjunto de escalado a través de la plantilla de ARM, PowerShell, la CLI de Azure o REST.

En el ejemplo siguiente, se usa una plantilla de ARM con una imagen de Windows Server 2016 Datacenter:
```json
"virtualMachineProfile": {
    "storageProfile": {
        "osDisk": {
            "createOption": "FromImage"
        },
        "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2016-Datacenter",
            "version": "latest"
        }
    },
    "licenseType": "Windows_Server",
    "osProfile": {
            "computerNamePrefix": "[parameters('vmssName')]",
            "adminUsername": "[parameters('adminUsername')]",
            "adminPassword": "[parameters('adminPassword')]"
    }
```
También puede obtener más información sobre cómo [modificar un conjunto de escalado de máquinas virtuales](../../virtual-machine-scale-sets/virtual-machine-scale-sets-upgrade-scale-set.md) para conocer más formas de actualizar su conjunto de escalado.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre [cómo ahorrar dinero con la Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-use-benefit/).
- Obtenga más información sobre [Preguntas más frecuentes sobre la Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-use-benefit/faq/).
- Aprenda más sobre la [concesión de licencias para la Ventaja híbrida de Azure en esta guía detallada](/windows-server/get-started/azure-hybrid-benefit).
- Obtenga más información sobre [Azure Hybrid Benefit for Windows Server and Azure Site Recovery make migrating applications to Azure even more cost-effective](https://azure.microsoft.com/blog/hybrid-use-benefit-migration-with-asr/) (la Ventaja híbrida de Azure para Windows Server y Azure Site Recovery rentabilizan aún más la migración de aplicaciones a Azure).
- Obtenga más información sobre [Implementación de Windows 10 en Azure con derechos de hospedaje multiinquilino](./windows-desktop-multitenant-hosting-deployment.md).
- Obtenga más información sobre el [uso de plantillas de Resource Manager](../../azure-resource-manager/management/overview.md).
