---
title: Uso de Azure AD para aplicaciones que no se enumeran en la galería de aplicaciones
description: Comprenda cómo integrar las aplicaciones que no se enumeran en la galería de Azure AD.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 07/27/2020
ms.author: kenwith
ms.reviewer: arvinh,luleon
ms.openlocfilehash: 262c3a7c8a668ef25d092b5d523743e0ece89cc3
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88641150"
---
# <a name="using-azure-ad-for-applications-not-listed-in-the-app-gallery"></a>Uso de Azure AD para aplicaciones que no se enumeran en la galería de aplicaciones

En el inicio rápido [Adición de una aplicación](add-application-portal.md), aprenderá a añadir una aplicación al inquilino de Azure AD.

Además de las opciones de la [galería de aplicaciones de Azure AD](https://azure.microsoft.com/documentation/articles/active-directory-saas-tutorial-list/), tiene la opción de agregar una **aplicación que no sea de la galería**. 

## <a name="capabilities-for-apps-not-listed-in-the-azure-ad-gallery"></a>Funcionalidad de las aplicaciones que no se enumeran en la galería de Azure AD

Puede agregar cualquier aplicación que ya exista en su organización o cualquier aplicación de terceros de un proveedor que aún no forme parte de la galería de Azure AD. Dependiendo de su [contrato de licencia](https://azure.microsoft.com/pricing/details/active-directory/), las siguientes funcionalidades están disponibles:

- Integración de autoservicio de cualquier aplicación que admita proveedores de identidades de [Lenguaje de marcado de aserción de seguridad (SAML) 2.0](https://wikipedia.org/wiki/SAML_2.0) (iniciado por el proveedor de servicios o por el proveedor de identidades)
- Integración de autoservicio de cualquier aplicación web que tenga una página de inicio de sesión basada en HTML que use [SSO basado en contraseña](sso-options.md#password-based-sso)
- Conexión de autoservicio de las aplicaciones que usan el protocolo [System for Cross-Domain Identity Management (SCIM) para el aprovisionamiento de usuarios](../app-provisioning/use-scim-to-provision-users-and-groups.md)
- Posibilidad de agregar vínculos a cualquier aplicación del [iniciador de aplicaciones de Office 365](https://www.microsoft.com/microsoft-365/blog/2014/10/16/organize-office-365-new-app-launcher-2/) o de la página [Aplicaciones](sso-options.md#linked-sign-on)

Si necesita directrices de los desarrolladores sobre cómo integrar aplicaciones personalizadas con Azure AD, consulte [Escenarios de autenticación para Azure AD](../develop/authentication-scenarios.md). Cuando desarrolle una aplicación que use un protocolo moderno como [OpenId Connect/OAuth](../develop/active-directory-v2-protocols.md) para autenticar a los usuarios, puede registrarla en la plataforma de identidad de Microsoft mediante la experiencia [Registros de aplicaciones](../develop/quickstart-register-app.md) en Azure Portal.

## <a name="next-steps"></a>Pasos siguientes

- [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)

