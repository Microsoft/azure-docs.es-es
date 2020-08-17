---
title: 'Cómo administrar una identidad administrada asignada por el usuario mediante Azure Portal: Azure AD'
description: Instrucciones paso a paso para crear, enumerar, eliminar y asignar un rol a una identidad administrada asignada por el usuario.
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/16/2018
ms.author: markvi
ms.collection: M365-identity-device-management
ms.openlocfilehash: 27144dcd4c7d7490aba3626264e1cb4b64fbb162
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87019206"
---
# <a name="create-list-delete-or-assign-a-role-to-a-user-assigned-managed-identity-using-the-azure-portal"></a>Creación, enumeración, eliminación o asignación de un rol a una identidad administrada asignada por el usuario mediante Azure Portal

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. 

En este artículo aprenderá a crear, enumerar, eliminar y asignar un rol a una identidad administrada asignada por el usuario mediante Azure Portal.

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.

## <a name="create-a-user-assigned-managed-identity"></a>Crear una identidad administrada asignada por el usuario

Para crear una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](/azure/role-based-access-control/built-in-roles#managed-identity-contributor).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure para crear la identidad administrada asignada por el usuario.
2. En el cuadro de búsqueda, escriba *Identidades administradas* y en **Servicios**, haga clic en **Identidades administradas**.
3. Haga clic en **Agregar** y escriba valores en los campos siguientes del panel **Crear identidad administrada asignada por el usuario**:
   - **Nombre del recurso**: este es el nombre de la identidad administrada asignada por el usuario, por ejemplo, UAI1.
   - **Suscripción**: elija la suscripción donde crear la identidad administrada asignada por el usuario.
   - **Grupo de recursos**: cree un grupo de recursos que contenga la identidad asignada por el usuario o elija **Usar existente** para crear esta en un grupo de recursos existente.
   - **Ubicación**: Elija una ubicación para implementar la identidad administrada asignada por el usuario, por ejemplo **Oeste de EE. UU.** .
4. Haga clic en **Crear**.

![Crear una identidad administrada asignada por el usuario](./media/how-to-manage-ua-identity-portal/create-user-assigned-managed-identity-portal.png)

## <a name="list-user-assigned-managed-identities"></a>Enumerar identidades administradas asignadas por el usuario

Para crear o leer una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Operador de identidades administradas ](/azure/role-based-access-control/built-in-roles#managed-identity-operator) o [Colaborador de identidades administradas](/azure/role-based-access-control/built-in-roles#managed-identity-contributor).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure para enumerar las identidades administradas asignadas por el usuario.
2. En el cuadro de búsqueda, escriba *Identidades administradas* y en Servicios, haga clic en **Identidades administradas**.
3. Se devuelve una lista de las identidades administradas asignadas por el usuario de la suscripción.  Para ver los detalles de una identidad administrada asignada por el usuario, haga clic en su nombre.

![Enumerar identidades administradas asignadas por el usuario](./media/how-to-manage-ua-identity-portal/list-user-assigned-managed-identity-portal.png)

## <a name="delete-a-user-assigned-managed-identity"></a>Eliminar una identidad administrada asignada por el usuario

Para eliminar una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Colaborador de identidades administradas](/azure/role-based-access-control/built-in-roles#managed-identity-contributor).

La eliminación de una identidad asignada por el usuario no la quita de ninguna VM o recurso a la que estuviera asignada.  Para quitar la identidad asignada por el usuario desde una máquina virtual, consulte [Eliminación de una identidad administrada asignada por el usuario de una VM](/azure/active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm#remove-a-user-assigned-managed-identity-from-a-vm).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure para eliminar una identidad administrada asignada por el usuario.
2. Seleccione la identidad administrada asignada por el usuario y haga clic en **Eliminar**.
3. En el cuadro de confirmación, elija **Sí**.

![Eliminar una identidad administradas asignada por el usuario](./media/how-to-manage-ua-identity-portal/delete-user-assigned-managed-identity-portal.png)

## <a name="assign-a-role-to-a-user-assigned-managed-identity"></a>Asignación de un rol a una identidad administrada asignada por el usuario 

Para asignar un rol a una identidad administrada asignada por el usuario, la cuenta requiere la asignación del rol [Administrador de acceso de usuario](/azure/role-based-access-control/built-in-roles#user-access-administrator).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta asociada a la suscripción de Azure para enumerar las identidades administradas asignadas por el usuario.
2. En el cuadro de búsqueda, escriba *Identidades administradas* y en Servicios, haga clic en **Identidades administradas**.
3. Se devuelve una lista de las identidades administradas asignadas por el usuario de la suscripción.  Seleccione la identidad administrada asignada por el usuario a la que desea asignar un rol.
4. Seleccione **Control de acceso (IAM)** y después **Agregar asignación de rol**.

   ![Inicio de Identidad administrada asignada por el usuario](./media/how-to-manage-ua-identity-portal/assign-role-screenshot1.png)

5. En la hoja Agregar asignación de roles, configure los siguientes valores y después, haga clic en **Guardar**:
   - **Rol**: el rol que se va a asignar
   - **Asignar acceso a**: el recurso al que se asigna la identidad administrada asignada por el usuario
   - **Seleccionar**: el miembro al que se asigna acceso
   
   ![IAM de identidad administrada asignada por el usuario](./media/how-to-manage-ua-identity-portal/assign-role-screenshot2.png)  
