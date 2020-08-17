---
title: Instrucciones para desarrolladores para el acceso condicional de Azure Active Directory
titleSuffix: Microsoft identity platform
description: Instrucciones para desarrolladores y escenarios para el acceso condicional de Azure AD y la plataforma de identidad de Microsoft.
services: active-directory
keywords: ''
author: rwike77
manager: CelesteDG
ms.author: ryanwi
ms.reviewer: jmprieur, saeeda
ms.date: 05/18/2020
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev
ms.topic: conceptual
ms.workload: identity
ms.openlocfilehash: b1bfefb3b72c151e7a61068b3c0ad9f3e2bc4a6f
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88120633"
---
# <a name="developer-guidance-for-azure-active-directory-conditional-access"></a>Instrucciones para desarrolladores para el acceso condicional de Azure Active Directory

La característica de acceso condicional en Azure Active Directory (Azure AD) ofrece una de las varias maneras que hay para proteger la aplicación y un servicio. El acceso condicional permite que los desarrolladores y clientes empresariales protejan los servicios de diversas formas, entre las que se incluyen las siguientes:

* [Autenticación multifactor](../authentication/concept-mfa-howitworks.md)
* Autorización para que solo los dispositivos inscritos en Intune accedan a servicios específicos
* Restricción de ubicaciones de usuario e intervalos IP

Para obtener más información sobre las funcionalidades completas del acceso condicional, consulte el artículo [¿Qué es el acceso condicional?](../conditional-access/overview.md)

Para desarrolladores que compilan aplicaciones para Azure AD, este artículo muestra cómo se puede usar el acceso condicional y también proporciona información sobre el impacto de acceder a los recursos sobre los que no se tiene control, y que pueden tener directivas de acceso condicional aplicadas. Este artículo explora además las implicaciones del acceso condicional en el flujo en el nombre de otra persona, aplicaciones web, el acceso a Microsoft Graph y las llamadas a las API.

En él se supone que tiene conocimientos sobre aplicaciones de inquilino [único](quickstart-register-app.md) y [multiinquilino](howto-convert-app-to-be-multi-tenant.md), además de sobre los [patrones comunes de autenticación](./authentication-vs-authorization.md).

> [!NOTE]
> Necesita una licencia de Azure AD Premium P1 para usar esta característica. Para obtener la licencia correcta para sus requisitos, consulte [Comparación de las características con disponibilidad general de las ediciones Gratis, Básico y Premium](https://azure.microsoft.com/pricing/details/active-directory/).
> Los clientes con [licencias de Microsoft 365 Empresa](/office365/servicedescriptions/microsoft-365-service-descriptions/microsoft-365-business-service-description) también tienen acceso a características de acceso condicional.

## <a name="how-does-conditional-access-impact-an-app"></a>¿Cómo el acceso condicional afecta a una aplicación?

### <a name="app-types-impacted"></a>Tipos de aplicación afectados

En los casos más comunes, el acceso condicional no cambia el comportamiento de una aplicación ni requiere ningún cambio por parte del desarrollador. Solo en ciertos casos en los que una aplicación, de manera indirecta o silenciosa, solicita un token para un servicio, una aplicación requiere cambios en el código para controlar los "desafíos" del acceso condicional. Puede ser tan sencillo como realizar una solicitud de inicio de sesión interactiva.

En concreto, los escenarios siguientes requieren código para controlar los "desafíos" del acceso condicional:

* Aplicaciones que realizan el flujo "en nombre de"
* Aplicaciones que acceden a varios servicios o recursos
* Aplicaciones de una sola página que usan MSAL.js
* Aplicaciones web que llaman a un recurso

Las directivas de acceso condicional se pueden aplicar a la aplicación, pero también se pueden aplicar a una API web a la que accede la aplicación. Para averiguar cómo configurar una directiva de acceso condicional, consulte [Inicio rápido: Exigir MFA para aplicaciones específicas con acceso condicional de Azure Active Directory](../authentication/tutorial-enable-azure-mfa.md).

Según el escenario, un cliente empresarial puede aplicar y quitar directivas de acceso condicional en cualquier momento. Con el fin de que la aplicación siga funcionando cuando se aplica una directiva nueva, es necesario implementar el control de "desafíos". En los ejemplos siguientes se ilustra el control de desafíos.

### <a name="conditional-access-examples"></a>Ejemplos de acceso condicional

En algunos escenarios se requieren cambios en el código para controlar el acceso condicional, mientras que en otros se trabaja tal cual. Estos son algunos escenarios que usan el acceso condicional para realizar la autenticación multifactor que proporciona información sobre la diferencia.

* Se crea una aplicación iOS de inquilino único y se aplica una directiva de acceso condicional. La aplicación inicia la sesión de un usuario y no solicita acceso a una API. Cuando el usuario inicia sesión, la directiva se invoca automáticamente y el usuario debe realizar la autenticación multifactor (MFA).
* Está creando una aplicación nativa que utiliza un servicio de nivel intermedio para tener acceso a una API de nivel inferior. Un cliente empresarial de la empresa que usa esta aplicación aplica una directiva a la API de nivel inferior. Cuando un usuario final inicia sesión, la aplicación nativa solicita acceso al nivel intermedio y envía el token. El nivel intermedio realiza el flujo "en nombre de" para solicitar acceso a la API de nivel inferior. En este punto, se presenta un "desafío" de notificaciones al nivel intermedio. El nivel intermedio envía el desafío de vuelta a la aplicación nativa, la que necesita cumplir con la directiva de acceso condicional.

#### <a name="microsoft-graph"></a>Microsoft Graph

Microsoft Graph tiene consideraciones especiales a la hora de crear aplicaciones en entornos de acceso condicional. Generalmente, la mecánica del acceso condicional se comporta de la misma manera, pero las directivas que los usuarios ven se basarán en los datos subyacentes que la aplicación está solicitando del gráfico.

Específicamente, todos los ámbitos de Microsoft Graph representan un conjunto de datos que puede tener directivas aplicadas individualmente. Dado que a las directivas de acceso condicional se les asignan conjuntos de datos específicos, Azure AD aplicará las directivas de acceso Condicional basadas en los datos detrás de Graph, en lugar de Graph mismo.

Por ejemplo, si una aplicación solicita los siguientes ámbitos de Microsoft Graph,

```
scopes="Bookings.Read.All Mail.Read"
```

Una aplicación puede esperar que sus usuarios cumplan con todas las directivas establecidas en Bookings y Exchange. Algunos ámbitos pueden asignarse a varios conjuntos de datos si se concede acceso.

### <a name="complying-with-a-conditional-access-policy"></a>Cumplimiento con una directiva de acceso condicional

En el caso de varias topologías de aplicaciones distintas, se evalúa una directiva de acceso condicional cuando se establece la sesión. Si bien una directiva de acceso condicional opera en la granularidad de aplicaciones y servicios, el punto en que se invoca depende en gran medida del escenario que intenta lograr.

Cuando la aplicación intenta acceder a un servicio con una directiva de acceso condicional, podría encontrar un desafío de acceso condicional. Este desafío se codifica en el parámetro `claims`, que se incluye en una respuesta de Azure AD. Este es un ejemplo de este parámetro de desafío:

```
claims={"access_token":{"polids":{"essential":true,"Values":["<GUID>"]}}}
```

Los desarrolladores pueden tomar este desafío y anexarlo a una solicitud nueva a Azure AD. Cuando se pasa este estado, se solicita al usuario final que realice cualquier acción necesaria para cumplir con la directiva de acceso condicional. En los escenarios siguientes, se explican los detalles específicos del error y cómo extraer el parámetro.

## <a name="scenarios"></a>Escenarios

### <a name="prerequisites"></a>Prerrequisitos

El acceso condicional de Azure AD es una característica que se incluye en [Azure AD Premium](../fundamentals/active-directory-whatis.md). Los clientes con [licencias de Microsoft 365 Empresa](/office365/servicedescriptions/microsoft-365-service-descriptions/microsoft-365-business-service-description) también tienen acceso a características de acceso condicional.

### <a name="considerations-for-specific-scenarios"></a>Consideraciones para escenarios específicos

La información siguiente solo se aplica en estos escenarios de acceso condicional:

* Aplicaciones que realizan el flujo "en nombre de"
* Aplicaciones que acceden a varios servicios o recursos
* Aplicaciones de una sola página que usan MSAL.js

Las siguientes secciones describen escenarios comunes que son más complejos. El principio operativo central es que las directivas de acceso condicional se evalúan en el momento en que se solicita el token para el servicio que tiene aplicada una directiva de acceso condicional.

## <a name="scenario-app-performing-the-on-behalf-of-flow"></a>Escenario: Aplicación que realiza el flujo "en nombre de"

En este escenario, analizaremos un caso en el que una aplicación nativa llama a una API o servicio web. A su vez, este servicio realiza el flujo "en nombre de" para llamar a un servicio de bajada. En este caso, se aplica la directiva de acceso condicional al servicio de bajada (API web 2) y se usa una aplicación nativa en lugar de una aplicación demonio/servidor.

![Aplicación que realiza el flujo "en nombre de"](./media/v2-conditional-access-dev-guide/app-performing-on-behalf-of-scenario.png)

La solicitud de token inicial para Web API 1 no solicita al usuario final que realice la autenticación multifactor, porque Web API 1 no siempre puede alcanzar la API de bajada. Una vez que API web 1 intenta solicitar un token en nombre del usuario para API web 2, la solicitud genera un error porque el usuario no inició sesión con la autenticación multifactor.

Azure AD devuelve una respuesta HTTP con algunos datos interesantes:

> [!NOTE]
> En esta instancia se trata de la descripción de un error de autenticación multifactor, sino que es un intervalo amplio de `interaction_required` posiblemente perteneciente al acceso condicional.

```
HTTP 400; Bad Request
error=interaction_required
error_description=AADSTS50076: Due to a configuration change made by your administrator, or because you moved to a new location, you must use multi-factor authentication to access '<Web API 2 App/Client ID>'.
claims={"access_token":{"polids":{"essential":true,"Values":["<GUID>"]}}}
```

En Web API 1, se captura el error `error=interaction_required` y el desafío `claims` se envía de vuelta a la aplicación de escritorio. En ese momento, la aplicación de escritorio puede realizar una llamada `acquireToken()` nueva y anexa el desafío `claims` como un parámetro de cadena de solicitud adicional. Esta solicitud nueva requiere que el usuario realice la autenticación multifactor y, luego, envíe el token nuevo de vuelta a la API web 1 y complete el flujo "en nombre de".

Para probar el escenario, consulte el [ejemplo de código .NET](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/tree/master/2.%20Web%20API%20now%20calls%20Microsoft%20Graph#handling-required-interactions-with-the-user-dynamic-consent-mfa-etc-). Muestra cómo pasar el desafío de notificaciones de vuelta desde API web 1 a la aplicación nativa y construye una solicitud nueva dentro de la aplicación cliente.

## <a name="scenario-app-accessing-multiple-services"></a>Escenario: Aplicación que accede a varios servicios

En este escenario se describe un caso en el que una aplicación web accede a dos servicios, uno de los cuales tiene asignada una directiva de acceso condicional. En función de la lógica de la aplicación, es posible que exista una ruta de acceso en la que la aplicación no requiera acceso a ambos servicios web. En este escenario, el orden en que se solicita un token representa un papel importante en la experiencia del usuario final.

Vamos a suponer que tenemos un servicio web A y B y que el servicio web B tiene aplicada la directiva de acceso condicional. Si bien la solicitud de autenticación interactiva inicial requiere consentimiento para ambos servicios, la directiva de acceso condicional no se requiere en todos los casos. Si la aplicación solicita un token para el servicio web B, se invoca la directiva y las solicitudes subsiguientes del servicio web A también se realizan correctamente, como se indica a continuación.

![Diagrama de flujo de la aplicación que accede a varios servicios](./media/v2-conditional-access-dev-guide/app-accessing-multiple-services-scenario.png)

De manera alternativa, si la aplicación inicialmente solicita un token para el servicio web A, el usuario final no invoca la directiva de acceso condicional. Esto permite que el desarrollador de la aplicación controle la experiencia del usuario final y no obligue a que la directiva de acceso condicional se invoque en todos los casos. La parte complicada aparece si la aplicación solicita posteriormente un token para el servicio web B. En este punto, el usuario final tiene que cumplir con la directiva de acceso condicional. Si la aplicación intenta `acquireToken`, puede generar el error siguiente (que se ilustra en el diagrama a continuación):

```
HTTP 400; Bad Request
error=interaction_required
error_description=AADSTS50076: Due to a configuration change made by your administrator, or because you moved to a new location, you must use multi-factor authentication to access '<Web API App/Client ID>'.
claims={"access_token":{"polids":{"essential":true,"Values":["<GUID>"]}}}
```

![Aplicación que accede a varios servicios que solicitan un token nuevo](./media/v2-conditional-access-dev-guide/app-accessing-multiple-services-new-token.png)

Si la aplicación usa la biblioteca MSAL, la adquisición de un token siempre se reintenta de forma interactiva si se genera un error. Cuando se produce esta solicitud interactiva, el usuario final tiene la oportunidad de cumplir con el acceso condicional. Esto se aplica a menos que la solicitud sea `AcquireTokenSilentAsync` o `PromptBehavior.Never`, en cuyo caso la aplicación debe realizar una solicitud ```AcquireToken``` interactiva para dar al usuario final la oportunidad de cumplir con la directiva.

## <a name="scenario-single-page-app-spa-using-msaljs"></a>Escenario: Aplicación de una sola página (SPA) que usa MSAL.js

En este escenario, se describe un caso en el que se tiene una aplicación de una sola página (SPA) que usa MSAL.js para llamar a una API web protegida por acceso condicional. Se trata de una arquitectura simple, pero tiene algunos matices que se deben considerar cuando se desarrollen aplicaciones alrededor del acceso condicional.

En MSAL.js, existen algunas funciones que obtienen tokens: `loginPopup()`, `acquireTokenSilent(...)`, `acquireTokenPopup(…)` y `acquireTokenRedirect(…)`.

* `loginPopup()` obtiene un token de identificador mediante una solicitud de inicio de sesión interactiva, pero no obtiene tokens de acceso para ningún servicio (incluida una API web protegida por acceso condicional).
* `acquireTokenSilent(…)` se puede usar para obtener silenciosamente un token de acceso, lo que significa que no muestra UI en ninguna circunstancia.
* Tanto `acquireTokenPopup(…)` como `acquireTokenRedirect(…)` se usan para solicitar de manera interactiva un token para un recurso, lo que significa que siempre muestra la UI de inicio de sesión.

Cuando una aplicación necesita un token de acceso para llamar a una API web, intenta un `acquireTokenSilent(…)`. Si la sesión de token expira o es necesario cumplir con una directiva de acceso condicional, la función *acquireToken* genera un error y la aplicación usa `acquireTokenPopup()` o `acquireTokenRedirect()`.

![Diagrama de flujo de la aplicación de una sola página que usa MSAL](./media/v2-conditional-access-dev-guide/spa-using-msal-scenario.png)

Veamos un ejemplo con el escenario de acceso condicional. El usuario final acaba de llegar al sitio y no tiene una sesión. Se realiza una llamada `loginPopup()` y se obtiene un token de identificador sin autenticación multifactor. Luego, el usuario presiona un botón que requiere que la aplicación solicite datos de una API web. La aplicación intenta realizar una llamada `acquireTokenSilent()` pero se produce un error, dado que el usuario todavía no realizó la autenticación multifactor y debe cumplir con la directiva de acceso condicional.

Azure AD envía de vuelta la respuesta HTTP siguiente:

```
HTTP 400; Bad Request
error=interaction_required
error_description=AADSTS50076: Due to a configuration change made by your administrator, or because you moved to a new location, you must use multi-factor authentication to access '<Web API App/Client ID>'.
```

La aplicación debe capturar `error=interaction_required`. Luego, la aplicación puede usar `acquireTokenPopup()` o `acquireTokenRedirect()` en el mismo recurso. Se obliga al usuario a realizar la autenticación multifactor. Una vez que el usuario completa la autenticación multifactor, la aplicación recibe un token de acceso nuevo para el recurso solicitado.

Para probar el escenario, consulte el [ejemplo de código "en nombre de" de SPA de JS](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/blob/a2b257381b410c765ee01ecb611aa6f98c099eb1/2.%20Web%20API%20now%20calls%20Microsoft%20Graph/README.md). Este ejemplo de código usa la directiva de acceso condicional y la API web que registró anteriormente con un SPA de JS para mostrar el escenario. Muestra cómo controlar de forma adecuada el desafío de notificaciones y obtener un token de acceso que se puede usar para la API web. De manera alternativa, revise el [ejemplo de código Angular.js](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2) general para información sobre un SPA de Angular

## <a name="see-also"></a>Consulte también

* Para más información sobre las funcionalidades, consulte [Acceso condicional en Azure Active Directory](../conditional-access/overview.md).
* Para obtener más información sobre los ejemplos de código de Azure AD, consulte los [ejemplos](sample-v2-code.md).
* Para obtener más información sobre el SDK de MSAL y acceso a la documentación de referencia, consulte la [información general de la Biblioteca de autenticación de Microsoft](msal-overview.md).
* Para más información sobre los escenarios multiinquilino, consulte el artículo sobre el [inicio de sesión de usuarios con el patrón multiinquilino](howto-convert-app-to-be-multi-tenant.md).
* Más información sobre [acceso condicional y protección del acceso a las aplicaciones de IoT](/azure/architecture/example-scenario/iot-aad/iot-aad).