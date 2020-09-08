---
title: Cierre de una cuenta profesional o educativa en una organización no administrada de Azure AD
description: Cierre de la cuenta profesional o educativa en una instancia de Azure Active Directory no administrada.
services: active-directory
author: rolyon
manager: mtillman
ms.service: active-directory
ms.subservice: users-groups-roles
ms.topic: how-to
ms.workload: identity
ms.date: 05/20/2019
ms.author: rolyon
ms.reviewer: ''
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 87e03214e2a00a16064763f35f38bdd3ea0e6123
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798336"
---
# <a name="close-your-work-or-school-account-in-an-unmanaged-azure-ad-organization"></a>Cierre de la cuenta profesional o educativa en una organización no administrada de Azure AD

Si es un usuario en una organización de Azure Active Directory (Azure AD) no administrada y ya no necesita usar las aplicaciones de la organización ni mantener asociación alguna con ella, puede cerrar su cuenta en cualquier momento. Una organización no administrada no tiene un administrador global. Los usuarios de una organización no administrada pueden cerrar sus cuentas por sí mismos, sin necesidad de ponerse en contacto con un administrador.

Los usuarios de una organización no administrada se suelen crear durante el registro de autoservicio. Un ejemplo podría ser el de un trabajador de la información de una organización, que se suscribe a un servicio gratuito. Para más información acerca del registro de autoservicio, consulte [¿Qué es el registro de autoservicio de Azure Active Directory?](directory-self-service-signup.md)

[!INCLUDE [GDPR-related guidance](../../../includes/gdpr-intro-sentence.md)]

## <a name="before-you-begin"></a>Antes de empezar

Antes de cerrar la cuenta, debe confirmar los siguientes elementos:

* Asegúrese de que sea un usuario de una organización de Azure AD no administrada. No puede cerrar la cuenta si pertenece a una organización administrada. Si pertenece a una organización administrada y desea cerrar la cuenta, debe ponerse en contacto con el administrador. Para más información sobre cómo determinar si pertenece a una organización no administrada, vea [Eliminar el usuario del inquilino no administrado](/flow/gdpr-dsr-delete#delete-the-user-from-unmanaged-tenant).

* Guarde los datos que desea conservar. Para más información acerca de cómo enviar una solicitud de exportación, consulte [Acceso y exportación de registros generados por el sistema para inquilinos no administrados](/power-platform/admin/powerapps-gdpr-dsr-guide-systemlogs#accessing-and-exporting-system-generated-logs-for-unmanaged-tenants)///.

> [!WARNING]
> El cierre de una cuenta es irreversible. Al cerrar la cuenta, se eliminarán todos los datos personales. Ya no tendrá acceso a la cuenta y a los datos asociados a ella.

## <a name="close-your-account"></a>Cierre de la cuenta

Para cerrar una cuenta profesional o educativa no administrada, siga estos pasos:

1. Inicie sesión para [cerrar la cuenta](https://go.microsoft.com/fwlink/?linkid=873123), utilizando la cuenta que desea cerrar.

1. En **Mis solicitudes de datos**, seleccione **Cerrar cuenta**.

    ![Mis solicitudes de datos: Cerrar cuenta](./media/users-close-account/close-account.png)

1. Vea el mensaje de confirmación y haga clic en **Sí**.

    ![Mis solicitudes de datos: Confirmar cierre](./media/users-close-account/confirm-close.png)

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es el registro de autoservicio de Azure Active Directory?](directory-self-service-signup.md)
- [Eliminar el usuario del inquilino no administrado](/flow/gdpr-dsr-delete#delete-the-user-from-unmanaged-tenant)
- [Acceso y exportación de registros generados por el sistema para inquilinos no administrados](/power-platform/admin/powerapps-gdpr-dsr-guide-systemlogs#accessing-and-exporting-system-generated-logs-for-unmanaged-tenants)///