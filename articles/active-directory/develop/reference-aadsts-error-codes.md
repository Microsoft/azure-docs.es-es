---
title: Códigos de error de autenticación y autorización de Azure AD
description: Obtenga información sobre los códigos de error AADSTS que devuelve el servicio de token de seguridad (STS) de Azure AD.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: reference
ms.date: 04/30/2020
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 2d18a50a21c41830796c913a424707897d277218
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87026771"
---
# <a name="azure-ad-authentication-and-authorization-error-codes"></a>Códigos de error de autenticación y autorización de Azure AD

¿Busca información sobre los códigos de error AADSTS que devuelve el servicio de token de seguridad (STS) de Azure Active Directory? Lea este documento para ver las descripciones de los errores AADSTS, sus correcciones y algunas soluciones alternativas.

> [!NOTE]
> Esta información es preliminar y está sujeta a cambios. ¿Tiene preguntas y no encuentra lo que busca? Abra una incidencia en GitHub o consulte [Opciones de ayuda y soporte técnico para desarrolladores](active-directory-develop-help-support.md) para ver otras maneras de obtener ayuda y soporte técnico.
>
> Esta documentación se proporciona como una guía para desarrolladores y administradores, pero nunca la debe usar el cliente por sí solo. Los códigos de error están sujetos a cambio en cualquier momento con el fin de proporcionar mensajes de error más pormenorizados diseñados para ayudar al desarrollador mientras compila su aplicación. Las aplicaciones que tienen dependencia de números de código de error o texto se interrumpirán en el tiempo.

## <a name="handling-error-codes-in-your-application"></a>Manejo de los códigos de error en una aplicación

La [especificación OAuth2.0](https://tools.ietf.org/html/rfc6749#section-5.2) brinda orientación sobre cómo manejar los errores durante la autenticación con la parte `error` de la respuesta de error. 

A continuación se muestra una respuesta de error de ejemplo:

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://example.contoso.com/activity.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7", 
  "error_uri":"https://login.microsoftonline.com/error?code=70011"
}
```

| Parámetro         | Descripción    |
|-------------------|----------------|
| `error`       | Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y debe usarse para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación. No use nunca este campo para reaccionar ante un error en el código. |
| `error_codes` | Una lista de los códigos de error específicos de STS que pueden ayudar en los diagnósticos.  |
| `timestamp`   | La hora a la que se produjo el error. |
| `trace_id`    | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos. |
| `correlation_id` | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos entre componentes. |
| `error_uri` |  Un vínculo a la página de búsqueda de errores con información adicional sobre el error.  Esto solo es para uso del desarrollador, no lo presente a los usuarios.  Solo preséntelo cuando el sistema de búsqueda de errores tenga información adicional sobre el error (no todos los errores incluyen información adicional).|

El campo `error` tiene varios valores posibles. Revise los vínculos de documentación del protocolo y las especificaciones de OAuth 2.0 para más información sobre errores específicos (por ejemplo, `authorization_pending` en el [flujo de código del dispositivo](v2-oauth2-device-code.md)) y cómo reaccionar a ellos.  A continuación se enumeran algunos de los más comunes:

| Código de error         | Descripción        | Acción del cliente    |
|--------------------|--------------------|------------------|
| `invalid_request`  | Error de protocolo, como un parámetro obligatorio que falta. | Corrija el error y vuelva a enviar la solicitud.|
| `invalid_grant`    | Parte del material de autenticación (código de autenticación, token de actualización, token de acceso, desafío PKCE) no es válida, no se puede analizar, falta o no se puede volver a usar. | Pruebe una solicitud nueva al punto de conexión `/authorize` para obtener un código de autorización nuevo.  Considere la posibilidad de revisar y validar el uso de los protocolos de la aplicación. |
| `unauthorized_client` | El cliente autenticado no está autorizado para usar este tipo de concesión de autorización. | Este error suele producirse cuando la aplicación cliente no está registrada en Azure AD o no se ha agregado al inquilino de Azure AD del usuario. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| `invalid_client` | Se produjo un error de autenticación de cliente.  | Las credenciales del cliente no son válidas. Para corregirlo, el administrador de la aplicación actualiza las credenciales.   |
| `unsupported_grant_type` | El servidor de autorización no admite el tipo de concesión de autorización. | Cambie el tipo de concesión de la solicitud. Este tipo de error solo debe producirse durante el desarrollo y detectarse en las pruebas iniciales. |
| `invalid_resource` | El recurso de destino no es válido porque no existe, Azure AD no lo encuentra o no está configurado correctamente. | Este error indica que el recurso, en caso de que exista, no se ha configurado en el inquilino. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD.  Durante el desarrollo, esto suele indicar que se configuró de manera incorrecta un inquilino de prueba o que hay un error tipográfico en el nombre del ámbito que se solicita. |
| `interaction_required` | La solicitud requiere la interacción del usuario. Por ejemplo, hay que realizar un paso de autenticación más. | Vuelva a intentar la solicitud con el mismo recurso, de manera interactiva, para que el usuario pueda completar los desafíos necesarios.  |
| `temporarily_unavailable` | De manera temporal, el servidor está demasiado ocupado para atender la solicitud. | Vuelva a intentarlo. La aplicación podría explicar al usuario que su respuesta se retrasó debido a una condición temporal. |

## <a name="lookup-current-error-code-information"></a>Búsqueda de información actual sobre códigos de error
Los códigos y los mensajes de error están sujetos a cambios.  Para tener la información más actualizada, eche un vistazo a la página [https://login.microsoftonline.com/error](https://login.microsoftonline.com/error) para encontrar descripciones de errores de AADSTS, correcciones y algunas soluciones recomendadas.  

Busque en la parte numérica del código de error devuelto.  Por ejemplo, si ha recibido el código de error "AADSTS16000", busque "16000" en [https://login.microsoftonline.com/error](https://login.microsoftonline.com/error).  También puede agregar el número de código de error a la dirección URL: [https://login.microsoftonline.com/error?code=16000](https://login.microsoftonline.com/error?code=16000) para crear un vínculo directo a un error específico.

## <a name="aadsts-error-codes"></a>Códigos de error AADSTS

| Error | Descripción |
|---|---|
| AADSTS16000 | SelectUserAccount: esta es una interrupción iniciada por Azure AD, que hace que la interfaz de usuario permita al usuario elegir entre varias sesiones SSO válidas. Este error es bastante habitual y puede devolverse a la aplicación si se especifica `prompt=none`. |
| AADSTS16001 | UserAccountSelectionInvalid: verá este error si el usuario hace clic en un icono que indica que la lógica de selección de sesión se ha rechazado. Cuando se desencadena, este error permite al usuario solucionarlo; para ello, selecciona en una lista de iconos o sesiones, o elige otra cuenta. Este error puede producirse debido a una condición de carrera o a defectos de código. |
| AADSTS16002 | AppSessionSelectionInvalid: no se cumplió el requisito de SID especificado por la aplicación.  |
| AADSTS16003 | SsoUserAccountNotFoundInResourceTenant: indica que el usuario no se ha agregado explícitamente al inquilino. |
| AADSTS17003 | CredentialKeyProvisioningFailed: Azure AD no puede aprovisionar la clave de usuario. |
| AADSTS20001 | WsFedSignInResponseError: hay un problema con el proveedor de identidades federado. Póngase en contacto con el IDP para resolver este problema. |
| AADSTS20012 | WsFedMessageInvalid: hay un problema con el proveedor de identidades federado. Póngase en contacto con el IDP para resolver este problema. |
| AADSTS20033 | FedMetadataInvalidTenantName: hay un problema con el proveedor de identidades federado. Póngase en contacto con el IDP para resolver este problema. |
| AADSTS40008 | OAuth2IdPUnretryableServerError: hay un problema con el proveedor de identidades federado. Póngase en contacto con el IDP para resolver este problema. |
| AADSTS40009 | OAuth2IdPRefreshTokenRedemptionUserError: hay un problema con el proveedor de identidades federado. Póngase en contacto con el IDP para resolver este problema. |
| AADSTS40010 | OAuth2IdPRetryableServerError: hay un problema con el proveedor de identidades federado. Póngase en contacto con el IDP para resolver este problema. |
| AADSTS40015 | OAuth2IdPAuthCodeRedemptionUserError: hay un problema con el proveedor de identidades federado. Póngase en contacto con el IDP para resolver este problema. |
| AADSTS50000 | TokenIssuanceError: hay un problema con el servicio de inicio de sesión. Abra un [vale de soporte](../fundamentals/active-directory-troubleshooting-support-howto.md) para resolver este problema. |
| AADSTS50001 | InvalidResource: el recurso está deshabilitado o no existe. Compruebe el código de la aplicación para asegurarse de que ha especificado la URL exacta del recurso al que está intentando acceder.  |
| AADSTS50002 | NotAllowedTenant: no se pudo iniciar sesión debido a un acceso de proxy restringido en el inquilino. Si se trata de su propia directiva de inquilino, puede cambiar la configuración restringida del inquilino para corregir este problema. |
| AADSTS50003 | MissingSigningKey: no se pudo iniciar sesión debido a que falta la clave de firma o el certificado. Esto se puede deber a que no había ninguna clave de firma configurada en la aplicación. Compruebe las soluciones detalladas en [https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#certificate-or-key-not-configured](https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#certificate-or-key-not-configured). Si siguen los problemas, póngase en contacto con el propietario o el administrador de la aplicación. |
| AADSTS50005 | DevicePolicyError: el usuario intentó iniciar sesión en un dispositivo desde una plataforma que no es compatible mediante una directiva de acceso condicional. |
| AADSTS50006 | InvalidSignature: no se pudo comprobar la firma debido a una firma no válida. |
| AADSTS50007 | PartnerEncryptionCertificateMissing: no se encontró el certificado de cifrado de asociado para esta aplicación. [Abra una incidencia de soporte técnico](../fundamentals/active-directory-troubleshooting-support-howto.md) en Microsoft para solucionar este problema. |
| AADSTS50008 | InvalidSamlToken: la aserción SAML falta o está mal configurada en el token. Póngase en contacto con el proveedor de federación. |
| AADSTS50010 | AudienceUriValidationFailed: se ha producido un error en la validación del identificador URI de audiencia de la aplicación debido a que no se configuraron audiencias de token. |
| AADSTS50011 | InvalidReplyTo: la dirección de respuesta falta, está mal configurada o no coincide con las direcciones de respuesta configuradas para la aplicación.  Como resolución, asegúrese de agregar esta dirección de respuesta omitida a la aplicación de Azure Active Directory o hacer que alguien que cuenta con los permisos para administrar la aplicación en Active Directory lo haga por usted.|
| AADSTS50012 | AuthenticationFailed: se ha producido un error de autenticación por uno de los motivos siguientes:<ul><li>El nombre del titular del certificado de firma no está autorizado.</li><li>No se encontró la directiva de autoridad de confianza correspondiente al nombre del titular autorizado.</li><li>La cadena de certificados no es válida.</li><li>El certificado de firma no es válido.</li><li>La directiva no está configurada en el inquilino.</li><li>La huella digital del certificado de firma no está autorizada.</li><li>La aserción de cliente contiene una firma no válida.</li></ul> |
| AADSTS50013 | InvalidAssertion: la aserción no es válida debido a varias razones: el emisor del token no coincide con la versión de la API en su intervalo de tiempo válido, la aserción ha expirado o tiene un formato incorrecto, o el token de actualización de la aserción no es un token de actualización principal. |
| AADSTS50014 | GuestUserInPendingState: el estado de la amortización del usuario es pendiente. La creación de la cuenta de usuario invitado aún no ha finalizado. |
| AADSTS50015 | ViralUserLegalAgeConsentRequiredState: el usuario requiere el consentimiento de grupo de edad legal. |
| AADSTS50017 | CertificateValidationFailed: no se pudo realizar la validación de la certificación por alguno de los siguientes motivos:<ul><li>No se pudo encontrar el certificado de emisión en la lista de certificados de confianza</li><li>No se pudo encontrar el CrlSegment esperado</li><li>No se pudo encontrar el certificado de emisión en la lista de certificados de confianza</li><li>El punto de distribución de la diferencia CRL se ha configurado sin el punto de distribución de CRL correspondiente</li><li>No se pueden recuperar segmentos CRL válidos debido a un problema de tiempo de espera</li><li>No se puede descargar el CRL</li></ul>Póngase en contacto con el administrador del inquilino. |
| AADSTS50020 | UserUnauthorized: los usuarios no tiene autorización para llamar a este punto de conexión. |
| AADSTS50027 | InvalidJwtToken: token JWT no válido debido a los siguientes motivos:<ul><li>no contiene la notificación nonce, sub notificación</li><li>no coincide el identificador de asunto</li><li>notificación duplicada en las notificaciones de idToken</li><li>emisor inesperado</li><li>audiencia inesperada</li><li>fuera de su intervalo de tiempo válido </li><li>el formato del token no es correcto</li><li>El token del identificador externo del emisor no pudo comprobar la firma.</li></ul> |
| AADSTS50029 | Identificador URI no válido: el nombre de dominio contiene caracteres no válidos. Póngase en contacto con el administrador del inquilino. |
| AADSTS50032 | WeakRsaKey: indica un intento erróneo del usuario para usar una clave RSA débil. |
| AADSTS50033 | RetryableError: indica un error transitorio no relacionado con las operaciones de base de datos. |
| AADSTS50034 | UserAccountNotFound: para iniciar sesión en esta aplicación, la cuenta debe agregarse al directorio. |
| AADSTS50042 | UnableToGeneratePairwiseIdentifierWithMissingSalt: falta la sal necesaria para generar un identificador en pares en la entidad de seguridad. Póngase en contacto con el administrador del inquilino. |
| AADSTS50043 | UnableToGeneratePairwiseIdentifierWithMultipleSalts |
| AADSTS50048 | SubjectMismatchesIssuer: el asunto no coincide con la notificación del emisor en la aserción de cliente. Póngase en contacto con el administrador del inquilino. |
| AADSTS50049 | NoSuchInstanceForDiscovery: instancia desconocida o no válida. |
| AADSTS50050 | MalformedDiscoveryRequest: la solicitud no está correctamente formada. |
| AADSTS50053 | IdsLocked: la cuenta está bloqueada porque el usuario intentó iniciar sesión demasiadas veces con un identificador de usuario o contraseña incorrectos. |
| AADSTS50055 | InvalidPasswordExpiredPassword: la contraseña ha expirado. |
| AADSTS50056 | Contraseña no válida o nula: la contraseña no existe en el almacén de este usuario. |
| AADSTS50057 | UserDisabled: la cuenta de usuario está deshabilitada. Un administrador ha deshabilitado la cuenta. |
| AADSTS50058 | UserInformationNotProvided: esto significa que un usuario no ha iniciado sesión. Este es un error común que se espera cuando un usuario no está autenticado y aún no ha iniciado sesión.</br>Si este error se produce en un contexto de inicio de sesión único en el que el usuario ha iniciado sesión anteriormente, significa que la sesión SSO no se encuentra o no es válida.</br>Este error puede devolverse a la aplicación si se especifica prompt=none. |
| AADSTS50059 | MissingTenantRealmAndNoUserInformationProvided: no se ha encontrado ninguna información de identificación del inquilino ni en la solicitud ni implícita en ninguna credencial proporcionada. El usuario puede ponerse en contacto con el administrador del inquilino para ayudar a resolver el problema. |
| AADSTS50061 | SignoutInvalidRequest: la solicitud de cierre de sesión no es válida. |
| AADSTS50064 | CredentialAuthenticationError: error de validación de credenciales en el nombre de usuario o la contraseña. |
| AADSTS50068 | SignoutInitiatorNotParticipant: error de cierre de sesión. La aplicación que inició el cierre de sesión no participa en la sesión actual. |
| AADSTS50070 | SignoutUnknownSessionIdentifier: error de cierre de sesión. La solicitud de cierre de sesión especificó un identificador de nombre que no coincidía con las sesiones existentes. |
| AADSTS50071 | SignoutMessageExpired: la solicitud de inicio de sesión ha expirado. |
| AADSTS50072 | UserStrongAuthEnrollmentRequiredInterrupt: el usuario debe inscribirse para el segundo factor de autenticación (interactivo). |
| AADSTS50074 | UserStrongAuthClientAuthNRequiredInterrupt: la autenticación segura es obligatoria y el usuario no ha superado el desafío de la autenticación multifactor. |
| AADSTS50076 | UserStrongAuthClientAuthNRequired: debido a un cambio de configuración realizado por el administrador, o porque usted se ha trasladado a una nueva ubicación, el usuario debe usar la autenticación multifactor para tener acceso al recurso. Vuelva a intentarlo con una nueva solicitud de autorización para el recurso. |
| AADSTS50079 | UserStrongAuthEnrollmentRequired: debido a un cambio de configuración realizado por el administrador, o porque el usuario se ha trasladado a una nueva ubicación, el usuario debe usar la autenticación multifactor. |
| AADSTS50085 | El token de actualización necesita un inicio de sesión de IDP social. Haga que el usuario intente iniciar sesión de nuevo con el nombre de usuario y la contraseña |
| AADSTS50086 | SasNonRetryableError |
| AADSTS50087 | SasRetryableError: el servicio no está disponible temporalmente. Inténtelo de nuevo. |
| AADSTS50089 | El token de flujo ha expirado: error de autenticación. Haga que el usuario intente iniciar sesión de nuevo con el nombre de usuario y la contraseña. |
| AADSTS50097 | DeviceAuthenticationRequired: la autenticación de dispositivo es obligatoria. |
| AADSTS50099 | PKeyAuthInvalidJwtUnauthorized: la firma JWT no es válida. |
| AADSTS50105 | EntitlementGrantsNotFound: el usuario que ha iniciado sesión no está asignado a un rol de la aplicación en la que inició sesión. Asigne el usuario a la aplicación. Para más información:[https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#user-not-assigned-a-role](https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#user-not-assigned-a-role). |
| AADSTS50107 | InvalidRealmUri: el objeto solicitado del dominio Kerberos de federación no existe. Póngase en contacto con el administrador del inquilino. |
| AADSTS50120 | ThresholdJwtInvalidJwtFormat: hay un problema con el encabezado JWT. Póngase en contacto con el administrador del inquilino. |
| AADSTS50124 | ClaimsTransformationInvalidInputParameter: la transformación de notificaciones contiene parámetros de entrada no válidos. Póngase en contacto con el administrador del inquilino para actualizar la directiva. |
| AADSTS50125 | PasswordResetRegistrationRequiredInterrupt: el inicio de sesión se interrumpió debido a una entrada de registro de contraseña o restablecimiento de contraseña. |
| AADSTS50126 | InvalidUserNameOrPassword: se ha producido un error al validar las credenciales debido a un nombre de usuario o contraseña no válidos. |
| AADSTS50127 | BrokerAppNotInstalled: el usuario necesita instalar una aplicación de agente para acceder a este contenido. |
| AADSTS50128 | Nombre de dominio no válido: No se ha encontrado ninguna información de identificación del inquilino en la solicitud ni implícita en ninguna credencial proporcionada. |
| AADSTS50129 | DeviceIsNotWorkplaceJoined: es obligatorio unirse al área de trabajo para registrar el dispositivo. |
| AADSTS50131 | ConditionalAccessFailed: indica varios errores de acceso condicional tales como mal estado del dispositivo Windows, solicitud bloqueada debido a actividad sospechosa, directiva de acceso o decisiones de directiva de seguridad. |
| AADSTS50132 | SsoArtifactInvalidOrExpired: la sesión no es válida debido a la expiración de la contraseña o a un cambio reciente de contraseña. |
| AADSTS50133 | SsoArtifactRevoked: la sesión no es válida debido a la expiración de la contraseña o a un cambio reciente de contraseña. |
| AADSTS50134 | DeviceFlowAuthorizeWrongDatacenter: centro de datos incorrecto. Para autorizar una solicitud iniciada por una aplicación en el flujo de dispositivo de OAuth 2.0, la entidad encargada de autorizar debe estar en el mismo centro de datos donde reside la solicitud original. |
| AADSTS50135 | PasswordChangeCompromisedPassword: es obligatorio cambiar la contraseña debido al riesgo de la cuenta. |
| AADSTS50136 | RedirectMsaSessionToApp: se ha detectado una única sesión de MSA. |
| AADSTS50139 | SessionMissingMsaOAuth2RefreshToken: la sesión no es válida debido a un token de actualización externo que falta. |
| AADSTS50140 | KmsiInterrupt: este error se ha producido debido a una interrupción en "Mantener la sesión iniciada" cuando el usuario estaba iniciando sesión. [Abra un vale de soporte](../fundamentals/active-directory-troubleshooting-support-howto.md) con el identificador de correlación, el de solicitud y el código de error para conocer más detalles. |
| AADSTS50143 | Error de coincidencia de sesión: la sesión no es válida porque el inquilino del usuario no coincide con la sugerencia de dominio debido a un recurso diferente.  [Abra una incidencia de soporte técnico](../fundamentals/active-directory-troubleshooting-support-howto.md) con el identificador de correlación, el identificador de solicitud y el código de error para más detalles. |
| AADSTS50144 | InvalidPasswordExpiredOnPremPassword: la contraseña de Active Directory del usuario ha expirado. Genere una nueva contraseña para el usuario o solicite al usuario que restablezca la contraseña con la herramienta de autoservicio de restablecimiento. |
| AADSTS50146 | MissingCustomSigningKey: es necesario que esta aplicación esté configurada con una clave de firma específica de la aplicación. En este momento no tiene configurada ninguna o ha expirado o ya no es válida. |
| AADSTS50147 | MissingCodeChallenge: el tamaño del parámetro de desafío de código no es válido. |
| AADSTS50155 | DeviceAuthenticationFailed: error de autenticación del dispositivo para este usuario. |
| AADSTS50158 | ExternalSecurityChallenge: no se cumplió el desafío de seguridad externo. |
| AADSTS50161 | InvalidExternalSecurityChallengeConfiguration: las notificaciones enviadas por el proveedor externo no son suficientes o falta la notificación que se solicitó al proveedor externo. |
| AADSTS50166 | ExternalClaimsProviderThrottled: no se pudo enviar la solicitud al proveedor de notificaciones. |
| AADSTS50168 | ChromeBrowserSsoInterruptRequired: el cliente es capaz de obtener un token de inicio de sesión único mediante la extensión Cuentas de Windows 10, pero no se encontró el token en la solicitud o el token proporcionado ha expirado. |
| AADSTS50169 | InvalidRequestBadRealm: el dominio Kerberos no es un dominio configurado del espacio de nombres de servicio actual. |
| AADSTS50170 | MissingExternalClaimsProviderMapping: falta la asignación de controles externos. |
| AADSTS50177 | ExternalChallengeNotSupportedForPassthroughUsers: no se admite el desafío externo para los usuarios de acceso directo. |
| AADSTS50178 | SessionControlNotSupportedForPassthroughUsers: no se admite el control de sesión para los usuarios de acceso directo. |
| AADSTS50180 | WindowsIntegratedAuthMissing: es obligatoria la autenticación de Windows integrada. Habilite el inquilino para un inicio de sesión único de conexión directa. |
| AADSTS50187 | DeviceInformationNotProvided: el servicio no pudo realizar la autenticación de dispositivos. |
| AADSTS50196 | LoopDetected: se ha detectado un bucle de cliente. Compruebe la lógica de la aplicación para asegurarse de que el almacenamiento en caché de tokens está implementado y de que las condiciones de error se controlan correctamente.  La aplicación ha realizado demasiadas veces la misma solicitud en un período demasiado corto, lo que indica que se encuentra en un estado defectuoso o que solicita tokens de forma abusiva. |
| AADSTS50197 | ConflictingIdentities: no se encontró el usuario. Intente iniciar sesión de nuevo. |
| AADSTS50199 | CmsiInterrupt: por razones de seguridad, se requiere la confirmación del usuario para esta solicitud.  Dado que se trata de un error "interaction_required", el cliente debe realizar la autenticación interactiva.  Esto se debe a que se ha usado una vista web del sistema para solicitar un token para una aplicación nativa. Se debe pedir al usuario que pregunte si realmente se trata de la aplicación en la que pretendía iniciar sesión. Para evitar que esto aparezca, el URI de redirección debe formar parte de la siguiente lista segura: <br />http://<br />https://<br />msauth://(solo iOS)<br />msauthv2://(solo iOS)<br />chrome-extension:// (solo explorador Chrome de escritorio) |
| AADSTS51000 | RequiredFeatureNotEnabled: la característica está deshabilitada. |
| AADSTS51001 | DomainHintMustbePresent: la sugerencia de dominio debe estar presente con el identificador de seguridad local o UPN local. |
| AADSTS51004 | UserAccountNotInDirectory: la cuenta de usuario no existe en el directorio. |
| AADSTS51005 | TemporaryRedirect: equivalente al código de estado HTTP 307, que indica que la información solicitada se encuentra en el URI especificado en el encabezado location. Cuando reciba este estado, siga el encabezado location asociado a la respuesta. Si el método de solicitud original era POST, la solicitud redirigida también utilizará el método POST. |
| AADSTS51006 | ForceReauthDueToInsufficientAuth: se necesita la autenticación de Windows integrada. El usuario ha iniciado sesión con un token de sesión al que le falta la notificación de autenticación de Windows integrada. Solicite al usuario que vuelva a iniciar sesión. |
| AADSTS52004 | DelegationDoesNotExistForLinkedIn: el usuario no ha proporcionado consentimiento para acceder a los recursos de LinkedIn. |
| AADSTS53000 | DeviceNotCompliant: la directiva de acceso condicional requiere un dispositivo compatible, y el dispositivo no lo es. El usuario debe inscribir su dispositivo con un proveedor de administración de datos maestros aprobado, como Intune. |
| AADSTS53001 | DeviceNotDomainJoined: la directiva de acceso condicional requiere un dispositivo unido a un dominio y este no lo está. Haga que el usuario utilice un dispositivo unido a un dominio. |
| AADSTS53002 | ApplicationUsedIsNotAnApprovedApp: la aplicación utilizada no es una aplicación aprobada para el acceso condicional. Para poder acceder, el usuario tiene que usar una de las aplicaciones de la lista de aplicaciones aprobadas. |
| AADSTS53003 | BlockedByConditionalAccess: se ha bloqueado el acceso a las directivas de acceso condicional. La directiva de acceso no admite la emisión de tokens. |
| AADSTS53004 | ProofUpBlockedDueToRisk: el usuario tiene que completar el proceso de registro de la autenticación multifactor antes de acceder a este contenido. El usuario se debe registrar en la autenticación multifactor. |
| AADSTS54000 | MinorUserBlockedLegalAgeGroupRule |
| AADSTS65001 | DelegationDoesNotExist: el usuario o el administrador no han dado su consentimiento para usar la aplicación con el identificador X. Envíe una solicitud de autorización interactiva para este usuario y recurso. |
| AADSTS65004 | UserDeclinedConsent: el usuario no ha dado su consentimiento para acceder a la aplicación. Haga que el usuario intente iniciar sesión de nuevo y dé el consentimiento para acceder a la aplicación|
| AADSTS65005 | MisconfiguredApplication: la lista de acceso a los recursos necesarios de la aplicación no contiene aplicaciones reconocibles por el recurso, o la aplicación cliente ha solicitado el acceso a un recurso que no se especificó en su lista de acceso a recursos requeridos, o el servicio Graph ha devuelto una solicitud incorrecta o no se encontró el recurso. Si la aplicación admite SAML, es posible que haya configurado la aplicación con el identificador incorrecto (entidad). Pruebe la solución indicada para SAML mediante el siguiente vínculo: [https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery#no-resource-in-requiredresourceaccess-list](https://docs.microsoft.com/azure/active-directory/application-sign-in-problem-federated-sso-gallery?/?WT.mc_id=DMC_AAD_Manage_Apps_Troubleshooting_Nav) |
| AADSTS650052 | La aplicación necesita acceso a un servicio `(\"{name}\")` que su organización `\"{organization}\"` no ha habilitado o al que no se ha suscrito. Póngase en contacto con el administrador de TI para revisar la configuración de las suscripciones de servicio. |
| AADSTS67003 | ActorNotValidServiceIdentity |
| AADSTS70000 | InvalidGrant: se ha producido un error de autenticación de cliente. El token de actualización no es válido. El error puede deberse a los siguientes motivos:<ul><li>El encabezado del enlace de token está vacío.</li><li>El hash del enlace de token no coincide.</li></ul> |
| AADSTS70001 | UnauthorizedClient: la aplicación está deshabilitada. |
| AADSTS70002 | InvalidClient: error al validar las credenciales. El valor de client_secret especificado no coincide con el esperado para este cliente. Corrija el valor de client_secret e inténtelo de nuevo. Para más información, consulte [Uso del código de autorización para solicitar un token de acceso](v2-oauth2-auth-code-flow.md#request-an-access-token). |
| AADSTS70003 | UnsupportedGrantType: la aplicación ha devuelto un tipo de concesión no admitido. |
| AADSTS70004 | InvalidRedirectUri: la aplicación ha devuelto un URI de redireccionamiento no válido. La dirección de redireccionamiento que ha especificado el cliente no coincide con ninguna de las direcciones configuradas ni con ninguna de las de la lista de direcciones aprobadas de OIDC. |
| AADSTS70005 | UnsupportedResponseType: la aplicación ha devuelto un tipo de respuesta no admitido por alguno de los siguientes motivos:<ul><li>el tipo de respuesta "token" no está habilitado para la aplicación</li><li>el tipo de respuesta "id_token" requiere el ámbito "OpenID", o contiene un valor de parámetro de OAuth no admitido en el wctx codificado</li></ul> |
| AADSTS70007 | UnsupportedResponseMode: la aplicación ha devuelto un valor no admitido de `response_mode` al solicitar un token.  |
| AADSTS70008 | ExpiredOrRevokedGrant: el token de actualización ha expirado debido por inactividad. El token se emitió en XXX y estuvo inactivo durante un período de tiempo. |
| AADSTS70011 | InvalidScope: el ámbito solicitado por la aplicación no es válido. |
| AADSTS70012 | MsaServerError: se ha producido un error de servidor al autenticar un usuario (consumidor) de MSA. Inténtelo de nuevo. Si el error persiste, [abra una incidencia de soporte técnico](../fundamentals/active-directory-troubleshooting-support-howto.md) |
| AADSTS70016 | AuthorizationPending: error de flujo de dispositivo de OAuth 2.0. La autorización está pendiente. El dispositivo volverá a intentar sondear la solicitud. |
| AADSTS70018 | BadVerificationCode: el código de verificación no es válido porque el usuario ha escrito un código de usuario incorrecto en el flujo de códigos del dispositivo. No se ha aprobado la autorización. |
| AADSTS70019 | CodeExpired: el código de verificación ha expirado. Pida al usuario que vuelva a iniciar sesión. |
| AADSTS75001 | BindingSerializationError: se ha producido un error durante el enlace de mensajes de SAML. |
| AADSTS75003 | UnsupportedBindingError: la aplicación ha devuelto un error relacionado con un enlace no admitido (la respuesta del protocolo SAML no se puede enviar a través de enlaces que no sean HTTP POST). |
| AADSTS75005 | Saml2MessageInvalid: Azure AD no admite la solicitud SAML enviada por la aplicación para el inicio de sesión único. |
| AADSTS7500514 | No se encontró ningún tipo admitido de respuesta SAML. Los tipos de respuesta admitidos son "Response" (en el espacio de nombres XML "urn:oasis:names:tc:SAML:2.0:protocol") o "Assertion" (en el espacio "urn:oasis:names:tc:SAML:2.0:assertion"). Error de aplicación: el desarrollador controlará este error.|
| AADSTS75008 | RequestDeniedError: se ha rechazado la solicitud de la aplicación ya que la solicitud SAML tenía un destino inesperado. |
| AADSTS75011 | NoMatchedAuthnContextInOutputClaims: el método de autenticación mediante el cual se autenticó el usuario en el servicio no coincide con el método de autenticación solicitado. |
| AADSTS75016 | Saml2AuthenticationRequestInvalidNameIDPolicy: la solicitud de autenticación de SAML2 tiene un valor de NameIdPolicy no válido. |
| AADSTS80001 | OnPremiseStoreIsNotAvailable: el agente de autenticación no puede conectarse a Active Directory. Asegúrese de que los servidores del agente sean miembros del mismo bosque de AD que los usuarios cuyas contraseñas haya que validar y que pueden conectarse a Active Directory. |
| AADSTS80002 | OnPremisePasswordValidatorRequestTimedout: la solicitud de validación de la contraseña ha agotado el tiempo de espera. Asegúrese de que Active Directory está disponible y responde a las solicitudes de los agentes. |
| AADSTS80005 | OnPremisePasswordValidatorUnpredictableWebException: se ha producido un error desconocido al procesar la respuesta del agente de autenticación. Vuelva a intentarlo. Si el error continúa, [abra una incidencia de soporte técnico](../fundamentals/active-directory-troubleshooting-support-howto.md) para obtener más información. |
| AADSTS80007 | OnPremisePasswordValidatorErrorOccurredOnPrem: el agente de autenticación no puede validar la contraseña del usuario. Compruebe los registros del agente para más información y asegúrese de que Active Directory está funcionando según lo previsto. |
| AADSTS80010 | OnPremisePasswordValidationEncryptionException: el agente de autenticación no puede descifrar la contraseña. |
| AADSTS80012 | OnPremisePasswordValidationEncryptionException: los usuarios han intentado iniciar sesión fuera de las horas permitidas (esto se especifica en Active Directory). |
| AADSTS80013 | OnPremisePasswordValidationTimeSkew: no se pudo completar el intento de autenticación debido al desfase horario entre la máquina que ejecuta el agente de autenticación y AD. Corrija los problemas de sincronización de tiempo. |
| AADSTS81004 | DesktopSsoIdentityInTicketIsNotAuthenticated: error durante el intento de autenticación de Kerberos. |
| AADSTS81005 | DesktopSsoAuthenticationPackageNotSupported: no se admite el paquete de autenticación. |
| AADSTS81006 | DesktopSsoNoAuthorizationHeader: no se encontró ningún encabezado de autorización. |
| AADSTS81007 | DesktopSsoTenantIsNotOptIn: el inquilino no está habilitado para SSO de conexión directa. |
| AADSTS81009 | DesktopSsoAuthorizationHeaderValueWithBadFormat: no se puede validar el vale de Kerberos del usuario. |
| AADSTS81010 | DesktopSsoAuthTokenInvalid: error de SSO de conexión directa porque el vale de Kerberos del usuario ha expirado o no es válido. |
| AADSTS81011 | DesktopSsoLookupUserBySidFailed: no se encuentra el objeto de usuario con la información del vale de Kerberos del usuario. |
| AADSTS81012 | DesktopSsoLookupUserBySidFailed: el usuario que intenta iniciar sesión en Azure AD es distinto del que inició sesión en el dispositivo. |
| AADSTS90002 | InvalidTenantName: no se encontró el nombre del inquilino en el almacén de datos. Compruebe que tiene el identificador de inquilino correcto. |
| AADSTS90004 | InvalidRequestFormat: el formato de la solicitud es incorrecto. |
| AADSTS90005 | InvalidRequestWithMultipleRequirements: no se puede completar la solicitud. La solicitud no es válida porque el identificador y la sugerencia de inicio de sesión no pueden usarse juntos.  |
| AADSTS90006 | ExternalServerRetryableError: el servicio no está disponible temporalmente.|
| AADSTS90007 | InvalidSessionId: solicitud incorrecta. El identificador de sesión aprobado no se puede analizar. |
| AADSTS90008 | TokenForItselfRequiresGraphPermission: el usuario o administrador no ha dado su consentimiento para usar la aplicación. Como mínimo, la aplicación requiere acceso a Azure AD especificando el permiso de inicio de sesión y lectura del perfil de usuario. |
| AADSTS90009 | TokenForItselfMissingIdenticalAppIdentifier: la aplicación solicita un token por sí misma. Este escenario se admite solamente si el recurso especificado usa el identificador de aplicación basado en GUID. |
| AADSTS90010 | NotSupported: no se puede crear el algoritmo. |
| AADSTS90012 | RequestTimeout: se superó el tiempo de espera de la solicitud. |
| AADSTS90013 | InvalidUserInput: la entrada del usuario no es válida. |
| AADSTS90014 | MissingRequiredField: este código de error puede aparecer en varios casos cuando un campo esperado no está presente en la credencial. |
| AADSTS90015 | QueryStringTooLong: la cadena de consulta es demasiado larga. |
| AADSTS90016 | MissingRequiredClaim: el token de acceso no es válido. Falta la notificación necesaria. |
| AADSTS90019 | MissingTenantRealm: Azure AD no pudo determinar el identificador de inquilino en la solicitud. |
| AADSTS90022 | AuthenticatedInvalidPrincipalNameFormat: el formato de nombre de la entidad de seguridad no es válido o no cumple con el formato `name[/host][@realm]` esperado. El nombre de la entidad de seguridad es obligatorio; el host y el dominio Kerberos son opcionales y se pueden establecer en NULL. |
| AADSTS90023 | InvalidRequest - la solicitud de servicio de autenticación no es válida. |
| AADSTS9002313 | InvalidRequest: la solicitud tiene un formato incorrecto o no es válida. -El problema se debe a que se ha producido un error con la solicitud a un punto de conexión determinado. Para resolver este problema, se aconseja obtener un seguimiento de Fiddler del error que se está produciendo y buscar si la solicitud tiene un formato correcto o no. |
| AADSTS90024 | RequestBudgetExceededError: se ha producido un error transitorio. Inténtelo de nuevo. |
| AADSTS90033 | MsodsServiceUnavailable: Microsoft Online Directory Service (MSODS) no está disponible. |
| AADSTS90036 | MsodsServiceUnretryableFailure: se ha producido un error inesperado y que no permite el reintento desde el servicio WCF hospedado por MSODS. [Abra un vale de soporte](../fundamentals/active-directory-troubleshooting-support-howto.md) para más información sobre el error. |
| AADSTS90038 | NationalCloudTenantRedirection: el inquilino "Y" especificado pertenece a la nube nacional "X". La instancia de nube actual "Z" no está federada con "X". Se devuelve un error de redireccionamiento de nube. |
| AADSTS90043 | NationalCloudAuthCodeRedirection: la característica está deshabilitada. |
| AADSTS90051 | InvalidNationalCloudId: el identificador de la nube nacional contiene un identificador de la nube no válido. |
| AADSTS90055 | TenantThrottlingError: hay demasiadas solicitudes entrantes. Esta excepción se produce para los inquilinos bloqueados. |
| AADSTS90056 | BadResourceRequest: para canjear el código por un token de acceso, la aplicación debe enviar una solicitud POST al punto de conexión `/token`. Además, antes de esto, debe proporcionar un código de autorización y enviarlo en la solicitud POST al punto de conexión `/token`. Consulte este artículo para más información sobre el flujo de los códigos de autorización de OAuth 2.0: [https://docs.microsoft.com/azure/active-directory/develop/active-directory-protocols-oauth-code](https://docs.microsoft.com/azure/active-directory/develop/active-directory-protocols-oauth-code). Dirija al usuario al punto de conexión `/authorize`, que devolverá un código de autorización. Al publicar una solicitud para el punto de conexión `/token`, el usuario obtiene el token de acceso. Inicie sesión en Azure Portal y compruebe **Registros de aplicaciones > Puntos de conexión** para confirmar que los dos puntos de conexión se configuraron correctamente. |
| AADSTS90072 | PassThroughUserMfaError: la cuenta externa con la que el usuario inicia sesión no existe en el inquilino en el que inició sesión; así pues, el usuario no puede satisfacer los requisitos de MFA para el inquilino. La cuenta debe agregarse primero como un usuario externo en el inquilino. Cierre e inicie sesión con otra cuenta de usuario de Azure AD. |
| AADSTS90081 | OrgIdWsFederationMessageInvalid: se produjo un error cuando el servicio intentó procesar un mensaje de WS-Federation. El mensaje no es válido. |
| AADSTS90082 | OrgIdWsFederationNotSupported: la directiva de autenticación seleccionada para la solicitud no se admite actualmente. |
| AADSTS90084 | OrgIdWsFederationGuestNotAllowed: las cuentas de invitado no están permitidas para este sitio. |
| AADSTS90085 | OrgIdWsFederationSltRedemptionFailed: el servicio no puede emitir un token porque el objeto de la empresa aún no se ha aprovisionado. |
| AADSTS90086 | OrgIdWsTrustDaTokenExpired: el token DA de usuario ha expirado. |
| AADSTS90087 | OrgIdWsFederationMessageCreationFromUriFailed: se ha producido un error al crear el mensaje de WS-Federation desde el URI. |
| AADSTS90090 | GraphRetryableError: el servicio no está disponible temporalmente. |
| AADSTS90091 | GraphServiceUnreachable |
| AADSTS90092 | GraphNonRetryableError |
| AADSTS90093 | GraphUserUnauthorized: Graph devolvió un código de error de prohibido para la solicitud. |
| AADSTS90094 | AdminConsentRequired: es necesario el consentimiento del administrador. |
| AADSTS900382 | El cliente confidencial no se admite en una solicitud entre nubes. |
| AADSTS90099 | No se ha autorizado la aplicación "{appId}" ({appName}) en el inquilino "{tenant}". Las aplicaciones deben estar autorizadas para acceder al inquilino del cliente para que los administradores delegados del asociado puedan usarlas. Proporcione consentimiento previo o ejecute la API del Centro de partners correspondiente para autorizar la aplicación. |
| AADSTS90100 | InvalidRequestParameter: el parámetro está vacío o no es válido. |
| AADSTS901002 | AADSTS901002: No se admite el parámetro de solicitud "resource". |
| AADSTS90101 | InvalidEmailAddress: los datos proporcionados no son una dirección de correo electrónico válida. El formato de la dirección de correo electrónico debe ser `someone@example.com`. |
| AADSTS90102 | InvalidUriParameter: el valor debe ser un URI absoluto válido. |
| AADSTS90107 | InvalidXml: la solicitud no es válida. Asegúrese de que sus datos no tienen caracteres no válidos.|
| AADSTS90114 | InvalidExpiryDate: la marca de tiempo de expiración del token masiva dará lugar a la emisión de un token expirado. |
| AADSTS90117 | InvalidRequestInput |
| AADSTS90119 | InvalidUserCode: el código de usuario es NULL o está vacío.|
| AADSTS90120 | InvalidDeviceFlowRequest: ya se autorizó o rechazó la solicitud. |
| AADSTS90121 | InvalidEmptyRequest: solicitud vacía no válida.|
| AADSTS90123 | IdentityProviderAccessDenied: el token no se puede emitir porque el proveedor de emisión de identidad o notificación rechazó la solicitud. |
| AADSTS90124 | V1ResourceV2GlobalEndpointNotSupported: el recurso no se admite a través de los puntos de conexión `/common` o `/consumers`. Use `/organizations` o el punto de conexión específico del inquilino en su lugar. |
| AADSTS90125 | DebugModeEnrollTenantNotFound: el usuario no está en el sistema. Asegúrese de haber escrito el nombre de usuario correctamente. |
| AADSTS90126 | DebugModeEnrollTenantNotInferred: el tipo de usuario no se admite en este punto de conexión. El sistema no puede inferir el inquilino del usuario desde el nombre de usuario. |
| AADSTS90130 | NonConvergedAppV2GlobalEndpointNotSupported: la aplicación no se admite a través de los puntos de conexión `/common` o `/consumers`. Use `/organizations` o el punto de conexión específico del inquilino en su lugar. |
| AADSTS120000 | PasswordChangeIncorrectCurrentPassword |
| AADSTS120002 | PasswordChangeInvalidNewPasswordWeak |
| AADSTS120003 | PasswordChangeInvalidNewPasswordContainsMemberName |
| AADSTS120004 | PasswordChangeOnPremComplexity |
| AADSTS120005 | PasswordChangeOnPremSuccessCloudFail |
| AADSTS120008 | PasswordChangeAsyncJobStateTerminated: se ha producido un error que no permite el reintento.|
| AADSTS120011 | PasswordChangeAsyncUpnInferenceFailed |
| AADSTS120012 | PasswordChangeNeedsToHappenOnPrem |
| AADSTS120013 | PasswordChangeOnPremisesConnectivityFailure |
| AADSTS120014 | PasswordChangeOnPremUserAccountLockedOutOrDisabled |
| AADSTS120015 | PasswordChangeADAdminActionRequired |
| AADSTS120016 | PasswordChangeUserNotFoundBySspr |
| AADSTS120018 | PasswordChangePasswordDoesnotComplyFuzzyPolicy |
| AADSTS120020 | PasswordChangeFailure |
| AADSTS120021 | PartnerServiceSsprInternalServiceError |
| AADSTS130004 | NgcKeyNotFound: la entidad de seguridad de usuario no tiene configurada la clave de identificador de NGC. |
| AADSTS130005 | NgcInvalidSignature: error de firma de clave de NGC comprobada.|
| AADSTS130006 | NgcTransportKeyNotFound: la clave de transporte de NGC no está configurada en el dispositivo. |
| AADSTS130007 | NgcDeviceIsDisabled: el dispositivo está deshabilitado. |
| AADSTS130008 | NgcDeviceIsNotFound: no se encontró el dispositivo al que la clave de NGC hace referencia. |
| AADSTS135010 | KeyNotFound |
| AADSTS140000 | InvalidRequestNonce: no se proporciona el valor de seguridad de la solicitud. |
| AADSTS140001 | InvalidSessionKey: la clave de sesión no es válida.|
| AADSTS165900 | InvalidApiRequest: solicitud no válida. |
| AADSTS220450 | UnsupportedAndroidWebViewVersion: no se admite la versión de Chrome WebView. |
| AADSTS220501 | InvalidCrlDownload |
| AADSTS221000 | DeviceOnlyTokensNotSupportedByResource: el recurso no está configurado para aceptar tokens solo de dispositivo. |
| AADSTS240001 | BulkAADJTokenUnauthorized: el usuario no tiene autorización para registrar dispositivos en Azure AD. |
| AADSTS240002 | RequiredClaimIsMissing: el token de identificador no se puede usar como concesión `urn:ietf:params:oauth:grant-type:jwt-bearer`.|
| AADSTS530032 | BlockedByConditionalAccessOnSecurityPolicy: el administrador del inquilino tiene configurada una directiva de seguridad que bloquea esta solicitud. Compruebe las directivas de seguridad que están definidas en el nivel de inquilino para determinar si la solicitud cumple con los requisitos de las directivas. |
| AADSTS700016 | UnauthorizedClient_DoesNotMatchRequest: no se encontró la aplicación en el directorio o inquilino. Esto puede pasar si el administrador del inquilino no es el que ha instalado el administrador del inquilino o no ha recibido el consentimiento de ningún usuario del inquilino. Puede que haya configurado de forma incorrecta el valor del identificador de la aplicación o que haya enviado la solicitud de autenticación al inquilino incorrecto. |
| AADSTS700020 | InteractionRequired: la concesión de acceso requiere interacción. |
| AADSTS700022 | InvalidMultipleResourcesScope: el valor proporcionado para el ámbito de parámetro de entrada no es válido porque contiene más de un recurso. |
| AADSTS700023 | InvalidResourcelessScope: el valor proporcionado para el ámbito de parámetro de entrada no es válido al solicitar un token de acceso. |
| AADSTS7000215 | Se ha proporcionado un secreto de cliente no válido. Error del desarrollador: la aplicación está intentando iniciar sesión sin los parámetros de autenticación necesarios o correctos.|
| AADSTS7000222 | InvalidClientSecretExpiredKeysProvided: las claves secretas de cliente que se proporcionaron expiraron. Visite Azure Portal para crear claves para la aplicación o considere usar credenciales de certificado para mayor seguridad: [https://aka.ms/certCreds](https://aka.ms/certCreds) |
| AADSTS700005 | InvalidGrantRedeemAgainstWrongTenant: el código de autorización proporcionado está diseñado para usarlo con otro inquilino, por lo que se rechaza. El código de autorización de OAuth2 se debe canjear en el mismo inquilino para el cual se adquirió (/common o /{tenant-ID} según corresponda). |
| AADSTS1000000 | UserNotBoundError: la API de Bind requiere que el usuario de Azure AD también se autentique con un IDP externo, que aún no se ha producido. |
| AADSTS1000002 | BindCompleteInterruptError: el enlace se completó correctamente, pero debe informarse al usuario. |
| AADSTS7000112 | UnauthorizedClientApplicationDisabled: la aplicación está deshabilitada. |
| AADSTS7500529 | El valor "SAMLId-Guid" no es un identificador de SAML válido. Azure AD usa este atributo para rellenar el atributo InResponseTo de la respuesta devuelta. El id. no debe empezar con un número. La estrategia habitual consiste en anteponer una cadena como "id" en la representación de cadena de un GUID. Por ejemplo, id6c1c178c166d486687be4aaf5e482730 es un identificador válido. |

## <a name="next-steps"></a>Pasos siguientes

* ¿Tiene preguntas y no encuentra lo que busca? Abra una incidencia en GitHub o consulte [Opciones de ayuda y soporte técnico para desarrolladores](active-directory-develop-help-support.md) para ver otras maneras de obtener ayuda y soporte técnico.
