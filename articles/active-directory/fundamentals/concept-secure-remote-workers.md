---
title: Respuesta rápida para proteger identidades con Azure Active Directory
description: Responda rápidamente a las amenazas con las identidades basadas en la nube de Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 04/27/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: davidspo
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4f4d9d1a2a4c88601e7dd7e0d6f56025b79aaac1
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90705374"
---
# <a name="rapidly-respond-to-secure-identities-with-azure-ad"></a>Respuesta rápida para proteger identidades con Azure AD

Tratar de proteger a sus trabajadores en el mundo actual puede parecer desalentador, especialmente cuando tiene que responder rápidamente y proporcionar acceso a muchos servicios con rapidez. Este artículo está pensado para proporcionar una lista concisa de todas las acciones que se deben llevar a cabo, lo que ayuda a identificar y priorizar el orden de implementación de las características de Azure AD en función del tipo de licencia que posea. Azure AD ofrece muchas características y proporciona muchos niveles de seguridad para las identidades, ya que determinar qué característica es relevante puede resultar abrumador en ocasiones. Muchas organizaciones ya están en la nube o están migrando rápidamente a la misma; este documento está diseñado para permitirle implementar servicios rápidamente, con la protección de las identidades como consideración principal. 

Cada tabla proporciona una recomendación de seguridad coherente, que protege las identidades de usuario y administrador de los principales ataques de seguridad (la reproducción de infracciones, la suplantación de identidad [phishing] y la difusión de contraseñas), a la vez que minimiza el impacto del usuario y mejora su experiencia. 

Las instrucciones también permitirán a los administradores configurar el acceso a las aplicaciones SaaS y locales de manera segura, se podrán aplicar a las identidades de nube o híbridas (sincronizadas) y se aplicarán a los usuarios que trabajan de forma remota o en la oficina.

Esta lista de comprobación le ayudará a implementar rápidamente las acciones críticas recomendadas para proteger su organización de inmediato; en ella se explica cómo:

- Reforzar las credenciales.
- Reducir el área de la superficie de ataque.
- Automatizar la respuesta frente a amenazas.
- Usar Cloud Intelligence.
- Habilitar el autoservicio del usuario final.

## <a name="prerequisites"></a>Prerrequisitos

En esta guía se da por supuesto que ya se han establecido en Azure AD las identidades híbridas o en la nube. Para obtener ayuda para elegir un tipo de identidad, consulte el artículo [Selección del método de autenticación adecuado para su solución de identidad híbrida de Azure Active Directory](../hybrid/choose-ad-authn.md). 

## <a name="summary"></a>Resumen

Existen muchos aspectos en una infraestructura de identidad segura, pero esta lista de comprobación se centra en una que permite a los usuarios trabajar de forma remota. La protección de su identidad es solo una parte de la seguridad; también se debe tener en cuenta la protección de datos, las aplicaciones y los dispositivos.

### <a name="guidance-for-azure-ad-free-office-365-or-microsoft-365-customers"></a>Guía para clientes de Azure AD Free, Office 365 o Microsoft 365.

Hay una serie de recomendaciones que los clientes de aplicaciones de Azure AD Free, Office 365 o Microsoft 365 deben seguir para proteger sus identidades de usuario. La tabla siguiente tiene como objetivo resaltar las acciones clave de las siguientes suscripciones de licencia:

- Office 365 (Office 365 E1, E3, E5, F1, A1, A3, A5)
- Microsoft 365 (Empresa Básico, Aplicaciones para negocios, Empresa Estándar, Empresa Premium, A1)
- Azure AD Free (incluido con Azure, Dynamics 365, Intune y Power Platform)

| Acción recomendada | Detalle |
| --- | --- |
| [Habilitar los valores predeterminados de seguridad](concept-fundamentals-security-defaults.md) | Proteja todas las aplicaciones e identidades de usuario al habilitar MFA y bloquear la autenticación heredada. |
| [Habilitar la sincronización de hash de contraseñas](../hybrid/how-to-connect-password-hash-synchronization.md) (si se usan identidades híbridas) | Proporcione redundancia para la autenticación y mejore la seguridad (incluido el bloqueo inteligente, el bloqueo de IP y la capacidad de detectar las credenciales filtradas). |
| [Habilitar el bloqueo inteligente de ADFS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) (si procede) | Protege a los usuarios de los bloqueos de cuentas de extranet debido a actividades malintencionadas. |
| [Habilitar el bloqueo inteligente de Azure Active Directory](../authentication/howto-password-smart-lockout.md) (si se usan identidades administradas) | El bloqueo inteligente ayuda a bloquear a los actores malintencionados que intentan adivinar las contraseñas de los usuarios o que usan métodos de fuerza bruta para obtenerlas. |
| [Deshabilitar el consentimiento del usuario final a las aplicaciones](../manage-apps/configure-user-consent.md) | El flujo de trabajo de consentimiento del administrador proporciona a los administradores una manera segura de conceder acceso a las aplicaciones que requieren la aprobación del administrador, de modo que los usuarios finales no expongan los datos de la empresa. Microsoft recomienda deshabilitar las operaciones futuras de consentimiento del usuario para ayudar a reducir el área expuesta y a mitigar este riesgo. |
| [Integrar aplicaciones SaaS compatibles de la galería con Azure AD y habilitar el inicio de sesión único](../manage-apps/add-application-portal.md) | Azure AD incluye una galería que contiene miles de aplicaciones previamente integradas. Algunas de las aplicaciones que su organización usa probablemente estén en la galería y se pueda acceder a ellas desde Azure Portal. Proporcione acceso a las aplicaciones SaaS empresariales de forma remota y segura mediante una experiencia de usuario mejorada (SSO). |
| [Automatizar el aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS](../app-provisioning/user-provisioning.md) (si procede) | Cree automáticamente roles e identidades de usuario en las aplicaciones en la nube (SaaS) a las que los usuarios necesitan acceder. Además de crear identidades de usuario, el aprovisionamiento automático incluye el mantenimiento y la eliminación de identidades de usuario a medida que el estado o los roles cambian, de modo que mejora la seguridad de su organización. |
| [Habilitar el acceso híbrido seguro: Protección de aplicaciones heredadas con redes y controladores de entrega de aplicaciones existentes](../manage-apps/secure-hybrid-access.md) (si procede) | Publique y proteja sus aplicaciones de autenticación heredadas locales y en la nube conectándolas a Azure AD con su red o controlador de entrega de aplicaciones existentes. |
| [Habilitar el autoservicio de restablecimiento de contraseña](../authentication/tutorial-enable-sspr.md) (aplicable solo a cuentas en la nube) | Esta capacidad reduce las llamadas al departamento de soporte técnico y la pérdida de productividad cuando un usuario no puede iniciar sesión en su dispositivo o en una aplicación. |
| [Uso de roles de administrador no global siempre que sea posible](../users-groups-roles/directory-assign-admin-roles.md) | Asigne a los administradores solo el acceso que necesitan a las áreas a las que necesitan acceso. No todos los administradores necesitan ser administradores globales. |
| [Habilitación de la guía de contraseñas de Microsoft](https://www.microsoft.com/research/publication/password-guidance/) | Olvídese de solicitar a los usuarios que cambien periódicamente la contraseña, deshabilite los requisitos de complejidad y a los usuarios les será más fácil recordar la contraseña y mantener una segura. |


### <a name="guidance-for-azure-ad-premium-plan-1-customers"></a>Instrucciones para los clientes del plan 1 de Azure AD Premium

La tabla siguiente tiene como objetivo resaltar las acciones clave de las siguientes suscripciones de licencia:

- Azure Active Directory Premium P1 (Azure AD P1)
- Enterprise Mobility + Security (EMS E3)
- Microsoft 365 (M365 E3, A3, F1, F3)

| Acción recomendada | Detalle |
| --- | --- |
| [Habilitar la experiencia de registro combinada para Azure MFA y SSPR para simplificar la experiencia de registro del usuario](../authentication/howto-registration-mfa-sspr-combined.md) | Permita que los usuarios se registren para una de las experiencias comunes: Azure Multi-Factor Authentication o el autoservicio de restablecimiento de contraseña. |
| [Configurar MFA para su organización](../authentication/howto-mfa-getstarted.md) | Asegúrese de que las cuentas están protegidas frente a los riesgos de la autenticación multifactor. |
| [Habilitar el autoservicio de restablecimiento de contraseña](../authentication/tutorial-enable-sspr.md) | Esta capacidad reduce las llamadas al departamento de soporte técnico y la pérdida de productividad cuando un usuario no puede iniciar sesión en su dispositivo o en una aplicación. |
| [Implementar la escritura diferida de contraseñas](../authentication/tutorial-enable-sspr-writeback.md) (si usa identidades híbridas) | Permita la escritura diferida de los cambios de contraseña en la nube en un entorno de Windows Server Active Directory local. |
| Crear y habilitar directivas de acceso condicional | [MFA para que los administradores protejan las cuentas a las que se asignan derechos administrativos.](../conditional-access/howto-conditional-access-policy-admin-mfa.md) <br><br> [Bloquear los protocolos de autenticación heredados debido al aumento de riesgo asociado a los protocolos de autenticación heredados.](../conditional-access/howto-conditional-access-policy-block-legacy.md) <br><br> [MFA para que todos los usuarios y las aplicaciones creen una directiva de MFA equilibrada para el entorno, protegiendo a los usuarios y las aplicaciones.](../conditional-access/howto-conditional-access-policy-all-users-mfa.md) <br><br> [Exigir que MFA para Azure Management proteja los recursos con privilegios al requerir la autenticación multifactor para cualquier usuario que tenga acceso a recursos de Azure.](../conditional-access/howto-conditional-access-policy-azure-management.md) |
| [Habilitar la sincronización de hash de contraseñas](../hybrid/how-to-connect-password-hash-synchronization.md) (si se usan identidades híbridas) | Proporcione redundancia para la autenticación y mejore la seguridad (incluido el bloqueo inteligente, el bloqueo de IP y la capacidad de detectar las credenciales filtradas). |
| [Habilitar el bloqueo inteligente de ADFS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) (si procede) | Protege a los usuarios de los bloqueos de cuentas de extranet debido a actividades malintencionadas. |
| [Habilitar el bloqueo inteligente de Azure Active Directory](../authentication/howto-password-smart-lockout.md) (si se usan identidades administradas) | El bloqueo inteligente ayuda a bloquear a los actores malintencionados que intentan adivinar las contraseñas de los usuarios o que usan métodos de fuerza bruta para obtenerlas. |
| [Deshabilitar el consentimiento del usuario final a las aplicaciones](../manage-apps/configure-user-consent.md) | El flujo de trabajo de consentimiento del administrador proporciona a los administradores una manera segura de conceder acceso a las aplicaciones que requieren la aprobación del administrador, de modo que los usuarios finales no expongan los datos de la empresa. Microsoft recomienda deshabilitar las operaciones futuras de consentimiento del usuario para ayudar a reducir el área expuesta y a mitigar este riesgo. |
| [Habilitar el acceso remoto a las aplicaciones heredadas locales con Application Proxy](../manage-apps/application-proxy-add-on-premises-application.md) | Habilite Azure Active Directory Application Proxy e intégrela con aplicaciones heredadas para que los usuarios tengan acceso seguro a aplicaciones locales, iniciando sesión con su cuenta de Azure AD. |
| [Habilitar el acceso híbrido seguro: Protección de aplicaciones heredadas con redes y controladores de entrega de aplicaciones existentes](../manage-apps/secure-hybrid-access.md) (si procede) | Publique y proteja sus aplicaciones de autenticación heredadas locales y en la nube conectándolas a Azure AD con su red o controlador de entrega de aplicaciones existentes. |
| [Integrar aplicaciones SaaS compatibles de la galería con Azure AD y habilitar el inicio de sesión único](../manage-apps/add-application-portal.md) | Azure AD incluye una galería que contiene miles de aplicaciones previamente integradas. Algunas de las aplicaciones que su organización usa probablemente estén en la galería y se pueda acceder a ellas desde Azure Portal. Proporcione acceso a las aplicaciones SaaS empresariales de forma remota y segura mediante una experiencia de usuario mejorada (SSO). |
| [Automatizar el aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS](../app-provisioning/user-provisioning.md) (si procede) | Cree automáticamente roles e identidades de usuario en las aplicaciones en la nube (SaaS) a las que los usuarios necesitan acceder. Además de crear identidades de usuario, el aprovisionamiento automático incluye el mantenimiento y la eliminación de identidades de usuario a medida que el estado o los roles cambian, de modo que mejora la seguridad de su organización. |
| [Habilitar el acceso condicional basado en dispositivos](../conditional-access/require-managed-devices.md) | Mejore la seguridad y las experiencias de usuario con el acceso condicional basado en dispositivos. Este paso garantiza que los usuarios solo pueden tener acceso desde dispositivos que cumplen los estándares de seguridad y cumplimiento. A estos dispositivos también se les conoce como dispositivos administrados. Los dispositivos administrados pueden ser conformes con Intune o dispositivos unidos a Azure AD híbrido. |
| [Habilitación de la protección con contraseña](../authentication/howto-password-ban-bad-on-premises-deploy.md) | Proteja a los usuarios del uso de contraseñas poco seguras y fáciles de adivinar. |
| [Designación de más de un administrador global](../users-groups-roles/directory-emergency-access.md) | Asigne al menos dos cuentas de administrador global permanentes solo en la nube para casos de emergencia. Estas cuentas no son para un uso diario y deben tener contraseña compleja y larga. Las cuentas de acceso de emergencia garantizan el acceso al servicio en situaciones de emergencia. |
| [Uso de roles de administrador no global siempre que sea posible](../users-groups-roles/directory-assign-admin-roles.md) | Asigne a los administradores solo el acceso que necesitan a las áreas a las que necesitan acceso. No todos los administradores necesitan ser administradores globales. |
| [Habilitación de la guía de contraseñas de Microsoft](https://www.microsoft.com/research/publication/password-guidance/) | Olvídese de solicitar a los usuarios que cambien periódicamente la contraseña, deshabilite los requisitos de complejidad y a los usuarios les será más fácil recordar la contraseña y mantener una segura. |
| [Creación de un plan para el acceso de usuarios invitados](../external-identities/what-is-b2b.md) | Colabore con los usuarios invitados y permítales iniciar sesión en sus aplicaciones y servicios con sus propias identidades profesionales, educativas o sociales. |

### <a name="guidance-for-azure-ad-premium-plan-2-customers"></a>Instrucciones para los clientes del plan 2 de Azure AD Premium

La tabla siguiente tiene como objetivo resaltar las acciones clave de las siguientes suscripciones de licencia:

- Azure Active Directory Premium P2 (Azure AD P2)
- Enterprise Mobility + Security (EMS E5)
- Microsoft 365 (M365 E5, A5)

| Acción recomendada | Detalle |
| --- | --- |
| [Habilitar la experiencia de registro combinada para Azure MFA y SSPR para simplificar la experiencia de registro del usuario](../authentication/howto-registration-mfa-sspr-combined.md) | Permita que los usuarios se registren para una de las experiencias comunes: Azure Multi-Factor Authentication o el autoservicio de restablecimiento de contraseña. |
| [Configurar MFA para su organización](../authentication/howto-mfa-getstarted.md) | Asegúrese de que las cuentas están protegidas frente a los riesgos de la autenticación multifactor. |
| [Habilitar el autoservicio de restablecimiento de contraseña](../authentication/tutorial-enable-sspr.md) | Esta capacidad reduce las llamadas al departamento de soporte técnico y la pérdida de productividad cuando un usuario no puede iniciar sesión en su dispositivo o en una aplicación. |
| [Implementar la escritura diferida de contraseñas](../authentication/tutorial-enable-sspr-writeback.md) (si usa identidades híbridas) | Permita la escritura diferida de los cambios de contraseña en la nube en un entorno de Windows Server Active Directory local. |
| [Habilitar las directivas de Identity Protection para aplicar el registro de MFA](../identity-protection/howto-identity-protection-configure-mfa-policy.md) | Administre la implementación de Azure Multi-Factor Authentication (MFA). |
| [Habilitar las directivas de riesgo de inicio de sesión y de usuario de Identity Protection](../identity-protection/howto-identity-protection-configure-risk-policies.md) | Habilite las directivas de inicio de sesión y de usuario de Identity Protection. La directiva de inicio de sesión recomendada está dirigida a los inicios de sesión de riesgo medio y requiere de MFA. En el caso de las directivas de usuario, deben dirigirse a los usuarios de alto riesgo que necesitan la acción de cambio de contraseña. |
| Crear y habilitar directivas de acceso condicional | [MFA para que los administradores protejan las cuentas a las que se asignan derechos administrativos.](../conditional-access/howto-conditional-access-policy-admin-mfa.md) <br><br> [Bloquear los protocolos de autenticación heredados debido al aumento de riesgo asociado a los protocolos de autenticación heredados.](../conditional-access/howto-conditional-access-policy-block-legacy.md) <br><br> [Exigir que MFA para Azure Management proteja los recursos con privilegios al requerir la autenticación multifactor para cualquier usuario que tenga acceso a recursos de Azure.](../conditional-access/howto-conditional-access-policy-azure-management.md) |
| [Habilitar la sincronización de hash de contraseñas](../hybrid/how-to-connect-password-hash-synchronization.md) (si se usan identidades híbridas) | Proporcione redundancia para la autenticación y mejore la seguridad (incluido el bloqueo inteligente, el bloqueo de IP y la capacidad de detectar las credenciales filtradas). |
| [Habilitar el bloqueo inteligente de ADFS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) (si procede) | Protege a los usuarios de los bloqueos de cuentas de extranet debido a actividades malintencionadas. |
| [Habilitar el bloqueo inteligente de Azure Active Directory](../authentication/howto-password-smart-lockout.md) (si se usan identidades administradas) | El bloqueo inteligente ayuda a bloquear a los actores malintencionados que intentan adivinar las contraseñas de los usuarios o que usan métodos de fuerza bruta para obtenerlas. |
| [Deshabilitar el consentimiento del usuario final a las aplicaciones](../manage-apps/configure-user-consent.md) | El flujo de trabajo de consentimiento del administrador proporciona a los administradores una manera segura de conceder acceso a las aplicaciones que requieren la aprobación del administrador, de modo que los usuarios finales no expongan los datos de la empresa. Microsoft recomienda deshabilitar las operaciones futuras de consentimiento del usuario para ayudar a reducir el área expuesta y a mitigar este riesgo. |
| [Habilitar el acceso remoto a las aplicaciones heredadas locales con Application Proxy](../manage-apps/application-proxy-add-on-premises-application.md) | Habilite Azure Active Directory Application Proxy e intégrela con aplicaciones heredadas para que los usuarios tengan acceso seguro a aplicaciones locales, iniciando sesión con su cuenta de Azure AD. |
| [Habilitar el acceso híbrido seguro: Protección de aplicaciones heredadas con redes y controladores de entrega de aplicaciones existentes](../manage-apps/secure-hybrid-access.md) (si procede) | Publique y proteja sus aplicaciones de autenticación heredadas locales y en la nube conectándolas a Azure AD con su red o controlador de entrega de aplicaciones existentes. |
| [Integrar aplicaciones SaaS compatibles de la galería con Azure AD y habilitar el inicio de sesión único](../manage-apps/add-application-portal.md) | Azure AD incluye una galería que contiene miles de aplicaciones previamente integradas. Algunas de las aplicaciones que su organización usa probablemente estén en la galería y se pueda acceder a ellas desde Azure Portal. Proporcione acceso a las aplicaciones SaaS empresariales de forma remota y segura mediante una experiencia de usuario mejorada (SSO). |
| [Automatizar el aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS](../app-provisioning/user-provisioning.md) (si procede) | Cree automáticamente roles e identidades de usuario en las aplicaciones en la nube (SaaS) a las que los usuarios necesitan acceder. Además de crear identidades de usuario, el aprovisionamiento automático incluye el mantenimiento y la eliminación de identidades de usuario a medida que el estado o los roles cambian, de modo que mejora la seguridad de su organización. |
| [Habilitar el acceso condicional basado en dispositivos](../conditional-access/require-managed-devices.md) | Mejore la seguridad y las experiencias de usuario con el acceso condicional basado en dispositivos. Este paso garantiza que los usuarios solo pueden tener acceso desde dispositivos que cumplen los estándares de seguridad y cumplimiento. A estos dispositivos también se les conoce como dispositivos administrados. Los dispositivos administrados pueden ser conformes con Intune o dispositivos unidos a Azure AD híbrido. |
| [Habilitación de la protección con contraseña](../authentication/howto-password-ban-bad-on-premises-deploy.md) | Proteja a los usuarios del uso de contraseñas poco seguras y fáciles de adivinar. |
| [Designación de más de un administrador global](../users-groups-roles/directory-emergency-access.md) | Asigne al menos dos cuentas de administrador global permanentes solo en la nube para casos de emergencia. Estas cuentas no son para un uso diario y deben tener contraseña compleja y larga. Las cuentas de acceso de emergencia garantizan el acceso al servicio en situaciones de emergencia. |
| [Uso de roles de administrador no global siempre que sea posible](../users-groups-roles/directory-assign-admin-roles.md) | Asigne a los administradores solo el acceso que necesitan a las áreas a las que necesitan acceso. No todos los administradores necesitan ser administradores globales. |
| [Habilitación de la guía de contraseñas de Microsoft](https://www.microsoft.com/research/publication/password-guidance/) | Olvídese de solicitar a los usuarios que cambien periódicamente la contraseña, deshabilite los requisitos de complejidad y a los usuarios les será más fácil recordar la contraseña y mantener una segura. |
| [Creación de un plan para el acceso de usuarios invitados](../external-identities/what-is-b2b.md) | Colabore con los usuarios invitados y permítales iniciar sesión en sus aplicaciones y servicios con sus propias identidades profesionales, educativas o sociales. |
| [Azure Privileged Identity Management](../privileged-identity-management/pim-configure.md) | Permite administrar, controlar y supervisar el acceso a recursos importantes de la organización, lo que garantiza que los administradores tengan acceso solo cuando sea necesario y reciban aprobación. |

## <a name="next-steps"></a>Pasos siguientes

- Para obtener instrucciones detalladas sobre la implementación de las características individuales de Azure AD, revise los [planes de implementación del proyecto de Azure AD](active-directory-deployment-plans.md).

- Para obtener una lista de comprobación detallada de la implementación de Azure AD, consulte el artículo [Guía de implementación de la característica Azure Active Directory](active-directory-deployment-checklist-p2.md).