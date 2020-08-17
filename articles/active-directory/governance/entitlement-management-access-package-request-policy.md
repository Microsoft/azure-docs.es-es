---
title: 'Cambio de la configuración de la solicitud y aprobación para un paquete de acceso en la administración de derechos de Azure AD: Azure Active Directory'
description: Aprenda a modificar la configuración de la solicitud y aprobación de un paquete de acceso en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: ''
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: compliance
ms.date: 06/18/2020
ms.author: ajburnle
ms.reviewer: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: edf38013efb14e412fbcd43e06dcf17e61c3bc4a
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798773"
---
# <a name="change-request-and-approval-settings-for-an-access-package-in-azure-ad-entitlement-management"></a>Cambio de la configuración de la solicitud y aprobación para un paquete de acceso en la administración de derechos de Azure AD

Como administrador de paquetes de acceso, puede cambiar los usuarios que pueden solicitar un paquete de acceso en cualquier momento si edita la directiva o agrega una nueva. También puede cambiar la configuración de aprobación.

En este artículo, se explica cómo se puede modificar la configuración de la solicitud y aprobación de un paquete de acceso existente.

## <a name="choose-between-one-or-multiple-polices"></a>Elección entre una o varias directivas

Para especificar quién puede solicitar un paquete de acceso, debe usar una directiva. Cuando se crea un paquete de acceso, se especifica la configuración de la solicitud y aprobación que crea una directiva. La mayoría de los paquetes de acceso tendrán una sola directiva, pero un único paquete de acceso puede tener varias. Se crearían varias directivas para un único paquete de acceso si deseara permitir que se concedan asignaciones a los distintos conjuntos de usuarios con una configuración de solicitudes y aprobación diferente. Por ejemplo, no se puede usar una sola directiva para asignar usuarios internos y externos al mismo paquete de acceso. Pero puede crear dos directivas en el mismo paquete de acceso: una para los usuarios internos y otra para los usuarios externos. Si hay varias directivas que se aplican a un usuario, se le pedirá en el momento de su solicitud que seleccione la directiva que le gustaría que se le asignara. En el diagrama siguiente se muestra un paquete de acceso con dos directivas.

![Varias directivas en un paquete de acceso](./media/entitlement-management-access-package-request-policy/access-package-policy.png)

| Escenario | Número de directivas |
| --- | --- |
| Quiero que todos los usuarios de mi directorio tengan la misma configuración de solicitudes y aprobación para un paquete de acceso | Uno |
| Quiero que todos los usuarios de ciertas organizaciones conectadas puedan solicitar un paquete de acceso | Uno |
| Deseo permitir que los usuarios de mi directorio y también los de fuera de mi directorio soliciten un paquete de acceso | Múltiple |
| Deseo especificar una configuración de aprobación diferente para algunos usuarios | Múltiple |
| Quiero que algunos usuarios accedan a las asignaciones de paquetes que van a expirar mientras que otros usuarios puedan extender su acceso | Múltiple |

Para información sobre la lógica de prioridad que se usa cuando se aplican varias directivas, consulte [Varias directivas](entitlement-management-troubleshoot.md#multiple-policies
).

### <a name="open-an-existing-policy-of-request-and-approval-settings"></a>Apertura de una directiva existente de configuración de solicitudes y aprobación

Para cambiar la configuración de solicitudes y aprobación de un paquete de acceso, debe abrir la directiva correspondiente. Siga estos pasos para abrir la configuración de solicitudes y aprobación de un paquete de acceso.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Haga clic en **Directivas** y luego en la directiva que quiere editar.

    Se abre el panel Detalles de directiva en la parte inferior de la página.

    ![Paquete de acceso - Panel Detalles de directiva](./media/entitlement-management-shared/policy-details.png)

1. Haga clic en **Editar** para editar la directiva.

    ![Paquete de acceso - Editar directiva](./media/entitlement-management-shared/policy-edit.png)

1. Haga clic en la pestaña **Solicitudes** para abrir la configuración de solicitudes y aprobación.

1. Realice los pasos en una de las siguientes secciones de la solicitud.

### <a name="add-a-new-policy-of-request-and-approval-settings"></a>Incorporación de una nueva directiva de configuración de solicitudes y aprobación

Si tiene un conjunto de usuarios que deben tener una configuración de solicitudes y aprobación diferente, probablemente tendrá que crear una directiva. Siga estos pasos para empezar a agregar una nueva directiva a un paquete de acceso existente.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. En el menú de la izquierda, haga clic en **Paquetes de acceso** y luego abra el paquete de acceso.

1. Haga clic en **Directivas**  y luego en **Agregar directiva**.

1. Escriba un nombre y una descripción para la directiva.

    ![Crear directiva con nombre y descripción](./media/entitlement-management-access-package-request-policy/policy-name-description.png)

1. Haga clic en **Siguiente** para abrir la pestaña **Solicitudes**.

1. Realice los pasos en una de las siguientes secciones de la solicitud.

[!INCLUDE [Entitlement management request policy](../../../includes/active-directory-entitlement-management-request-policy.md)]

Si va a editar una directiva, haga clic en **Actualizar**. Si va a agregar una directiva nueva, haga clic en **Crear**.

## <a name="next-steps"></a>Pasos siguientes

- [Cambiar la configuración del ciclo de vida de un paquete de acceso](entitlement-management-access-package-lifecycle-policy.md)
- [Ver las solicitudes de un paquete de acceso](entitlement-management-access-package-requests.md)