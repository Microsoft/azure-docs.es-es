---
title: Uso del servicio de correo electrónico SendGrid (.NET) | Microsoft Docs
description: Obtenga información sobre cómo enviar correo electrónico con el servicio de correo electrónico SendGrid en Azure. Los ejemplos de código están escritos en C# y utilizan la API .NET.
services: ''
documentationcenter: .net
author: georgewallace
ms.author: gwallace
editor: ''
ms.assetid: 21bf4028-9046-476b-9799-3d3082a0f84c
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 02/15/2017
ms.reviewer: dx@sendgrid.com
ms.custom: devx-track-dotnet
ms.openlocfilehash: c8ac20378cbae9334cedb59878311f2541b40bd3
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020599"
---
# <a name="how-to-send-email-using-sendgrid-with-azure"></a>Envío de correos electrónicos con SendGrid y Azure
## <a name="overview"></a>Información general
Esta guía describe cómo realizar tareas comunes de programación con el servicio de correo electrónico SendGrid en Azure. Los ejemplos están escritos en C\# y admiten .NET Standard 1.3. Entre los escenarios descritos se incluyen creación de correos electrónicos, envío de correos electrónicos, incorporación de datos adjuntos y habilitación de varias configuraciones de correo y seguimiento. Para obtener más información sobre SendGrid y el envío de correo electrónico, consulte la sección [Pasos siguientes][Next steps] .

## <a name="what-is-the-sendgrid-email-service"></a>¿Qué es el servicio de correo electrónico SendGrid?
SendGrid es un [servicio de correo electrónico basado en la nube] que ofrece un sistema confiable de [entrega de correo electrónico transaccional], escalabilidad y análisis en tiempo real junto, con API flexibles que facilitan la integración personalizada. A continuación se indican casos de uso comunes de SendGrid:

* Envío automático de recepciones o compra de confirmaciones a clientes.
* Administración de las listas de distribución para el envío mensual de folletos y promociones a clientes.
* Recopilación de métricas en tiempo real para, por ejemplo, direcciones de correo electrónico bloqueadas y captación de clientes.
* Reenvío de las consultas de los clientes.
* Procesamiento de mensajes de correo electrónico entrantes.

Para más información, visite [https://sendgrid.com](https://sendgrid.com) o el repositorio de GitHub de la [biblioteca de C#][sendgrid-csharp] de SendGrid.

## <a name="create-a-sendgrid-account"></a>Creación de una cuenta de SendGrid
[!INCLUDE [sendgrid-sign-up](../includes/sendgrid-sign-up.md)]

## <a name="reference-the-sendgrid-net-class-library"></a>Referencia de la biblioteca de clases .NET de SendGrid
El [paquete NuGet de SendGrid](https://www.nuget.org/packages/Sendgrid) es la forma más fácil de obtener la API de SendGrid y configurar la aplicación con todas las dependencias. NuGet es una extensión de Visual Studio incluida en Microsoft Visual Studio 2015 y superior que facilita la instalación y la actualización de las bibliotecas y las herramientas.

> [!NOTE]
> Para instalar NuGet si ejecuta una versión de Visual Studio anterior a Visual Studio 2015, visite [https://www.nuget.org](https://www.nuget.org) y haga clic en el botón **Instalar NuGet**.
>
>

Realice los pasos siguientes para instalar el paquete NuGet de SendGrid en su aplicación:

1. Haga clic en **Nuevo proyecto** y seleccione una **Plantilla**.

   ![Creación de un nuevo proyecto][create-new-project]
2. En el **Explorador de soluciones**, haga clic con el botón derecho en **Referencias** y, después, en **Administrar paquetes NuGet**.

   ![paquete NuGet de SendGrid][SendGrid-NuGet-package]
3. Busque **SendGrid** y seleccione el elemento **SendGrid** en la lista de resultados.
4. Seleccione la versión estable más reciente del paquete de NuGet en la lista desplegable de versiones para poder trabajar con el objeto de modelo y las API que se muestran en este artículo.

   ![Paquete de SendGrid][sendgrid-package]
5. Haga clic en **Instalar** para completar la instalación y cierre este cuadro de diálogo.

La biblioteca de clases .NET de SendGrid se llama **SendGrid**. Contiene los siguientes espacios de nombres:

* **SendGrid** para comunicarse con la API de SendGrid.
* **SendGrid.Helpers.Mail** para métodos auxiliares para crear fácilmente objetos SendGridMessage que especifican cómo enviar correos electrónicos.

Agregue las siguientes declaraciones de espacio de nombres de código en la parte superior de todo archivo C# en el que desee obtener acceso al servicio de correo electrónico SendGrid mediante programación:

```csharp
using SendGrid;
using SendGrid.Helpers.Mail;
```

## <a name="how-to-create-an-email"></a>Procedimientos: Crear un correo electrónico
Use el objeto **SendGridMessage** para crear un mensaje de correo. Una vez creado el objeto de mensaje, establecer propiedades y métodos, incluidos el remitente, el destinatario, el asunto y el cuerpo del correo electrónico.

El siguiente ejemplo muestra la forma de crear un objeto de correo electrónico completo:

```csharp
var msg = new SendGridMessage();

msg.SetFrom(new EmailAddress("dx@example.com", "SendGrid DX Team"));

var recipients = new List<EmailAddress>
{
    new EmailAddress("jeff@example.com", "Jeff Smith"),
    new EmailAddress("anna@example.com", "Anna Lidman"),
    new EmailAddress("peter@example.com", "Peter Saddow")
};
msg.AddTos(recipients);

msg.SetSubject("Testing the SendGrid C# Library");

msg.AddContent(MimeType.Text, "Hello World plain text!");
msg.AddContent(MimeType.Html, "<p>Hello World!</p>");
```

Para obtener más información sobre las propiedades y los métodos que admite el tipo **SendGrid** , consulte [sendgrid-csharp][sendgrid-csharp] en GitHub.

## <a name="how-to-send-an-email"></a>Procedimientos: Enviar un correo electrónico
Después de crear un mensaje de correo electrónico, puede enviarlo con la API de SendGrid. También puede usar la [biblioteca integrada de .NET][NET-library].

El envío de correos electrónicos requiere que el usuario proporcione su clave de API de SendGrid. Si desea detalles sobre cómo configurar claves de API, visite la [documentación][documentation] de claves de API de SendGrid.

Para almacenar estas credenciales en Azure Portal, haga clic en Configuración de la aplicación y agregue pares clave-valor en la configuración de la aplicación.

 ![Configuración de la aplicación de Azure][azure_app_settings]

A continuación, puede tener acceso a ellas de la siguiente manera:

```csharp
var apiKey = System.Environment.GetEnvironmentVariable("SENDGRID_APIKEY");
var client = new SendGridClient(apiKey);
```

En los ejemplos siguientes se muestra cómo enviar un mensaje de correo electrónico mediante la API web de SendGrid con una aplicación de consola.

```csharp
using System;
using System.Threading.Tasks;
using SendGrid;
using SendGrid.Helpers.Mail;

namespace Example
{
    internal class Example
    {
        private static void Main()
        {
            Execute().Wait();
        }

        static async Task Execute()
        {
            var apiKey = System.Environment.GetEnvironmentVariable("SENDGRID_APIKEY");
            var client = new SendGridClient(apiKey);
            var msg = new SendGridMessage()
            {
                From = new EmailAddress("test@example.com", "DX Team"),
                Subject = "Hello World from the SendGrid CSharp SDK!",
                PlainTextContent = "Hello, Email!",
                HtmlContent = "<strong>Hello, Email!</strong>"
            };
            msg.AddTo(new EmailAddress("test@example.com", "Test User"));
            var response = await client.SendEmailAsync(msg);
        }
    }
}
```

## <a name="how-to-send-email-from-asp-net-core-api-using-mailhelper-class"></a>Procedimientos: Enviar un correo electrónico desde la API de ASP .NET Core mediante la clase MailHelper

El ejemplo siguiente se puede utilizar para enviar un solo correo electrónico a varias personas desde la API de ASP .NET Core mediante la clase `MailHelper` del espacio de nombres `SendGrid.Helpers.Mail`. En este ejemplo, usamos ASP .NET Core 1.0. 

En este ejemplo, la clave de la API se ha almacenado en el archivo `appsettings.json` que se puede invalidar desde Azure Portal, tal como se muestra en los ejemplos anteriores.

El contenido del archivo `appsettings.json` debe tener un aspecto similar al siguiente:

```csharp
{
   "Logging": {
   "IncludeScopes": false,
   "LogLevel": {
   "Default": "Debug",
   "System": "Information",
   "Microsoft": "Information"
     }
   },
 "SENDGRID_API_KEY": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}
```

En primer lugar, tenemos que agregar el siguiente código en el archivo `Startup.cs` del proyecto de la API de .NET Core. Esto es necesario para que podamos acceder a `SENDGRID_API_KEY` desde el archivo `appsettings.json` mediante la inyección de dependencias en el controlador de API. La interfaz `IConfiguration` se puede inyectar en el constructor del controlador después de agregarla en el método `ConfigureServices` siguiente. El contenido del archivo `Startup.cs` tiene un aspecto similar al siguiente después de agregar el código requerido:

```csharp
    public IConfigurationRoot Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        // Add mvc here
        services.AddMvc();
        services.AddSingleton<IConfiguration>(Configuration);
    }
```

En el controlador, después de insertar la interfaz `IConfiguration`, podemos usar el método `CreateSingleEmailToMultipleRecipients` de la clase `MailHelper` para enviar un solo correo electrónico a varios destinatarios. El método acepta un parámetro booleano adicional denominado `showAllRecipients`. Este parámetro se puede utilizar para controlar si los destinatarios del correo electrónico pueden ver las direcciones de correo electrónico de los demás en la sección Para del encabezado del correo electrónico. El código de ejemplo para el controlador debe ser similar a lo siguiente: 

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using SendGrid;
using SendGrid.Helpers.Mail;
using Microsoft.Extensions.Configuration;

namespace SendgridMailApp.Controllers
{
    [Route("api/[controller]")]
    public class NotificationController : Controller
    {
       private readonly IConfiguration _configuration;

       public NotificationController(IConfiguration configuration)
       {
         _configuration = configuration;
       }      
    
       [Route("SendNotification")]
       public async Task PostMessage()
       {
          var apiKey = _configuration.GetSection("SENDGRID_API_KEY").Value;
          var client = new SendGridClient(apiKey);
          var from = new EmailAddress("test1@example.com", "Example User 1");
          List<EmailAddress> tos = new List<EmailAddress>
          {
              new EmailAddress("test2@example.com", "Example User 2"),
              new EmailAddress("test3@example.com", "Example User 3"),
              new EmailAddress("test4@example.com","Example User 4")
          };
        
          var subject = "Hello world email from Sendgrid ";
          var htmlContent = "<strong>Hello world with HTML content</strong>";
          var displayRecipients = false; // set this to true if you want recipients to see each others mail id 
          var msg = MailHelper.CreateSingleEmailToMultipleRecipients(from, tos, subject, "", htmlContent, false);
          var response = await client.SendEmailAsync(msg);
      }
   }
}
```

## <a name="how-to-add-an-attachment"></a>Procedimientos: Agregar datos adjuntos
Es posible agregar datos adjuntos a un mensaje llamando al método **AddAttachment** y especificando mínimamente el nombre de archivo y el contenido codificado en Base64 que desea adjuntar. Puede incluir múltiples datos adjuntos mediante la llamada a este método, que debe utilizar una vez por cada archivo que desee adjuntar, o utilizando el método **AddAttachments**. El siguiente ejemplo demuestra la incorporación de datos adjuntos a un mensaje:

```csharp
var banner2 = new Attachment()
{
    Content = Convert.ToBase64String(raw_content),
    Type = "image/png",
    Filename = "banner2.png",
    Disposition = "inline",
    ContentId = "Banner 2"
};
msg.AddAttachment(banner2);
```

## <a name="how-to-use-mail-settings-to-enable-footers-tracking-and-analytics"></a>Procedimientos: Usar la configuración de correo para habilitar pies de página, seguimiento y análisis
SendGrid proporciona funciones de correo electrónico adicionales mediante el uso de configuraciones de correo y de seguimiento. Todas estas configuraciones se pueden agregar a un mensaje de correo electrónico para habilitar funciones específicas, como el seguimiento por clics, Google Analytics, el seguimiento de suscripciones, etc. Para una lista completa de las aplicaciones, consulte la [documentación de la configuración][settings-documentation].

Es posible incluir aplicaciones en los mensajes de correo de **SendGrid** con métodos implementados como parte de la clase **SendGridMessage**. Los siguientes ejemplos demuestran el uso de los filtros de pie de página y seguimiento por clics:

Los siguientes ejemplos demuestran el uso de los filtros de pie de página y seguimiento por clics:

### <a name="footer-settings"></a>Configuración de pie de página

```csharp
msg.SetFooterSetting(
                     true,
                     "Some Footer HTML",
                     "<strong>Some Footer Text</strong>");
```

### <a name="click-tracking"></a>Seguimiento por clics

```csharp
msg.SetClickTracking(true);
```

## <a name="how-to-use-additional-sendgrid-services"></a>Procedimientos: Usar servicios SendGrid adicionales
SendGrid ofrece varias API y webhooks que puede usar para aprovechar la funcionalidad adicional dentro de la aplicación de Azure. Para más detalles, consulte la [referencia de la API de SendGrid][SendGrid API documentation].

## <a name="next-steps"></a>Pasos siguientes
Ahora que conoce los fundamentos del servicio de correo electrónico SendGrid, siga estos vínculos para obtener más información:

* Repositorio de bibliotecas de C#\# de SendGrid: [sendgrid-csharp][sendgrid-csharp]
* Documentación sobre la API de SendGrid: <https://sendgrid.com/docs>

[Next steps]: #next-steps
[What is the SendGrid Email Service?]: #whatis
[Create a SendGrid Account]: #createaccount
[Reference the SendGrid .NET Class Library]: #reference
[How to: Create an Email]: #createemail
[How to: Send an Email]: #sendemail
[How to: Add an Attachment]: #addattachment
[How to: Use Filters to Enable Footers, Tracking, and Analytics]: #usefilters
[How to: Use Additional SendGrid Services]: #useservices

[create-new-project]: ./media/sendgrid-dotnet-how-to-send-email/new-project.png
[SendGrid-NuGet-package]: ./media/sendgrid-dotnet-how-to-send-email/reference.png
[sendgrid-package]: ./media/sendgrid-dotnet-how-to-send-email/sendgrid-package.png
[azure_app_settings]: ./media/sendgrid-dotnet-how-to-send-email/azure-app-settings.png
[sendgrid-csharp]: https://github.com/sendgrid/sendgrid-csharp
[SMTP vs. Web API]: https://sendgrid.com/docs/Integrate/index.html
[App Settings]: https://sendgrid.com/docs/API_Reference/SMTP_API/apps.html
[SendGrid API documentation]: https://sendgrid.com/docs/API_Reference/api_v3.html
[NET-library]: https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html#-Using-NETs-Builtin-SMTP-Library
[documentation]: https://sendgrid.com/docs/Classroom/Send/api_keys.html
[settings-documentation]: https://sendgrid.com/docs/API_Reference/SMTP_API/apps.html

[servicio de correo electrónico basado en la nube]: https://sendgrid.com/solutions
[entrega de correo electrónico transaccional]: https://sendgrid.com/use-cases/transactional-email

