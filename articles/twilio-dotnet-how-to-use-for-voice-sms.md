---
title: Uso de Twilio para voz y SMS (.NET) | Microsoft Docs
description: Aprenda a realizar llamadas telefónicas y a enviar mensajes SMS con el servicio de la API de Twilio en Azure. Los ejemplos de código están escritos en .NET.
services: ''
documentationcenter: .net
author: georgewallace
ms.assetid: 74d4f3c9-f1cb-4968-b744-36b32cd0e834
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 04/24/2015
ms.author: gwallace
ms.custom: devx-track-dotnet
ms.openlocfilehash: 104f969f5e27ef36ad43eb10e19176a4bcfd6648
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021143"
---
# <a name="how-to-use-twilio-for-voice-and-sms-capabilities-from-azure"></a>Uso de Twilio para funciones de voz y SMS desde Azure
Esta guía describe cómo realizar tareas comunes de programación con el servicio de API de Twilio en Azure. Entre los escenarios descritos se incluyen realizar una llamada telefónica y enviar un mensaje de servicio de mensajes cortos (SMS). Para obtener más información sobre Twilio y el uso de voz y mensajes SMS en sus aplicaciones, consulte la sección [Pasos siguientes](#NextSteps) .

## <a name="what-is-twilio"></a><a id="WhatIs"></a>¿Qué es Twilio?
Twilio está potenciando el futuro de las comunicaciones empresariales, al permitir que los desarrolladores inserten voz, VoIP y mensajería en las aplicaciones. Pueden virtualizar toda la infraestructura necesaria en un entorno global basado en la nube y exponerlo a través de la plataforma API de comunicaciones de Twilio. Las aplicaciones son sencillas de compilar y pueden escalarse. Disfrute de la flexibilidad de pagar por lo que utiliza y aproveche la confiabilidad de la nube.

**Twilio Voice** permite que sus aplicaciones realicen y reciban llamadas. **Twilio SMS** permite que sus aplicaciones envíen y reciban mensajes SMS. **Twilio Client** permite realizar llamadas VoIP desde cualquier teléfono, tableta o explorador, y es compatible con WebRTC.

## <a name="twilio-pricing-and-special-offers"></a><a id="Pricing"></a>Precios de Twilio y ofertas especiales
Los clientes de Azure reciben una [oferta especial](https://www.twilio.com/azure): 10 $ de regalo en crédito Twilio al actualizar su cuenta Twilio. Este crédito Twilio se puede aplicar a cualquier uso de Twilio (10 $ de crédito equivalen al envío de aproximadamente 1.000 mensajes SMS o recibir hasta 1.000 minutos de voz entrante, según la ubicación del número de teléfono y el destino del mensaje o de la llamada). Canjee este crédito Twilio y comience en [twilio.com/azure](https://twilio.com/azure).

Twilio es un servicio de pago por uso. No hay comisiones establecidas y puede cerrar su cuenta en cualquier momento. Puede encontrar más detalles en [Precios de Twilio](https://www.twilio.com/voice/pricing).

## <a name="concepts"></a><a id="Concepts"></a>Conceptos
La API de Twilio es una API de RESTful que proporciona funciones de voz y SMS para las aplicaciones. Las bibliotecas de cliente están disponibles en varios lenguajes; para obtener una lista, consulte las [bibliotecas de API de Twilio][twilio_libraries].

Aspectos fundamentales de la API de Twilio son los verbos de Twilio y el lenguaje de marcado de Twilio (TwiML).

### <a name="twilio-verbs"></a><a id="Verbs"></a>Verbos de Twilio
La API usa dos verbos de Twilio; por ejemplo, el verbo **&lt;Say&gt;** indica a Twilio que entregue de manera audible un mensaje en una llamada.

A continuación se presenta una lista de verbos de Twilio.  Obtenga información sobre los demás verbos y las capacidades de Twilio a través de [Documentación de lenguaje de marcado de Twilio](https://www.twilio.com/docs/api/twiml).

* `<Dial>`: conecta la persona que llama con otro teléfono.
* `<Gather>`: recopila los dígitos numéricos que se introdujeron en el teclado del teléfono.
* `<Hangup>`: finaliza una llamada.
* `<Play>`: reproduce un archivo de audio.
* `<Pause>`: espera en silencio una cantidad de segundos especificada.
* `<Record>`: graba la voz de la persona que llama y devuelve una dirección URL de un archivo que contiene la grabación.
* `<Redirect>`: transfiere el control de una llamada o SMS al TwiML en una URL diferente.
* `<Reject>`: rechaza una llamada entrante a su número de Twilio sin cobrarle.
* `<Say>`: convierte texto en voz para hacer una llamada.
* `<Sms>`: envía un mensaje SMS.

### <a name="twiml"></a>TwiML
TwiML es un conjunto de instrucciones basadas en XML y en los verbos de Twilio que informan a Twilio sobre cómo procesar una llamada o un SMS.

Como ejemplo, el siguiente TwiML convierte el texto **Hello World** en voz.

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<Response>
  <Say>Hello World</Say>
</Response>
```

Cuando su aplicación llama a la API de Twilio, uno de los parámetros de API es la URL que devuelve la respuesta de TwiML. Con fines de desarrollo, puede usar las URL que ofrece Twilio para proporcionar las respuestas de TwiML que usaron sus aplicaciones. También puede hospedar sus propias URL para producir las repuestas de TwiML; otra opción es usar el objeto **TwiMLResponse** .

Para más información sobre los verbos de Twilio, sus atributos y TwiML, consulte [TwiML][twiml]. Para más información sobre la API de Twilio, consulte [API de Twilio][twilio_api].

## <a name="create-a-twilio-account"></a><a id="CreateAccount"></a>Creación de una cuenta de Twilio
Cuando esté preparado para obtener una cuenta de Twilio, regístrese para obtener una [evaluación de Twilio][try_twilio]. Puede empezar con una cuenta gratuita y, posteriormente, actualizarla.

Cuando se registre para obtener una cuenta de Twilio, recibirá un identificador de cuenta y un token de autenticación. Necesitará ambos para realizar llamadas a la API de Twilio. Para evitar el acceso no autorizado a su cuenta, mantenga protegido el token de autenticación. Puede ver el identificador de cuenta y el token de autenticación en la [página de la cuenta de Twilio][twilio_account], en los campos etiquetados como **ACCOUNT SID** y **AUTH TOKEN**, respectivamente.

## <a name="create-an-azure-application"></a><a id="create_app"></a>Creación de una aplicación de Azure
Una aplicación de Azure que hospeda una aplicación habilitada para Twilio no es diferente de ninguna otra aplicación de Azure. Agregue la biblioteca .NET de Twilio y configure el rol que para usar las bibliotecas .NET de Twilio.
Para obtener información sobre la creación de un proyecto inicial de Azure, consulte [Creación de un proyecto de Azure en Visual Studio][vs_project].

## <a name="configure-your-application-to-use-twilio-libraries"></a><a id="configure_app"></a>Configuración de su aplicación para utilizar bibliotecas de Twilio
Twilio proporciona un conjunto de bibliotecas de asistentes .NET que encapsulan varios aspectos de Twilio a fin de proporcionar maneras sencillas y fáciles de interactuar con la API de Twilio REST y el Cliente de Twilio para producir respuestas TwiML.

Twilio proporciona cinco bibliotecas para desarrolladores de .NET:

| Biblioteca | Descripción |
| --- | --- |
| Twilio.API | La biblioteca central de Twilio que encapsula la API de Twilio REST en una biblioteca .NET fácil de usar. Esta biblioteca está disponible para .NET, Silverlight y Windows Phone 7. |
| Twilio.TwiML | Proporciona un método .NET fácil de usar para generar una marca TwiML. |
| Twilio.MVC | Para los desarrolladores que usan ASP.NET MVC, esta biblioteca incluye un atributo TwilioController, TwiML ActionResult y de validación de solicitudes. |
| Twilio.WebMatrix | Para los desarrolladores que usan la herramienta de desarrollo WebMatrix gratuita de Microsoft, esta biblioteca contiene asistentes de sintaxis Razor para diversas acciones de Twilio. |
| Twilio.Client.Capability | Contiene el generador de token de capacidad para usarlo con el SDK de JavaScript del Cliente de Twilio. |

> [!Important]
> Todas las bibliotecas requieren .NET 3.5, Silverlight 4 o Windows Phone 7 o posterior.

Las muestras que se proporcionan en esta guía usan la biblioteca Twilio.API.

Las bibliotecas se pueden [instalar mediante la extensión del Administrador de paquetes de NuGet](https://www.twilio.com/docs/csharp/install) disponible para Visual Studio 2010 hasta 2015.  El código fuente se hospeda en [GitHub][twilio_github_repo], que incluye un wiki que contiene documentación completa para usar las bibliotecas.

De manera predeterminada, Microsoft Visual Studio 2010 instala la versión 1.2 de NuGet. La instalación de las bibliotecas de Twilio requiere la versión 1.6 de NuGet o posterior. Para más información sobre la instalación o actualización de NuGet, consulte [https://nuget.org/][nuget].

> [!NOTE]
> Para instalar la última versión de NuGet, primero debe desinstalar la versión cargada usando el Administrador de extensiones de Visual Studio. Para ello, debe ejecutar Visual Studio como administrador. De lo contrario, el botón de desinstalación estará deshabilitado.
>
>

### <a name="to-add-the-twilio-libraries-to-your-visual-studio-project"></a><a id="use_nuget"></a>Para agregar dos bibliotecas de Twilio a su proyecto de Visual Studio:
1. Abra su solución en Visual Studio.
2. Haga clic con el botón secundario en **Referencias**.
3. Haga clic en **Administrar paquetes de NuGet...**
4. Haga clic en **En línea**.
5. En el cuadro de búsqueda en línea, escriba *twilio*.
6. Haga clic en **Instalar** en el paquete de Twilio.

## <a name="how-to-make-an-outgoing-call"></a><a id="howto_make_call"></a>Procedimientos: Realización de una llamada saliente
A continuación se indica cómo realizar una llamada saliente con la clase **CallResource**. Este código también utiliza un sitio que proporciona Twilio para devolver la respuesta de lenguaje de marcado de Twilio (TwiML). Sustituya los valores de los números de teléfono **to** y **from**, y asegúrese de comprobar el número de teléfono **from** de la cuenta de Twilio antes de ejecutar el código.

```csharp
// Use your account SID and authentication token instead
// of the placeholders shown here.
const string accountSID = "your_twilio_account";
const string authToken = "your_twilio_authentication_token";

// Initialize the TwilioClient.
TwilioClient.Init(accountSID, authToken);

// Use the Twilio-provided site for the TwiML response.
var url = "https://twimlets.com/message";
url = $"{url}?Message%5B0%5D=Hello%20World";

// Set the call From, To, and URL values to use for the call.
// This sample uses the sandbox number provided by
// Twilio to make the call.
var call = CallResource.Create(
    to: new PhoneNumber("+NNNNNNNNNN"),
    from: new PhoneNumber("NNNNNNNNNN"),
    url: new Uri(url));
    }
```

Para más información sobre los parámetros pasados al método **CallResource.Create**, consulte [https://www.twilio.com/docs/api/rest/making-calls][twilio_rest_making_calls].

Como se mencionó, este código usa el sitio proporcionado por Twilio para devolver la respuesta de TwiML. Podría en cambio usar su propio sitio para proporcionar la respuesta de TwiML. Para más información, vea: [Cómo: Entrega de respuestas de TwiML desde su propio sitio web](#howto_provide_twiml_responses).

## <a name="how-to-send-an-sms-message"></a><a id="howto_send_sms"></a>Procedimientos: Envío de un mensaje SMS
A continuación se indica cómo enviar un mensaje SMS usando la clase **MessageResource**. El número **from** lo proporciona Twilio en las cuentas de evaluación para enviar mensajes SMS. El número **to** se debe comprobar en la cuenta de Twilio antes de ejecutar el código.

```csharp
// Use your account SID and authentication token instead
// of the placeholders shown here.
const string accountSID = "your_twilio_account";
const string authToken = "your_twilio_authentication_token";

// Initialize the TwilioClient.
TwilioClient.Init(accountSID, authToken);

try
{
    // Send an SMS message.
    var message = MessageResource.Create(
        to: new PhoneNumber("+12069419717"),
        from: new PhoneNumber("+14155992671"),
        body: "This is my SMS message.");
}
catch (TwilioException ex)
{
    // An exception occurred making the REST call
    Console.WriteLine(ex.Message);
}
```

## <a name="how-to-provide-twiml-responses-from-your-own-website"></a><a id="howto_provide_twiml_responses"></a>Procedimientos: Entrega de respuestas de TwiML desde su propio sitio web
Cuando la aplicación inicia una llamada a la API de Twilio, por ejemplo a través del método **CallResource.Create**, Twilio envía la solicitud a una dirección URL que debe devolver una respuesta de TwiML. En el ejemplo de [Instrucciones: Realización de una llamada saliente](#howto_make_call) usa la dirección URL [https://twimlets.com/message][twimlet_message_url] de Twilio para devolver la respuesta.

> [!NOTE]
> Aun cuando TwiML está diseñado para que lo usen los servicios web, puede ver el TwiML en su explorador. Por ejemplo, haga clic en [https://twimlets.com/message][twimlet_message_url] para ver un elemento `<Response>` vacío; en este otro ejemplo, haga clic en [https://twimlets.com/message?Message%5B0%5D=Hello%20World](https://twimlets.com/message?Message%5B0%5D=Hello%20World) para ver un elemento `<Response>` que contiene un elemento &lt; Say&gt;.
>

En lugar de confiar en la URL que Twilio proporciona, puede crear su propio sitio URL que devuelve procesos HTTP. Puede crear el sitio en cualquier lenguaje que devuelva respuestas HTTP. En este tema se asume que va a hospedar la URL desde un controlador de ASP.NET genérico.

El siguiente controlador de ASP.NET elabora una respuesta de TwiML que dice **Hello World** en la llamada.

```csharp
using System.Text;
using System.Web;

namespace WebRole1
{
    /// <summary>
    /// Summary description for Handler1
    /// </summary>
    public class Handler1 : IHttpHandler
    {
        public void ProcessRequest(HttpContext context)
        {
            const string twiMLResponse =
                "<Response><Say>Hello World.</Say></Response>";

            context.Response.Clear();
            context.Response.ContentType = "text/xml";
            context.Response.ContentEncoding = Encoding.UTF8;
            context.Response.Write(twiMLResponse);
            context.Response.End();
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}
```
    
Como puede ver en el ejemplo anterior, la respuesta de TwiML es sencillamente un documento XML. La biblioteca Twilio.TwiML contiene clases que generarán TwiML de manera automática. El siguiente ejemplo produce la respuesta equivalente como se muestra arriba, pero usa la clase **VoiceResponse**.

```csharp
using System.Web;
using Twilio.TwiML;

namespace WebRole1
{
    /// <summary>
    /// Summary description for Handler1
    /// </summary>
    public class Handler1 : IHttpHandler
    {

        public void ProcessRequest(HttpContext context)
        {
            var twiml = new VoiceResponse();
            twiml.Say("Hello World.");

            context.Response.Clear();
            context.Response.ContentType = "text/xml";
            context.Response.Write(twiml.ToString());
            context.Response.End();
        }

        public bool IsReusable
        {
            get
            {
                return false;
            }
        }
    }
}
```

Para más información acerca de TwiML, consulte [https://www.twilio.com/docs/api/twiml](https://www.twilio.com/docs/api/twiml).

Después de que haya configurado una manera de proporcionar respuestas de TwiML, puede pasar esa URL al método **CallResource.Create**. Por ejemplo, si tiene una aplicación web llamada MyTwiML implementada en un servicio en la nube de Azure y el nombre de su controlador de ASP.NET es mytwiml.ashx, la URL puede traspasarse a **CallResource.Create** como se muestra en el siguiente ejemplo de código:

```csharp
// This sample uses the sandbox number provided by Twilio to make the call.
// Place the call.
var call = CallResource.Create(
    to: new PhoneNumber("+NNNNNNNNNN"),
    from: new PhoneNumber("NNNNNNNNNN"),
    url: new Uri("http://<your_hosted_service>.cloudapp.net/MyTwiML/mytwiml.ashx"));
    }
```

Para obtener información adicional sobre el uso de Twilio en Azure con ASP.NET, consulte [Realización de una llamada telefónica con Twilio en un rol web en Azure][howto_phonecall_dotnet].

[!INCLUDE [twilio-additional-services-and-next-steps](../includes/twilio-additional-services-and-next-steps.md)]

[howto_phonecall_dotnet]: partner-twilio-cloud-services-dotnet-phone-call-web-role.md

[twimlet_message_url]: https://twimlets.com/message

[twilio_rest_making_calls]: https://www.twilio.com/docs/api/rest/making-calls

[vs_project]:https://docs.microsoft.com/visualstudio/azure/vs-azure-tools-azure-project-create
[nuget]:https://nuget.org/
[twilio_github_repo]:https://github.com/twilio/twilio-csharp

[twilio_libraries]: https://www.twilio.com/docs/libraries
[twiml]: https://www.twilio.com/docs/api/twiml
[twilio_api]: https://www.twilio.com/api
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_account]:  https://www.twilio.com/console
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified
