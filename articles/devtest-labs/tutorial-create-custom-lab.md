---
title: Creación de un laboratorio con Azure DevTest Labs | Microsoft Docs
description: En este tutorial, se creará un laboratorio en Azure DevTest Labs desde Azure Portal. Los administradores de laboratorio configuran laboratorios, crean máquinas virtuales en los laboratorios y configuran las directivas.
ms.topic: tutorial
ms.date: 06/26/2020
ms.openlocfilehash: 9d016b38beb2f338a97ac862a060dc5ec3a77780
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "85476570"
---
# <a name="tutorial-set-up-a-lab-by-using-azure-devtest-labs"></a>Tutorial: Configuración de un laboratorio mediante Azure DevTest Labs
En este tutorial, va a crear un laboratorio mediante Azure Portal. El administrador de laboratorio configura un laboratorio en una organización, crea máquinas virtuales en el laboratorio y configura las directivas. Los usuarios de laboratorio (por ejemplo: desarrolladores y evaluadores) reclaman las máquinas virtuales en el laboratorio, se conectan a ellas y las usan. 

En este tutorial realizará lo siguiente:

> [!div class="checklist"]
> * Creación de un laboratorio
> * Adición de máquinas virtuales al laboratorio
> * Adición de un usuario al rol de usuario de laboratorio

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="create-a-lab"></a>Creación de un laboratorio
Los pasos siguientes muestran cómo usar Azure Portal para crear un laboratorio en Azure DevTest Labs. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. En el menú principal de la izquierda, seleccione **Crear un recurso**  (en la parte superior de la lista), seleccione **Herramientas de desarrollo** y haga clic en **DevTest Labs**. 

    ![Nuevo menú de DevTest Lab](./media/tutorial-create-custom-lab/new-custom-lab-menu.png)
1. En la ventana **Crear un laboratorio de DevTest Labs**, lleve a cabo las siguientes acciones: 
    1. En **Nombre del laboratorio**, escriba un nombre para el laboratorio. 
    2. En **Suscripción**, seleccione la suscripción en la que desea crear el laboratorio. 
    3. En **Grupo de recursos**, seleccione **Crear nuevo** y escriba un nombre para el grupo de recursos. 
    4. En **Ubicación**, seleccione la ubicación o región en la que desea crear el laboratorio. 
    5. Seleccione **Crear**. 
    6. Seleccione **Anclar al panel**. Una vez creado el laboratorio, este se muestra en el panel. 

        ![Creación de una sección de laboratorio de DevTest Labs](./media/tutorial-create-custom-lab/create-custom-lab-blade.png)
2. Examine las notificaciones para confirmar que el laboratorio se crea correctamente. Haga clic en **Go to resource** (Ir al recurso).  

    ![Notification](./media/tutorial-create-custom-lab/creation-notification.png)
3. Confirme que ve la página **DevTest Lab** de su laboratorio. 

    ![Página principal de su laboratorio](./media/tutorial-create-custom-lab/lab-home-page.png)

## <a name="add-a-vm-to-the-lab"></a>Adición de una máquina virtual al laboratorio

1. En la página **DevTest Lab**, seleccione **+ Agregar** en la barra de herramientas. 

    ![Botón Agregar](./media/tutorial-create-custom-lab/add-vm-to-lab-button.png)
1. En la página **Elegir base de datos**, busque con la palabra clave (por ejemplo, Windows, Ubuntu) y seleccione una de las imágenes de base de la lista. 
1. En la página **Máquina virtual**, realice las acciones siguientes: 
    1. En **Nombre de máquina virtual**, escriba un nombre para la máquina virtual. 
    2. En **Nombre de usuario**, escriba un nombre para el usuario que tiene acceso a la máquina virtual. 
    3. En **Password** (Contraseña), escriba la contraseña del usuario. 

        ![Elegir base de datos](./media/tutorial-create-custom-lab/new-virtual-machine.png)
1. Seleccione la pestaña **Advanced settings** (Configuración avanzada).
    1. En **Hacer que esta máquina sea reclamable**, seleccione **Sí**.
    2. Confirme que el **número de instancias** está establecido en **1**. Si se establece en **2**, se crean dos máquinas virtuales con nombres: `<base image name>00' and <base image name>01`. Por ejemplo, `win10vm00` y `win10vm01`.     
    3. Seleccione **Submit** (Enviar). 

        ![Elegir base de datos](./media/tutorial-create-custom-lab/new-vm-advanced-settings.png)
    9. Puede ver el estado de la máquina virtual en la lista de **Máquinas virtuales que se pueden reclamar**. La creación de la máquina virtual puede tardar aproximadamente 25 minutos. La máquina virtual se crea en un grupo de recursos independiente de Azure, cuyo nombre comienza por el nombre del grupo de recursos actual que tiene el laboratorio. Por ejemplo, si el laboratorio se encuentra en `labrg`, la máquina virtual se puede crear en el grupo de recursos `labrg3988722144002`. 

        ![Estado de la creación de la máquina virtual](./media/tutorial-create-custom-lab/vm-creation-status.png)
1. Después de crear la máquina virtual, puede verla en la lista **Máquinas virtuales que se pueden reclamar**. 

    > [!NOTE] 
    > En la página **Configuración avanzada** puede configurar la dirección IP pública, privada o de uso compartido para la máquina virtual. Cuando la **IP compartida** está habilitada, Azure DevTest Labs habilita RDP automáticamente para las máquinas virtuales Windows y SSH para las máquinas virtuales Linux. Si crea máquinas virtuales con direcciones **IP públicas**, RDP y SSH están habilitados sin cambios de DevTest Labs.  

## <a name="add-a-user-to-the-lab-user-role"></a>Adición de un usuario al rol de usuario de laboratorio

1. Seleccione **Configuración y directivas**. 

    ![Directivas y configuración](./media/tutorial-create-custom-lab/configuration-and-policies-menu.png)
1. Seleccione **Control de acceso (IAM)** en el menú y seleccione **+ Agregar asignación de rol** en la barra de herramientas. 

    ![Botón Agregar asignación de rol](./media/tutorial-create-custom-lab/add-role-assignment-button.png)
1. En la página **Agregar permisos**, realice las siguientes acciones:
    1. En **Rol**, seleccione **Usuario de DevTest Labs**. 
    2. Seleccione el **usuario** que desea agregar. 
    3. Seleccione **Guardar**.

        ![Agregar usuario](./media/tutorial-create-custom-lab/add-user.png)

## <a name="clean-up-resources"></a>Limpieza de recursos
En el siguiente tutorial se muestra cómo un usuario de laboratorio puede reclamar y conectarse a una máquina virtual en el laboratorio. Si no quiere hacer ese tutorial, y limpiar los recursos creados como parte de este tutorial, siga estos pasos: 

1. En Azure Portal, seleccione **Grupos de recursos** en el menú. 

    ![Grupos de recursos](./media/tutorial-create-custom-lab/resource-groups.png)
1. Seleccione el grupo de recursos en el que creó el laboratorio. 
1. Seleccione **Eliminar grupo de recursos** en la barra de herramientas. La eliminación de un grupo de recursos supone la eliminación de todos los recursos del grupo, incluido el laboratorio. 

    ![Grupo de recursos de laboratorio](./media/tutorial-create-custom-lab/lab-resource-group.png)
1. Repita estos pasos para eliminar el grupo de recursos adicionales que se creó automáticamente con el nombre `<your resource group name><random numbers>`. Por ejemplo: `splab3988722144001`. Las máquinas virtuales se crean en este grupo de recursos en lugar de en el grupo de recursos en el que existe el laboratorio. 

## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha creado un laboratorio con una máquina virtual y ha dado acceso al usuario al laboratorio. Para más información sobre cómo acceder al laboratorio como un usuario de laboratorio, avance al tutorial siguiente:

> [!div class="nextstepaction"]
> [Tutorial: Acceso al laboratorio](tutorial-use-custom-lab.md)

