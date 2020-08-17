---
title: Habilitar el inicio de sesión único para la aplicación multiinquilino
description: Guía para proveedores de software independientes sobre la integración con Azure Active Directory
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: conceptual
ms.workload: identity
ms.date: 05/22/2019
ms.author: kenwith
ms.reviewer: jeeds
ms.collection: M365-identity-device-management
ms.openlocfilehash: cabcaacbdc7e3e94e62c8fb45274c4d4f9390ddd
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87273412"
---
# <a name="enable-single-sign-on-for-your-multi-tenant-application"></a>Habilitar el inicio de sesión único para la aplicación multiinquilino  

Cuando ofrece la aplicación para que la utilicen otras empresas mediante su compra o suscripción, hace que la aplicación esté disponible para los clientes dentro de sus propios inquilinos de Azure. Esto se conoce como creación de una aplicación multiinquilino. Para obtener información general sobre este concepto, consulte [Aplicaciones multiinquilino en Azure](https://docs.microsoft.com/azure/dotnet-develop-multitenant-applications) e [Inquilinos en Azure Active Directory](../develop/single-and-multi-tenant-apps.md).

## <a name="what-is-single-sign-on"></a>¿Qué es el inicio de sesión único?

El inicio de sesión único (SSO) agrega seguridad y comodidad cuando los usuarios inician sesión en aplicaciones mediante Azure Active Directory y otras identidades. Si una aplicación tiene el inicio de sesión único habilitado, los usuarios no tienen que especificar credenciales independientes para acceder a esa aplicación. Para obtener una explicación completa sobre el inicio de sesión único, consulte [Inicio de sesión único en aplicaciones en Azure Active Directory](what-is-single-sign-on.md).

## <a name="why-enable-single-sign-on-in-your-application"></a>¿Por qué habilitar el inicio de sesión único en la aplicación?

Habilitar el inicio de sesión único en la aplicación multiinquilino presenta muchas ventajas. Si habilita el inicio de sesión único de la aplicación:

* La aplicación puede aparecer en Azure Marketplace, donde millones de organizaciones que usan Azure Active Directory pueden detectarla.
  * Permite a los clientes configurar rápidamente la aplicación con Azure AD.

* Se puede detectar la aplicación en la galería de aplicaciones de Office 365, el iniciador de aplicaciones de Office 365 y en Microsoft Search en Office.com.

* La aplicación puede usar la API REST de Microsoft Graph para acceder a los datos que ayudan a la productividad del usuario que están disponibles en Microsoft Graph.

* Puede reducir los costos de soporte técnico haciéndola más sencilla para los clientes.
  * La documentación específica de la aplicación coproducida con el equipo de Azure AD para los clientes mutuos facilita la adopción.
  * Si está habilitado el inicio de sesión único con un solo clic, los administradores de TI de los clientes no tendrán que aprender a configurar la aplicación para su uso en su organización.

* Puede proporcionar a los clientes la posibilidad de administrar por completo la autenticación y autorización de las identidades de sus empleados e invitados.

  * Colocando toda la responsabilidad de administración de la cuenta y del cumplimiento normativo en el cliente propietario de esas identidades.

  * Proporcionando la capacidad de habilitar o deshabilitar el inicio de sesión único para proveedores de identidades específicos, grupos o usuarios satisfagan sus necesidades empresariales.

* Puede aumentar la capacidad de comercialización y adopción. Muchas organizaciones grandes requieren (o aspiran a ello) que sus empleados tengan experiencias de inicio de sesión único sin problemas en todas las aplicaciones. Facilitar el inicio de sesión único es muy importante.

* Puede reducir los problemas del usuario final a la hora de utilizarla, lo cual podría aumentar el uso por parte del usuario final e incrementar los ingresos.

## <a name="how-to-enable-single-sign-on-in-your-published-application"></a>Habilitar el inicio de sesión único en la aplicación publicada

1. [Seleccione el protocolo de federación correcto para una aplicación multiinquilino](isv-choose-multi-tenant-federation.md).
1. Implemente el inicio de sesión único en la aplicación.
   - Consulte la [guía sobre patrones de autenticación](../develop/v2-app-types.md).
   - Consulte [Ejemplos de código de Azure Active Directory](../develop/sample-v2-code.md) para protocolos OIDC y OAuth.
1. [Cree el inquilino de Azure](isv-tenant-multi-tenant-app.md) y pruebe la aplicación.
1. [Cree y publique la documentación sobre el inicio de sesión único en su sitio web](isv-create-sso-documentation.md).
1. [Envíe el listado de aplicaciones](https://microsoft.sharepoint.com/teams/apponboarding/Apps/SitePages/Default.aspx) y asóciese con Microsoft para crear la documentación en el sitio web de Microsoft.
1. [Únase a Microsoft Partner Network (gratis) y cree su plan de comercialización](https://partner.microsoft.com/explore/commercial#gtm).
