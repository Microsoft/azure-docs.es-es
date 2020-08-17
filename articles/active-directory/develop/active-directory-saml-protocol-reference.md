---
title: Uso del protocolo SAML por parte de la Plataforma de identidad de Microsoft
description: En este artículo se proporciona información general de los perfiles SAML de inicio y cierre de sesión único de Azure Active Directory.
services: active-directory
author: kenwith
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 10/05/2018
ms.author: kenwith
ms.custom: aaddev
ms.reviewer: paulgarn
ms.openlocfilehash: 06f80f94be25e42c9e8f0270e6cb15aca086ae18
ms.sourcegitcommit: 1b2d1755b2bf85f97b27e8fbec2ffc2fcd345120
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87552805"
---
# <a name="how-microsoft-identity-platform-uses-the-saml-protocol"></a>Uso del protocolo SAML por parte de la Plataforma de identidad de Microsoft

La Plataforma de identidad de Microsoft usa el protocolo SAML 2.0 para permitir a las aplicaciones proporcionar una experiencia de inicio de sesión único a sus usuarios. Los perfiles SAML de [inicio](single-sign-on-saml-protocol.md) y [cierre de sesión único](single-sign-out-saml-protocol.md) de Azure AD explican cómo se usan las aserciones, los protocolos y los enlaces SAML en el servicio de proveedor de identidades.

El protocolo SAML requiere que el proveedor de identidades (Plataforma de identidad de Microsoft) y el proveedor de servicios (la aplicación) intercambien información sobre sí mismos.

Cuando una aplicación se registra en Azure AD, el desarrollador de esta registra información relacionada con la federación en Azure AD. Esta información incluye el **URI de redireccionamiento** y de **metadatos** de la aplicación.

La Plataforma de identidad de Microsoft usa el **URI de metadatos** del servicio en la nube para recuperar la clave de firma y el URI de cierre de sesión. El cliente puede abrir la aplicación en **Azure AD -> Registro de aplicaciones** y, a continuación, en **Configuración -> Propiedades**, puede actualizar la dirección URL de cierre de sesión. De este modo, la Plataforma de identidad de Microsoft puede enviar la respuesta a la dirección URL correcta. 

Azure Active Directory expone puntos de conexión de inicio y cierre de sesión único comunes y específicos del inquilino (independientes del inquilino). Estas direcciones URL representan ubicaciones direccionables; no se trata únicamente identificadores, así que puede acceder al punto de conexión para leer los metadatos.

* El punto de conexión específico del inquilino se encuentra en `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml`. El marcador de posición *\<TenantDomainName>* representa un nombre de dominio registrado o GUID TenantID de un inquilino de Azure AD. Por ejemplo, los metadatos de federación del inquilino contoso.com están en: https://login.microsoftonline.com/contoso.com/FederationMetadata/2007-06/FederationMetadata.xml

* El punto de conexión independiente del inquilino se encuentra en `https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml`. En esta dirección de punto de conexión, se muestra **common**, en lugar de un id. o nombre de dominio del inquilino.

Para obtener información sobre los documentos de metadatos de federación que publica Azure AD, consulte [Metadatos de federación](../azuread-dev/azure-ad-federation-metadata.md).
