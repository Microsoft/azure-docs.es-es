---
title: Consideraciones de iOS de Xamarin (biblioteca de autenticación de Microsoft para. NET) | Azure
description: Obtenga información sobre las consideraciones específicas al utilizar Xamarin iOS con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/24/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: bf236bff2300129ec97d3b8946c4c2a2748bca77
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65602134"
---
# <a name="xamarin-ios-specific-considerations-with-msalnet"></a>Consideraciones de específica de iOS de Xamarin con MSAL.NET
En Xamarin iOS, hay varias consideraciones que debe tener en cuenta al usar MSAL.NET

- [Problemas conocidos con iOS 12 y autenticación](#known-issues-with-ios-12-and-authentication)
- [Invalide e implemente el `OpenUrl` funcionando en el `AppDelegate`](#implement-openurl)
- [Habilitar los grupos de la cadena de claves](#enable-keychain-groups)
- [Habilitar el uso compartido de caché de tokens](#enable-token-cache-sharing-across-ios-applications)
- [Habilitar el acceso a llaves](#enable-keychain-access)

## <a name="known-issues-with-ios-12-and-authentication"></a>Problemas conocidos con iOS 12 y autenticación
Microsoft ha lanzado un [documento informativo sobre seguridad](https://github.com/aspnet/AspNetCore/issues/4647) para proporcionar información sobre una incompatibilidad entre iOS12 y algunos tipos de autenticación. La incompatibilidad saltos social, WSFed y OIDC inicios de sesión. En este aviso también proporciona instrucciones sobre lo que los desarrolladores pueden hacer para quitar las restricciones de seguridad actual agregadas por ASP.NET para sus aplicaciones para ser compatible con iOS12.  

Al desarrollar aplicaciones MSAL.NET en Xamarin iOS, puede ver un bucle infinito al intentar iniciar sesión sitios Web desde iOS 12 (similar al siguiente [ADAL problema](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/issues/1329). 

También puede ver una interrupción en la autenticación de ASP.NET Core OIDC con 12 iOS Safari tal como se describe en este [WebKit problema](https://bugs.webkit.org/show_bug.cgi?id=188165).

## <a name="implement-openurl"></a>Implementar OpenUrl

En primer lugar debe invalidar el `OpenUrl` método de la `FormsApplicationDelegate` derivados de la clase y llamar a `AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs`.

```csharp
public override bool OpenUrl(UIApplication app, NSUrl url, NSDictionary options)
{
    AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url);
    return true;
}
```

También deberá definir un esquema de dirección URL, requieren permisos para la aplicación a llamar a otra aplicación, tiene un formulario específico para la dirección URL de redireccionamiento y registrar esta dirección URL de redireccionamiento en el [portal Azure](https://portal.azure.com).

## <a name="enable-keychain-groups"></a>Habilitar los grupos de la cadena de claves

Para hacer que el token de trabajo de caché y tiene el `AcquireTokenSilentAsync` trabajo de método, se deben seguir varios pasos:
1. Habilitar el acceso a llaves en su *`* Entitlements.plist* archivo y especifique el **Keychain Groups** en el identificador del conjunto.
2. Seleccione *`*Entitlements.plist*`* de archivos en el **derechos personalizados** campo en la ventana Opciones de iOS y proyecto **vista de firma de lote**.
3. Cuando un certificado de firma, asegúrese de que XCode usa el mismo identificador de Apple.

## <a name="enable-token-cache-sharing-across-ios-applications"></a>Habilitar caché de tokens compartir entre aplicaciones iOS

A partir de MSAL 2.x, puede especificar un grupo de seguridad de la cadena de claves a utilizar para conservar la caché de tokens en varias aplicaciones. Esto le permite compartir la caché de tokens entre varias aplicaciones, tener el mismo grupo de seguridad de cadena de claves incluidas aquellas que se desarrollaron con [ADAL.NET](https://aka.ms/adal-net), Xamarin.iOS MSAL.NET y aplicaciones nativas para iOS desarrolladas con [ADAL.objc](https://github.com/AzureAD/azure-activedirectory-library-for-objc) o [MSAL.objc](https://github.com/AzureAD/microsoft-authentication-library-for-objc)).

Uso compartido de la caché de tokens permite el inicio de sesión único (SSO) entre todas las aplicaciones que usan el mismo grupo de seguridad de la cadena de claves.

Para habilitar el inicio de sesión único, deberá establecer el `PublicClientApplication.iOSKeychainSecurityGroup` propiedad en el mismo valor en todas las aplicaciones.

Un ejemplo de esto con MSAL v3.x sería:
```csharp
var builder = PublicClientApplicationBuilder
     .Create(ClientId)
     .WithIosKeychainSecurityGroup("com.microsoft.msalrocks")
     .Build();
```

Un ejemplo de esto con MSAL v2.7.x sería:

```csharp
PublicClientApplication.iOSKeychainSecurityGroup = "com.microsoft.msalrocks";
```

> [!NOTE]
> El `KeychainSecurityGroup` propiedad está desusada. Anteriormente, en MSAL 2.x, los desarrolladores se veían obligados a incluir el prefijo de TeamId cuando se usa el `KeychainSecurityGroup` propiedad. 
> 
> Ahora, a partir de MSAL 2.7. x, MSAL se resolverá el prefijo TeamId en tiempo de ejecución cuando se usa el `iOSKeychainSecurityGroup` propiedad. Cuando se usa esta propiedad, el valor no debe contener el prefijo TeamId. 
> 
> Use la nueva `iOSKeychainSecurityGroup` propiedad, que no requiere que los desarrolladores proporcionar el TeamId. El `KeychainSecurityGroup` propiedad ahora está obsoleta. 

## <a name="enable-keychain-access"></a>Habilitar el acceso a llaves

En MSAL 2.x y 4.x ADAL, el TeamId se usa para tener acceso a la cadena de claves, lo que permite a las bibliotecas de autenticación proporcionar inicio de sesión-único (SSO) entre las aplicaciones del mismo publicador. 

¿Qué es el [TeamIdentifierPrefix](/xamarin/ios/deploy-test/provisioning/entitlements?tabs=vsmac) (TeamId)? Es un identificador único (empresa o personal) en la aplicación de Store. El valor de AppId es único para una aplicación. Si tiene más de una aplicación, el TeamId para todas las aplicaciones será el mismo, pero el valor de AppId será diferente. El grupo de acceso de la cadena de claves es el prefijo TeamId automáticamente para cada grupo por el sistema. Es cómo el sistema operativo exige que las aplicaciones del mismo editor pueden tener acceso a la cadena de claves compartida. 

Al inicializar el `PublicClientApplication`, si recibe un `MsalClientException` con el mensaje: `TeamId returned null from the iOS keychain...`, deberá hacer lo siguiente en la aplicación de Xamarin iOS:

1. En Visual Studio, en la pestaña de depuración, vaya a nameOfMyApp.iOS propiedades...
2. A continuación, vaya a la firma del lote de iOS 
3. En los derechos personalizados, haga clic en el... y seleccione el archivo Entitlements.plist de la aplicación
4. En el archivo csproj de la aplicación de iOS, debe tener esta línea ahora incluye: `<CodesignEntitlements>Entitlements.plist</CodesignEntitlements>`
5. **Volver a generar** el proyecto.

Se trata de *además* para habilitar el acceso a llaves en el `Entitlements.plist` de archivos, ya sea mediante la por debajo del grupo de acceso o el suyo propio:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>keychain-access-groups</key>
  <array>
    <string>$(AppIdentifierPrefix)com.microsoft.adalcache</string>
  </array>
</dict>
</plist>
```

## <a name="next-steps"></a>Pasos siguientes

Se proporcionan más detalles en el [iOS consideraciones específicas](https://github.com/azure-samples/active-directory-xamarin-native-v2#ios-specific-considerations) párrafo del archivo del archivo readme.md del ejemplo siguiente:

Muestra | Plataforma | DESCRIPCIÓN 
------ | -------- | -----------
[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) | Xamarin iOS, Android, UWP | Una aplicación de Xamarin Forms sencilla que muestra cómo usar MSAL para autenticar MSA y Azure AD a través del punto de conexión V2.0 de AAD y tener acceso a Microsoft Graph con el token resultante. <br>![Topología](media/msal-net-xamarin-ios-considerations/topology.png)