---
title: Aplicaciones de página única en Azure Active Directory
description: Se describe qué son las aplicaciones de página única (SPA) y los conceptos básicos sobre el flujo de protocolos, el registro y la expiración de tokens en este tipo de aplicación.
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
ms.openlocfilehash: 1f6f66779bec9ed4e38e5a662c2d3728ba2034b6
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545294"
---
# <a name="single-page-applications"></a>Aplicación de página única

Las aplicaciones de página única (SPA) normalmente están estructuradas como una capa de presentación (front-end) de JavaScript que se ejecuta en el explorador y un back-end de API web que se ejecuta en un servidor e implementa la lógica de negocios de la aplicación. Para más información sobre la concesión de autorización implícita y ayudarle a decidir si es adecuada para su escenario de aplicación, consulte [Descripción del flujo de concesión implícita de OAuth2 de Azure Active Directory](v1-oauth2-implicit-grant-flow.md).

En este escenario, cuando el usuario inicia sesión, el front-end de JavaScript usa la [biblioteca de autenticación de Active Directory para JavaScript (ADAL.JS)](https://github.com/AzureAD/azure-activedirectory-library-for-js) y la concesión de autorización implícita para obtener un token de identificación (id_token) de Azure AD. El token se almacena en caché y el cliente lo adjunta a la solicitud como token de portador al realizar llamadas al back-end de la API web, que está protegido con el middleware de OWIN.

## <a name="diagram"></a>Diagrama

![Diagrama de aplicación de página única](./media/authentication-scenarios/single_page_app.png)

## <a name="protocol-flow"></a>Flujo de protocolos

1. El usuario navega a la aplicación web.
1. La aplicación devuelve el front-end de JavaScript (capa de presentación) en el explorador.
1. El usuario inicia sesión, por ejemplo, haciendo clic en un vínculo de inicio de sesión. El explorador envía un comando GET al extremo de autorización de Azure AD para solicitar un identificador de token. Esta solicitud incluye el identificador de la aplicación y la dirección URL de respuesta en los parámetros de consulta.
1. Azure AD valida la dirección URL de respuesta contrastándola con la dirección URL de respuesta registrada que se configuró en Azure Portal.
1. El usuario inicia sesión en la página de inicio de sesión.
1. Si la autenticación es correcta, Azure AD crea un token de identificador y lo devuelve como un fragmento de URL (#) a la URL de respuesta de la aplicación. En el caso de una aplicación de producción, esta URL de respuesta debe ser HTTPS. El token devuelto incluye notificaciones sobre el usuario y Azure AD, necesarias para que la aplicación valide el token.
1. El código de cliente de JavaScript que se ejecuta en el explorador extrae el token de la respuesta para usarlo para proteger las llamadas al back-end de la API web de la aplicación.
1. El explorador llama al back-end de la API web de la aplicación con el token de identificador en el encabezado de autorización. El servicio Autenticación de Azure AD emite un token de identificador que puede usarse como un token de portador, si el recurso es el mismo que el identificador de cliente (en este caso, esto es true porque la API web es el back-end de la aplicación).

## <a name="code-samples"></a>Ejemplos de código

Consulte los [ejemplos de código para escenarios de aplicación de página única](sample-v1-code.md#single-page-applications). Agregamos nuevos ejemplos continuamente, así que no olvide consultarlos con frecuencia.

## <a name="app-registration"></a>Registro de aplicación

* Inquilino único: si va a compilar una aplicación únicamente para la organización, esta debe registrarse en el directorio de la empresa mediante Azure Portal.
* Multiinquilino: si va a compilar una aplicación que puede ser utilizada por usuarios ajenos a la organización, esta debe registrarse en el directorio de la compañía, pero también en el directorio de cada organización que vaya a usar la aplicación. Para que la aplicación esté disponible en su directorio, puede incluir un proceso de inicio de sesión para que los clientes puedan dar su consentimiento a la aplicación. Cuando inicien sesión en la aplicación, se les presentará un cuadro de diálogo que muestra los permisos que la aplicación requiere y la opción de consentimiento. Según los permisos necesarios, es posible que se requiera que un administrador de la otra organización dé su consentimiento. Cuando el usuario o el administrador dan su consentimiento, la aplicación queda registrada en el directorio.

Después de registrar la aplicación, esta debe configurarse para usar el protocolo de concesión implícita OAuth 2.0. De forma predeterminada, este protocolo está deshabilitado para las aplicaciones. Para habilitar el protocolo de concesión implícita OAuth2 para su aplicación, edite el manifiesto de aplicación en Azure Portal y establezca el valor de "oauth2AllowImplicitFlow" en true. Para más información, consulte [Manifiesto de aplicación](reference-app-manifest.md).

## <a name="token-expiration"></a>Expiración del token

Usar ADAL.js le ayuda a:

* Actualizar un token expirado
* Solicitar un token de acceso para llamar a un recurso de la API web

Después de una autenticación correcta, Azure AD escribe una cookie en el explorador del usuario para establecer una sesión. Tenga en cuenta que la sesión tiene lugar entre el usuario y Azure AD, no entre el usuario y la aplicación web. Cuando un token expira, ADAL.js usa esta sesión para obtener otro token automáticamente. ADAL.js usa un iFrame oculto para enviar y recibir la solicitud con el protocolo de concesión implícita OAuth. ADAL.js también puede usar el mismo mecanismo para obtener acceso automático a tokens para otros recursos de la API web a los que la aplicación llame, siempre que dichos recursos admitan el uso compartido de recursos entre orígenes (CORS), estén registrados en el directorio del usuario y este haya dado el consentimiento requerido durante el inicio de sesión.

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre otros [tipos y escenarios de aplicaciones](app-types.md).
* Más información sobre los [conceptos básicos de autenticación](authentication-scenarios.md) de Azure AD.
