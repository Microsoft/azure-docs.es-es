---
title: 'Información general sobre la verificación del publicador: plataforma de identidad de Microsoft | Azure'
description: Proporciona información general sobre el programa de verificación del publicador (versión preliminar) para la plataforma de identidad de Microsoft. Enumera las ventajas, los requisitos del programa y las preguntas más frecuentes. Cuando una aplicación está marcada como comprobada por el publicador, significa que el publicador ha comprobado su identidad con una cuenta de Microsoft Partner Network que ha completado el proceso de verificación y ha asociado esta cuenta de MPN con el registro de aplicación.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/19/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: jesakowi
ms.openlocfilehash: f611eefc50fede2ef4d738cd5abfd6afcc08b9ff
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120786"
---
# <a name="publisher-verification-preview"></a>Comprobación del publicador (versión preliminar)

La verificación del publicador (versión preliminar) ayuda a los administradores y usuarios finales a comprender la autenticidad de los desarrolladores de aplicaciones que se integran con la plataforma de identidad de Microsoft. Cuando una aplicación se marca como comprobada por el publicador, significa que el publicador ha comprobado su identidad con una cuenta de [Microsoft Partner Network](https://partner.microsoft.com/membership) que ha completado el proceso de [verificación](/partner-center/verification-responses) y ha asociado esta cuenta de MPN con el registro de aplicación. 

Aparece una notificación "comprobado" de color azul en la petición de consentimiento de Azure AD y en otras pantallas: ![Petición de consentimiento](./media/publisher-verification-overview/consent-prompt.png)

Esta característica es principalmente para desarrolladores que compilan aplicaciones multiinquilino que utilizan [OAuth 2.0 y OpenID Connect](active-directory-v2-protocols.md) con la [plataforma de identidad de Microsoft](v2-overview.md). Estas aplicaciones pueden iniciar la sesión de los usuarios mediante OpenID Connect, o bien pueden usar OAuth 2.0 para solicitar acceso a los datos mediante API como [Microsoft Graph](https://developer.microsoft.com/graph/).

## <a name="benefits"></a>Ventajas
La verificación del publicador proporciona las ventajas siguientes:
- **Mayor transparencia y reducción de riesgos para los clientes**: esta funcionalidad ayuda a los clientes a comprender cuáles de las aplicaciones que se usan en sus organizaciones están publicadas por desarrolladores en los que confían. 

- **Personalización de marca mejorada**: aparece una notificación "comprobado" en la [petición de consentimiento](application-consent-experience.md) de Azure AD, la página Aplicaciones empresariales y las superficies de experiencia de usuario adicionales que usan los usuarios finales y los administradores. 

- **Adopción empresarial más fluida**: los administradores pueden configurar nuevas directivas de consentimiento del usuario, y el estado de verificación del publicador será uno de los criterios de la directiva principal. 

- **Evaluación de riesgos mejorada**: las detecciones que Microsoft lleve a cabo de solicitudes de consentimiento "arriesgadas" incluirán la verificación del publicador como una señal. 

## <a name="requirements"></a>Requisitos
Hay algunos requisitos previos para la verificación del publicador, algunos de los cuales ya cumplirán muchos asociados de Microsoft. Son las siguientes: 

-  Un id. de MPN de una cuenta de [Microsoft Partner Network](https://partner.microsoft.com/membership) válida que haya completado el proceso de [verificación](/partner-center/verification-responses). Esta cuenta de MPN debe ser la [cuenta global de asociado (PGA)](/partner-center/account-structure#the-top-level-is-the-partner-global-account-pga) de su organización. 

-  Un inquilino de Azure AD con un [dominio personalizado](../fundamentals/add-custom-domain.md) verificado por DNS. El dominio personalizado debe coincidir con el dominio de la dirección de correo electrónico utilizada durante la comprobación en el paso anterior. 

-  Una aplicación registrada en un inquilino de Azure AD, con un [dominio de publicador](howto-configure-publisher-domain.md) configurado con el mismo dominio que se usó anteriormente. 

-  El usuario que realiza la comprobación debe estar autorizado para realizar cambios en el registro de aplicación en Azure AD y en la cuenta de MPN del Centro de partners. 

    -  En Azure AD, este usuario debe ser el propietario de la aplicación o tener uno de los siguientes [roles](../users-groups-roles/directory-assign-admin-roles.md): Administrador de aplicaciones, administrador de aplicaciones en la nube o administrador global. 

    -  En el Centro de partners, este usuario debe tener uno de los siguientes [roles](/partner-center/permissions-overview): Administrador de MPN, administrador de cuentas o administrador global (se trata de un rol compartido que se controla en Azure AD).
    
-  El publicador acepta las [Condiciones de uso de la plataforma de identidad de Microsoft para desarrolladores](/legal/microsoft-identity-platform/terms-of-use).

Los desarrolladores que ya han cumplido estos requisitos previos pueden comprobarse en cuestión de minutos. Si no se cumplen los requisitos, la configuración es gratuita. 

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes 
A continuación se muestran algunas de las preguntas más frecuentes sobre el programa de comprobación del publicador. Para obtener las preguntas más frecuentes relacionadas con los requisitos y el proceso, consulte [marcar una aplicación como comprobada por el publicador](mark-app-as-publisher-verified.md).

- **¿Qué información __no__ proporciona el publicador de comprobación?**  Que una aplicación esté marcada como comprobada por el publicador no indica que la aplicación o el publicador hayan logrado alguna certificación específica, cumplan con los estándares del sector, sigan los procedimientos recomendados, etc. Otros programas de Microsoft proporcionan esta información, como [Certificación de aplicaciones de Microsoft 365](/microsoft-365-app-certification/overview).

- **¿Cuánto cuesta? ¿Requiere alguna licencia?** Microsoft no cobra a los desarrolladores por la comprobación del publicador ni requiere ninguna licencia específica. 

- **¿Cómo se relaciona con Atestación del publicador de Microsoft 365? ¿Qué ocurre con Certificación de aplicaciones de Microsoft 365?** Se trata de programas complementarios que los desarrolladores pueden utilizar para crear aplicaciones confiables que los clientes pueden adoptar con confianza. La comprobación del publicador es el primer paso de este proceso y deben completarla todos los programadores que crean aplicaciones que cumplen los criterios anteriores. 

  Los desarrolladores que también se integran con Microsoft 365 pueden recibir ventajas adicionales de estos programas. Para obtener más información, consulte [Atestación del publicador de Microsoft 365](/microsoft-365-app-certification/docs/attestation) y [Certificación de aplicaciones de Microsoft 365](/microsoft-365-app-certification/docs/certification). 

- **¿Es lo mismo que la Galería de aplicaciones de Azure AD?** No: la comprobación del publicador es un programa complementario, aunque independiente, para la [Galería de aplicaciones de Azure Active Directory](../azuread-dev/howto-app-gallery-listing.md). Los desarrolladores que cumplen los criterios anteriores deben completar el proceso de comprobación del publicador independientemente de la participación en dicho programa. 

## <a name="next-steps"></a>Pasos siguientes
* Obtenga información sobre cómo [marcar una aplicación como comprobada por el publicador](mark-app-as-publisher-verified.md).
* [Solucione los problemas](troubleshoot-publisher-verification.md) de comprobación del publicador.