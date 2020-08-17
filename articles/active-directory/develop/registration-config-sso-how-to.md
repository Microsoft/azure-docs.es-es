---
title: Configuración del inicio de sesión único de la aplicación
description: Aprenda a configurar el inicio de sesión único para una aplicación personalizada desarrollada y registrada con Azure AD.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 07/15/2019
ms.author: ryanwi
ms.openlocfilehash: 465adbb71abaa45160399ecba2ebfb692a8307c2
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120684"
---
# <a name="how-to-configure-single-sign-on-for-an-application"></a>Para configurar el inicio de sesión único para una aplicación

El inicio de sesión único federado (SSO) de la aplicación se habilita automáticamente cuando la federación se realiza a través de Azure AD para OpenID Connect, SAML 2.0, o WS-Fed. Si los usuarios finales tienen que iniciar sesión aunque ya tengan una sesión con Azure AD, es probable que la aplicación no esté configurada correctamente.

* Si utiliza ADAL/MSAL, asegúrese de que **PromptBehavior** está establecido en **Auto** y no en **Always**.

* Si está creando una aplicación móvil, tendrá que configurar otras opciones para habilitar el SSO con y sin intermediación.

En el caso de Android, consulte [Habilitación de SSO entre aplicaciones en Android](../azuread-dev/howto-v1-enable-sso-android.md).<br>

En el caso de iOS, consulte [Habilitación del inicio de sesión único entre aplicaciones en iOS ](../azuread-dev/howto-v1-enable-sso-ios.md).

## <a name="next-steps"></a>Pasos siguientes

[SSO en Azure AD](../manage-apps/what-is-single-sign-on.md)<br>

[Habilitación de SSO entre aplicaciones en Android](../azuread-dev/howto-v1-enable-sso-android.md)<br>

[Habilitación del inicio de sesión único entre aplicaciones en iOS](../azuread-dev/howto-v1-enable-sso-ios.md)<br>

[Integración de aplicaciones con Azure Active Directory](./quickstart-register-app.md)<br>

[Permisos y consentimiento en el punto de conexión de la Plataforma de identidad de Microsoft](./v2-permissions-and-consent.md)<br>

[Azure AD en StackOverflow](https://stackoverflow.com/questions/tagged/azure-active-directory)