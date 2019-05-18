---
title: Prácticas recomendadas de control de errores para los clientes de la Biblioteca de autenticación de Azure Active Directory (ADAL)
description: Proporciona instrucciones y procedimientos recomendados de control de errores para las aplicaciones cliente de ADAL.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
ms.author: ryanwi
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/27/2017
ms.custom: ''
ms.collection: M365-identity-device-management
ms.openlocfilehash: 652175e99c800b8e4aa69c639f0bdb9aba838987
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544645"
---
# <a name="error-handling-best-practices-for-azure-active-directory-authentication-library-adal-clients"></a>Prácticas recomendadas de control de errores para los clientes de la Biblioteca de autenticación de Azure Active Directory (ADAL)

Este artículo proporciona instrucciones sobre el tipo de errores que pueden encontrarse los desarrolladores al usar ADAL para autenticar a los usuarios. Al usar ADAL, existen varios casos en los cuales un desarrollador quiera intervenir y controlar los errores. Un correcto control de errores garantiza una experiencia de usuario final excelente y limita el número de veces que el usuario final debe iniciar sesión.

En este artículo exploramos los casos específicos para cada plataforma que admite ADAL y cómo su aplicación puede tratar cada caso correctamente. La guía de errores se divide en dos categorías más amplias, según los patrones de adquisición de tokens proporcionados por las API de ADAL:

- **AcquireTokenSilent**: el cliente intenta obtener un token de forma silenciosa (sin interfaz de usuario) y es posible que se produzca un error si la instancia ADAL no es correcta. 
- **AcquireToken**: el cliente puede intentar una adquisición silenciosa pero también puede realizar solicitudes interactivas que requieran un inicio de sesión.

> [!TIP]
> Es una buena idea registrar todos los errores y excepciones cuando se usa ADAL y Azure AD. Los registros no solo son útiles para entender el estado general de la aplicación, sino que también son importantes cuando se realiza la depuración de problemas más amplios. Aunque es posible que su aplicación se recupere de determinados errores, eso podría indicar la presencia de problemas de diseño más amplios que requieren cambios en el código para poder resolverlos. 
> 
> Al implementar las condiciones de error que se tratan en este documento, debería registrar el código de error y la descripción debido a los motivos mencionados anteriormente. Consulte [Error and logging reference](#error-and-logging-reference) (Referencia de errores y registros) para obtener ejemplos de código de registro. 
>

## <a name="acquiretokensilent"></a>AcquireTokenSilent

AcquireTokenSilent intenta obtener un token con la garantía de que el usuario final no ve una interfaz de usuario (UI). Hay varios casos donde quizá no se pueda llevar a cabo la adquisición silenciosa y deba controlarse a través de solicitudes interactivas o mediante un controlador predeterminado. Veremos en profundidad los detalles sobre cuándo y cómo emplear cada caso en las secciones siguientes.

Hay un conjunto de errores que genera el sistema operativo que podrían requerir un control de errores específico para la aplicación. Para obtener más información, vea la sección de errores "Operating System" (Sistema operativo) en [Error and logging reference](#error-and-logging-reference) (Referencia de errores y registros). 

### <a name="application-scenarios"></a>Escenarios de aplicación

- Aplicaciones de [cliente nativo](developer-glossary.md#native-client) (iOS, Android, escritorio de .NET o Xamarin)
- Aplicaciones de [cliente web](developer-glossary.md#web-client) que llaman a un [recurso](developer-glossary.md#resource-server) (. NET)

### <a name="error-cases-and-actionable-steps"></a>Casos de error y acciones recomendadas

Principalmente, hay dos casos de errores de AcquireTokenSilent:

| Caso | DESCRIPCIÓN |
|------|-------------|
| **Caso 1**: Puede resolverse el error con un inicio de sesión interactivo | Es necesaria una solicitud interactiva para los errores debidos a la ausencia de tokens válidos. En concreto, una búsqueda en caché y un token de actualización no válido o que haya expirado requieren una llamada a AcquireToken para resolverse.<br><br>En estos casos, debe pedirse al usuario final que inicie sesión. La aplicación puede optar por hacer una solicitud interactiva inmediatamente, tras la interacción del usuario final (por ejemplo, pulsar un botón de inicio de sesión), o hacerla posteriormente. La elección depende del comportamiento deseado para la aplicación.<br><br>Vea el código en la sección siguiente para este caso concreto y los errores que permiten diagnosticarlo.|
| **Caso 2**: No puede resolverse el error con un inicio de sesión interactivo | Para los errores de red y los errores transitorios o temporales, u otros problemas, una solicitud interactiva de AcquireToken no resuelve el problema. Las solicitudes de inicio de sesión interactivas innecesarias pueden frustrar a los usuarios finales. ADAL realiza automáticamente un reintento único para la mayoría de los errores de AcquireTokenSilent.<br><br>La aplicación cliente también puede realizar un reintento en algún momento posterior, pero cuándo y cómo hacerlo depende del comportamiento de la aplicación y de la experiencia del usuario final deseada. Por ejemplo, la aplicación puede realizar un reintento de AcquireTokenSilent tras unos minutos o como respuesta a alguna acción del usuario final. Un reintento inmediato dará como resultado que la aplicación se encuentre limitada y no debe realizarse.<br><br>Un reintento subsiguiente que produzca el mismo error no significa que el cliente debe realizar una solicitud interactiva mediante AcquireToken porque eso no soluciona el error.<br><br>Vea el código en la sección siguiente para este caso concreto y los errores que permiten diagnosticarlo. |

### <a name="net"></a>.NET

La guía siguiente proporciona ejemplos para el control de errores junto con métodos de ADAL: 

- acquireTokenSilentAsync(…)
- acquireTokenSilentSync(…) 
- (En desuso) acquireTokenSilent(…)
- (En desuso) acquireTokenByRefreshToken(…) 

El código debería implementarse así:

```csharp
try{
    AcquireTokenSilentAsync(…);
} 

catch (AdalSilentTokenAcquisitionException e) {
    // Exception: AdalSilentTokenAcquisitionException
    // Caused when there are no tokens in the cache or a required refresh failed. 

    // Action: Case 1, resolvable with an interactive request. 
} 

catch(AdalServiceException e) {
    // Exception: AdalServiceException 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.

    // Action: Case 2, not resolvable with an interactive request.
    // Attempt retry after a timed interval or user action.
} 
    
catch (AdalException e) {
    // Exception: AdalException 
    // Represents a library exception generated by ADAL .NET. 
    // e.ErrorCode contains the error code. 

    // Action: Case 2, not resolvable with an interactive request.
    // Attempt retry after a timed interval or user action.
    // Example Error: network_not_available, default case.
}
```

### <a name="android"></a>Android

La guía siguiente proporciona ejemplos para el control de errores junto con métodos de ADAL: 

- acquireTokenSilentSync(…)
- acquireTokenSilentAsync(...)
- (En desuso) acquireTokenSilent(…)

El código debería implementarse así:

```java
// *Inside callback*
public void onError(Exception e) {

    if (e instanceof AuthenticationException) {
        // Exception: AdalException
        // Represents a library exception generated by ADAL Android.
        // Error Code: e.getCode().

        // Errors: ADALError.ERROR_SILENT_REQUEST,
        // ADALError.AUTH_REFRESH_FAILED_PROMPT_NOT_ALLOWED,
        // ADALError.INVALID_TOKEN_CACHE_ITEM
        // Description: Request failed due to no tokens in
        // cache or failed a required refresh. 

        // Action: Case 1, resolvable with an interactive request. 

        // Action: Case 2, not resolvable with an interactive request.
        // Attempt retry after a timed interval or user action.
        // Example Errors: default case,
        // DEVICE_CONNECTION_IS_NOT_AVAILABLE, 
        // BROKER_AUTHENTICATOR_ERROR_GETAUTHTOKEN,
    }
}
```

### <a name="ios"></a>iOS

La guía siguiente proporciona ejemplos para el control de errores junto con métodos de ADAL: 

- acquireTokenSilentWithResource(…)

El código debería implementarse así:

```objc
[context acquireTokenSilentWithResource:[ARGS], completionBlock:^(ADAuthenticationResult *result) {
    if (result.status == AD_FAILED) {
        if ([error.domain isEqualToString:ADAuthenticationErrorDomain]){
            // Exception: AD_FAILED 
            // Represents a library error generated by ADAL Objective-C.
            // Error Code: result.error.code

            // Errors: AD_ERROR_SERVER_REFRESH_TOKEN_REJECTED, AD_ERROR_CACHE_NO_REFRESH_TOKEN
            // Description: No tokens in cache or failed a required token refresh failed. 
            // Action: Case 1, resolvable with an interactive request. 

            // Error: AD_ERROR_CACHE_MULTIPLE_USERS
            // Description: There was ambiguity in the silent request resulting in multiple cache items.
            // Action: Special Case, application should perform another silent request and specify the user using ADUserIdentifier. 
            // Can be caused in cases of a multi-user application. 

            // Action: Case 2, not resolvable with an interactive request.
            // Attempt retry after some time or user action.
            // Example Errors: default case,
            // AD_ERROR_CACHE_BAD_FORMAT
        }
    }
}]
```

## <a name="acquiretoken"></a>AcquireToken

AcquireToken es el método de ADAL predeterminado que se usa para obtener tokens. En los casos donde se requiere la identidad del usuario, AcquireToken primero intenta obtener un token de forma silenciosa y, a continuación, muestra la interfaz de usuario si es necesario (a menos que se pase el elemento PromptBehavior.Never). En los casos donde se requiere la identidad de la aplicación, AcquireToken intenta obtener un token pero no muestra la interfaz de usuario porque no hay ningún usuario final. 

En caso de controlar errores de AcquireToken, el control de errores depende de la plataforma y situación que la aplicación está intentando alcanzar. 

El sistema operativo también puede generar un conjunto de errores, que requieren un control de errores que depende de la aplicación específica. Para obtener más información, vea la sección "Operating System errors" (Errores del sistema operativo) en [Error and logging reference](#error-and-logging-reference) (Referencia de errores y registros). 

### <a name="application-scenarios"></a>Escenarios de aplicación

- Aplicaciones de cliente nativo (iOS, Android, escritorio de .NET o Xamarin)
- Aplicaciones web que llaman a una API de recurso (.NET)
- Aplicaciones de una sola página (JavaScript)
- Aplicaciones de servicio a servicio (.NET, Java)
  - Todos las situaciones, incluida en nombre de
  - Situaciones concretas de en nombre de

### <a name="error-cases-and-actionable-steps-native-client-applications"></a>Casos de error y acciones recomendadas: Aplicaciones cliente nativas

Si va a compilar una aplicación de cliente nativo, hay algunos casos de control de errores a tener en cuenta que se relacionan con problemas de red, errores transitorios y otros errores específicos de la plataforma. En la mayoría de los casos, una aplicación no debería realizar intentos inmediatos, sino esperar a la interacción del usuario final que solicita un inicio de sesión. 

Hay algunos casos especiales en los que un reintento único puede resolver el problema. Por ejemplo, cuando un usuario necesita habilitar los datos en un dispositivo o completó la descarga del agente de Azure AD tras el error inicial. 

En los casos de error, una aplicación puede mostrar la interfaz de usuario para permitir que el usuario final realice algún tipo de interacción que solicita un reintento. Por ejemplo, si se produce un error en el dispositivo debido a un error de conexión, es recomendable el uso de un botón "Intente volver a iniciar sesión" que solicite un reintento de AcquireToken en lugar de realizar un reintento inmediatamente tras el error. 

El control de errores en las aplicaciones nativas puede definirse mediante dos casos:

|  |  |
|------|-------------|
| **Caso 1**:<br>Error que no se puede volver a intentar (la mayoría de los casos) | 1. No realice ningún reintento inmediato. Muestre la interfaz de usuario final basada en el error específico que invoque un reintento ("Intente volver a iniciar sesión", "Descargue la aplicación del agente de Azure AD", etc.). |
| **Caso 2**:<br>Error que se puede volver a intentar | 1. Realice un reintento único, ya que es posible que el usuario se encuentre en un estado que resulte en un estado correcto.<br><br>2. Si no se puede llevar a cabo el reintento, muestre la interfaz de usuario final basada en el error específico que invoque un reintento ("Intente volver a iniciar sesión", "Descargue la aplicación del agente de Azure AD", etc.). |

> [!IMPORTANT]
> Si se pasa una cuenta de usuario a ADAL en una llamada silenciosa y se produce un error, la siguiente solicitud interactiva permite que el usuario final inicie sesión con una cuenta diferente. Tras una instancia de AcquireToken correcta mediante una cuenta de usuario, la aplicación debe comprobar el usuario que inició sesión coincide con objeto de usuario local de la aplicación. Un error de coincidencia no genera ninguna excepción (excepto en Objective C), pero debería considerarse en los casos donde un usuario se conoce localmente antes de las solicitudes de autenticación (por ejemplo, un error de llamada silenciosa).
>

#### <a name="net"></a>.NET

La guía siguiente proporciona ejemplos para el control de errores junto con todos los métodos ADAL de AcquireToken(…) no silenciosos, *con excepción de*: 

- AcquireTokenAsync(…, IClientAssertionCertification, …)
- AcquireTokenAsync(…,ClientCredential, …)
- AcquireTokenAsync(…,ClientAssertion, …)
- AcquireTokenAsync(…,UserAssertion,…)   

El código debería implementarse así:

```csharp
try {
    AcquireTokenAsync(…);
} 
    
catch(AdalServiceException e) {
    // Exception: AdalServiceException 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.
    
    // Design time consideration: Certain errors may be caused at development and exposed through this exception. 
    // Looking inside the description will give more guidance on resolving the specific issue. 

    // Action: Case 1: Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: default case

    } 

catch (AdalException e) {
    // Exception: AdalException 
    // Represents a library exception generated by ADAL .NET.
    // e.ErrorCode contains the error code

    // Action: Case 1, Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: network_not_available, default case
}
```

> [!NOTE]
> AAL .NET tiene una consideración adicional, ya que admite el elemento PromptBehavior.Never, que tiene un comportamiento similar a AcquireTokenSilent.
>

La guía siguiente proporciona ejemplos para el control de errores junto con métodos de ADAL: 

- acquireToken(…, PromptBehavior.Never)

El código debería implementarse así:

```csharp
    try {acquireToken(…, PromptBehavior.Never);
    } 

catch(AdalServiceException e) {
    // Exception: AdalServiceException represents 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.

    // Action: Case 1: Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: default case

} catch (AdalException e) {
    // Error Code: e.ErrorCode == "user_interaction_required"
    // Description: user_interaction_required indicates the silent request failed 
    // in a way that's resolvable with an interactive request.
    // Action: Resolvable with an interactive request. 

    // Action: Case 1, Non-Retryable 
    // Do not perform an immediate retry. Only retry after user action. 
    // Example Errors: network_not_available, default case
}
```

#### <a name="android"></a>Android

La guía siguiente proporciona ejemplos para el control de errores junto con todos los métodos ADAL de AcquireToken(…) no silenciosos. 

El código debería implementarse así:

```java
AcquireTokenAsync(…);

// *Inside callback*
public void onError(Exception e) {
    if (e instanceof AuthenticationException) {
        // Exception: AdalException 
        // Represents a library exception generated by ADAL Android.
        // Error Code: e.getCode();

        // Error: ADALError.BROKER_APP_INSTALLATION_STARTED
        // Description: Broker app not installed, user will be prompted to download the app. 

        // Action: Case 2, Retriable Error 
        // Perform a single retry. If that fails, only try again after user action. 

        // Action: Case 1, Non-Retriable 
        // Do not perform an immediate retry. Only retry after user action. 
        // Example Errors: default case, DEVICE_CONNECTION_IS_NOT_AVAILABLE
    }
}
```

#### <a name="ios"></a>iOS

La guía siguiente proporciona ejemplos para el control de errores junto con todos los métodos ADAL de AcquireToken(…) no silenciosos. 

El código debería implementarse así:

```objc
[context acquireTokenWithResource:[ARGS], completionBlock:^(ADAuthenticationResult *result) {
    if (result.status == AD_FAILED) {
        if ([error.domain isEqualToString:ADAuthenticationErrorDomain]){
            // Exception: AD_FAILED 
            // Represents a library error generated by ADAL ObjC.
            // Error Code: result.error.code 

            // Error: AD_ERROR_SERVER_WRONG_USER
            // Description: App passed a user into ADAL and the end user signed in with a different account. 
            // Action: Case 1, Non-retriable (as is) and up to the application on how to handle this case. 
            // It can attempt a new request without specifying the user, or use UI to clarify the user account to sign in. 

            // Action: Case 1, Non-Retriable 
            // Do not perform an immediate retry. Only retry after user action. 
            // Example Errors: default case
        }
    }
}]
```

### <a name="error-cases-and-actionable-steps-web-applications-that-call-a-resource-api-net"></a>Casos de error y acciones recomendadas: Aplicaciones web que llaman a una API de recurso (.NET)

Si va a compilar una aplicación web de .NET que obtiene un token con un código de autorización para un recurso, el único código necesario es un controlador predeterminado para el caso genérico. 

La guía siguiente proporciona ejemplos para el control de errores junto con métodos de ADAL: 

- AcquireTokenByAuthorizationCodeAsync(…)

El código debería implementarse así:

```csharp
try {
    AcquireTokenByAuthorizationCodeAsync(…);
} 

catch (AdalException e) {
    // Exception: AdalException
    // Represents a library exception generated by ADAL .NET.
    // Error Code: e.ErrorCode

    // Action: Do not perform an immediate retry. Only try again after user action or wait until much later. 
    // Example Errors: default case
}
```

### <a name="error-cases-and-actionable-steps-single-page-applications-adaljs"></a>Casos de error y acciones recomendadas: Aplicaciones de página única (adal.js)

Si va a compilar una aplicación de una sola página mediante adal.js con AcquireToken, el código del control de errores es similar al de una llamada silenciosa típica. En concreto, en adal.js, AcquireToken nunca muestra la interfaz de usuario. 

Un error de AcquireToken tiene los casos siguientes:

|  |  |
|------|-------------|
| **Caso 1**:<br>Puede resolverse con una solicitud interactiva | 1. Si se produce un error en login(), no realice reintento inmediato. Solo vuelva a intentarlo después de que la acción de un usuario solicite un reintento.|
| **Caso 2**:<br>No puede resolverse con una solicitud interactiva. Puede realizarse un reintento tras el error. | 1. Realice un reintento único, ya que es posible que el usuario se encuentre en un estado que resulte en un estado correcto.<br><br>2. Si no se puede llevar a cabo el reintento, muéstrele al usuario final una acción basada en el error específico que pueda invocar un reintento ("Intente volver a iniciar sesión"). |
| **Caso 3**:<br>No puede resolverse con una solicitud interactiva. No puede realizarse ningún reintento tras el error. | 1. No realice ningún reintento inmediato. Muéstrele al usuario final una acción basada en el error específico que pueda invocar un reintento ("Intente volver a iniciar sesión"). |

El código debería implementarse así:

```javascript
AuthContext.acquireToken(…, function(error, errorDesc, token) {
    if (error || errorDesc) {
        // Represents any token acquisition failure that occurred. 
        // Error Code: error.indexOf("<ERROR_STRING>")

        // Errors: if (error.indexOf("interaction_required"))
        //         if (error.indexOf("login required"))
        // Description: ADAL wasn't able to silently acquire a token because of expire or fresh session. 
        // Action: Case 1, Resolvable with an interactive login() request. 

        // Error: if (error.indexOf("Token Renewal Failed")) 
        // Description: Timeout when refreshing the token.
        // Action: Case 2, Not resolvable interactively, error is retriable.
        // Perform a single retry. Only try again after user action.

        // Action: Case 3, Not resolvable interactively, error is not retriable. 
        // Do not perform an immediate retry. Only retry after user action.
        // Example Errors: default case
    }
}
```

### <a name="error-cases-and-actionable-steps-service-to-service-applications-net-only"></a>Casos de error y acciones recomendadas: aplicaciones de servicio a servicio (solo para .NET)

Si va a compilar una aplicación de servicio a servicio que utiliza AcquireToken, hay algunos errores clave que el código debe controlar. El único recurso ante un error es devolverlo a la aplicación que realiza la llamada (para casos de en nombre de) o aplicar una estrategia de reintento. 

#### <a name="all-scenarios"></a>Todas las situaciones

Para *todas* las situaciones de aplicación de servicio al servicio, incluso en nombre de:

- No realice ningún reintento inmediato. ADAL realiza un reintento único para determinadas solicitudes erróneas. 
- Solo continúe con el reintento después de que una acción del usuario o una aplicación lo solicite. Por ejemplo, una aplicación demonio que funcionan con un intervalo definido debe esperar hasta el siguiente intervalo para volver a intentarlo.

La guía siguiente proporciona ejemplos para el control de errores junto con métodos de ADAL: 

- AcquireTokenAsync(…, IClientAssertionCertification, …)
- AcquireTokenAsync(…,ClientCredential, …)
- AcquireTokenAsync(…,ClientAssertion, …)
- AcquireTokenAsync(…,UserAssertion, …)

El código debería implementarse así:

```csharp
try {
    AcquireTokenAsync(…);
} 

catch (AdalException e) {
    // Exception: AdalException
    // Represents a library exception generated by ADAL .NET.
    // Error Code: e.ErrorCode

    // Action: Do not perform an immediate retry. Only try again after user action (if applicable) or wait until much later. 
    // Example Errors: default case
}  
```

#### <a name="on-behalf-of-scenarios"></a>Situaciones en nombre de

Para situaciones de aplicaciones de servicio a servicio *en nombre de*.

La guía siguiente proporciona ejemplos para el control de errores junto con métodos de ADAL: 

- AcquireTokenAsync(…, UserAssertion, …)

El código debería implementarse así:

```csharp
try {
AcquireTokenAsync(…);
} 

catch (AdalServiceException e) {
    // Exception: AdalServiceException 
    // Represents an error produced by the STS. 
    // e.ErrorCode contains the error code and description, which can be used for debugging. 
    // NOTE: Do not code a dependency on the contents of the error description, as it can change over time.

    // Error: On-Behalf-Of Error Handler
    if (e.ErrorCode == "interaction_required") {
        // Description: The client needs to perform some action due to a config from admin. 
        // Action: Capture `claims` parameter inside ex.InnerException.InnerException.
        // Generate HTTP error 403 with claims, throw back HTTP error to client.
        // Wait for client to retry. 
    }
} 
        
catch (AdalException e) {
    // Exception: AdalException 
    // Represents a library exception generated by ADAL .NET.
    // Error Code: e.ErrorCode

    // Action: Do not perform an immediate retry. Only try again after user action (if applicable) or wait until much later. 
    // Example Error: default case
}
```

Hemos creado un [ejemplo completo](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof-ca) que ilustra esta situación.

## <a name="error-and-logging-reference"></a>Referencia de errores y registro

### <a name="logging-personal-identifiable-information-pii--organizational-identifiable-information-oii"></a>Registro de información de identificación personal (PII) e información de identificación organizativa (OII)
De forma predeterminada, el registro de ADAL no captura ni registrar información de identificación personal (PII) ni información de identificación organizativa (OII). La biblioteca permite a los desarrolladores de aplicaciones activar esto mediante un establecedor en la clase de registrador. Al activar PII o OII, la aplicación tiene la responsabilidad de controlar los datos confidenciales y cumplir con los requisitos normativos de forma segura.

### <a name="net"></a>.NET

#### <a name="adal-library-errors"></a>Errores de la biblioteca ADAL

Para explorar los errores específicos de ADAL, el código fuente en el [repositorio azure-activedirectory-library-for-dotnet](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/8f6d560fbede2247ec0e217a21f6929d4375dcaa/src/ADAL.PCL/Utilities/Constants.cs#L58) es la referencia de errores recomendada.

#### <a name="guidance-for-error-logging-code"></a>Instrucciones para el código de registro de errores

El registro de ADAL .NET cambia según la plataforma en la que se esté trabajando. Consulte [wiki de registro](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Logging-in-ADAL.Net) para obtener código sobre cómo habilitar el registro.

### <a name="android"></a>Android

#### <a name="adal-library-errors"></a>Errores de la biblioteca ADAL

Para explorar los errores específicos de ADAL, el código fuente en el [repositorio azure-activedirectory-library-for-android](https://github.com/AzureAD/azure-activedirectory-library-for-android/blob/dev/adal/src/main/java/com/microsoft/aad/adal/ADALError.java#L33) es la referencia de errores recomendada.

#### <a name="operating-system-errors"></a>Errores del sistema operativo

Los errores en el sistema operativo Android se muestran mediante una instancia de AuthenticationException en ADAL, pueden identificarse como "SERVER_INVALID_REQUEST" y pueden ser más detallados mediante sus descripciones. 

Para obtener una lista completa de los errores habituales y qué pasos debe realizar cuando su aplicación o los usuarios finales se encuentran con ellos, consulte [wiki de Android de ADAL](https://github.com/AzureAD/azure-activedirectory-library-for-android/wiki). 

#### <a name="guidance-for-error-logging-code"></a>Instrucciones para el código de registro de errores

```java
// 1. Configure Logger
Logger.getInstance().setExternalLogger(new ILogger() {    
    @Override   
    public void Log(String tag, String message, String additionalMessage, LogLevel level, ADALError errorCode) { 
    // …
    // You can write this to logfile depending on level or errorcode. 
    writeToLogFile(getApplicationContext(), tag +":" + message + "-" + additionalMessage);    
    }
}

// 2. Set the log level
Logger.getInstance().setLogLevel(Logger.LogLevel.Verbose);

// By default, the `Logger` does not capture any PII or OII

// PII or OII will be logged
Logger.getInstance().setEnablePII(true);

// To STOP logging PII or OII, use the following setter
Logger.getInstance().setEnablePII(false);


// 3. Send logs to logcat.
adb logcat > "C:\logmsg\logfile.txt";
```

### <a name="ios"></a>iOS

#### <a name="adal-library-errors"></a>Errores de la biblioteca ADAL

Para explorar los errores específicos de ADAL, el código fuente en el [repositorio azure-activedirectory-library-for-objc](https://github.com/AzureAD/azure-activedirectory-library-for-objc/blob/dev/ADAL/src/ADAuthenticationError.m#L295) es la referencia de errores recomendada.

#### <a name="operating-system-errors"></a>Errores del sistema operativo

Es posible que se produzcan errores de iOS durante el inicio de sesión cuando los usuarios usen vistas web y por la naturaleza de la autenticación. Esto puede deberse a condiciones como errores de SSL, tiempos de espera o errores de red:

- Par el uso compartido de derechos, los inicios de sesión no son persistentes y la memoria caché aparece vacía. Para resolver la situación, agregue la siguiente línea de código a la cadena de claves: `[[ADAuthenticationSettings sharedInstance] setSharedCacheKeychainGroup:nil];`
- Para el conjunto de errores de NsUrlDomain, la acción cambia dependiendo de la lógica de la aplicación. Consulte la [documentación de referencia de NSURLErrorDomain](https://developer.apple.com/documentation/foundation/nsurlerrordomain#declarations) para obtener instancias específicas que se pueden administrar.
- Vea [ADAL Obj-C Common Issues](https://github.com/AzureAD/azure-activedirectory-library-for-objc#adauthenticationerror) (problemas comunes en Objective-C de ADAL) para obtener la lista de errores comunes que mantiene el equipo de Objective-C de ADAL.

#### <a name="guidance-for-error-logging-code"></a>Instrucciones para el código de registro de errores

```objc
// 1. Enable NSLogging
[ADLogger setNSLogging:YES];

// 2. Set the log level (if you want verbose)
[ADLogger setLevel:ADAL_LOG_LEVEL_VERBOSE];

// 3. Set up a callback block to simply print out
[ADLogger setLogCallBack:^(ADAL_LOG_LEVEL logLevel, NSString *message, NSString *additionalInformation, NSInteger errorCode, NSDictionary *userInfo) {
     NSString* log = [NSString stringWithFormat:@"%@ %@", message, additionalInformation];    
     NSLog(@"%@", log);
}];
```

### <a name="guidance-for-error-logging-code---javascript"></a>Instrucciones para el código de registro de errores: JavaScript 

```javascript
0: Error1: Warning2: Info3: Verbose
window.Logging = {
    level: 3,
    log: function (message) {
        console.log(message);
    }
};
```
## <a name="related-content"></a>Contenido relacionado

* [Guía del desarrollador de Azure AD][AAD-Dev-Guide]
* [Bibliotecas de autenticación de Azure AD][AAD-Auth-Libraries]
* [Escenarios de autenticación de Azure AD][AAD-Auth-Scenarios]
* [Integración de aplicaciones con Azure Active Directory][AAD-Integrating-Apps]

Use la siguiente sección de comentarios para proporcionar sus opiniones y ayudarnos a refinar y remodelar nuestro contenido.

[![Botón de inicio de sesión][AAD-Sign-In]][AAD-Sign-In]
<!--Reference style links -->

[AAD-Auth-Libraries]: ./active-directory-authentication-libraries.md
[AAD-Auth-Scenarios]:authentication-scenarios.md
[AAD-Dev-Guide]:azure-ad-developers-guide.md
[AAD-Integrating-Apps]:quickstart-v1-integrate-apps-with-azure-ad.md
[AZURE-portal]: https://portal.azure.com

<!--Image references-->
[AAD-Sign-In]:./media/active-directory-devhowto-multi-tenant-overview/sign-in-with-microsoft-light.png

