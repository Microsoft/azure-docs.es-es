---
title: Lista de comprobación de implementación de Azure AD
description: Lista de comprobación de la implementación de la característica Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.topic: conceptual
ms.date: 07/20/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: martinco
ms.collection: M365-identity-device-management
ms.openlocfilehash: fd33845c331f907dbd5720ac92c6b1c627f01873
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89318416"
---
# <a name="azure-active-directory-feature-deployment-guide"></a>Guía de implementación de la característica Azure Active Directory

Puede parecer muy complicado implementar Azure Active Directory (Azure AD) para su organización y mantener la seguridad. En este artículo se identifican las tareas comunes que los clientes encuentran útiles para completar en fases, a lo largo de 30, 60, 90 días o más, para mejorar su posición de seguridad. Incluso las organizaciones que ya han implementado Azure AD pueden usar esta guía para asegurarse de que sacan el máximo partido de su inversión.

Una infraestructura de identidades bien planeada y ejecutada prepara el terreno para que únicamente los usuarios y los dispositivos conocidos puedan acceder de forma segura a los datos y a las cargas de trabajo de productividad.

Además, los clientes pueden consultar su [puntuación segura de identidad](identity-secure-score.md) para ver hasta qué punto siguen los procedimientos recomendados de Microsoft. Compruebe su puntuación segura antes y después de implementar estas recomendaciones para ver qué tal va en comparación con otros usuarios del sector y otras organizaciones de su tamaño.

## <a name="prerequisites"></a>Prerrequisitos

Muchas de las recomendaciones de esta guía se pueden implementar con Azure AD Free o sin licencia. Cuando se requiera licencia, indicaremos cuál es necesaria como mínimo para completar la tarea.

En las páginas siguientes encontrará información adicional sobre las licencias:

* [Licencias de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/)
* [Microsoft 365 Enterprise](https://www.microsoft.com/en-us/licensing/product-licensing/microsoft-365-enterprise)
* [Enterprise Mobility + Security](https://www.microsoft.com/en-us/licensing/product-licensing/enterprise-mobility-security)
* [Guía de concesión de licencias B2B de Azure AD](../external-identities/licensing-guidance.md)

## <a name="phase-1-build-a-foundation-of-security"></a>Fase 1: Creación de una base de seguridad

En esta fase, los administradores habilitan unas características de seguridad como base de referencia para crear una base más segura y fácil de usar en Azure AD antes de importar o crear cuentas de usuario normales. Esta base fundamental garantiza un estado inicial seguro y que a los usuarios finales solo hay que presentarles los conceptos nuevos una vez.

| Tarea | Detail | Licencia necesaria |
| ---- | ------ | ---------------- |
| [Designación de más de un administrador global](../users-groups-roles/directory-emergency-access.md) | Asigne al menos dos cuentas de administrador global permanentes solo en la nube para casos de emergencia. Estas cuentas no son para un uso diario y deben tener contraseña compleja y larga. | Azure AD Free |
| [Uso de roles de administrador no global siempre que sea posible](../users-groups-roles/directory-assign-admin-roles.md) | Asigne a los administradores solo el acceso que necesitan a las áreas a las que necesitan acceso. No todos los administradores necesitan ser administradores globales. | Azure AD Free |
| [Habilitación de Privileged Identity Management para el seguimiento del uso del rol de administrador](../privileged-identity-management/pim-getting-started.md) | Habilite Privileged Identity Management para empezar el seguimiento del uso del rol de administrador. | Azure AD Premium P2 |
| [Implementación del autoservicio de restablecimiento de contraseña](../authentication/howto-sspr-deployment.md) | Reduzca las llamadas al departamento de soporte técnico para el restablecimiento de contraseña al permitir a los empleados restablecerlas ellos mismos mediante directivas que controle usted como administrador. | |
| [Creación de una lista de contraseñas prohibidas personalizada específica de la organización](../authentication/tutorial-configure-custom-password-protection.md) | Evite que los usuarios creen contraseñas que incluyan palabras o frases comunes de su organización o zona. | |
| [Habilitación de la integración local con la protección de contraseñas de Azure AD](../authentication/concept-password-ban-bad-on-premises.md) | Amplíe la lista de contraseñas prohibidas al directorio local para garantizar que las contraseñas locales también cumplen los requisitos de las listas de contraseñas prohibidas globales y específicas del inquilino. | Azure AD Premium P1 |
| [Habilitación de la guía de contraseñas de Microsoft](https://www.microsoft.com/research/publication/password-guidance/) | Olvídese de solicitar a los usuarios que cambien periódicamente la contraseña, deshabilite los requisitos de complejidad y a los usuarios les será más fácil recordar la contraseña y mantener una segura. | Azure AD Free |
| [Deshabilitación de los restablecimientos de contraseña periódicos para las cuentas de usuario en la nube](../authentication/concept-sspr-policy.md#set-a-password-to-never-expire) | Los restablecimientos de contraseña periódicos promueven un aumento de las contraseñas existentes para los usuarios. Sírvase de la documentación de guía de las directrices de Microsoft sobre las contraseñas y copie la directiva local con los usuarios únicamente de la nube. | Azure AD Free |
| [Personalización del bloqueo inteligente de Azure Active Directory](../authentication/howto-password-smart-lockout.md) | Deje de tener bloqueos de los usuarios de la nube al replicarlos a usuarios de Active Directory locales. | |
| [Habilitación del bloqueo inteligente de la extranet para AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-extranet-smart-lockout-protection) | El bloqueo de la extranet de AD FS protege contra los ataques de adivinación de contraseñas por fuerza bruta al tiempo que permite a los usuarios de AD FS válidos continuar usando sus cuentas. | |
| [Bloqueo de la autenticación heredada en Azure AD con Acceso condicional](../conditional-access/block-legacy-authentication.md) | Bloquee los protocolos de autenticación heredados, como POP, SMTP, IMAP y MAPI, no puedan aplicar Multi-Factor Authentication, lo que los convierte en los puntos de entrada preferidos de los adversarios. | Azure AD Premium P1 |
| [Implementación de Azure AD Multi-Factor Authentication mediante directivas de acceso condicional](../authentication/howto-mfa-getstarted.md) | Haga que los usuarios realicen la verificación en dos pasos al acceder a aplicaciones confidenciales mediante directivas de acceso condicional. | Azure AD Premium P1 |
| [Habilitación de Azure Active Directory Identity Protection](../identity-protection/overview-identity-protection.md) | Habilite el seguimiento de los inicios de sesión de riesgo y de las credenciales en riesgo para los usuarios de su organización. | Azure AD Premium P2 |
| [Uso de detecciones de riesgos para desencadenar Multi-Factor Authentication y cambios de contraseñas](../authentication/tutorial-risk-based-sspr-mfa.md) | Habilite la automatización que desencadene eventos como la autenticación multifactor, el restablecimiento de contraseña y el bloqueo del inicio de sesión en caso de riesgo. | Azure AD Premium P2 |
| [Habilitación del registro combinado para autoservicio de restablecimiento de contraseña y Azure AD Multi-Factor Authentication](../authentication/concept-registration-mfa-sspr-combined.md) | Permita que los usuarios se registren para una de las experiencias comunes: Azure Multi-Factor Authentication o el autoservicio de restablecimiento de contraseña. | Azure AD Premium P1 |

## <a name="phase-2-import-users-enable-synchronization-and-manage-devices"></a>Fase 2: Importación de usuarios, habilitación de la sincronización y administración de dispositivos

A continuación agregaremos la base de la fase 1 mediante la importación de nuestros usuarios y la habilitación de la sincronización, la planeación del acceso de invitados y la preparación de la compatibilidad con otras funcionalidades.

| Tarea | Detail | Licencia necesaria |
| ---- | ------ | ---------------- |
| [Instalación de Azure AD Connect](../hybrid/how-to-connect-install-select-installation.md) | Prepárese para sincronizar usuarios de su directorio local existente con la nube. | Azure AD Free |
| [Implementación de la sincronización de hash de contraseñas](../hybrid/how-to-connect-password-hash-synchronization.md) | Sincronice los hash de contraseña para permitir que se repliquen los cambios de contraseña, la detección y la solución de problemas con la contraseña y los informes de revelación de credenciales. | Azure AD Free |
| [Implementación de la escritura diferida de contraseñas](../authentication/tutorial-enable-sspr-writeback.md) | Permita la escritura diferida de los cambios de contraseña en la nube en un entorno de Windows Server Active Directory local. | Azure AD Premium P1 |
| [Implementación de Azure AD Connect Health](../hybrid/whatis-azure-ad-connect.md#what-is-azure-ad-connect-health) | Habilite la supervisión de estadísticas de las claves de estado para los servidores de Azure AD Connect y de AD FS, y los controladores de dominio. | Azure AD Premium P1 |
| [Asignación de licencias a usuarios según su pertenencia a un grupo en Azure Active Directory](../users-groups-roles/licensing-groups-assign.md) | Ahorre tiempo y esfuerzo mediante la creación de grupos de licencias que habiliten o deshabiliten características por grupo en lugar de la configuración por usuario. | |
| [Creación de un plan para el acceso de usuarios invitados](../external-identities/what-is-b2b.md) | Colabore con los usuarios invitados y permítales iniciar sesión en las aplicaciones y los servicios con sus propias identidades profesionales, educativas o sociales. | [Guía de concesión de licencias B2B de Azure AD](../external-identities/licensing-guidance.md) |
| [Decisión acerca de la estrategia de administración de dispositivos](../devices/overview.md) | Decida lo que permite su organización con respecto a los dispositivos. Registro o unión, dispositivos propios o proporcionados por la empresa, etc. | |
| [Implementación de Windows Hello para empresas en su organización](/windows/security/identity-protection/hello-for-business/hello-manage-in-organization) | Prepárese para la autenticación sin contraseña con Windows Hello. | |
| [Implementar métodos de autenticación sin contraseña para los usuarios](../authentication/concept-authentication-passwordless.md) | Proporcione a los usuarios métodos cómodos de autenticación sin contraseñas. | Azure AD Premium P1 |

## <a name="phase-3-manage-applications"></a>Fase 3: Administración de aplicaciones

Siguiendo con la compilación de las fases anteriores, identificaremos aplicaciones candidatas para la migración y la integración con Azure AD y completaremos la configuración de esas aplicaciones.

| Tarea | Detail | Licencia necesaria |
| ---- | ------ | ---------------- |
| Identificación de las aplicaciones | Identifique las aplicaciones que se usan en la organización: locales, SaaS en la nube y otras de línea de negocio. Determinar si estas aplicaciones pueden y deben administrarse con Azure AD. | Sin necesidad de licencia |
| [Integración de aplicaciones SaaS compatibles en la galería](../manage-apps/add-application-portal.md) | Azure AD incluye una galería que contiene miles de aplicaciones previamente integradas. Algunas de las aplicaciones que su organización usa probablemente estén en la galería y se pueda acceder a ellas desde Azure Portal. | Azure AD Free |
| [Uso de un proxy de aplicación para integrar las aplicaciones locales](../manage-apps/application-proxy-add-on-premises-application.md) | Un proxy de aplicación permite a los usuarios acceder a aplicaciones locales mediante el sesión con su cuenta de Azure AD. | |

## <a name="phase-4-audit-privileged-identities-complete-an-access-review-and-manage-user-lifecycle"></a>Fase 4: Auditoría de las identidades con permisos, revisión del acceso y administración del ciclo de vida de los usuarios

En la fase 4 los administradores deben aplicar los últimos principios de los permisos de administración, completar sus primeras revisiones de acceso y permitir la automatización de las tareas comunes del ciclo de vida de los usuarios.

| Tarea | Detail | Licencia necesaria |
| ---- | ------ | ---------------- |
| [Inicio del uso de Privileged Identity Management](../privileged-identity-management/pim-security-wizard.md) | Elimine los roles de administrador de las cuentas de usuario diarias normales. Haga que los usuarios administradores puedan usar su rol tras la comprobación de la autenticación multifactor para proporcionar una justificación de negocios o solicitar la autorización de los aprobadores designados. | Azure AD Premium P2 |
| [Completar una revisión de acceso para los roles de directorio de Azure AD en PIM](../privileged-identity-management/pim-how-to-start-security-review.md) | Trabaje con los equipos de seguridad y dirección para crear una directiva de revisión de acceso para revisar el acceso de los administradores de conformidad con las directivas de la organización. | Azure AD Premium P2 |
| [Implementación de directivas de pertenencia dinámica a grupos](../users-groups-roles/groups-dynamic-membership.md) | Use grupos dinámicos para asignar automáticamente usuarios a grupos en función de sus atributos de RR. HH. (o su origen fiable), como el departamento, el puesto, la región y otros atributos. |  |
| [Implementación del aprovisionamiento de aplicaciones basado en grupos](../manage-apps/what-is-access-management.md) | Use el aprovisionamiento de administración del acceso basado en grupos para aprovisionar usuarios automáticamente para las aplicaciones SaaS. |  |
| [Automatización del aprovisionamiento y desaprovisionamiento de usuarios](../app-provisioning/user-provisioning.md) | Elimine los pasos manuales del ciclo de vida de la cuenta del empleado para evitar el acceso no autorizado. Sincronización de identidades desde un origen fiable (sistema de RR. HH.) en Azure AD. |  |

## <a name="next-steps"></a>Pasos siguientes

[Detalles de precios y licencias de Azure AD](https://azure.microsoft.com/pricing/details/active-directory/)

[Configuraciones de acceso de dispositivos e identidades](/microsoft-365/enterprise/microsoft-365-policies-configurations)

[Directivas comunes de acceso a dispositivos e identidades](/microsoft-365/enterprise/identity-access-policies)