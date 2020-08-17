---
title: Agregar o actualizar la información del perfil de usuario - Azure AD
description: Instrucciones sobre cómo agregar información a un perfil de usuario en Azure Active Directory, incluidos una imagen y los detalles del trabajo.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 04/11/2019
ms.author: ajburnle
ms.reviewer: jeffsta
ms.collection: M365-identity-device-management
ms.openlocfilehash: 63b716aa22954af51a9eb53a0d04a8299d348c71
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87797244"
---
# <a name="add-or-update-a-users-profile-information-using-azure-active-directory"></a>Adición o actualización de la información de perfil de un usuario mediante Azure Active Directory
Agregue información de perfil de usuario, como la imagen de perfil, información específica del trabajo y algunos valores de configuración mediante Azure Active Directory (Azure AD). Para obtener más información acerca de la adición de nuevos usuarios, consulte [cómo agregar o eliminar usuarios en Azure Active Directory](add-users-azure-active-directory.md).

## <a name="add-or-change-profile-information"></a>Adiciones o cambios a la información del perfil
Como verá, hay más información disponible en un perfil de usuario que la que puede agregarse durante la creación del usuario. Toda esta información adicional es opcional y se puede agregar según sea necesario para su organización.

## <a name="to-add-or-change-profile-information"></a>Para agregar o cambiar la información del perfil
1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de usuarios para la organización.

2. Seleccione **Azure Active Directory**, **Usuarios** y, a continuación, seleccione un usuario. Por ejemplo, _Alain Charon_.

    Se muestra la página **Alain Charon - Perfil**.

    ![Página de perfil del usuario, incluida la información editable](media/active-directory-users-profile-azure-portal/user-profile-all-blade.png)

3. Seleccione **Editar** para agregar o actualizar de forma opcional la información incluida en cada una de las secciones disponibles.

    ![Página de perfil del usuario, se muestran las áreas editables](media/active-directory-users-profile-azure-portal/user-profile-edit.png)

    - **Imagen de perfil.** Seleccione una imagen en miniatura para la cuenta de usuario. Esta imagen aparece en Azure Active Directory y en las páginas personales del usuario, tales como la página myapps.microsoft.com.

    - **Identidad.** Agregue o actualice un valor de identidad adicional para el usuario como, por ejemplo, el apellido de casada. Puede establecer este nombre de forma independiente de los valores de Nombre y Apellidos. Por ejemplo, podría utilizarlo para incluir iniciales, un nombre de empresa o para cambiar la secuencia de nombres mostrada. En otro ejemplo, para dos usuarios cuyos nombres son "Chris Green", podría utilizar la cadena de identidad para establecer sus nombres como "Chris B. Green" y "Chris R. Green (Contoso)".

    - **Información del trabajo.** Agregue cualquier información relacionada con el trabajo, como el puesto, departamento o administrador del usuario.

    - **Configuración.** Establezca si el usuario puede iniciar sesión en el inquilino de Azure Active Directory. También puede especificar la ubicación global del usuario.

    - **Información de contacto.** Agregue cualquier información de contacto pertinente para el usuario excepto, en algunos usuarios, el teléfono o la información de contacto móvil (solo un administrador global puede actualizar los usuarios con roles de administrador).

    - **Información de contacto para la autenticación.** Compruebe esta información para asegurarse de que hay una dirección de correo electrónico y un número de teléfono activos para el usuario. Azure Active Directory usa esta información para comprobar la identidad del usuario durante el inicio de sesión. Solo un administrador global puede actualizar la información de contacto para la autenticación.

4. Seleccione **Guardar**.

    Todos los cambios se guardan para el usuario.

    >[!Note]
    >Debe usar Windows Server Active Directory para actualizar la identidad, la información de contacto o la información del trabajo para los usuarios cuyo origen de autoridad es Windows Server Active Directory. Después de completar la actualización, debe esperar a que se complete el próximo ciclo de sincronización antes de poder ver los cambios.

## <a name="next-steps"></a>Pasos siguientes
Después de actualizar los perfiles de los usuarios, puede realizar los siguientes procesos básicos:

- [Adición o eliminación de usuarios](add-users-azure-active-directory.md)

- [Asignación de roles a usuarios](active-directory-users-assign-role-azure-portal.md)

- [Creación de un grupo básico e incorporación de miembros](active-directory-groups-create-azure-portal.md)

O bien, puede realizar otras tareas de administración de usuarios, como asignar delegados, usar directivas y compartir cuentas de usuario. Para obtener más información acerca de otras acciones disponibles, consulte la [documentación de administración de usuarios en Azure Active Directory](../users-groups-roles/index.yml).
