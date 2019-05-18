---
title: Introducción a la integración de Azure AD con aplicaciones | Microsoft Docs
description: Este artículo es una guía de introducción a la integración de Azure Active Directory (AD) con aplicaciones locales y de nube.
services: active-directory
documentationcenter: ''
author: msmimart
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/16/2018
ms.author: mimart
ms.reviewer: asteen
ms.collection: M365-identity-device-management
ms.openlocfilehash: b8db7ac63aaf9ae5b1b16bb233e87ace06867309
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65824314"
---
# <a name="integrating-azure-active-directory-with-applications-getting-started-guide"></a>Guía de introducción a la integración de Azure Active Directory con las aplicaciones

En este tema se resume el proceso de integración de las aplicaciones con Azure Active Directory (AD). Cada una de las secciones siguientes contiene un breve resumen de un tema más detallado para que pueda identificar qué partes de esta guía de introducción son importantes para usted.

Para descargar los planes de implementación detallada, vea [Pasos siguientes](#next-steps).

## <a name="take-inventory"></a>Realización del inventario
Antes de integrar las aplicaciones con Azure AD, es importante saber dónde se encuentra y adónde quiere ir.  Las siguientes preguntas están diseñadas para ayudarle a pensar en su proyecto de integración de aplicaciones de Azure AD.

### <a name="application-inventory"></a>Inventario de aplicaciones
* ¿Dónde están todas las aplicaciones? ¿De quiénes son?
* ¿Qué tipo de autenticación necesitan las aplicaciones?
* ¿Quién necesita acceso a qué aplicaciones?
* ¿Quiere implementar una nueva aplicación?
  * ¿La integrará de forma interna y la implementará en una instancia de proceso de Azure?
  * ¿Usará una que esté disponible en la Galería de aplicaciones de Azure?

### <a name="user-and-group-inventory"></a>Inventario de usuarios y grupos
* ¿Dónde residen las cuentas de usuario?
  * Active Directory local
  * Azure AD
  * Dentro de una base de datos de aplicaciones independiente de su propiedad
  * En aplicaciones no sancionadas
  * Todo lo anterior
* ¿Qué permisos y asignaciones de rol tienen actualmente los usuarios individuales? ¿Debe revisar su acceso o está seguro de que las asignaciones de acceso y rol de usuario son apropiadas ahora?
* ¿Ya están los grupos establecidos en su Active Directory local?
  * ¿Cómo están organizados los grupos?
  * ¿Quiénes son los miembros del grupo?
  * ¿Qué asignaciones de permisos y roles tienen actualmente los grupos?
* ¿Deberá limpiar las bases de datos de usuarios o grupos antes de la integración?  (Esta es una pregunta bastante importante. Entrada y salida de elementos no utilizados).

### <a name="access-management-inventory"></a>Inventario de administración de acceso
* ¿Cómo administra actualmente el acceso de los usuarios a las aplicaciones? ¿Es necesario cambiarlo?  ¿Ha pensado en otras formas de administrar el acceso, como con [RBAC](../../role-based-access-control/role-assignments-portal.md) por ejemplo?
* ¿Quién necesita acceso a qué?

Quizás no tenga respuestas a todas estas preguntas por adelantado, pero no importa.  Esta guía puede ayudarle a responder a algunas de esas preguntas y a tomar algunas decisiones informadas.

### <a name="find-unsanctioned-cloud-applications-with-cloud-discovery"></a>Búsqueda de aplicaciones en la nube no autorizadas con Cloud Discovery

Como se mencionó anteriormente, puede haber aplicaciones que no han sido administradas por su organización hasta ahora.  Como parte del proceso de inventario, es posible encontrar aplicaciones de nube no sancionadas. Consulte [Configuración de Cloud Discovery](/cloud-app-security/set-up-cloud-discovery).

## <a name="integrating-applications-with-azure-ad"></a>Integración de aplicaciones con Azure AD
Los siguientes artículos describen las distintas formas en que las aplicaciones se integran con Azure AD, y ofrecen alguna orientación.

* [Determinación de qué Active Directory usar](../fundamentals/active-directory-administer.md)
* [Uso de aplicaciones de la Galería de aplicaciones de Azure](what-is-single-sign-on.md)
* [Lista de tutoriales sobre integración de aplicaciones SaaS](../active-directory-saas-tutorial-list.md)

### <a name="authentication-types"></a>Tipos de autenticación
Cada una de las aplicaciones puede tener requisitos de autenticación diferentes. Con Azure AD, pueden usar certificados de firma con aplicaciones que usan SAML 2.0, WS-Federation o protocolos de conexión OpenID, así como inicio de sesión único con contraseña. Para más información sobre los tipos de autenticación de aplicaciones para su uso con Azure AD, consulte [Administración de certificados para federada el inicio de sesión único federado en Azure Active Directory](manage-certificates-for-federated-single-sign-on.md) e [Inicio de sesión único basado en contraseña](what-is-single-sign-on.md).

### <a name="enabling-sso-with-azure-ad-app-proxy"></a>Habilitación de SSO con el proxy de la aplicación de Azure AD
Con el proxy de la aplicación de Microsoft Azure Active Directory, puede proporcionar acceso seguro a las aplicaciones que se encuentran dentro de la red privada, desde cualquier parte y en cualquier dispositivo. Cuando haya instalado un conector de proxy de aplicación dentro de su entorno, se podrá configurar con facilidad con Azure AD.

### <a name="integrating-custom-applications"></a>Integración de aplicaciones personalizadas
Si va a crear una nueva aplicación y quiere ayudar a los desarrolladores a aprovechar la eficacia de Azure AD, consulte [Directrices para los desarrolladores](../active-directory-applications-guiding-developers-for-lob-applications.md).

Si quiere agregar su aplicación personalizada a la Galería de aplicaciones de Azure, consulte ["Traiga su propia aplicación" con la configuración de SALM de autoservicio de Azure AD](https://cloudblogs.microsoft.com/enterprisemobility/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-now-in-preview/).

## <a name="managing-access-to-applications"></a>Administración del acceso a las aplicaciones
En los artículos siguientes se describen formas de administrar el acceso a las aplicaciones después de que se han integrado con Azure AD mediante conectores de Azure AD y Azure AD.

* [Administración del acceso a aplicaciones con Azure AD](what-is-access-management.md)
* [Automatización con conectores de Azure AD](user-provisioning.md)
* [Asignación de usuarios a una aplicación](../active-directory-applications-guiding-developers-assigning-users.md)
* [Asignación de grupos a una aplicación](../active-directory-applications-guiding-developers-assigning-groups.md)
* [Uso compartido de cuentas](../active-directory-sharing-accounts.md)

## <a name="next-steps"></a>Pasos siguientes
Para obtener información detallada, puede descargar los planes de implementación de Azure Active Directory desde [GitHub](https://aka.ms/deploymentplans). Para las aplicaciones de galería, puede descargar los planes de implementación para el inicio de sesión único, acceso condicional y aprovisionamiento de usuarios a través de [Azure Portal](https://portal.azure.com). 

Para descargar un plan de implementación desde Azure Portal:

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Seleccione **Aplicaciones empresariales** | **Elegir una aplicación** | **Plan de implementación**.

Realice la [Encuesta del plan de implementación](https://aka.ms/DeploymentPlanFeedback) para proporcionar comentarios sobre los planes de implementación.
