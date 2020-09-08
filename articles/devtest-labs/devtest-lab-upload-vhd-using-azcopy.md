---
title: Carga de archivos VHD en Azure DevTest Labs mediante AzCopy | Microsoft Docs
description: En este artículo se proporciona un tutorial para usar la utilidad de línea de comandos AzCopy para cargar un archivo VHD en la cuenta de almacenamiento de un laboratorio en Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 1d8ede0f78726b04ac862a00b559b8d42c3ed1cd
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88650781"
---
# <a name="upload-vhd-file-to-labs-storage-account-using-azcopy"></a>Carga de archivos VHD en la cuenta de almacenamiento del laboratorio mediante AzCopy

[!INCLUDE [devtest-lab-upload-vhd-selector](../../includes/devtest-lab-upload-vhd-selector.md)]

En Azure DevTest Labs, se pueden usar archivos VHD para crear imágenes personalizadas, que se usan para aprovisionar máquinas virtuales. Los siguientes pasos le guían en el uso de la utilidad de línea de comandos AzCopy para cargar un archivo VHD en una cuenta de almacenamiento del laboratorio. Cuando haya cargado el archivo VHD, en la sección de [pasos siguientes](#next-steps) se muestran algunos artículos que ilustran cómo crear una imagen personalizada a partir del archivo VHD cargado. Para más información sobre los discos y los discos duros virtuales en Azure, consulte [Introducción a Azure Managed Disks](../virtual-machines/managed-disks-overview.md)

> [!NOTE] 
>  
> AzCopy es una utilidad de línea de comandos solo de Windows.

## <a name="step-by-step-instructions"></a>Instrucciones paso a paso

Los siguientes pasos le guían en la carga de un archivo VHD en Azure DevTest Labs mediante [AzCopy](https://aka.ms/downloadazcopy). 

1. Obtenga el nombre de la cuenta de almacenamiento del laboratorio mediante el portal de Azure:

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.

1. En la lista de laboratorios, seleccione el laboratorio que desee.  

1. En la hoja del laboratorio, seleccione **Configuración**. 

1. En la hoja **Configuración** del laboratorio, seleccione **Custom images (VHDs)** (Imágenes personalizadas [VHD]).

1. En la hoja **Custom images** (Imágenes personalizadas), seleccione **+Agregar**. 

1. En la hoja **Custom images** (Imágenes personalizadas), seleccione **+Agregar**.

1. En la hoja **VHD**, seleccione **Upload a VHD using PowerShell** (Cargar un VHD mediante PowerShell).

    ![Carga del VHD mediante PowerShell](./media/devtest-lab-upload-vhd-using-azcopy/upload-image-using-psh.png)

1. La hoja **Upload an image using PowerShell** (Cargar una imagen mediante PowerShell) muestra una llamada al cmdlet **Add-AzureVhd**. El primer parámetro (*Destination*) contiene el URI de un contenedor de blobs (*uploads*) en el siguiente formato:

    ```
    https://<STORAGE-ACCOUNT-NAME>.blob.core.windows.net/uploads/...
    ``` 

1. Anote el URI completo tal como se usa en pasos posteriores.

1. Carga del archivo VHD mediante AzCopy:
 
1. [Descargue e instale la versión más reciente de AzCopy](https://aka.ms/downloadazcopy).

1. Abra una ventana de comandos y vaya al directorio de instalación de AzCopy. Opcionalmente, puede agregar la ubicación de instalación de AzCopy a la ruta de acceso del sistema. De forma predeterminada, AzCopy se instala en el directorio siguiente:

    ```command-line
    %ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy
    ```

1. Mediante la clave de la cuenta de almacenamiento y el URI del contenedor de blobs, ejecute el siguiente comando en el símbolo del sistema. El valor *vhdFileName* debe incluirse entre comillas. El proceso de cargar un archivo VHD puede ser largo en función de su tamaño y de la velocidad de conexión.   

    ```command-line
    AzCopy /Source:<sourceDirectory> /Dest:<blobContainerUri> /DestKey:<storageAccountKey> /Pattern:"<vhdFileName>" /BlobType:page
    ```

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una imagen personalizada en Azure DevTest Labs a partir de un archivo VHD mediante el portal de Azure](devtest-lab-create-template.md)
- [Creación de una imagen personalizada en Azure DevTest Labs a partir de un archivo VHD mediante PowerShell](devtest-lab-create-custom-image-from-vhd-using-powershell.md)