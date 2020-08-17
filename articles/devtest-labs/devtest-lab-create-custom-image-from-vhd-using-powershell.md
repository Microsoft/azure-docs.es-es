---
title: Creación de una imagen personalizada a partir de un archivo VHD mediante Azure PowerShell
description: Automatización de la creación de una imagen personalizada en Azure DevTest Labs a partir de un archivo VHD mediante PowerShell
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 4b0712fdbec1ce23ad9e09d972e425cb7941107b
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288981"
---
# <a name="create-a-custom-image-from-a-vhd-file-using-powershell"></a>Creación de una imagen personalizada a partir de un archivo VHD mediante PowerShell

[!INCLUDE [devtest-lab-create-custom-image-from-vhd-selector](../../includes/devtest-lab-create-custom-image-from-vhd-selector.md)]

[!INCLUDE [devtest-lab-custom-image-definition](../../includes/devtest-lab-custom-image-definition.md)]

[!INCLUDE [devtest-lab-upload-vhd-options](../../includes/devtest-lab-upload-vhd-options.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="step-by-step-instructions"></a>Instrucciones paso a paso

Los siguientes pasos le guían en la creación de una imagen personalizada a partir de un archivo VHD mediante PowerShell:

1. En un símbolo del sistema de PowerShell, inicie sesión en su cuenta de Azure con la siguiente llamada al cmdlet **Connect-AzAccount**.

    ```powershell
    Connect-AzAccount
    ```

1.  Seleccione la suscripción de Azure deseada mediante una llamada al cmdlet **Select-AzSubscription**. Reemplace el marcador de posición siguiente de la variable **$subscriptionId** por un identificador de suscripción válido de Azure.

    ```powershell
    $subscriptionId = '<Specify your subscription ID here>'
    Select-AzSubscription -SubscriptionId $subscriptionId
    ```

1.  Obtenga el objeto de laboratorio mediante la llamada al cmdlet **Get-AzResource**. Reemplace los siguientes marcadores de posición de las variables **$labRg** y **$labName** por los valores adecuados para su entorno.

    ```powershell
    $labRg = '<Specify your lab resource group name here>'
    $labName = '<Specify your lab name here>'
    $lab = Get-AzResource -ResourceId ('/subscriptions/' + $subscriptionId + '/resourceGroups/' + $labRg + '/providers/Microsoft.DevTestLab/labs/' + $labName)
    ```

1.  Reemplace el siguiente marcador de posición de la variable **$vhdUri** por el identificador URI al archivo VHD cargado. Puede obtener el identificador URI del archivo VHD en la hoja del blob de la cuenta de almacenamiento en el portal de Azure.

    ```powershell
    $vhdUri = '<Specify the VHD URI here>'
    ```

1.  Cree la imagen personalizada mediante el cmdlet **New-AzResourceGroupDeployment**. Reemplace los siguientes marcadores de posición de las variables **$customImageName** y **$customImageDescription** por nombres significativos en su entorno.

    ```powershell
    $customImageName = '<Specify the custom image name>'
    $customImageDescription = '<Specify the custom image description>'

    $parameters = @{existingLabName="$($lab.Name)"; existingVhdUri=$vhdUri; imageOsType='windows'; isVhdSysPrepped=$false; imageName=$customImageName; imageDescription=$customImageDescription}

    New-AzResourceGroupDeployment -ResourceGroupName $lab.ResourceGroupName -Name CreateCustomImage -TemplateUri 'https://raw.githubusercontent.com/Azure/azure-devtestlab/master/samples/DevTestLabs/QuickStartTemplates/201-dtl-create-customimage-from-vhd/azuredeploy.json' -TemplateParameterObject $parameters
    ```

## <a name="powershell-script-to-create-a-custom-image-from-a-vhd-file"></a>Script de PowerShell para crear una imagen personalizada a partir de un archivo VHD

El siguiente script de PowerShell se puede usar para crear una imagen personalizada a partir de un archivo VHD. Reemplace los marcadores de posición (inicio y fin con corchetes angulares) por los valores adecuados según sus necesidades.

```powershell
# Log in to your Azure account.
Connect-AzAccount

# Select the desired Azure subscription.
$subscriptionId = '<Specify your subscription ID here>'
Select-AzSubscription -SubscriptionId $subscriptionId

# Get the lab object.
$labRg = '<Specify your lab resource group name here>'
$labName = '<Specify your lab name here>'
$lab = Get-AzResource -ResourceId ('/subscriptions/' + $subscriptionId + '/resourceGroups/' + $labRg + '/providers/Microsoft.DevTestLab/labs/' + $labName)

# Set the URI of the VHD file.
$vhdUri = '<Specify the VHD URI here>'

# Set the custom image name and description values.
$customImageName = '<Specify the custom image name>'
$customImageDescription = '<Specify the custom image description>'

# Set up the parameters object.
$parameters = @{existingLabName="$($lab.Name)"; existingVhdUri=$vhdUri; imageOsType='windows'; isVhdSysPrepped=$false; imageName=$customImageName; imageDescription=$customImageDescription}

# Create the custom image.
New-AzResourceGroupDeployment -ResourceGroupName $lab.ResourceGroupName -Name CreateCustomImage -TemplateUri 'https://raw.githubusercontent.com/Azure/azure-devtestlab/master/samples/DevTestLabs/QuickStartTemplates/201-dtl-create-customimage-from-vhd/azuredeploy.json' -TemplateParameterObject $parameters
```

## <a name="related-blog-posts"></a>Entradas blogs relacionadas

- [Custom images or formulas? (¿Imágenes personalizadas o fórmulas?)](./devtest-lab-faq.md#blog-post)
- [Copying Custom Images between Azure DevTest Labs (Copiar imágenes personalizadas entre instancias de Azure DevTest Labs)](https://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>Pasos siguientes

- [Agregar una máquina virtual al laboratorio](devtest-lab-add-vm.md)
