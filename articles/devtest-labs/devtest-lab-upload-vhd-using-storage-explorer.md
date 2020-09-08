---
title: Carga de archivos VHD en Azure DevTest Labs mediante el Explorador de Storage
description: Carga de un archivo VHD en la cuenta de almacenamiento del laboratorio mediante el Explorador de Microsoft Azure Storage
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 6ed6ace63a135628b6d306b946c0356091bf1bd2
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654521"
---
# <a name="upload-vhd-file-to-labs-storage-account-using-microsoft-azure-storage-explorer"></a>Carga de un archivo VHD en la cuenta de almacenamiento del laboratorio mediante el Explorador de Microsoft Azure Storage

[!INCLUDE [devtest-lab-upload-vhd-selector](../../includes/devtest-lab-upload-vhd-selector.md)]

En Azure DevTest Labs, se pueden usar archivos VHD para crear imágenes personalizadas, que se usan para aprovisionar máquinas virtuales. En este artículo se ilustra cómo usar el [Explorador de Microsoft Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md) para cargar un archivo VHD en una cuenta de almacenamiento del laboratorio. Cuando haya cargado el archivo VHD, en la sección de [pasos siguientes](#next-steps) se muestran algunos artículos que ilustran cómo crear una imagen personalizada a partir del archivo VHD cargado. Para más información sobre los discos y los discos duros virtuales en Azure, consulte [Introducción a Azure Managed Disks](../virtual-machines/managed-disks-overview.md)

## <a name="step-by-step-instructions"></a>Instrucciones paso a paso

Los siguientes pasos le guían en la carga de un archivo VHD en DevTest Labs mediante el [Explorador de Microsoft Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md).

1. [Descargue e instale la versión más reciente del Explorador de Microsoft Azure Storage](https://www.storageexplorer.com).

1. Obtenga el nombre de la cuenta de almacenamiento del laboratorio mediante el portal de Azure:

    1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
    
    1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
    
    1. En la lista de laboratorios, seleccione el laboratorio que desee.  
    
    1. En la hoja del laboratorio, seleccione **Configuración**. 
    
    1. En la hoja **Configuración** del laboratorio, seleccione **Custom images (VHDs)** (Imágenes personalizadas [VHD]).
    
    1. En la hoja **Custom images** (Imágenes personalizadas), seleccione **+Agregar**. 
    
    1. En la hoja **Custom images** (Imágenes personalizadas), seleccione **+Agregar**.
    
    1. En la hoja **VHD**, seleccione **Upload a VHD using PowerShell** (Cargar un VHD mediante PowerShell).
    
        ![Carga del VHD mediante PowerShell][0]
    
    1. La hoja **Upload an image using PowerShell** (Cargar una imagen mediante PowerShell) muestra una llamada al cmdlet **Add-AzureVhd**. El primer parámetro (*Destination*) contiene el nombre de la cuenta de almacenamiento del laboratorio en el siguiente formato:
    
        `https://<STORAGE-ACCOUNT-NAME>.blob.core.windows.net/uploads/...`

    1. Anote el nombre de la cuenta de almacenamiento tal como se ha usado en los últimos pasos.
    
1. Conéctese a una cuenta de suscripción de Azure mediante el Explorador de Storage.

    > [!TIP] 
    > 
    > El Explorador de Storage admite varias opciones de conexión. En esta sección se ilustra la conexión a una cuenta de almacenamiento asociada a la suscripción de Azure. Para ver las demás opciones de conexión admitidas por el Explorador de Storage, consulte el artículo [Introducción al Explorador de Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md).
 
    1. Abra el Explorador de Storage.
    
    1. En el Explorador de Storage, seleccione **Azure Account settings** (Configuración de la cuenta de Azure). 
    
        ![Configuración de la cuenta de Azure][1]
    
    1. El panel izquierdo muestra todas las cuentas de Microsoft en las que ha iniciado sesión. Para conectarse a otra cuenta, seleccione **Agregar una cuenta**y siga los cuadros de diálogo para iniciar sesión con una cuenta de Microsoft que esté asociada con al menos una suscripción activa de Azure.
    
        ![Agregar una cuenta][2]
    
    1. Una vez que haya iniciado sesión correctamente con una cuenta de Microsoft, el panel izquierdo se llena con las suscripciones de Azure asociadas a dicha cuenta. Elija las suscripciones de Azure con las que desea trabajar y seleccione **Aplicar**. (Al seleccionar **All subscriptions** (Todas las suscripciones), se alterna la selección entre todas o ninguna de las suscripciones de Azure que aparecen).
    
        ![Selección de suscripciones de Azure][3]
    
    1. El panel izquierdo muestra ahora las cuentas de almacenamiento asociadas a las suscripciones de Azure seleccionadas.
    
        ![Suscripciones de Azure seleccionadas][4]

1. Busque la cuenta de almacenamiento del laboratorio:

    1. En el panel izquierdo del Explorador de Storage, busque y expanda el nodo de la suscripción de Azure que posee el laboratorio.
    
    1. En el nodo de la suscripción, expanda **Cuentas de Storage**.

    1. Expanda el nodo de cuenta de almacenamiento del laboratorio para mostrar los nodos **Contenedores de blobs**, **Recursos compartidos de archivos**, **Colas** y **Tablas**.
    
    1. Expanda el nodo **Contenedores de blobs**.
    
    1. Seleccione el contenedor de blobs de cargas para mostrar su contenido en el panel derecho.
        
        ![Directorio de carga][5]

1. Cargue el archivo VHD mediante el Explorador de Storage:

    1. En el panel derecho del Explorador de Storage, verá una lista de los blobs en el contenedor de blobs **uploads** de la cuenta de almacenamiento del laboratorio. En la barra de herramientas del editor de blobs, seleccione **Cargar** 
        
        ![Botón Cargar][6]
    
    1. En el menú desplegable **Cargar**, seleccione **Upload files...** (Cargar archivos...).
    
    1. En el diálogo **Upload files** (Cargar archivos), seleccione los puntos suspensivos.
        
        ![Selección del archivo][8]  

    1. En el diálogo **Select files to upload** (Seleccionar archivos para cargar), vaya al archivo VHD deseado, selecciónelo y luego haga clic en **Abrir**.
    
    1. Cuando regrese al diálogo **Upload files** (Cargar archivos), cambie el **tipo de blob** por **Blob en páginas**.
    
    1. Seleccione **Cargar**.

        ![Selección del archivo][9]  
    
    1. El panel **Activity Log** (Registro de actividad) muestra el estado de descarga (junto con vínculos para cancelar la carga). El proceso de cargar un archivo VHD puede ser largo en función de su tamaño y de la velocidad de conexión. 

        ![Estado del archivo de carga][10]  

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una imagen personalizada en Azure DevTest Labs a partir de un archivo VHD mediante el portal de Azure](devtest-lab-create-template.md)
- [Creación de una imagen personalizada en Azure DevTest Labs a partir de un archivo VHD mediante PowerShell](devtest-lab-create-custom-image-from-vhd-using-powershell.md)

[0]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-image-using-psh.png
[1]: ./media/devtest-lab-upload-vhd-using-storage-explorer/settings-icon.png
[2]: ./media/devtest-lab-upload-vhd-using-storage-explorer/add-account-link.png
[3]: ./media/devtest-lab-upload-vhd-using-storage-explorer/subscriptions-list.png
[4]: ./media/devtest-lab-upload-vhd-using-storage-explorer/storage-accounts-list.png
[5]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-dir.png
[6]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-button.png
[7]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-files.png
[8]: ./media/devtest-lab-upload-vhd-using-storage-explorer/select-file.png
[9]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-file.png
[10]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-status.png
