---
title: Solución de los errores que aparecen al eliminar cuentas de almacenamiento clásico, contenedores o VHD de Azure | Microsoft Docs
description: Solución de problemas al eliminar recursos de almacenamiento que contienen discos duros virtuales conectados.
services: storage
author: AngshumanNayakMSFT
tags: top-support-issue,azure-service-management
ms.service: storage
ms.topic: troubleshooting
ms.date: 01/11/2019
ms.author: annayak
ms.openlocfilehash: 3e7469f0d53a154f605480b811d36937e3d4ad6c
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88649864"
---
# <a name="troubleshoot-classic-storage-resource-deletion-errors"></a>Solución de errores de eliminación de recursos de almacenamiento clásico
En este artículo se proporcionan instrucciones para solucionar problemas cuando se produce uno de los siguientes errores al intentar eliminar un archivo de blob de páginas *.vhd, un contenedor o una cuenta de almacenamiento clásico de Azure. 


En este artículo solo se tratan los problemas de los recursos de almacenamiento clásico. Si un usuario elimina una máquina virtual clásica mediante Azure Portal, PowerShell o la CLI, los discos no se eliminan automáticamente. El usuario dispone de la opción para eliminar el recurso "Disco". En caso de que la opción no se seleccione, el recurso "Disco" impedirá la eliminación de la cuenta de almacenamiento, el contenedor y el archivo de blob de páginas *.vhd real.

Se puede encontrar más información sobre los discos de Azure [aquí](../../virtual-machines/managed-disks-overview.md). Azure impide la eliminación de un disco que esté asociado a una máquina virtual para evitar daños. También impide la eliminación de los contenedores y las cuentas de almacenamiento que tengan un blob de páginas que esté asociado a una máquina virtual. 

## <a name="what-is-a-disk"></a>¿Qué es un "Disco"?
Un recurso "Disco" se utiliza para montar un archivo de blob de páginas *.vhd en una máquina virtual, como un disco del sistema operativo o un disco de datos. Hasta que se elimine un recurso de disco de datos o de disco del sistema operativo, seguirá manteniendo una concesión en el archivo *.vhd. Ningún recurso de almacenamiento en la ruta de acceso que se muestre en la imagen de abajo se puede eliminar si un recurso "Disco" hace referencia a él.

![Captura de pantalla del portal, con el panel "Propiedad" de disco (clásico) abierto](./media/storage-classic-cannot-delete-storage-account-container-vhd/Disk_Lease_Illustration.jpg) 


## <a name="steps-while-deleting-a-classic-virtual-machine"></a>Pasos al eliminar una máquina virtual clásica 

[!INCLUDE [classic-vm-deprecation](../../../includes/classic-vm-deprecation.md)]


1. Elimine la máquina virtual clásica.
2. Si se selecciona la casilla de verificación "Discos", la **concesión del disco** (que se muestra en la imagen anterior) asociada con el blob en páginas *.vhd se interrumpe. El archivo *.vhd de blob en páginas real todavía existirá en la cuenta de almacenamiento.
![Captura de pantalla del portal, con el panel de errores "Eliminar" de la máquina virtual (clásica)](./media/storage-classic-cannot-delete-storage-account-container-vhd/steps_while_deleting_classic_vm.jpg) 

3. Una vez que se interrumpe la concesión de los discos, se pueden eliminar los propios blobs en páginas. Una cuenta de almacenamiento o un contenedor se pueden eliminar una vez que se eliminan todos los recursos "Disco" presentes en ellos.

>[!NOTE] 
>Si el usuario elimina la máquina virtual pero no el disco duro virtual, se seguirán acumulando cargos por almacenamiento en el archivo *.vhd del blob en páginas. Los cargos estarán en consonancia con el tipo de cuenta de almacenamiento; consulte la [página de precios](https://azure.microsoft.com/pricing/details/storage/) para obtener más detalles. Si el usuario ya no va a utilizar los discos duros virtuales, elimínelos para evitar cargos futuros. 

## <a name="unable-to-delete-storage-account"></a>No se puede eliminar la cuenta de almacenamiento 

Cuando el usuario intenta eliminar una cuenta de almacenamiento clásico que ya no se necesita, puede observar el siguiente comportamiento.

#### <a name="azure-portal"></a>Portal de Azure 
Si el usuario va a la cuenta de almacenamiento clásico en [Azure Portal](https://portal.azure.com) y hace clic en **Eliminar**, verá un mensaje similar al siguiente: 

Con discos "asociados" a una máquina virtual

![Captura de pantalla del portal, con el panel de errores "Eliminar" de la máquina virtual (clásica)](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_storage_account_disks_attached_portal.jpg) 


Con discos "no asociados" a una máquina virtual

![Captura de pantalla del portal, con el panel sin errores "Eliminar" de la máquina virtual (clásica)](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_storage_account_disks_unattached_portal.jpg)


#### <a name="azure-powershell"></a>Azure PowerShell
El usuario intenta eliminar una cuenta de almacenamiento que ya no se está usando, mediante el uso de cmdlets de PowerShell clásico. Verá el siguiente mensaje:

> <span style="color:cyan">**Remove-AzureStorageAccount -StorageAccountName myclassicaccount**</span>
> 
> <span style="color:red">Remove-AzureStorageAccount: BadRequest: La cuenta de almacenamiento myclassicaccount tiene algunas imágenes o discos activos, por ejemplo,  
> myclassicaccount. Controle que se quiten las imágenes y los discos antes de eliminar esta cuenta de almacenamiento.</span>

## <a name="unable-to-delete-storage-container"></a>No se pudo eliminar el contenedor de almacenamiento

Cuando el usuario intenta eliminar un contenedor de blog de almacenamiento clásico que ya no se necesita, puede observar el siguiente comportamiento.

#### <a name="azure-portal"></a>Portal de Azure 
Azure Portal no permite al usuario eliminar un contenedor si no existe una concesión "Discos" que apunta a un archivo de blob en páginas *.vhd en el contenedor. Esto es así intencionadamente, para impedir la eliminación accidental de un archivo VHD con concesión de discos. 

![Captura de pantalla del portal, con el panel "lista" del contenedor de almacenamiento abierto](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_container_portal.jpg)


#### <a name="azure-powershell"></a>Azure PowerShell
Si el usuario decide eliminar mediante PowerShell, se producirá el siguiente error. 

> <span style="color:cyan">**Remove-AzureStorageContainer -Context $context -Name vhds**</span>
> 
> <span style="color:red">Remove-AzureStorageContainer: Error en el servidor remoto: (412) Actualmente hay una concesión en el contenedor y no se especificó ningún identificador de concesión en la solicitud. Código de estado HTTP: 412 - Mensaje de error HTTP: Actualmente hay una concesión en el contenedor y no se especificó ningún identificador de concesión en la solicitud.</span>

## <a name="unable-to-delete-a-vhd"></a>No se puede eliminar un disco duro virtual 

Después de eliminar la máquina virtual de Azure, el usuario intenta eliminar el archivo de disco duro virtual (blob en páginas) y recibe el mensaje siguiente:

#### <a name="azure-portal"></a>Portal de Azure 
En el portal, podría haber dos experiencias, en función de la lista de blobs seleccionados para eliminarse.

1. Si solo se han seleccionado los blobs "Concedidos", el botón Eliminar no aparece.
![Captura de pantalla del portal, con el panel "lista" del blob del contenedor abierto](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_vhd_leased_portal.jpg)


2. Si se selecciona una mezcla de blobs "Concedido" y "Disponible", aparece el botón "Eliminar". Pero la operación "Eliminar" dejará los blobs en páginas, que tienen una concesión de disco en ellos. 
![Captura de pantalla del portal, con el panel "lista" del blob de contenedor abierto](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_vhd_leased_and_unleased_portal_1.jpg)
![Screenshot of the portal, with the selected blob "delete" pane open](./media/storage-classic-cannot-delete-storage-account-container-vhd/unable_to_delete_vhd_leased_and_unleased_portal_2.jpg)

#### <a name="azure-powershell"></a>Azure PowerShell 
Si el usuario decide eliminar mediante PowerShell, se producirá el siguiente error. 

> <span style="color:cyan">**Remove-AzureStorageBlob -Context $context -Container vhds -Blob "classicvm-os-8698.vhd"** </span>
> 
> <span style="color:red">Remove-AzureStorageBlob : Error en el servidor remoto: (412) Actualmente hay una concesión en el blob y no se especificó ningún identificador de concesión en la solicitud. Código de estado HTTP: 412 - Mensaje de error HTTP: Actualmente hay una concesión en el blob y no se especificó ningún identificador de concesión en la solicitud.</span>


## <a name="resolution-steps"></a>Pasos de resolución

### <a name="to-remove-classic-disks"></a>Para quitar discos clásicos
Siga estos pasos en Azure Portal:
1.  Acceda a [Azure Portal](https://portal.azure.com).
2.  Vaya a los discos (clásicos). 
3.  Haga clic en la pestaña Discos. ![Captura de pantalla del portal, con el panel "lista" del blob del contenedor abierto](./media/storage-classic-cannot-delete-storage-account-container-vhd/resolution_click_disks_tab.jpg)
 
4.  Seleccione el disco de datos y luego haga clic en Eliminar disco.
 ![Captura de pantalla del portal, con el panel "lista" del blob del contenedor abierto](./media/storage-classic-cannot-delete-storage-account-container-vhd/resolution_click_delete_disk.jpg)
 
5.  Vuelva a intentar la operación de eliminación que no se pudo realizar antes.
6.  No se puede eliminar una cuenta de almacenamiento ni un contenedor mientras tenga algún disco.

### <a name="to-remove-classic-images"></a>Para quitar imágenes clásicas   
Siga estos pasos en Azure Portal:
1.  Acceda a [Azure Portal](https://portal.azure.com).
2.  Vaya a las imágenes de sistema operativo (clásicas).
3.  Elimine la imagen.
4.  Vuelva a intentar la operación de eliminación que no se pudo realizar antes.
5.  No se puede eliminar una cuenta de almacenamiento ni un contenedor mientras tenga alguna imagen.
