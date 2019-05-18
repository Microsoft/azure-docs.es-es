---
title: Tipos de aplicaciones para la plataforma Microsoft identity | Azure
description: Los tipos de aplicaciones y escenarios admitidos por el punto de conexión de la plataforma (v2.0) de Microsoft identity.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 494a06b8-0f9b-44e1-a7a2-d728cf2077ae
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/06/2019
ms.author: ryanwi
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3ec10359c6f684b1d7fb008946b7ceca2ed626ba
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545373"
---
# <a name="application-types-for-microsoft-identity-platform"></a>Tipos de aplicaciones para la plataforma Microsoft identity

El punto de conexión de Microsoft identity platform (v2.0) admite la autenticación para una variedad de arquitecturas de aplicaciones modernas, todas ellas basadas en protocolos estándar del sector [OAuth 2.0 u OpenID Connect](active-directory-v2-protocols.md). En este artículo se describe los tipos de aplicaciones que se pueden compilar mediante el uso de la plataforma Microsoft identity, independientemente de su plataforma o idioma preferido. La información está diseñada para ayudarle a entender los escenarios de alto nivel antes de [empiece a trabajar con el código](v2-overview.md#getting-started).

> [!NOTE]
> El punto de conexión de plataforma de identidad de Microsoft no es compatible con todas las características y escenarios de Azure Active Directory (Azure AD). Para determinar si debe utilizar el punto de conexión de plataforma de identidad de Microsoft, obtenga información sobre [limitaciones de la plataforma de identidad de Microsoft](active-directory-v2-limitations.md).

## <a name="the-basics"></a>Conceptos básicos

Debe registrar cada aplicación que usa el punto de conexión de plataforma de identidad de Microsoft en el nuevo [portal los registros de aplicación](https://go.microsoft.com/fwlink/?linkid=2083908). El proceso de registro de la aplicación recopila y asigna algunos valores a la aplicación:

* Un **Id. de aplicación (cliente)** que identifica la aplicación
* Un **URI de redirección** que puede usarse para dirigir las respuestas de nuevo a la aplicación
* Algunos otros tipos de valores específicos del escenario que se admite como cuenta

Para más información, aprenda a [registrar una aplicación](quickstart-register-app.md).

Una vez registrada la aplicación, la aplicación se comunica con la plataforma Microsoft identity mediante el envío de solicitudes al punto de conexión. Proporcionamos bibliotecas y marcos de código abierto que controlan los detalles de estas solicitudes. También tiene la opción de implementar la lógica de autenticación por su cuenta mediante la creación de solicitudes a estos puntos de conexión:

```
https://login.microsoftonline.com/common/oauth2/v2.0/authorize
https://login.microsoftonline.com/common/oauth2/v2.0/token
```

## <a name="single-page-apps-javascript"></a>Aplicaciones de una página (JavaScript)

Muchas aplicaciones modernas tienen un front-end de aplicación de una página escrito principalmente en JavaScript. A menudo, se escribe con un marco como AngularJS, Ember.js o Durandal.js. El punto de conexión de plataforma de identidad de Microsoft es compatible con estas aplicaciones mediante el uso de la [flujo implícito de OAuth 2.0](v2-oauth2-implicit-grant-flow.md).

En este flujo, la aplicación recibe tokens directamente desde la plataforma Microsoft identity autorizar un extremo sin ningún intercambio de servidor a servidor. Toda la lógica de autenticación y el control de sesiones tiene lugar por completo en el cliente de JavaScript, sin redireccionamientos de página adicionales.

![Flujo de autenticación implícita](./media/v2-app-types/convergence-scenarios-implicit.svg)

Para ver este escenario en acción, pruebe uno de los ejemplos de código de aplicación de una página en el [Introducción a la plataforma de identidad de Microsoft](v2-overview.md#getting-started) sección.

## <a name="web-apps"></a>Aplicaciones web

En las aplicaciones web (. NET, PHP, Java, Ruby, Python, Node) a las que el usuario accede a través de un explorador, puede usar [OpenID Connect](active-directory-v2-protocols.md) para el inicio de sesión de usuario. En OpenID Connect, la aplicación web recibe un identificador de token. Un token de id. es un token de seguridad que comprueba la identidad del usuario y proporciona información sobre el usuario en forma de notificaciones:

```
// Partial raw ID token
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6ImtyaU1QZG1Cd...

// Partial content of a decoded ID token
{
    "name": "John Smith",
    "email": "john.smith@gmail.com",
    "oid": "d9674823-dffc-4e3f-a6eb-62fe4bd48a58"
    ...
}
```

Más detalles sobre los diferentes tipos de tokens que se usan en el punto de conexión de plataforma de identidad de Microsoft están disponibles en el [token de acceso](access-tokens.md) referencia y [referencia id_token](id-tokens.md)

En las aplicaciones de servidor web, el flujo de autenticación de inicio de sesión realiza estos pasos de alto nivel:

![Flujo de autenticación de aplicaciones web](./media/v2-app-types/convergence-scenarios-webapp.svg)

Puede garantizar la identidad del usuario al validar el token de identificador con una clave de firma pública que se recibe desde el punto de conexión de plataforma de identidad de Microsoft. Se establece una cookie de sesión, que puede usarse para identificar al usuario en las sucesivas solicitudes de página.

Para ver este escenario en acción, pruebe uno de los ejemplos de código de inicio de sesión de aplicación web en el [Introducción a la plataforma de identidad de Microsoft](v2-overview.md#getting-started) sección.

Además del inicio de sesión sencillo, una aplicación web de servidor podría tener la necesidad de acceder a otros servivio web, como una API de REST. En este caso, la aplicación de servidor web participa en un flujo combinado de OpenID Connect y OAuth 2.0, mediante el [flujo de código de autorización de OAuth 2.0](active-directory-v2-protocols.md). Para más información sobre este escenario, lea acerca de cómo [comenzar con aplicaciones web y API web](active-directory-v2-devquickstarts-webapp-webapi-dotnet.md).

## <a name="web-apis"></a>API web

Puede usar el punto de conexión de plataforma de identidad de Microsoft para proteger los servicios web, como la API Web RESTful de la aplicación. En lugar de tokens de id. y cookies de sesión, una API web usa un token de acceso de OAuth 2.0 para proteger sus datos y para autenticar las solicitudes entrantes. El llamador de una API web anexa un token de acceso en el encabezado de autorización de una solicitud HTTP, como esta:

```
GET /api/items HTTP/1.1
Host: www.mywebapi.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6...
Accept: application/json
...
```

La API web usa el token de acceso para comprobar la identidad del llamador de API y extraer información sobre el llamador de notificaciones que se codifican en dicho token. Más detalles sobre los diferentes tipos de tokens que se usan en el punto de conexión de plataforma de identidad de Microsoft están disponibles en el [token de acceso](access-tokens.md) referencia y [referencia id_token](id-tokens.md)

Una API web puede ofrecer a los usuarios la capacidad para administrar la participación/no participación de ciertas funcionalidades o datos mediante la exposición de permisos, conocidos también como [ámbitos](v2-permissions-and-consent.md). Para que una aplicación de llamada adquiera permiso para un ámbito, el usuario debe consentir el ámbito durante un flujo. El punto de conexión de plataforma de identidad de Microsoft solicita al usuario permiso y, a continuación, registra los permisos en todos los tokens de acceso que recibe la API Web. La API web valida los tokens de acceso que recibe en cada llamada y realiza comprobaciones de autorización.

Una API web puede recibir tokens de acceso de todos los tipos de aplicaciones, incluidas las aplicaciones de servidor web, aplicaciones móviles y de escritorio, aplicaciones de una página, demonios del lado del servidor e incluso otras API web. El flujo de alto nivel de una API web se parece a este:

![Flujo de autenticación de API web](./media/v2-app-types/convergence-scenarios-webapi.svg)

Para obtener información sobre cómo proteger una API Web mediante el uso de tokens de acceso de OAuth2, consulte los ejemplos de código de API Web en el [Introducción a la plataforma de identidad de Microsoft](v2-overview.md#getting-started) sección.

En muchos casos, las API web también se necesita realizar solicitudes salientes a otra bajada las API web protegidas por la plataforma Microsoft identity. Para ello, puede aprovechar las API web de la **On-Behalf-Of** flujo, lo que permite que la API web intercambiar un token de acceso entrante por otro token de acceso que se usará en las solicitudes salientes. Para obtener más información, consulte [plataforma de identidad de Microsoft y flujo en nombre de OAuth 2.0](v2-oauth2-on-behalf-of-flow.md).

## <a name="mobile-and-native-apps"></a>Aplicaciones móviles y nativas

Las aplicaciones instaladas en un dispositivo, como las aplicaciones móviles y de escritorio, suelen necesitar el acceso a servicios back-end o a las API web que almacenan datos y realizan varias funciones en nombre del usuario. Estas aplicaciones pueden agregar el inicio de sesión y la autorización a los servicios back-end mediante el [flujo de código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md).

En este flujo, la aplicación recibe un código de autorización desde el punto de conexión de plataforma de identidad de Microsoft cuando el usuario inicia sesión. El código de autorización representa el permiso de la aplicación para llamar a servicios de back-end en nombre del usuario que inició la sesión. La aplicación podrá intercambiar el código de autorización en segundo plano para un token de acceso de OAuth 2.0 y un token de actualización. La aplicación puede usar el token de acceso para autenticar las API web en las solicitudes HTTP y utilizar el token de actualización para obtener nuevos tokens de acceso cuando expiren los antiguos.

![Flujo de autenticación de la aplicación nativa](./media/v2-app-types/convergence-scenarios-native.svg)

## <a name="daemons-and-server-side-apps"></a>Demonios y aplicaciones de servidor

Las aplicaciones que contienen procesos de larga duración o que funcionan sin la interacción con un usuario también necesitan un modo de acceder a recursos protegidos, como las API web. Estas aplicaciones pueden autenticarse y obtener tokens mediante la identidad de la aplicación, en lugar de una identidad delegada del usuario, con el flujo de credenciales de cliente de OAuth 2.0. Puede demostrar la identidad de la aplicación mediante un certificado o secreto de cliente. Para obtener más información, consulte [autenticarse en la plataforma de identidad de Microsoft en aplicaciones de demonio con certificados](https://azure.microsoft.com/resources/samples/active-directory-dotnet-daemon-certificate-credential/).

En este flujo, la aplicación interactúa directamente con el `/token` extremo para obtener acceso:

![Flujo de autenticación de aplicación de demonio](./media/v2-app-types/convergence-scenarios-daemon.svg)

Para compilar una aplicación demonio, consulte la [documentación de credenciales de cliente](v2-oauth2-client-creds-grant-flow.md) en nuestra sección de introducción o pruebe una [aplicación de ejemplo de .NET](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2).
