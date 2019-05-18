---
title: Sincronizar cuentas de asociado local a la nube como usuarios de B2B - Azure Active Directory | Microsoft Docs
description: Proporcione a los asociados externos administrados localmente acceso a recursos locales y en la nube con las mismas credenciales mediante la colaboración B2B de Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 04/24/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2ae75311ab61449f37ccea15a0bcb88fed80c3ed
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65767354"
---
# <a name="grant-locally-managed-partner-accounts-access-to-cloud-resources-using-azure-ad-b2b-collaboration"></a>Conceder a las cuentas de asociado administradas localmente acceso a los recursos en la nube mediante la colaboración B2B de Azure AD

Antes de Azure Active Directory (Azure AD), las organizaciones con sistemas de identidad locales han administrado tradicionalmente las cuentas de asociado en sus directorios locales. En este tipo de organización, cuando comienzan a moverse las aplicaciones a Azure AD, quiere tener la seguridad de que los asociados pueden acceder a los recursos que necesitan. No importa si los recursos se encuentran en el entorno local o en la nube. Además, quiere que los usuarios asociados puedan usar las mismas credenciales de inicio de sesión tanto para los recursos locales como para los de Azure AD. 

Si crea cuentas para los asociados externos en el directorio local (por ejemplo, crea una cuenta con un nombre de inicio de sesión de "wmoran" para un usuario externo llamado Wendy Moran en el dominio partners.contoso.com), ahora puede sincronizarlas con la nube. En concreto, puede usar Azure AD Connect para sincronizar las cuentas de asociado con la nube como usuarios B2B de Azure AD (es decir, usuarios con UserType = Invitado). De esta manera, los usuarios asociados pueden acceder a los recursos en la nube con las mismas credenciales que las de sus cuentas locales, sin concederles más acceso del que necesitan. 

## <a name="identify-unique-attributes-for-usertype"></a>Identificación de atributos únicos para UserType

Antes de permitir la sincronización del atributo UserType, primero debe decidir cómo obtener el atributo UserType de Active Directory local. En otras palabras, ¿qué parámetros del entorno local son únicos para los colaboradores externos? Determine un parámetro que distinga estos colaboradores externos de los miembros de su propia organización.

Los dos enfoques comunes para ello son los siguientes:

- Designe un atributo de Active Directory local sin usar (por ejemplo, Atributodeextensión1) que se usará como el atributo de origen. 
- Como alternativa, obtenga el valor del atributo UserType de otras propiedades. Por ejemplo, desea sincronizar todos los usuarios como invitado si su atributo de UserPrincipalName de Active Directory local finaliza con el dominio  *\@partners.contoso.com*.
 
Para conocer los requisitos detallados de atributos, consulte [Habilitar la sincronización de UserType](../hybrid/how-to-connect-sync-change-the-configuration.md#enable-synchronization-of-usertype). 

## <a name="configure-azure-ad-connect-to-sync-users-to-the-cloud"></a>Configuración de Azure AD Connect para sincronizar los usuarios con la nube

Después de identificar el atributo único, puede configurar Azure AD Connect para sincronizar estos usuarios con la nube como usuarios de Azure AD B2B (es decir, usuarios con UserType = Invitado). Desde un punto de vista de autorización, estos usuarios no se distinguen de los usuarios B2B creados mediante el proceso de invitación a la colaboración B2B de Azure AD.

Para obtener instrucciones de implementación, consulte [Habilitar la sincronización de UserType](../hybrid/how-to-connect-sync-change-the-configuration.md#enable-synchronization-of-usertype).

## <a name="next-steps"></a>Pasos siguientes

- [Colaboración B2B de Azure Active Directory para organizaciones híbridas](hybrid-organizations.md)
- [Conceder a los usuarios B2B de Azure AD acceso a las aplicaciones locales](hybrid-cloud-to-on-premises.md)
- Para información general sobre Azure AD Connect, consulte [Integración de los directorios locales con Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

