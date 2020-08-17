---
title: Identificación de un grupo para su administración en Privileged Identity Management (Azure AD) | Microsoft Docs
description: Aprenda a incorporar grupos a los que se pueden asignar roles para administrarlos como grupos de acceso con privilegios en Privileged Identity Management (PIM).
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
ms.date: 08/03/2020
ms.author: curtand
ms.collection: M365-identity-device-management
ms.openlocfilehash: 76f7b61b6fb065409a2fe4b2ed8f599f321cd1db
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87542312"
---
# <a name="bring-a-privileged-access-groups-preview-into-privileged-identity-management"></a>Incorporación de grupos de acceso con privilegios (versión preliminar) en Privileged Identity Management

En Azure Active Directory (Azure AD), puede asignar roles integrados de Azure AD a grupos en la nube para simplificar la administración de las asignaciones de roles. Para proteger los roles de Azure AD y proteger el acceso, ahora puede usar Privileged Identity Management (PIM) para administrar el acceso Just-in-Time para los miembros o propietarios de estos grupos. Para administrar un grupo al que se pueden asignar roles de Azure AD como grupo de acceso privilegiado en Privileged Identity Management, debe incorporarlo a la administración de PIM.

## <a name="identify-groups-to-manage"></a>Identificación de los grupos que se administrarán

Puede crear un grupo al que se pueden asignar roles en Azure AD como se describe en [Creación de un grupo al que se pueden asignar roles en Azure Active Directory](../users-groups-roles/roles-groups-create-eligible.md). Debe ser propietario del grupo para poder incorporarlo a la administración con Privileged Identity Management.

1. [Inicie sesión en Azure AD](https://aad.portal.azure.com) con permisos del rol Administrador de roles con privilegios.
1. Seleccione **Grupos** y, a continuación, seleccione el grupo al que se pueden asignar roles que quiera administrar en PIM. Puede buscar y filtrar la lista.

    ![buscar un grupo al que se pueden asignar roles para administrarlo en PIM](./media/groups-discover-groups/groups-list-in-azure-ad.png)

1. Abra el grupo y seleccione **Acceso con privilegios (versión preliminar)** .

    ![Abrir la experiencia de Privileged Identity Management](./media/groups-discover-groups/groups-discover-groups.png)

1. Comience a administrar las asignaciones en PIM.

    ![Administrar asignaciones en Privileged Identity Management](./media/groups-discover-groups/groups-bring-under-management.png)

> [!NOTE]
> Una vez que se administra un grupo de acceso con privilegios, no se puede quitar de la administración. Esto impide que otro administrador de recursos pueda eliminar la configuración de Privileged Identity Management.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración las asignaciones del grupo de acceso con privilegios en Privileged Identity Management](pim-resource-roles-configure-role-settings.md)
- [Asignación de grupos de acceso con privilegios en Privileged Identity Management](pim-resource-roles-assign-roles.md)
