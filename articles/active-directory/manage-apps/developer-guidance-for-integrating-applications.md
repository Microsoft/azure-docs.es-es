---
title: Registro de la aplicación para utilizar Azure Active Directory | Microsoft Docs
description: Escrito para los profesionales de TI, este artículo ofrece instrucciones para integrar las aplicaciones de Azure con Active Directory.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: article
ms.date: 10/30/2018
ms.author: mimart
ms.custom: seohack1
ms.collection: M365-identity-device-management
ms.openlocfilehash: fd51bfd377aed2a7edd8c64d2f3dc99c3388e45a
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824669"
---
# <a name="develop-line-of-business-apps-for-azure-active-directory"></a>Desarrollo de aplicaciones de línea de negocio para Azure Active Directory
Esta guía proporciona información general sobre el desarrollo de aplicaciones de línea de negocio (LoB) para Azure Active Directory (AD) y se ha diseñado expresamente para administradores globales de Active Directory y Office 365.

## <a name="overview"></a>Información general
La creación de aplicaciones integradas con Azure AD proporciona a los usuarios de la organización inicio de sesión único con Office 365. Si tiene la aplicación en Azure AD, podrá controlar la directiva de autenticación de dicha aplicación. Para obtener más información sobre el acceso condicional y cómo proteger las aplicaciones con Multi-Factor Authentication (MFA), consulte [Configuración de las reglas de acceso](../conditional-access/app-based-mfa.md).

Registro de la aplicación para utilizar Azure Active Directory Registrar la aplicación significa que los programadores pueden usar Azure AD para autenticar usuarios y solicitar acceso a recursos de usuarios; por ejemplo, correo electrónico, calendario y documentos.

Cualquier miembro del directorio (no invitados) puede registrar una aplicación, conocido también como *crear un objeto de aplicación*.

El registro de una aplicación permite a cualquier usuario hacer lo siguiente:

* Obtener una identidad para su aplicación que Azure AD reconoce
* Obtener uno o más secretos o claves que la aplicación puede usar para autenticarse a sí misma en AD
* Identificar la aplicación, por ejemplo, con un nombre o logotipo personalizado en el Portal de Azure
* Aplicar las características de autorización de Azure AD en su aplicación, como las siguientes:

  * Control de acceso basado en rol (RBAC)
  * Azure Active Directory como servidor de autorización de OAuth (proteger una API expuesta por la aplicación)
* Indicar los permisos necesarios para que la aplicación funcione según lo previsto, como:

     - Permisos de la aplicación (solo administradores globales). Por ejemplo: pertenencia a roles en otra aplicación de Azure AD o respecto a una suscripción, un grupo de recursos o un recurso de Azure.
     - Permisos delegados (cualquier usuario) Por ejemplo: Azure AD, inicio de sesión y perfil de lectura

> [!NOTE]
> De forma predeterminada, cualquier miembro puede registrar una aplicación. Para obtener información sobre cómo restringir permisos de registro de aplicaciones a miembros específicos, consulte [Cómo se agregan aplicaciones a Azure AD](../develop/active-directory-how-applications-are-added.md#who-has-permission-to-add-applications-to-my-azure-ad-instance).
>
>

A continuación, se indica lo que el administrador global tendrá que hacer para ayudar a los desarrolladores a que su aplicación esté lista para la producción:

* Configurar las reglas de acceso (directiva de acceso/MFA)
* Configurar la aplicación para requerir asignación de usuarios y asignar usuarios
* Suprimir la experiencia de consentimiento del usuario predeterminada

## <a name="configure-access-rules"></a>Configuración de las reglas de acceso
Configure las reglas de acceso por aplicación a las aplicaciones SaaS. Por ejemplo, puede obligar a utilizar MFA o solo permitir el acceso a usuarios de redes de confianza. En el documento de [configuración de las reglas de acceso](../conditional-access/app-based-mfa.md)se muestran los detalles pertinentes.

## <a name="configure-the-app-to-require-user-assignment-and-assign-users"></a>Configurar la aplicación para requerir asignación de usuarios y asignar usuarios
De forma predeterminada, los usuarios pueden acceder a las aplicaciones sin asignación previa. Sin embargo, si la aplicación expone roles o si quiere que aparezca en el panel de acceso de un usuario, debe requerir la asignación de usuarios.

Si es un suscriptor de Azure AD Premium o Enterprise Mobility Suite (EMS), se recomienda encarecidamente utilizar grupos. La asignación de grupos a la aplicación permite delegar la administración del acceso continuo al propietario del grupo. Puede crear el grupo o pedir a la parte responsable de la organización que lo haga usando sus recursos de administración de grupos.

[Asignar usuarios y grupos a una aplicación](methods-for-assigning-users-and-groups.md)  


## <a name="suppress-user-consent"></a>Supresión del consentimiento del usuario
De forma predeterminada, los usuarios otorgan su consentimiento para iniciar sesión. El consentimiento, que se solicita para conceder permisos a una aplicación, puede desconcertar a los usuarios que no estén familiarizados con la toma de este tipo de decisiones.

Para aplicaciones de confianza, puede otorgar el consentimiento para la aplicación en nombre de su organización con el fin de simplificar la experiencia de los usuarios.

Para obtener más información sobre el consentimiento de usuarios y cómo se utiliza esta característica en Azure, consulte [Integración de aplicaciones con Azure Active Directory](../develop/quickstart-v1-integrate-apps-with-azure-ad.md).

## <a name="related-articles"></a>Artículos relacionados
* [Provisión de acceso remoto seguro a aplicaciones locales](application-proxy.md)
* [Administración del acceso a las aplicaciones con Azure AD](what-is-access-management.md)

