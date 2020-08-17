---
title: Creación de una imagen personalizada de Azure DevTest Labs a partir de una máquina virtual | Microsoft Azure
description: Aprenda a crear una imagen personalizada en Azure DevTest Labs a partir de una máquina virtual aprovisionada mediante el portal de Azure.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: ad45ed6eb7f97e14ec0ca0bb89efb2967c90fc16
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87277034"
---
# <a name="create-a-custom-image-from-a-vm"></a>Crear una imagen personalizada a partir de una máquina virtual

[!INCLUDE [devtest-lab-custom-image-definition](../../includes/devtest-lab-custom-image-definition.md)]

## <a name="step-by-step-instructions"></a>Instrucciones paso a paso

Puede crear una imagen personalizada a partir de una máquina virtual aprovisionada y luego usar esa imagen personalizada para crear máquinas virtuales idénticas. Los siguientes pasos muestran cómo crear una imagen personalizada desde una máquina virtual:

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.

1. En la lista de laboratorios, seleccione el laboratorio que desee.  

1. En el panel principal del laboratorio, seleccione **Mis máquinas virtuales**.
 
1. En el panel **Mis máquinas virtuales**, seleccione la máquina virtual desde la que quiere crear la imagen personalizada.

1. En el panel de administración de la máquina virtual, seleccione **Create custom image (VHD)** [Crear imagen personalizada (VHD)] bajo **OPERATIONS** (OPERACIONES).

    :::image type="content" source="./media/devtest-lab-create-template/create-custom-image.png" alt-text="Creación del elemento de menú de imagen personalizada":::
1. En el panel **Imagen personalizada**, escriba un nombre y una descripción para la imagen personalizada. Esta información se muestra en la lista de bases cuando se crea una máquina virtual. La imagen personalizada incluirá el disco del SO y todos los discos de datos asociados a la máquina virtual.

    :::image type="content" source="./media/devtest-lab-create-template/create-custom-image-blade.png" alt-text="Creación de una página de imagen personalizada":::
1. Seleccione si sysprep se ha ejecutado en la máquina virtual. Si no se ha ejecutado sysprep en la máquina virtual, especifique si quiere que sysprep se ejecute en la máquina virtual cuando se cree la imagen personalizada.
1. Cuando termine de crear la imagen personalizada, seleccione **Aceptar** .

    Después de unos minutos, la imagen personalizada se crea y se almacena dentro de la cuenta de almacenamiento del laboratorio. Cuando un usuario de laboratorio quiera crear una nueva máquina virtual, la imagen estará disponible en la lista de imágenes base.

    :::image type="content" source="./media/devtest-lab-create-template/custom-image-available-as-base.png" alt-text="Imagen personalizada disponible en la lista de imágenes base":::

## <a name="related-blog-posts"></a>Entradas blogs relacionadas

- [Custom images or formulas? (¿Imágenes personalizadas o fórmulas?)](./devtest-lab-faq.md#blog-post)
- [Copying Custom Images between Azure DevTest Labs (Copiar imágenes personalizadas entre instancias de Azure DevTest Labs)](https://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>Pasos siguientes

- [Agregar una máquina virtual al laboratorio](devtest-lab-add-vm.md)
