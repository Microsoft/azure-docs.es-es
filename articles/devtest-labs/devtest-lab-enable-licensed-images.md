---
title: Habilitación de una imagen con licencia en su laboratorio en Azure DevTest Labs | Microsoft Docs
description: Aprenda a habilitar una imagen con licencia en Azure DevTest Labs mediante Azure Portal
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a043679e27580254c56fe118838dd355be5a58de
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286519"
---
# <a name="enable-a-licensed-image-in-your-lab-in-azure-devtest-labs"></a>Habilitación de una imagen con licencia en su laboratorio en Azure DevTest Labs

En Azure DevTest Labs, una imagen con licencia es la que incluye los términos y condiciones, normalmente de terceros, que deben aceptarse para que los usuarios del laboratorio puedan acceder a ella. Las secciones siguientes describen cómo trabajar con imágenes con licencia para que estén disponibles para su uso para crear máquinas virtuales.

## <a name="determining-whether-a-licensed-image-is-available-to-users"></a>Determinación de si una imagen con licencia está disponible para los usuarios
El primer paso para permitir a los usuarios crear máquinas virtuales desde una imagen con licencia es asegurarse de que esos términos y condiciones se han aceptado para la imagen con licencia. Los pasos siguientes muestran cómo puede ver el estado de la oferta de una imagen con licencia y, si fuese necesario, aceptar los términos y condiciones.

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.

1. En la lista de laboratorios, seleccione el laboratorio que desee.  

1. En el panel izquierdo, en **CONFIGURACIÓN**, seleccione **Configuración y directivas**.

1. En el panel izquierdo, en **BASES PARA MÁQUINAS VIRTUALES**, seleccione **Imágenes de Marketplace**. 

    ![Elemento de menú de imágenes de Marketplace](./media/devtest-lab-create-custom-image-from-licensed-image/devtest-lab-marketplace-images.png)

    Se muestra una lista de todas las imágenes de Marketplace disponibles, incluido el **ESTADO DE LA OFERTA** de cada imagen.

    ![Lista de imágenes de Marketplace que muestran el estado de la oferta para cada imagen](./media/devtest-lab-create-custom-image-from-licensed-image/devtest-lab-offer-status.png)

    Una imagen con licencia muestra un estado de la oferta de 
    
    - **Términos aceptados:** la imagen con licencia está disponible para que los usuarios creen máquinas virtuales. 
    - **Revisión de términos necesaria:** la imagen con licencia no está actualmente disponible para los usuarios. Deben aceptarse los términos y condiciones para que los usuarios del laboratorio puedan usarla para crear máquinas virtuales. 

## <a name="making-a-licensed-image-available-to-lab-users"></a>Imagen con licencia disponible para los usuarios del laboratorio
Para asegurarse de que una imagen con licencia está disponible para los usuarios del laboratorio, un propietario del laboratorio con permisos de administrador debe aceptar primero los términos y condiciones para esa imagen con licencia. Si se habilita a implementación mediante programación para la suscripción asociada a una imagen con licencia, se acepta automáticamente los términos legales y las declaraciones de privacidad para esa imagen. [Funcionamiento con imágenes de Marketplace en Azure Resource Manager](https://azure.microsoft.com/blog/working-with-marketplace-images-on-azure-resource-manager/) ofrece información adicional sobre la implementación mediante programación de imágenes de Marketplace.

Siga estos pasos para habilitar la implementación mediante programación para una imagen con licencia:

1. En [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040), vaya a la lista de **imágenes de Marketplace**.

1. Identifique una imagen con licencia para la que desee que los usuarios tengan acceso, pero cuyos términos no se hayan aceptado. Por ejemplo, podría ver una máquina virtual de ciencia de datos que muestra un estado de **Términos aceptados** o **Revisión de términos necesaria**.

    ![Configuración de la ventana de implementación mediante programación](./media/devtest-lab-create-custom-image-from-licensed-image/devtest-lab-licensed-images.png)

   > [!NOTE]
   > Las máquinas virtuales de ciencia de datos son imágenes de Azure Virtual Machines preinstaladas, configuradas y probadas con varias de las herramientas más populares que se usan habitualmente para análisis de datos, aprendizaje automático y entrenamiento con inteligencia artificial. La [Introducción a la máquina virtual de ciencia de datos de Azure ](../machine-learning/data-science-virtual-machine/overview.md) es una gran oportunidad para obtener información sobre DSVM.
   >
   >

1. En la columna **ESTADO DE LA OFERTA** para la imagen, seleccione **Revisión de términos necesaria**.

1. En la ventana Configurar la implementación mediante programación, seleccione **Habilitar**.

    ![Configuración de la ventana de implementación mediante programación](./media/devtest-lab-create-custom-image-from-licensed-image/devtest-lab-enable-programmatic-deployment.png)

   > [!IMPORTANT]
   > Puede que vea varias suscripciones en la ventana Configurar la implementación mediante programación. Asegúrese de que habilita la implementación mediante programación solo para la suscripción prevista.
   >
   >


1. Seleccione **Guardar**. 

    En la lista de imágenes de Marketplace, la imagen ahora muestra **Términos aceptados** y está disponible para que los usuarios creen máquinas virtuales.

> [!NOTE]
> Los usuarios pueden crear una imagen personalizada desde una imagen con licencia. Consulte [Crear una imagen personalizada a partir de un archivo VHD](devtest-lab-create-template.md) para más información.
>
>


## <a name="related-blog-posts"></a>Entradas blogs relacionadas

- [Custom images or formulas? (¿Imágenes personalizadas o fórmulas?)](./devtest-lab-faq.md#blog-post)
- [Copying Custom Images between Azure DevTest Labs (Copiar imágenes personalizadas entre instancias de Azure DevTest Labs)](https://www.visualstudiogeeks.com/blog/DevOps/How-To-Move-CustomImages-VHD-Between-AzureDevTestLabs#copying-custom-images-between-azure-devtest-labs)

## <a name="next-steps"></a>Pasos siguientes

- [Crear una imagen personalizada a partir de una máquina virtual](devtest-lab-create-custom-image-from-vm-using-portal.md)
- [Crear una imagen personalizada a partir de un archivo VHD](devtest-lab-create-template.md)
- [Agregar una máquina virtual al laboratorio](devtest-lab-add-vm.md)
