---
title: Control de los cambios de cookies de SameSite en el explorador Chrome | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo controlar los cambios de cookies de SameSite en el explorador Chrome.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: how-to
ms.date: 01/27/2020
ms.author: jmprieur
ms.reviewer: kkrishna
ms.custom: aaddev
ms.openlocfilehash: 5742ddf9553c3ac9187dbef93fc7927564cbc095
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88116978"
---
# <a name="handle-samesite-cookie-changes-in-chrome-browser"></a>Control de los cambios de cookies de SameSite en el explorador Chrome

## <a name="what-is-samesite"></a>¿Qué es SameSite?

`SameSite` es una propiedad que se puede establecer en las cookies HTTP para evitar los ataques de falsificación de solicitud entre sitios (CSRF) en las aplicaciones web:

- Cuando `SameSite` se establece en **Lax**, la cookie se envía en las solicitudes del mismo sitio y en las solicitudes GET de otros sitios. No se envía en las solicitudes GET entre dominios.
- Un valor **Strict** garantiza que la cookie se envíe solo en las solicitudes dentro del mismo sitio.

De forma predeterminada, el valor de la propiedad `SameSite` no está establecido en los exploradores. Por ese motivo, no hay restricciones al enviar cookies en las solicitudes. Una aplicación deberá habilitar la protección CSRF al establecer el valor en **Lax** o **Strict** según sus requisitos.

## <a name="samesite-changes-and-impact-on-authentication"></a>Cambios de SameSite y su efecto en la autenticación

[Actualizaciones recientes de los estándares de SameSite](https://tools.ietf.org/html/draft-west-cookie-incrementalism-00) proponen proteger las aplicaciones al configurar `SameSite` como Lax de forma predeterminada cuando no se haya establecido ningún valor. Esta mitigación significa que las cookies se restringirán en las solicitudes HTTP (a excepción de las solicitudes GET) que se realicen desde otros sitios. Además, se introduce un valor **None** para quitar las restricciones de las cookies que se envían. Estas actualizaciones se publicarán pronto en una versión próxima del explorador Chrome.

Cuando las aplicaciones web se autentican con la plataforma de identidad de Microsoft mediante el modo de respuesta "form_post", el servidor de inicio de sesión responde a estas con una solicitud HTTP POST para enviar los tokens o el código de autenticación. Dado que esta es una solicitud entre dominios (de `login.microsoftonline.com` a su dominio; por ejemplo, `https://contoso.com/auth`), las cookies que se establecieron en la aplicación ahora se rigen por las nuevas reglas de Chrome. Las cookies que deben usarse en escenarios entre sitios contienen los valores *state* y *nonce*, que también se envían en la solicitud de inicio de sesión. Azure AD genera otras cookies para almacenar la sesión.

Si no actualiza las aplicaciones web, este nuevo comportamiento generará errores de autenticación.

## <a name="mitigation-and-samples"></a>Mitigación y ejemplos

Para solucionar los errores de autenticación, las aplicaciones web que se autentican con la plataforma de identidad de Microsoft pueden establecer la propiedad `SameSite` en `None` para las cookies que se usan en escenarios entre dominios cuando dichas aplicaciones se ejecuten en el explorador Chrome.
Otros exploradores (consulte [este vínculo](https://www.chromium.org/updates/same-site/incompatible-clients) para obtener una lista completa) siguen el comportamiento anterior de `SameSite` y no incluirán las cookies si `SameSite=None`.
Por eso, para admitir la autenticación en varios exploradores, las aplicaciones web deben establecer el valor `SameSite` en `None` solo en Chrome y dejarlo vacío en otros exploradores.

Esta estrategia se muestra en los siguientes ejemplos de código.

# <a name="net"></a>[.NET](#tab/dotnet)

En la tabla siguiente se muestran las solicitudes de incorporación de cambios que subsanaron los cambios de SameSite en nuestros ejemplos de ASP.NET y ASP.NET Core.

| Muestra | Solicitud de incorporación de cambios |
| ------ | ------------ |
|  [Tutorial incremental de la aplicación web de ASP.NET Core](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)  |  [Revisión de las cookies de SameSite n.º 261](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/pull/261)  |
|  [Ejemplo de una aplicación web MVC para ASP.NET](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect)  |  [Revisión de las cookies de SameSite n.º 35](https://github.com/Azure-Samples/ms-identity-aspnet-webapp-openidconnect/pull/35)  |
|  [active-directory-dotnet-admin-restricted-scopes-v2](https://github.com/azure-samples/active-directory-dotnet-admin-restricted-scopes-v2)  |  [Revisión de las cookies de SameSite n.º 28](https://github.com/Azure-Samples/active-directory-dotnet-admin-restricted-scopes-v2/pull/28)  |

Para obtener más información sobre cómo administrar las cookies de SameSite en ASP.NET y ASP.NET Core, consulte también:

- [Trabajo con cookies de SameSite en ASP.NET Core](/aspnet/core/security/samesite).
- [Problema de SameSite en el blog de ASP.NET](https://devblogs.microsoft.com/aspnet/upcoming-samesite-cookie-changes-in-asp-net-and-asp-net-core/)

# <a name="python"></a>[Python](#tab/python)

| Muestra |
| ------ |
|  [ms-identity-python-webapp](https://github.com/Azure-Samples/ms-identity-python-webapp)  |

# <a name="java"></a>[Java](#tab/java)

| Muestra | Solicitud de incorporación de cambios |
| ------ | ------------ |
|  [ms-identity-java-webapp](https://github.com/Azure-Samples/ms-identity-java-webapp)  | [Revisión de las cookies de SameSite n.º 24](https://github.com/Azure-Samples/ms-identity-java-webapp/pull/24)
|  [ms-identity-java-webapi](https://github.com/Azure-Samples/ms-identity-java-webapi)  | [Revisión de las cookies de SameSite n.º 4](https://github.com/Azure-Samples/ms-identity-java-webapi/pull/4)

---

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre SameSite y el escenario de una aplicación web:

> [!div class="nextstepaction"]
> [Preguntas más frecuentes de Google Chrome sobre SameSite](https://www.chromium.org/updates/same-site/faq)

> [!div class="nextstepaction"]
> [Página de SameSite de Chromium](https://www.chromium.org/updates/same-site)

> [!div class="nextstepaction"]
> [Escenario: Aplicación web que permite iniciar sesión a los usuarios](scenario-web-app-sign-user-overview.md)