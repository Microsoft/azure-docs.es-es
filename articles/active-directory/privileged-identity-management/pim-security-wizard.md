---
title: Detección e información (versión preliminar) para roles de Azure AD en Privileged Identity Management (anteriormente, Asistente de seguridad de Azure Active Directory)
description: Detección e información (anteriormente, Asistente de seguridad) ayudan a convertir las asignaciones de roles permanentes de Azure AD en asignaciones Just-in-Time con Privileged Identity Management.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 08/07/2020
ms.author: curtand
ms.custom: pim ; H1Hack27Feb2017
ms.collection: M365-identity-device-management
ms.openlocfilehash: ef1de7e98a9cb57f83b87589ceddedc3cdd80927
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88005978"
---
# <a name="discovery-and-insights-preview-for-azure-ad-roles-formerly-security-wizard"></a>Detección e información (versión preliminar) para roles de Azure AD (anteriormente, Asistente para seguridad)

Si es la primera vez que usa Privileged Identity Management (PIM) en la organización de Azure Active Directory (Azure AD), podrá usar la página **Detección e información (versión preliminar**) para empezar. Esta característica muestra quién tiene asignado roles con privilegios en su organización y cómo usar PIM para cambiar rápidamente las asignaciones de roles permanentes en asignaciones Just-in-Time. Puede ver o realizar cambios en las asignaciones de roles con privilegios permanentes en **Detección e información (versión preliminar)** . Se trata de una herramienta de análisis y una herramienta de acción.

## <a name="discovery-and-insights-preview"></a>Detección e información (versión preliminar)

Antes de que la organización empiece a usar Privileged Identity Management, todas las asignaciones de roles son permanentes. Los usuarios siempre tienen sus roles asignados aunque no necesiten sus privilegios. Detección e información (versión preliminar), que reemplaza al antiguo Asistente de seguridad, muestra una lista de roles con privilegios y el número de usuarios que tienen actualmente esos roles. Puede obtener una lista de las asignaciones de un rol para obtener más información sobre los usuarios asignados si uno o varios de ellos no están familiarizados.

:heavy_check_mark: **Microsoft recomienda** que mantenga 2 cuentas de emergencia que estén asignadas permanentemente al rol de administrador global y que no requieran la autenticación multifactor al iniciar sesión. Puede usarlas en cualquier escenario de emergencia o en caso de que nadie pueda convertirse en administrador global.

Además, mantenga las asignaciones de roles permanentes si un usuario tiene una cuenta de Microsoft (es decir, una cuenta que usan para iniciar sesión en servicios de Microsoft como Skype u Outlook.com). Si necesita que la autenticación multifactor se active en ese rol, el usuario se bloqueará.

## <a name="open-discovery-and-insights-preview"></a>Apertura de Detección e información (versión preliminar)

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Roles de Azure AD** y, a continuación, seleccione **Detección e información (versión preliminar)** . Al abrir la página, se inicia el proceso de detección para buscar las asignaciones de roles pertinentes.

    ![La página Roles de Azure AD - Detección e información que muestra las 3 opciones](./media/pim-security-wizard/new-preview-link.png)

1. Seleccione **Reducir administradores globales**.

    ![Panel Reducir administradores globales - Rol que muestra todos los miembros](./media/pim-security-wizard/new-preview-page.png)

1. Revise la lista de asignaciones de roles de administradores globales.

    ![Panel Reducir administradores globales - Rol que muestra todos los miembros](./media/pim-security-wizard/new-global-administrator-list.png)

1. Seleccione **Siguiente** para seleccionar los usuarios o grupos que desea que sean válidos y, a continuación, seleccione **Establecer como apto** o **Quitar asignación**.

    ![Página de conversión de los miembros en aptos con opciones para seleccionar los miembros que quiere que sean aptos para los roles](./media/pim-security-wizard/new-global-administrator-buttons.png)

1. También puede requerir que todos los administradores globales revisen su propio acceso.

    ![Página de administradores globales que muestra la sección de revisiones de acceso](./media/pim-security-wizard/new-global-administrator-access-review.png)

1. Después de seleccionar cualquiera de estos cambios, verá una notificación de Azure.

1. Después, puede seleccionar **Eliminar acceso permanente** o **Revisar entidades de servicio** para repetir los pasos anteriores en otros roles con privilegios y en las asignaciones de roles de entidades de servicio. En el caso de las asignaciones de roles de entidades de servicio, solo se pueden quitar asignaciones de roles.

    ![Opciones de información adicionales para eliminar el acceso permanente y revisar las entidades de servicio ](./media/pim-security-wizard/new-preview-page-service-principals.png)

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de roles de Azure AD en Privileged Identity Management](pim-how-to-add-role-to-user.md)
- [Concesión de acceso a otros administradores para administrar Privileged Identity Management](pim-how-to-give-access-to-pim.md)
