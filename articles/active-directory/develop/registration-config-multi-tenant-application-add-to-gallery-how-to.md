---
title: Incorporación de una aplicación multiinquilino a la galería de aplicaciones de Azure AD
description: Se explica cómo mostrar una aplicación multiinquilino personalizada en la galería de aplicaciones de Azure AD.
services: active-directory
documentationCenter: na
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.workload: identity
ms.topic: conceptual
ms.date: 09/11/2018
ms.author: ryanwi
ms.reviewer: jeedes
ms.openlocfilehash: 37f305a7564382b9c6c5a031340d81a22cfedbf2
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120718"
---
# <a name="add-a-multitenant-application-to-the-azure-ad-application-gallery"></a>Incorporación de una aplicación multiinquilino a la galería de aplicaciones de Azure AD

## <a name="what-is-the-azure-ad-application-gallery"></a>¿Qué es la galería de aplicaciones de Azure AD?

Azure Active Directory (Azure AD) es un servicio de identidad basado en la nube. La [galería de aplicaciones de Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureActiveDirectory) es una tienda de aplicaciones de Microsoft Azure Marketplace donde se publican todos los conectores de aplicaciones que permiten el inicio de sesión único y el aprovisionamiento de usuarios. Los clientes que usan Azure AD como proveedor de identidades encontrarán publicados aquí diferentes conectores de aplicaciones SaaS. Los administradores de TI agregan los conectores desde la galería de aplicaciones, y los configuran y usan para poder realizar el inicio de sesión único y el aprovisionamiento. Azure AD admite todos los principales protocolos de federación para el inicio de sesión único, como SAML 2.0, OpenID Connect, OAuth y WS-Fed. 

## <a name="if-your-application-supports-saml-or-openidconnect"></a>Si la aplicación es compatible con SAML o con OpenIDConnect
Si tiene una aplicación multiinquilino que le gustaría mostrar en la galería de aplicaciones de Azure AD, primero debe asegurarse de que admita una de las siguientes tecnologías de inicio de sesión único:

- **OpenID Connect**: para que la aplicación se muestre, cree la aplicación multiinquilino en Azure AD e implemente el [marco de consentimiento de Azure AD](./consent-framework.md) de la misma. Envíe la solicitud de inicio de sesión a un punto de conexión común para que cualquier cliente pueda proporcionar su consentimiento a la aplicación. Puede controlar el acceso de un usuario en función del identificador de inquilino y del UPN del usuario recibido en el token. Para enviar la aplicación, use el proceso subrayado en [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../azuread-dev/howto-app-gallery-listing.md).

- **SAML**: si la aplicación admite SAML 2.0, se puede mostrar en la galería. Siga las instrucciones que se indican en [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../azuread-dev/howto-app-gallery-listing.md).

## <a name="if-your-application-does-not-support-saml-or-openidconnect"></a>Si la aplicación no es compatible con SAML ni con OpenIDConnect
Las aplicaciones que no admitan SAML u OpenIDConnect se pueden integrar en la galería de aplicaciones mediante la tecnología de inicio de sesión único con contraseña.

El inicio de sesión único con contraseña, también conocido como almacenamiento de contraseñas, permite administrar el acceso y las contraseñas de los usuarios en aplicaciones web que no admiten la federación de identidades. También es útil para escenarios en los que varios usuarios deben compartir la misma cuenta, como las cuentas de las aplicaciones de redes sociales de la organización. 

Si desea mostrar su aplicación con esta tecnología:
1. Cree una aplicación web que tenga una página de inicio de sesión HTML para configurar el [inicio de sesión único con contraseña](../manage-apps/what-is-single-sign-on.md). 
2. Envíe la solicitud como se describe en [Aprenda a mostrar su aplicación en la galería de aplicaciones de Azure Active Directory](../azuread-dev/howto-app-gallery-listing.md).

## <a name="escalations"></a>Extensiones

Si necesita ponerse en contacto con una instancia superior, envíe un correo electrónico al [equipo de integración de SSO de Azure AD](<mailto:SaaSApplicationIntegrations@service.microsoft.com>) y le responderemos lo antes posible.

## <a name="next-steps"></a>Pasos siguientes
Aprenda cómo [mostrar la aplicación en la galería de aplicaciones de Azure Active Directory](../azuread-dev/howto-app-gallery-listing.md).