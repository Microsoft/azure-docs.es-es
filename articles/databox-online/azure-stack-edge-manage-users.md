---
title: Usuarios de administración de Azure Stack Edge Pro con GPU | Microsoft Docs
description: Se describe cómo usar Azure Portal para administrar usuarios en Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 03/11/2019
ms.author: alkohli
ms.openlocfilehash: 3fb82e95d9d7dcfdf59bd05f91beff8e1184955e
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904385"
---
# <a name="use-the-azure-portal-to-manage-users-on-your-azure-azure-stack-edge-pro"></a>Uso de Azure Portal para administrar usuarios en Azure Stack Edge Pro

En este artículo, se explica cómo administrar los usuarios en Azure Stack Edge Pro. Azure Stack Edge Pro se puede administrar con Azure Portal o con la interfaz de usuario web local. Usar Azure Portal para agregar, modificar o eliminar usuarios.

En este artículo aprenderá a:

> [!div class="checklist"]
> * Adición de un usuario
> * Modificación de un usuario
> * Eliminar un usuario

## <a name="about-users"></a>Acerca de los usuarios

Los usuarios pueden ser de solo lectura o tener privilegios totales. Como indican los nombres, los usuarios de solo lectura solo pueden ver los datos de los recursos compartidos. Los usuarios con privilegios totales pueden leer datos de los recursos compartidos, escribir en estos recursos compartidos y modificar o eliminar los datos de los recursos compartidos.

 - **Usuario con privilegios totales**: un usuario local con acceso completo.
 - **Usuario de solo lectura**: un usuario local con acceso de solo lectura. Estos usuarios están asociados a los recursos compartidos que permiten operaciones de solo lectura.

Los permisos del usuario se definen por primera vez cuando se crea el usuario durante la creación de los recursos compartidos. Actualmente no se admite la modificación de permisos en el nivel de recurso compartido.

## <a name="add-a-user"></a>Adición de un usuario

Para agregar un usuario siga estos pasos en Azure Portal.

1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **Información general > Usuarios**. Seleccione **+ Agregar usuario** en la barra de comandos.

    ![Seleccionar Agregar usuario](media/azure-stack-edge-manage-users/add-user-1.png)

2. Especifique el nombre de usuario y la contraseña del usuario que desea agregar. Configure la contraseña y seleccione **Agregar**.

    ![Especificación de nombre de usuario y contraseña](media/azure-stack-edge-manage-users/add-user-2.png)

    > [!IMPORTANT] 
    > El sistema reserva a estos usuarios y no se deben usar: Administrator, EdgeUser, EdgeSupport, HcsSetupUser, WDAGUtilityAccount, CLIUSR, DefaultAccount, Guest.  

3. Aparece una notificación cuando se inicia y se completa la creación del usuario. Una vez que se crea el usuario, en la barra de comandos, seleccione **Actualizar** para ver la lista actualizada de usuarios.


## <a name="modify-user"></a>Modificación de un usuario

Una vez que se crea un usuario es posible cambiar la contraseña asociada con él. Seleccione en la lista de usuarios. Escriba la contraseña nueva y confírmela. Guarde los cambios.
 
![Modificación de un usuario](media/azure-stack-edge-manage-users/modify-user-1.png)


## <a name="delete-a-user"></a>Eliminar un usuario

Siga estos pasos en Azure Portal para eliminar un usuario.


1. En Azure Portal, vaya al recurso de Azure Stack Edge y luego a **Información general > Usuarios**.

    ![Seleccionar el usuario que se va a eliminar](media/azure-stack-edge-manage-users/delete-user-1.png)

2. Seleccione un usuario en la lista de usuarios y, luego, seleccione **Eliminar**.  

   ![Seleccionar Eliminar](media/azure-stack-edge-manage-users/delete-user-2.png)

3. Cuando se le solicite, confirme la eliminación. 

   ![Confirmar eliminación](media/azure-stack-edge-manage-users/delete-user-3.png)

La lista de usuarios se actualiza para reflejar el usuario eliminado.

![Lista de usuarios actualizada](media/azure-stack-edge-manage-users/delete-user-4.png)


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [administrar el ancho de banda](azure-stack-edge-manage-bandwidth-schedules.md).
