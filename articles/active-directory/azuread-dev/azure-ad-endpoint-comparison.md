---
title: Motivos para actualizar a la Plataforma de identidad de Microsoft (v2.0) | Azure
description: Conozca las diferencias entre el punto de conexión de la Plataforma de identidad de Microsoft (v2.0) y el punto de conexión de Azure Active Directory (Azure AD) v1.0 y descubra las ventajas de actualizar a la versión 2.0.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.workload: identity
ms.topic: conceptual
ms.date: 07/17/2020
ms.author: ryanwi
ms.reviewer: saeeda, hirsin, jmprieur, sureshja, jesakowi, lenalepa, kkrishna, negoe
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: c6e59ab0432ad2b7bdccb5ce9916e85eb6d95048
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88116400"
---
# <a name="why-update-to-microsoft-identity-platform-v20"></a>Motivos para actualizar a la Plataforma de identidad de Microsoft (v2.0)

Al desarrollar una nueva aplicación, es importante conocer las diferencias entre los puntos de conexión de la Plataforma de identidad de Microsoft (v2.0) y de Azure Active Directory (v1.0). En este artículo se tratan las principales diferencias entre los puntos de conexión y algunas limitaciones existentes para la Plataforma de identidad de Microsoft.

> [!NOTE]
> No todas las características y escenarios de Azure AD son compatibles con el punto de conexión de la Plataforma de identidad de Microsoft. Para determinar si debe usar el punto de conexión de la plataforma de identidad de Microsoft, conozca las [limitaciones de dicha plataforma](#limitations).

## <a name="who-can-sign-in"></a>Quién puede iniciar sesión

![Quién puede iniciar sesión con los puntos de conexión v1.0 y v2.0](media/azure-ad-endpoint-comparison/who-can-signin.svg)

* El punto de conexión v1.0 solo permite iniciar sesión en la aplicación con cuentas profesionales y educativas (Azure AD).
* El punto de conexión de la Plataforma de identidad de Microsoft permite iniciar sesión con cuentas profesionales y educativas de Azure AD y con cuentas Microsoft personales (MSA), como hotmail.com, outlook.com y msn.com.
* Ambos puntos de conexión aceptan inicios de sesión de *[usuarios invitados](../external-identities/what-is-b2b.md)* de un directorio de Azure AD para las aplicaciones configuradas como *[inquilino único](../develop/single-and-multi-tenant-apps.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json)* o para las aplicaciones configuradas como *multiinquilino* para el punto de conexión específico del inquilino (`https://login.microsoftonline.com/{TenantId_or_Name}`).

El punto de conexión de la Plataforma de identidad de Microsoft le permite escribir aplicaciones que aceptan inicios de sesión desde cuentas Microsoft personales y cuentas profesionales y educativas. Esto le permite escribir la aplicación con independencia absoluta de la cuenta. Por ejemplo, si su aplicación llama a [Microsoft Graph](https://graph.microsoft.io), las cuentas profesionales tendrán a su disposición algunos datos y funcionalidades adicionales, como los sitios de SharePoint o datos de directorio. Pero, para muchas acciones, como [leer correo de un usuario](/graph/api/user-list-messages?view=graph-rest-1.0), el mismo código puede acceder al correo electrónico tanto para cuentas personales como profesionales y educativas.

En el caso del punto de conexión de la Plataforma de identidad de Microsoft, puede usar la Biblioteca de autenticación de Microsoft (MSAL) para obtener acceso a entornos educativos, de consumidor y de empresa. El punto de conexión v1.0 de Azure AD acepta inicios de sesión solo desde cuentas profesionales y educativas.

## <a name="incremental-and-dynamic-consent"></a>Consentimiento incremental y dinámico

Las aplicaciones que usan el punto de conexión v1.0 de Azure AD tienen que especificar los permisos de OAuth 2.0 necesarios de antemano, por ejemplo:

![Ejemplo que muestra la interfaz de usuario de registro de permisos](./media/azure-ad-endpoint-comparison/app-reg-permissions.png)

Los permisos establecidos directamente en el registro de la aplicación son **estáticos**. Aunque los permisos estáticos de la aplicación definidos en Azure Portal mantenían un código ordenado y sencillo, presenta algunos problemas para los desarrolladores:

* Una aplicación tiene que solicitar todos los permisos que podría necesitar alguna vez tras el primer inicio de sesión del usuario. Esto puede originar una lista larga de permisos, lo que desanimaría a los usuarios finales de aprobar el acceso de la aplicación en el inicio de sesión inicial.

* La aplicación tiene que conocer por adelantado todos los recursos a los que va a tener acceso alguna vez. Era difícil crear aplicaciones que podrían tener acceso a un número arbitrario de recursos.

Con el punto de la Plataforma de identidad de Microsoft, puede ignorar los permisos estáticos definidos en la información de registro de la aplicación en Azure Portal y solicitar permisos incrementales en su lugar, lo que significa solicitar un conjunto mínimo de permisos por adelantado y aumentar más con el tiempo a medida que el cliente usa características de la aplicación. Para ello, puede especificar los ámbitos que la aplicación necesita en cualquier momento incluyendo los nuevos ámbitos en el parámetro `scope` cuando solicite un token de acceso. No es necesario predefinirlos en la información de registro de la aplicación. Si el usuario aún no ha dado su consentimiento a los ámbitos nuevos que se agregan a la solicitud, se le pedirá que dé su consentimiento solo a los nuevos permisos. Para obtener más información, consulte sobre [permisos, consentimiento y ámbitos](../develop/v2-permissions-and-consent.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

Permitir que una aplicación solicite permisos dinámicamente mediante el parámetro `scope` da a los desarrolladores un control total sobre la experiencia del usuario. También puede adelantar la experiencia de consentimiento y pedir todos los permisos en una solicitud de autorización inicial. Si su aplicación requiere un gran número de permisos, puede elegir recopilarlos del usuario de forma incremental, a medida que intentan usar determinadas características de la aplicación con el tiempo.

El consentimiento del administrador que se realiza en nombre de una organización sigue requriendo los permisos estáticos registrados para la aplicación, por lo que debería establecer esos permisos para las aplicaciones en el portal de registro de aplicaciones si necesita que un administrador dé su consentimiento en nombre de toda la organización. Esto reduce los ciclos que el administrador de la organización necesita para instalar la aplicación.

## <a name="scopes-not-resources"></a>Ámbitos, no recursos

Las aplicaciones que usan el punto de conexión v1.0 pueden comportarse como un **recurso** o como un destinatario de tokens. Un recurso puede definir varios **ámbitos** o **oAuth2Permissions** que comprende, lo que permite a las aplicaciones cliente solicitar tokens para ese recurso para un conjunto determinado de ámbitos. Piense en Microsoft Graph API como ejemplo de un recurso:

* Identificador de recursos o `AppID URI`: `https://graph.microsoft.com/`
* Los ámbitos, o `oAuth2Permissions`: `Directory.Read`, `Directory.Write`, etc.

Esto es válido para el punto de conexión de la Plataforma de identidad de Microsoft. Una aplicación todavía se puede comportar como recurso, definir ámbitos y ser identificada por un URI. Las aplicaciones cliente todavía pueden solicitar acceso a esos ámbitos. Sin embargo, ha cambiado la manera en que un cliente solicita esos permisos.

Para el punto de conexión v1.0, una solicitud de autorización de OAuth 2.0 para Azure AD podría haber tenido un aspecto similar al siguiente:

```text
GET https://login.microsoftonline.com/common/oauth2/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&resource=https://graph.microsoft.com/
...
```

Aquí, el parámetro **resource** indicaba para qué recurso la aplicación cliente solicitaba autorización. Azure AD calculaba los permisos requeridos por la aplicación en función de la configuración estática en Azure Portal y emitía tokens en consecuencia.

Para las aplicaciones que usan el punto de conexión de la Plataforma de identidad de Microsoft, la misma solicitud de autorización de OAuth 2.0 tendrá el siguiente aspecto:

```text
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&scope=https://graph.microsoft.com/directory.read%20https://graph.microsoft.com/directory.write
...
```

Aquí, el parámetro **scope** indica para qué recursos y permisos está solicitando autorización la aplicación. El recurso deseado sigue estando presente en la solicitud; está incluido en cada uno de los valores del parámetro scope. El uso del parámetro scope de esta manera permite que el punto de conexión de la Plataforma de identidad de Microsoft respete en mayor medida la especificación de OAuth 2.0 y se alinee más estrechamente con prácticas comunes del sector. También permite que las aplicaciones realicen un [consentimiento incremental](#incremental-and-dynamic-consent): solo solicita permisos cuando la aplicación los requiere, en lugar de hacerlo por adelantado.

## <a name="well-known-scopes"></a>Ámbitos conocidos

### <a name="offline-access"></a>Acceso sin conexión

Es posible que las aplicaciones que usan el punto de conexión de la Plataforma de identidad de Microsoft requieran el uso de un nuevo permiso conocido en las aplicaciones: el ámbito `offline_access`. Todas las aplicaciones deberán solicitar este permiso si necesitan tener acceso a los recursos en nombre de un usuario durante un período de tiempo prolongado, incluso cuando es posible que el usuario no haya estado usando la aplicación activamente. El ámbito `offline_access` se mostrará al usuario en cuadros de diálogo de consentimiento como **Obtener acceso a los datos en cualquier momento**, que el usuario debe aceptar. Solicitar el permiso `offline_access` permitirá a su aplicación web recibir refresh_tokens de OAuth 2.0 desde el extremo de la Plataforma de identidad de Microsoft. Los token de actualización son de larga duración y se pueden intercambiar por nuevos tokens de actualización de OAuth 2.0 para períodos de acceso más prolongados.

Si la aplicación no solicita el ámbito `offline_access`, no recibirá tokens de actualización. Esto significa que cuando se canjea un código de autorización en el flujo del código de autorización de OAuth 2.0, solo se recibirá un token de acceso desde el punto de conexión `/token`. Ese token de acceso sigue siendo válido durante un breve período de tiempo (normalmente una hora), pero finalmente expirará. En ese momento, la aplicación tendrá que redirigir el usuario de nuevo al punto de conexión `/authorize` para recuperar un nuevo código de autorización. Durante esta redirección, es posible o no que el usuario necesite escribir sus credenciales de nuevo o volver a dar el consentimiento a permisos, según el tipo de aplicación.

Para más información sobre OAuth 2.0, `refresh_tokens` y `access_tokens`, consulte la [referencia del protocolo de la Plataforma de identidad de Microsoft](../develop/active-directory-v2-protocols.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

### <a name="openid-profile-and-email"></a>OpenID, perfil y correo electrónico

Tradicionalmente, el flujo más básico de inicio de sesión de OpenID Connect en la Plataforma de identidad de Microsoft ofrecería gran cantidad de información sobre el usuario en el *id_token* resultante. Las notificaciones de un id_token pueden incluir el nombre de usuario, el nombre de usuario preferido, la dirección de correo electrónico, el id. de objeto, etc.

La información a la que el ámbito `openid` permite acceder a la aplicación está restringida. El ámbito `openid` solo permitirá que el usuario inicie sesión en la aplicación y que esta reciba un identificador específico de la aplicación para el usuario. Si quiere obtener información personal acerca del usuario en la aplicación, esta tendrá que solicitar permisos adicionales al usuario. Dos nuevos ámbitos, `email` y `profile`, le permitirán solicitar permisos adicionales.

* El ámbito `email` permite que la aplicación acceda a la dirección de correo electrónico principal del usuario a través de la notificación `email` en id_token, suponiendo que el usuario tiene una dirección de correo electrónico direccionable.
* El ámbito `profile` ofrece a la aplicación acceso a toda la demás información básica sobre el usuario, como su nombre, el nombre de usuario preferido, el identificador de objeto y demás en el id_token.

Estos ámbitos le permiten codificar la aplicación en un modo de divulgación mínima, así que puede pedir al usuario solo el conjunto de información que la aplicación necesita para hacer su trabajo. Para obtener más información sobre estos ámbitos, consulte [la referencia de los ámbitos de la Plataforma de identidad de Microsoft](../develop/v2-permissions-and-consent.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

## <a name="token-claims"></a>Notificaciones de token

El punto de conexión de la Plataforma de identidad de Microsoft emite un conjunto más pequeño de notificaciones en sus tokens de manera predeterminada para mantener el tamaño reducido de las cargas. Si tiene aplicaciones y servicios que tienen una dependencia de una notificación concreta en un token de v1.0 que ya no se proporciona de manera predeterminada en un token de la Plataforma de identidad de Microsoft, considere el uso de la característica [notificaciones opcionales](../develop/active-directory-optional-claims.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json) para incluir esa notificación.

> [!IMPORTANT]
> Los puntos de conexión de las versiones v1.0 y v2.0 pueden emitir tokens v1.0 y v2.0. Los tokens id_tokens *siempre* coinciden con el punto de conexión desde el que se los solicita, y los tokens de acceso *siempre* coinciden con el formato que espera la API web a la que el cliente llamará mediante ese token.  Por lo tanto, si la aplicación usa el punto de conexión v2.0 para obtener un token para llamar a Microsoft Graph, el cual espera tokens de acceso de formato v1.0, la aplicación recibirá un token con el formato v1.0.  

## <a name="limitations"></a>Limitaciones

Hay que tener en cuenta algunas restricciones cuando se usa el punto de conexión de la Plataforma de identidad de Microsoft.

Cuando compile aplicaciones que se integran con la Plataforma de identidad de Microsoft, debe decidir si los protocolos de autenticación y el punto de conexión de la Plataforma de identidad de Microsoft satisfacen sus necesidades. La plataforma y el punto de conexión v1.0 se siguen admitiendo totalmente y, en algunos aspectos, tienen más características que la Plataforma de identidad de Microsoft. Sin embargo, la Plataforma de identidad de Microsoft [presenta ventajas importantes](azure-ad-endpoint-comparison.md) para los desarrolladores.

Esta es nuestra recomendación simplificada para desarrolladores en este momento:

* Si quiere o necesita admitir cuentas Microsoft personales en la aplicación, o está escribiendo una nueva aplicación, use la Plataforma de identidad de Microsoft. Pero antes, asegúrese de comprender las limitaciones que se analizan en este artículo.
* Si va a migrar o actualizar una aplicación que se basa en SAML, no puede usar la Plataforma de identidad de Microsoft. En su lugar, consulte la [Guía de v1.0 de Azure AD](v1-overview.md).

Desarrollaremos el punto de conexión de la Plataforma de identidad de Microsoft para eliminar las restricciones que mencionamos en este artículo, por lo que siempre debe usar este punto de conexión. Mientras tanto, use este artículo para determinar si el punto de conexión de la Plataforma de identidad de Microsoft es correcto para usted. Actualizaremos este artículo constantemente para reflejar el estado actual del punto de conexión de la Plataforma de identidad de Microsoft. Consúltelo de nuevo para volver a evaluar los requisitos en relación con las funcionalidades de la Plataforma de identidad de Microsoft.

### <a name="restrictions-on-app-registrations"></a>Restricciones en los registros de aplicaciones

Para cada aplicación que quiera integrar con el punto de conexión de la Plataforma de identidad de Microsoft, puede crear un registro de aplicación en la nueva [experiencia de **Registros de aplicaciones**](https://aka.ms/appregistrations) de Azure Portal. Las aplicaciones de cuenta Microsoft existentes no son compatibles con el portal, pero están todas las aplicaciones de Azure AD, con independencia de dónde o cuándo se registraron.

Los registros de aplicaciones que admiten cuentas profesionales y educativas y cuentas personales tienen las siguientes limitaciones:

* Solo se permiten dos secretos de aplicación por cada identificador de aplicación.
* Una aplicación que no se registró en un inquilino solo puede administrarse por la cuenta que la registró. No se puede compartir con otros desarrolladores. Este es el caso de la mayoría de aplicaciones que se registraron con una cuenta Microsoft personal en el portal de registros para aplicaciones. Si quiere compartir el registro de la aplicación con varios desarrolladores, registre la aplicación en un inquilino mediante la sección **Registros de aplicaciones** de Azure Portal.
* Existen varias restricciones para el formato de la URL de redireccionamiento permitido. Para más información sobre las URL de redireccionamiento, consulte la sección siguiente.

### <a name="restrictions-on-redirect-urls"></a>Restricciones en las URL de redireccionamiento

Para obtener la información más actualizada sobre las restricciones en las direcciones URL de redireccionamiento para las aplicaciones registradas en la Plataforma de identidad de Microsoft, consulte [Limitaciones y restricciones de URI de redireccionamiento o URL de respuesta](../develop/reply-url.md) en la documentación de la Plataforma de identidad de Microsoft.

Para obtener información sobre cómo registrar una aplicación para usar con la Plataforma de identidad de Microsoft, consulte [Registro de una aplicación con la nueva experiencia de registros de aplicaciones](../develop/quickstart-register-app.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

### <a name="restrictions-on-libraries-and-sdks"></a>Restricciones en las bibliotecas y SDK

Actualmente, la compatibilidad con bibliotecas para el punto de conexión de la Plataforma de identidad de Microsoft es limitada. Si quiere usar el punto de conexión de la Plataforma de identidad de Microsoft en una aplicación de producción, tiene las opciones siguientes:

* Si está creando una aplicación web, puede usar sin ningún riesgo nuestro software intermedio del lado del servidor disponible con carácter general, para realizar el inicio de sesión y la validación de tokens. Incluye el software intermedio OWIN OpenID Connect para ASP.NET y el complemento NodeJS Passport. Para ejemplos de código que usan el software intermedio de Microsoft, consulte la sección [Introducción a la Plataforma de identidad de Microsoft](../develop/v2-overview.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json#getting-started).
* Si crea una aplicación de escritorio o para dispositivos móviles, puede usar una de las bibliotecas de autenticación de Microsoft (MSAL). Estas bibliotecas están disponible con carácter general o en una versión preliminar compatible con producción, por lo que su uso en aplicaciones de producción es seguro. Puede obtener más información sobre los términos y condiciones de la versión preliminar y las bibliotecas disponibles en la [referencia de bibliotecas de autenticación](../develop/reference-v2-libraries.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).
* En el caso de otras plataformas no cubiertas por las bibliotecas de Microsoft, pueden integrarse con el punto de conexión de la Plataforma de identidad de Microsoft enviando y recibiendo mensajes de protocolo directamente en el código de la aplicación. Los protocolos OpenID Connect y OAuth [se documentan explícitamente](../develop/active-directory-v2-protocols.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json) para ayudarle a realizar dicha integración.
* Por último, puede usar las bibliotecas de código abierto de OpenID Connect y OAuth para integrarse con el punto de conexión de la Plataforma de identidad de Microsoft. El punto de conexión de la Plataforma de identidad de Microsoft debe ser compatible con muchas bibliotecas de código abierto de los protocolos sin cambios. La disponibilidad de estos tipos de bibliotecas varía según el lenguaje y la plataforma. Los sitios web [OpenID Connect](https://openid.net/connect/) y [OAuth 2.0](https://oauth.net/2/) mantienen una lista de las implementaciones populares. Para más información, consulte [Plataforma de identidad de Microsoft y bibliotecas de autenticación](../develop/reference-v2-libraries.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json), así como la lista de bibliotecas cliente de código abierto y los ejemplos que se probaron con el punto de conexión de la Plataforma de identidad de Microsoft.
* Como referencia, el punto de conexión `.well-known` para el punto de conexión común de la Plataforma de identidad de Microsoft es `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`. Reemplace `common` con el identificador de inquilino para obtener los datos específicos para el inquilino.  

### <a name="protocol-changes"></a>Cambios en el protocolo

El punto de conexión de la Plataforma de identidad de Microsoft no admite SAML ni WS-Federation; solo admite OpenID Connect y OAuth 2.0.  Los cambios importantes en los protocolos OAuth 2.0 desde el punto de conexión v1.0 son: 

* La notificación `email` se devuelve si se configura una notificación opcional **o** scope=email se ha especificado en la solicitud. 
* El parámetro `scope` ahora se admite en lugar del parámetro `resource`.  
* Muchas respuestas se han modificado para que sean más compatibles con la especificación OAuth 2.0; por ejemplo, devolver correctamente `expires_in` como int en lugar de una cadena.  

Para comprender mejor el alcance de la funcionalidad de protocolo que se admite en el punto de conexión de la Plataforma de identidad de Microsoft, consulte nuestra [referencia a los protocolos OpenID Connect y OAuth 2.0](../develop/active-directory-v2-protocols.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

#### <a name="saml-restrictions"></a>Restricciones de SAML

Si ha usado la biblioteca de autenticación de Active Directory (ADAL) en aplicaciones de Windows, es posible que haya aprovechado la autenticación integrada de Windows, que usa la concesión de aserción de Lenguaje de marcado de aserción de seguridad (SAML). Con esta concesión, los usuarios de los inquilinos de Azure AD federado pueden autenticarse de manera silenciosa con su instancia local de Active Directory sin escribir las credenciales. La concesión de aserción de SAML no se admite en el punto de conexión de la Plataforma de identidad de Microsoft.