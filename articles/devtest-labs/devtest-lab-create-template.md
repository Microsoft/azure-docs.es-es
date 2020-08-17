---
title: Creación de una imagen personalizada de Azure DevTest Labs a partir de un archivo VHD | Microsoft Azure
description: Aprenda a crear una imagen personalizada en Azure DevTest Labs a partir de un archivo VHD mediante el portal de Azure.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: cac812a9c38fc1dedfd31659a626b122f9527e63
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289410"
---
# <a name="create-a-custom-image-from-a-vhd-file"></a>Crear una imagen personalizada a partir de un archivo VHD

[!INCLUDE [devtest-lab-create-custom-image-from-vhd-selector](../../includes/devtest-lab-create-custom-image-from-vhd-selector.md)]

[!INCLUDE [devtest-lab-custom-image-definition](../../includes/devtest-lab-custom-image-definition.md)]

[!INCLUDE [devtest-lab-upload-vhd-options](../../includes/devtest-lab-upload-vhd-options.md)]

## <a name="step-by-step-instructions"></a>Instrucciones paso a paso

Los siguientes pasos le guían en la creación de una imagen personalizada a partir de un archivo VHD mediante el portal de Azure:

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.

1. En la lista de laboratorios, seleccione el laboratorio que desee.  

1. En el panel principal del laboratorio, seleccione **Configuración y directivas**. 

1. En el panel **Configuración y directivas**, seleccione **Imágenes personalizadas**.

1. En el panel **Imágenes personalizadas**, seleccione **+Agregar**.

    ![Adición de imágenes personalizadas](./media/devtest-lab-create-template/add-custom-image.png)

1. Escriba el nombre de la imagen personalizada. Este nombre se muestra en la lista de imágenes base al crear una máquina virtual.

1. Escriba la descripción de la imagen personalizada. Esta descripción se muestra en la lista de imágenes base al crear una máquina virtual.

1. En **Tipo de SO**, seleccione **Windows** o **Linux**.

    - Si selecciona **Windows**, especifique mediante la casilla si se ha ejecutado *sysprep* en la máquina. 
    - Si selecciona **Linux**, especifique mediante la casilla si se ha ejecutado *deprovision* en la máquina. 

1. Seleccione un **VHD** en el menú desplegable. Es el disco duro virtual que se usará para crear la nueva imagen personalizada. Si es necesario, seleccione **Cargar un VHD con PowerShell**.

1. También puede escribir el nombre, la oferta y el publicador del plan si la imagen que se usa para crear la imagen personalizada no es una imagen con licencia (publicada por Microsoft).

   - **Nombre del plan:** escriba el nombre de la imagen de Marketplace (SKU) a partir de la cual se crea esta imagen personalizada 
   - **Oferta del plan:** escriba el producto (oferta) de la imagen de Marketplace a partir de la cual se crea esta imagen personalizada 
   - **Publicador del plan:** escriba el publicador de la imagen de Marketplace a partir de la cual se crea esta imagen personalizada

   > [!NOTE]
   > Si la imagen que usa para crear una imagen personalizada **no** es una imagen con licencia, estos campos están vacíos y se pueden rellenar si así lo decide. Si la imagen **es** una imagen con licencia, los campos se rellenan automáticamente con la información del plan. Si intenta cambiarlos en este caso, aparecerá un mensaje de advertencia.
   >
   >

1. Seleccione **Aceptar** para crear la imagen personalizada.

Después de unos minutos, la imagen personalizada se crea y se almacena dentro de la cuenta de almacenamiento del laboratorio. Cuando un usuario de laboratorio quiera crear una nueva máquina virtual, la imagen estará disponible en la lista de imágenes base.

![Imagen personalizada disponible en la lista de imágenes base](./media/devtest-lab-create-template/custom-image-available-as-base.png)


[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="related-blog-posts"></a>Entradas blogs relacionadas

- [Custom images or formulas? (¿Imágenes personalizadas o fórmulas?)](./devtest-lab-faq.md#blog-post)
- [Copying Custom Images between Azure DevTest Labs (Copiar imágenes personalizadas entre instancias de Azure DevTest Labs)](https://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>Pasos siguientes

- [Agregar una máquina virtual al laboratorio](./devtest-lab-add-vm.md)
