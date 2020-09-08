---
title: 'Habilitar el aprovisionamiento automático de usuarios para aplicaciones multiinquilino: Azure AD'
description: Esta es una guía para proveedores de software independientes para habilitar el aprovisionamiento automatizado
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.topic: reference
ms.workload: identity
ms.date: 07/23/2019
ms.author: kenwith
ms.reviewer: zhchia
ms.openlocfilehash: 7ffdef43aa0fdcaab9e8ceae519cef9dfe5cdf6e
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235645"
---
# <a name="enable-automatic-user-provisioning-for-your-multi-tenant-application"></a>Habilitar el aprovisionamiento automático de usuarios para aplicaciones multiinquilino

El aprovisionamiento automático de usuarios es el proceso de automatizar la creación, el mantenimiento y la eliminación de identidades de usuario en sistemas de destino, como las aplicaciones de software como servicio.

## <a name="why-enable-automatic-user-provisioning"></a>¿Por qué debería habilitar el aprovisionamiento automático de usuarios?

Las aplicaciones que requieren un registro de usuario se encuentran en la aplicación antes de que el primer inicio de sesión de un usuario requiera el aprovisionamiento de usuarios. Existen beneficios para usted como proveedor de servicios y beneficios para sus clientes.

### <a name="benefits-to-you-as-the-service-provider"></a>Beneficios para usted como proveedor de servicios

* Aumente la seguridad de su aplicación gracias a la plataforma de identidad de Microsoft.

* Reduzca el esfuerzo real y percibido del cliente para adoptar su aplicación.

* Reduzca los costos gracias a la integración con varios proveedores de identidades (IdP) para el aprovisionamiento automático de usuarios mediante el uso del aprovisionamiento basado en System for Cross-Domain Identity Management (SCIM).

* Reduzca los costos del soporte técnico al proporcionar registros enriquecidos para ayudar a los clientes a solucionar problemas de aprovisionamiento de usuarios.

* Aumente la visibilidad de su aplicación en la [galería de aplicaciones de Azure AD](https://azuremarketplace.microsoft.com/marketplace/apps).

* Obtenga una lista prioritaria en la página de tutoriales de la aplicación.

### <a name="benefits-to-your-customers"></a>Beneficios para sus clientes

* Aumento de la seguridad al eliminar automáticamente el acceso a la aplicación para los usuarios que cambian roles o dejan la organización en la aplicación.

* Oportunidad de simplificar la administración de usuarios de la aplicación evitando errores humanos y trabajos repetitivos asociados con el aprovisionamiento manual.

* Reducción de los costos de hospedaje y mantenimiento de las soluciones de aprovisionamiento desarrolladas de forma personalizada.

## <a name="choose-a-provisioning-method"></a>Elija un método de aprovisionamiento

Azure AD proporciona varias rutas de integración para habilitar el aprovisionamiento automático de usuarios para la aplicación.

* El [Servicio de aprovisionamiento de Azure AD](../app-provisioning/user-provisioning.md) administra el aprovisionamiento y el desaprovisionamiento de usuarios desde Azure AD a la aplicación (aprovisionamiento saliente) y desde la aplicación hacia Azure AD (aprovisionamiento entrante). El servicio se conecta a los puntos de conexión de la API de administración de usuarios de System for Cross-Domain Identity Management (SCIM) que proporciona la aplicación.

* Al usar [Microsoft Graph](/graph/), la aplicación administra el aprovisionamiento entrante y saliente de usuarios y grupos desde Azure AD a la aplicación, consultando para ello Microsoft Graph API.

* El aprovisionamiento de usuarios del lenguaje de marcado de aserción de seguridad de tipo Just-In-Time (SAML JIT) se puede habilitar si su aplicación está usando SAML para la federación. Usa la información de las notificaciones enviada en el token SAML para aprovisionar usuarios.

Para poder determinar qué opción de integración debe usar en la aplicación, consulte la tabla de comparación de alto nivel y, a continuación, lea la información más detallada sobre cada opción.

| Capacidades habilitadas o mejoradas gracias al aprovisionamiento automático| Servicio de aprovisionamiento de Azure AD (SCIM 2.0)| Microsoft Graph API (OData v4.0)| SAML JIT |
|---|---|---|---|
| Administración de usuarios y grupos en Azure AD| √| √| Solo usuario |
| Administrar usuarios y grupos sincronizados desde la instancia local de Active Directory| √*| √*| Solo usuario* |
| Acceso a los datos más allá de los usuarios y grupos durante el aprovisionamiento; acceso a los datos de O365 (equipos, SharePoint, correo electrónico, calendario, documentos, etc.)| X+| √| X |
| Crear, leer y actualizar usuarios en función de las reglas de negocio| √| √| √ |
| Eliminar usuarios según las reglas de negocio| √| √| X |
| Administrar el aprovisionamiento automático de usuarios de todas las aplicaciones desde Azure Portal| √| X| √ |
| Compatibilidad con varios proveedores de identidades| √| X| √ |
| Compatibilidad con cuentas de invitado (B2B)| √| √| √ |
| Compatibilidad con cuentas no empresariales (B2C)| X| √| √ |

<sup>*</sup> – es necesaria la configuración de Azure AD Connect para sincronizar los usuarios de AD a Azure AD.  
<sup>+</sup >: el uso de SCIM para el aprovisionamiento no le impide integrar su aplicación con Microsoft Graph para otros fines.

## <a name="azure-ad-provisioning-service-scim"></a>Servicio de aprovisionamiento de Azure AD (SCIM)

Los servicios de aprovisionamiento de Azure AD usan [SCIM](https://aka.ms/SCIMOverview), un estándar del sector para el aprovisionamiento compatible con muchos proveedores de identidades (IdP), así como aplicaciones (por ejemplo, Slack, G Suite o Dropbox). Le recomendamos que use el servicio de aprovisionamiento de Azure AD si quiere admitir IdP además de Azure AD, ya que cualquier IdP compatible con SCIM puede conectarse a su punto de conexión de SCIM. Al compilar un punto de conexión de usuario sencillo, puede habilitar el aprovisionamiento sin tener que mantener su propio motor de sincronización. 

Para obtener más información sobre cómo usa el servicio de aprovisionamiento de Azure AD la especificación SCIM, consulte: 

* [Más información acerca del estándar SCIM](https://aka.ms/SCIMOverview)

* [Uso de System for Cross-Domain Identity Management (SCIM) para aprovisionar automáticamente a los usuarios y grupos de Azure Active Directory para aplicaciones](../app-provisioning/use-scim-to-provision-users-and-groups.md)

* [Información sobre la implementación de SCIM de Azure AD](../app-provisioning/use-scim-to-provision-users-and-groups.md)

## <a name="microsoft-graph-for-provisioning"></a>Microsoft Graph para el aprovisionamiento

Cuando usa Microsoft Graph para el aprovisionamiento, tiene acceso a todos los datos de usuario enriquecidos disponibles en Graph. Además de los detalles de los usuarios y grupos, también puede obtener información adicional como los roles del usuario, el administrador y los informes directos, los dispositivos propios y registrados y cientos de otras partes de datos disponibles en [Microsoft Graph](/graph/api/overview?view=graph-rest-1.0). 

Más de 15 millones de organizaciones y el 90 % de las compañías de la lista Fortune 500 usan Azure AD a la vez que se suscriben a servicios en la nube de Microsoft como Office 365, Microsoft Azure, Enterprise Mobility Suite o Microsoft 365. Puede usar Microsoft Graph para integrar su aplicación con flujos de trabajo administrativos, como la incorporación (y finalización de contratos) de empleados, el mantenimiento de los perfiles y muchas cosas más. 

Más información sobre el uso de Microsoft Graph para el aprovisionamiento:

* [Página principal de Microsoft Graph](https://developer.microsoft.com/graph)

* [Overview of Microsoft Graph (Información general de Microsoft Graph)](/graph/overview)

* [Información general de la autenticación de Microsoft Graph](/graph/auth/)

* [Introducción a Microsoft Graph](https://developer.microsoft.com/graph/get-started)

## <a name="using-saml-jit-for-provisioning"></a>Usar SAML JIT para el aprovisionamiento

Si quiere aprovisionar usuarios solo cuando inicien sesión por primera vez en la aplicación, y no necesita desaprovisionar usuarios automáticamente, SAML JIT es una opción que puede usar. Para poder usar SAML JIT, su aplicación debe ser compatible con SAML 2.0 como protocolo de federación.

SAML JIT usa la información de notificaciones en el token SAML para crear y actualizar la información del usuario en la aplicación. Los clientes pueden configurar estas notificaciones necesarias en la aplicación Azure AD según sea necesario. A veces, el aprovisionamiento de JIT debe habilitarse desde el lado de la aplicación para que el cliente pueda usar esta característica. SAML JIT es útil para crear y actualizar usuarios, pero no puede eliminar o desactivar los usuarios de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

* [Habilitar el inicio de sesión único en la aplicación](../manage-apps/isv-sso-content.md)

* [Envíe el listado de aplicaciones](https://microsoft.sharepoint.com/teams/apponboarding/Apps/SitePages/Default.aspx) y asóciese con Microsoft para crear la documentación en el sitio web de Microsoft.

* [Únase a Microsoft Partner Network (gratis) y cree su plan de comercialización](https://partner.microsoft.com/explore/commercial).
