---
title: Configuración de una nueva aplicación multiinquilino | Microsoft Docs
description: Aprenda a configurar el inicio de sesión único para una aplicación personalizada desarrollada y registrada con Azure AD.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: ryanwi
ms.collection: M365-identity-device-management
ms.openlocfilehash: 83fae56dd0cf7157575b7c5a07e33ca1888d8560
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545449"
---
# <a name="how-to-configure-a-new-multi-tenant-application"></a>Configuración de una nueva aplicación multiinquilino

El inicio de sesión único federado (SSO) de la aplicación se habilita automáticamente cuando la federación se realiza a través de Azure AD para OpenID Connect, SAML 2.0, o WS-Fed. Si los usuarios finales tienen que iniciar sesión aunque ya tengan una sesión con Azure AD, es probable que la aplicación no esté configurada correctamente.

* Si utiliza ADAL/MSAL, asegúrese de que **PromptBehavior** está establecido en **Auto** y no en **Always**.

* Si está creando una aplicación móvil, tendrá que configurar otras opciones para habilitar el SSO con y sin intermediación.

En el caso de Android, consulte [Habilitación de SSO entre aplicaciones en Android](https://docs.microsoft.com/azure/active-directory/develop/active-directory-sso-android).<br>

En el caso de iOS, consulte [Habilitación del inicio de sesión único entre aplicaciones en iOS ](https://docs.microsoft.com/azure/active-directory/develop/active-directory-sso-ios).

## <a name="next-steps"></a>Pasos siguientes

[SSO en Azure AD](https://docs.microsoft.com/azure/active-directory/active-directory-appssoaccess-whatis)<br>

[Habilitación de SSO entre aplicaciones en Android](https://docs.microsoft.com/azure/active-directory/develop/active-directory-sso-android)<br>

[Habilitación del inicio de sesión único entre aplicaciones en iOS](https://docs.microsoft.com/azure/active-directory/develop/active-directory-sso-ios)<br>

[Integración de aplicaciones con Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/active-directory-integrating-applications)<br>

[Consentimiento y permisos para las aplicaciones convergentes v2.0 de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/active-directory-v2-scopes)<br>

[Azure AD en StackOverflow](https://stackoverflow.com/questions/tagged/azure-active-directory)
