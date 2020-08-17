---
title: Administradores de la suscripción clásica de Azure
description: Se describe cómo agregar o cambiar los roles de coadministrador y de administrador de servicios, y cómo ver el de administrador de cuenta.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.assetid: ''
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 01/22/2020
ms.author: rolyon
ms.reviewer: bagovind
ms.openlocfilehash: 76b41e25a95f23b66edfbd4715037074537221f9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076437"
---
# <a name="azure-classic-subscription-administrators"></a>Administradores de la suscripción clásica de Azure

Microsoft recomienda que administre el acceso a los recursos de Azure mediante el control de acceso basado en rol (RBAC) de Azure. Sin embargo, si aún utiliza el modelo de implementación clásico, deberá usar un rol de administrador de suscripciones clásicas: Administrador de servicios y coadministrador. Para más información, consulte [Implementación de Azure Resource Manager frente a la implementación clásica](../azure-resource-manager/management/deployment-models.md).

En este artículo se describe cómo agregar o cambiar los roles de coadministrador y de administrador de servicios de Azure, y cómo ver el de administrador de cuenta.

## <a name="add-a-co-administrator"></a>Adición de un coadministrador

> [!TIP]
> Solo es necesario agregar un coadministrador si el usuario necesita administrar implementaciones clásicas de Azure mediante el [módulo de PowerShell de Azure Service Management](/powershell/module/servicemanagement/azure.service). Si el usuario solamente usa Azure Portal para administrar los recursos clásicos, no tendrá que agregar el administrador clásico para el usuario.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de servicios o coadministrador.

1. Abra [Suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) y seleccione una suscripción.

    Solo se pueden asignar coadministradores en el ámbito de la suscripción.

1. Haga clic en **Control de acceso (IAM).**

1. Haga clic en la pestaña **Administradores clásicos**.

    ![Captura de pantalla que abre la pestaña Administradores clásicos](./media/classic-administrators/classic-administrators.png)

1. Haga clic en **Agregar** > **Agregar coadministrador** para abrir el panel correspondiente.

    Si la opción Agregar coadministrador está deshabilitada, no tendrá permisos.

1. Seleccione el usuario que desea agregar y haga clic en **Agregar**.

    ![Captura de pantalla donde se agrega el coadministrador](./media/classic-administrators/add-coadmin.png)

## <a name="add-a-guest-user-as-a-co-administrator"></a>Adición de un usuario invitado como coadministrador

Para agregar un usuario invitado como coadministrador, siga los mismos pasos que en la sección [Adición de un coadministrador](#add-a-co-administrator). El usuario invitado debe cumplir los siguientes criterios:

- Debe tener una presencia en el directorio. Esto significa que se ha invitado al usuario al directorio y ha aceptado la invitación.

Para más información sobre cómo agregar usuarios invitados a su directorio, consulte [Incorporación de usuarios de colaboración B2B de Azure Active Directory en Azure Portal](../active-directory/b2b/add-users-administrator.md).

### <a name="differences-for-guest-users"></a>Diferencias para los usuarios invitados

Es posible que los usuarios invitados a los que se haya asignado el rol de coadministrador observen algunas diferencias con respecto a los usuarios miembros con este mismo rol. Considere el caso siguiente:

- El usuario A, con una cuenta de Azure AD (profesional o educativa), es un administrador de servicios en una suscripción de Azure.
- El usuario B tiene una cuenta de Microsoft.
- El usuario A asigna el rol de coadministrador al usuario B.
- El usuario B puede hacer casi todo, pero no puede registrar las aplicaciones ni buscar usuarios en el directorio de Azure AD.

Sería de esperar que el usuario B pudiera administrarlo todo. Esta diferencia se debe a que la cuenta de Microsoft se agrega a la suscripción como usuario invitado y no como usuario miembro. En comparación con los usuarios miembros, los usuarios invitados tienen distintos permisos predeterminados en Azure AD. Por ejemplo, los usuarios miembros pueden leer otros usuarios en Azure AD y los usuarios invitados no. Los usuarios miembros pueden registrar a nuevas entidades de servicio en Azure AD y los usuarios invitados no.

Si un usuario invitado debe ser capaz de realizar estas tareas, una posible solución consiste en asignarle los roles de Azure AD que el usuario invitado necesita. Por ejemplo, en el escenario anterior, podría asignarle el rol [Lectores de directorios](../active-directory/users-groups-roles/directory-assign-admin-roles.md#directory-readers) para que pueda leer otros usuarios y asignarle el rol [Desarrollador de aplicaciones](../active-directory/users-groups-roles/directory-assign-admin-roles.md#application-developer) para que pueda crear entidades de servicio. Para más información sobre los usuarios miembros e invitados y sus permisos, vea [¿Cuales son los permisos de usuario predeterminados en Azure Active Directory?](../active-directory/fundamentals/users-default-permissions.md) Para más información sobre cómo conceder acceso a los usuarios invitados, consulte [Incorporación o eliminación de asignaciones de roles de Azure para usuarios externos invitados mediante Azure Portal](role-assignments-external-users.md).

Tenga en cuenta que los [roles integrados de Azure](../role-based-access-control/built-in-roles.md) son diferentes de los [roles de Azure AD](../active-directory/users-groups-roles/directory-assign-admin-roles.md). Los roles integrados no conceden acceso a Azure AD. Para más información, vea [Descripción de los distintos roles](../role-based-access-control/rbac-and-directory-admin-roles.md).

Para información donde se comparan los usuarios miembros y los usuarios invitados, consulte [¿Cuáles son los permisos de usuario predeterminados en Azure Active Directory?](../active-directory/fundamentals/users-default-permissions.md)

## <a name="remove-a-co-administrator"></a>Eliminación de un coadministrador

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de servicios o coadministrador.

1. Abra [Suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) y seleccione una suscripción.

1. Haga clic en **Control de acceso (IAM).**

1. Haga clic en la pestaña **Administradores clásicos**.

1. Agregue una marca de verificación junto al coadministrador que desea quitar.

1. Haga clic en **Quitar**.

1. En el cuadro de mensaje que aparece, haga clic en **Sí**.

    ![Captura de pantalla donde se quita el coadministrador](./media/classic-administrators/remove-coadmin.png)

## <a name="change-the-service-administrator"></a>Cambiar el administrador de servicios

Solo el administrador de cuenta puede cambiar el administrador de servicios de una suscripción. De forma predeterminada, al registrarse en una suscripción de Azure, el administrador de servicios es también el administrador de cuenta. El usuario con el rol de administrador de cuenta no tiene acceso a Azure Portal. El usuario con el rol de administrador de servicios tiene acceso total a Azure Portal. Si el administrador de cuenta y el administrador de servicios son el mismo usuario y cambia el administrador de servicios a otro usuario distinto, el administrador de cuenta pierde el acceso a Azure Portal. Sin embargo, el administrador de cuenta siempre puede usar el Centro de cuentas para cambiar el administrador de servicio de nuevo a sí mismo.

Siga estos pasos para cambiar el administrador de servicios en el **Centro de cuentas**.

### <a name="account-center"></a>Centro de cuentas

1. Asegúrese de que el escenario sea compatible mediante la comprobación de las [limitaciones para cambiar de administrador de servicios](#limitations-for-changing-the-service-administrator).

1. Inicie sesión en el [Centro de cuentas](https://account.windowsazure.com/subscriptions) como administrador de cuenta.

1. Haga clic en una suscripción.

1. En la parte derecha, haga clic en **Editar detalles de suscripción**.

    ![Captura de pantalla que muestra el botón Editar suscripción en el Centro de cuentas](./media/classic-administrators/editsub.png)

1. En el cuadro **ADMINISTRADOR DE SERVICIOS** , escriba la dirección de correo electrónico del nuevo administrador de servicios.

    ![Captura de pantalla que muestra el cuadro para cambiar el correo electrónico del administrador de servicios](./media/classic-administrators/change-service-admin.png)

1. Haga clic en la marca de verificación para guardar el cambio.

### <a name="limitations-for-changing-the-service-administrator"></a>Limitaciones para cambiar el administrador de servicios

Solo puede haber un administrador de servicios por suscripción de Azure. El cambio del administrador de servicios se comportará de forma diferente en función de si el administrador de la cuenta es un cuenta Microsoft o si se trata de una cuenta de Azure AD (cuenta profesional o educativa).

| Cuenta de administrador de cuenta | ¿Se puede cambiar el administrador de servicios por otra cuenta Microsoft diferente? | ¿Se puede cambiar el administrador de servicios por una cuenta de Azure AD del mismo directorio? | ¿Se puede cambiar el administrador de servicios por una cuenta de Azure AD de un directorio diferente? |
| --- | --- | --- | --- |
| Cuenta Microsoft | Sí | No | No |
| Cuenta de Azure AD | Sí | Sí | No |

Si el administrador de la cuenta es una cuenta de Azure AD, puede cambiar el administrador de servicios por una cuenta de Azure AD del mismo directorio, pero no de un directorio diferente. Por ejemplo, abby@contoso.com puede cambiar el administrador de servicios por bob@contoso.com, pero no puede cambiarlo por john@notcontoso.com a menos que john@notcontoso.com exista en el directorio contoso.com.

Para más información sobre las cuentas Microsoft y de Azure AD, consulte [¿Qué es Azure Active Directory?](../active-directory/fundamentals/active-directory-whatis.md).

## <a name="view-the-account-administrator"></a>Visualización del administrador de cuenta

El administrador de cuenta es el usuario que se registró inicialmente en la suscripción a Azure y es el responsable de la suscripción como propietario de la facturación. Para cambiar el administrador de cuenta de una suscripción, consulte [Transferencia de la propiedad de una suscripción de Azure a otra cuenta](../cost-management-billing/manage/billing-subscription-transfer.md).

Siga estos pasos para ver el administrador de cuenta.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. Abra [Suscripciones](https://portal.azure.com/#blade/Microsoft_Azure_Billing/SubscriptionsBlade) y seleccione una suscripción.

1. Haga clic en **Propiedades**.

    El administrador de cuenta de la suscripción se muestra en el cuadro **Administrador de cuenta**.

    ![Captura de pantalla en la que aparece el administrador de cuenta](./media/classic-administrators/account-admin.png)

## <a name="next-steps"></a>Pasos siguientes

* [Descripción de los distintos roles](../role-based-access-control/rbac-and-directory-admin-roles.md)
* [Incorporación o eliminación de asignaciones de roles de Azure mediante Azure Portal](../role-based-access-control/role-assignments-portal.md)
* [Adición o cambio de los administradores de la suscripción de Azure](../cost-management-billing/manage/add-change-subscription-administrator.md)
