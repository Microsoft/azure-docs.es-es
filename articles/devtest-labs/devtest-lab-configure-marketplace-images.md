---
title: Configuración de imágenes de Azure Marketplace en Azure DevTest Labs
description: Configuración de las imágenes de Azure Marketplace que se pueden usar al crear una máquina virtual en Azure DevTest Labs
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: a5158ed33bf253db1dbe0eb3232bc43d27ce15e7
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86512442"
---
# <a name="configure-azure-marketplace-image-settings-in-azure-devtest-labs"></a>Configuración de imágenes de Azure Marketplace en Azure DevTest Labs
DevTest Labs admite la creación de máquinas virtuales basadas en imágenes de Azure Marketplace, en función de cómo se hayan configurado las imágenes de Azure Marketplace para usarlas en el laboratorio. En este artículo se muestra cómo especificar qué imágenes de Azure Marketplace se pueden utilizar al crear máquinas virtuales en un laboratorio, si se puede usar alguna. Esto garantiza que el equipo solo tenga acceso a las imágenes de Marketplace que necesitan. 

## <a name="specify-allowed-images-for-creating-vms"></a>Especificación de imágenes permitidas para crear máquinas virtuales
Siga estos pasos para especificar las imágenes de Azure Marketplace que se permiten al crear una máquina virtual. 

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
2. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
3. En la lista de laboratorios, seleccione el suyo. 
4. En la página principal de la página del laboratorio, seleccione **Configuración y directivas**.
5. En la página **Configuración y directivas** del laboratorio de **Bases de datos de las máquinas virtuales**, seleccione **Imágenes de Marketplace**.
6. Especifique si desea que todas las imágenes cualificadas de Azure Marketplace estén disponibles para su uso como base de una nueva máquina virtual. Si selecciona **Sí**, se permitirán en el laboratorio todas las imágenes de Azure Marketplace que cumplan los criterios siguientes:
   
   * La imagen crea una única máquina virtual **y**
   * La imagen utiliza Azure Resource Manager para aprovisionar máquinas virtuales **y**
   * La imagen no requiere la compra de un plan de licencias adicional.
     
     Si desea que no se permitan imágenes o desea especificar las imágenes que se pueden utilizar, seleccione **No**.
     
     ![Opción para permitir que todas las imágenes de Marketplace se usen como imágenes base para las máquinas virtuales](./media/devtest-lab-configure-marketplace-images/allow-all-marketplace-images.png)
7. Si ha seleccionado **No** en el paso anterior, se habilita la casilla **Allowed images/Select all** (Imágenes permitidas/seleccionar todas). 
   Puede utilizar esta opción junto con el cuadro de búsqueda para seleccionar o anular la selección de todos los elementos que se muestran en la lista rápidamente.
   * Seleccione individualmente las imágenes de Azure Marketplace que desea permitir para la creación de máquinas virtuales. Para ello, active la casilla correspondiente a cada imagen.
   * No seleccione nada en la lista si no desea permitir que las imágenes de Azure Marketplace se usen en el laboratorio.
   
     ![Puede especificar qué imágenes de Azure Marketplace se pueden usar como imágenes base para las máquinas virtuales](./media/devtest-lab-configure-marketplace-images/select-marketplace-images.png)


## <a name="troubleshoot"></a>Solución de problemas
Si no puede encontrar una imagen específica a fin de habilitarla para el laboratorio, siga estos pasos: 

- Compruebe si puede ver la imagen mientras crea una máquina virtual de proceso.
- Es posible que la imagen no esté disponible en el tipo de la suscripción que use. Compruebe, junto con el administrador de la suscripción, el tipo de la suscripción (por ejemplo: MSDN, gratis, pago por uso, etc.). 
- La compatibilidad con imágenes de Gen 2 en DevTest Labs es limitada. Si hay versiones de Gen 1 y Gen 2 disponibles para una imagen, DevTest Labs solo muestra la versión de Gen 1 de la imagen al crear una máquina virtual. La solución alternativa consiste en crear una imagen de Gen 2 personalizada fuera del laboratorio y usarla para crear una máquina virtual. Si solo se tiene acceso a la versión de Gen 2 de la imagen disponible, DevTest Labs la admitirá y mostrará en la lista. 
      


## <a name="next-steps"></a>Pasos siguientes
Una vez que haya configurado cómo se permiten las imágenes de Azure Marketplace al crear una máquina virtual, el siguiente paso es [agregar una máquina virtual al laboratorio](devtest-lab-add-vm.md).

