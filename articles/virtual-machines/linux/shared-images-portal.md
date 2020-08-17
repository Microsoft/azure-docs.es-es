---
title: Creación de imágenes de máquina virtual Linux de Azure compartidas mediante el portal
description: Aprenda a usar Azure Portal para crear y compartir imágenes de máquina virtual Linux.
author: cynthn
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: d49bbebe4d38783342170693bd06f0dfc645f34d
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87288403"
---
# <a name="create-an-azure-shared-image-gallery-using-the-portal"></a>Creación de una instancia de Azure Shared Image Gallery mediante el portal

Una [galería de imágenes compartidas](shared-image-galleries.md) simplifica el uso compartido de imágenes personalizadas en toda una organización. Las imágenes personalizadas son como las imágenes de Marketplace, pero las puede crear usted mismo. Las imágenes personalizadas se pueden usar para realizar tareas de implementación de arranque, como la carga previa de aplicaciones, configuraciones de aplicaciones y otras configuraciones del sistema operativo. 

La Galería de imágenes compartidas le permite compartir sus imágenes de máquina virtual personalizadas con otros usuarios de su organización, ya sea dentro o entre regiones, dentro de un inquilino de AAD. Elija las imágenes que desea compartir, qué regiones desea que estén disponibles en ellas y con quién desea compartirlas. Puede crear varias galerías que le permitirán agrupar lógicamente las imágenes compartidas. 

La galería es un recurso de nivel superior que proporciona control de acceso basado en roles (RBAC). Las imágenes pueden tener varias versiones y se puede optar por replicar cada versión de la imagen en un conjunto diferente de regiones de Azure. La galería solo funciona con imágenes administradas.

La característica de galería de imágenes compartidas tiene varios tipos de recursos. En este artículo, usaremos o crearemos los siguientes elementos:


[!INCLUDE [virtual-machines-shared-image-gallery-resources](../../../includes/virtual-machines-shared-image-gallery-resources.md)]

<br>





## <a name="before-you-begin"></a>Antes de empezar

Para completar el ejemplo de este artículo, debe tener una imagen administrada existente de una máquina virtual generalizada o una instantánea de una máquina virtual especializada. Puede seguir [Tutorial: Cree una imagen personalizada de una máquina virtual de Azure con Azure PowerShell](tutorial-custom-images.md) para crear una imagen administrada, o bien [cree una instantánea](../windows/snapshot-copy-managed-disk.md) para una máquina virtual especializada. Ni en las instantáneas ni en las imágenes administradas, el tamaño del disco de datos no puede ser superior a 1 TB.

Al trabajar en este artículo, reemplace los nombres de grupo de recursos y máquina virtual cuando proceda.

 
[!INCLUDE [virtual-machines-common-shared-images-portal](../../../includes/virtual-machines-common-shared-images-portal.md)]

## <a name="create-vms"></a>Creación de máquinas virtuales 

Ahora puede crear una o varias máquinas virtuales. Este ejemplo crea una máquina virtual denominada *myVMfromImage*, en *myResourceGroup* en el centro de datos del *Este de EE. UU.*

1. Vaya a la definición de la imagen. Puede usar el filtro de recursos para mostrar todas las definiciones de imagen disponibles.
1. En la página de la definición de la imagen, seleccione **Crear máquina virtual** en el menú de la parte superior de la página.
1. En **Grupo de recursos**, seleccione **Crear nuevo** y escriba *myResourceGroup* como nombre.
1. En **Nombre de máquina virtual**, escriba *myVM*.
1. En **Región**, seleccione *Este de EE. UU.* .
1. En **Opciones de disponibilidad**, deje el valor predeterminado *No se requiere redundancia de la infraestructura*.
1. El valor de **Imagen** se completa automáticamente con la versión de imagen `latest` si el inicio se realizó desde la página para la definición de la imagen.
1. En **Tamaño**, elija un tamaño de máquina virtual en la lista de tamaños disponibles y, después, elija **Seleccionar**.
1. En **Cuenta de administrador**, si la máquina virtual de origen era generalizada, escriba el **nombre de usuario**  y la **clave pública SSH**. Si la máquina virtual de origen era especializada, estas opciones se atenuarán porque se usa la información de la máquina virtual de origen.
1. Si desea permitir el acceso remoto a la máquina virtual, en **Puertos de entrada públicos**, elija **Permitir puertos seleccionados** y, a continuación, seleccione **SSH (22)** en la lista desplegable. Si no desea permitir el acceso remoto a la máquina virtual, deje **Ninguno** seleccionado en **Puertos de entrada públicos**.
1. Cuando haya terminado, seleccione el botón **Revisar y crear** situado en la parte inferior de la página.
1. Una vez que la máquina virtual haya superado la validación, seleccione **Crear** en la parte inferior de la página para iniciar la implementación.


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar el grupo de recursos, la máquina virtual y todos los recursos relacionados. Para ello, seleccione el grupo de recursos de la máquina virtual, seleccione **Eliminar** y luego confirme el nombre del grupo de recursos para eliminar.

Si desea eliminar recursos individuales, deberá eliminarlos en el orden inverso. Por ejemplo, para eliminar una definición de la imagen, deberá eliminar todas las versiones de la imagen creadas a partir de esa imagen.

## <a name="next-steps"></a>Pasos siguientes

Puede crear también recursos de galería de imágenes compartidas con plantillas. Hay varias plantillas de Inicio rápido de Azure disponibles: 

- [Creación de una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Creación de una definición de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Creación de una versión de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
- [Creación de una máquina virtual a partir de la versión de la imagen](https://azure.microsoft.com/resources/templates/101-vm-from-sig/)

Para más información sobre las galerías de imágenes compartidas, consulte la [Introducción](shared-image-galleries.md). Si encuentra problemas, consulte [Solución de problemas de galerías de imágenes compartidas](troubleshooting-shared-images.md).

