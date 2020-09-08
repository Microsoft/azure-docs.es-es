---
title: Asociación de un disco de datos, o su desasociación, a una máquina virtual en Azure DevTest Labs
description: Aprenda a asociar un disco de datos a una máquina virtual, o a desasociarlo, en Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 222bf06dc07cc2538bc0512b4a41b7bd91bc1ebf
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653926"
---
# <a name="attach-or-detach-a-data-disk-to-a-virtual-machine-in-azure-devtest-labs"></a>Asociación de un disco de datos, o su desasociación, a una máquina virtual en Azure DevTest Labs
[Azure Managed Disks](../virtual-machines/managed-disks-overview.md) administra las cuentas de almacenamiento asociadas a los discos de datos de máquina virtual. Un usuario asocia un nuevo disco de datos a una máquina virtual, especifica el tipo y el tamaño del disco que se necesita, y Azure crea y administra el disco automáticamente. El disco de datos se puede desasociar de la máquina virtual y volver a asociarlo más tarde a la misma máquina virtual, o bien se puede asociar a otra máquina virtual que pertenezca al mismo usuario.

Esta funcionalidad es útil para administrar el almacenamiento o el software fuera de cada máquina virtual individual. Si el almacenamiento o software ya existe dentro de un disco de datos, se puede asociar, desasociar y volver a asociar fácilmente a cualquier máquina virtual que sea propiedad del usuario que posee ese disco de datos.

## <a name="attach-a-data-disk"></a>Acoplamiento de un disco de datos
Antes de asociar un disco de datos a una máquina virtual, revise estas sugerencias:

- El tamaño de la máquina virtual controla cuántos discos de datos puede conectar. Para obtener más información, consulte [Tamaños de máquinas virtuales](../virtual-machines/sizes.md).
- Solo puede asociar un disco de datos a una máquina virtual que se está ejecutando. Asegúrese de que la máquina virtual se está ejecutando antes de intentar asociar un disco de datos.

### <a name="attach-a-new-disk"></a>Conexión de un disco nuevo
Siga estos pasos para crear y asociar un nuevo disco de datos administrado a una máquina virtual en Azure DevTest Labs.

1. Inicie sesión en [Azure Portal](https://go.microsoft.com/fwlink/p/?LinkID=525040).
1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la lista.
1. En la lista de laboratorios, seleccione el laboratorio que desee. 
1. En la lista **Mis máquinas virtuales**, seleccione una máquina virtual en ejecución.
1. En el menú de la izquierda, seleccione **Discos**.
1. Elija **Asociar nuevo** para crear un nuevo disco de datos y asociarlo a la máquina virtual.

    ![Asociación de un disco de datos nuevo a una máquina virtual](./media/devtest-lab-attach-detach-data-disk/devtest-lab-attach-new.png)
1. Complete el panel **Asociar nuevo disco** escribiendo el nombre, tipo y tamaño del disco de datos.

    ![Rellenado del formulario "Asociar nuevo disco"](./media/devtest-lab-attach-detach-data-disk/devtest-lab-attach-new-form.png)
1. Seleccione **Aceptar**.

Transcurridos unos instantes, el nuevo disco de datos se crea y se asocia a la máquina virtual, y aparece en la lista de **Discos de datos** de esa máquina virtual.

### <a name="attach-an-existing-disk"></a>un disco existente
Siga estos pasos para volver a asociar un disco de datos disponible a una máquina virtual en ejecución. 

1. Seleccione una máquina virtual en ejecución para la que desee volver a asociar un disco de datos.
1. En el menú de la izquierda, seleccione **Discos**.
1. Seleccione **Asociar existente** para asociar un disco de datos disponible a la máquina virtual.

    ![Asociación de un disco de datos existente a una máquina virtual](./media/devtest-lab-attach-detach-data-disk/devtest-lab-attach-existing-button.png)

1. Desde el panel **Asociar disco existente**, seleccione Aceptar.

    ![Asociación de un disco de datos existente a una máquina virtual](./media/devtest-lab-attach-detach-data-disk/devtest-lab-attach-existing.png)

Transcurridos unos instantes, el disco de datos se asocia a la máquina virtual y aparece en la lista de **Discos de datos** de esa máquina virtual.

## <a name="detach-a-data-disk"></a>Desacoplar un disco de datos
Cuando ya no necesite un disco de datos que se encuentra asociado a una máquina virtual, puede desasociarlo fácilmente. La desasociación quita el disco de la máquina virtual, pero lo sigue almacenando para su uso posterior.

Si desea volver a usar los datos existentes en el disco, puede asociarlo de nuevo a la misma máquina virtual (o a otra).

### <a name="detach-from-the-vms-management-pane"></a>Asociación desde el panel de administración de la máquina virtual
1. En la lista de máquinas virtuales, seleccione una máquina virtual que tiene un disco de datos asociado.
1. En el menú de la izquierda, seleccione **Discos**.
1. En la lista de **Discos de datos**, seleccione el disco de datos que quiere desasociar.

    ![Selección de discos de datos para una máquina virtual](./media/devtest-lab-attach-detach-data-disk/devtest-lab-detach-button.png) 
1. Seleccione **Asociar** en la parte superior del panel de detalles del disco.

    ![Desacoplar un disco de datos](./media/devtest-lab-attach-detach-data-disk/devtest-lab-detach-data-disk2.png)
1. Haga clic en **Sí** para confirmar que desea desasociar el disco de datos.

El disco se desasocia y está disponible para asociarse a otra máquina virtual. 
### <a name="detach-from-the-labs-main-pane"></a>Desasociación del panel principal del laboratorio
1. En el panel principal del laboratorio, seleccione **Mis discos de datos**.
1. Haga clic con el botón derecho en el disco de datos que quiere desasociar, o seleccione los puntos suspensivos ( **...** ) y elija **Desasociar**.

    ![Desacoplar un disco de datos](./media/devtest-lab-attach-detach-data-disk/devtest-lab-detach-data-disk.png)
1. Haga clic en **Sí** para confirmar que desea crear desasociarlo.

   > [!NOTE]
   > Si ya se ha desasociado un disco de datos, puede quitarlo de la lista de discos de datos disponibles con **Eliminar**.
   >
   >

## <a name="upgrade-an-unmanaged-data-disk"></a>Actualización de un disco de datos no administrado
Si tiene una máquina virtual existente que utiliza discos de datos no administrado, puede convertir con facilidad la máquina virtual para usar discos administrados. Este proceso convierte el disco del SO y los discos de datos conectados.

Para actualizar un disco de datos no administrado, siga los pasos descritos en este artículo para [desasociar el disco de datos](#detach-a-data-disk) de una máquina virtual no administrada. A continuación, [vuelva a asociar el disco](#attach-an-existing-disk) a una máquina virtual administrada para actualizar automáticamente el disco de datos de no administrado a administrado.

## <a name="next-steps"></a>Pasos siguientes
Aprenda a administrar discos de datos para [máquinas virtuales reclamables](devtest-lab-add-claimable-vm.md#unclaim-a-vm).
