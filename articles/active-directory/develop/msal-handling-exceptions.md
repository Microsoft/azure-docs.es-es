---
title: Errores y excepciones (MSAL)
titleSuffix: Microsoft identity platform
description: Aprenda a controlar errores y excepciones, y obtenga más información sobre el acceso condicional y el desafío de las notificaciones en las aplicaciones de MSAL.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/18/2020
ms.author: marsma
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ms.openlocfilehash: 60c61ff4753413d2241820400dcbc899e925eecc
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120956"
---
# <a name="handle-msal-exceptions-and-errors"></a>Control de excepciones y errores en MSAL

En este artículo se proporciona información general sobre los distintos tipos de errores y las recomendaciones para controlar los errores de inicio de sesión comunes.

## <a name="msal-error-handling-basics"></a>Aspectos básicos del control de errores en MSAL

Las excepciones de la Biblioteca de autenticación de Microsoft (MSAL) están diseñadas para que los desarrolladores de aplicaciones solucionen problemas y no para mostrarse a los usuarios finales. Los mensajes de excepción no están traducidos.

Cuando procesa excepciones y errores, puede usar el propio tipo de excepción y el código de error para distinguir entre las excepciones.  Para ver una lista con los códigos de error, consulte [Códigos de error de autenticación y autorización](reference-aadsts-error-codes.md).

Durante la experiencia de inicio de sesión, puede encontrar errores de consentimiento, de acceso condicional (MFA, administración de dispositivos, restricciones basadas en la ubicación), de emisión y canje de tokens, y de las propiedades del usuario.

Vea la siguiente sección, que coincide con el idioma que está usando, para obtener detalles sobre el control de errores de la aplicación.

## <a name="net"></a>[.NET](#tab/dotnet)

Al procesar las excepciones de .NET, puede usar el propio tipo de excepción y el miembro `ErrorCode` para distinguir entre las excepciones. Los valores de `ErrorCode` son constantes del tipo [MsalError](/dotnet/api/microsoft.identity.client.msalerror?view=azure-dotnet).

También puede echar un vistazo a los campos de [MsalClientException](/dotnet/api/microsoft.identity.client.msalexception?view=azure-dotnet), [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet) y [MsalUIRequiredException](/dotnet/api/microsoft.identity.client.msaluirequiredexception?view=azure-dotnet).

Si se produce un error [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet), revise los [códigos de error de autenticación y autorización](reference-aadsts-error-codes.md) para ver si el código aparece aquí.

### <a name="common-net-exceptions"></a>Excepciones .NET comunes

Estas son las excepciones comunes que pueden producirse y algunas de las posibles mitigaciones:  

| Excepción | Código de error | Mitigación|
| --- | --- | --- |
| [MsalUiRequiredException](/dotnet/api/microsoft.identity.client.msaluirequiredexception?view=azure-dotnet) | AADSTS65001: The user or administrator has not consented to use the application with ID '{appId}' named '{appName}'. Send an interactive authorization request for this user and resource (Ni el usuario ni el administrador han dado consentimiento para usar la aplicación con ID "{appId}" llamada "{appName}". Envíe una solicitud de autorización interactiva para este usuario y este recurso).| Debe obtener primero el consentimiento del usuario. Si no utiliza .NET Core (que no tiene interfaz de usuario web), llame a `AcquireTokeninteractive` (solo una vez). Si lo utiliza o si no quiere realizar una operación `AcquireTokenInteractive`, el usuario puede ir a una dirección URL para dar su consentimiento: `https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read`. para llamar a `AcquireTokenInteractive`: `app.AcquireTokenInteractive(scopes).WithAccount(account).WithClaims(ex.Claims).ExecuteAsync();`|
| [MsalUiRequiredException](/dotnet/api/microsoft.identity.client.msaluirequiredexception?view=azure-dotnet) | AADSTS50079: The user is required to use [multi-factor authentication (MFA)](../authentication/concept-mfa-howitworks.md) (Es necesario que el usuario utilice la autenticación multifactor [MFA]).| No hay mitigación. Si la autenticación multifactor está configurada para el inquilino y Azure Active Directory (AAD) decide aplicarla, deberá recurrir a un flujo interactivo, como `AcquireTokenInteractive` o `AcquireTokenByDeviceCode`.|
| [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet) |AADSTS90010: The grant type isn't supported over the */common* or */consumers* endpoints. Use the */organizations* or tenant-specific endpoint. You used */common*. (El tipo de concesión no es compatible con los puntos de conexión /common o /consumers. Use /organizations o el punto de conexión específico del inquilino. Usó /common).| Como se ha explicado en el mensaje de Azure AD, la autoridad debe tener un inquilino o, en caso contrario, */organizations*.|
| [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet) | AADSTS70002: El cuerpo de la solicitud debe contener el siguiente parámetro: `client_secret or client_assertion`.| Esta excepción puede devolverse si la aplicación no estaba registrada como aplicación cliente pública en Azure AD. En Azure Portal, edite el manifiesto de la aplicación y establezca `allowPublicClient` en `true`. |
| [MsalClientException](/dotnet/api/microsoft.identity.client.msalclientexception?view=azure-dotnet)| `unknown_user Message`: Could not identify logged in user (No se pudo identificar al usuario que inició sesión).| La biblioteca no pudo consultar el usuario actual de Windows que ha iniciado sesión o este usuario no está unido a AD o AAD (los usuarios unidos a un lugar de trabajo no son compatibles). Mitigación 1: en UWP, compruebe que la aplicación tiene las siguientes funcionalidades: Autenticación empresarial, Redes privadas (cliente y servidor), Información de la cuenta de usuario. Mitigación2: Implemente su propia lógica para capturar el nombre de usuario (por ejemplo, john@contoso.com) y use el formulario `AcquireTokenByIntegratedWindowsAuth` que toma el nombre de usuario.|
| [MsalClientException](/dotnet/api/microsoft.identity.client.msalclientexception?view=azure-dotnet)|integrated_windows_auth_not_supported_managed_user| Este método se basa en un protocolo que Active Directory (AD) expone. Si un usuario se creó en Azure Active Directory sin respaldo de AD (usuario "administrado"), se producirá un error en este método. Los usuarios creados en AD y con respaldo de AAD (usuarios "federados") pueden beneficiarse de este método de autenticación no interactiva. Mitigación: Use la autenticación interactiva.|

### `MsalUiRequiredException`

Uno de los códigos de estado comunes que devuelve MSAL.NET al llamar a `AcquireTokenSilent()` es `MsalError.InvalidGrantError`. Este código de estado implica que la aplicación debe volver a llamar a la biblioteca de autenticación, aunque en modo interactivo (AcquireTokenInteractive o AcquireTokenByDeviceCodeFlow para las aplicaciones cliente públicas y realizar un desafío en las aplicaciones web). Esto se debe a que se requiere interacción adicional del usuario para emitir el token de autenticación.

La mayoría de las veces en que `AcquireTokenSilent` produce un error, se debe a que la memoria caché del token no tiene tokens que coincidan con la solicitud. Los tokens de acceso expiran en 1 hora y `AcquireTokenSilent` intentará capturar uno nuevo en función de un token de actualización (en términos de OAuth2, este es el flujo de "actualización de token"). Este flujo también puede producir un error por diversos motivos; por ejemplo, si un administrador de inquilinos configura directivas de inicio de sesión más rigurosas. 

La interacción tiene el objetivo de hacer que el usuario realice una acción. Algunas de esas condiciones son fáciles de solucionar para los usuarios (por ejemplo, aceptar las condiciones de uso con un solo clic) y otras no se pueden solucionar con la configuración actual (por ejemplo, el equipo en cuestión necesita conectarse a una red corporativa específica). Algunas ayudan al usuario a configurar la autenticación multifactor o a instalar Microsoft Authenticator en su dispositivo.

### <a name="msaluirequiredexception-classification-enumeration"></a>Enumeración de la clasificación `MsalUiRequiredException`

MSAL expone un campo `Classification` que puede leer para ofrecer una mejor experiencia de usuario. Por ejemplo, puede indicar al usuario que su contraseña expiró o que necesitará dar su consentimiento para usar ciertos recursos. Los valores admitidos son parte de la enumeración `UiRequiredExceptionClassification`:

| clasificación    | Significado           | Uso recomendado |
|-------------------|-------------------|----------------------|
| BasicAction | La interacción del usuario puede resolver la condición durante el flujo de autenticación interactiva. | Llame a AcquireTokenInteractively(). |
| AdditionalAction | La interacción de corrección con el sistema puede resolver la condición, independientemente del flujo de autenticación interactiva. | Llame a AcquireTokenInteractively() para mostrar un mensaje que explique la acción correctiva. La aplicación que realiza la llamada puede ocultar los flujos que requieren acciones adicionales si es poco probable que el usuario complete la acción correctiva. |
| MessageOnly      | No se puede resolver la condición en este momento. Al iniciar el flujo de autenticación interactiva, se mostrará un mensaje que explica la condición. | Llame a AcquireTokenInteractively() para mostrar un mensaje que explique la condición. AcquireTokenInteractively() devolverá el error UserCanceled después de que el usuario lea el mensaje y cierre la ventana. La aplicación que realiza la llamada puede optar por ocultar los flujos que dan como resultado solo un mensaje si es poco probable que el usuario se beneficie del mensaje.|
| ConsentRequired  | Falta el consentimiento del usuario o se ha revocado. | Llame a AcquireTokenInteractively() para que el usuario dé su consentimiento. |
| UserPasswordExpired | La contraseña del usuario ha expirado. | Llame a AcquireTokenInteractively() para que el usuario pueda restablecer su contraseña. |
| PromptNeverFailed| Se llamó a la autenticación interactiva con el parámetro prompt=never; esto obliga a MSAL a basarse en las cookies del explorador y no mostrar el explorador. Se produjo un error. | Llame a AcquireTokenInteractively() sin Prompt.None. |
| AcquireTokenSilentFailed | El SDK de MSAL no tiene suficiente información para capturar un token de la memoria caché. Esto puede deberse a que no hay tokens en la memoria caché o a que no se encontró una cuenta. El mensaje de error contiene más detalles.  | Llame a AcquireTokenInteractively(). |
| None    | No se proporcionan más detalles. La interacción del usuario puede resolver la condición durante el flujo de autenticación interactiva. | Llame a AcquireTokenInteractively(). |

## <a name="net-code-example"></a>Ejemplo de código .NET

```csharp
AuthenticationResult res;
try
{
 res = await application.AcquireTokenSilent(scopes, account)
        .ExecuteAsync();
}
catch (MsalUiRequiredException ex) when (ex.ErrorCode == MsalError.InvalidGrantError)
{
 switch (ex.Classification)
 {
  case UiRequiredExceptionClassification.None:
   break;
  case UiRequiredExceptionClassification.MessageOnly:
  // You might want to call AcquireTokenInteractive(). Azure AD will show a message
  // that explains the condition. AcquireTokenInteractively() will return UserCanceled error
  // after the user reads the message and closes the window. The calling application may choose
  // to hide features or data that result in message_only if the user is unlikely to benefit 
  // from the message
  try
  {
      res = await application.AcquireTokenInteractive(scopes).ExecuteAsync();
  }
  catch (MsalClientException ex2) when (ex2.ErrorCode == MsalError.AuthenticationCanceledError)
  {
   // Do nothing. The user has seen the message
  }
  break;

  case UiRequiredExceptionClassification.BasicAction:
  // Call AcquireTokenInteractive() so that the user can, for instance accept terms
  // and conditions

  case UiRequiredExceptionClassification.AdditionalAction:
  // You might want to call AcquireTokenInteractive() to show a message that explains the remedial action. 
  // The calling application may choose to hide flows that require additional_action if the user 
  // is unlikely to complete the remedial action (even if this means a degraded experience)

  case UiRequiredExceptionClassification.ConsentRequired:
  // Call AcquireTokenInteractive() for user to give consent.
  
  case UiRequiredExceptionClassification.UserPasswordExpired:
  // Call AcquireTokenInteractive() so that user can reset their password
  
  case UiRequiredExceptionClassification.PromptNeverFailed:
  // You used WithPrompt(Prompt.Never) and this failed
  
  case UiRequiredExceptionClassification.AcquireTokenSilentFailed:
  default:
  // May be resolved by user interaction during the interactive authentication flow.
  res = await application.AcquireTokenInteractive(scopes)
                         .ExecuteAsync(); break;
 }
}
```

## <a name="javascript"></a>[JavaScript](#tab/javascript)

MSAL.js proporciona objetos de error que abstraen y clasifican los distintos tipos de errores comunes. También proporciona la interfaz para tener acceso a detalles específicos de los errores, como los mensajes de error, para controlarlos de manera adecuada.

### <a name="error-object"></a>Objeto de error

```javascript
export class AuthError extends Error {
    // This is a short code describing the error
    errorCode: string;
    // This is a descriptive string of the error,
    // and may also contain the mitigation strategy
    errorMessage: string;
    // Name of the error class
    this.name = "AuthError";
}
```

Si extiende la clase de error, tendrá acceso a las siguientes propiedades:
- `AuthError.message`:  Igual que `errorMessage`.
- `AuthError.stack`: Seguimiento de la pila de los errores que se produjeron.

### <a name="error-types"></a>Tipos de errores

Están disponibles los siguientes tipos de errores:

- `AuthError`: Clase de error básica para la biblioteca MSAL.js. También se usa para los errores inesperados.

- `ClientAuthError`: clase de error que indica un problema con la autenticación de cliente. La mayoría de los errores que proceden de la biblioteca serán del tipo ClientAuthErrors. Estos errores son consecuencia de cosas como una llamada a un método de inicio de sesión con el inicio de sesión ya en curso, cuando el usuario cancela el inicio de sesión, etc.

- `ClientConfigurationError`: clase de error que extiende `ClientAuthError` que se produce antes de que se realicen las solicitudes si los parámetros de configuración de un usuario determinado faltan o tienen un formato incorrecto.

- `ServerError`: clase de error que representa las cadenas de error que envía el servidor de autenticación. Puede tratarse de errores como formatos o parámetros de solicitud no válidos o cualquier otro error que impida al servidor autenticar o autorizar al usuario.

- `InteractionRequiredAuthError`: clase de error que extiende `ServerError` para representar errores de servidor que requieren una llamada interactiva. `acquireTokenSilent` genera este error si se solicita al usuario que interactúe con el servidor para proporcionar credenciales o consentimiento para la autenticación o autorización. Los códigos de error incluyen `"interaction_required"`, `"login_required"` y `"consent_required"`.

Para el control de errores en flujos de autenticación con métodos de redireccionamiento (`loginRedirect`, `acquireTokenRedirect`), deberá registrar la devolución de llamada a la que se llama correctamente o con error después del redireccionamiento mediante el método `handleRedirectCallback()` de la siguiente manera:

```javascript
function authCallback(error, response) {
    //handle redirect response
}

var myMSALObj = new Msal.UserAgentApplication(msalConfig);

// Register Callbacks for redirect flow
myMSALObj.handleRedirectCallback(authCallback);
myMSALObj.acquireTokenRedirect(request);
```

Los métodos de la experiencia emergente (`loginPopup`, `acquireTokenPopup`) devuelven promesas, de forma que puede usar el patrón de promesa (.then y .catch) para controlarlas de la siguiente manera:

```javascript
myMSALObj.acquireTokenPopup(request).then(
    function (response) {
        // success response
    }).catch(function (error) {
        console.log(error);
    });
```

### <a name="errors-that-require-interaction"></a>Errores que requieren interacción

Se devuelve un error al intentar usar un método no interactivo para adquirir un token (por ejemplo, `acquireTokenSilent`) y que MSAL no pudiera hacerlo de manera silenciosa.

Los posibles motivos son:

- Debe iniciar sesión
- Debe dar su consentimiento
- Debe pasar por una experiencia de autenticación multifactor.

La solución consiste en llamar a un método interactivo como `acquireTokenPopup` o `acquireTokenRedirect`:

```javascript
// Request for Access Token
myMSALObj.acquireTokenSilent(request).then(function (response) {
    // call API
}).catch( function (error) {
    // call acquireTokenPopup in case of acquireTokenSilent failure
    // due to consent or interaction required
    if (error.errorCode === "consent_required"
    || error.errorCode === "interaction_required"
    || error.errorCode === "login_required") {
        myMSALObj.acquireTokenPopup(request).then(
            function (response) {
                // call API
            }).catch(function (error) {
                console.log(error);
            });
    }
});
```

## <a name="python"></a>[Python](#tab/python)

En MSAL para Python, la mayoría de los errores se transmiten como un valor devuelto de la llamada a la API. El error se representa como un diccionario que contiene la respuesta JSON de la Plataforma de identidad de Microsoft.

* Una respuesta correcta contiene la clave `"access_token"`. El formato de la respuesta se define mediante el protocolo OAuth2. Para más información, consulte [5.1 Respuesta correcta](https://tools.ietf.org/html/rfc6749#section-5.1)
* Una respuesta de error contiene `"error"` y, normalmente, `"error_description"`. El formato de la respuesta se define mediante el protocolo OAuth2. Para más información, consulte [5.2 Respuesta de error](https://tools.ietf.org/html/rfc6749#section-5.2)

Cuando se devuelve un error, la clave `"error_description"` suele contener un mensaje legible, que a su vez contiene un código de error de la Plataforma de identidad de Microsoft. Para ver los detalles de los diversos códigos de error, consulte [Códigos de error de autenticación y autorización](./reference-aadsts-error-codes.md).

En MSAL para Python, las excepciones son poco frecuentes, ya que, para controlar la mayoría de los errores, se devuelve un valor de error. La excepción `ValueError` solo se produce cuando hay un problema con la forma en que se intenta usar la biblioteca, como cuando los parámetros de la API no tienen el formato correcto.

## <a name="java"></a>[Java](#tab/java)

En MSAL para Java, hay tres tipos de excepciones: `MsalClientException`, `MsalServiceException` y `MsalInteractionRequiredException`; todo lo cual se hereda de `MsalException`.

- `MsalClientException` se produce cuando ocurre un error que es local en la biblioteca o el dispositivo.
- `MsalServiceException` se produce cuando el servicio de token seguro (STS) devuelve una respuesta de error o tiene lugar otro error de red.
- `MsalInteractionRequiredException` se produce cuando se requiere la interacción con la interfaz de usuario para que la autenticación se realice correctamente.

### <a name="msalserviceexception"></a>MsalServiceException

`MsalServiceException` expone los encabezados HTTP devueltos en las solicitudes al STS. Acceda a ellas a través de `MsalServiceException.headers()`.

### <a name="msalinteractionrequiredexception"></a>MsalInteractionRequiredException

Uno de los códigos de estado comunes que devuelve MSAL para Java al llamar a `AcquireTokenSilently()` es `InvalidGrantError`. Esto significa que se requiere interacción adicional del usuario para emitir el token de autenticación. La aplicación debe volver a llamar a la biblioteca de autenticación, pero en modo interactivo mediante el envío de `AuthorizationCodeParameters` o `DeviceCodeParameters` para aplicaciones cliente públicas.

La mayoría de las veces en que `AcquireTokenSilently` produce un error, se debe a que la memoria caché del token no tiene tokens que coincidan con la solicitud. Los tokens de acceso expiran en una hora y `AcquireTokenSilently` intentará obtener uno nuevo en función de un token de actualización. Según OAuth2, este es el flujo del token de actualización. Este flujo también puede producir un error por diversos motivos; por ejemplo, cuando un administrador de inquilinos configura directivas de inicio de sesión más rigurosas.

Algunas de las condiciones que producen este error son fáciles de resolver para los usuarios. Por ejemplo, puede que tengan que aceptar los términos de uso. O quizás la solicitud no se puede cumplir con la configuración actual porque la máquina debe conectarse a una red corporativa específica.

MSAL expone un campo de `reason`, que puede usar para proporcionar una mejor experiencia de usuario. Por ejemplo, el campo `reason` puede indicarle que debe decirle al usuario que su contraseña expiró o que necesitará dar su consentimiento para usar ciertos recursos. Los valores admitidos son parte de la enumeración `InteractionRequiredExceptionReason`:

| Motivo | Significado | Uso recomendado |
|---------|-----------|-----------------------------|
| `BasicAction` | La interacción del usuario puede resolver la condición durante el flujo de autenticación interactiva | Llame a `acquireToken` con parámetros interactivos |
| `AdditionalAction` | La interacción de corrección adicional con el sistema puede resolver la condición, independientemente del flujo de autenticación interactiva. | Llame a `acquireToken` con parámetros interactivos para mostrar un mensaje que explique la acción correctiva. La aplicación que realiza la llamada puede ocultar los flujos que requieren acciones adicionales si es poco probable que el usuario complete la acción correctiva. |
| `MessageOnly` | No se puede resolver la condición en este momento. Al iniciar el flujo de autenticación interactiva, se mostrará un mensaje que explica la condición. | Llame a `acquireToken` con parámetros interactivos para mostrar un mensaje que explique la condición. `acquireToken` devolverá el error `UserCanceled` después de que el usuario lea el mensaje y cierre la ventana. La aplicación puede optar por ocultar los flujos que dan como resultado un mensaje si es poco probable que el usuario se beneficie del mensaje. |
| `ConsentRequired`| Falta el consentimiento del usuario o se ha revocado. |Llame a `acquireToken` con parámetros interactivos para que el usuario pueda dar su consentimiento. |
| `UserPasswordExpired` | La contraseña del usuario ha expirado. | Llame a `acquireToken` con parámetros interactivos para que el usuario pueda restablecer su contraseña. |
| `None` |  No se proporcionan más detalles. La interacción del usuario puede resolver la condición durante el flujo de autenticación interactiva. | Llame a `acquireToken` con parámetros interactivos |

### <a name="code-example"></a>Ejemplo de código

```java
        IAuthenticationResult result;
        try {
            PublicClientApplication application = PublicClientApplication
                    .builder("clientId")
                    .b2cAuthority("authority")
                    .build();

            SilentParameters parameters = SilentParameters
                    .builder(Collections.singleton("scope"))
                    .build();

            result = application.acquireTokenSilently(parameters).join();
        }
        catch (Exception ex){
            if(ex instanceof MsalInteractionRequiredException){
                // AcquireToken by either AuthorizationCodeParameters or DeviceCodeParameters
            } else{
                // Log and handle exception accordingly
            }
        }
```

## <a name="iosmacos"></a>[iOS/macOS](#tab/iosmacos)

La lista completa de errores de MSAL para iOS y macOS se muestra en la [enumeración de MSALError](https://github.com/AzureAD/microsoft-authentication-library-for-objc/blob/master/MSAL/src/public/MSALError.h#L128).

Todos los errores de MSAL generados se devuelven con el dominio `MSALErrorDomain`.

Respecto a los errores de sistema, MSAL devuelve el `NSError` original de la API del sistema. Por ejemplo, si se produce un error en la adquisición de token debido a la falta de conectividad de red, MSAL devuelve un error con el dominio `NSURLErrorDomain` y el código `NSURLErrorNotConnectedToInternet`.

Es recomendable controlar al menos los dos errores de MSAL siguientes desde el lado cliente:

- `MSALErrorInteractionRequired`: El usuario debe realizar una solicitud interactiva. Hay muchas condiciones que pueden causar este error, como la expiración de la sesión de autenticación o la necesidad de requisitos de autenticación adicionales. Llame a la API de adquisición interactiva de tokens de MSAL para la recuperación. 

- `MSALErrorServerDeclinedScopes`: Se rechazaron algunos o todos los ámbitos. Decida si desea continuar con solo los ámbitos concedidos o detener el proceso de inicio de sesión.

> [!NOTE]
> La lista `MSALInternalError` solo se debe usar para la depuración y como referencia. No intente controlar automáticamente estos errores en tiempo de ejecución. Si la aplicación encuentra alguno de los errores de `MSALInternalError`, puede que desee mostrar un mensaje genérico orientado al usuario que explique lo que ha sucedido.

Por ejemplo, `MSALInternalErrorBrokerResponseNotReceived` significa que el usuario no completó la autenticación y se volvió manualmente a la aplicación. En este caso, la aplicación debe mostrar un mensaje de error genérico que explique que la autenticación no se ha completado y que sugiera un reintento de autenticación.

El siguiente código de ejemplo de Objective-C muestra los procedimientos recomendados para controlar algunas condiciones de error comunes:

```objc
    MSALInteractiveTokenParameters *interactiveParameters = ...;
    MSALSilentTokenParameters *silentParameters = ...;
    
    MSALCompletionBlock completionBlock;
    __block __weak MSALCompletionBlock weakCompletionBlock;
    
    weakCompletionBlock = completionBlock = ^(MSALResult *result, NSError *error)
    {
        if (!error)
        {
            // Use result.accessToken
            NSString *accessToken = result.accessToken;
            return;
        }
        
        if ([error.domain isEqualToString:MSALErrorDomain])
        {
            switch (error.code)
            {
                case MSALErrorInteractionRequired:
                {
                    // Interactive auth will be required
                    [application acquireTokenWithParameters:interactiveParameters
                                            completionBlock:weakCompletionBlock];
                    
                    break;
                }
                    
                case MSALErrorServerDeclinedScopes:
                {
                    // These are list of granted and declined scopes.
                    NSArray *grantedScopes = error.userInfo[MSALGrantedScopesKey];
                    NSArray *declinedScopes = error.userInfo[MSALDeclinedScopesKey];
                    
                    // To continue acquiring token for granted scopes only, do the following
                    silentParameters.scopes = grantedScopes;
                    [application acquireTokenSilentWithParameters:silentParameters
                                                  completionBlock:weakCompletionBlock];
                    
                    // Otherwise, instead, handle error fittingly to the application context
                    break;
                }
                    
                case MSALErrorServerProtectionPoliciesRequired:
                {
                    // Integrate the Intune SDK and call the
                    // remediateComplianceForIdentity:silent: API.
                    // Handle this error only if you integrated Intune SDK.
                    // See more info here: https://aka.ms/intuneMAMSDK
                    
                    break;
                }
                    
                case MSALErrorUserCanceled:
                {
                    // The user cancelled the web auth session.
                    // You may want to ask the user to try again.
                    // Handling of this error is optional.
                    
                    break;
                }
                    
                case MSALErrorInternal:
                {
                    // Log the error, then inspect the MSALInternalErrorCodeKey
                    // in the userInfo dictionary.
                    // Display generic error message to the end user
                    // More detailed information about the specific error
                    // under MSALInternalErrorCodeKey can be found in MSALInternalError enum.
                    NSLog(@"Failed with error %@", error);
                    
                    break;
                }
                    
                default:
                    NSLog(@"Failed with unknown MSAL error %@", error);
                    
                    break;
            }
            
            return;
        }
        
        // Handle no internet connection.
        if ([error.domain isEqualToString:NSURLErrorDomain] && error.code == NSURLErrorNotConnectedToInternet)
        {
            NSLog(@"No internet connection.");
            return;
        }
        
        // Other errors may require trying again later,
        // or reporting authentication problems to the user.
        NSLog(@"Failed with error %@", error);
    };
    
    // Acquire token silently
    [application acquireTokenSilentWithParameters:silentParameters
                                  completionBlock:completionBlock];

     // or acquire it interactively.
     [application acquireTokenWithParameters:interactiveParameters
                             completionBlock:completionBlock];
```

```swift
    let interactiveParameters: MSALInteractiveTokenParameters = ...
    let silentParameters: MSALSilentTokenParameters = ...
            
    var completionBlock: MSALCompletionBlock!
    completionBlock = { (result: MSALResult?, error: Error?) in
                
        if let result = result
        {
            // Use result.accessToken
            let accessToken = result.accessToken
            return
        }

        guard let error = error as NSError? else { return }

        if error.domain == MSALErrorDomain, let errorCode = MSALError(rawValue: error.code)
        {
            switch errorCode
            {
                case .interactionRequired:
                    // Interactive auth will be required
                    application.acquireToken(with: interactiveParameters, completionBlock: completionBlock)

                case .serverDeclinedScopes:
                    let grantedScopes = error.userInfo[MSALGrantedScopesKey]
                    let declinedScopes = error.userInfo[MSALDeclinedScopesKey]

                    if let scopes = grantedScopes as? [String] {
                        silentParameters.scopes = scopes
                        application.acquireTokenSilent(with: silentParameters, completionBlock: completionBlock)
                    }
                        
                    case .serverProtectionPoliciesRequired:
                        // Integrate the Intune SDK and call the
                        // remediateComplianceForIdentity:silent: API.
                        // Handle this error only if you integrated Intune SDK.
                        // See more info here: https://aka.ms/intuneMAMSDK
                        break
                        
                    case .userCanceled:
                       // The user cancelled the web auth session.
                       // You may want to ask the user to try again.
                       // Handling of this error is optional.
                       break
                        
                    case .internal:
                        // Log the error, then inspect the MSALInternalErrorCodeKey
                        // in the userInfo dictionary.
                        // Display generic error message to the end user
                        // More detailed information about the specific error
                        // under MSALInternalErrorCodeKey can be found in MSALInternalError enum.
                        print("Failed with error \(error)");
                        
                    default:
                        print("Failed with unknown MSAL error \(error)")
            }
        }
                
        // Handle no internet connection.
        if error.domain == NSURLErrorDomain && error.code == NSURLErrorNotConnectedToInternet
        {
            print("No internet connection.")
            return
        }
                
        // Other errors may require trying again later,
        // or reporting authentication problems to the user.
        print("Failed with error \(error)");    
    }
   
    // Acquire token silently
    application.acquireToken(with: interactiveParameters, completionBlock: completionBlock)
 
    // or acquire it interactively.
    application.acquireTokenSilent(with: silentParameters, completionBlock: completionBlock)
```

---

## <a name="conditional-access-and-claims-challenges"></a>Acceso condicional y desafíos de notificaciones

Al obtener los tokens de forma automática, la aplicación puede recibir errores si una API a la que está intentando acceder requiere un [desafío de notificaciones de acceso condicional](../azuread-dev/conditional-access-dev-guide.md) como, por ejemplo, una directiva de autenticación multifactor.

El patrón para controlar este error consiste en adquirir interactivamente un token mediante MSAL. La adquisición interactiva de un token avisa al usuario y le da la oportunidad de cumplir con la directiva de acceso condicional requerida.

En algunos casos, al llamar a una API que requiere acceso condicional, puede recibir un desafío de notificaciones en el error de la API. Por ejemplo, si la directiva de acceso condicional consiste en tener un dispositivo administrado (Intune) el error puede ser parecido a este: [AADSTS53000: Your device is required to be managed to access this resource](reference-aadsts-error-codes.md) (Se requiere que su dispositivo sea administrado para acceder a este recurso) o algo parecido. En este caso, puede pasar las notificaciones de la llamada de adquisición del token de manera que se le pida al usuario que cumpla con la directiva adecuada.

### <a name="net"></a>.NET

Cuando llama a una API que requiere acceso condicional desde MSAL.NET, la aplicación debe controlar las excepciones del desafío de notificaciones. Esto aparecerá como [MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet) donde la propiedad [Claims](/dotnet/api/microsoft.identity.client.msalserviceexception.claims?view=azure-dotnet) no estará vacía.

Para controlar el desafío de notificaciones, deberá usar el método `.WithClaim()` de la clase `PublicClientApplicationBuilder`.

### <a name="javascript"></a>JavaScript

Al obtener los tokens de forma automática (mediante `acquireTokenSilent`) con MSAL.js, la aplicación puede recibir errores si una API a la que está intentando acceder requiere un [desafío de notificaciones de acceso condicional](../azuread-dev/conditional-access-dev-guide.md) como, por ejemplo, una directiva de autenticación multifactor.

El patrón para controlar este error consiste en realizar una llamada interactiva para adquirir el token en MSAL.js como, por ejemplo, `acquireTokenPopup` o `acquireTokenRedirect` como se puede ver en el ejemplo siguiente:

```javascript
myMSALObj.acquireTokenSilent(accessTokenRequest).then(function(accessTokenResponse) {
    // call API
}).catch(function(error) {
    if (error instanceof InteractionRequiredAuthError) {
    
        // extract, if exists, claims from error message
        if (error.ErrorMessage.claims) {
            accessTokenRequest.claimsRequest = JSON.stringify(error.ErrorMessage.claims);
        }
        
        // call acquireTokenPopup in case of InteractionRequiredAuthError failure
        myMSALObj.acquireTokenPopup(accessTokenRequest).then(function(accessTokenResponse) {
            // call API
        }).catch(function(error) {
            console.log(error);
        });
    }
});
```

La adquisición interactiva del token avisa al usuario y le da la oportunidad de cumplir con la directiva de acceso condicional requerida.

Al llamar a una API que requiere acceso condicional, puede recibir un desafío de notificaciones en el error de la API. En este caso, puede pasar las notificaciones devueltas en el error al campo `claimsRequest` de la clase `AuthenticationParameters.ts` para satisfacer la directiva adecuada. 

Consulte cómo [solicitar notificaciones adicionales](active-directory-optional-claims.md) para más información.

### <a name="msal-for-ios-and-macos"></a>MSAL para iOS y macOS

MSAL para iOS y macOS permite solicitar notificaciones específicas en escenarios de adquisición de tokens interactivos y silenciosos.

Para solicitar notificaciones personalizadas, especifique `claimsRequest` en `MSALSilentTokenParameters` o `MSALInteractiveTokenParameters`.

Consulte [Solicitud de notificaciones personalizadas en MSAL para iOS y macOS](request-custom-claims.md) para más información.

## <a name="retrying-after-errors-and-exceptions"></a>Nuevos intentos después de errores y excepciones

Se espera que implemente sus propias directivas de reintentos al llamar a MSAL. MSAL realiza llamadas HTTP al servicio AAD y pueden producirse errores ocasionales; por ejemplo, la red puede dejar de funcionar o el servidor puede estar sobrecargado.  

### <a name="http-error-codes-500-600"></a>Códigos de error HTTP 500-600

MSAL.NET implementa un único mecanismo de un solo reintento para aquellos errores con códigos de error 500-600.

### <a name="http-429"></a>HTTP 429

Cuando el servidor de token de servicio (STS) se sobrecarga con demasiadas solicitudes, devuelve un error HTTP 429 con una sugerencia sobre el tiempo de espera necesario antes de volver a intentarlo en el campo de respuesta `Retry-After`.

### <a name="net"></a>.NET

[MsalServiceException](/dotnet/api/microsoft.identity.client.msalserviceexception?view=azure-dotnet) muestra `System.Net.Http.Headers.HttpResponseHeaders` como propiedad `namedHeaders`. Puede usar la información adicional del código de error para mejorar la confiabilidad de las aplicaciones. En el caso que acabamos de describir, puede usar `RetryAfterproperty` (del tipo `RetryConditionHeaderValue`) y calcular cuándo se debe volver a intentar.

Este es un ejemplo de una aplicación de demonio que usa el flujo de credenciales de cliente. Puede adaptarlo a cualquiera de los métodos de adquisición de un token.

```csharp
do
{
    retry = false;
    TimeSpan? delay;
    try
    {
         result = await publicClientApplication.AcquireTokenForClient(scopes, account)
                                           .ExecuteAsync();
    }
    catch (MsalServiceException serviceException)
    {
         if (ex.ErrorCode == "temporarily_unavailable")
         {
             RetryConditionHeaderValue retryAfter = serviceException.Headers.RetryAfter;
             if (retryAfter.Delta.HasValue)
             {
                 delay = retryAfter.Delta;
             }
             else if (retryAfter.Date.HasValue)
             {
                 delay = retryAfter.Date.Value.Offset;
             }
         }
    }
    …
    if (delay.HasValue)
    {
        Thread.Sleep((int)delay.Value.TotalMilliseconds); // sleep or other
        retry = true;
    }
} while (retry);
```