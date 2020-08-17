---
title: Suscripción a Microsoft 365 con una cuenta de Azure
description: Aprenda a crear una suscripción a Microsoft 365 mediante una cuenta de Azure. También puede asociar entre sí cuentas de Azure y Microsoft 365 existentes.
author: JiangChen79
ms.reviewer: adwise
tags: billing,top-support-issue
ms.service: cost-management-billing
ms.topic: conceptual
ms.date: 06/15/2020
ms.author: banders
ms.openlocfilehash: 050bd9c420ccecdb91a6c438782d9be1c6c23b88
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461891"
---
# <a name="sign-up-for-a-microsoft-365-subscription-with-your-azure-account"></a>Suscripción a Microsoft 365 con la cuenta de Azure

Los suscriptores de Azure pueden usar su cuenta de Azure para suscribirse a Microsoft 365. Quienes formen parte de una organización que tenga una suscripción a Azure, pueden crear una suscripción a Microsoft 365 para los usuarios de su cuenta actual de Azure Active Directory (Azure AD). Regístrese en Microsoft 365 con una cuenta que tenga permisos de administrador global o de administrador de facturación en el inquilino de Azure Active Directory. Para más información, consulte [Comprobación de permisos de cuenta en Azure AD](#RoleInAzureAD) y [Asignación de roles de administrador en Azure Active Directory (Azure AD)](../../active-directory/users-groups-roles/directory-assign-admin-roles.md).

Si ya tiene una cuenta de Microsoft 365 y una suscripción a Azure, puede [asociar un inquilino de Microsoft 365 a una suscripción a Azure](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="get-a-microsoft-365-subscription-by-using-your-azure-account"></a>Obtención de una suscripción a Microsoft 365 con una cuenta de Azure

1. Vaya a la [página de productos de Microsoft 365](https://www.microsoft.com/microsoft-365/business/all-business) y seleccione un plan.
2. Haga clic en **Iniciar sesión** en la esquina superior derecha de la página.

    ![captura de pantalla de la página de evaluación gratuita de Microsoft 365](./media/azure-account-for-microsoft-365-subscription/12-office-365-trial-page.png)
3. Inicie sesión con las credenciales de su cuenta de Azure. Si va a crear una suscripción para una organización, use una cuenta de Azure que sea miembro del rol de directorio de administrador global o de administrador de facturación en el inquilino de Azure Active Directory.

    ![Captura de pantalla del inicio de sesión de Microsoft](./media/azure-account-for-microsoft-365-subscription/13-office-365-sign-in.png)
4. Haga clic en **Probar ahora**.

    ![Captura de pantalla que confirma su pedido de Microsoft 365.](./media/azure-account-for-microsoft-365-subscription/14-office-365-confirm-your-order.png)
5. En la página de confirmación del pedido, haga clic en **Continuar**.

    ![Captura de pantalla del recibo del pedido de Microsoft 365](./media/azure-account-for-microsoft-365-subscription/15-office-365-order-receipt.png)

Ahora ya está listo. Si ha creado la suscripción a Microsoft 365 para su organización, siga los pasos siguientes para comprobar que los usuarios de Azure AD aparecen ahora en Microsoft 365.

1. Abra el centro de administración de Microsoft 365.
2. Expanda **USUARIOS** y luego haga clic en **Usuarios activos**.

    ![Captura de pantalla de los usuarios del centro de administración de Microsoft 365](./media/azure-account-for-microsoft-365-subscription/16-microsoft-365-admin-center-users.png)

Una vez completado el registro, la suscripción a Microsoft 365 se agrega a la misma instancia de Azure Active Directory a la que pertenece su suscripción de Azure. Para más información, consulte [Más información acerca de las suscripciones de Azure y Microsoft 365](microsoft-365-account-for-azure-subscription.md#more-about-subs) y [Asociación de las suscripciones de Azure con Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="check-my-account-permissions-in-azure-ad"></a><a id="RoleInAzureAD"></a>Comprobación de los permisos de mi cuenta en Azure AD
1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Haga clic en **Todos los servicios** y, a continuación, busque **Active Directory**.

    ![Captura de pantalla de Active Directory en Azure Portal](./media/azure-account-for-microsoft-365-subscription/billing-more-services-active-directory.png)
3. Haga clic en **Usuarios y grupos** > **Todos los usuarios**.
4. Seleccione el nombre del usuario.

    ![Captura de pantalla que muestra los usuarios de Azure Active Directory](./media/azure-account-for-microsoft-365-subscription/billing-users-groups.png)

5. Haga clic en **Rol del directorio**.

    ![Captura de pantalla que muestra el rol del directorio de Azure Portal](./media/azure-account-for-microsoft-365-subscription/billing-user-directory-role.png)
6.  El rol **Administrador global** o **Administrador limitado** > **Administrador de facturación** es necesario para crear una suscripción a Microsoft 365 para los usuarios de su cuenta actual de Azure Active Directory.

    ![Captura de pantalla que muestra el rol del directorio de administrador de facturación de Azure Portal](./media/azure-account-for-microsoft-365-subscription/billing-directoryrole-limited.png)

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- [Asociación de un inquilino de Microsoft 365a una suscripción de Azure](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md)
