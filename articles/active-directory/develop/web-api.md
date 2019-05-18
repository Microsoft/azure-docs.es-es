---
title: Aplicaciones de API web en Azure Active Directory
description: Se describe qué son las aplicaciones de API web y los conceptos básicos sobre el flujo de protocolos, el registro y la expiración de tokens en este tipo de aplicación.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: saeeda, jmprieur, andret
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 484e6b4c5f0e064254c957b07b8ba15ef98f2634
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545208"
---
# <a name="web-api"></a>API web

Las aplicaciones de API web son aplicaciones web que necesitan obtener los recursos de una API web. En este escenario, existen dos tipos de identidad que la aplicación web puede usar para autenticar y llamar a la API web:

- **Identidad de aplicación:** en este escenario se usa la concesión de credenciales de cliente de OAuth 2.0 para autenticarse como aplicación y obtener acceso a la API web. Cuando se usa una identidad de aplicación, la API web solo puede detectar que la aplicación web la llama, ya que la API web no recibe ninguna información sobre el usuario. Si la aplicación recibe información sobre el usuario, se enviará mediante el protocolo de aplicación, sin la firma de Azure AD. La API web confía en que la aplicación web autenticó al usuario. Por ello, este patrón se conoce como subsistema de confianza.
- **Identidad de usuario delegado:** este escenario se puede conseguir de dos maneras: OpenID Connect y concesión de código de autorización OAuth 2.0 con un cliente confidencial. La aplicación web obtiene un token de acceso para el usuario, que demuestra a la API web que el usuario se autenticó correctamente ante la aplicación web y que la aplicación web pudo obtener una identidad de usuario delegado para llamar a la API web. Este token de acceso se envía en la respuesta a la API web, que autoriza al usuario y devuelve el recurso deseado.

Los tipos de identidad de aplicación y de identidad de usuario delegado se tratan en el flujo siguiente. La diferencia clave entre ambos es que la identidad de usuario delegado debe adquirir primero un código de autorización para que el usuario pueda iniciar sesión y obtener acceso a la API web.

## <a name="diagram"></a>Diagrama

![Diagrama de aplicación web a API web](./media/authentication-scenarios/web_app_to_web_api.png)

## <a name="protocol-flow"></a>Flujo de protocolos

### <a name="application-identity-with-oauth-20-client-credentials-grant"></a>Identidad de aplicación con concesión de credenciales de cliente OAuth 2.0

1. Un usuario inicia sesión en Azure AD en la aplicación web (consulte la sección **Aplicaciones web**).
1. La aplicación web necesita adquirir un token de acceso para poder autenticarse ante la API web y recuperar el recurso deseado. Realiza una solicitud al punto de conexión de token de Azure AD y proporciona las credenciales, el identificador de la aplicación y el identificador URI del identificador de aplicación de la API web.
1. Azure AD autentica la aplicación y devuelve un token de acceso de JWT que se usa para llamar a la API web.
1. A través de HTTPS, la aplicación web usa el token de acceso de JWT devuelto para agregar la cadena JWT con una designación “Bearer” en el encabezado Authorization de la solicitud a la API web. A continuación, la API web valida el token de JWT y, si la validación es correcta, devuelve el recurso deseado.

### <a name="delegated-user-identity-with-openid-connect"></a>Identidad de usuario delegado con OpenID Connect

1. Un usuario inicia sesión en una aplicación web mediante Azure AD (vea la sección anterior Explorador web a aplicación web ). Si el usuario de la aplicación web todavía no ha dado su consentimiento para permitir que la aplicación web llame a la API web en su nombre, el usuario tendrá que dar su consentimiento. La aplicación mostrará los permisos que requiere y, si alguno de estos permisos es de nivel de administrador, un usuario normal del directorio no podrá dar su consentimiento. Este proceso de consentimiento solo se aplica a las aplicaciones multiempresa (de varios inquilinos), no a las aplicaciones de un solo inquilino, puesto que la aplicación ya tendrá los permisos necesarios. Cuando el usuario inicia sesión, la aplicación web recibe un token de identificador con información sobre el usuario, así como un código de autorización.
1. Con el código de autorización emitido por Azure AD, la aplicación web envía una solicitud al punto de conexión de token de Azure AD que incluye el código de autorización, detalles sobre la aplicación cliente (URI de redirección e identificador de la aplicación), además del recurso deseado (URI del identificador de la aplicación para la API web).
1. Azure AD valida el código de autorización e información sobre la aplicación web y la API web. Tras la correcta validación, Azure AD devuelve dos tokens: un token de acceso de JWT y un token de actualización de JWT.
1. A través de HTTPS, la aplicación web usa el token de acceso de JWT devuelto para agregar la cadena JWT con una designación “Bearer” en el encabezado Authorization de la solicitud a la API web. A continuación, la API web valida el token de JWT y, si la validación es correcta, devuelve el recurso deseado.

### <a name="delegated-user-identity-with-oauth-20-authorization-code-grant"></a>Identidad de usuario delegado con concesión de código de autorización OAuth 2.0

1. Un usuario ya ha iniciado sesión en una aplicación web, cuyo mecanismo de autenticación es independiente de Azure AD.
1. La aplicación web requiere un código de autorización para adquirir un token de acceso, por lo que emite una solicitud a través del explorador al punto de conexión de autorización de Azure AD y proporciona el identificador de la aplicación y el identificador URI de redirección de la aplicación web después de la correcta autenticación. El usuario inicia sesión en Azure AD.
1. Si el usuario de la aplicación web todavía no ha dado su consentimiento para permitir que la aplicación web llame a la API web en su nombre, el usuario tendrá que dar su consentimiento. La aplicación mostrará los permisos que requiere y, si alguno de estos permisos es de nivel de administrador, un usuario normal del directorio no podrá dar su consentimiento. Este consentimiento se aplica tanto a aplicaciones de un solo inquilino como a aplicaciones multiempresa. En el caso de las aplicaciones de un solo inquilino, un administrador puede realizar una operación de consentimiento de administrador en nombre de sus usuarios. Esto puede hacerse mediante el botón `Grant Permissions` en [Azure Portal](https://portal.azure.com). 
1. Una vez que el usuario haya dado su consentimiento, la aplicación web recibe el código de autorización que necesita para adquirir un token de acceso.
1. Con el código de autorización emitido por Azure AD, la aplicación web envía una solicitud al punto de conexión de token de Azure AD que incluye el código de autorización, detalles sobre la aplicación cliente (URI de redirección e identificador de la aplicación), además del recurso deseado (URI del identificador de la aplicación para la API web).
1. Azure AD valida el código de autorización e información sobre la aplicación web y la API web. Tras la correcta validación, Azure AD devuelve dos tokens: un token de acceso de JWT y un token de actualización de JWT.
1. A través de HTTPS, la aplicación web usa el token de acceso de JWT devuelto para agregar la cadena JWT con una designación “Bearer” en el encabezado Authorization de la solicitud a la API web. A continuación, la API web valida el token de JWT y, si la validación es correcta, devuelve el recurso deseado.

## <a name="code-samples"></a>Ejemplos de código

Vea los ejemplos de código para escenarios de aplicación web a API web. Agregamos nuevos ejemplos continuamente, así que no olvide consultarlos con frecuencia. [Aplicación web a API web](sample-v1-code.md#web-applications-signing-in-users-calling-microsoft-graph-or-a-web-api-with-the-users-identity).

## <a name="app-registration"></a>Registro de aplicación

Para registrar una aplicación con el extremo de la versión 1.0 de Azure AD, consulte [registrar una aplicación](quickstart-register-app.md).

* Un solo inquilino: tanto en el caso de la identidad de aplicación como en el de la identidad de usuario delegado, la aplicación web y la API web deben estar registradas en el mismo directorio de Azure AD. La API web puede configurarse para exponer un conjunto de permisos, que se usan para limitar el acceso de la aplicación web a sus recursos. Si se usa una identidad de usuario delegado, es necesario que la aplicación web seleccione los permisos deseados en el menú desplegable **Permisos para otras aplicaciones** de Azure Portal. Este paso no es necesario si se usa el tipo de identidad de aplicación.
* Multiempresa (varios inquilinos): en primer lugar, la aplicación web se configura para indicar los permisos que requiere para ser funcional. La lista de permisos necesarios se muestra en un cuadro de diálogo cuando un usuario o administrador del directorio de destino da su consentimiento a la aplicación, lo que la pone a disposición de la organización. Algunas aplicaciones solo requieren permisos de nivel de usuario, para los que cualquier usuario de la organización puede dar el consentimiento. Otras aplicaciones requieren permisos de nivel de administrador, para los que un usuario de la organización no puede dar su consentimiento. Solo un administrador de directorio puede dar su consentimiento a las aplicaciones que requieran este nivel de permisos. Cuando el usuario o el administrador dan su consentimiento, la aplicación web y la API web quedan registradas en el directorio.

## <a name="token-expiration"></a>Expiración del token

Cuando la aplicación web usa su código de autorización para obtener un token de acceso de JWT, también recibe un token de actualización de JWT. Cuando el token de acceso expira, se puede usar el token de actualización para volver a autenticar al usuario sin solicitarle que inicie sesión de nuevo. Este token de actualización se usa a continuación para autenticar al usuario, lo que se traduce en un token de acceso y un token de actualización nuevos.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre otros [tipos y escenarios de aplicaciones](app-types.md).
- Más información sobre los [conceptos básicos de autenticación](authentication-scenarios.md) de Azure AD.
