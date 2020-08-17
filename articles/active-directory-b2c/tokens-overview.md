---
title: 'Información general sobre los tokens: Azure Active Directory B2C'
description: Obtenga información sobre los tokens que se utilizan en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 05/21/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 8acdf714f459ae604ccd7788b021aee3ee037935
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87482590"
---
# <a name="overview-of-tokens-in-azure-active-directory-b2c"></a>Configuración de tokens en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) emite varios tipos de tokens de seguridad a medida que procesa cada [flujo de autenticación](application-types.md). Este documento describe el formato, las características de seguridad y el contenido de cada tipo de token.

## <a name="token-types"></a>Tipos de token

Azure AD B2C admite los [protocolos OAuth 2.0 y OpenID Connect](protocols-overview.md), que utilizan tokens para la autenticación y el acceso seguro a los recursos. Todos los tokens usados en Azure AD B2C son [tokens web JSON (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html), que contienen aserciones de información sobre el portador y el asunto del token.

Los tokens siguientes se utilizan en la comunicación con Azure AD B2C:

- *Token de identificador*: JWT que contiene las notificaciones que puede usar para identificar usuarios en la aplicación. Este token se envía de forma segura en las solicitudes HTTP para la comunicación entre dos componentes de la misma aplicación o el mismo servicio. Puede usar las notificaciones en un token de identificador como considere oportuno. Se suelen usar para mostrar información sobre la cuenta o para tomar decisiones sobre el control de acceso en una aplicación. Los tokens de identificador están firmados, pero no cifrados. Cuando una aplicación o API recibe un token de identificador, debe validar la firma para demostrar que es auténtico. La aplicación o la API también deben validar algunas notificaciones del token para demostrar que es válido. En función de los requisitos del escenario, las notificaciones validadas por una aplicación pueden variar, pero la aplicación debe realizar algunas validaciones de notificación comunes en todos los escenarios.
- *Token de acceso*: JWT que contiene notificaciones que puede usar para identificar los permisos concedidos a las API. Los tokens de acceso están firmados, pero no cifrados. Los tokens de acceso se utilizan para proporcionar acceso a las API y los servidores de recursos.  Cuando una API recibe un token de acceso, debe validar la firma para demostrar que es auténtico. La API también debe validar algunas notificaciones del token para demostrar que es válido. En función de los requisitos del escenario, las notificaciones validadas por una aplicación pueden variar, pero la aplicación debe realizar algunas validaciones de notificación comunes en todos los escenarios.
- *Token de actualización*: los tokens de actualización se utilizan para adquirir nuevos tokens de identificador y tokens de acceso en un flujo de OAuth 2.0. Permite a la aplicación obtener acceso a largo plazo a los recursos en nombre de los usuarios sin necesidad de interacción con los usuarios. Los tokens de actualización son totalmente opacos para su aplicación. Los emite Azure AD B2C y solo Azure AD B2C los puede inspeccionar e interpretar. Son de larga duración, pero la aplicación no se debe escribir esperando que un token de actualización dure un período de tiempo especificado. Los tokens de actualización pueden invalidarse en cualquier momento por varios motivos. La única forma de que su aplicación sepa si un token de actualización es válido es intentar canjearlo mediante una solicitud de token a Azure AD B2C. Al canjear un token de actualización por un nuevo token de acceso, recibe un nuevo token de actualización en la respuesta del token. Guarde el nuevo token de actualización. Reemplaza al token de actualización que usó anteriormente en la solicitud. Esta acción ayuda a garantizar que los tokens de actualización sigan siendo válidos mientras sea posible.

## <a name="endpoints"></a>Puntos de conexión

Una [aplicación registrada](tutorial-register-applications.md) recibe los tokens y se comunica con Azure AD B2C mediante el envío de solicitudes a estos puntos de conexión:

- `https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/authorize`
- `https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/token`

Los tokens de seguridad que la aplicación recibe de Azure AD B2C pueden proceder de los puntos de conexión `/authorize` o `/token`. Cuando los tokens de identificador se adquieren en el punto de conexión `/authorize`, el procedimiento se realiza con el [flujo implícito](implicit-flow-single-page-application.md), que se usa a menudo para los usuarios que inician sesión en aplicaciones web basadas en JavaScript. Cuando se adquieren los tokens de identificador en el punto de conexión `/token`, el procedimiento se realiza utilizando el [flujo de código de autorización](openid-connect.md#get-a-token), que mantiene el token oculto al explorador.

## <a name="claims"></a>Notificaciones

Con Azure AD B2C, tendrá un control preciso sobre el contenido de los tokens. Se pueden configurar [flujos de usuario](user-flow-overview.md) y [directivas personalizadas](custom-policy-overview.md) para enviar determinados conjuntos de datos de usuario en las notificaciones que son necesarios para la aplicación. Dichas notificaciones pueden incluir propiedades estándar, tales como **displayName** y **emailAddress**. Las aplicaciones pueden usar estas notificaciones para autenticar usuarios y solicitudes de manera segura.

Las notificaciones de tokens de identificador no se devuelven en ningún orden concreto. Se pueden agregar nuevas notificaciones en tokens de identificador en cualquier momento. No se debe interrumpir la aplicación cuando se agreguen nuevas notificaciones. También puede incluir [atributos de usuario personalizados](user-flow-custom-attributes.md) en sus notificaciones.

En la tabla siguiente se enumeran las notificaciones que puede esperar de los tokens de identificador y los tokens de acceso que emite Azure AD B2C.

| Nombre | Notificación | Valor de ejemplo | Descripción |
| ---- | ----- | ------------- | ----------- |
| Público | `aud` | `90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6` | Identifica al destinatario previsto del token. Para Azure AD B2C, la audiencia es el identificador de la aplicación. La aplicación tiene que validar este valor y rechazar el token si no coincide. Audiencia es sinónimo de recursos. |
| Emisor | `iss` |`https://<tenant-name>.b2clogin.com/775527ff-9a37-4307-8b3d-cc311f58d925/v2.0/` | Identifica el servicio de token de seguridad (STS) que construye y devuelve el token. También identifica el directorio en el que se autenticó el usuario. La aplicación tiene que validar la notificación del emisor para asegurarse de que el token proviene del punto de conexión adecuado. |
| Emitido a las | `iat` | `1438535543` | La hora en que se emitió el token, que se representa en tiempo de época. |
| Fecha de expiración | `exp` | `1438539443` | La hora en que el token deja de ser válido, que se representa en tiempo de época. La aplicación tiene que usar esta notificación para comprobar la validez de la duración del token. |
| No antes de | `nbf` | `1438535543` | Hora a la que el token pasa a ser válido, representada en tiempo de época. Suele ser la misma hora a la que se emitió el token. La aplicación tiene que usar esta notificación para comprobar la validez de la duración del token. |
| Versión | `ver` | `1.0` | Versión del token de identificador, según se ha definido en Azure AD B2C. |
| Código hash | `c_hash` | `SGCPtt01wxwfgnYZy2VJtQ` | El código hash se incluye en los tokens de identificador solo cuando el token se emite junto con un código de autorización de OAuth 2.0. Los códigos hash se pueden usar para validar la autenticidad de un código de autorización. Consulte la [especificación de OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) para obtener más información sobre cómo llevar a cabo esta validación.  |
| Hash de token de acceso | `at_hash` | `SGCPtt01wxwfgnYZy2VJtQ` | El hash de token de acceso se incluye en los tokens de identificador solo cuando el token se emite junto con un token de acceso de OAuth 2.0. El hash de token de acceso se puede usar para validar la autenticidad de un token de acceso. Consulte la [especificación de OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) para obtener más información sobre cómo llevar a cabo esta validación.  |
| Valor de seguridad | `nonce` | `12345` | El valor de seguridad es una estrategia que se usa para mitigar los ataques de reproducción de tokens. La aplicación puede especificar un valor de seguridad en una solicitud de autorización mediante el parámetro de consulta `nonce`. El valor que se proporcione en la solicitud solo se emitirá sin modificar en la notificación `nonce` de un token de identificador. Esta notificación permite que la aplicación compruebe el valor con respecto al valor especificado en la solicitud. La aplicación tiene que realizar esta validación durante el proceso de validación del token de identificador. |
| Asunto | `sub` | `884408e1-2918-4cz0-b12d-3aa027d7563b` | Entidad de seguridad sobre la que el token declara información como, por ejemplo, el usuario de una aplicación. Este valor es inmutable y no se puede reasignar ni volver a usar. Se puede usar para realizar comprobaciones de autorización de forma segura, por ejemplo, cuando el token se usa para acceder a un recurso. De manera predeterminada, la notificación del asunto se rellena con el identificador de objeto del usuario del directorio. |
| Referencia de clase de contexto de autenticación | `acr` | No aplicable | Solo se utiliza con las directivas antiguas. |
| Directiva de marco de confianza | `tfp` | `b2c_1_signupsignin1` | Nombre de la directiva que se usó para adquirir el token de identificador. |
| Hora de autenticación | `auth_time` | `1438535543` | Hora a la que un usuario especificó sus credenciales por última vez, representada en hora de época. No existe distinción entre una autenticación que sea un inicio de sesión nuevo, una sesión de inicio de sesión único (SSO) u otro tipo de inicio de sesión. `auth_time` muestra la última vez que la aplicación (o el usuario) inició un intento de autenticación en Azure AD B2C. No se indica el método usado para la autenticación. |
| Ámbito | `scp` | `Read`| Permisos concedidos al recurso para un token de acceso. Si se trata de varios permisos concedidos, se separan con un espacio. |
| Entidad autorizada | `azp` | `975251ed-e4f5-4efd-abcb-5f1a8f566ab7` | El **id. de la aplicación** cliente que inició la solicitud. |

## <a name="configuration"></a>Configuración

Las siguientes propiedades se utilizan para [administrar la vigencia de los tokens de seguridad](configure-tokens.md) emitidos por Azure AD B2C:

- **Vigencia (en minutos) de token de acceso y de identificador**: la vigencia del token de portador de OAuth 2.0 que se utiliza para obtener acceso a un recurso protegido. El valor predeterminado es 60 minutos. El mínimo (incluido) es de 5 minutos. El máximo (incluido) es de 1440 minutos.

- **Duración del token de actualización (días)** : período de tiempo máximo antes del que puede usarse un token de actualización para adquirir un nuevo token de identificador o de acceso. El período de tiempo también cubre la adquisición de un nuevo token de actualización si se ha concedido el ámbito `offline_access` a la aplicación. El valor predeterminado es 14 días. El mínimo (incluido) es de 1 día. El máximo (incluido) es de 90 días.

- **Duración de la ventana deslizante del token de actualización (días)** : después de transcurrido este período de tiempo, el usuario está obligado a volver a autenticarse, independientemente de cuál sea el período de validez del último token de actualización obtenido por la aplicación. Solo se pueden proporcionar si el conmutador se establece en **Bounded**(Enlazado). Es preciso que sea mayor o igual que el valor de **Vigencia del token de actualización (en días)** . Si el conmutador se establece en **Unbounded**(Sin enlazar), no se puede proporcionar un valor específico. El valor predeterminado es de 90 días. El mínimo (incluido) es de 1 día. El máximo (incluido) es de 365 días.

Los siguientes casos de uso se habilitan mediante estas propiedades:

- Permiso para que un usuario pueda permanecer conectado en una aplicación móvil indefinidamente, siempre que esté continuamente activo en la misma. Establezca **Duración de la ventana deslizante del token de actualización (días)** en **Unbounded** (Sin enlazar) en el flujo de usuario de inicio de sesión.
- Cumpla los requisitos de cumplimiento normativo y seguridad de la industria mediante el establecimiento de la vigencia adecuada del token de acceso.

Estas opciones no están disponibles para flujos de usuario de restablecimiento de contraseña.

## <a name="compatibility"></a>Compatibilidad

Las siguientes propiedades se utilizan para [administrar la compatibilidad de los tokens](configure-tokens.md):

- **Notificación de emisor (iss)** : esta propiedad identifica el inquilino de Azure AD B2C que emitió el token. El valor predeterminado es `https://<domain>/{B2C tenant GUID}/v2.0/`. El valor de `https://<domain>/tfp/{B2C tenant GUID}/{Policy ID}/v2.0/` incluye los identificadores tanto del inquilino de Azure AD B2C como del flujo de usuario usado en la solicitud de token. Si una aplicación o biblioteca necesita que Azure AD B2C sea compatible con la [especificación OpenID Connect Discovery 1.0](https://openid.net/specs/openid-connect-discovery-1_0.html), use este valor.

- **Notificación de asunto (sub)** : esta propiedad identifica la entidad para la que el token valida la información. El valor predeterminado es **ObjectID**, que rellena la notificación `sub` en el token con el identificador de objeto del usuario. El valor de **No compatible** solo se proporciona por motivos de compatibilidad con versiones anteriores. Se recomienda cambiar a **ObjectID** lo antes posible.

- **Notificación que representa el identificador de la directiva**: esta propiedad identifica el tipo de notificación en el que se rellena el nombre de directiva utilizado en la solicitud de token. El valor predeterminado es `tfp`. El valor de `acr` solo se proporciona por motivos de compatibilidad con versiones anteriores.

## <a name="pass-through"></a>Paso a través

Cuando se inicia un recorrido del usuario, Azure AD B2C recibe un token de acceso de un proveedor de identidades. Azure AD B2C usa ese token para recuperar información sobre el usuario. [Habilite una notificación en el flujo de usuario](idp-pass-through-user-flow.md) o [defina una notificación en la directiva personalizada](idp-pass-through-custom.md) para pasar el token a través de las aplicaciones que registre en Azure AD B2C. La aplicación debe usar un [flujo de usuario recomendado](user-flow-versions.md) para aprovechar las ventajas de pasar el token como una notificación.

Azure AD B2C actualmente solo admite pasar el token de acceso de proveedores de identidades de OAuth 2.0, incluidos Facebook y Google. Para todos los demás proveedores de identidades, la notificación se devuelve en blanco.

## <a name="validation"></a>Validación

Para validar un token, la aplicación debe comprobar tanto la firma como las notificaciones del token. Hay muchas bibliotecas de código abierto disponibles para validar los JWT según su lenguaje preferido. Se recomienda explorar esas opciones en lugar de implementar su propia lógica de validación.

### <a name="validate-signature"></a>Validar la firma

Un JWT contiene tres segmentos, un *encabezado*, un *cuerpo* y un *firma*. El segmento de firma se puede utilizar para validar la autenticidad del token con el fin de que la aplicación pueda confiar en él. Los tokens de Azure AD B2C se firman con algoritmos de cifrado asimétrico estándar del sector, como RSA 256.

El encabezado del token contiene información acerca del método de cifrado y la clave utilizados para firmar el token:

```
{
        "typ": "JWT",
        "alg": "RS256",
        "kid": "GvnPApfWMdLRi8PDmisFn7bprKg"
}
```

El valor de la notificación **alg** es el algoritmo que se usó para firmar el token. El valor de la notificación **kid** es la clave pública que se usó para firmar el token. En cualquier momento, Azure AD B2C puede firmar un token mediante cualquier opción de un conjunto de pares de claves pública y privada. Azure AD B2C gira los posibles conjuntos de claves periódicamente. La aplicación debe escribirse para controlar automáticamente esos cambios de clave. Una frecuencia razonable para buscar actualizaciones para las claves públicas que usa Azure AD B2C es cada 24 horas.

Azure AD B2C tiene un punto de conexión de metadatos OpenID Connect. Con este punto de conexión, las aplicaciones pueden solicitar información acerca de Azure AD B2C en tiempo de ejecución. En esta información se incluyen los extremos, los contenidos del token y las claves de firma de los token. Hay un documento de metadatos JSON para cada directiva en su inquilino de Azure AD B2C. El documento de metadatos es un objeto JSON que contiene varias piezas de información útiles. Los metadatos contienen **jwks_uri**, que ofrece la ubicación del conjunto de claves públicas que se usan para firmar los tokens. La ubicación se proporciona aquí, pero es mejor capturarla dinámicamente mediante el documento de metadatos y el análisis de **jwks_uri**:

```
https://contoso.b2clogin.com/contoso.onmicrosoft.com/b2c_1_signupsignin1/discovery/v2.0/keys
```
El documento JSON que se encuentra en esta dirección URL contiene toda la información de clave pública en uso en ese momento determinado. La aplicación puede usar la notificación `kid` en el encabezado de JWT para seleccionar en el documento JSON la clave pública que se ha usado en este documento para firmar un token concreto. Después, puede realizar la validación de la firma mediante la clave pública correcta y el algoritmo indicado.

El documento de metadatos de la directiva `B2C_1_signupsignin1` en el inquilino `contoso.onmicrosoft.com` se encuentra en:

```
https://contoso.b2clogin.com/contoso.onmicrosoft.com/b2c_1_signupsignin1/v2.0/.well-known/openid-configuration
```

Para determinar qué directiva se usó para firmar un token (y dónde solicitar los metadatos), tiene dos opciones. En primer lugar, el nombre de la directiva se incluye en la notificación `acr` del token. Las notificaciones se pueden analizar fuera del cuerpo del JWT; para ello, descodifique la descodificación en base 64 del cuerpo y deserialice la cadena JSON resultante. La notificación `acr` es el nombre de la directiva que se usó para emitir el token. La otra opción consiste en codificar la directiva en el valor del parámetro `state` al emitir la solicitud y, a continuación, descodificarla para determinar qué directiva se ha usado. Cualquiera de estos métodos es válido.

La descripción de cómo realizar la validación de la firma queda fuera del ámbito de este documento. Hay muchas bibliotecas de código abierto disponibles para ayudarle a validar un token.

### <a name="validate-claims"></a>Validar notificaciones

Cuando una aplicación o una API reciben un token de identificador, también es preciso realizar varias comprobaciones de las notificaciones en el token de identificador. Se deben comprobar las notificaciones siguientes:

- **audience**: comprueba que está previsto que el token de identificador se proporcione a su aplicación.
- **not before** y **expiration time**: comprueban que el token de identificador no ha expirado.
- **issuer**: comprueba que Azure AD B2C emitió el token para la aplicación.
- **nonce**: una estrategia para mitigar los ataques de reproducción de tokens.

Para ver una lista completa de las validaciones que la aplicación debe llevar a cabo, consulte la [especificación de OpenID Connect](https://openid.net).

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo [usar los tokens de acceso](access-tokens.md).

