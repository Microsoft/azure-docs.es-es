---
title: 'Primer uso de PIM: Azure Active Directory | Microsoft Docs'
description: Aprenda a habilitar y empezar a usar Azure AD Privileged Identity Management (PIM) en Azure Portal.
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.subservice: pim
ms.topic: how-to
ms.workload: identity
ms.date: 08/24/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8bad5b0256ee445767689ea2752e918a1fbf5fae
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88816004"
---
# <a name="start-using-privileged-identity-management"></a>Empiece a usar Privileged Identity Management

En este artículo se describe cómo habilitar y empezar a usar Privileged Identity Management (PIM).

Puede utilizar Privileged Identity Management (PIM) para administrar, controlar y supervisar el acceso dentro de la organización de Azure Active Directory (Azure AD). Con PIM, puede proporcionar acceso Just-in-Time y según sea necesario a los recursos de Azure y Azure AD, así como a otros servicios en línea de Microsoft, como Office 365 o Microsoft Intune.

## <a name="prerequisites"></a>Prerrequisitos

Para usar Privileged Identity Management, debe tener una de las licencias siguientes:

- Azure AD Premium P2
- Enterprise Mobility + Security (EMS) E5

Para más información, consulte [Requisitos de licencia para usar Privileged Identity Management](subscription-requirements.md).

## <a name="prepare-pim-for-azure-ad-roles"></a>Preparación de PIM para roles de Azure AD

Estas son las tareas que se recomiendan para preparar Privileged Identity Management a fin de administrar los roles de Azure AD:

1. [Configuración de roles de Azure AD](pim-how-to-change-default-settings.md)
1. [Proporcione asignaciones elegibles](pim-how-to-add-role-to-user.md).
1. [Permita que los usuarios elegibles activen su rol de Azure AD cuando sea necesario](pim-how-to-activate-role.md).

## <a name="prepare-pim-for-azure-roles"></a>Preparación de PIM para roles de Azure

Estas son las tareas que recomendamos si desea preparar Privileged Identity Management para administrar los roles de Azure para una suscripción:

1. [Detección de recursos de Azure](pim-resource-roles-discover-resources.md)
1. [Configuración de roles de Azure AD](pim-resource-roles-configure-role-settings.md)
1. [Proporcione asignaciones elegibles](pim-resource-roles-assign-roles.md).
1. [Permita que los usuarios elegibles activen sus roles de Azure cuando sea necesario](pim-resource-roles-activate-your-roles.md).

## <a name="navigate-to-your-tasks"></a>Navegación a sus tareas

Una vez que se haya configurado Privileged Identity Management, puede aprender a orientarse.

![Ventana de navegación en Privileged Identity Management que muestra las opciones Tareas y Administrar](./media/pim-getting-started/pim-quickstart-tasks.png)

| Tarea y administrar | Descripción |
| --- | --- |
| **Mis roles**  | Muestra una lista de los roles elegibles y activos que tiene asignados. Aquí es donde puede activar cualquier función elegible asignada. |
| **Mis solicitudes** | Muestra las solicitudes pendientes para activar las asignaciones de rol elegibles. |
| **Aprobar solicitudes** | Muestra una lista de las solicitudes realizadas por los usuarios de su directorio para activar roles elegibles que usted tiene designados para aprobar. |
| **Revisar acceso** | Enumera las revisiones de acceso activas que tiene asignadas para completar, tanto si revisa el acceso usted mismo como si lo hace otro usuario. |
| **Roles de Azure AD** | Muestra un panel y la configuración de los administradores de rol con privilegios para administrar las asignaciones de roles de Azure AD. Este panel está deshabilitado para todos aquellos que no sean administradores de roles con privilegios. Estos usuarios tienen acceso a un panel especial denominado My view (Mi vista). El panel Mi vista solo muestra información sobre el acceso al panel del usuario, no de la organización completa. |
| **Recursos de Azure** | Muestra un panel y la configuración de los administradores de rol con privilegios para administrar las asignaciones de roles de recursos de Azure. Este panel está deshabilitado para todos aquellos que no sean administradores de roles con privilegios. Estos usuarios tienen acceso a un panel especial denominado My view (Mi vista). El panel Mi vista solo muestra información sobre el acceso al panel del usuario, no de la organización completa. |

## <a name="add-a-pim-tile-to-the-dashboard"></a>Incorporación de un icono de PIM al panel

Para que sea más fácil abrir Privileged Identity Management, agregue un icono de PIM al panel de Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Seleccione **Todos los servicios** y busque el servicio **Azure AD Privileged Identity Management**.

    ![Azure AD Privileged Identity Management en Todos los servicios](./media/pim-getting-started/pim-all-services-find.png)

1. Seleccione el **inicio rápido** de Privileged Identity Management.

1. Active la opción **Anclar hoja al panel** para anclar la hoja **Inicio rápido** de Privileged Identity Management al panel.

    ![Icono de marcador para anclar la página Privileged Identity Management al panel](./media/pim-getting-started/pim-quickstart-pin-to-dashboard.png)

    En el panel de Azure, verá un icono como este:

    ![Icono de Inicio rápido de Privileged Identity Management en el panel](./media/pim-getting-started/pim-quickstart-dashboard-tile.png)

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de roles de Azure AD en Privileged Identity Management](pim-how-to-add-role-to-user.md)
- [Gestión del acceso a los recursos de Azure en Privileged Identity Management](pim-resource-roles-discover-resources.md)
