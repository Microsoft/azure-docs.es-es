---
title: 'Colaboración B2B para organizaciones híbridas: Azure Active Directory | Microsoft Docs'
description: Proporcione a los asociados acceso tanto a los recursos locales como en la nube con la colaboración B2B de Azure AD.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 04/26/2018
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: b30e9c14863f67d28203eac916606e8786101e3b
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65768275"
---
# <a name="azure-active-directory-b2b-collaboration-for-hybrid-organizations"></a>Colaboración B2B de Azure Active Directory para organizaciones híbridas

La colaboración B2B de Azure Active Directory (Azure AD) facilita que pueda proporcionar a los asociados externos acceso a las aplicaciones y recursos de la organización. Esto es cierto incluso en una configuración híbrida en la que hay recursos locales y en la nube. No importa si actualmente administra las cuentas de asociados externos localmente en el sistema de identidades o si administra las cuentas externas en la nube como usuarios de Azure AD B2B. Ya puede conceder a estos usuarios acceso a los recursos en cualquier ubicación mediante las mismas credenciales de inicio de sesión para ambos entornos.

## <a name="grant-b2b-users-in-azure-ad-access-to-your-on-premises-apps"></a>Conceder a los usuarios B2B de Azure AD acceso a las aplicaciones locales

Si la organización usa las funcionalidades de colaboración B2B de Azure AD para invitar a los usuarios invitados de organizaciones asociadas a su instancia de Azure AD, ahora puede proporcionar a estos usuarios B2B acceso a las aplicaciones locales.

Para aquellas aplicaciones que usan la autenticación basada en SAML, puede hacer que esas aplicaciones estén disponibles para los usuarios B2B en Azure Portal usando Azure AD Application Proxy para la autenticación.

Para aquellas aplicaciones que usan la autenticación integrada de Windows (IWA) con la delegación restringida de Kerberos (KCD), también puede usar Azure AD Application Proxy para la autenticación. No obstante, para que la autorización funcione, se requiere que un objeto de usuario esté en la instancia local de Windows Server Active Directory. Hay dos métodos que puede usar para crear objetos de usuario locales que representen a los usuarios B2B invitados.

- Puede usar Microsoft Identity Manager (MIM) 2016 SP1 y el agente de administración de MIM para Microsoft Graph.
- Puede usar un script de PowerShell. (Esta solución no necesita MIM).

Para más información sobre cómo implementar estas soluciones, consulte [Conceder a los usuarios B2B de Azure AD acceso a las aplicaciones locales](hybrid-cloud-to-on-premises.md).

## <a name="grant-locally-managed-partner-accounts-access-to-cloud-resources"></a>Conceder a las cuentas de asociado administradas localmente acceso a los recursos en la nube

Antes de Azure AD, las organizaciones con sistemas de identidad locales administraban tradicionalmente las cuentas de asociado en sus directorios locales. Si es una organización de este tipo, querrá asegurarse de que los socios seguirán teniendo acceso cuando mueva las aplicaciones y demás recursos a la nube. Idealmente, querrá que estos usuarios utilicen el mismo conjunto de credenciales para acceder a los recursos locales y en la nube. 

Ahora se ofrecen métodos en los que puede usar Azure AD Connect para sincronizar estas cuentas locales en la nube como "usuarios invitados", en los que las cuentas se comportan de igual manera que los usuarios B2B de Azure AD.

Para ayudar a proteger los datos de su empresa, puede controlar el acceso solo a los recursos adecuados y configurar directivas de autorización que traten a estos usuarios invitados de manera diferente a los empleados.

Para más información sobre la implementación, consulte [Conceder a las cuentas de asociado administradas localmente acceso a los recursos en la nube mediante la colaboración B2B de Azure AD](hybrid-on-premises-to-cloud.md).
 
## <a name="next-steps"></a>Pasos siguientes

- [Conceder a los usuarios B2B de Azure AD acceso a las aplicaciones locales](hybrid-cloud-to-on-premises.md)
- [Conceder a las cuentas de asociado administradas localmente acceso a los recursos en la nube mediante la colaboración B2B de Azure AD](hybrid-on-premises-to-cloud.md)


