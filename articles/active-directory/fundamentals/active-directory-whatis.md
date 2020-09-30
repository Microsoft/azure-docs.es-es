---
title: ¿Qué es Azure Active Directory? Azure Active Directory | Microsoft Docs
description: Información general y conceptual acerca de Azure Active Directory, lo que incluye la terminología, qué licencias están disponibles y una lista de las características asociadas con vínculos para obtener más información.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.subservice: fundamentals
ms.topic: overview
ms.date: 06/05/2020
ms.author: ajburnle
ms.custom: it-pro, seodec18, seo-update-azuread-jan, contperfq4
ms.collection: M365-identity-device-management
ms.openlocfilehash: 1d66e8b15b7af73517a7f838186821248bffeccc
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89566116"
---
# <a name="what-is-azure-active-directory"></a>¿Qué es Azure Active Directory?

Azure Active Directory (Azure AD) es un servicio de administración de identidades y acceso basado en la nube de Microsoft que ayuda a los empleados a iniciar sesión y acceder a recursos en:

- Recursos externos, como Microsoft 365, Azure Portal y miles de otras aplicaciones SaaS.

- Recursos internos, como las aplicaciones de la red corporativa y la intranet, junto con todas las aplicaciones en la nube desarrolladas por su propia organización. Para más información sobre la creación de un inquilino para su organización, consulte [Guía de inicio rápido: Creación de un inquilino en Azure Active Directory](active-directory-access-create-new-tenant.md).

Para obtener información sobre la diferencia entre Azure AD y Active Directory Domain Services, consulte [Comparación de Active Directory y Azure Active Directory](active-directory-compare-azure-ad-to-ad.md). Puede utilizar los diversos pósteres de la [Serie de Microsoft Cloud para arquitectos empresariales](/microsoft-365/solutions/cloud-architecture-models?view=o365-worldwide) para conocer mejor los servicios de identidad principales de Azure, Azure AD y Microsoft 365.

## <a name="who-uses-azure-ad"></a>¿Quién usa Azure AD?

Azure AD va dirigido a:

- **Administradores de TI.** Si es administrador de TI, puede usar Azure AD para controlar el acceso a sus aplicaciones y a los recursos de estás en función de los requisitos de su empresa. Por ejemplo, puede usar Azure AD para requerir autenticación multifactor al acceder a recursos importantes de la organización. Además, puede usar Azure AD para automatizar el aprovisionamiento de usuarios entre la instancia existente de Windows Server AD y las aplicaciones en la nube, incluida Microsoft 365. Por último, Azure AD proporciona eficaces herramientas que ayudan a proteger automáticamente las identidades y credenciales de los usuarios y a cumplir los requisitos de gobernanza de acceso. Para empezar, regístrese para obtener una [versión de evaluación gratuita durante 30 días de Azure Active Directory Premium](https://azure.microsoft.com/trial/get-started-active-directory/).

- **Desarrolladores de aplicaciones.** Como desarrollador de aplicaciones, puede usar Azure AD como enfoque basado en estándares para agregar el inicio de sesión único (SSO) a cualquier aplicación, lo que le permite trabajar con las credenciales existentes de un usuario. Azure AD también proporciona varias API que pueden ayudarle a crear experiencias de aplicación personalizadas que usen los datos existentes de la organización. Para empezar, regístrese para obtener una [versión de evaluación gratuita durante 30 días de Azure Active Directory Premium](https://azure.microsoft.com/trial/get-started-active-directory/). Para más información, también puede consultar [Azure Active Directory para desarrolladores](../develop/index.yml).

- **Suscriptores de Microsoft 365, Office 365, Azure o Dynamics CRM Online.** Los suscriptores usan Azure AD. Cada inquilino de Microsoft 365, Office 365, Azure y Dynamics CRM Online es automáticamente un inquilino de Azure AD. Puede empezar de inmediato a administrar el acceso a las aplicaciones en la nube integradas.

## <a name="what-are-the-azure-ad-licenses"></a>¿Qué son las licencias de Azure AD?

Los servicios para la empresa de Microsoft Online, como Microsoft 365 o Microsoft Azure, requieren Azure AD para iniciar sesión y ayudar con la protección de identidad. Por consiguiente, si se suscribe a cualquiera de los servicios de negocio de Microsoft Online, obtendré automáticamente Azure AD con acceso a todas las características gratuitas.

Para mejorar la implementación de Azure AD, también puede agregar funcionalidades de pago mediante la actualización a las licencias Azure Active Directory Premium P1 o Premium P2. Las licencias de pago de Azure AD se crean sobre el directorio gratuito existente y proporcionan autoservicio, supervisión mejorada, informes de seguridad y un acceso seguro a sus usuarios móviles.

>[!Note]
>Para ver las opciones de precios de estas licencias, consulte [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).
>
>Azure Active Directory Premium P1 y Premium P2 no se admiten actualmente en China. Para más información acerca de los precios de Azure AD, póngase en contacto con el [foro de Azure Active Directory](https://azure.microsoft.com/support/community/?product=active-directory).

- **Azure Active Directory Free.** Proporciona administración de grupos y usuarios, sincronización de directorios locales, informes básicos, cambio de contraseñas de autoservicio para usuarios en la nube e inicio de sesión único en Azure, Microsoft 365 y muchas aplicaciones SaaS populares.

- **Azure Active Directory Premium P1.** Además de las características de la versión Free, P1 también permite a los usuarios de entornos híbridos acceder a recursos locales y en la nube. También admite la administración avanzada, como grupos dinámicos, administración de grupos de autoservicio, Microsoft Identity Manager (un conjunto de administración local de identidades y acceso) y funcionalidades de reescritura en la nube, que permiten el restablecimiento de contraseña de autoservicio a los usuarios locales.

- **Azure Active Directory Premium P2.** Además de las características de las licencias Free y P1, la licencia P2 ofrece también [Azure Active Directory Identity Protection](../identity-protection/overview-identity-protection.md), que proporciona acceso condicional basado en riesgos a sus aplicaciones y datos críticos de la compañía, y [Privileged Identity Management](../privileged-identity-management/pim-getting-started.md), que permite detectar, restringir y supervisar los administradores y su acceso a los recursos, además de proporcionar acceso Just-In-Time cuando sea necesario.

- **Licencias de la característica de "Pago por uso".** También puede obtener licencias de características adicionales, como Azure Active Directory Business-to-Customer (B2C). B2C puede ayudarle a proporcionar soluciones de administración de acceso y de identidad para las aplicaciones orientadas al cliente. Para más información, consulte [Documentación de Azure Active Directory B2C](../../active-directory-b2c/index.yml).

Para más información acerca de cómo asociar una suscripción de Azure a Azure AD, consulte [Asociación o adición de una suscripción de Azure a Azure Active Directory](active-directory-how-subscriptions-associated-directory.md) y para más información acerca de cómo asignar licencias a usuarios, consulte [Cómo asignar o quitar licencias de Azure Active Directory](license-users-groups.md).

## <a name="which-features-work-in-azure-ad"></a>¿Qué características funcionan en Azure AD?

Después de elegir su licencia de Azure AD, obtendrá acceso a algunas de las características siguientes para su organización, o a todas ellas:

|Category|Descripción|
|-------|-----------|
|Administración de aplicaciones|Administre las aplicaciones en la nube y locales mediante el proxy de aplicación, el inicio de sesión único, el portal Mis aplicaciones (también conocido como Panel de acceso) y aplicaciones de software como servicio (SaaS). Para más información, consulte [Provisión de acceso remoto seguro a aplicaciones locales](../manage-apps/application-proxy.md) y [Documentación sobre la administración de aplicaciones](../manage-apps/index.yml).|
|Authentication|Administre el restablecimiento de contraseñas de autoservicio de Azure Active Directory, Multi-Factor Authentication, la lista personalizada de contraseñas prohibidas y el bloqueo inteligente. Para más información, consulte la [documentación de Autenticación de Azure AD](../authentication/index.yml).|
|Azure Active Directory para desarrolladores|Cree aplicaciones que inicie sesión en todas las identidades de Microsoft, obtenga tokens para llamar a Microsoft Graph, otras API de Microsoft API o API personalizadas. Para más información, consulte [Plataforma de identidad de Microsoft (Azure Active Directory para desarrolladores)](../develop/index.yml).|
|Negocio a negocio (B2B)|Administre los usuarios invitados y los asociados externos sin perder el control sobre sus propios datos corporativos. Para más información, consulte [Documentación de Azure Active Directory B2B](../external-identities/index.yml).|
|Negocio a cliente (B2C)|Personalice y controle la forma en que los usuarios se suscriben, inician sesión y administran sus perfiles al usar sus aplicaciones. Para más información, consulte [Documentación de Azure Active Directory B2C](../../active-directory-b2c/index.yml).|
|Acceso condicional|Administre el acceso a sus aplicaciones en la nube. Para más información, consulte [Documentación sobre el acceso condicional de Azure AD](../conditional-access/index.yml).|
|Administración de dispositivos|Administre la forma en que los dispositivos en la nube o locales acceden a los datos corporativos. Para más información, consulte la [documentación de Azure AD Device Management](../devices/index.yml).|
|Servicios de dominio|Combine máquinas virtuales de Azure en un dominio sin controladores de dominio. Para más información, consulte [Documentación de Azure AD Domain Services](../../active-directory-domain-services/index.yml).|
|Usuarios de empresa|Administre la asignación de licencias, el acceso a las aplicaciones y configure delegados mediante grupos y roles de administrador. Para más información, consulte [Documentación sobre administración de usuarios de Azure Active Directory](../users-groups-roles/index.yml).|
|Identidad híbrida|Use Azure Active Directory Connect y Connect Health para proporcionar una identidad de usuario individual para la autenticación y autorización en todos los recursos, independientemente de la ubicación (nube o local). Para más información, consulte [Documentación de identidad híbrida](../hybrid/index.yml).|
|Gobernanza de identidades|Administre la identidad de su organización a través de controles de acceso a empleados, asociados comerciales, proveedores, servicios y aplicaciones. También puede realizar revisiones de acceso. Para más información, consulte la [documentación sobre la gobernanza de identidades en Azure AD](../governance/identity-governance-overview.md) y las [revisiones de acceso de Azure AD](../governance/access-reviews-overview.md).|
|Protección de identidad|Detecte posibles puntos vulnerables que afecten a identidades de su organización, configure directivas para responder a acciones sospechosas y, después, realice las acciones adecuadas para resolverlas. Para más información, consulte [Azure AD Identity Protection](../identity-protection/index.yml).|
|Identidades administradas de recursos de Azure|Proporcione a los servicios de Azure una identidad administrada automáticamente en Azure AD que puede autenticar cualquier servicio de autenticación compatible con Azure AD, incluido Key Vault. Para más información, consulte [¿Qué es Managed Identities for Azure Resources?](../managed-identities-azure-resources/overview.md)|
|Privileged Identity Management (PIM)|Administre, controle y supervise el acceso dentro de su organización. Esta característica incluye el acceso a los recursos de Azure AD y Azure, así como a otros servicios Microsoft Online Services, como Microsoft 365 o Intune. Para obtener más información, vea [Azure AD Privileged Identity Management](../privileged-identity-management/index.yml).|
|Informes y supervisión|Obtenga información acerca de los patrones de seguridad y de uso del entorno. Para más información, consulte [Informes y supervisión de Azure Active Directory](../reports-monitoring/index.yml).|

## <a name="terminology"></a>Terminología

Para conocer mejor Azure AD y su documentación, es aconsejable revisar los términos siguientes.

|Término o concepto|Descripción|
|---------------|-----------|
|Identidad| Algo que se puede autenticar. Una identidad puede ser un usuario con un nombre de usuario y una contraseña. Entre las identidades también se incluyen aplicaciones u otros servidores que podrían requerir autenticación a través de claves secretas o certificados.|
|Cuenta| Una identidad que tiene datos asociados a ella. No puede tener una cuenta sin una identidad.|
|Cuenta de Azure AD| Una identidad que se crea mediante Azure AD u otro servicio en la nube de Microsoft, como Microsoft 365. Las identidades se almacenan en Azure AD y pueden acceder a ellas las suscripciones de servicio en la nube de su organización. Esta cuenta se denomina a veces también cuenta profesional o educativa.|
|Administrador de cuenta|Este rol de administrador de suscripción clásica conceptualmente es el propietario de facturación de una suscripción. Este rol tiene acceso al [centro de cuentas de Azure](https://account.azure.com/Subscriptions) y permite administrar todas las suscripciones de una cuenta. Para más información, consulte [Roles de administrador de suscripciones clásico, de Azure y de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).|
|Administrador de servicios|Este rol de administrador de suscripciones clásico permite administrar todos los recursos de Azure, incluido el acceso. Tiene el acceso equivalente a un usuario al que se le ha asignado la función de propietario en el ámbito de la suscripción. Para más información, consulte [Roles de administrador de suscripciones clásico, de Azure y de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).|
|Propietario|Este rol ayuda a administrar todos los recursos de Azure, incluido el acceso. Este rol se basa en un sistema de autorización más reciente llamado control de acceso basado en rol de Azure (RBAC de Azure) que proporciona una administración detallada del acceso a los recursos de Azure. Para más información, consulte [Roles de administrador de suscripciones clásico, de Azure y de administrador de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).|
|Administrador global de Azure AD|Este rol de administrador se asigna automáticamente a quien haya creado el inquilino de Azure AD. Los administradores globales pueden realizar todas las funciones administrativas de Azure AD y los servicios que se federan con Azure AD, como Exchange Online, SharePoint Online y Skype Empresarial Online. Puede tener varios administradores globales, pero estos administradores son los únicos que pueden asignar roles de administrador (lo que incluye la asignación de otros administradores globales) a los usuarios. Tenga en cuenta que este rol de administrador se denomina Administrador global en Azure Portal, pero **Administrador de empresa** en la API de Microsoft Graph y Azure AD PowerShell. Para obtener más información acerca de los distintos roles de administrador, consulte [Permisos de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md).|
|Suscripción de Azure| Se usa para pagar los servicio en la nube de Azure. Puede tener muchas suscripciones y están vinculadas a una tarjeta de crédito.|
|Inquilino de Azure| Es una instancia dedicada y de confianza de Azure AD que se crea automáticamente cuando una organización se suscribe a un servicio en la nube de Microsoft, como Microsoft Azure, Microsoft Intune u Microsoft 365. Un inquilino de Azure representa una organización individual.|
|Un solo inquilino| Los inquilinos de Azure que acceden a otros servicios en un entorno dedicado se consideran inquilino individuales.|
|Multiinquilino| Los inquilinos de Azure que tienen acceso a otros servicios en un entorno compartido, en varias organizaciones, se consideran varios inquilinos.|
|Directorio de Azure AD|Todos los inquilinos de Azure tienen un directorio de Azure AD dedicado y de confianza. El directorio de Azure AD incluye los usuarios, grupos y aplicaciones del inquilino y se usa para realizar funciones de administración de acceso y de identidad en los recursos del inquilino.|
|Dominio personalizado|Cada directorio nuevo de Azure AD incluye un nombre de dominio inicial, nombre_de_dominio.onmicrosoft.com. Además de dicho nombre inicial, también puede agregar nombres de dominio de la organización, que incluyen los nombres que utiliza para hacer negocios y los que utilizan los usuarios para acceder a los recursos de su organización, a la lista. La adición de nombres de dominio personalizados le ayuda a crear nombres de usuario que resultan familiares a los usuarios, como alain@contoso.com.|
|Cuenta Microsoft (también denominada MSA)|Las cuentas personales que proporcionan acceso a los productos y servicios en la nube de Microsoft orientados al consumidor, como Outlook, OneDrive, Xbox LIVE o Microsoft 365. Su cuenta Microsoft se crean y almacenan en el sistema de cuentas de identidad de consumidor de Microsoft que ejecuta Microsoft.|

## <a name="next-steps"></a>Pasos siguientes

- [Suscripción a Azure Active Directory Premium](active-directory-get-started-premium.md)

- [Asociación o adición de una suscripción de Azure a Azure Active Directory](active-directory-how-subscriptions-associated-directory.md)

- [Lista de comprobación de la característica de licencia de Azure Active Directory Premium P2](active-directory-deployment-checklist-p2.md)