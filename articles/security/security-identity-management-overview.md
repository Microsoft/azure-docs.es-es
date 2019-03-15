---
title: Características de seguridad de Azure que ayudan a la administración de identidades | Microsoft Docs
description: " Este artículo proporciona una visión general de las principales características de seguridad de Azure que contribuyen a la administración de identidades. Las soluciones de administración de identidades y acceso de Microsoft ayudan al departamento de TI a proteger el acceso a las aplicaciones y los recursos en el centro de datos corporativo y en la nube, para lo que se habilitan más niveles de validación, tales como Multi-Factor Authentication y directivas de acceso condicional. "
services: security
documentationcenter: na
author: TerryLanfear
manager: barbkess
editor: TomSh
ms.assetid: 5aa0a7ac-8f18-4ede-92a1-ae0dfe585e28
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/19/2018
ms.author: terrylan
Customer intent: As a IT Pro or decision maker I am trying to learn about identity management capabilities in Azure
ms.openlocfilehash: b58cbcdccced3952c3171daa14f80986e473c5dd
ms.sourcegitcommit: fec0e51a3af74b428d5cc23b6d0835ed0ac1e4d8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/12/2019
ms.locfileid: "56111499"
---
# <a name="azure-identity-management-security-overview"></a>Información general sobre seguridad de administración de identidades de Azure

 La administración de identidades es el proceso de autenticación y autorización de las [entidades de seguridad](https://docs.microsoft.com/windows/security/identity-protection/access-control/security-principals). También implica controlar información acerca de esas entidades de seguridad (identidades). Las entidades de seguridad (identidades) pueden incluir servicios, aplicaciones, usuarios, grupos, etc. Las soluciones de administración de identidades y acceso de Microsoft ayudan al departamento de TI a proteger el acceso a las aplicaciones y los recursos en el centro de datos corporativo y en la nube. Esta protección permite niveles adicionales de validación, como Multi-Factor Authentication y directivas de acceso condicional. La supervisión de actividades sospechosas mediante auditorías, alertas e informes de seguridad avanzados contribuye a minimizar los posibles problemas de seguridad. [Azure Active Directory Premium](../active-directory/active-directory-editions.md) ofrece un inicio de sesión único (SSO) para miles de aplicaciones de software como servicio (SaaS) en la nube y acceso a aplicaciones web que se ejecutan de forma local.
 
Si aprovecha las ventajas en materia de seguridad de Azure Active Directory (Azure AD), podrá:

* Crear y administrar una identidad única para cada usuario en toda la empresa híbrida, lo que mantiene los usuarios, grupos y dispositivos sincronizados. 
* Proporcionar acceso de SSO a las aplicaciones, incluidas miles de aplicaciones SaaS preintegradas.
* Habilitar la seguridad del acceso de las aplicaciones mediante la aplicación de Multi-Factor Authentication basado en reglas para las aplicaciones locales y en la nube.
* Proporcionar un acceso remoto seguro a las aplicaciones web locales a través del proxy de la aplicación de Azure AD.

El objetivo de este artículo es proporcionar una visión general de las principales características de seguridad de Azure que contribuyen a la administración de identidades. Además, se incluyen vínculos a artículos que ofrecen detalles de cada una de estas características para que pueda tener más información al respecto.  

El artículo se centra en las siguientes funcionalidades de administración de identidades de Azure principales:

* Inicio de sesión único
* Proxy inverso
* Multi-Factor Authentication
* Control de acceso basado en roles (RBAC)
* Supervisión de seguridad, alertas e informes basados en aprendizaje automático
* Administración de identidades y acceso de consumidores
* Registro de dispositivos
* Privileged Identity Management
* Protección de identidad
* Administración de identidades híbridas/Azure AD Connect
* Revisiones de acceso de Azure AD

## <a name="single-sign-on"></a>Inicio de sesión único

SSO significa tener acceso a todas las aplicaciones y los recursos que necesita para hacer negocios, al iniciar sesión una sola vez con una única cuenta de usuario. Una vez que ha iniciado sesión, puede tener acceso a todas las aplicaciones que necesite sin tener que autenticarse (por ejemplo, escribiendo una contraseña) una segunda vez.

Muchas organizaciones se basan en aplicaciones SaaS tales como Office 365, Box y Salesforce para la productividad del usuario. Tradicionalmente, el personal de TI tenía que crear y actualizar individualmente cuentas de usuario en cada aplicación SaaS y los usuarios tenían que recordar una contraseña para cada aplicación SaaS.

Azure AD extiende los entornos de Active Directory locales a la nube, lo que permite a los usuarios usar su cuenta de organización principal para iniciar sesión no solo en sus dispositivos unidos a dominios y recursos de la empresa, sino también en todas las aplicaciones web y SaaS necesarias para su trabajo.

Los usuarios no solo no tienen que administrar varios conjuntos de nombres de usuario y contraseñas, sino que se puede aprovisionar o desaprovisionar el acceso a las aplicaciones automáticamente en función de los grupos de la organización y de su estado de empleado. Azure AD introduce controles de gobierno de acceso y seguridad con los que puede gestionar de forma centralizada el acceso de los usuarios a través de aplicaciones SaaS.

Más información:

* [Información general de inicio de sesión único](https://azure.microsoft.com/documentation/videos/overview-of-single-sign-on/)
* [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../active-directory/manage-apps/what-is-single-sign-on.md)
* [Integración del inicio de sesión único de Azure Active Directory con aplicaciones SaaS](../active-directory/manage-apps/configure-single-sign-on-portal.md)

## <a name="reverse-proxy"></a>Proxy inverso

El proxy de aplicación de Azure AD permite publicar aplicaciones locales (como sitios de [SharePoint](https://support.office.com/article/What-is-SharePoint-97b915e6-651b-43b2-827d-fb25777f446f?ui=en-US&rs=en-US&ad=US), [Outlook Web App](https://technet.microsoft.com/library/jj657718.aspx) y aplicaciones basadas en [IIS](http://www.iis.net/)) en la red privada y proporciona un acceso seguro a los usuarios externos a la red. El proxy de aplicación ofrece acceso remoto y de SSO para muchos tipos de aplicaciones web locales, junto con las miles de aplicaciones SaaS que Azure AD admite. Los empleados pueden iniciar sesión en sus aplicaciones desde casa, en sus propios dispositivos, y autenticarse a través de este proxy basado en la nube.

Más información:

* [Habilitación del proxy de la aplicación de Azure AD](../active-directory/manage-apps/application-proxy-enable.md)
* [Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](../active-directory/active-directory-application-proxy-publish.md)
* [Inicio de sesión único con el proxy de aplicación](../active-directory/manage-apps/application-proxy-configure-single-sign-on-with-kcd.md)
* [Uso de acceso condicional](../active-directory/manage-apps/application-proxy-integrate-with-sharepoint-server.md)

## <a name="multi-factor-authentication"></a>Multi-Factor Authentication

Azure Multi-factor Authentication es un método de autenticación que requiere el uso de más de un método de verificación y agrega un segundo nivel de seguridad crítico a las transacciones y los inicios de sesión del usuario. Multi-Factor Authentication le ayudará a proteger el acceso a los datos y las aplicaciones, además de satisfacer la demanda de los usuarios de un proceso de inicio de sesión simple. Proporciona autenticación sólida mediante diversas opciones de verificación: llamadas telefónicas, mensajes de texto, notificaciones de aplicaciones móviles, códigos de verificación y tokens OAuth de terceros.

Más información:

* [Multi-Factor Authentication](https://azure.microsoft.com/documentation/services/multi-factor-authentication/)
* [¿Qué es Azure Multi-Factor Authentication?](../active-directory/authentication/multi-factor-authentication.md)
* [Cómo funciona Azure Multi-Factor Authentication](../active-directory/authentication/concept-mfa-howitworks.md)

## <a name="rbac"></a>RBAC

RBAC es un sistema de autorización basado en Azure Resource Manager que proporciona administración de acceso específico a los recursos de Azure. RBAC permite controlar el nivel de acceso que tienen los usuarios de forma granular. Por ejemplo, puede limitar a un usuario para que solo administre redes virtuales y otro usuario para que administre todos los recursos de un grupo de recursos. Azure incluye varios roles integrados que puede usar. A continuación se enumeran cuatros roles integrados fundamentales. Los tres primeros se aplican a todos los tipos de recursos.

Más información:

* [¿Qué es el control de acceso basado en rol (RBAC)?](../role-based-access-control/overview.md)
* [Roles integrados en los recursos de Azure](../role-based-access-control/built-in-roles.md)

## <a name="security-monitoring-alerts-and-machine-learning-based-reports"></a>Supervisión de seguridad, alertas e informes basados en aprendizaje automático

La supervisión de seguridad, las alertas y los informes basados en aprendizaje automático que identifican patrones de acceso incoherentes, pueden ayudarle a proteger su negocio. Puede usar los informes de acceso y uso de Azure AD para proporcionar visibilidad de la integridad y la seguridad del directorio de la organización. Con esta información, un administrador de directorios puede determinar mejor dónde puede haber posibles riesgos de seguridad de modo que pueda planear adecuadamente la mitigación de estos riesgos.

En Azure Portal, los informes se dividen en las siguientes categorías:

* **Informes de anomalías**: contienen eventos de inicio de sesión que se consideran anómalos. Nuestro objetivo es que sea consciente de dicha actividad y que pueda tomar una decisión sobre si un evento es sospechoso.
* **Informes de aplicaciones integradas**: proporciona información sobre cómo se usan en la organización las aplicaciones en la nube. Azure AD ofrece integración con miles de aplicaciones en la nube.
* **Informes de errores**: indican errores que se pueden producir al aprovisionar cuentas en aplicaciones externas.
* **Informes específicos del usuario**: muestran los datos de actividad de dispositivo o de inicio de sesión de un usuario concreto.
* **Registros de actividad**: contienen un registro de todos los eventos auditados en las últimas 24 horas, los últimos 7 días o los últimos 30 días, así como los cambios en la actividad del grupo y la actividad de registro y de restablecimiento de contraseña.

Más información:

* [Visualización de los informes de acceso y uso](../active-directory/active-directory-view-access-usage-reports.md)
* [Introducción a los informes de Azure Active Directory](../active-directory/active-directory-reporting-getting-started.md)
* [Guía de informes de Azure Active Directory](../active-directory/active-directory-reporting-guide.md)

## <a name="consumer-identity-and-access-management"></a>Administración de identidades y acceso de consumidores

Azure AD B2C es un servicio de administración de identidades global y de alta disponibilidad para aplicaciones orientadas al consumidor que se puede escalar hasta cientos de millones de identidades. Se puede integrar en plataformas móviles y web. Los consumidores pueden conectarse a todas sus aplicaciones con una experiencia totalmente personalizable, usando sus cuentas de las redes sociales o mediante credenciales nuevas.

En el pasado, los desarrolladores de aplicaciones que querían registrar clientes e iniciar sesión en sus aplicaciones tenían que escribir su propio código. Y usaban bases de datos o sistemas locales para almacenar nombres de usuario y contraseñas. Azure AD B2C ofrece a su organización una mejor manera de integrar la administración de identidades del consumidor en las aplicaciones con la ayuda de una plataforma segura basada en estándares y un amplio conjunto de políticas extensibles.

El uso de Azure AD B2C permite a los consumidores registrarse en las aplicaciones con sus cuentas de redes sociales existentes (Facebook, Google, Amazon, LinkedIn) o creando unas credenciales (dirección de correo electrónico y contraseña o nombre de usuario y contraseña).

Más información:

* [¿Qué es Azure Active Directory B2C?](https://azure.microsoft.com/services/active-directory-b2c/)
* [Azure Active Directory B2C (versión preliminar): Registro e inicio de sesión de los consumidores en las aplicaciones](../active-directory-b2c/active-directory-b2c-overview.md)
* [Azure Active Directory B2C (versión preliminar): tipos de aplicaciones](../active-directory-b2c/active-directory-b2c-apps.md)

## <a name="device-registration"></a>Registro de dispositivos

El Registro de dispositivos de Azure AD es la base de los escenarios de [acceso condicional](../active-directory/active-directory-conditional-access-device-registration-overview.md) basado en dispositivos. Cuando se registra un dispositivo, el Registro de dispositivos de Azure AD le proporciona una identidad que se utiliza para autenticar el dispositivo cuando el usuario inicia sesión. El dispositivo autenticado y los atributos del dispositivo pueden utilizarse para aplicar directivas de acceso condicional a las aplicaciones que se hospedan en la nube y en el entorno local.

Cuando se combina con una solución de administración de dispositivos móviles como Intune, los atributos del dispositivo en Azure AD se actualizan con información adicional sobre este. Luego puede crear reglas de acceso condicional que obligan a que el acceso desde dispositivos cumpla con las normas de seguridad y cumplimiento.

Más información:

* [Introducción al Registro de dispositivos de Azure Active Directory](../active-directory/active-directory-conditional-access-device-registration-overview.md)
* [Registro automático de dispositivos en Azure AD para dispositivos Windows unidos a un dominio](../active-directory/active-directory-conditional-access-automatic-device-registration.md)
* [Configuración del registro automático de dispositivos unidos a un dominio de Windows con Azure Active Directory](../active-directory/active-directory-conditional-access-automatic-device-registration-setup.md)

## <a name="privileged-identity-management"></a>Privileged Identity Management

Con Azure AD Privileged Identity Management, puede administrar, controlar y supervisar las identidades con privilegios y el acceso a los recursos en Azure AD y en otros servicios en línea de Microsoft, como Office 365 y Microsoft Intune.

En ocasiones, los usuarios tienen que realizar operaciones con privilegios en recursos de Azure u Office 365 o en otras aplicaciones SaaS. Esta necesidad suele acarrear que las organizaciones tienen que dar a los usuarios acceso con privilegios permanente en Azure AD. Este acceso es un riesgo de seguridad cada vez mayor para los recursos hospedados en la nube, ya que las organizaciones no pueden supervisar suficientemente lo que los usuarios hacen con sus privilegios de administrador. Además, si una cuenta de usuario con acceso privilegiado se ve comprometida, esa vulneración podría afectar a la seguridad global de la organización en la nube. Azure AD Privileged Identity Management le ayuda a mitigar este riesgo.

Con Azure AD Privileged Identity Management, podrá:

* Ver los usuarios que son administradores de Azure AD.
* Habilitar el acceso administrativo Just-In-Time (JIT) a petición a servicios de Microsoft tales como Office 365 e Intune.
* Obtener informes sobre el historial de acceso de administrador y los cambios en las asignaciones de administrador.
* Obtener alertas sobre el acceso a un rol con privilegios.

Más información:

* [¿Qué es Azure AD Privileged Identity Management?](../active-directory/privileged-identity-management/pim-configure.md)
* [Asignación de roles de directorio de Azure AD en PIM](../active-directory/privileged-identity-management/pim-how-to-add-role-to-user.md)

## <a name="identity-protection"></a>Protección de identidad

Azure AD Identity Protection es un servicio de seguridad que proporciona una vista consolidada de los eventos de riesgos y las vulnerabilidades potenciales que afectan a las identidades de su organización. Identity Protection aprovecha las funcionalidades de detección de anomalías de Azure AD existentes, que están disponibles a través de los informes de actividades anómalas de Azure AD. Identity Protection también incluye nuevos tipos de eventos de riesgo que pueden detectar anomalías en tiempo real.

Más información:

* [Azure AD Identity Protection](../active-directory/active-directory-identityprotection.md)
* [Channel 9: Azure AD and Identity Show: Identity Protection Preview](https://channel9.msdn.com/Series/Azure-AD-Identity/Azure-AD-and-Identity-Show-Identity-Protection-Preview)

## <a name="hybrid-identity-managementazure-ad-connect"></a>Administración de identidades híbridas/Azure AD Connect

Las soluciones de identidad de Microsoft abarcan funcionalidades locales y de nube, de forma que se crea una sola identidad de usuario para la autenticación y la autorización en todos los recursos, sin importar su ubicación. A esto le llamamos identidad híbrida. Azure AD Connect es la herramienta de Microsoft diseñada para satisfacer y lograr sus objetivos de identidad híbrida. Esto le permite proporcionar una identidad común para los usuarios de aplicaciones de Office 365, Azure y SaaS integradas con Azure AD. Ofrece las siguientes características:

* Sincronización
* Integración de federación y AD FS
* Autenticación de paso a través
* Supervisión del estado

Más información:

* [Notas del producto sobre identidad híbrida](https://download.microsoft.com/download/D/B/A/DBA9E313-B833-48EE-998A-240AA799A8AB/Hybrid_Identity_White_Paper.pdf)
* [Azure Active Directory](https://azure.microsoft.com/documentation/services/active-directory/)
* [Blog del equipo de Azure AD](https://blogs.technet.microsoft.com/ad/)

## <a name="azure-ad-access-reviews"></a>Revisiones de acceso de Azure AD

Las revisiones de acceso de Azure Active Directory (Azure AD) permiten a las organizaciones administrar de forma eficiente la pertenencia a grupos, el acceso a las aplicaciones empresariales y las asignaciones de roles con privilegios.

Más información:

* [Revisiones de acceso de Azure AD](../active-directory/governance/access-reviews-overview.md)
* [Administración del acceso de los usuarios con las revisiones de acceso de Azure AD](../active-directory/governance/access-reviews-overview.md)
