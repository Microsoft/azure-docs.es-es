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
ms.devlang: na
ms.topic: article
ms.date: 04/04/2019
ms.author: apimpm
ms.openlocfilehash: 4db99f23019b34e7361e3ead4096939b9499320d
ms.sourcegitcommit: 17411cbf03c3fa3602e624e641099196769d718b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65518089"
---
# <a name="how-to-delegate-user-registration-and-product-subscription"></a>Delegación de registros de usuario y suscripciones a producto

La delegación permite que usa el sitio Web existente para controlar el inicio de sesión para desarrolladores o Suscríbase y suscripciones a productos, en lugar de usar la funcionalidad integrada en el portal para desarrolladores. Esto habilita su sitio web como propietario de los datos de usuario para poder realizar la validación de estos pasos de forma personalizada.

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="delegate-signin-up"> </a>Delegar para desarrolladores iniciar sesión y registrarse

Para delegar el inicio de sesión para desarrolladores y suscribirse a su sitio Web existente, deberá crear un punto de conexión de delegación especial en su sitio. Necesita para que actúe como el punto de entrada para cualquier solicitud de este tipo iniciada desde el portal para desarrolladores de API Management.

El flujo de trabajo final será el siguiente:

1. El desarrollador hace clic en el inicio de sesión o registro de vínculo, en el portal para desarrolladores de API Management
2. El explorador se redirige al extremo de delegación.
3. Extremo de delegación se redirige a cambio o presenta la interfaz de usuario que solicita a usuario para iniciar sesión o registrarse
4. Una vez conseguido, se redirige de nuevo al usuario a la página del portal para desarrolladores de API Management de la que partió.

Para empezar, configuremos primero Administración de API para que dirija las solicitudes a través del extremo de delegación. En el portal para editores de API Management, haga clic en **Seguridad** y, a continuación, haga clic en la pestaña **Delegación**. Haga clic en la casilla para activar "Delegar inicio de sesión & registrarse".

![Delegation page][api-management-delegation-signin-up]

* Determine cuál será la dirección URL del extremo especial de delegación y escríbala en el campo **Dirección URL del extremo de delegación** . 
* En el campo Clave de autenticación de delegación, escriba el secreto que se usará para procesar una firma suministrada para su comprobación con objeto de garantizar que la solicitud procede efectivamente de Azure API Management. Puede hacer clic en el botón **generar** para que API Management genere de forma aleatoria una clave en su lugar.

Ahora debe crear el **extremo de delegación**. Este tiene que realizar varias acciones:

1. Recibir una solicitud de la forma siguiente:
   
   > *http:\//www.yourwebsite.com/apimdelegation?operation=SignIn & returnUrl = {URL de página de origen} & salt = {string} & sig = {string}*
   > 
   > 
   
    Consultar los parámetros para el inicio de sesión / registro caso:
   
   * **operation**: identifica el tipo de solicitud de delegación del que se trata. Solo puede ser **SignIn** en este caso.
   * **returnUrl**: la dirección URL de la página donde el usuario hace clic en un inicio de sesión o vínculo de registro
   * **salt**: una cadena salt especial que se usa para procesar un hash de seguridad
   * **sig**: un hash de seguridad procesado que se comparará con su propio hash procesado
2. Compruebe que la solicitud procede de Azure API Management (opcional, pero especialmente recomendado por motivos de seguridad).
   
   * Procese un hash HMAC-SHA512 de una cadena según los parámetros de consulta **returnUrl** y **salt** ([se proporciona código de ejemplo a continuación]):
     
     > HMAC(**salt** + '\n' + **returnUrl**)
     > 
     > 
   * Compare el hash procesado anteriormente con el valor del parámetro de consulta **sig** . Si los dos hashes coinciden, vaya a paso siguiente; de lo contrario, deniegue la solicitud.
3. Compruebe que recibe una solicitud de inicio de sesión o registrarse: el **operación** parámetro de consulta se establecerá en "**SignIn**".
4. Presentar al usuario con la interfaz de usuario para iniciar sesión o registrarse
5. Si el usuario se suscribe, hay que crear la cuenta correspondiente en Administración de API. [Cree un usuario] con la API de REST de API Management. Al hacerlo, asegúrese de establecer el identificador de usuario en el mismo valor, como se muestra en el almacén de usuario o a un identificador que puede realizar un seguimiento de.
6. Cuando el usuario se autentique correctamente:
   
   * [solicite un token de inicio de sesión único (SSO)] a través de la API de REST de API Management
   * anexe un parámetro de consulta returnUrl a la URL de SSO que se recibió de la llamada de API anterior:
     
     > por ejemplo, https://customer.portal.azure-api.net/signin-sso?token&returnUrl=/return/url 
     > 
     > 
   * redirija al usuario a la URL producida anteriormente

Además de la operación **SignIn**, también puede realizar la administración de cuentas siguiendo los pasos anteriores y utilizando una de las siguientes operaciones:

* **ChangePassword**
* **ChangeProfile**
* **CloseAccount**

Debe pasar los siguientes parámetros de consulta para las operaciones de administración de cuenta.

* **operation**: identifica qué tipo de solicitud de delegación es (ChangePassword, ChangeProfile o CloseAccount)
* **userId**: el identificador de usuario de la cuenta para administrar
* **salt**: una cadena salt especial que se usa para procesar un hash de seguridad
* **sig**: un hash de seguridad procesado que se comparará con su propio hash procesado

## <a name="delegate-product-subscription"></a>Delegación de suscripciones a productos
Delegación de suscripciones a productos funciona de forma similar a la delegación de usuario de inicio de sesión/copia de seguridad. El flujo de trabajo final sería el siguiente:

1. Desarrollador selecciona un producto en el portal para desarrolladores de API Management y hace clic en el botón de suscripción.
2. Explorador se redirige al extremo de delegación.
3. Extremo de delegación realiza los pasos de la suscripción de producto necesario. Depende de los pasos de diseño. Pueden incluir redirigir a otra página para solicitar información de facturación, preguntas adicionales, o simplemente el almacenamiento de la información y no requiere ninguna acción del usuario.

Para habilitar la funcionalidad, en la página **Delegación**, haga clic en **Delegar suscripción de productos**.

A continuación, asegúrese del que punto de conexión de delegación realiza las siguientes acciones:

1. Recibir una solicitud de la forma siguiente:
   
   > *http:\//www.yourwebsite.com/apimdelegation?operation= {operation} & productId = {product para suscribirse a} & userId = {usuario realizar solicitud} & salt = {string} & sig = {string}*
   >
   
    Parámetros de consulta en el caso de suscripción a producto:
   
   * **operation**: identifica el tipo de solicitud de delegación del que se trata. En las solicitudes de suscripción a producto las opciones válidas son:
     * "Subscribe": una solicitud para suscribir al usuario a un producto determinado con el id. especificado (consulte más información a continuación).
     * "Unsubscribe": una solicitud para cancelar la suscripción de un usuario a un producto.
     * "Renew": una solicitud para renovar una suscripción (por ejemplo, que esté a punto de expirar).
   * **productId**: el id. del producto al que el usuario solicitó suscribirse.
   * **subscriptionId**: en *Unsubscribe* y *Renew* -el identificador de la suscripción de producto
   * **userId**: el identificador del usuario se realiza la solicitud para
   * **salt**: una cadena salt especial que se usa para procesar un hash de seguridad
   * **sig**: un hash de seguridad procesado que se comparará con su propio hash procesado

2. Compruebe que la solicitud procede de Azure API Management (opcional, pero especialmente recomendado por motivos de seguridad).
   
   * Calcular un hash HMAC-SHA512 de una cadena según la **productId**, **userId**, y **salt** parámetros de consulta:
     
     > HMAC(**salt** + '\n' + **productId** + '\n' + **userId**)
     > 
     > 
   * Compare el hash procesado anteriormente con el valor del parámetro de consulta **sig** . Si los dos hashes coinciden, vaya a paso siguiente; de lo contrario, deniegue la solicitud.
3. Procesar la suscripción del producto en función del tipo de operación solicitada en **operación** : por ejemplo, facturación, preguntas adicionales, etcetera.
4. Acerca de la suscripción correctamente al usuario que el producto por su parte, suscriba al usuario al producto de API Management mediante [una llamada a la API de REST para las suscripciones].

## <a name="delegate-example-code"></a> Ejemplo de código

Estos muestran ejemplos de código cómo para:

* Tome el *clave de validación de delegación*, que se establece en la pantalla de delegación del portal del publicador
* Crear un HMAC, que, a continuación, se utiliza para validar la firma, probando la validez de la returnUrl pasado.

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

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de la delegación, vea el siguiente vídeo:

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Delegating-User-Authentication-and-Product-Subscription-to-a-3rd-Party-Site/player]
> 
> 

[Delegating developer sign in and sign up]: #delegate-signin-up
[Delegating product subscription]: #delegate-product-subscription
[solicite un token de inicio de sesión único (SSO)]: https://docs.microsoft.com/rest/api/apimanagement/User/GenerateSsoUrl
[Cree un usuario]: https://docs.microsoft.com/rest/api/apimanagement/user/createorupdate
[una llamada a la API de REST para las suscripciones]: https://docs.microsoft.com/rest/api/apimanagement/subscription/createorupdate
[Next steps]: #next-steps
[se proporciona código de ejemplo a continuación]: #delegate-example-code

[api-management-delegation-signin-up]: ./media/api-management-howto-setup-delegation/api-management-delegation-signin-up.png 
