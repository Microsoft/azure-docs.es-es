---
title: Realización de una llamada telefónica desde Twilio (.NET) | Microsoft Docs
description: Aprenda a realizar llamadas telefónicas y a enviar mensajes SMS con el servicio de la API de Twilio en Azure. Los ejemplos de código están escritos en .NET.
services: ''
documentationcenter: .net
author: mimckitt
editor: ''
ms.assetid: 789185ad-69dc-4e9e-a936-42e0a25315c8
ms.service: cloud-services
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 05/04/2016
ms.author: mimckitt
ms.custom: devx-track-dotnet
ms.openlocfilehash: 90e7bafab3fb9658cd9e11d0417f84269adcfa13
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022350"
---
# <a name="how-to-make-a-phone-call-using-twilio-in-a-web-role-on-azure"></a>Realización de una llamada telefónica con Twilio en un rol web en Azure
En esta guía se describe cómo usar Twilio para realizar una llamada desde una página web hospedada en Azure. La aplicación resultante pide al usuario que llame a un determinado número y deje un mensaje, como se muestra en la siguiente captura de pantalla.

![Formulario de llamada de Azure con Twilio y ASP.NET][twilio_dotnet_basic_form]

## <a name="prerequisites"></a><a name="twilio-prereqs"></a>Requisitos previos
Tendrá que hacer lo siguiente para usar el código de este tema:

1. Adquiera una cuenta de Twilio y un token de autenticación desde la [consola de Twilio][twilio_console]. Para empezar a trabajar con Twilio, regístrese en [https://www.twilio.com/try-twilio][try_twilio]. Puede evaluar los precios en [https://www.twilio.com/pricing][twilio_pricing]. Para obtener más información sobre las API proporcionadas por Twilio, vea [https://www.twilio.com/voice/api][twilio_api].
2. Agregue la *biblioteca .NET de Twilio*  al rol web. Vea **Para agregar las bibliotecas de Twilio al proyecto de rol web**, más adelante en este tema.

Debe estar familiarizado con la creación de un [rol web básico en Azure][azure_webroles_get_started].

## <a name="how-to-create-a-web-form-for-making-a-call"></a><a name="howtocreateform"></a>Creación de un formulario web para hacer una llamada
<a id="use_nuget"></a>Para agregar las bibliotecas de Twilio al proyecto de rol web:

1. Abra su solución en Visual Studio.
2. Haga clic con el botón secundario en **Referencias**.
3. Haga clic en **Administración de paquetes de NuGet**.
4. Haga clic en **En línea**.
5. En el cuadro de búsqueda en línea, escriba *twilio*.
6. Haga clic en **Instalar** en el paquete de Twilio.

El siguiente código muestra cómo crear un formulario web para recuperar datos de usuario para hacer una llamada. En este ejemplo, se crea un rol web de ASP.NET denominado **TwilioCloud**.

```aspx
<%@ Page Title="Home Page" Language="C#" MasterPageFile="~/Site.master"
    AutoEventWireup="true" CodeBehind="Default.aspx.cs"
    Inherits="WebRole1._Default" %>

<asp:Content ID="HeaderContent" runat="server" ContentPlaceHolderID="HeadContent">
</asp:Content>
<asp:Content ID="BodyContent" runat="server" ContentPlaceHolderID="MainContent">
    <div>
        <asp:BulletedList ID="varDisplay" runat="server" BulletStyle="NotSet">
        </asp:BulletedList>
    </div>
    <div>
        <p>Fill in all fields and click <b>Make this call</b>.</p>
        <div>
            To:<br /><asp:TextBox ID="toNumber" runat="server" /><br /><br />
            Message:<br /><asp:TextBox ID="message" runat="server" /><br /><br />
            <asp:Button ID="callpage" runat="server" Text="Make this call"
                onclick="callpage_Click" />
        </div>
    </div>
</asp:Content>
```

## <a name="how-to-create-the-code-to-make-the-call"></a><a id="howtocreatecode"></a>Creación del código para realizar la llamada
El siguiente código, al que se llama cuando el usuario completa el formulario, crea el mensaje de llamada y genera la llamada. En este ejemplo, el código se ejecuta en el controlador de eventos onclick del botón en el formulario. (Use su cuenta Twilio y su token de autenticación en lugar de los valores de marcador de posición asignados a `accountSID` y `authToken` en el siguiente código).

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using Twilio;
using Twilio.Http;
using Twilio.Types;
using Twilio.Rest.Api.V2010;

namespace WebRole1
{
    public partial class _Default : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {

        }

        protected void callpage_Click(object sender, EventArgs e)
        {
            // Call processing happens here.

            // Use your account SID and authentication token instead of
            // the placeholders shown here.
            var accountSID = "ACNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN";
            var authToken =  "NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN";

            // Instantiate an instance of the Twilio client.
            TwilioClient.Init(accountSID, authToken);

            // Retrieve the account, used later to retrieve the
            var account = AccountResource.Fetch(accountSID);

            this.varDisplay.Items.Clear();

            if (this.toNumber.Text == "" || this.message.Text == "")
            {
                this.varDisplay.Items.Add(
                        "You must enter a phone number and a message.");
            }
            else
            {
                // Retrieve the values entered by the user.
                var to = PhoneNumber(this.toNumber.Text);
                var from = new PhoneNumber("+14155992671");
                var myMessage = this.message.Text;

                // Create a URL using the Twilio message and the user-entered
                // text. You must replace spaces in the user's text with '%20'
                // to make the text suitable for a URL.
                var url = $"https://twimlets.com/message?Message%5B0%5D={myMessage.Replace(" ", "%20")}";
                var twimlUri = new Uri(url);

                // Display the endpoint, API version, and the URL for the message.
                this.varDisplay.Items.Add($"Using Twilio endpoint {
                }");
                this.varDisplay.Items.Add($"Twilioclient API Version is {apiVersion}");
                this.varDisplay.Items.Add($"The URL is {url}");

                // Place the call.
                var call = CallResource.create(to, from, url: twimlUri);
                this.varDisplay.Items.Add("Call status: " + call.Status);
            }
        }
    }
}
```

Se realiza la llamada y se muestran el extremo de Twilio, la versión de la API y el estado de la llamada. En la captura de pantalla siguiente se muestra el resultado de una ejecución de muestra.

![Respuesta de llamada de Azure con Twilio y ASP.NET][twilio_dotnet_basic_form_output]

Puede encontrar más información sobre TwiML en [https://www.twilio.com/docs/api/twiml][twiml]. Puede encontrar más información sobre &lt;Say&gt; y otros verbos de Twilio en [https://www.twilio.com/docs/api/twiml/say][twilio_say].

## <a name="next-steps"></a><a id="nextsteps"></a>Pasos siguientes
Este código se proporciona para mostrar la funcionalidad básica del uso de Twilio en un rol web de ASP.NET en Azure. Antes de implementarlo en Azure en producción, es posible que desee agregar más controles de errores u otras características. Por ejemplo:

* En lugar de usar un formulario web, puede usar Azure Blob Storage o una base de datos de Azure SQL para almacenar los números de teléfono y el texto de llamada. Para obtener más información sobre cómo usar blobs en Azure, vea [Cómo usar el servicio Azure Blob Storage en .NET][howto_blob_storage_dotnet]. Para obtener información sobre cómo usar SQL Database, vea [Cómo usar Azure SQL Database en aplicaciones de .NET][howto_sql_azure_dotnet].
* Puede usar `RoleEnvironment.getConfigurationSettings` para recuperar el identificador de la cuenta de Twilio y el token de autenticación desde los ajustes de configuración de su implementación, en vez de codificar de forma rígida los valores en el formulario. Para obtener información sobre la clase `RoleEnvironment`, vea [Microsoft.WindowsAzure.ServiceRuntime (espacio de nombres)][azure_runtime_ref_dotnet].
* Lea las directrices de seguridad de Twilio en [https://www.twilio.com/docs/security][twilio_docs_security].
* Obtenga más información sobre Twilio en [https://www.twilio.com/docs][twilio_docs].

## <a name="see-also"></a><a name="seealso"></a>Otras referencias
* [Uso de Twilio para funciones de voz y SMS desde Azure](twilio-dotnet-how-to-use-for-voice-sms.md)

[twilio_console]: https://www.twilio.com/console
[twilio_pricing]: https://www.twilio.com/pricing
[try_twilio]: https://www.twilio.com/try-twilio
[twilio_api]: https://www.twilio.com/voice/api
[verify_phone]: https://www.twilio.com/console/phone-numbers/verified

[twilio_dotnet_basic_form]: ./media/partner-twilio-cloud-services-dotnet-phone-call-web-role/WA_twilio_dotnet_basic_form.png
[twilio_dotnet_basic_form_output]: ./media/partner-twilio-cloud-services-dotnet-phone-call-web-role/WA_twilio_dotnet_basic_form_output.png

[twiml]: https://www.twilio.com/docs/api/twiml



[howto_twilio_voice_sms_dotnet]: /develop/net/how-to-guides/twilio/

[howto_blob_storage_dotnet]: https://www.windowsazure.com/develop/net/how-to-guides/blob-storage/

[howto_sql_azure_dotnet]: https://www.windowsazure.com/develop/net/how-to-guides/sql-database/


[twilio_docs_security]: https://www.twilio.com/docs/security
[twilio_docs]: https://www.twilio.com/docs
[twilio_say]: https://www.twilio.com/docs/api/twiml/say


[azure_runtime_ref_dotnet]: https://msdn.microsoft.com/library/windowsazure/microsoft.windowsazure.serviceruntime.aspx
[azure_webroles_get_started]: https://docs.microsoft.com/azure/cloud-services/cloud-services-dotnet-get-started
