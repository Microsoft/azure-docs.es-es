---
title: 'Error de administración de arranque de Windows : estado 0xC0000428, hash de imagen no válido'
titlesuffix: Azure Virtual Machines
description: En este artículo se proporcionan los pasos para resolver los problemas en los que se usó una imagen de versión preliminar y el período de prueba ha expirado, lo que impide el arranque de una máquina virtual (VM) de Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 26c411f9-6c22-4f45-a445-c5781e547828
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 09/03/2020
ms.author: v-miegge
ms.openlocfilehash: f607ebb64b27c45ec696d7fcd431a0ba2342697f
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89447667"
---
# <a name="windows-boot-manager-error---0xc0000428-status-invalid-image-hash"></a>Error de administración de arranque de Windows : estado 0xC0000428, hash de imagen no válido

En este artículo se proporcionan los pasos para resolver los problemas en los que se usó una imagen de versión preliminar y el período de prueba ha expirado, lo que impide el arranque de una máquina virtual (VM) de Azure.

## <a name="symptom"></a>Síntoma

Cuando use los [diagnósticos de arranque](https://docs.microsoft.com/azure/virtual-machines/troubleshooting/boot-diagnostics) para ver la captura de pantalla de la máquina virtual, verá que dicha captura de pantalla muestra la administración de arranque de Windows con el mensaje:

  `File: \windows\system32\boot\winload.exe`

  `Status: 0xc0000428`

  `Info: Windows cannot verify the digital signature for this file.`

  ![La ilustración 1 muestra la administración de arranque de Windows con el estado "Ox0000428" y la información "Windows no puede comprobar la firma digital de este archivo".](./media/windows-boot-error-invalid-image-hash/1-cannot-verify-signature.png)

o el mensaje:

  `File: \Windows\system32\winload.exe`

  `Status: 0xc0000428`

  `Info: The digital signature for this file couldn't be verified.`
    
  ![La ilustración 2 muestra la administración de arranque de Windows con el estado "Ox0000428" y la información "No se pudo comprobar la firma digital de este archivo".](./media/windows-boot-error-invalid-image-hash/2-digital-signature-not-verified.png)

## <a name="cause"></a>Causa

La imagen que se usó para crear la máquina virtual era una imagen de versión preliminar con una fecha de expiración en lugar de una imagen RTM (versión a fabricación). 

Las imágenes de versión preliminar tienen un ciclo de vida designado y la captura de pantalla que se muestra cuando se supera la fecha de expiración, lo que significa que la prueba de la imagen se ha superado.

### <a name="example-of-preview-images"></a>Ejemplo de imágenes de versión preliminar

`a699494373c04fc0bc8f2bb1389d6106__Windows-Server-Technical-Preview-201505.01-en.us-127GB.vhd`

No se puede ampliar la fecha de expiración de una imagen de versión preliminar. Cuando la versión preliminar haya expirado, la máquina virtual ya no podrá arrancar.

- Dependiendo del producto, el período de prueba puede variar. Por ejemplo, las imágenes de versión preliminar de Windows tienen un período de prueba de 180 días.

- En Azure, todas las imágenes de Windows que son versiones preliminares incluirán una nota en su descripción indicando que no están destinadas para producción y que están disponibles para usarse solo durante un período de prueba específico o como "versión preliminar".

## <a name="solution"></a>Solución

Si se trata de una imagen de versión preliminar, no hay ninguna manera de ampliar la fecha de expiración de la imagen usada, por lo que deberá [implementar una nueva máquina virtual](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal) mediante una imagen que no sea de versión preliminar. Los siguientes pasos le ayudarán a identificar si ha usado una imagen de versión preliminar, así como a proporcionar recursos para ayudarle a transferir datos de esta máquina virtual a una nueva. Si la ha identificado correctamente como una imagen de versión preliminar, no se podrá recuperar, ya que habrá expirado.

En función de sus preferencias, puede usar Azure PowerShell o la CLI de Azure para consultar la imagen y determinar si se trata de una imagen de versión preliminar. Puede usar estos comandos para confirmar que se trata de una imagen de versión preliminar.

### <a name="query-using-azure-powershell"></a>Consulta mediante Azure PowerShell

1. Abra la aplicación Windows PowerShell.
1. Ejecute los comandos siguientes:

   ```powershell
   $locName = "<LOCATION>" 
   $pubName = "<PUBLISHER NAME>" 
   $offerName = "<OFFER NAME>" 
   $skuName= "<YEAR WHEN THIS IMAGE WAS RELEASED>"
   Get-AzVMImagePublisher -Location $locName | Select $pubName
   Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
   Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
   Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Skus $skuName | Select Version
   ```

- En los comandos anteriores, reemplace `<LOCATION>`, `<PUBLISHER NAME>`, `<OFFER NAME>` y `<YEAR WHEN THIS IMAGE WAS RELEASED>` por la información indicada. Quite también los símbolos "<" y ">".

  Observe el ejemplo siguiente:

  ```powershell
  $locName = "West US" 
  $pubName = "MicrosoftWindowsServer" 
  $offerName = "WindowsServer" 
  $skuName= "2016-Datacenter"
  Get-AzVMImagePublisher -Location $locName | Select $pubName
  Get-AzVMImageOffer -Location $locName -PublisherName $pubName | Select Offer
  Get-AzVMImageSku -Location $locName -PublisherName $pubName -Offer $offerName | Select Skus
  Get-AzVMImage -Location $locName -PublisherName $pubName -Offer $offerName -Skus $skuName | Select Version
  ```

### <a name="query-using-the-azure-cli"></a>Consulta mediante la CLI de Azure

1. Si aún no lo hace, tendrá que [instalar la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli).
1. Una vez descargado, use el símbolo del sistema o PowerShell para escribir el comando `az login` y, a continuación, inicie sesión con las credenciales de su cuenta.
1. Una vez que haya iniciado sesión, escriba los comandos siguientes:

   ```powershell
   az vm image list-publishers --location "<LOCATION>"
   az vm image list-offers --location "West US" --publisher "<PUBLISHER NAME>"
   az vm image list-skus --location "West US" --publisher "<PUBLISHER NAME>" --offer "<OFFER NAME>"
   az vm image list  --location "West US" --publisher "<PUBLISHER NAME>" --offer "<OFFER NAME>" --sku "<YEAR WHEN THIS IMAGE WAS RELEASED>"
   ```

- En los comandos anteriores, reemplace `<LOCATION>`, `<PUBLISHER NAME>`, `<OFFER NAME>` y `<YEAR WHEN THIS IMAGE WAS RELEASED>` por la información indicada. Quite también los símbolos "<" y ">".

  Observe el ejemplo siguiente:

  ```powershell
  az vm image list-publishers --location "West US"
  az vm image list-offers --location "West US" --publisher "MicrosoftWindowsServer"
  az vm image list-skus --location "West US" --publisher "MicrosoftWindowsServer" --offer "WindowsServer"
  az vm image list  --location "West US" --publisher "MicrosoftWindowsServer" --offer "WindowsServer" --sku "2016-Datacenter"
  ```
