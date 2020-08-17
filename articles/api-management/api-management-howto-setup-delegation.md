---
title: Delegación de registros de usuario y suscripciones a producto
description: Obtenga información acerca de cómo delegar el registro de usuario y la suscripción de producto un tercero en la administración de la API de Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.assetid: 8b7ad5ee-a873-4966-a400-7e508bbbe158
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 07/30/2020
ms.author: apimpm
ms.openlocfilehash: e7f2fb966aa323063220bc798706c8401745ba20
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461007"
---
# <a name="how-to-delegate-user-registration-and-product-subscription"></a>Delegación de registros de usuario y suscripciones a producto

La delegación le permite usar su sitio web actual para controlar el inicio de sesión y la suscripción de los desarrolladores, así como sus suscripciones a productos, en contraposición al uso de la funcionalidad integrada en el portal para desarrolladores. Permite que el sitio web sea propietario de los datos de usuario y realice la validación de estos pasos de forma personalizada.

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="delegating-developer-sign-in-and-sign-up"></a><a name="delegate-signin-up"> </a>Delegación de inicios de sesión y suscripciones de desarrolladores

Para delegar el inicio de sesión y la suscripción para desarrolladores al sitio web existente, deberá crear un punto de conexión de delegación especial en su sitio. Debe actuar como un punto de entrada para cualquier solicitud de este tipo iniciada desde el portal para desarrolladores de API Management.

El flujo de trabajo final será el siguiente:

1. El desarrollador hace clic en el enlace de inicio de sesión o de suscripción que se encuentra en el portal para desarrolladores de API Management.
2. El explorador se redirige al extremo de delegación.
3. Como respuesta, el punto de conexión de delegación se redirige a la interfaz de usuario o la presenta para pedir al usuario que inicie sesión o se suscriba.
4. Una vez conseguido, se redirige de nuevo al usuario a la página del portal para desarrolladores de API Management de la que partió.

Para empezar, configuremos primero Administración de API para que dirija las solicitudes a través del extremo de delegación. En Azure Portal, busque **Seguridad** en el recurso de API Management y, luego, haga clic en el elemento de **Delegación**. Haga clic en la casilla para activar "Delegar inicio de sesión y suscripción".

![Delegation page][api-management-delegation-signin-up]

* Determine cuál será la dirección URL del extremo especial de delegación y escríbala en el campo **Dirección URL del extremo de delegación** . 
* En el campo Clave de autenticación de delegación, escriba el secreto que se usará para procesar una firma suministrada para su comprobación con objeto de garantizar que la solicitud procede efectivamente de Azure API Management. Puede hacer clic en el botón **generar** para que API Management genere de forma aleatoria una clave en su lugar.

Ahora debe crear el **extremo de delegación**. Este tiene que realizar varias acciones:

1. Recibir una solicitud de la forma siguiente:
   
   > *http:\//www.yourwebsite.com/apimdelegation?operation=SignIn&returnUrl={URL de la página de origen}&salt={string}&amp;sig={string}*
   
    Parámetros de consulta en el caso de inicio de sesión o suscripción:
   
   * **operation**: identifica el tipo de solicitud de delegación del que se trata. Solo puede ser **SignIn** en este caso.
   * **returnUrl**: la dirección URL de la página en la que el usuario hizo clic en un vínculo de suscripción o de inicio de sesión.
   * **salt**: una cadena salt especial que se usa para procesar un hash de seguridad
   * **sig**: un hash de seguridad procesado que se comparará con su propio hash procesado
2. Compruebe que la solicitud procede de Azure API Management (opcional, pero especialmente recomendado por motivos de seguridad).
   
   * Procese un hash HMAC-SHA512 de una cadena según los parámetros de consulta **returnUrl** y **salt** ([se proporciona código de ejemplo a continuación]):
     
     > HMAC(**salt** + '\n' + **returnUrl**)
     > 
     > 
   * Compare el hash procesado anteriormente con el valor del parámetro de consulta **sig** . Si los dos hashes coinciden, vaya a paso siguiente; de lo contrario, deniegue la solicitud.
3. Compruebe que ha recibido una solicitud para inicio de sesión/suscripción: el parámetro de consulta **operation** se establecerá en "**SignIn**".
4. Presente al usuario la interfaz de usuario para que inicie sesión o se suscriba.
5. Si el usuario se suscribe, hay que crear la cuenta correspondiente en Administración de API. [Cree un usuario] con la API de REST de API Management. Al hacerlo, asegúrese de que el identificador de usuario se establece en el mismo valor que existe en su almacén de usuario o en un identificador al que pueda realizar el seguimiento.
6. Cuando el usuario se autentique correctamente:
   
   * [solicite un token de inicio de sesión único (SSO)] a través de la API de REST de API Management
   * anexe un parámetro de consulta returnUrl a la URL de SSO que se recibió de la llamada de API anterior:
     
     > por ejemplo, `https://customer.portal.azure-api.net/signin-sso?token&returnUrl=/return/url` 
     > 
     > 
   * redirija al usuario a la URL producida anteriormente

Además de la operación **SignIn**, también puede realizar la administración de cuentas siguiendo los pasos anteriores y utilizando una de las siguientes operaciones:

* **ChangePassword**
* **ChangeProfile**
* **CloseAccount**
* **SignOut**

Debe pasar los siguientes parámetros de consulta para las operaciones de administración de cuenta.

* **operation**: identifica qué tipo de solicitud de delegación es (ChangePassword, ChangeProfile o CloseAccount)
* **userId**: el identificador de usuario de la cuenta para administrar
* **salt**: una cadena salt especial que se usa para procesar un hash de seguridad
* **sig**: un hash de seguridad procesado que se comparará con su propio hash procesado

## <a name="delegating-product-subscription"></a><a name="delegate-product-subscription"> </a>Delegación de suscripciones a productos

La delegación de una suscripción a productos funciona de forma similar a la delegación de inicio de sesión y suscripción de usuario. El flujo de trabajo final sería el siguiente:

1. El desarrollador selecciona un producto en el portal para desarrolladores de API Management y hace clic en el botón Suscribir.
2. El explorador se redirige al extremo de delegación.
3. El extremo de delegación realiza los pasos necesarios para la suscripción al producto. Usted es el encargado de diseñar estos pasos. Pueden implicar la redirección a otra página para solicitar información de facturación, la formulación de otras preguntas o simplemente el almacenamiento de la información sin que se requiera ninguna acción del usuario.

Para habilitar la funcionalidad, en la página **Delegación**, haga clic en **Delegar suscripción de productos**.

A continuación, asegúrese de que el extremo de delegación realiza las siguientes acciones:

1. Recibir una solicitud de la forma siguiente:
   
   > *http:\//www.yourwebsite.com/apimdelegation?operation={operación}&amp;productId={producto al que se suscribe}&amp;userId={usuario que realiza la solicitud}&amp;salt={cadena}&amp;sig={cadena}*
   >
   
    Parámetros de consulta en el caso de suscripción a producto:
   
   * **operation**: identifica el tipo de solicitud de delegación del que se trata. En las solicitudes de suscripción a producto las opciones válidas son:
     * "Subscribe": una solicitud para suscribir al usuario a un producto determinado con el id. especificado (consulte más información a continuación).
     * "Unsubscribe": una solicitud para cancelar la suscripción de un usuario a un producto.
     * "Renew": una solicitud para renovar una suscripción (por ejemplo, que esté a punto de expirar).
   * **productId**: en *Suscribirse*, el id. del producto al que el usuario solicitó suscribirse.
   * **subscriptionId**: en *Cancelar suscripción* y *Renovar*, el identificador de la suscripción del producto.
   * **userId**: en *Suscribirse*, el id. del usuario para el que se realiza la solicitud.
   * **salt**: una cadena salt especial que se usa para procesar un hash de seguridad
   * **sig**: un hash de seguridad procesado que se comparará con su propio hash procesado

2. Compruebe que la solicitud procede de Azure API Management (opcional, pero especialmente recomendado por motivos de seguridad).
   
   * Procesar un hash HMAC-SHA512 de una cadena en función de los parámetros de consulta **productId**, **userId** y **salt**:
     
     > HMAC(**salt** + '\n' + **productId** + '\n' + **userId**)
     > 
     > 
   * Compare el hash procesado anteriormente con el valor del parámetro de consulta **sig** . Si los dos hashes coinciden, vaya a paso siguiente; de lo contrario, deniegue la solicitud.
3. Procese cualquier suscripción a producto en función del tipo de operación solicitada en **operation**; por ejemplo, facturación, preguntas adicionales, etc.
4. Tras la correcta suscripción del usuario al producto por su parte, suscriba al usuario al producto de API Management [llamando a la API de REST para las suscripciones].

## <a name="example-code"></a><a name="delegate-example-code"> </a>Ejemplo de código

Estos ejemplos de código enseñan cómo:

* Tomar la *clave de validación de delegación*, que se establece en la pantalla de delegación del portal del publicador
* Crear un HMAC que, después, se usa para validar la firma, probando la validez del valor de returnUrl que se pasó.

El mismo código funciona para productId y userId con pequeñas modificaciones.

**Código C# para generar el hash de returnUrl**

```csharp
using System.Security.Cryptography;

string key = "delegation validation key";
string returnUrl = "returnUrl query parameter";
string salt = "salt query parameter";
string signature;
using (var encoder = new HMACSHA512(Convert.FromBase64String(key)))
{
    signature = Convert.ToBase64String(encoder.ComputeHash(Encoding.UTF8.GetBytes(salt + "\n" + returnUrl)));
    // change to (salt + "\n" + productId + "\n" + userId) when delegating product subscription
    // compare signature to sig query parameter
}
```

**Código NodeJS para generar el hash de returnUrl**

```
var crypto = require('crypto');

var key = 'delegation validation key'; 
var returnUrl = 'returnUrl query parameter';
var salt = 'salt query parameter';

var hmac = crypto.createHmac('sha512', new Buffer(key, 'base64'));
var digest = hmac.update(salt + '\n' + returnUrl).digest();
// change to (salt + "\n" + productId + "\n" + userId) when delegating product subscription
// compare signature to sig query parameter

var signature = digest.toString('base64');
```

> [!IMPORTANT]
> Debe [volver a publicar el portal para desarrolladores](api-management-howto-developer-portal-customize.md#publish) para que los cambios en la delegación surtan efecto.

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de la delegación, vea el siguiente vídeo:

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Delegating-User-Authentication-and-Product-Subscription-to-a-3rd-Party-Site/player]
> 
> 

[Delegating developer sign in and sign up]: #delegate-signin-up
[Delegating product subscription]: #delegate-product-subscription
[solicite un token de inicio de sesión único (SSO)]: /rest/api/apimanagement/2019-12-01/user/generatessourl
[Cree un usuario]: /rest/api/apimanagement/2019-12-01/user/createorupdate
[llamando a la API de REST para las suscripciones]: /rest/api/apimanagement/2019-12-01/subscription/createorupdate
[Next steps]: #next-steps
[se proporciona código de ejemplo a continuación]: #delegate-example-code

[api-management-delegation-signin-up]: ./media/api-management-howto-setup-delegation/api-management-delegation-signin-up.png
