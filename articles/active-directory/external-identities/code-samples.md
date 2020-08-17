---
title: 'Código de colaboración B2B y ejemplos de PowerShell: Azure AD'
description: Ejemplos de código y PowerShell para la colaboración B2B de Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: sample
ms.date: 04/11/2017
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: elisolMS
ms.custom: it-pro, seo-update-azuread-jan, has-adal-ref
ms.collection: M365-identity-device-management
ms.openlocfilehash: fcf6e7a4fb5e76dddba6162bbabfdc5abc806a20
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907048"
---
# <a name="azure-active-directory-b2b-collaboration-code-and-powershell-samples"></a>Ejemplos de código y PowerShell para la colaboración B2B de Azure Active Directory

## <a name="powershell-example"></a>Ejemplo de PowerShell
Puede invitar de forma masiva a usuarios externos a una organización desde direcciones de correo electrónico que ha almacenado en un archivo CSV.

1. Prepare el archivo: cree un archivo CSV y asígnele el nombre invitations.csv. En este ejemplo, el archivo se guarda en C:\data y contiene la información siguiente:

   Nombre                  |  InvitedUserEmailAddress
   --------------------- | --------------------------
   Invitado de B2B de Gmail     | b2binvitee@gmail.com
   Invitado de B2B de Outlook   | b2binvitee@outlook.com


2. Obtenga la última versión de Azure AD PowerShell para usar los nuevos cmdlets: debe instalar el módulo de Azure AD PowerShell actualizado, que puede descargar de la [página de la versión del módulo de Powershell](https://www.powershellgallery.com/packages/AzureADPreview).

3. Inicie sesión en el espacio.

    ```powershell
    $cred = Get-Credential
    Connect-AzureAD -Credential $cred
    ```

4. Ejecución del cmdlet de PowerShell

   ```powershell
   $invitations = import-csv C:\data\invitations.csv
   $messageInfo = New-Object Microsoft.Open.MSGraph.Model.InvitedUserMessageInfo
   $messageInfo.customizedMessageBody = "Hey there! Check this out. I created an invitation through PowerShell"
   foreach ($email in $invitations) {New-AzureADMSInvitation -InvitedUserEmailAddress $email.InvitedUserEmailAddress -InvitedUserDisplayName $email.Name -InviteRedirectUrl https://wingtiptoysonline-dev-ed.my.salesforce.com -InvitedUserMessageInfo $messageInfo -SendInvitationMessage $true}
   ```

Este cmdlet enviará una invitación a la dirección de correo electrónico de invitations.csv. Algunas características adicionales de este cmdlet son las siguientes:
- Texto personalizado en el mensaje de correo electrónico
- Inclusión de un nombre para mostrar de los usuarios invitados
- Envío de mensajes en copias o supresión de mensajes de correo electrónico por completo

## <a name="code-sample"></a>Código de ejemplo
En este ejemplo se muestra cómo llamar a la API de invitación, en el modo de solo de aplicación, para obtener la URL de pago del recurso al que va a invitar al usuario B2B. El objetivo consiste en enviar un correo electrónico de invitación personalizado. El correo electrónico se puede redactar con un cliente HTTP, lo que le permite personalizar su aspecto y enviarlo a través de Microsoft Graph API.

```csharp
namespace SampleInviteApp
{
    using System;
    using System.Linq;
    using System.Net.Http;
    using System.Net.Http.Headers;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json;
    class Program
    {
        /// <summary>
        /// Microsoft Graph resource.
        /// </summary>
        static readonly string GraphResource = "https://graph.microsoft.com";

        /// <summary>
        /// Microsoft Graph invite endpoint.
        /// </summary>
        static readonly string InviteEndPoint = "https://graph.microsoft.com/v1.0/invitations";

        /// <summary>
        ///  Authentication endpoint to get token.
        /// </summary>
        static readonly string EstsLoginEndpoint = "https://login.microsoftonline.com";

        /// <summary>
        /// This is the tenantid of the tenant you want to invite users to.
        /// </summary>
        private static readonly string TenantID = "";

        /// <summary>
        /// This is the application id of the application that is registered in the above tenant.
        /// The required scopes are available in the below link.
        /// https://developer.microsoft.com/graph/docs/api-reference/v1.0/api/invitation_post
        /// </summary>
        private static readonly string TestAppClientId = "";

        /// <summary>
        /// Client secret of the application.
        /// </summary>
        private static readonly string TestAppClientSecret = @"";

        /// <summary>
        /// This is the email address of the user you want to invite.
        /// </summary>
        private static readonly string InvitedUserEmailAddress = @"";

        /// <summary>
        /// This is the display name of the user you want to invite.
        /// </summary>
        private static readonly string InvitedUserDisplayName = @"";

        /// <summary>
        /// Main method.
        /// </summary>
        /// <param name="args">Optional arguments</param>
        static void Main(string[] args)
        {
            Invitation invitation = CreateInvitation();
            SendInvitation(invitation);
        }

        /// <summary>
        /// Create the invitation object.
        /// </summary>
        /// <returns>Returns the invitation object.</returns>
        private static Invitation CreateInvitation()
        {
            // Set the invitation object.
            Invitation invitation = new Invitation();
            invitation.InvitedUserDisplayName = InvitedUserDisplayName;
            invitation.InvitedUserEmailAddress = InvitedUserEmailAddress;
            invitation.InviteRedirectUrl = "https://www.microsoft.com";
            invitation.SendInvitationMessage = true;
            return invitation;
        }

        /// <summary>
        /// Send the guest user invite request.
        /// </summary>
        /// <param name="invitation">Invitation object.</param>
        private static void SendInvitation(Invitation invitation)
        {
            string accessToken = GetAccessToken();

            HttpClient httpClient = GetHttpClient(accessToken);

            // Make the invite call.
            HttpContent content = new StringContent(JsonConvert.SerializeObject(invitation));
            content.Headers.Add("ContentType", "application/json");
            var postResponse = httpClient.PostAsync(InviteEndPoint, content).Result;
            string serverResponse = postResponse.Content.ReadAsStringAsync().Result;
            Console.WriteLine(serverResponse);
        }

        /// <summary>
        /// Get the HTTP client.
        /// </summary>
        /// <param name="accessToken">Access token</param>
        /// <returns>Returns the Http Client.</returns>
        private static HttpClient GetHttpClient(string accessToken)
        {
            // setup http client.
            HttpClient httpClient = new HttpClient();
            httpClient.Timeout = TimeSpan.FromSeconds(300);
            httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
            httpClient.DefaultRequestHeaders.Add("client-request-id", Guid.NewGuid().ToString());
            Console.WriteLine(
                "CorrelationID for the request: {0}",
                httpClient.DefaultRequestHeaders.GetValues("client-request-id").Single());
            return httpClient;
        }

        /// <summary>
        /// Get the access token for our application to talk to Microsoft Graph.
        /// </summary>
        /// <returns>Returns the access token for our application to talk to Microsoft Graph.</returns>
        private static string GetAccessToken()
        {
            string accessToken = null;

            // Get the access token for our application to talk to Microsoft Graph.
            try
            {
                AuthenticationContext testAuthContext =
                    new AuthenticationContext(string.Format("{0}/{1}", EstsLoginEndpoint, TenantID));
                AuthenticationResult testAuthResult = testAuthContext.AcquireTokenAsync(
                    GraphResource,
                    new ClientCredential(TestAppClientId, TestAppClientSecret)).Result;
                accessToken = testAuthResult.AccessToken;
            }
            catch (AdalException ex)
            {
                Console.WriteLine("An exception was thrown while fetching the token: {0}.", ex);
                throw;
            }

            return accessToken;
        }

        /// <summary>
        /// Invitation class.
        /// </summary>
        public class Invitation
        {
            /// <summary>
            /// Gets or sets display name.
            /// </summary>
            public string InvitedUserDisplayName { get; set; }

            /// <summary>
            /// Gets or sets display name.
            /// </summary>
            public string InvitedUserEmailAddress { get; set; }

            /// <summary>
            /// Gets or sets a value indicating whether Invitation Manager should send the email to InvitedUser.
            /// </summary>
            public bool SendInvitationMessage { get; set; }

            /// <summary>
            /// Gets or sets invitation redirect URL
            /// </summary>
            public string InviteRedirectUrl { get; set; }
        }
    }
}
```


## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
