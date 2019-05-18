---
title: ¿Qué es Azure Active Directory? Azure Active Directory | Microsoft Docs
description: Información general y conceptual acerca de Azure Active Directory, lo que incluye la terminología, qué licencias están disponibles y una lista de las características asociadas con vínculos para obtener más información.
services: active-directory
author: eross-msft
manager: daveba
ms.service: active-directory
ms.topic: overview
ms.date: 05/08/2019
ms.author: lizross
ms.custom: it-pro, seodec18, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: ce8fad7f0dc76aad306e0f2a8e26692ec997952c
ms.sourcegitcommit: 399db0671f58c879c1a729230254f12bc4ebff59
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65470355"
---
# <a name="what-is-azure-active-directory"></a>¿Qué es Azure Active Directory?

Azure Active Directory (Azure AD) es un servicio de administración de acceso y de identidades basado en la nube de Microsoft que ayuda a los recursos de acceso y de inicio de sesión de los empleados en:

- Recursos externos, como Microsoft Office 365, Azure Portal y miles de otras aplicaciones SaaS.

- Recursos internos, como las aplicaciones de la red corporativa y la intranet, junto con todas las aplicaciones en la nube desarrolladas por su propia organización.

Puede utilizar los diversos pósteres de la [Serie de Microsoft Cloud para arquitectos empresariales](https://docs.microsoft.com/office365/enterprise/microsoft-cloud-it-architecture-resources#identity) para conocer mejor los servicios de identidad principales de Azure, Azure AD y Office 365.

## <a name="who-uses-azure-ad"></a>¿Quién usa Azure AD?

Azure AD va dirigido a:

- **Administradores de TI.** Si es administrador de TI, puede usar Azure AD para controlar el acceso a sus aplicaciones y a los recursos de estás en función de los requisitos de su empresa. Por ejemplo, puede usar Azure AD para requerir autenticación multifactor al acceder a recursos importantes de la organización. Además, puede usar Azure AD para automatizar el aprovisionamiento de usuarios entre la instancia existente de Windows Server AD y las aplicaciones en la nube, incluida Office 365. Por último, Azure AD proporciona eficaces herramientas que ayudan a proteger automáticamente las identidades y credenciales de los usuarios y a cumplir los requisitos de gobernanza de acceso. Para empezar, regístrese para obtener una [versión de evaluación gratuita durante 30 días de Azure Active Directory Premium](https://azure.microsoft.com/trial/get-started-active-directory/).

- **Desarrolladores de aplicaciones.** A los desarrolladores de aplicaciones, Azure AD les ofrece un enfoque basado en estándares para agregar el inicio de sesión único (SSO) a cualquier aplicación, lo que le permite trabajar con las credenciales existentes de un usuario. Azure AD también proporciona varias API que pueden ayudarle a crear experiencias de aplicación personalizadas que usen los datos existentes de la organización. Para empezar, regístrese para obtener una [versión de evaluación gratuita durante 30 días de Azure Active Directory Premium](https://azure.microsoft.com/trial/get-started-active-directory/). Para más información, también puede consultar [Azure Active Directory para desarrolladores](../develop/index.yml).

- **Suscriptores de Microsoft 365, Office 365, Azure o Dynamics CRM Online.** Los suscriptores usan Azure AD. Cada inquilino de Microsoft 365, Office 365, Azure y Dynamics CRM Online es automáticamente un inquilino de Azure AD. Puede empezar de inmediato a administrar el acceso a las aplicaciones en la nube integradas.

## <a name="what-are-the-azure-ad-licenses"></a>¿Qué son las licencias de Azure AD?

Los servicios para la empresa de Microsoft Online, como Office 365 o Microsoft Azure, requieren Azure AD para iniciar sesión y ayudar con la protección de identidad. Por consiguiente, si se suscribe a cualquiera de los servicios de negocio de Microsoft Online, obtendré automáticamente Azure AD con acceso a todas las características gratuitas.

Para mejorar la implementación de Azure AD, también puede agregar funcionalidades de pago mediante la actualización a las licencias Azure Active Directory Basic, Premium P1 o Premium P2. Las licencias de pago de Azure AD se crean sobre el directorio gratuito existente y proporcionan autoservicio, supervisión mejorada, informes de seguridad y un acceso seguro a sus usuarios móviles.

>[!Note]
>Para ver las opciones de precios de estas licencias, consulte [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).
>
>Azure Active Directory Premium P1, Premium P2 y Azure Active Directory Basic no se admiten actualmente en China. Para más información acerca de los precios de Azure AD, póngase en contacto con el [foro de Azure Active Directory](https://azure.microsoft.com/support/community/?product=active-directory).

- **Azure Active Directory Free.** Proporciona administración de grupos y usuarios, sincronización de directorios locales, informes básicos e inicio de sesión único en Azure, Office 365 y muchas aplicaciones SaaS populares.

- **Azure Active Directory Basic.** Además de las características de la licencia Free, Basic proporciona acceso a aplicaciones centrado en la nube, administración de acceso basado en grupos, restablecimiento de contraseña de autoservicio para aplicaciones en la nube y Azure AD Application Proxy, que permite publicar aplicaciones locales mediante Azure AD.

- **Azure Active Directory Premium P1.** Además de las características de las licencias Free y Basic, P1 también permite a los usuarios de entornos híbridos acceder a recursos locales y en la nube. También admite la administración avanzada, como grupos dinámicos, administración de grupos de autoservicio, Microsoft Identity Manager (un conjunto de administración local de identidades y acceso) y funcionalidades de reescritura en la nube, que permiten el restablecimiento de contraseña de autoservicio a los usuarios locales.

- **Azure Active Directory Premium P2.** Además de las características de las licencias Free, Basic y P1, la licencia P2 ofrece también [Azure Active Directory Identity Protection](../identity-protection/enable.md), que ayuda a proporcionar acceso condicional basado en riesgos a sus aplicaciones y datos críticos de la empresa, y [Privileged Identity Management](../privileged-identity-management/pim-getting-started.md), que ayuda a detectar, restringir y supervisar los administradores y su acceso a los recursos, y a proporcionar acceso Just-In-Time cuando sea necesario.

- **Licencias de la característica de "Pago por uso".** También puede obtener licencias de características adicionales, como Azure Active Directory Business-to-Customer (B2C). B2C puede ayudarle a proporcionar soluciones de administración de acceso y de identidad para las aplicaciones orientadas al cliente. Para más información, consulte [Documentación de Azure Active Directory B2C](../../active-directory-b2c/index.yml).

Para obtener más información sobre cómo asociar una suscripción de Azure a Azure AD, consulte [Cómo: Asociación o adición de una suscripción de Azure a Azure Active Directory](active-directory-how-subscriptions-associated-directory.md), y para obtener más información sobre cómo asignar licencias a los usuarios, consulte [Cómo asignar o quitar licencias de Azure Active Directory](license-users-groups.md).

## <a name="terminology"></a>Terminología

Para conocer mejor Azure AD y su documentación, es aconsejable revisar los términos siguientes.

|Término o concepto|DESCRIPCIÓN|
|---------------|-----------|
|Suscripción de Azure| Se usa para pagar los servicio en la nube de Azure. Puede tener muchas suscripciones y están vinculadas a una tarjeta de crédito.|
|Inquilino de Azure| Es una instancia dedicada y de confianza de Azure AD que se crea automáticamente cuando una organización se suscribe a un servicio en la nube de Microsoft, como Microsoft Azure, Microsoft Intune u Office 365. Un inquilino de Azure representa una organización individual.|
|Un solo inquilino| Los inquilinos de Azure que acceden a otros servicios en un entorno dedicado se consideran inquilino individuales.|
|Multiinquilino| Los inquilinos de Azure que tienen acceso a otros servicios en un entorno compartido, en varias organizaciones, se consideran varios inquilinos.|
|Directorio de Azure AD|Todos los inquilinos de Azure tienen un directorio de Azure AD dedicado y de confianza. El directorio de Azure AD incluye los usuarios, grupos y aplicaciones del inquilino y se usa para realizar funciones de administración de acceso y de identidad en los recursos del inquilino.|
|Cuenta de Azure AD | Una identidad que se crean mediante Azure AD u otro servicio en la nube de Microsoft, como Office 365. Las identidades se almacenan en Azure AD y pueden acceder a ellas las suscripciones de servicio en la nube de su organización. Esta cuenta se denomina a veces también cuenta profesional o educativa.|
|Dominio personalizado|Cada directorio nuevo de Azure AD incluye un nombre de dominio inicial, nombre_de_dominio.onmicrosoft.com. Además de dicho nombre inicial, también puede agregar nombres de dominio de la organización, que incluyen los nombres que utiliza para hacer negocios y los que utilizan los usuarios para acceder a los recursos de su organización, a la lista. La adición de nombres de dominio personalizados le ayuda a crear nombres de usuario que resultan familiares a los usuarios, como alain@contoso.com.|
|Administrador de cuenta|Este rol de administrador de suscripción clásica conceptualmente es el propietario de facturación de una suscripción. Este rol tiene acceso al [centro de cuentas de Azure](https://account.azure.com/Subscriptions) y permite administrar todas las suscripciones de una cuenta. Para más información, consulte [Roles de administrador de suscripciones clásico de RBAC de Azure y de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).|
|Administrador de servicios|Este rol de administrador de suscripciones clásico permite administrar todos los recursos de Azure, incluido el acceso. Tiene el acceso equivalente a un usuario al que se le ha asignado la función de propietario en el ámbito de la suscripción. Para más información, consulte [Roles de administrador de suscripciones clásico, de RBAC de Azure y de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).|
|Propietario|Este rol ayuda a administrar todos los recursos de Azure, incluido el acceso. Este rol se basa en un sistema de autorización más reciente denominado control de acceso basado en rol (RBAC) que proporciona una administración detallada del acceso a recursos de Azure. Para más información, consulte [Roles de administrador de suscripciones clásico, de RBAC de Azure y de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).|
|Administrador global de Azure AD|Este rol de administrador se asigna automáticamente a quien haya creado el inquilino de Azure AD. Los administradores globales pueden realizar todas las funciones administrativas de Azure AD y los servicios que se federan con Azure AD, como Exchange Online, SharePoint Online y Skype Empresarial Online. Puede tener varios administradores globales, pero estos administradores son los únicos que pueden asignar roles de administrador (lo que incluye la asignación de otros administradores globales) a los usuarios.<br><br>**Nota:**<br>Este rol de administrador se denomina Administrador global en Azure Portal, pero se denomina  **Administrador de la compañía** en Microsoft Graph API, Azure AD Graph API y Azure AD PowerShell.<br><br>Para más información acerca de los distintos roles de administrador, consulte [Permisos de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md).|
|Cuenta Microsoft (también denominada MSA)|Las cuentas personales que proporcionan acceso a los productos y servicios en la nube de Microsoft orientados al consumidor, como Outlook, OneDrive, Xbox LIVE u Office 365. Su cuenta Microsoft se crean y almacenan en el sistema de cuentas de identidad de consumidor de Microsoft que ejecuta Microsoft.|

## <a name="which-features-work-in-azure-ad"></a>¿Qué características funcionan en Azure AD?

Después de elegir su licencia de Azure AD, obtendrá acceso a algunas de las características siguientes para su organización, o a todas ellas:

|Categoría|DESCRIPCIÓN|
|-------|-----------|
|Administración de aplicaciones|Administre las aplicaciones en la nube y locales mediante el proxy de aplicación, el inicio de sesión único, el portal Mis aplicaciones (también conocido como Panel de acceso) y aplicaciones de software como servicio (SaaS). Para más información, consulte [Provisión de acceso remoto seguro a aplicaciones locales](../manage-apps/application-proxy.md) y [Documentación sobre la administración de aplicaciones](../manage-apps/index.yml).|
|Authentication|Administre el restablecimiento de contraseñas de autoservicio de Azure Active Directory, Multi-Factor Authentication, la lista personalizada de contraseñas prohibidas y el bloqueo inteligente. Para más información, consulte la [documentación de Autenticación de Azure AD](../authentication/index.yml).|
|Negocio a negocio (B2B)|Administre los usuarios invitados y los asociados externos sin perder el control sobre sus propios datos corporativos. Para más información, consulte [Documentación de Azure Active Directory B2B](../b2b/index.yml).|
|Negocio a cliente (B2C)|Personalice y controle la forma en que los usuarios se suscriben, inician sesión y administran sus perfiles al usar sus aplicaciones. Para más información, consulte [Documentación de Azure Active Directory B2C](../../active-directory-b2c/index.yml).|
|Acceso condicional|Administre el acceso a sus aplicaciones en la nube. Para más información, consulte [Documentación sobre el acceso condicional de Azure AD](../conditional-access/index.yml).|
|Azure Active Directory para desarrolladores|Cree aplicaciones que inicie sesión en todas las identidades de Microsoft, obtenga tokens para llamar a Microsoft Graph, otras API de Microsoft API o API personalizadas. Para más información, consulte [Plataforma de identidad de Microsoft (Azure Active Directory para desarrolladores)](../develop/index.yml).|
|Administración de dispositivos|Administre la forma en que los dispositivos en la nube o locales acceden a los datos corporativos. Para más información, consulte la [documentación de Azure AD Device Management](../devices/index.yml).|
|Servicios de dominio|Combine máquinas virtuales de Azure en un dominio sin controladores de dominio. Para más información, consulte [Documentación de Azure AD Domain Services](../../active-directory-domain-services/index.yml).|
|Usuarios de empresa|Administre la asignación de licencias, el acceso a las aplicaciones y configure delegados mediante grupos y roles de administrador. Para más información, consulte [Documentación sobre administración de usuarios de Azure Active Directory](../users-groups-roles/index.yml).|
|Identidad híbrida|Use Azure Active Directory Connect y Connect Health para proporcionar una identidad de usuario individual para la autenticación y autorización en todos los recursos, independientemente de la ubicación (nube o local). Para más información, consulte [Documentación de identidad híbrida](../hybrid/index.yml).|
|Gobernanza de identidades|Administre la identidad de su organización a través de controles de acceso a empleados, asociados comerciales, proveedores, servicios y aplicaciones. También puede realizar revisiones de acceso. Para más información, consulte la [documentación sobre la gobernanza de identidades en Azure AD](../governance/identity-governance-overview.md) y las [revisiones de acceso de Azure AD](../governance/access-reviews-overview.md).|
|Protección de identidad|Detecte posibles puntos vulnerables que afecten a identidades de su organización, configure directivas para responder a acciones sospechosas y, después, realice las acciones adecuadas para resolverlas. Para más información, consulte [Azure AD Identity Protection](../identity-protection/index.yml).|
|Identidades administradas de recursos de Azure|Proporcione a los servicios de Azure una identidad administrada automáticamente en Azure AD que puede autenticar cualquier servicio de autenticación compatible con Azure AD, incluido Key Vault. Para más información, consulte [¿Qué es Managed Identities for Azure Resources?](../managed-identities-azure-resources/overview.md)|
|Privileged Identity Management (PIM)|Administre, controle y supervise el acceso dentro de su organización. Esta característica incluye el acceso a los recursos de Azure AD y Azure, así como a otros servicios de Microsoft Online Services, como Office 365 o Intune. Para obtener más información, vea [Azure AD Privileged Identity Management](../privileged-identity-management/index.yml).|
|Informes y supervisión|Obtenga información acerca de los patrones de seguridad y de uso del entorno. Para más información, consulte [Informes y supervisión de Azure Active Directory](../reports-monitoring/index.yml).|

## <a name="next-steps"></a>Pasos siguientes

- [Suscripción a Azure Active Directory Premium](active-directory-get-started-premium.md)

- [Asociación o adición de una suscripción de Azure a Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)

- [Acceso a Azure Active Directory para crear un inquilino](active-directory-access-create-new-tenant.md)

- [Lista de comprobación de la característica de licencia de Azure Active Directory Premium P2](active-directory-deployment-checklist-p2.md)
