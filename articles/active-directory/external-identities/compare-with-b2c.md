---
title: 'Comparación de External Identities: Azure Active Directory | Microsoft Docs'
description: Azure AD External Identities permite que personas ajenas a la organización tengan acceso a sus aplicaciones y recursos mediante su propia identidad. Compare soluciones para External Identities, incluida la colaboración B2B de Azure Active Directory y Azure AD B2C.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: overview
ms.date: 08/05/2020
ms.custom: project-no-code
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.collection: M365-identity-device-management
ms.openlocfilehash: 08623e6d69ac4db1790c1e9b46089f0c72c0526d
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87910585"
---
# <a name="what-are-external-identities-in-azure-active-directory"></a>¿Qué son External Identities de Azure Active Directory?

Mediante External Identities de Azure AD, puede facilitar a personas ajenas a una organización el acceso a sus aplicaciones y recursos, además de permitirles iniciar sesión con la identidad que prefieran. Los asociados, distribuidores, proveedores y otros usuarios invitados pueden "traer sus propias identidades". Tanto si forman parte de Azure AD u otro sistema administrado por TI, como si tienen una identidad social no administrada, como Google o Facebook, pueden usar sus propias credenciales para iniciar sesión. El proveedor de identidades administra la identidad del usuario externo y usted administra el acceso a sus aplicaciones con Azure AD para mantener protegidos los recursos. 

## <a name="external-identities-scenarios"></a>Escenarios de External Identities

Azure AD External Identities se centra menos en la relación de un usuario con su organización y más en la forma en que un usuario quiere iniciar sesión en sus aplicaciones y recursos. Dentro de este marco, Azure AD admite una variedad de escenarios, desde la colaboración de negocio a negocio (B2B) al desarrollo de aplicaciones para clientes y consumidores (negocio a consumidor o B2C).

- **Comparta aplicaciones con usuarios externos (colaboración B2B)** . Invite a usuarios externos a su propio inquilino como usuarios "invitados" a los que puede asignar permisos (para autorización) al tiempo que les permite usar sus credenciales existentes (para autenticación). Los usuarios inician sesión en los recursos compartidos con un proceso sencillo de invitación y canje con su cuenta profesional o educativa o con cualquier cuenta de correo electrónico. Y ahora con la disponibilidad de los flujos de usuario de registro de autoservicio (versión preliminar), también pude proporcionar una experiencia de inicio de sesión para los usuarios externos a través de la aplicación que quiera compartir. Puede definir la configuración de flujo de usuario para controlar el modo en que el usuario se registra en la aplicación y de forma que le permita usar su cuenta profesional, cuenta educativa o cualquier identidad social (como Google o Facebook) que quiera usar.  Para obtener más información, consulte la [Documentación de Azure AD B2B](index.yml).

- **Desarrolle aplicaciones destinadas a otros inquilinos de Azure AD (inquilino único o multiinquilino)** . Al desarrollar aplicaciones para Azure AD, puede dirigirse a usuarios de una única organización (inquilino único) o usuarios de cualquier organización que ya tenga un inquilino de Azure AD (denominadas aplicaciones multiinquilino). Estas aplicaciones multiinquilino las registra el usuario una vez en su propia instancia de Azure AD, pero pueden usarlas cualquier usuario de Azure AD de cualquier organización sin que sea necesario hacer nada más.

- **Desarrolle aplicaciones de marca blanca para consumidores y clientes (Azure AD B2C)** . Si es un negocio o un desarrollador que crea aplicaciones usadas por el cliente, puede escalar a los consumidores, clientes o ciudadanos mediante Azure AD B2C. Los desarrolladores pueden usar Azure AD como el sistema de identidad completo para su aplicación, permitiendo a los clientes que inicien sesión con una identidad que ya han establecido (como Facebook o Gmail). Con Azure AD B2C, puede personalizar y controlar el modo en que los clientes se suscriben, inician sesión y administran sus perfiles al usar las aplicaciones. Para obtener más información, consulte la [Documentación de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/).

## <a name="compare-external-identities-solutions"></a>Comparación de las soluciones de External Identities

En la tabla siguiente se proporciona una comparación detallada de los diferentes escenarios que se pueden habilitar con Azure AD External Identities.

| Aplicaciones multiinquilino  | Colaboración de usuario externo (B2B) | Aplicaciones para consumidores o clientes (B2C)  |
| ---- | --- | --- |
| Escenario principal: Software como servicio(SaaS) de empresa | Escenario principal: colaboración con aplicaciones de Microsoft (Microsoft 365, Teams...) o su propio software de colaboración.  | Escenario principal: aplicaciones transaccionales mediante aplicaciones de desarrollo personalizado.   |
| Destinado a: organizaciones que quieren proporcionar software a muchos clientes empresariales.    | Destinado a: organizaciones que desean poder autenticar a los usuarios desde una organización asociada, independientemente del proveedor de identidades.    | Destinado a: invitar a los clientes de sus aplicaciones web y para dispositivos móviles, independientemente de que sean personas, instituciones u organizaciones, a un directorio de Azure AD independiente del directorio de su propia organización. |
| Identidades que se admiten: empleados con cuentas de Azure AD. | Identidades que se admiten: empleados con cuentas profesionales o educativas, asociados con cuentas profesionales o educativas, o cualquier dirección de correo electrónico. Pronto se admitirá la federación directa.      | Identidades que se admiten: usuarios consumidores con cuentas de aplicación local (cualquier dirección de correo electrónico o nombre de usuario) o cualquier identidad social compatible con federación directa.       |
| Los usuarios externos se administran en su propio directorio, aislados del directorio en el que se registró la aplicación.    | Los usuarios externos se administran en el mismo directorio que los empleados, pero tienen una anotación especial. Pueden administrarse del mismo modo que los empleados, pueden agregarse a los mismos grupos, etc.    | Los usuarios externos se administran en el directorio de la aplicación. Se administran de manera independiente del directorio de asociados y de empleados de la organización (si existe).  |
| Inicio de sesión único: se admite SSO en todas las aplicaciones conectadas a Azure AD.          | Inicio de sesión único: se admite SSO en todas las aplicaciones conectadas a Azure AD. Por ejemplo, puede proporcionar acceso a Microsoft 365, o bien a aplicaciones locales y a otras aplicaciones SaaS como Salesforce o Workday.    | Inicio de sesión único: Se admite el inicio de sesión único para aplicaciones propiedad de los clientes dentro de los inquilinos de Azure AD B2C. No se admite el inicio de sesión único en Microsoft 365 ni en otras aplicaciones SaaS de Microsoft.    |
| Ciclo de vida de cliente: administrado por la organización principal del usuario.      | Ciclo de vida de asociado: lo administra la organización anfitriona o que realiza la invitación.    | Ciclo de vida de cliente: mediante autoservicio o administrado por la aplicación.      |
| Directiva de seguridad y cumplimiento: los administra la organización anfitriona o que realiza la invitación (por ejemplo, con [directivas de acceso condicional](https://docs.microsoft.com/azure/active-directory/b2b/conditional-access)).           | Directiva de seguridad y cumplimiento: los administra la organización anfitriona o que realiza la invitación (por ejemplo, con [directivas de acceso condicional](https://docs.microsoft.com/azure/active-directory/b2b/conditional-access)). | Directiva de seguridad y cumplimiento: las administra la aplicación.        |
| Personalización de marca: se utiliza la marca de la organización anfitriona o que realiza la invitación.   | Personalización de marca: se utiliza la marca de la organización anfitriona o que realiza la invitación.    | Personalización de marca: la administra la aplicación. Normalmente suele tratarse de productos con marca, y la organización pasa a un segundo plano.   |
| Más información: [Administración de identidades en aplicaciones multiinquilino](https://docs.microsoft.com/azure/architecture/multitenant-identity/), [Guía de procedimientos](https://docs.microsoft.com/azure/active-directory/develop/howto-convert-app-to-be-multi-tenant) | Más información: [Entrada de blog](https://blogs.technet.microsoft.com/enterprisemobility/2017/02/01/azure-ad-b2b-new-updates-make-cross-business-collab-easy/), [Documentación](what-is-b2b.md)                   | Más información: [página de producto](https://azure.microsoft.com/services/active-directory-b2c/), [documentación](https://docs.microsoft.com/azure/active-directory-b2c/)       |

Proteja y administre los clientes y asociados más allá de los límites organizacionales con Azure AD External Identities.

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [Acerca de Azure AD B2C](https://docs.microsoft.com/azure/active-directory-b2c/overview)
