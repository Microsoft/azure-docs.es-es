---
title: 'Edición de la información de un grupo: Azure Active Directory | Microsoft Docs'
description: Instrucciones sobre cómo editar la información de su grupo mediante Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 08/27/2018
ms.author: ajburnle
ms.reviewer: krbain
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1e9b38208526ba83e0a0e31e2e139e340a3ee511
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87797532"
---
# <a name="edit-your-group-information-using-azure-active-directory"></a>Edición de la información de un grupo mediante Azure Active Directory

Con Azure Active Directory (Azure AD), puede editar la configuración de un grupo, lo que incluye actualizar su nombre, descripción o tipo de pertenencia.

## <a name="to-edit-your-group-settings"></a>Para editar la configuración de un grupo
1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta de administrador global para el directorio.

2. Seleccione **Azure Active Directory** y después seleccione **Grupos**.

    Aparecerá la página **Grupos - Todos los grupos** con todos los grupos activos.

3. Desde la página **Grupos - Todos los grupos**, escriba parte del nombre del grupo en el cuadro de **Búsqueda**. Para los fines de este artículo, buscaremos el grupo **MDM policy - West**.

    Los resultados de búsqueda aparecen bajo el cuadro de **Búsqueda**, y se actualizan a medida que escribe más caracteres.

    ![Página Todos los grupos, con texto en el cuadro de búsqueda](media/active-directory-groups-settings-azure-portal/search-for-specific-group.png)

4. Seleccione el grupo **MDM policy - West** y, a continuación, seleccione **Propiedades** desde la sección **Administrar**.

    ![Página Información general del grupo, con la opción Miembros e información resaltadas](media/active-directory-groups-settings-azure-portal/group-overview-blade.png)

5. Actualice la información de **configuración general** según sea necesario, incluidos:

    ![Configuración de propiedades de un grupo](media/active-directory-groups-settings-azure-portal/group-properties-settings.png)

    - **Nombre del grupo.** Edite el nombre de grupo existente.
    
    - **Descripción del grupo.** Edite la descripción de grupo existente.

    - **Tipo de grupo.** No se puede cambiar el tipo de grupo después de que se ha creado. Para cambiar el **tipo de grupo**, debe eliminar el grupo y crear uno nuevo.
    
    - **Tipo de pertenencia.** Cambie el tipo de pertenencia de un grupo. Para más información sobre los distintos tipos de pertenencia disponibles, consulte [Creación de un grupo básico e incorporación de miembros mediante el portal de Azure Active Directory](active-directory-groups-create-azure-portal.md).
    
    - **Identificador de objeto.** No se puede cambiar el identificador de objeto, pero puede copiarlo para usarlo en los comandos de PowerShell para el grupo. Para más información acerca del uso de cmdlets de PowerShell, consulte [Cmdlets de Azure Active Directory para configurar las opciones de grupo](../users-groups-roles/groups-settings-v2-cmdlets.md).

## <a name="next-steps"></a>Pasos siguientes
Estos artículos proporcionan información adicional sobre Azure Active Directory.

- [Visualización de grupos y miembros](active-directory-groups-view-azure-portal.md)

- [Creación de un grupo básico e incorporación de miembros](active-directory-groups-create-azure-portal.md)

- [Incorporación o eliminación de miembros de un grupo](active-directory-groups-members-azure-portal.md)

- [Administrar reglas dinámicas de los usuarios de un grupo](../users-groups-roles/groups-create-rule.md)

- [Administrar la pertenencia a grupos](active-directory-groups-membership-azure-portal.md)

- [Administrar el acceso a los recursos mediante grupos](active-directory-manage-groups.md)

- [Asociar o agregar una suscripción de Azure a Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)
