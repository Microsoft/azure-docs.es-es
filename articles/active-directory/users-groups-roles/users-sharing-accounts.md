---
title: Uso compartido de cuentas y credenciales en Azure Active Directory | Microsoft Docs
description: Describe cómo Azure Active Directory permite a las organizaciones compartir de forma segura las cuentas de aplicaciones locales y los servicios en la nube de consumidor.
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 08/01/2019
ms.author: curtand
ms.reviewer: jeffsta
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 38b474d4b454c61b83bfb1d62230930a0b569b1d
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88164230"
---
# <a name="sharing-accounts-with-azure-ad"></a>Uso compartido de cuentas con Azure AD

## <a name="overview"></a>Información general

A veces, las organizaciones necesitan usar un nombre de usuario y una contraseña únicos para varias personas. Esto sucede normalmente en dos casos:

* Al obtener acceso a aplicaciones que requieren un inicio de sesión único y una contraseña para cada usuario, ya sean aplicaciones locales o servicios en la nube de consumidor (por ejemplo, cuentas de medios sociales corporativos).
* Al crear entornos multiusuario. En este caso puede tener una sola cuenta local que tenga privilegios elevados y que se use para realizar actividades básicas de configuración, administración y recuperación. Por ejemplo, puede tener la cuenta local de "administrador global" de Office 365 o la cuenta raíz en Salesforce.

Tradicionalmente, estas cuentas se comparten al distribuir las credenciales (nombre de usuario y contraseña) a las personas adecuadas o almacenándolas en una ubicación compartida donde varios agentes de confianza pueden tener acceso a ellas.

El modelo tradicional de uso compartido tiene varias desventajas:

* Habilitar el acceso a las nuevas aplicaciones requiere distribuir las credenciales a todos los usuarios que necesiten acceso.
* Cada aplicación compartida puede requerir su propio conjunto de credenciales compartidas, por lo que los usuarios tienen que recordar varios conjuntos de credenciales. Cuando los usuarios tienen que recordar muchas credenciales, aumenta el riesgo de que recurran a prácticas peligrosas. (por ejemplo, anotar las contraseñas).
* No se puede determinar quién tiene acceso a una aplicación.
* No se puede determinar quién *accedió* a una aplicación.
* Si tiene que quitar el acceso a una aplicación, debe actualizar las credenciales y volver a distribuirlas a todos los usuarios que necesiten acceder a esa aplicación.

## <a name="azure-active-directory-account-sharing"></a>Uso compartido de cuentas de Azure Active Directory

Azure AD proporciona un enfoque nuevo para el uso de cuentas compartidas que elimina estos inconvenientes.

El administrador de Azure AD configura las aplicaciones a las que un usuario puede tener acceso mediante el panel de acceso y la selección del tipo de inicio de sesión único que mejor se adapta a esa aplicación. Uno de estos tipos, el *inicio de sesión único con contraseña*, permite a Azure AD actuar como una especie de "intermediario" durante el proceso de inicio de sesión de esa aplicación.

Los usuarios inician sesión una sola vez con sus cuentas profesionales. Estas cuentas suelen ser las que se usan habitualmente para obtener acceso al correo electrónico o al escritorio. Pueden detectar y acceder solo a aquellas aplicaciones que tienen asignadas. Con cuentas compartidas, esta lista de aplicaciones puede incluir cualquier número de credenciales compartidas. El usuario final no necesita recordar ni anotar las distintas cuentas que puede que use.

Las cuentas compartidas no solo aumentan la supervisión y mejoran la facilidad de uso, también optimizan la seguridad. Los usuarios con permisos para usar las credenciales no ven la contraseña compartida, sino que obtienen permiso para usar la contraseña como parte de un flujo de autenticación orquestado. Además, algunas aplicaciones de SSO con contraseña le ofrecen la opción de usar Azure AD para sustituir (actualizar) periódicamente las contraseñas. El sistema usa contraseñas largas y complejas, lo que aumenta la seguridad de la cuenta. Gracias a ello, el administrador puede conceder o revocar fácilmente el acceso a una aplicación y saber quién tiene acceso a la cuenta y quién ha accedido a ella previamente.

Azure AD admite las cuentas compartidas de cualquier usuario de Enterprise Mobility Suite (EMS) o plan de licencias Azure AD Premium en todos los tipos de aplicaciones de inicio de sesión único con contraseña. Puede compartir las cuentas de las miles de aplicaciones previamente integradas en la galería de aplicaciones y puede agregar su propia aplicación de autenticación mediante contraseña con [aplicaciones de inicio de sesión único personalizadas](../manage-apps/what-is-single-sign-on.md).

Entre las características de Azure AD que permiten el uso compartido de las cuentas se incluyen las siguientes:

* [Inicio de sesión único con contraseña](../manage-apps/sso-options.md#password-based-sso)
* Agente de inicio de sesión único con contraseña
* [Asignación de grupos](groups-self-service-management.md)
* Aplicaciones de contraseñas personalizadas
* [Panel/informes de uso de aplicaciones](../active-directory-passwords-get-insights.md)
* Portales de acceso para usuarios finales
* [Proxy de aplicaciones](../manage-apps/application-proxy.md)
* [Active Directory Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureActiveDirectory)

## <a name="sharing-an-account"></a>Uso compartido de una cuenta

Para usar Azure AD para compartir una cuenta, debe hacer lo siguiente:

* Agregar una aplicación de la [galería de aplicaciones](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.AzureActiveDirectory) o una [aplicación personalizada](https://cloudblogs.microsoft.com/enterprisemobility/2015/06/17/bring-your-own-app-with-azure-ad-self-service-saml-configuration-now-in-preview/)
* Configurar la aplicación para el inicio de sesión único (SSO) con contraseña
* Usar la [asignación basada en grupos](groups-saasapps.md) y seleccionar la opción para escribir una credencial compartida.

También puede hacer que su cuenta compartida sea más segura con Multi-Factor Authentication (MFA) (más información acerca de la [protección de aplicaciones con Azure AD](../authentication/concept-mfa-whichversion.md)) y puede delegar la capacidad para administrar quién tiene acceso a la aplicación mediante la administración de grupos de [autoservicio de Azure AD](groups-self-service-management.md).

## <a name="next-steps"></a>Pasos siguientes

* [Administración de aplicaciones en Azure Active Directory](../manage-apps/what-is-application-management.md)
* [Protección de aplicaciones con acceso condicional](../active-directory-conditional-access-azure-portal.md)
* [Administración de grupos de autoservicio/SSAA](groups-self-service-management.md)
