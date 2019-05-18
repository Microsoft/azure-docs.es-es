---
title: Protocolos de autenticación de Azure Active Directory | Microsoft Docs
description: Información general sobre los protocolos de autenticación admitidos por Azure Active Directory (AD).
documentationcenter: dev-center-name
author: ryanwi
services: active-directory
manager: CelesteDG
editor: ''
ms.assetid: 7a838ae2-c24c-4304-b6c0-e77fb888e6c0
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/27/2017
ms.author: rwike77
ms.custom: aaddev
ms.reviewer: hirsin
ms.collection: M365-identity-device-management
ms.openlocfilehash: d36e4304c4775ac335c1cc405ee8c6a0208da52a
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546216"
---
# <a name="azure-active-directory-authentication-protocols"></a>Protocolos de autenticación de Azure Active Directory
Azure Active Directory (Azure AD) admite varios de los protocolos de autenticación y autorización utilizados más comúnmente. En los temas de esta sección se describen los protocolos admitidos y su implementación en Azure AD. En los temas se incluye una revisión de los tipos de notificación compatibles, una introducción al uso de los metadatos de federación, documentación de referencia detallada sobre los protocolos OAuth 2.0. y SAML 2.0, y una sección de solución de problemas.

## <a name="authentication-protocols-articles-and-reference"></a>Artículos y referencia de protocolos de autenticación
* [Información importante acerca de la cadencia de sustitución de clave en Azure AD](active-directory-signing-key-rollover.md) : obtenga información sobre la cadencia de sustitución de clave de firma de Azure AD, los cambios que puede realizar para actualizar la clave automáticamente y una explicación sobre cómo actualizar los escenarios de aplicación más comunes.
* [Tipos de token y notificación compatibles](v1-id-and-access-tokens.md): obtenga información sobre las notificaciones en los tokens que emite Azure AD.
* [Metadatos de federación](azure-ad-federation-metadata.md) : obtenga información sobre cómo buscar e interpretar los documentos de metadatos que Azure AD genera.
* [OAuth 2.0 en Azure AD](v1-protocols-oauth-code.md) : obtenga información sobre la implementación de OAuth 2.0 en Azure AD.
* [OpenID Connect 1.0](v1-protocols-openid-connect-code.md) : obtenga información sobre cómo usar el protocolo de autorización OAuth 2.0 para la autenticación.
* [Llamadas de servicio a servicio con credenciales de cliente](v1-oauth2-client-creds-grant-flow.md): aprenda a usar el flujo de concesión de credenciales de cliente de OAuth 2.0 para las llamadas de servicio a servicio.
* [Llamadas de servicio a servicio con el flujo En nombre de](v1-oauth2-on-behalf-of-flow.md): aprenda a usar el flujo "En nombre de" de OAuth 2.0 en las llamadas de servicio a servicio.
* [Referencia del protocolo SAML](active-directory-saml-protocol-reference.md) : obtenga información sobre los perfiles SAML de inicio de sesión único y de cierre de sesión único de Azure AD.

## <a name="see-also"></a>Vea también
[Guía del desarrollador de Azure Active Directory](v1-overview.md)

[Ejemplos de código de Azure Active Directory](sample-v1-code.md)
