---
title: Guía de migración de ADAL a a MSAL (MSAL4j) | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo migrar la aplicación de Java de la Biblioteca de autenticación de Azure Active Directory (ADAL) a la Biblioteca de autenticación de Microsoft (MSAL).
services: active-directory
author: sangonzal
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.tgt_pltfrm: Java
ms.workload: identity
ms.date: 11/04/2019
ms.author: sagonzal
ms.reviewer: nacanuma, twhitney
ms.custom: aaddev, devx-track-java
ms.openlocfilehash: ec4103251d27114b8fe40101c0e78c259106a440
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120888"
---
# <a name="adal-to-msal-migration-guide-for-java"></a>Guía de migración de ADAL a MSAL para Java

En este artículo se resaltan los cambios que debe realizar para migrar una aplicación que usa la biblioteca de autenticación de Azure Active Directory (ADAL) para que use la biblioteca de autenticación de Microsoft (MSAL).

Tanto la Biblioteca de autenticación de Microsoft para Java (MSAL4J) como la Biblioteca de autenticación de Azure AD para Java (ADAL4J) se usan para autenticar entidades de Azure AD y solicitar tokens a Azure AD. Hasta ahora, la mayoría de los desarrolladores ha usado la plataforma Azure AD para desarrolladores (v1.0) para autenticar identidades de Azure AD (cuentas profesionales y educativas) mediante la solicitud de tokens con la Biblioteca de autenticación de Azure AD (ADAL).

MSAL ofrece las siguientes ventajas:

- Dado que usa el punto de conexión de la Plataforma de identidad de Microsoft más reciente, puede autenticar un conjunto más amplio de identidades de Microsoft, como identidades de Azure AD, cuentas de Microsoft y cuentas sociales y locales mediante Azure AD Negocio a consumidor (B2C).
- Los usuarios obtendrán la mejor experiencia de inicio de sesión único.
- La aplicación puede habilitar el consentimiento incremental y es más fácil admitir el acceso condicional.

MSAL para Java es la biblioteca de autenticación que se recomienda usar con la Plataforma de identidad de Microsoft. No se implementarán nuevas características en ADAL4J. Todos los trabajos se centran en mejorar MSAL.

## <a name="differences"></a>Diferencias

Si ya ha trabajado con el punto de conexión de Azure AD para desarrolladores (v1.0) (y ADAL4J), puede que desee leer [Motivos para actualizar a la Plataforma de identidad de Microsoft (v2.0)](../azuread-dev/azure-ad-endpoint-comparison.md).

## <a name="scopes-not-resources"></a>Ámbitos, no recursos

ADAL4J adquiere tokens para los recursos, en tanto que MSAL para Java adquiere tokens para los ámbitos. Muchas clases de MSAL para Java requieren un parámetro de ámbito. Este parámetro es una lista de cadenas que declaran los permisos y recursos deseados que se solicitan. Consulte [Ámbitos de Microsoft Graph](/graph/permissions-reference) para ver ámbitos de ejemplo.

Puede agregar el sufijo del ámbito `/.default` al recurso para ayudar a migrar las aplicaciones del punto de conexión v1.0 (ADAL) al punto de conexión de la plataforma de identidad de Microsoft (MSAL). Por ejemplo, el valor de recurso de `https://graph.microsoft.com`, tiene como valor de ámbito equivalente `https://graph.microsoft.com/.default`.  Si el recurso no está en el formato de dirección URL, sino en el del identificador de recurso `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX`, todavía puede usar el valor de ámbito como `XXXXXXXX-XXXX-XXXX-XXXXXXXXXXXX/.default`.

Para más información sobre los diferentes tipos de ámbitos, consulte los artículos [Permisos y consentimiento en la plataforma de identidad de Microsoft](./v2-permissions-and-consent.md) y [Ámbitos para una API web que acepta tokens de la versión 1.0](./msal-v1-app-scopes.md).

## <a name="core-classes"></a>Clases principales

En ADAL4J, la clase `AuthenticationContext` representa la conexión con el servicio de token de seguridad (STS) o el servidor de autorización, mediante una autoridad. Aunque MSAL para Java está diseñado en torno a las aplicaciones cliente. Proporciona dos clases independientes: `PublicClientApplication` y `ConfidentialClientApplication`, para representar las aplicaciones cliente.  La última, `ConfidentialClientApplication`, representa una aplicación que está diseñada para mantener de forma segura un secreto, como un identificador de aplicación para una aplicación tipo demonio.

En la tabla siguiente se muestra cómo se asignan las funciones de ADAL4J a las nuevas funciones de MSAL para Java:

| Método de ADAL4J| Método de MSAL4J|
|------|-------|
|acquireToken(String resource, ClientCredential credential, AuthenticationCallback callback) | acquireToken(ClientCredentialParameters)|
|acquireToken(String resource, ClientAssertion assertion, AuthenticationCallback callback)|acquireToken(ClientCredentialParameters)|
|acquireToken(String resource, AsymmetricKeyCredential credential, AuthenticationCallback callback)|acquireToken(ClientCredentialParameters)|
|acquireToken(String resource, String clientId, String username, String password, AuthenticationCallback callback)| acquireToken(UsernamePasswordParameters)|
|acquireToken(String resource, String clientId, String username, String password=null, AuthenticationCallback callback)|acquireToken(IntegratedWindowsAuthenticationParameters)|
|acquireToken(String resource, UserAssertion userAssertion, ClientCredential credential, AuthenticationCallback callback)| acquireToken(OnBehalfOfParameters)|
|acquireTokenByAuthorizationCode() | acquireToken(AuthorizationCodeParameters) |
| acquireDeviceCode() y acquireTokenByDeviceCode()| acquireToken(DeviceCodeParameters)|
|acquireTokenByRefreshToken()| acquireTokenSilently(SilentParameters)|

## <a name="iaccount-instead-of-iuser"></a>IAccount en lugar de IUser

Usuarios manipulados de ADAL4J. Aunque un usuario representa un solo agente humano o software, puede tener una o más cuentas en el sistema de identidades de Microsoft. Por ejemplo, un usuario puede tener varias cuentas de Azure AD, de Azure AD B2C o personales de Microsoft.

MSAL para Java define el concepto de cuenta mediante la interfaz `IAccount`. Se trata de un cambio importante con respecto a ADAL4J, pero es una buena idea porque captura el hecho de que el mismo usuario puede tener varias cuentas y quizás incluso en diferentes directorios de Azure AD. MSAL para Java proporciona mejor información en escenarios de invitado, ya que se ofrece información de la cuenta de inicio.

## <a name="cache-persistence"></a>Persistencia en la caché

ADAL4J no admitía la memoria caché de tokens.
MSAL para Java agrega una [memoria caché de tokens](msal-acquire-cache-tokens.md) para simplificar la administración de la duración de los tokens mediante la actualización automática de los tokens expirados cuando sea posible y la prevención de solicitudes innecesarias para que el usuario proporcione credenciales cuando sea posible.

## <a name="common-authority"></a>Autoridad común

En la versión 1.0, si usa la autoridad `https://login.microsoftonline.com/common`, los usuarios pueden iniciar sesión con cualquier cuenta de Azure Active Directory (AAD) (para cualquier organización).

Si usa la autoridad `https://login.microsoftonline.com/common` en la versión 2.0, los usuarios pueden iniciar sesión con cualquier organización de AAD o incluso con una cuenta personal de Microsoft (MSA). En MSAL para Java, si quiere restringir el inicio de sesión a cualquier cuenta de AAD, ha de usar la autoridad `https://login.microsoftonline.com/organizations` (que tiene el mismo comportamiento que con ADAL4J). Para especificar una autoridad, establezca el parámetro `authority` en el método [PublicClientApplication.Builder](https://javadoc.io/doc/com.microsoft.azure/msal4j/1.0.0/com/microsoft/aad/msal4j/PublicClientApplication.Builder.html) al crear la clase de `PublicClientApplication`.

## <a name="v10-and-v20-tokens"></a>Tokens de las versiones 1.0 y 2.0

El punto de conexión de la versión 1.0 (usada por ADAL) solo emite tokens de la versión 1.0.

El punto de conexión de la versión 2.0 (usado por MSAL) puede emitir tokens de las versiones 1.0 y 2.0. Una propiedad del manifiesto de aplicación de la API web permite a los desarrolladores elegir la versión del token que se acepta. Consulte `accessTokenAcceptedVersion` en la documentación de referencia del [manifiesto de aplicación](./reference-app-manifest.md).

Para más información acerca de los tokens de las versiones 1.0 y 2.0, consulte [Tokens de acceso de Azure Active Directory](./access-tokens.md).

## <a name="adal-to-msal-migration"></a>Migración de ADAL a MSAL

En ADAL4J, se exponían los tokens de actualización, lo que permitía a los desarrolladores almacenarlos en memoria caché. A continuación, usarán `AcquireTokenByRefreshToken()` para habilitar soluciones como la implementación de servicios de ejecución prolongada que actualizan los paneles en nombre del usuario cuando el usuario ya no está conectado.

Por motivos de seguridad, MSAL para Java no expone tokens de actualización. En su lugar, MSAL controla la actualización de tokens automáticamente.

MSAL para Java tiene una API que le permite migrar los tokens de actualización adquiridos con ADAL4J a la aplicación cliente: [acquireToken(RefreshTokenParameters)](https://javadoc.io/static/com.microsoft.azure/msal4j/1.0.0/com/microsoft/aad/msal4j/PublicClientApplication.html#acquireToken-com.microsoft.aad.msal4j.RefreshTokenParameters-). Con este método puede proporcionar el token de actualización que ha usado anteriormente junto con todos los ámbitos (recursos) que desee. El token de actualización se intercambiará por otro nuevo y se almacenará en memoria caché para su uso por parte de la aplicación.

En el fragmento de código siguiente se muestra código de migración en una aplicación cliente confidencial.

```java
String rt = GetCachedRefreshTokenForSIgnedInUser(); // Get refresh token from where you have them stored
Set<String> scopes = Collections.singleton("SCOPE_FOR_REFRESH_TOKEN");

RefreshTokenParameters parameters = RefreshTokenParameters.builder(scopes, rt).build();

PublicClientApplication app = PublicClientApplication.builder(CLIENT_ID) // ClientId for your application
                .authority(AUTHORITY)  //plug in your authority
                .build();

IAuthenticationResult result = app.acquireToken(parameters);
```

`IAuthenticationResult` devuelve un token de acceso y un token de identificador, mientras que el nuevo token de actualización se almacena en la memoria caché.
La aplicación también contendrá ahora un elemento IAccount:

```java
Set<IAccount> accounts =  app.getAccounts().join();
```

Para usar los tokens que se encuentran ahora en la memoria caché, llame a:

```java
SilentParameters parameters = SilentParameters.builder(scope, accounts.iterator().next()).build();
IAuthenticationResult result = app.acquireToken(parameters);
```