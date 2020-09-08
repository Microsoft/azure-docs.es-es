---
title: 'Administración de grupos a los que se pueden asignar roles como grupos de acceso con privilegios: Azure AD | Microsoft Docs'
description: Consienta la incorporación de grupos a los que se pueden asignar roles para administrarlos como grupos de acceso con privilegios en Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 08/17/2020
ms.author: curtand
ms.collection: M365-identity-device-management
ms.openlocfilehash: a707c66f2562fe442c58ce6292e51c4e67a20ae2
ms.sourcegitcommit: 927dd0e3d44d48b413b446384214f4661f33db04
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88869545"
---
# <a name="manage-privileged-access-groups-preview-in-privileged-identity-management"></a>Administración de grupos de acceso con privilegios (versión preliminar) en Privileged Identity Management

En Azure Active Directory (Azure AD), puede asignar roles integrados de Azure AD a grupos en la nube para simplificar la administración de las asignaciones de roles. Para proteger los roles de Azure AD y proteger el acceso, ahora puede usar Privileged Identity Management (PIM) para administrar el acceso Just-in-Time para los miembros o propietarios de estos grupos. Para administrar un grupo al que se pueden asignar roles de Azure AD como grupo de acceso privilegiado en Privileged Identity Management, debe incorporarlo a la administración de PIM.

## <a name="identify-groups-to-manage"></a>Identificación de los grupos que se administrarán

Puede crear un grupo al que se pueden asignar roles en Azure AD como se describe en [Creación de un grupo al que se pueden asignar roles en Azure Active Directory](../users-groups-roles/roles-groups-create-eligible.md). Debe ser propietario del grupo para poder incorporarlo a la administración en Privileged Identity Management.

1. [Inicie sesión en Azure AD](https://aad.portal.azure.com) con permisos de Administrador de roles con privilegios.
1. Seleccione **Grupos** y, a continuación, seleccione el grupo al que se pueden asignar roles que quiera administrar. Puede buscar o filtrar la lista.

    ![buscar un grupo al que se pueden asignar roles para administrarlo en PIM](./media/groups-discover-groups/groups-list-in-azure-ad.png)

1. Abra el grupo y seleccione **Acceso con privilegios (versión preliminar)** .

    ![Abrir la experiencia de Privileged Identity Management](./media/groups-discover-groups/groups-discover-groups.png)

1. Si los grupos todavía no se han incorporado a la administración en PIM, seleccione **Enable privileged access** (Habilitar acceso con privilegios) para dar su consentimiento a la administración. Solo el administrador global o el propietario de un grupo puede conceder este consentimiento.

    ![consentir la administración del grupo en Privileged Identity Management si es necesario](./media/groups-discover-groups/consent-page.png)

1. Comience a administrar las asignaciones en PIM.

    ![Administrar asignaciones en Privileged Identity Management](./media/groups-discover-groups/groups-bring-under-management.png)

> [!NOTE]
> Una vez que se administra un grupo de acceso con privilegios, no se puede quitar de la administración. Esto impide que otro administrador pueda eliminar la configuración de Privileged Identity Management.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración las asignaciones del grupo de acceso con privilegios en Privileged Identity Management](pim-resource-roles-configure-role-settings.md)
- [Asignación de grupos de acceso con privilegios en Privileged Identity Management](pim-resource-roles-assign-roles.md)
