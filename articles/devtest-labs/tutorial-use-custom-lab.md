---
title: Acceso a un laboratorio en Azure DevTest Labs | Microsoft Docs
description: En este tutorial, va a acceder al laboratorio que se ha creado con Azure DevTest Labs, reclamar máquinas virtuales, utilizarlas y, después, anular la reclamación.
ms.topic: tutorial
ms.date: 06/26/2020
ms.author: spelluru
ms.openlocfilehash: b4477e0b98ef534b8170ee297edf88ac6fa62dd7
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85476451"
---
# <a name="tutorial-access-a-lab-in-azure-devtest-labs"></a>Tutorial: Acceso a un laboratorio en Azure DevTest Labs
En este tutorial, va a utilizar el laboratorio que se creó en el [Tutorial: Creación de un laboratorio en Azure DevTest Labs](tutorial-create-custom-lab.md).

En este tutorial realizará lo siguiente:

> [!div class="checklist"]
> * Notificación de una máquina virtual en el laboratorio
> * Conexión a la máquina virtual
> * Anulación de la reclamación de la máquina virtual

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="access-the-lab"></a>Acceso al laboratorio

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Todos los recursos** en el menú izquierdo. 
3. Seleccione **DevTest Labs** como tipo de recurso. 
4. Seleccione el laboratorio. 

    ![Selección del laboratorio](./media/tutorial-use-custom-lab/search-for-select-custom-lab.png)

## <a name="claim-a-vm"></a>Reclamación de una máquina virtual

1. En la lista de **Máquinas virtuales que se pueden reclamar**, seleccione **...**  (puntos suspensivos) y seleccione **Reclamar máquina**.

    ![Reclamación de una máquina virtual](./media/tutorial-use-custom-lab/claim-virtual-machine.png)
1. Confirme que ve la máquina virtual en la lista **Mis máquinas virtuales**.

    ![Mi máquina virtual](./media/tutorial-use-custom-lab/my-virtual-machines.png)

## <a name="connect-to-the-vm"></a>Conexión a la máquina virtual

1. Seleccione la máquina virtual en la lista. Verá la **página de máquina virtual** para la suya. Seleccione **Conectar** en la barra de herramientas.

    ![Conexión a la máquina virtual](./media/tutorial-use-custom-lab/connect-button.png)
2. Guarde el archivo **RDP** descargado en el disco duro y utilícelo para conectarse a la máquina virtual. Especifique el nombre de usuario y la contraseña que mencionó cuando se creó la máquina virtual en la sección anterior. 

    Para conectarse a una máquina virtual Linux, el acceso RDP o SSH debe estar habilitado para la máquina virtual. Para conocer el procedimiento de conexión a una máquina virtual Linux a través de RDP, consulte [Instalación y configuración de Escritorio remoto para conectarse a una máquina virtual Linux en Azure](../virtual-machines/linux/use-remote-desktop.md). 

    > [!NOTE]
    > Hay otras maneras de llegar a la página de la máquina virtual. Esta información incluye lo siguiente: 
    > 
    > 1. Busque todas las máquinas virtuales de la suscripción. Seleccione la máquina virtual en la lista de máquinas virtuales para llegar a la página **Máquina virtual**.
    > 2. Vaya a la página **Grupo de recursos** para el grupo de recursos. A continuación, seleccione la máquina virtual de la lista de recursos del grupo de recursos para llegar a la página **Máquina virtual**. 
    >
    > No use el botón **Conectar** de la barra de herramientas de la página **Máquina virtual** a la que llega con estas opciones. En su lugar, vaya a la página **Máquina virtual** desde la página **DevTest Labs** como se muestra en este artículo y use el botón **Conectar** de la barra de herramientas.


## <a name="unclaim-the-vm"></a>Anulación de la reclamación de la máquina virtual
Cuando haya terminado de usar la máquina virtual, anule la reclamación de dicha máquina siguiendo estos pasos: 

1. En la página de la máquina virtual, seleccione **Anular reclamación** en la barra de herramientas. 

    ![Anulación de la reclamación de una máquina virtual](./media/tutorial-use-custom-lab/unclaim-vm-menu.png)
1. La máquina virtual se apaga antes de anular su reclamación. Puede ver el estado de esta operación en las notificaciones.  
3. Vaya a la página de DevTest Lab; para ello, haga clic en el nombre del laboratorio en el menú de la ruta de navegación de la parte superior. 
    
    ![Regreso al laboratorio](./media/tutorial-use-custom-lab/breadcrumb-to-lab.png)
1. Confirme que ve la máquina virtual en la lista de **Máquinas virtuales que se pueden reclamar** en la parte de abajo.

    
## <a name="next-steps"></a>Pasos siguientes
Este tutorial le ha mostrado cómo acceder y usar un laboratorio creado con Azure DevTest Labs. Para más información sobre el acceso y el uso de máquinas virtuales en un laboratorio, consulte 

> [!div class="nextstepaction"]
> [Procedimiento: Uso de máquinas virtuales en un laboratorio](devtest-lab-add-vm.md)

