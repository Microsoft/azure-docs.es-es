---
title: Guía de migración de ADAL a MSAL (MSAL para iOS/macOS) | Azure
titleSuffix: Microsoft identity platform
description: Conozca las diferencias entre MSAL para iOS o macOS y la Biblioteca de autenticación de Azure AD para ObjectiveC (ADAL.ObjC) y aprenda cómo migrar a MSAL para iOS y macOS.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/28/2019
ms.author: marsma
ms.reviewer: oldalton
ms.custom: aaddev
ms.openlocfilehash: 13923596b7ad0f6d3fdef24e847f469645b448ee
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119936"
---
# <a name="migrate-applications-to-msal-for-ios-and-macos"></a>Migración de aplicaciones a MSAL para iOS y macOS

La biblioteca de autenticación de Azure Active Directory ([ADAL Objective-C](https://github.com/AzureAD/azure-activedirectory-library-for-objc)) se creó para trabajar con cuentas de Azure Active Directory a través del punto de conexión v1.0.

La biblioteca de autenticación de Microsoft para iOS y macOS (MSAL) se ha creado para trabajar con todas las identidades de Microsoft, como cuentas de Azure Active Directory (Azure AD), cuentas de Microsoft personales y cuentas de Azure AD B2C mediante la Plataforma de identidad de Microsoft (oficialmente, el punto de conexión de Azure AD v2.0).

La Plataforma de identidad de Microsoft tiene algunas diferencias importantes con Azure Active Directory v1.0. En este artículo se resaltan estas diferencias y se proporcionan instrucciones para migrar una aplicación de ADAL a MSAL.

## <a name="adal-and-msal-app-capability-differences"></a>Diferencias de funcionalidad entre las aplicaciones de ADAL y MSAL

### <a name="who-can-sign-in"></a>Quién puede iniciar sesión

* ADAL solo admite cuentas profesionales y educativas, también conocidas como cuentas de Azure AD.
* MSAL admite cuentas personales de Microsoft (cuentas de MSA) como Hotmail.com, Outlook.com y Live.com.
* MSAL admite cuentas profesionales y educativas, y cuentas de Azure AD B2C.

### <a name="standards-compliance"></a>Cumplimiento normativo

* El punto de conexión de la Plataforma de identidad de Microsoft sigue las normas OAuth 2.0 y OpenId Connect.

### <a name="incremental-and-dynamic-consent"></a>Consentimiento incremental y dinámico

* El punto de conexión de Azure Active Directory v1.0 requiere que todos los permisos se declaren por adelantado durante el registro de la aplicación. Esto significa que esos permisos son estáticos.
* La Plataforma de identidad de Microsoft le permite solicitar permisos de forma dinámica. Las aplicaciones pueden pedir permisos solo según sea necesario y solicitar más cuando la aplicación los necesite.

Para más información sobre las diferencias entre Azure Active Directory v1.0 y la Plataforma de identidad de Microsoft, consulte [Motivos para actualizar a la Plataforma de identidad de Microsoft (v2.0)](../azuread-dev/azure-ad-endpoint-comparison.md).

## <a name="adal-and-msal-library-differences"></a>Diferencias entre las bibliotecas ADAL y MSAL

La API pública de MSAL refleja algunas diferencias clave entre Azure AD v1.0 y la Plataforma de identidad de Microsoft.

### <a name="msalpublicclientapplication-instead-of-adauthenticationcontext"></a>MSALPublicClientApplication en lugar de ADAuthenticationContext

`ADAuthenticationContext` es el primer objeto que una aplicación de ADAL crea. Representa la creación de una instancia de ADAL. Las aplicaciones crean una nueva instancia de `ADAuthenticationContext` para cada combinación de nube e inquilino (entidad) de Azure Active Directory. Se puede usar el mismo `ADAuthenticationContext` para obtener tokens para varias aplicaciones de cliente públicas.

En MSAL, la interacción principal se realiza a través de un objeto `MSALPublicClientApplication`, que se modela según el [cliente público de OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-2.1). Se puede usar una instancia de `MSALPublicClientApplication` para interactuar con varias nubes de AAD, e inquilinos, sin necesidad de crear una nueva instancia para cada entidad. Para la mayoría de las aplicaciones, una instancia `MSALPublicClientApplication` es suficiente.

### <a name="scopes-instead-of-resources"></a>Ámbitos en lugar de recursos

En ADAL, una aplicación tenía que proporcionar un identificador *resource*, como `https://graph.microsoft.com`, para adquirir los tokens del punto de conexión de Azure Active Directory v1.0. Un recurso puede definir varios ámbitos, o oAuth2Permissions en el manifiesto de la aplicación, que comprende. Esto permitía que las aplicaciones cliente solicitaran tokens de ese recurso para un determinado conjunto de ámbitos predefinidos durante el registro de la aplicación.

En MSAL, en lugar de un identificador de recurso único, las aplicaciones proporcionan un conjunto de ámbitos por solicitud. Un ámbito es un identificador de recurso seguido de un nombre de permiso con el formato recurso/permiso. Por ejemplo: `https://graph.microsoft.com/user.read`

Hay dos maneras de proporcionar ámbitos en MSAL:

* Proporcionar una lista de todos los permisos que la aplicación necesita. Por ejemplo: 

    `@[@"https://graph.microsoft.com/directory.read", @"https://graph.microsoft.com/directory.write"]`

    En este caso, la aplicación solicita los permisos `directory.read` y `directory.write`. Se le pedirá al usuario que dé su consentimiento para esos permisos, si no lo ha hecho antes para esta aplicación. La aplicación también podría recibir permisos adicionales para los que el usuario ya haya dado su consentimiento. Solo se le pedirá al usuario que dé su consentimiento para permisos nuevos o permisos que aún no se hayan concedido.

* El ámbito `/.default`.

Este es el ámbito integrado para todas las aplicaciones. Hace referencia a la lista estática de permisos configurados cuando la aplicación se registró. Su comportamiento es similar al de `resource`. Esto puede ser útil cuando se realiza la migración para asegurarse de que se mantiene un conjunto de ámbitos y una experiencia del usuario similares.

Para usar el ámbito `/.default`, anexe `/.default` al identificador de recurso. Por ejemplo: `https://graph.microsoft.com/.default`. Aunque el recurso termine con una barra diagonal (`/`), hay que anexar `/.default`, incluida la barra diagonal inicial, lo que da lugar a un ámbito que tiene una doble barra diagonal (`//`).

Puede leer más información sobre el uso del ámbito "/.default" [aquí](./v2-permissions-and-consent.md#the-default-scope).

### <a name="supporting-different-webview-types--browsers"></a>Compatibilidad con distintos tipos de vistas web y exploradores

ADAL solo es compatible con UIWebView/WKWebView para iOS y WebView para macOS. MSAL para iOS admite más opciones para mostrar el contenido web al solicitar un código de autorización y ya no admite `UIWebView`; esto puede mejorar la experiencia del usuario y la seguridad.

De forma predeterminada, MSAL en iOS usa [ASWebAuthenticationSession](https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession?language=objc), que es el componente web que Apple recomienda para la autenticación en dispositivos iOS 12 y versiones posteriores. Proporciona ventajas de inicio de sesión único (SSO) mediante el uso compartido de cookies entre aplicaciones y el explorador Safari.

Puede optar por usar un componente web diferente en función de los requisitos de la aplicación y la experiencia del usuario final que desee. Consulte los [tipos de vistas web admitidos](customize-webviews.md) para ver más opciones.

Al migrar de ADAL a MSAL, `WKWebView` proporciona una experiencia del usuario más parecida a ADAL en iOS y macOS. Se recomienda migrar a `ASWebAuthenticationSession` en iOS, si es posible. Para macOS, le recomendamos que use `WKWebView`.

### <a name="account-management-api-differences"></a>Diferencias entre las API de administración de cuentas

Cuando se llama a los métodos de ADAL `acquireToken()` o `acquireTokenSilent()`, se recibe un objeto `ADUserInformation` que contiene una lista de notificaciones del `id_token`, que representa la cuenta que se va a autenticar. Además, `ADUserInformation` devuelve `userId` basado en la notificación `upn`. Después de la adquisición del token interactivo inicial, ADAL espera que el desarrollador proporcione `userId` en todas las llamadas silenciosas.

ADAL no proporciona una API para recuperar identidades de usuario conocidas. Confía en la aplicación para guardar y administrar esas cuentas.

MSAL proporciona un conjunto de API para mostrar todas las cuentas conocidas para MSAL sin tener que adquirir un token.

Como ADAL, MSAL devuelve información de la cuenta que contiene una lista de notificaciones del `id_token`. Forma parte del objeto `MSALAccount` dentro del objeto `MSALResult`.

MSAL proporciona un conjunto de API para quitar cuentas, lo que permite que las cuentas quitadas sean inaccesibles para la aplicación. Después de quitar la cuenta, las llamadas de adquisición de token posteriores pedirán al usuario que realice una adquisición de tokens interactiva. La eliminación de cuentas solo se aplica a la aplicación cliente que la inició y no quita la cuenta de las otras aplicaciones que se ejecutan en el dispositivo o en el explorador del sistema. Esto garantiza que el usuario siga pudiendo utilizar SSO en el dispositivo incluso después de cerrar la sesión de una aplicación individual.

Además, MSAL también devuelve un identificador de cuenta que se puede usar para solicitar un token en modo silencioso más adelante. Sin embargo, el identificador de cuenta (accesible a través de la propiedad `identifier` del objeto `MSALAccount`) no se puede mostrar y no se puede asumir el formato en el que se encuentra, ni se intenta interpretar o analizar.

### <a name="migrating-the-account-cache"></a>Migración de la memoria caché de cuentas

Al migrar desde ADAL, las aplicaciones normalmente almacenan el `userId` de ADAL, que no tiene el valor de `identifier` que requiere MSAL. Como paso de migración único, una aplicación puede consultar una cuenta de MSAL mediante el userId de ADAL con la siguiente API:

`- (nullable MSALAccount *)accountForUsername:(nonnull NSString *)username error:(NSError * _Nullable __autoreleasing * _Nullable)error;`

Esta API lee las memorias caché de MSAL y ADAL para encontrar la cuenta mediante el userId de ADAL (UPN).

Si se encuentra la cuenta, el desarrollador debe utilizarla para realizar la adquisición de tokens silenciosa. La primera adquisición de tokens silenciosa actualizará la cuenta y el desarrollador obtendrá un identificador de cuenta compatible con MSAL en el resultado de MSAL (`identifier`). Después de eso, solo se debe usar `identifier` para buscar cuentas con la siguiente API:

`- (nullable MSALAccount *)accountForIdentifier:(nonnull NSString *)identifier error:(NSError * _Nullable __autoreleasing * _Nullable)error;`

Aunque es posible seguir usando el `userId` de ADAL para todas las operaciones de MSAL, como el `userId` se basa en el UPN, está sujeto a varias limitaciones que dan lugar a una mala experiencia del usuario. Por ejemplo, si el UPN cambia, el usuario tiene que iniciar sesión de nuevo. Se recomienda que todas las aplicaciones usen el valor `identifier` de la cuenta no visible para todas las operaciones.

Obtenga más información sobre la [migración del estado de caché](sso-between-adal-msal-apps-macos-ios.md).

### <a name="token-acquisition-changes"></a>Cambios en la adquisición de tokens

MSAL introduce algunos cambios en la llamada de adquisición de tokens:

* Al igual que ADAL, `acquireTokenSilent` siempre da como resultado una solicitud silenciosa.
* A diferencia de ADAL, `acquireToken` siempre da como resultado una interfaz de usuario que el usuario puede accionar mediante la vista web o la aplicación Microsoft Authenticator. En función del estado de SSO dentro de la vista web o Microsoft Authenticator, es posible que se le pida al usuario que escriba sus credenciales.
* En ADAL, `acquireToken` con `AD_PROMPT_AUTO` primero intenta la adquisición de tokens silenciosa y solo muestra la interfaz de usuario si se produce un error en la solicitud silenciosa. En MSAL, esta lógica se puede lograr llamando primero a `acquireTokenSilent`, y llamando a `acquireToken` solo si se produce un error en la adquisición silenciosa. Esto permite a los desarrolladores personalizar la experiencia del usuario antes de iniciar la adquisición de tokens interactiva.

### <a name="error-handling-differences"></a>Diferencias en el control de errores

MSAL proporciona más claridad entre los errores que puede controlar la aplicación y los que requieren la intervención del usuario. Hay un número limitado de errores que el desarrollador debe administrar:

* `MSALErrorInteractionRequired`: El usuario debe realizar una solicitud interactiva. Esto puede deberse a diversos motivos: la sesión de autenticación ha expirado, la directiva de acceso condicional ha cambiado, un token de actualización ha expirado o se ha revocado, no hay tokens válidos en la memoria caché, etc.
* `MSALErrorServerDeclinedScopes`: La solicitud no se completó totalmente y a algunos ámbitos no se les concedió acceso. Esto puede deberse a que un usuario declina el consentimiento en uno o más ámbitos.

El control de todos los demás errores de la [lista `MSALError`](https://github.com/AzureAD/microsoft-authentication-library-for-objc/blob/master/MSAL/src/public/MSALError.h#L128) es opcional. Puede usar la información de estos errores para mejorar la experiencia del usuario.

Consulte [Control de excepciones y errores con MSAL](msal-handling-exceptions.md) para más información sobre este tema.

### <a name="broker-support"></a>Compatibilidad con el agente

MSAL, a partir de la versión 0.3.0, proporciona compatibilidad con la autenticación con agente mediante la aplicación Microsoft Authenticator. Microsoft Authenticator también habilita la compatibilidad con escenarios de acceso condicional. Algunos ejemplos de escenarios de acceso condicional incluyen directivas de cumplimiento de dispositivos que requieren que el usuario inscriba el dispositivo mediante Intune o que se registre en AAD para obtener un token. Y las directivas de acceso condicional de administración de aplicaciones móviles (MAM), que requieren la prueba de cumplimiento antes de que la aplicación pueda obtener un token.

Para habilitar el agente en la aplicación:

1. Registre un formato de URI de redireccionamiento compatible con el agente para la aplicación. El formato de URI de redireccionamiento compatible con el agente es `msauth.<app.bundle.id>://auth`. Reemplace `<app.bundle.id>` por el identificador de paquete de la aplicación. Si va a migrar desde ADAL y la aplicación ya es compatible con el agente, no es necesario hacer nada más. El URI de redireccionamiento anterior es totalmente compatible con MSAL, por lo que puede ir al paso 3.

2. Agregue el esquema de URI de redireccionamiento de la aplicación al archivo info.plist. El formato del URI de redireccionamiento de MSAL predeterminado es `msauth.<app.bundle.id>`. Por ejemplo:

    ```xml
    <key>CFBundleURLSchemes</key>
    <array>
        <string>msauth.<app.bundle.id></string>
    </array>
    ```

3. Agregue los siguientes esquemas al archivo Info.plist de la aplicación en LSApplicationQueriesSchemes:

    ```xml
    <key>LSApplicationQueriesSchemes</key>
    <array>
         <string>msauthv2</string>
         <string>msauthv3</string>
    </array>
    ```

4. Agregue lo siguiente al archivo AppDelegate.m para controlar las devoluciones de llamada: Objective-C:
    
    ```objc
    - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options`
    {
        return [MSALPublicClientApplication handleMSALResponse:url sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
    }
    ```
    
    Swift:
    
    ```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
    }
    ```

### <a name="business-to-business-b2b"></a>Colaboración entre negocios (B2B)

En ADAL, se crean instancias independientes de `ADAuthenticationContext` para cada inquilino del que la aplicación solicita tokens. Esto ya no es un requisito en MSAL. En MSAL, puede crear una única instancia de `MSALPublicClientApplication` y usarla para cualquier nube y organización de AAD; para ello, se especifica una autoridad diferente para las llamadas a acquireToken y acquireTokenSilent.

## <a name="sso-in-partnership-with-other-sdks"></a>SSO en colaboración con otros SDK

MSAL para iOS puede lograr SSO mediante una memoria caché unificada con los siguientes SDK:

- ADAL Objective-C 2.7.x+
- MSAL.NET para Xamarin 2.4.x+
- ADAL.NET para Xamarin 4.4.x+

El SSO se consigue mediante el uso compartido del llavero de iOS y solo está disponible entre las aplicaciones publicadas desde la misma cuenta de desarrollador de Apple.

El SSO mediante el uso compartido del llavero de iOS es el único tipo de SSO silencioso.

En macOS, MSAL puede lograr SSO con otras aplicaciones basadas en MSAL para iOS y macOS y aplicaciones basadas en ADAL Objective-C.

En iOS, MSAL también admite otros dos tipos de SSO:

* SSO mediante el explorador web. MSAL para iOS admite `ASWebAuthenticationSession`, que proporciona SSO mediante cookies compartidas entre otras aplicaciones del dispositivo y específicamente el explorador Safari.
* SSO mediante un agente de autenticación En un dispositivo iOS, Microsoft Authenticator actúa como agente de autenticación. Puede seguir directivas de acceso condicional, como requerir un dispositivo compatible, y proporciona SSO para los dispositivos registrados. A partir de la versión 0.3.0, los SDK de MSAL admiten un agente de forma predeterminada.

## <a name="intune-mam-sdk"></a>ADK de MAM de Intune

El [SDK de MAM de Intune](/intune/app-sdk-get-started) admite MSAL para iOS a partir de la versión [11.1.2](https://github.com/msintuneappsdk/ms-intune-app-sdk-ios/releases/tag/11.1.2).

## <a name="msal-and-adal-in-the-same-app"></a>MSAL y ADAL en la misma aplicación

ADAL 2.7.0 y las versiones posteriores no pueden coexistir con MSAL en la misma aplicación. La razón principal es el código común del submódulo compartido. Como Objective-C no admite espacios de nombres, si agrega ADAL y MSAL a la aplicación, habrá dos instancias de la misma clase. No hay ninguna garantía de cuál se selecciona en tiempo de ejecución. Si ambos SDK usan la misma versión de la clase en conflicto, es posible que la aplicación siga funcionando. Sin embargo, si se trata de una versión diferente, la aplicación podría experimentar bloqueos inesperados que son difíciles de diagnosticar.

No se admite la ejecución de ADAL y MSAL en la misma aplicación de producción. Sin embargo, si solo está probando y migrando los usuarios de ADAL Objective-C a MSAL para iOS y macOS, puede seguir usando [ADAL Objective-C 2.6.10](https://github.com/AzureAD/azure-activedirectory-library-for-objc/releases/tag/2.6.10). Es la única versión que funciona con MSAL en la misma aplicación. No habrá nuevas actualizaciones de características para esta versión de ADAL, por lo que solo se debe usar con fines de migración y pruebas. La aplicación no debe depender de la coexistencia de ADAL y MSAL a largo plazo.

No se admite la coexistencia de ADAL y MSAL en la misma aplicación de producción.
ADAL y MSAL pueden coexistir en distintas aplicaciones sin problemas.

## <a name="practical-migration-steps"></a>Pasos prácticos de la migración

### <a name="app-registration-migration"></a>Migración del registro de aplicaciones

No es necesario modificar la aplicación de AAD existente para que cambie a MSAL y habilitar las cuentas de AAD. Sin embargo, si la aplicación basada en ADAL no es compatible con la autenticación con agente, deberá registrar un nuevo URI de redireccionamiento para la aplicación antes de cambiar a MSAL.

El URI de redireccionamiento debe tener este formato: `msauth.<app.bundle.id>://auth` Reemplace `<app.bundle.id>` por el identificador de paquete de la aplicación. Especifique el URI de redireccionamiento en [Azure Portal](https://aka.ms/MobileAppReg).

Solo para iOS, para admitir la autenticación basada en certificados, es necesario registrar un URI de redireccionamiento adicional en la aplicación y en Azure Portal, con el siguiente formato: `msauth://code/<broker-redirect-uri-in-url-encoded-form>`. Por ejemplo: `msauth://code/msauth.com.microsoft.mybundleId%3A%2F%2Fauth`

Se recomienda que todas las aplicaciones registren ambos URI de redireccionamiento.

Si desea agregar compatibilidad para el consentimiento incremental, seleccione las API y los permisos que la aplicación tiene configurados para solicitar acceso en el registro de la aplicación, en la pestaña **Permisos de API**.

Si va a migrar desde ADAL y quiere admitir cuentas de AAD y MSA, el registro de la aplicación existente debe actualizarse para admitir ambos. No se recomienda actualizar la aplicación de producción existente para que admita AAD y MSA directamente. En su lugar, cree otro identificador de cliente que admita AAD y MSA para hacer pruebas y, después de comprobar que todos los escenarios funcionan, actualice la aplicación existente.

### <a name="add-msal-to-your-app"></a>Adición de MSAL a la aplicación

Puede agregar el SDK de MSAL a la aplicación con la herramienta de administración de paquetes que prefiera. Consulte las [instrucciones detalladas aquí](https://github.com/AzureAD/microsoft-authentication-library-for-objc/wiki/Installation).

### <a name="update-your-apps-infoplist-file"></a>Actualización del archivo Info.plist de la aplicación

Solo para iOS, agregue el esquema de URI de redireccionamiento de la aplicación al archivo info.plist. En el caso de las aplicaciones compatibles con el agente de ADAL, ya deberían estar ahí. El formato del esquema de URI de redireccionamiento de MSAL predeterminado es `msauth.<app.bundle.id>`.  

```xml
<key>CFBundleURLSchemes</key>
<array>
    <string>msauth.<app.bundle.id></string>
</array>
```

Agregue los siguientes esquemas al archivo Info.plist de la aplicación en `LSApplicationQueriesSchemes`:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
     <string>msauthv2</string>
     <string>msauthv3</string>
</array>
```

### <a name="update-your-appdelegate-code"></a>Actualización del código de AppDelegate

Solo para iOS, agregue lo siguiente al archivo AppDelegate.m:

Objective-C:

```objc
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<NSString *,id> *)options`
{
    return [MSALPublicClientApplication handleMSALResponse:url sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
}
```

Swift:

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
}
```

**Si usa Xcode 11**, debe colocar la devolución de llamada de MSAL en el archivo `SceneDelegate` en su lugar.
Si admite UISceneDelegate y UIApplicationDelegate para la compatibilidad con sistemas operativos iOS anteriores, la devolución de llamada de MSAL debe colocarse en ambos archivos.

Objective-C:

```objc
 - (void)scene:(UIScene *)scene openURLContexts:(NSSet<UIOpenURLContext *> *)URLContexts
 {
     UIOpenURLContext *context = URLContexts.anyObject;
     NSURL *url = context.URL;
     NSString *sourceApplication = context.options.sourceApplication;
     
     [MSALPublicClientApplication handleMSALResponse:url sourceApplication:sourceApplication];
 }
```

Swift:

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        
        guard let urlContext = URLContexts.first else {
            return
        }
        
        let url = urlContext.url
        let sourceApp = urlContext.options.sourceApplication
        
        MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApp)
    }
```

Esto permite a MSAL controlar las respuestas del agente y el componente web.
Esto no era necesario en ADAL, ya que referenciaba los métodos de delegación de la aplicación automáticamente. Agregarla manualmente produce menos errores y ofrece más control a la aplicación.

### <a name="enable-token-caching"></a>Habilitación del almacenamiento en caché de tokens

De forma predeterminada, MSAL almacena en caché los tokens de la aplicación en el llavero de iOS o macOS. 

Para habilitar el almacenamiento en caché de tokens:
1. Asegúrese de que la aplicación está firmada correctamente.
2. Vaya a la configuración del proyecto de Xcode > **Pestaña Capabilities** (Funcionalidades) > **Enable Keychain Sharing** (Habilitar uso compartido de la cadena de claves)
3. Haga clic en **+** y, en **Keychain Groups** (Grupos de llaveros), escriba el valor siguiente: 3.a Para iOS, escriba `com.microsoft.adalcache` 3.b Para macOS, escriba `com.microsoft.identity.universalstorage`.

### <a name="create-msalpublicclientapplication-and-switch-to-its-acquiretoken-and-acquiretokesilent-calls"></a>Creación de MSALPublicClientApplication y cambio a sus llamadas acquireToken y acquireTokeSilent

Puede crear `MSALPublicClientApplication` mediante el código siguiente:

Objective-C:

```objc
NSError *error = nil;
MSALPublicClientApplicationConfig *configuration = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"];
    
MSALPublicClientApplication *application =
[[MSALPublicClientApplication alloc] initWithConfiguration:configuration
                                                     error:&error];
```

Swift:

```swift
let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>")
do {
  let application = try MSALPublicClientApplication(configuration: config)
  // continue on with application
            
} catch let error as NSError {
  // handle error here
}
```

Después, llame a la API de administración de cuentas para ver si hay alguna cuenta en la memoria caché:

Objective-C:

```objc
NSString *accountIdentifier = nil /*previously saved MSAL account identifier */;
NSError *error = nil;
MSALAccount *account = [application accountForIdentifier:accountIdentifier error:&error];
```

Swift:

```swift
// definitions that need to be initialized
let application: MSALPublicClientApplication!
let accountIdentifier: String! /*previously saved MSAL account identifier */

do {
  let account = try application.account(forIdentifier: accountIdentifier)
  // continue with account usage
} catch let error as NSError {
  // handle error here
}
```



O bien, lea todas las cuentas:

Objective-C:

```objc
NSError *error = nil;
NSArray<MSALAccount *> *accounts = [application allAccounts:&error];
```

Swift:

```swift
let application: MSALPublicClientApplication!
do {
  let accounts = try application.allAccounts()
  // continue with account usage
} catch let error as NSError {
  // handle error here
}
```



Si se encuentra una cuenta, llame a la API `acquireTokenSilent` de MSAL:

Objective-C:

```objc
MSALSilentTokenParameters *silentParameters = [[MSALSilentTokenParameters alloc] initWithScopes:@[@"<your-resource-here>/.default"] account:account];
    
[application acquireTokenSilentWithParameters:silentParameters
                              completionBlock:^(MSALResult *result, NSError *error)
{
    if (result)
    {
        NSString *accessToken = result.accessToken;
        // Use your token
    }
    else
    {
        // Check the error
        if ([error.domain isEqual:MSALErrorDomain] && error.code == MSALErrorInteractionRequired)
        {
            // Interactive auth will be required
        }
            
        // Other errors may require trying again later, or reporting authentication problems to the user
    }
}];
```

Swift:

```swift
let application: MSALPublicClientApplication!
let account: MSALAccount!
        
let silentParameters = MSALSilentTokenParameters(scopes: ["<your-resource-here>/.default"], 
                                                 account: account)
application.acquireTokenSilent(with: silentParameters) {
  (result: MSALResult?, error: Error?) in
  if let accessToken = result?.accessToken {
     // use accessToken
  }
  else {
    // Check the error
    guard let error = error else {
      assert(true, "callback should contain a valid result or error")
      return
    }
    
    let nsError = error as NSError
    if (nsError.domain == MSALErrorDomain
        && nsError.code == MSALError.interactionRequired.rawValue) {
      // Interactive auth will be required
    }
                
    // Other errors may require trying again later, or reporting authentication problems to the user
  }
}
```



## <a name="next-steps"></a>Pasos siguientes

Más información sobre [flujos de autenticación y escenarios de aplicaciones](authentication-flows-app-scenarios.md).