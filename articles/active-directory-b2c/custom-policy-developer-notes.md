---
title: Notas del desarrollador para las directivas personalizadas
titleSuffix: Azure AD B2C
description: Notas para desarrolladores sobre la configuración y el mantenimiento de Azure AD B2C con directivas personalizadas.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 05/19/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 23e443a542ed22906abb237e402834b707e623a9
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88852032"
---
# <a name="developer-notes-for-custom-policies-in-azure-active-directory-b2c"></a>Notas para desarrolladores sobre directivas personalizadas en Azure Active Directory B2C

La configuración de directivas personalizadas en Azure Active Directory B2C ya está disponible con carácter general. Este método de configuración está destinado a desarrolladores de identidades avanzados que crean soluciones de identidad complejas. Las directivas personalizadas permiten aprovechar la capacidad de Identity Experience Framework en los inquilinos de Azure AD B2C.
Los desarrolladores de identidades avanzados que usan directivas personalizadas deben planificar un tiempo para realizar tutoriales y leer documentos de referencia.

Aunque la mayoría de las opciones de directivas personalizadas ya se encuentran disponibles con carácter general, existen funcionalidades subyacentes, como tipos de perfiles técnicos y API de definición de contenido, que se encuentran en fases distintas del ciclo de vida del software. Próximamente se ofrecerán muchas más. En la tabla siguiente se especifica el nivel de disponibilidad a un nivel más pormenorizado.

## <a name="features-that-are-generally-available"></a>Características disponibles con carácter general

- Crear y cargar recorridos del usuario de autenticación personalizada mediante directivas personalizadas.
    - Describir los recorridos del usuario paso a paso como intercambios entre proveedores de notificaciones.
    - Definir la creación de ramas condicional en recorridos del usuario.
- Interoperar con servicios con la API REST habilitada en los recorridos del usuario de autenticación personalizada.
- Federarse con proveedores de identidades que cumplen con el protocolo OpenIDConnect.
- Federarse con proveedores de identidades que cumplen el protocolo SAML 2.0.

## <a name="responsibilities-of-custom-policy-feature-set-developers"></a>Responsabilidades de los desarrolladores de conjunto de características de directivas personalizadas

La configuración manual de directivas concede un acceso de nivel inferior a la plataforma subyacente de Azure AD B2C y da como resultado la creación de un marco de confianza único. Las múltiples permutaciones de proveedores de identidades personalizados, las relaciones de confianza, las integraciones con servicios externos y los flujos de trabajo paso a paso exigen un enfoque de diseño y configuración más metódico.

Los desarrolladores que utilizan el conjunto de características de directivas personalizadas deben cumplir las siguientes directrices:

- Conocer el lenguaje de configuración de las directivas personalizadas y la administración de claves/secretos. Para más información, consulte [TrustFrameworkPolicy](trustframeworkpolicy.md).
- Tomar posesión de escenarios e integraciones personalizadas. Documentar su trabajo e informar a la organización del sitio activo.
- Realizar pruebas metódicas de los escenarios.
- Seguir los procedimientos recomendados de desarrollo de software y almacenamiento provisional con un mínimo de un entorno de desarrollo y prueba, y un entorno de producción.
- Mantenerse informado sobre los nuevos desarrollos de los proveedores de identidad y los servicios con los que está integrado. Por ejemplo, realizar un seguimiento de cambios en los secretos y los cambios programados y no programados en el servicio.
- Configurar la supervisión activa y supervisar la capacidad de respuesta de los entornos de producción. Para más información sobre la integración con Application Insights, vea [Azure Active Directory B2C: Recopilación de registros](analytics-with-application-insights.md).
- Mantenga actualizadas las direcciones de correo electrónico de contacto y responda a los correos mensajes de correo electrónico del equipo de Microsoft.
- Realizar las acciones pertinentes cuando se lo indique el equipo del sitio activo de Microsoft.

## <a name="terms-for-features-in-public-preview"></a>Términos de características en versión preliminar pública

- Se aconseja usar las características en versión preliminar pública solo con fines de evaluación.
- Los Acuerdos de Nivel de Servicio (SLA) no se aplican a las características en versión preliminar pública.
- Las solicitudes de soporte técnico para características en versión preliminar pública pueden enviarse a través de los canales de soporte técnico habituales.

## <a name="features-by-stage-and-known-issues"></a>Características por fase y problemas conocidos

Las directivas personalizadas y las funcionalidades de Identity Experience Framework están en desarrollo constante y rápido. La siguiente tabla es un índice de disponibilidad de características y componentes.


### <a name="protocols-and-authorization-flows"></a>Protocolos y flujos de autorización

| Característica | Desarrollo | Vista previa | GA | Notas |
|-------- | :-----------: | :-------: | :--: | ----- |
| [Código de autorización OAuth2](authorization-code-flow.md) |  |  | X |  |
| Código de autorización OAuth2 con PKCE |  |  | X | Solo aplicaciones móviles  |
| [Flujo implícito de OAuth2](implicit-flow-single-page-application.md) |  |  | X |  |
| [Credenciales de contraseña del propietario del recurso OAuth2](ropc-custom.md) |  | X |  |  |
| [Conexión OIDC](openid-connect.md) |  |  | X |  |
| [SAML2](connect-with-saml-service-providers.md)  |  |  |X  | Enlaces POST y de redirección. |
| OAuth1 |  |  |  | No compatible. |
| WSFED | X |  |  |  |

### <a name="identify-providers-federation"></a>Identificación de la federación de proveedores 

| Característica | Desarrollo | Vista previa | GA | Notas |
|-------- | :-----------: | :-------: | :--: | ----- |
| [OpenID Connect](openid-connect-technical-profile.md) |  |  | X | Por ejemplo, Google+.  |
| [OAuth2](oauth2-technical-profile.md) |  |  | X | Por ejemplo, Facebook.  |
| [OAuth1](oauth1-technical-profile.md) |  | X |  | Por ejemplo, Twitter. |
| [SAML2](saml-identity-provider-technical-profile.md) |  |   | X | Por ejemplo, Salesforce, ADFS. |
| WSFED| X |  |  |  |


### <a name="rest-api-integration"></a>Integración de API REST

| Característica | Desarrollo | Vista previa | GA | Notas |
|-------- | :-----------: | :-------: | :--: | ----- |
| [API REST con autenticación básica](secure-rest-api.md#http-basic-authentication) |  |  | X |  |
| [API REST con autenticación de certificado de cliente](secure-rest-api.md#https-client-certificate-authentication) |  |  | X |  |
| [API REST con autenticación de portador de OAuth2](secure-rest-api.md#oauth2-bearer-authentication) |  | X |  |  |

### <a name="component-support"></a>Compatibilidad de componentes

| Característica | Desarrollo | Vista previa | GA | Notas |
| ------- | :-----------: | :-------: | :--: | ----- |
| [Autenticación de factor de teléfono](phone-factor-technical-profile.md) |  |  | X |  |
| [Autenticación de Azure MFA](multi-factor-auth-technical-profile.md) |  | X |  |  |
| [Contraseña de un solo uso](one-time-password-technical-profile.md) |  | X |  |  |
| [Azure Active Directory](active-directory-technical-profile.md) como directorio local |  |  | X |  |
| Subsistema de correo electrónico de Azure para la comprobación de correo electrónico |  |  | X |  |
| [Proveedores de servicios de correo electrónico de terceros](custom-email-mailjet.md) |  |X  |  |  |
| [Compatibilidad con varios lenguajes](localization.md)|  |  | X |  |
| [Validaciones de predicado](predicates.md) |  |  | X | Por ejemplo, la complejidad de la contraseña. |
| [Controles de presentación](display-controls.md) |  |X  |  |  |


### <a name="page-layout-versions"></a>Versiones de diseño de página

| Característica | Desarrollo | Vista previa | GA | Notas |
| ------- | :-----------: | :-------: | :--: | ----- |
| [2.0.0](page-layout.md) |  | X |  |  |
| [1.2.0](page-layout.md) |  | X |  |  |
| [1.1.0](page-layout.md) |  |  | X |  |
| [1.0.0](page-layout.md) |  |  | X |  |
| [Compatibilidad con JavaScript](javascript-samples.md) |  | X |  |  |

### <a name="app-ief-integration"></a>Integración de aplicaciones IEF

| Característica | Desarrollo | Vista previa | GA | Notas |
| ------- | :-----------: | :-------: | :--: | ----- |
| Parámetro `domain_hint` de cadena de consulta |  |  | X | Disponible como notificación, puede pasarse al IDP. |
| Parámetro `login_hint` de cadena de consulta |  |  | X | Disponible como notificación, puede pasarse al IDP. |
| Inserción de JSON en el recorrido del usuario a través de `client_assertion` | X |  |  | En desuso. |
| Inserción de JSON en el recorrido del usuario como `id_token_hint` |  | X |  | Enfoque de avance para pasar a JSON. |
| [Paso del token del proveedor de identidades a la aplicación](idp-pass-through-custom.md) |  | X |  | Por ejemplo, de Facebook a la aplicación. |

### <a name="session-management"></a>Administración de sesiones

| Característica | Desarrollo | Vista previa | GA | Notas |
| ------- | :-----------: | :-------: | :--: | ----- |
| [Proveedor predeterminado de sesión de SSO](custom-policy-reference-sso.md#defaultssosessionprovider) |  |  | X |  |
| [Proveedor externo de sesión de inicio de sesión](custom-policy-reference-sso.md#externalloginssosessionprovider) |  |  | X |  |
| [Proveedor de sesión de SSO de SAML](custom-policy-reference-sso.md#samlssosessionprovider) |  |  | X |  |
| [OAuthSSOSessionProvider](custom-policy-reference-sso.md#oauthssosessionprovider)  |  | X |  |  |
| [Cierre de sesión único](session-overview.md#sign-out)  |  | X |  |  |

### <a name="security"></a>Seguridad

| Característica | Desarrollo | Vista previa | GA | Notas |
|-------- | :-----------: | :-------: | :--: | ----- |
| Claves de directivas: generar, manual, carga |  |  | X |  |
| Claves de directivas: RSA/Certificado, secretos |  |  | X |  |


### <a name="developer-interface"></a>Interfaz del desarrollador

| Característica | Desarrollo | Vista previa | GA | Notas |
| ------- | :-----------: | :-------: | :--: | ----- |
| Azure Portal-IEF UX |  |  | X |  |
| Carga de directiva |  |  | X |  |
| [Registros de recorridos del usuario de Application Insights](troubleshoot-with-application-insights.md) |  | X |  | Se usa para solucionar problemas durante el desarrollo.  |
| [Registros de eventos de Application Insights](application-insights-technical-profile.md) |  | X |  | Se usa para supervisar flujos de usuario en producción. |


## <a name="next-steps"></a>Pasos siguientes

- Consulte las [Operaciones de Microsoft Graph disponibles para Azure AD B2C](microsoft-graph-operations.md).
- Obtenga más información sobre las [directivas personalizadas y las diferencias con los flujos de usuario](custom-policy-overview.md).
