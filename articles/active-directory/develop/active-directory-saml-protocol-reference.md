---
title: Uso del protocolo SAML por parte de Azure AD | Microsoft Docs
description: En este artículo se proporciona información general de los perfiles SAML de inicio y cierre de sesión único de Azure Active Directory.
services: active-directory
documentationcenter: .net
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 88125cfc-45c1-448b-9903-a629d8f31b01
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/05/2018
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: hirsin
ms.collection: M365-identity-device-management
ms.openlocfilehash: 07d07f73412e889b018c1f667a500d7625912751
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546146"
---
# <a name="how-azure-ad-uses-the-saml-protocol"></a>Uso del protocolo SAML por parte de Azure AD

Azure Active Directory (Azure AD) utiliza el protocolo SAML 2.0 para permitir que las aplicaciones ofrezcan una experiencia de inicio de sesión único a sus usuarios. Los perfiles SAML de [inicio](single-sign-on-saml-protocol.md) y [cierre de sesión único](single-sign-out-saml-protocol.md) de Azure AD explican cómo se usan las aserciones, los protocolos y los enlaces SAML en el servicio de proveedor de identidades.

El protocolo SAML requiere que el proveedor de identidades (Azure AD) y el proveedor de servicios (la aplicación) intercambien información sobre sí mismos.

Cuando una aplicación se registra en Azure AD, el desarrollador de esta registra información relacionada con la federación en Azure AD. Esta información incluye el **URI de redireccionamiento** y de **metadatos** de la aplicación.

Azure AD usa el **URI de metadatos** del servicio en la nube para recuperar la clave de firma y el URI de cierre de sesión. El cliente puede abrir la aplicación en **Azure AD -> Registro de aplicaciones** y, a continuación, en **Configuración -> Propiedades**, puede actualizar la dirección URL de cierre de sesión. De este modo, Azure AD puede enviar la respuesta a la dirección URL correcta. 

Azure Active Directory expone puntos de conexión de inicio y cierre de sesión único comunes y específicos del inquilino (independientes del inquilino). Estas direcciones URL representan ubicaciones direccionables; no se trata únicamente identificadores, así que puede acceder al punto de conexión para leer los metadatos.

* El punto de conexión específico del inquilino se encuentra en `https://login.microsoftonline.com/<TenantDomainName>/FederationMetadata/2007-06/FederationMetadata.xml`. El  *\<TenantDomainName >* marcador de posición representa un nombre de dominio registrado o GUID TenantID de un inquilino de Azure AD. Por ejemplo, los metadatos de federación del inquilino contoso.com están en: https://login.microsoftonline.com/contoso.com/FederationMetadata/2007-06/FederationMetadata.xml

* El punto de conexión independiente del inquilino se encuentra en `https://login.microsoftonline.com/common/FederationMetadata/2007-06/FederationMetadata.xml`. En esta dirección de punto de conexión, se muestra **common**, en lugar de un id. o nombre de dominio del inquilino.

Para obtener información sobre los documentos de metadatos de federación que publica Azure AD, consulte [Metadatos de federación](azure-ad-federation-metadata.md).
