---
title: Autenticación de Azure Active Directory y códigos de error de autorización | Microsoft Docs
description: Obtenga información sobre los códigos de error AADSTS que devuelve el servicio de token de seguridad (STS) de Azure AD.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 02/13/2019
ms.author: ryanwi
ms.reviewer: hirsin, justhu
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 287e4ee53a108d1e2b83d4a8b11a98a2c7727721
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545586"
---
# <a name="authentication-and-authorization-error-codes"></a>Códigos de error de autenticación y autorización

¿Busca información sobre los códigos de error AADSTS que devuelve el servicio de token de seguridad (STS) de Azure Active Directory? Lea este documento para ver las descripciones de los errores AADSTS, sus correcciones y algunas soluciones alternativas.

> [!NOTE]
> Esta información es preliminar y está sujeta a cambios. ¿Tiene preguntas y no encuentra lo que busca? Abra una incidencia en GitHub o consulte [Opciones de ayuda y soporte técnico para desarrolladores](active-directory-develop-help-support.md) para ver otras maneras de obtener ayuda y soporte técnico.
>
> Esta documentación se proporciona para desarrolladores e instrucciones de administración, pero nunca debe utilizarse con el propio cliente. Códigos de error están sujetos a cambios en cualquier momento con el fin de proporcionar mensajes de error más específicos que están diseñados para ayudar al programador durante la compilación de su aplicación. Las aplicaciones que toman una dependencia en los números de código de error o de texto, se interrumpirá en el tiempo.  

## <a name="aadsts-error-codes"></a>Códigos de error AADSTS

| Error | DESCRIPCIÓN |
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
| AADSTS50011 | InvalidReplyTo: la dirección de respuesta falta, está mal configurada o no coincide con las direcciones de respuesta configuradas para la aplicación.  Asegúrese de agregar esta dirección de respuesta que faltan a la aplicación de Azure Active Directory o pedir a alguien con los permisos para administrar la aplicación en Active Directory hace esto como una resolución.|
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
| AADSTS50042 | UnableToGeneratePairwiseIdentifierWithMissingSalt - la sal necesaria para generar un identificador en pares falta en principio. Póngase en contacto con el administrador del inquilino. |
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
| AADSTS50087 | SasRetryableError: el servicio no está disponible temporalmente. Vuelva a intentarlo. |
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
| AADSTS67003 | ActorNotValidServiceIdentity |
| AADSTS70000 | InvalidGrant: se ha producido un error de autenticación de cliente. El token de actualización no es válido. El error puede deberse a los siguientes motivos:<ul><li>El encabezado del enlace de token está vacío.</li><li>El hash del enlace de token no coincide.</li></ul> |
| AADSTS70001 | UnauthorizedClient: la aplicación está deshabilitada. |
| AADSTS70002 | InvalidClient: error al validar las credenciales. El valor de client_secret especificado no coincide con el esperado para este cliente. Corrija el valor de client_secret e inténtelo de nuevo. Para más información, consulte [Uso del código de autorización para solicitar un token de acceso](v1-protocols-oauth-code.md#use-the-authorization-code-to-request-an-access-token). |
| AADSTS70003 | UnsupportedGrantType: la aplicación ha devuelto un tipo de concesión no admitido. |
| AADSTS70004 | InvalidRedirectUri: la aplicación ha devuelto un URI de redireccionamiento no válido. La dirección de redireccionamiento que ha especificado el cliente no coincide con ninguna de las direcciones configuradas ni con ninguna de las de la lista de direcciones aprobadas de OIDC. |
| AADSTS70005 | UnsupportedResponseType: la aplicación ha devuelto un tipo de respuesta no admitido por alguno de los siguientes motivos:<ul><li>el tipo de respuesta "token" no está habilitado para la aplicación</li><li>el tipo de respuesta "id_token" requiere el ámbito "OpenID", o contiene un valor de parámetro de OAuth no admitido en el wctx codificado</li></ul> |
| AADSTS70007 | UnsupportedResponseMode: la aplicación ha devuelto un valor no admitido de `response_mode` al solicitar un token.  |
| AADSTS70008 | ExpiredOrRevokedGrant: el token de actualización ha expirado debido por inactividad. El token se emitió en XXX y estuvo inactivo durante un período de tiempo. |
| AADSTS70011 | InvalidScope: el ámbito solicitado por la aplicación no es válido. |
| AADSTS70012 | MsaServerError: se ha producido un error de servidor al autenticar un usuario (consumidor) de MSA. Vuelva a intentarlo. Si el error persiste, [abra una incidencia de soporte técnico](../fundamentals/active-directory-troubleshooting-support-howto.md) |
| AADSTS70016 | AuthorizationPending: error de flujo de dispositivo de OAuth 2.0. La autorización está pendiente. El dispositivo volverá a intentar sondear la solicitud. |
| AADSTS70018 | BadVerificationCode: el código de verificación no es válido porque el usuario ha escrito un código de usuario incorrecto en el flujo de códigos del dispositivo. No se ha aprobado la autorización. |
| AADSTS70019 | CodeExpired: el código de verificación ha expirado. Pida al usuario que vuelva a iniciar sesión. |
| AADSTS75001 | BindingSerializationError: se ha producido un error durante el enlace de mensajes de SAML. |
| AADSTS75003 | UnsupportedBindingError: la aplicación ha devuelto un error relacionado con un enlace no admitido (la respuesta del protocolo SAML no se puede enviar a través de enlaces que no sean HTTP POST). |
| AADSTS75005 | Saml2MessageInvalid: Azure AD no admite la solicitud SAML enviada por la aplicación para el inicio de sesión único. |
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
| AADSTS90024 | RequestBudgetExceededError: se ha producido un error transitorio. Vuelva a intentarlo. |
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
| AADSTS90100 | InvalidRequestParameter: el parámetro está vacío o no es válido. |
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
| AADSTS530032 | BlockedByConditionalAccessOnSecurityPolicy: el Administrador de inquilinos ha configurado una directiva de seguridad que impide esta solicitud. Compruebe las directivas de seguridad que se definen en el nivel del inquilino para determinar si la solicitud cumple los requisitos de directiva. |
| AADSTS700016 | UnauthorizedClient_DoesNotMatchRequest: no se encontró la aplicación en el directorio o inquilino. Esto puede pasar si el administrador del inquilino no es el que ha instalado el administrador del inquilino o no ha recibido el consentimiento de ningún usuario del inquilino. Puede que haya configurado de forma incorrecta el valor del identificador de la aplicación o que haya enviado la solicitud de autenticación al inquilino incorrecto. |
| AADSTS700020 | InteractionRequired: la concesión de acceso requiere interacción. |
| AADSTS700022 | InvalidMultipleResourcesScope: el valor proporcionado para el ámbito de parámetro de entrada no es válido porque contiene más de un recurso. |
| AADSTS700023 | InvalidResourcelessScope: el valor proporcionado para el ámbito de parámetro de entrada no es válido al solicitar un token de acceso. |
| AADSTS1000000 | UserNotBoundError: la API de Bind requiere que el usuario de Azure AD también se autentique con un IDP externo, que aún no se ha producido. |
| AADSTS1000002 | BindCompleteInterruptError: el enlace se completó correctamente, pero debe informarse al usuario. |

## <a name="next-steps"></a>Pasos siguientes

* ¿Tiene preguntas y no encuentra lo que busca? Abra una incidencia en GitHub o consulte [Opciones de ayuda y soporte técnico para desarrolladores](active-directory-develop-help-support.md) para ver otras maneras de obtener ayuda y soporte técnico.
