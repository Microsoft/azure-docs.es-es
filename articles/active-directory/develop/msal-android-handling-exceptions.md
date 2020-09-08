---
title: Errores y excepciones (MSAL para Android) | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a controlar errores y excepciones, y obtenga más información sobre el acceso condicional y el desafío de las notificaciones en las aplicaciones de MSAL para Android.
services: active-directory
author: hamiltonha
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: troubleshooting
ms.workload: identity
ms.date: 08/07/2020
ms.author: hahamil
ms.reviewer: marsma
ms.openlocfilehash: c0b08a6c1a784216abe2bd562109dbb1586252c9
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119817"
---
# <a name="handle-exceptions-and-errors-in-msal-for-android"></a>Control de excepciones y errores en MSAL para Android

Las excepciones de la Biblioteca de autenticación de Microsoft (MSAL) están diseñadas para ayudar a los desarrolladores de aplicaciones a solucionar problemas de sus aplicaciones. Los mensajes de excepción no están traducidos.

Cuando procesa excepciones y errores, puede usar el propio tipo de excepción y el código de error para distinguir entre las excepciones.  Para ver una lista con los códigos de error, consulte [Códigos de error de autenticación y autorización](reference-aadsts-error-codes.md).

Durante la experiencia de inicio de sesión, puede encontrar errores de consentimiento, de acceso condicional (MFA, administración de dispositivos, restricciones basadas en la ubicación), de emisión y canje de tokens, y de las propiedades del usuario.


|Clase de error | Causa o cadena error| Realización del control |
|-----------|------------|----------------|
|`MsalUiRequiredException`| <ul><li>`INVALID_GRANT`: el token de actualización usado para canjear el token de acceso no es válido, ha expirado o se ha revocado. Esta excepción puede deberse a un cambio de contraseña. </li><li>`NO_TOKENS_FOUND`: el token de acceso no existe y no se puede encontrar ningún token de actualización para canjear el token de acceso.</li> <li>Se requiere una actualización a edición superior<ul><li>MFA</li><li>Faltan notificaciones</li></ul></li><li>Bloqueado por el acceso condicional (por ejemplo, instalación necesaria del [agente de autenticación](./brokered-auth.md))</li><li>`NO_ACCOUNT_FOUND`: No hay ninguna cuenta disponible en la memoria caché para la autenticación silenciosa.</li></ul> |Llame a `acquireToken()` para solicitar al usuario que escriba su nombre de usuario y contraseña y, posiblemente, dé su consentimiento y realice la autenticación multifactor.|
|`MsalDeclinedScopeException`|<ul><li>`DECLINED_SCOPE`: el usuario o el servidor no ha aceptado todos los ámbitos. El servidor puede rechazar un ámbito si el ámbito solicitado no es compatible, no se reconoce o no se admite para una cuenta determinada. </li></ul>| El desarrollador debe decidir si desea continuar con la autenticación con los ámbitos concedidos o finalizar el proceso de autenticación. Opción para volver a enviar la solicitud de adquisición de token solo para los ámbitos concedidos y proporcionar sugerencias para los permisos que se han concedido pasando `silentParametersForGrantedScopes` y llamando a `acquireTokenSilent`. |
|`MsalServiceException`|<ul><li>`INVALID_REQUEST`: en esta la solicitud falta un parámetro necesario, dicha solicitud incluye un parámetro no válido o un parámetro repetido, o bien no tiene el formato correcto. </li><li>`SERVICE_NOT_AVAILABLE`: representa los códigos de error 500/503/506 debido a que el servicio está inactivo. </li><li>`UNAUTHORIZED_REQUEST`: el cliente no está autorizado para solicitar un código de autorización.</li><li>`ACCESS_DENIED`: El propietario del recurso o el servidor de consentimiento rechazaron la solicitud.</li><li>`INVALID_INSTANCE`: error de validación de `AuthorityMetadata`</li><li>`UNKNOWN_ERROR`: no se pudo realizar la solicitud al servidor, pero el servicio no devuelve ningún error ni `error_description`.</li><ul>| Esta clase de excepción representa los errores al comunicarse con el servicio, puede ser desde los puntos de conexión de autorización o de token. MSAL lee el error y error_description de la respuesta del servidor. Normalmente, estos errores se resuelven corrigiendo las configuraciones de la aplicación en el código o en el portal de registro de aplicaciones. Rara vez una interrupción del servicio puede desencadenar esta advertencia, que solo se puede mitigar esperando a que el servicio se recupere.  |
|`MsalClientException`|<ul><li> `MULTIPLE_MATCHING_TOKENS_DETECTED`: se encontraron varias entradas de caché y el SDK no puede identificar el acceso correcto o el token de actualización de la memoria caché. Esta excepción suele indicar un error en el SDK para almacenar tokens o que la autoridad no se proporciona en la solicitud silenciosa y se encuentran varios tokens coincidentes. </li><li>`DEVICE_NETWORK_NOT_AVAILABLE`: no hay ninguna red activa disponible en el dispositivo. </li><li>`JSON_PARSE_FAILURE`: El SDK no pudo analizar el formato JSON.</li><li>`IO_ERROR`: se produjo `IOException`; podría ser un error de dispositivo o de red. </li><li>`MALFORMED_URL`: la dirección URL tiene un formato incorrecto. Probablemente se originó al construir la solicitud de autenticación, la autoridad o el URI de redirección. </li><li>`UNSUPPORTED_ENCODING`: el dispositivo no admite la codificación. </li><li>`NO_SUCH_ALGORITHM`: no se admite el algoritmo utilizado para generar el desafío [PKCE](https://tools.ietf.org/html/rfc7636). </li><li>`INVALID_JWT`: el elemento `JWT` devuelto por el servidor no es válido, está vacío o tiene un formato incorrecto. </li><li>`STATE_MISMATCH`: el estado de la respuesta de autorización no coincidía con el estado de la solicitud de autorización. En el caso de las solicitudes de autorización, el SDK comprobará el estado devuelto de la redirección y el que se envió en la solicitud. </li><li>`UNSUPPORTED_URL`: la dirección URL no es compatible, no se puede realizar la validación de la autoridad de ADFS. </li><li> `AUTHORITY_VALIDATION_NOT_SUPPORTED`: la autoridad no es compatible con la validación de autoridad. El SDK admite las autoridades de B2C, pero no la validación de tales autoridades. Solo se admitirá el host conocido. </li><li>`CHROME_NOT_INSTALLED`: Chrome no está instalado en el equipo. El SDK usa la pestaña de personalización de Chrome para las solicitudes de autorización si está disponible y se revertirá explorador Chrome. </li><li>`USER_MISMATCH`: el usuario proporcionado en la solicitud de adquisición del token no coincide con el usuario devuelto desde el servidor.</li></ul>|Esta clase de excepción representa los errores generales que son locales para la biblioteca. Estas excepciones se pueden controlar corrigiendo la solicitud.|
|`MsalUserCancelException`|<ul><li>`USER_CANCELED`: el usuario inició el flujo interactivo y, antes de recibir los tokens, canceló la solicitud. </li></ul>||
|`MsalArgumentException`|<ul><li>`ILLEGAL_ARGUMENT_ERROR_CODE`</li><li>`AUTHORITY_REQUIRED_FOR_SILENT`: se debe especificar la autoridad para `acquireTokenSilent`.</li></ul>|El desarrollador puede mitigar estos errores corrigiendo de argumentos y garantizando la actividad para la autenticación interactiva, la devolución de llamada de finalización, los ámbitos y una cuenta con un identificador válido.|


## <a name="catching-errors"></a>Almacenamiento en caché de los errores

En el fragmento de código siguiente se muestra un ejemplo de detección de errores en el caso de llamadas `acquireToken` silenciosas.

```java
/**
 * Callback used in for silent acquireToken calls.
 */
private SilentAuthenticationCallback getAuthSilentCallback() {
    return new SilentAuthenticationCallback() {

        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            Log.d(TAG, "Successfully authenticated");

            /* Successfully got a token, use it to call a protected resource - MSGraph */
            callGraphAPI(authenticationResult);
        }

        @Override
        public void onError(MsalException exception) {
            /* Failed to acquireToken */
            Log.d(TAG, "Authentication failed: " + exception.toString());
            displayError(exception);

            if (exception instanceof MsalClientException) {
                /* Exception inside MSAL, more info inside MsalError.java */
            } else if (exception instanceof MsalServiceException) {
                /* Exception when communicating with the STS, likely config issue */
            } else if (exception instanceof MsalUiRequiredException) {
                /* Tokens expired or no session, retry with interactive */
            }
        }
    };
}
```

## <a name="next-steps"></a>Pasos siguientes 

Obtenga más información sobre el [registro de errores](./msal-logging.md?tabs=android).