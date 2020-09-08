---
title: 'Registro de autoservicio para usuarios comprobados por correo electrónico: Azure AD | Microsoft Docs'
description: Uso del registro de autoservicio en una organización de Azure Active Directory (Azure AD)
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: users-groups-roles
ms.topic: overview
ms.workload: identity
ms.date: 08/13/2020
ms.author: curtand
ms.reviewer: elkuzmen
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3ec578370d32664e36ffcc4bd075c3f6a3ed68ab
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88795358"
---
# <a name="what-is-self-service-sign-up-for-azure-active-directory"></a>¿Qué es el registro de autoservicio de Azure Active Directory?

En este artículo se explica cómo usar el registro de autoservicio para rellenar una organización en Azure Active Directory (Azure AD). Si desea asumir un nombre de dominio de una organización de Azure AD no administrado, consulte [Adquisición de un directorio no administrado como administrador](domains-admin-takeover.md).

## <a name="why-use-self-service-sign-up"></a>¿Por qué usar el registro de autoservicio?

* Permita que los usuarios tengan acceso a los servicios que desean más rápido.
* Cree ofertas basadas en correo electrónico para un servicio.
* Cree flujos de registro basados en correo electrónico que permitan a los usuarios crear identidades rápidamente con sus alias de correo electrónico del trabajo fáciles de recordar.
* Un directorio de Azure AD creado por autoservicio puede convertirse en un directorio administrado que se puede utilizar para otros servicios.

## <a name="terms-and-definitions"></a>Términos y definiciones

* **Registro de autoservicio**: método por el que un usuario se registra en un servicio en la nube y por el que se le crea automáticamente una identidad en Azure AD basada en su dominio de correo electrónico.
* **Directorio de Azure AD no administrado**: directorio donde se crea la identidad. Un directorio no administrado es un directorio que no tiene administrador global.
* **Usuario comprobado por correo electrónico**: tipo de cuenta de usuario en Azure AD. Un usuario que tiene una identidad que se crean automáticamente después de suscribirse a una oferta de autoservicio se conoce como usuario comprobado por correo electrónico. Un usuario comprobado por correo electrónico es un miembro regular de un directorio etiquetado con creationmethod = EmailVerified.

## <a name="how-do-i-control-self-service-settings"></a>¿Cómo controlo la configuración de autoservicio?

Actualmente, los administradores tienen dos controles de autoservicio . Pueden controlar:

* Si los usuarios pueden unirse o no al directorio mediante el correo electrónico.
* Si los usuarios pueden concederse o no licencias para aplicaciones y servicios.

### <a name="how-can-i-control-these-capabilities"></a>¿Cómo puedo controlar estas capacidades?

Un administrador puede configurar estas capacidades con estos parámetros Set-MsolCompanySettings de cmdlet de Azure AD:

* **AllowEmailVerifiedUsers** controla si un usuario puede crear un directorio o unirse a él. Si establece este parámetro en $false, ningún usuario verificado por correo electrónico podrá unirse al directorio.
* **AllowAdHocSubscriptions** controla la capacidad de los usuarios de realizar registros de autoservicio. Si establece el parámetro en $false, ningún usuario podrá realizar registros de autoservicio.
  
AllowEmailVerifiedUsers y AllowAdHocSubscriptions son configuraciones de todo el directorio que se pueden aplicar a un directorio administrado o no administrado. A continuación, se muestra un ejemplo en el que:

* Administra un directorio con un dominio verificado, como contoso.com.
* Usa colaboración B2B de un directorio diferente para invitar a un usuario que aún no existe (userdoesnotexist@contoso.com) en el directorio principal de contoso.com.
* El directorio principal tiene la opción AllowEmailVerifiedUsers activada.

Si se cumplen las condiciones anteriores, a continuación, se crea un usuario miembro en el directorio principal y un usuario invitado B2B en el directorio que invita.

Para más información sobre las suscripciones de Flow y PowerApps, consulte los siguientes artículos:

* [¿Cómo puedo impedir que mis usuarios existentes comiencen a utilizar Power BI?](https://support.office.com/article/Power-BI-in-your-Organization-d7941332-8aec-4e5e-87e8-92073ce73dc5#bkmk_preventjoining)
* [Preguntas y respuestas sobre Flow en su organización](/flow/organization-q-and-a)

### <a name="how-do-the-controls-work-together"></a>¿Cómo funciona los controles conjuntamente?
Estos dos parámetros se pueden usar juntos para definir un control más preciso del registro de autoservicio. Por ejemplo, el comando siguiente permitirá a los usuarios realizar registros de autoservicio, pero solo si estos usuarios ya tienen una cuenta en Azure AD (es decir, los usuarios que necesitan crear primero una cuenta comprobada por correo electrónico no pueden realizar registros de autoservicio):

```powershell
    Set-MsolCompanySettings -AllowEmailVerifiedUsers $false -AllowAdHocSubscriptions $true
```

En el siguiente diagrama de flujo se explican las distintas combinaciones de estos parámetros y las condiciones resultantes para el directorio y el registro de autoservicio.

![diagrama de flujo de los controles del registro de autoservicio](./media/directory-self-service-signup/SelfServiceSignUpControls.png)

Para obtener más información y ejemplos de cómo usar estos parámetros, consulte [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings?view=azureadps-1.0).

## <a name="next-steps"></a>Pasos siguientes

* [Incorporación de su nombre de dominio personalizado a Azure Active Directory](../fundamentals/add-custom-domain.md)
* [Instalación y configuración de Azure PowerShell](/powershell/azure/)
* [Azure PowerShell](/powershell/azure/)
* [Referencia de cmdlets de Azure](/powershell/azure/get-started-azureps)
* [Set-MsolCompanySettings](/powershell/module/msonline/set-msolcompanysettings?view=azureadps-1.0)
* [Cierre de la cuenta profesional o educativa en un directorio no administrado](users-close-account.md)