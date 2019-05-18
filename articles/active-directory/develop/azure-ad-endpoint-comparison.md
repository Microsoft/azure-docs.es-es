---
title: ¿Por qué actualizar a la plataforma de identidad de Microsoft (v2.0) | Azure
description: Conocer las diferencias entre el punto de conexión de Microsoft identity platform (v2.0) y el punto de conexión de Azure Active Directory (Azure AD) v1.0 y descubra las ventajas de actualizar a la versión 2.0.
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
ms.topic: conceptual
ms.date: 05/07/2019
ms.author: ryanwi
ms.reviewer: saeeda, hirsin, jmprieur, sureshja, jesakowi, lenalepa, kkrishna, dadobali, negoe
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 7389431212fd34960279aad5321ef2eeb70b7790
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540437"
---
# <a name="why-update-to-microsoft-identity-platform-v20"></a>¿Por qué actualizar a la plataforma de identidad de Microsoft (v2.0)?

Al desarrollar una nueva aplicación, es importante conocer las diferencias entre la plataforma de identidad de Microsoft (v2.0) y los puntos de conexión de Azure Active Directory (v1.0). En este artículo se trata las principales diferencias entre los puntos de conexión y algunas limitaciones para la plataforma Microsoft identity existentes.

> [!NOTE]
> El punto de conexión de plataforma de identidad de Microsoft no es compatible con todas las características y escenarios de Azure AD. Para determinar si debe usar el punto de conexión de plataforma de identidad de Microsoft, obtenga información sobre [limitaciones de la plataforma de identidad de Microsoft](#limitations).

## <a name="who-can-sign-in"></a>Quién puede iniciar sesión

![Quién puede iniciar sesión con los puntos de conexión v1.0 y v2.0](media/azure-ad-endpoint-comparison/who-can-sign-in.svg)

* El punto de conexión v1.0 solo permite iniciar sesión en la aplicación con cuentas profesionales y educativas (Azure AD).
* El punto de conexión de plataforma de identidad de Microsoft permite trabajo y cuentas educativas de Azure AD y cuentas personales de Microsoft (MSA), como hotmail.com, outlook.com y msn.com, para iniciar sesión.
* Ambos puntos de conexión también aceptan inicios de sesión de *[usuarios invitados](https://docs.microsoft.com/azure/active-directory/b2b/what-is-b2b)* de un directorio de Azure AD para las aplicaciones configuradas como *[inquilino único](single-and-multi-tenant-apps.md)* o para *multiinquilino* aplicaciones configuradas para el punto de conexión específico del inquilino (`https://login.microsoftonline.com/{TenantId_or_Name}`).

El punto de conexión de plataforma de identidad de Microsoft le permite crear aplicaciones que aceptan inicios de sesión desde cuentas personales de Microsoft y cuentas profesionales y educativas. Esto le permite escribir la aplicación con independencia absoluta de la cuenta. Por ejemplo, si su aplicación llama a [Microsoft Graph](https://graph.microsoft.io), las cuentas profesionales tendrán a su disposición algunos datos y funcionalidades adicionales, como los sitios de SharePoint o datos de directorio. Pero, para muchas acciones, como [leer correo de un usuario](https://developer.microsoft.com/graph/docs/api-reference/v1.0/api/user_list_messages), el mismo código puede acceder al correo electrónico tanto para cuentas personales como profesionales y educativas.

Punto de conexión de plataforma de identidad de Microsoft, puede usar la biblioteca de autenticación de Microsoft (MSAL) para obtener acceso al consumidor, educativo y los mundos de la empresa. El punto de conexión v1.0 de Azure AD acepta inicios de sesión solo desde cuentas profesionales y educativas.

## <a name="incremental-and-dynamic-consent"></a>Consentimiento incremental y dinámico

Las aplicaciones que usan el punto de conexión v1.0 de Azure AD tienen que especificar los permisos de OAuth 2.0 necesarios de antemano, por ejemplo:

![Interfaz de usuario de registro de permisos](./media/azure-ad-endpoint-comparison/app_reg_permissions.png)

Los permisos establecidos directamente en el registro de la aplicación son **estáticos**. Aunque los permisos estáticos de la aplicación definidos en Azure Portal mantenían un código ordenado y sencillo, presenta algunos problemas para los desarrolladores:

* Una aplicación tiene que solicitar todos los permisos que podría necesitar alguna vez tras el primer inicio de sesión del usuario. Esto puede originar una lista larga de permisos, lo que desanimaría a los usuarios finales de aprobar el acceso de la aplicación en el inicio de sesión inicial.

* La aplicación tiene que conocer por adelantado todos los recursos a los que va a tener acceso alguna vez. Era difícil crear aplicaciones que podrían tener acceso a un número arbitrario de recursos.

Con el punto de conexión de plataforma de identidad de Microsoft, puede omitir los permisos estáticos definidos en la información de registro de aplicación en el Azure portal y solicitar permisos incrementalmente en su lugar, lo que significa que solicita un conjunto de permisos por adelantado mínimo y crezca más con el tiempo que el cliente usa las características de aplicaciones adicionales. Para ello, puede especificar los ámbitos que la aplicación necesita en cualquier momento incluyendo los nuevos ámbitos en el parámetro `scope` cuando solicite un token de acceso. No es necesario predefinirlos en la información de registro de la aplicación. Si el usuario aún no ha dado su consentimiento a los ámbitos nuevos que se agregan a la solicitud, se le pedirá que dé su consentimiento solo a los nuevos permisos. Para obtener más información, consulte sobre [permisos, consentimiento y ámbitos](v2-permissions-and-consent.md).

Permitir que una aplicación solicite permisos dinámicamente mediante el parámetro `scope` da a los desarrolladores un control total sobre la experiencia del usuario. También puede adelantar la experiencia de consentimiento y pedir todos los permisos en una solicitud de autorización inicial. Si su aplicación requiere un gran número de permisos, puede elegir recopilarlos del usuario de forma incremental, a medida que intentan usar determinadas características de la aplicación con el tiempo.

El consentimiento del administrador que se realiza en nombre de una organización sigue requriendo los permisos estáticos registrados para la aplicación, por lo que debería establecer esos permisos para las aplicaciones en el portal de registro de aplicaciones si necesita que un administrador dé su consentimiento en nombre de toda la organización. Esto reduce los ciclos que el administrador de la organización necesita para instalar la aplicación.

## <a name="scopes-not-resources"></a>Ámbitos, no recursos

Las aplicaciones que usan el punto de conexión v1.0 pueden comportarse como un **recurso** o como un destinatario de tokens. Un recurso puede definir varios **ámbitos** o **oAuth2Permissions** que comprende, lo que permite a las aplicaciones cliente solicitar tokens para ese recurso para un conjunto determinado de ámbitos. Piense en Graph API de Azure AD como ejemplo de un recurso:

* Identificador de recursos o `AppID URI`: `https://graph.windows.net/`
* Los ámbitos, o `oAuth2Permissions`: `Directory.Read`, `Directory.Write`, etc.

Esto es válido para el punto de conexión de plataforma de identidad de Microsoft. Una aplicación todavía se puede comportar como recurso, definir ámbitos y ser identificada por un URI. Las aplicaciones cliente todavía pueden solicitar acceso a esos ámbitos. Sin embargo, la forma en que un cliente solicita esos permisos han cambiado.

Para el punto de conexión v1.0, una solicitud de autorización de OAuth 2.0 para Azure AD podría haber tenido un aspecto similar al siguiente:

```text
GET https://login.microsoftonline.com/common/oauth2/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&resource=https://graph.windows.net/
...
```

Aquí, el parámetro **resource** indicaba para qué recurso la aplicación cliente solicitaba autorización. Azure AD calculaba los permisos requeridos por la aplicación en función de la configuración estática en Azure Portal y emitía tokens en consecuencia.

Para aplicaciones que usen el punto de conexión de plataforma de identidad de Microsoft, el mismo de OAuth 2.0 autorizar la solicitud es similar al siguiente:

```text
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&scope=https://graph.windows.net/directory.read%20https://graph.windows.net/directory.write
...
```

Aquí, el parámetro **scope** indica para qué recursos y permisos está solicitando autorización la aplicación. El recurso deseado sigue estando presente en la solicitud; está incluido en cada uno de los valores del parámetro scope. Usar el parámetro de ámbito de esta manera permite que el punto de conexión de la plataforma de identidad de Microsoft sea más compatible con la especificación OAuth 2.0 y alinee más estrechamente con prácticas comunes del sector. También permite que las aplicaciones realizar [consentimiento incremental](#incremental-and-dynamic-consent) : solo solicita permisos cuando la aplicación los necesita en contraposición a por adelantado.

## <a name="well-known-scopes"></a>Ámbitos conocidos

### <a name="offline-access"></a>Acceso sin conexión

Aplicaciones con el punto de conexión de plataforma de identidad de Microsoft pueden requerir el uso de un nuevo permiso conocido para aplicaciones: el `offline_access` ámbito. Todas las aplicaciones deberán solicitar este permiso si necesitan tener acceso a los recursos en nombre de un usuario durante un período de tiempo prolongado, incluso cuando es posible que el usuario no haya estado usando la aplicación activamente. El ámbito `offline_access` se mostrará al usuario en cuadros de diálogo de consentimiento como **Obtener acceso a los datos en cualquier momento**, que el usuario debe aceptar. Solicitar el `offline_access` permiso permitirá a la aplicación web recibir refresh_tokens de OAuth 2.0 desde el punto de conexión de plataforma de identidad de Microsoft. Los token de actualización son de larga duración y se pueden intercambiar por nuevos tokens de actualización de OAuth 2.0 para períodos de acceso más prolongados.

Si la aplicación no solicita el `offline_access` ámbito, no recibirá tokens de actualización. Esto significa que cuando se canjea un código de autorización en el flujo del código de autorización de OAuth 2.0, solo se recibirá un token de acceso desde el punto de conexión `/token`. Ese token de acceso sigue siendo válido durante un breve período de tiempo (normalmente una hora), pero finalmente expirará. En ese momento, la aplicación tendrá que redirigir el usuario de nuevo al punto de conexión `/authorize` para recuperar un nuevo código de autorización. Durante esta redirección, es posible o no que el usuario necesite escribir sus credenciales de nuevo o volver a dar el consentimiento a permisos, según el tipo de aplicación.

Para obtener más información sobre OAuth 2.0, `refresh_tokens`, y `access_tokens`, consulte el [referencia del protocolo de plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

### <a name="openid-profile-and-email"></a>OpenID, perfil y correo electrónico

Históricamente, la más básica inicio de sesión de flujo de OpenID Connect con la plataforma Microsoft identity proporcionaría una gran cantidad de información sobre el usuario en el cuadro *id_token*. Las notificaciones de un id_token pueden incluir el nombre de usuario, el nombre de usuario preferido, la dirección de correo electrónico, el id. de objeto, etc.

La información a la que el ámbito `openid` permite acceder a la aplicación está restringida. El ámbito `openid` solo permitirá que el usuario inicie sesión en la aplicación y que esta reciba un identificador específico de la aplicación para el usuario. Si quiere obtener información personal acerca del usuario en la aplicación, esta tendrá que solicitar permisos adicionales al usuario. Dos nuevos ámbitos, `email` y `profile`, le permitirán solicitar permisos adicionales.

* El ámbito `email` permite que la aplicación acceda a la dirección de correo electrónico principal del usuario a través de la notificación `email` en id_token, suponiendo que el usuario tiene una dirección de correo electrónico direccionable.
* El `profile` ámbito ofrece a su aplicación acceso a toda la demás información básica sobre el usuario, como su nombre, nombre de usuario preferido, identificador y así sucesivamente, en el id_token de objeto.

Estos ámbitos le permiten codificar la aplicación en un modo de divulgación mínima, así que puede pedir al usuario solo el conjunto de información que la aplicación necesita para hacer su trabajo. Para obtener más información sobre estos ámbitos, consulte [la referencia de ámbito de plataforma de identidad de Microsoft](v2-permissions-and-consent.md).

## <a name="token-claims"></a>Notificaciones de token

El punto de conexión de plataforma de identidad de Microsoft emite un conjunto más pequeño de notificaciones en sus tokens de manera predeterminada para que las cargas pequeñas. Si tiene aplicaciones y servicios que tienen una dependencia en una notificación concreta en un token de v1.0 que ya no se proporciona de forma predeterminada en un token de plataforma de identidad de Microsoft, considere el uso de la [notificaciones opcionales](active-directory-optional-claims.md) característica para que incluya esa notificación.

## <a name="limitations"></a>Limitaciones

Hay algunas restricciones para tener en cuenta al usar la plataforma Microsoft identity.

Al compilar aplicaciones que se integran con la plataforma Microsoft identity, deberá decidir si los protocolos de autenticación y de punto de conexión a la plataforma de identidad de Microsoft cumplen con sus necesidades. La plataforma y el punto de conexión v1.0 sigue siendo totalmente compatible y, en algunos aspectos, tiene más características que la plataforma Microsoft identity. Sin embargo, plataforma Microsoft identity [presenta ventajas importantes](azure-ad-endpoint-comparison.md) para desarrolladores.

Esta es una recomendación simplificada para desarrolladores ahora:

* Si desea o necesita admitir cuentas personales de Microsoft en la aplicación, o está escribiendo una nueva aplicación, use la plataforma Microsoft identity. Pero antes, asegúrese de comprender las limitaciones que se analizan en este artículo.
* Si va a migrar o actualizar una aplicación que se basa en SAML, no puede usar la plataforma Microsoft identity. En su lugar, consulte el [Guía de Azure AD v1.0](v1-overview.md).

El punto de conexión de plataforma de identidad de Microsoft evolucionará para eliminar las restricciones indicadas en este caso, por lo que solo deberá usar el punto de conexión de plataforma de identidad de Microsoft. Mientras tanto, use este artículo para determinar si el punto de conexión de plataforma de identidad de Microsoft es adecuada para usted. Seguiremos actualizando este artículo para reflejar el estado actual del punto de conexión de plataforma de identidad de Microsoft. Consúltelo de nuevo para volver a evaluar sus requisitos en relación con las funcionalidades de plataforma de identidad de Microsoft.

### <a name="restrictions-on-app-registrations"></a>Restricciones en los registros de aplicaciones

Para cada aplicación que desee integrar con el punto de conexión de plataforma de identidad de Microsoft, puede crear un registro de aplicación en el nuevo [ **registros de aplicaciones** experimentar](https://aka.ms/appregistrations) en Azure portal. Las aplicaciones de la cuenta de Microsoft existentes no son compatibles con el portal, pero son todas las aplicaciones de Azure AD, independientemente de dónde y cuándo que se han registrado.

Los registros de aplicaciones que admiten cuentas profesionales y educativas y cuentas personales tienen las siguientes limitaciones:

* Solo se permiten dos secretos de aplicación por cada identificador de aplicación.
* Una aplicación que no se registró en un inquilino solo puede administrarse por la cuenta que la registró. No se puede compartir con otros desarrolladores. Este es el caso de la mayoría de aplicaciones que se registraron con una cuenta Microsoft personal en el portal de registros para aplicaciones. Si desea compartir el registro de aplicación con varios desarrolladores, registrar la aplicación en un inquilino mediante el nuevo **registros de aplicaciones** sección del portal de Azure.
* Existen varias restricciones para el formato de la URL de redireccionamiento permitido. Para más información sobre las URL de redireccionamiento, consulte la sección siguiente.

### <a name="restrictions-on-redirect-urls"></a>Restricciones en las URL de redireccionamiento

Las aplicaciones que están registradas para la plataforma Microsoft identity están restringidas a un conjunto limitado de valores de dirección URL de redireccionamiento. La URL de redireccionamiento de aplicaciones y servicios web debe comenzar por el esquema `https`, y todos los valores de URL de redireccionamiento deben compartir un único dominio DNS.  El sistema de registro compara el nombre DNS completo de la URL de redireccionamiento existente con el nombre DNS de la URL de redireccionamiento que va a agregar. `http://localhost` también se admite como una dirección URL de redireccionamiento.  

La solicitud para agregar el nombre DNS presentará un error si se cumple alguna de las condiciones siguientes:  

* El nombre DNS completo de la nueva URL de redireccionamiento no coincide con el nombre DNS de la URL de redireccionamiento existente.
* El nombre DNS completo de la nueva URL de redireccionamiento no es un subdominio de la URL de redireccionamiento existente.

#### <a name="example-1"></a>Ejemplo 1

Si la aplicación tiene una dirección URL de redireccionamiento de `https://login.contoso.com`, puede agregar una dirección URL de redireccionamiento donde el nombre DNS coincide exactamente, como se muestra en el ejemplo siguiente:

`https://login.contoso.com/new`

O bien, puede hacer referencia a un subdominio DNS de login.contoso.com, tal como se muestra en el ejemplo siguiente:

`https://new.login.contoso.com`

#### <a name="example-2"></a>Ejemplo 2

Si desea que una aplicación tenga `login-east.contoso.com` y `login-west.contoso.com` como direcciones URL de redireccionamiento, debe agregar las URL de redireccionamiento en el orden siguiente:

`https://contoso.com`  
`https://login-east.contoso.com`  
`https://login-west.contoso.com`  

Puede agregar los dos últimos porque son subdominios de la primera dirección URL de redireccionamiento, contoso.com.

Puede tener solo 20 direcciones URL de respuesta para una aplicación determinada: este límite se aplica a todos los tipos de aplicación que es compatible con el registro (aplicación de página única (SPA), native client, web app y servicio).  

Para obtener información sobre cómo registrar una aplicación para su uso con la plataforma Microsoft identity, vea [registrar una aplicación mediante la nueva experiencia de los registros de aplicación](quickstart-register-app.md).

### <a name="restrictions-on-libraries-and-sdks"></a>Restricciones en las bibliotecas y SDK

Actualmente, la compatibilidad con bibliotecas para el punto de conexión de plataforma de identidad de Microsoft es limitado. Si desea usar el punto de conexión de plataforma de identidad de Microsoft en una aplicación de producción, tienes estas opciones:

* Si va a compilar una aplicación web, puede utilizar el middleware de servidor disponible con carácter general para el inicio de sesión y validación de tokens. Incluye el software intermedio OWIN OpenID Connect para ASP.NET y el complemento NodeJS Passport. Para obtener ejemplos de código que usar el software intermedio de Microsoft, consulte el [Introducción a la plataforma de identidad de Microsoft](v2-overview.md#getting-started) sección.
* Si va a compilar una aplicación de escritorio o móvil, puede usar una de las bibliotecas de autenticación de Microsoft (MSAL). Estas bibliotecas están disponibles con carácter general o por lo que en una versión preliminar compatible con producción, es seguro para usarlos en aplicaciones de producción. Puede obtener más información sobre los términos y condiciones de la versión preliminar y las bibliotecas disponibles en la [referencia de bibliotecas de autenticación](reference-v2-libraries.md).
* Para las plataformas no cubiertas por las bibliotecas de Microsoft, puede integrar con el punto de conexión de plataforma de identidad de Microsoft directamente enviando y recibiendo mensajes de protocolo en el código de aplicación. Los protocolos OpenID Connect y OAuth [se documentan explícitamente](active-directory-v2-protocols.md) para ayudarle a realizar dicha integración.
* Por último, puede usar las bibliotecas de código abierto de OpenID Connect y OAuth para integrarse con el punto de conexión de plataforma de identidad de Microsoft. El punto de conexión de plataforma de identidad de Microsoft debe ser compatible con muchas bibliotecas de protocolo abierto sin cambios. La disponibilidad de estos tipos de bibliotecas varía según el lenguaje y la plataforma. Los sitios web [OpenID Connect](https://openid.net/connect/) y [OAuth 2.0](https://oauth.net/2/) mantienen una lista de las implementaciones populares. Para obtener más información, consulte [bibliotecas de autenticación y la plataforma de identidad de Microsoft](reference-v2-libraries.md)y la lista de bibliotecas de cliente de código abierto y ejemplos que se han probado con el punto de conexión de plataforma de identidad de Microsoft.
* Como referencia, el `.well-known` es el punto de conexión para el extremo en la plataforma de identidad de Microsoft comunes `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`. Reemplace `common` con el identificador de inquilino para obtener los datos específicos para el inquilino.  

### <a name="protocol-changes"></a>Cambios en el protocolo

El punto de conexión de plataforma de identidad de Microsoft no admite SAML o WS-Federation; solo es compatible con OpenID Connect y OAuth 2.0.  Los cambios importantes en los protocolos OAuth 2.0 desde el punto de conexión v1.0 son: 

* La notificación `email` se devuelve si se configura una notificación opcional **o** scope=email se ha especificado en la solicitud. 
* El parámetro `scope` ahora se admite en lugar del parámetro `resource`.  
* Muchas respuestas se han modificado para que sean más compatibles con la especificación OAuth 2.0; por ejemplo, devolver correctamente `expires_in` como int en lugar de una cadena.  

Para entender mejor el ámbito de funcionalidad de protocolo admitida en el punto de conexión de plataforma de identidad de Microsoft, consulte [referencia del protocolo OAuth 2.0 y OpenID Connect](active-directory-v2-protocols.md).

#### <a name="saml-restrictions"></a>Restricciones de SAML

Si usó Active Directory Authentication Library (ADAL) en aplicaciones de Windows, es posible que haya aprovechado de la autenticación integrada de Windows, que usa la concesión de aserción de lenguaje de marcado de aserción de seguridad (SAML). Con esta concesión, los usuarios de los inquilinos de Azure AD federado pueden autenticarse de manera silenciosa con su instancia local de Active Directory sin escribir las credenciales. No se admite la concesión de aserción de SAML en el punto de conexión de plataforma de identidad de Microsoft.
