---
title: Consideraciones sobre Xamarin iOS (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Conozca las consideraciones para usar Xamarin iOS con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/16/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 73303d64d7ebeb94290819226e5e5944f95f6e42
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165709"
---
# <a name="considerations-for-using-xamarin-ios-with-msalnet"></a>Consideraciones para usar Xamarin iOS con MSAL.NET
Cuando use la biblioteca de autenticación de Microsoft para .NET (MSAL.NET) en Xamarin iOS, debe: 

- Reemplazar e implementar la función `OpenUrl` en `AppDelegate`.
- Habilitar grupos de cadenas de claves.
- Habilitar el uso compartido de la caché de tokens.
- Habilitar el acceso a cadenas de claves.
- Entender los problemas conocidos de iOS 12 y la autenticación.

## <a name="implement-openurl"></a>Implementación de OpenUrl

Reemplace el método `OpenUrl` de la clase derivada `FormsApplicationDelegate` y llame a `AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs`. Este es un ejemplo:

```csharp
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

Además, realice las siguientes tareas: 
* Defina un esquema URL.
* Exija permisos para que la aplicación llame a otra aplicación.
* Disponga de un formato específico para la URL de redireccionamiento.
* Registre la URL de redireccionamiento en [Azure Portal](https://portal.azure.com).

### <a name="enable-keychain-access"></a>Habilitación de acceso a la cadena de claves

Para habilitar el acceso a cadenas de claves, asegúrese de que la aplicación tenga un grupo de acceso a cadenas de claves. Puede establecer el grupo de acceso a cadenas de claves cuando cree la aplicación mediante la API `WithIosKeychainSecurityGroup()`.

Para beneficiarse de la caché y del inicio de sesión único (SSO), establezca el grupo de acceso a cadenas de claves en el mismo valor en todas las aplicaciones.

En este ejemplo de configuración se usa MSAL 4.x:
```csharp
var builder = PublicClientApplicationBuilder
     .Create(ClientId)
     .WithIosKeychainSecurityGroup("com.microsoft.adalcache")
     .Build();
```

Habilite también el acceso a cadenas de claves en el archivo `Entitlements.plist`. Use el siguiente grupo de acceso o el suyo propio.

```xml
<dict>
  <key>keychain-access-groups</key>
  <array>
    <string>$(AppIdentifierPrefix)com.microsoft.adalcache</string>
  </array>
</dict>
```

Cuando se usa la API `WithIosKeychainSecurityGroup()`, MSAL anexa automáticamente el grupo de seguridad al final del *identificador de equipo* (`AppIdentifierPrefix`) de la aplicación. MSAL agrega el grupo de seguridad porque, al compilar la aplicación en Xcode, va a hacer lo mismo. Por eso, los derechos del archivo `Entitlements.plist` deben incluir `$(AppIdentifierPrefix)` antes del grupo de acceso a cadenas de claves.

Para obtener más información, vea la [documentación sobre derechos de iOS](https://developer.apple.com/documentation/security/keychain_services/keychain_items/sharing_access_to_keychain_items_among_a_collection_of_apps). 

### <a name="enable-token-cache-sharing-across-ios-applications"></a>Habilitación del uso compartido de la caché de tokens entre aplicaciones de iOS

A partir de MSAL 2.x, puede especificar un grupo de acceso a cadenas de claves para conservar la caché de tokens en varias aplicaciones. Este valor permite compartir la caché de tokens entre varias aplicaciones con el mismo grupo de acceso a cadenas de claves. Puede compartir la caché de tokens entre aplicaciones de [ADAL.NET](https://aka.ms/adal-net), aplicaciones de Xamarin.iOS para MSAL.NET y aplicaciones nativas de iOS desarrolladas en [ADAL.objc](https://github.com/AzureAD/azure-activedirectory-library-for-objc) o [MSAL.objc](https://github.com/AzureAD/microsoft-authentication-library-for-objc).

Al compartir la caché de tokens, se permite el inicio de sesión único (SSO) entre todas las aplicaciones que usan el mismo grupo de acceso a cadenas de claves.

Para habilitar este uso compartido de la caché, use el método `WithIosKeychainSecurityGroup()` para establecer el grupo de acceso a cadenas de claves en el mismo valor en todas las aplicaciones que comparten la misma caché. En el primer ejemplo de código de este artículo se muestra cómo usar el método.

Anteriormente en este artículo ha aprendido que MSAL agrega `$(AppIdentifierPrefix)` siempre que se usa la API `WithIosKeychainSecurityGroup()`. MSAL agrega este elemento porque el identificador de equipo `AppIdentifierPrefix` garantiza que solo las aplicaciones creadas por el mismo publicador puedan compartir el acceso a cadenas de claves.

> [!NOTE]
> La propiedad `KeychainSecurityGroup` está en desuso.
> 
> A partir de MSAL 2.x, los desarrolladores se veían obligados a incluir el prefijo `TeamId` al usar la propiedad `KeychainSecurityGroup`. Pero a partir de MSAL 2.7.x, cuando se usa la nueva propiedad `iOSKeychainSecurityGroup`, MSAL resuelve el prefijo `TeamId` durante el tiempo de ejecución. Al usar esta propiedad, no incluya el prefijo `TeamId` en el valor. El prefijo no es necesario.
>
> Como la propiedad `KeychainSecurityGroup` está obsoleta, use la propiedad `iOSKeychainSecurityGroup`.

### <a name="use-microsoft-authenticator"></a>Uso de Microsoft Authenticator

La aplicación puede usar Microsoft Authenticator como agente para habilitar:

- **SSO**: al habilitar el inicio de sesión único, los usuarios no necesitan iniciar sesión en cada aplicación.
- **Identificación del dispositivo**: use la identificación del dispositivo para autenticar mediante el acceso al certificado del dispositivo. Este certificado se crea en el dispositivo cuando se une al área de trabajo. La aplicación está lista si los administradores de inquilinos habilitan el acceso condicional relacionado con los dispositivos.
- **Comprobación de identificación de la aplicación**: cuando una aplicación llama al agente, pasa su URL de redireccionamiento. El agente comprueba la URL de redireccionamiento.

Para obtener detalles sobre cómo habilitar un agente, vea [Uso de Microsoft Authenticator o el Portal de empresa de Microsoft Intune en aplicaciones de Android y Xamarin iOS](msal-net-use-brokers-with-xamarin-apps.md).

## <a name="known-issues-with-ios-12-and-authentication"></a>Problemas conocidos con iOS 12 y la autenticación
Microsoft ha publicado un [aviso de seguridad](https://github.com/aspnet/AspNetCore/issues/4647) sobre una incompatibilidad entre iOS 12 y algunos tipos de autenticación. La incompatibilidad afecta a los inicios de sesión en redes sociales, WSFed y OIDC. El aviso de seguridad ayuda a los desarrolladores a comprender cómo quitar las restricciones de seguridad de ASP.NET de sus aplicaciones para que sean compatibles con iOS 12.  

Al desarrollar aplicaciones MSAL.NET en Xamarin iOS, puede experimentar un bucle infinito al intentar iniciar sesión en sitios web desde iOS 12. Este comportamiento es similar a este [problema de ADAL](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/issues/1329). 

También puede observar una incidencia en la autenticación en OIDC de ASP.NET Core con Safari para iOS 12. Para obtener más información, vea este [problema de WebKit](https://bugs.webkit.org/show_bug.cgi?id=188165).

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre las propiedades de Xamarin iOS, vea el párrafo [Consideraciones específicas de iOS](https://github.com/Azure-Samples/active-directory-xamarin-native-v2/tree/master/1-Basic#ios-specific-considerations) del archivo README.md del ejemplo siguiente:

Muestra | Plataforma | Descripción
------ | -------- | -----------
[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) | Xamarin iOS, Android, Plataforma universal de Windows (UWP) | Una aplicación sencilla de Xamarin Forms que muestra cómo usar MSAL para autenticar cuentas personales de Microsoft y Azure AD a través del punto de conexión de Azure AD 2.0. La aplicación también muestra cómo usar el token resultante para acceder a Microsoft Graph.

<!--- https://github.com/Azure-Samples/active-directory-xamarin-native-v2/blob/master/ReadmeFiles/Topology.png -->