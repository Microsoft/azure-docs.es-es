---
title: Suscripción a Azure con una cuenta de Microsoft 365
description: Aprenda a crear una suscripción de Azure mediante una cuenta de Microsoft 365. También puede asociar entre sí cuentas de Azure y Microsoft 365 existentes.
author: JiangChen79
ms.reviewer: adwise
tags: billing,top-support-issue
ms.service: cost-management-billing
ms.subservice: billing
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: banders
ms.openlocfilehash: 5fef68c9de4dae8063799255fcc7d68ff6b1e6b8
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88684854"
---
# <a name="sign-up-for-an-azure-subscription-with-your-microsoft-365-account"></a>Regístrese para obtener una suscripción a Azure con su cuenta de Microsoft 365

Si tiene una suscripción a Microsoft 365, puede usar su cuenta de Microsoft 365 para crear una suscripción a Azure. Inicie sesión en [Azure Portal](https://portal.azure.com/) con su nombre de usuario y contraseña de Microsoft 365. Si desea configurar máquinas virtuales o usar otros servicios de Azure, debe suscribirse a una suscripción de Azure. Puede compartir su suscripción de Azure con otras personas y [usar el control de acceso basado en rol para administrar el acceso tanto a su suscripción como a sus recursos de Azure](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal).

Si ya tiene una cuenta de Microsoft 365 y una suscripción a Azure, consulte [Asociación de un inquilino de Microsoft 365 a una suscripción a Azure](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="get-an-azure-subscription-using-your-microsoft-365-account"></a>Obtención de una suscripción a Azure con una cuenta de Microsoft 365

Para ahorrar tiempo y evitar una proliferación de cuentas, utilice su nombre de usuario y contraseña de Microsoft 365 para suscribirse a Azure.

1. Suscríbase en [Azure.com](https://account.azure.com/signup?offer=MS-AZR-0044p&appId=docs).
2. Inicie sesión con su nombre de usuario y contraseña de Microsoft 365. La cuenta que use no necesita tener permisos de administrador. Si tiene más de una cuenta de Microsoft 365, asegúrese de utilizar las credenciales de la que desea asociar a su suscripción de Azure.

   ![Captura de pantalla que muestra la página de registro.](./media/microsoft-365-account-for-azure-subscription/billing-sign-in-with-office-365-account.png)

3. Escriba la información necesaria para completar el proceso de suscripción. Si ya tiene una cuenta de Microsoft 365, es posible que una parte de la información no sea necesaria.

    ![Captura de pantalla que muestra el formulario de registro.](./media/microsoft-365-account-for-azure-subscription/billing-azure-sign-up-fill-information.png)

- Si necesita agregar otras personas de su organización a la suscripción de Azure, consulte [Introducción a la administración de acceso en Azure Portal](../../role-based-access-control/overview.md).

## <a name=""></a><a id="more-about-subs">Más información acerca de las suscripciones a Azure y Microsoft 365</a>

Microsoft 365 y Azure usan el servicio Azure AD para administrar usuarios y suscripciones. El directorio de Azure es como un contenedor en el que puede agrupar usuarios y suscripciones. Para poder usar las mismas cuentas de usuario con sus suscripciones de Azure y Microsoft 365, asegúrese de que las suscripciones de Azure se hayan creado en el mismo directorio que las de Microsoft 365. Tenga en cuenta los siguientes puntos:

* Una suscripción se crea en un directorio
* Los usuarios pertenecen a directorios
* Una suscripción se incluye en el directorio del usuario que la crea. Por lo que la suscripción a Microsoft 365 está asociada a la misma cuenta que la suscripción de Azure.
* Las suscripciones de Azure son propiedad de usuarios individuales del directorio
* Las suscripciones a Microsoft 365 pertenecen al propio directorio. Los usuarios con los permisos adecuados en el directorio pueden administrar estas suscripciones.

![Captura de pantalla que muestra la relación entre el directorio, los usuarios y las suscripciones.](./media/microsoft-365-account-for-azure-subscription/19-background-information.png)

Para más información, consulte [Asociación de las suscripciones de Azure con Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

## <a name="need-help-contact-us"></a>¿Necesita ayuda? Póngase en contacto con nosotros.

Si tiene alguna pregunta o necesita ayuda, [cree una solicitud de soporte técnico](https://go.microsoft.com/fwlink/?linkid=2083458).

## <a name="next-steps"></a>Pasos siguientes

- Comparta su suscripción a Azure con otras personas y [use el control de acceso basado en roles para administrar el acceso a su suscripción de Azure y a sus recursos](https://docs.microsoft.com/azure/role-based-access-control/role-assignments-portal).