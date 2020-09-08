---
title: 'Creación de un modelo de inquilino (versión preliminar): servicio de voz'
titleSuffix: Azure Cognitive Services
description: Genere automáticamente un modelo de inquilino compatible y seguro (Habla personalizada con datos de Office 365) que use los datos de Office 365 para ofrecer un reconocimiento de voz óptimo para términos específicos de una organización.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: tutorial
ms.date: 06/25/2020
ms.author: erhopf
ms.custom: devx-track-csharp
ms.openlocfilehash: 52e4271fca02dc9b0eab45ca98581ecd85119b59
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934487"
---
# <a name="tutorial-create-a-tenant-model-preview"></a>Tutorial: Creación de un modelo de inquilino (versión preliminar)

Tenant Model (Habla personalizada con datos de Office 365) es un servicio de participación para clientes empresariales de Office 365 que genera automáticamente un modelo de reconocimiento de voz personalizado a partir de los datos de Office 365 de su organización. El modelo está optimizado para términos técnicos, jerga y nombres de personas, y todo ello de forma segura y compatible.

> [!IMPORTANT]
> Si su organización se inscribe mediante el servicio Tenant Model, el servicio de voz puede acceder al modelo de lenguaje de su organización. El modelo se genera a partir de documentos y correos electrónicos de grupos públicos de Office 365, que puede ver cualquier usuario de la organización. El administrador de Office 365 de la organización puede activar o desactivar el uso del modelo de lenguaje en toda la organización desde el portal de administración de Office 365.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Inscribirse en Tenant Model desde el Centro de administración de Microsoft 365
> * Obtener una clave de suscripción del servicio Voz
> * Crear un modelo de inquilino
> * Implementar un modelo de inquilino
> * Usar un modelo de inquilino con el SDK de Voz

## <a name="enroll-in-the-tenant-model-service"></a>Inscripción en el servicio Tenant Model

Para poder implementar un modelo de inquilino, es preciso estar inscrito en el servicio Tenant Model. La inscripción se completa en el Centro de administración de Microsoft 365 y solo puede realizarla un administrador de Microsoft 365.

1. Inicie sesión en el [Centro de administración de Microsoft 365](https://admin.microsoft.com).

1. En el panel izquierdo, seleccione **Configuración**, de nuevo **Configuración** en el menú anidado y, finalmente, seleccione **Servicios de voz de Azure** en la ventana principal.

   ![El panel "Servicios y complementos"](media/tenant-language-model/tenant-language-model-enrollment.png)

1. Seleccione la casilla **Permitir el modelo de idioma de toda la organización** y, después, **Guardar cambios**.

   ![El panel Servicios de voz de Azure](media/tenant-language-model/tenant-language-model-enrollment-2.png)

Para desactivar la instancia del modelo de inquilino:
1. Repita los pasos 1 y 2 anteriores.
1. Desactive la casilla **Permitir el modelo de idioma de toda la organización** y seleccione **Guardar cambios**.

## <a name="get-a-speech-subscription-key"></a>Obtener una clave de suscripción del servicio Voz

Para usar un modelo de inquilino con el SDK de Voz, necesita un recurso de Voz y su clave de suscripción asociada.

1. Inicie sesión en [Azure Portal](https://aka.ms/azureportal).
1. Seleccione **Crear un recurso**.
1. En el cuadro **Buscar en Marketplace**, escriba **Speech**.
1. En la lista de resultados, seleccione **Speech** y, después, **Crear**.
1. Siga las instrucciones en pantalla para crear el recurso. Asegúrese de lo siguiente:
   * La **ubicación** se ha establecido en **eastus** o **westus**.
   * El **plan de tarifa** está establecido en **S0**.
1. Seleccione **Crear**.

   Después de unos minutos, se crea el recurso. La clave de suscripción está disponible en la sección **Información general** del recurso.

## <a name="create-a-language-model"></a>Creación de un modelo de lenguaje

Una vez que el administrador haya habilitado Tenant Model para su organización, puede crear un modelo de lenguaje basado en los datos de Office 365.

1. Inicie sesión en [Speech Studio](https://speech.microsoft.com/).
1. En la parte superior derecha, seleccione **Settings** (Configuración) (icono del engranaje) y, después, **Tenant Model settings** (Configuración de Tenant Model).

   ![El vínculo "Tenant Model settings" (Configuración de Tenant Model)](media/tenant-language-model/tenant-language-settings.png)

   Speech Studio muestra un mensaje que le indica que si es apto para crear un modelo de inquilino.

   > [!NOTE]
   > Los clientes empresariales de Office 365 en Norteamérica son aptos para crear un modelo de inquilino (inglés). Para los clientes de Caja de seguridad del cliente, Clave de cliente u Office 365 Administración Pública característica no está disponible. Para determinar si es un cliente de Caja de seguridad del cliente o de Clave de cliente, consulte:
   > * [Caja de seguridad del cliente](/microsoft-365/compliance/customer-lockbox-requests)
   > * [Clave de cliente](/microsoft-365/compliance/customer-key-overview)
   > * [Office 365 Administración Pública](https://www.microsoft.com/microsoft-365/government)

1. Seleccione **Habilitar envío**.

   Cuando el modelo de inquilino esté listo, recibirá un mensaje de correo electrónico de confirmación con más instrucciones.

## <a name="deploy-your-tenant-model"></a>Implementación de un modelo de inquilino

Cuando la instancia del modelo de inquilino esté lista, siga estos pasos para implementarla:

1. En el mensaje de correo electrónico de confirmación, seleccione el botón **View model** (Ver modelo). O bien, inicie sesión en [Speech Studio](https://speech.microsoft.com/).
1. En la parte superior derecha, seleccione **Settings** (Configuración) (icono del engranaje) y, después, **Tenant Model settings** (Configuración de Tenant Model).

   ![El vínculo "Tenant Model settings" (Configuración de Tenant Model)](media/tenant-language-model/tenant-language-settings.png)

1. Seleccione **Implementar**.

   Cuando el modelo se haya implementado, el estado cambiará a *Implementado*.

## <a name="use-your-tenant-model-with-the-speech-sdk"></a>Usar un modelo de inquilino con el SDK de Voz

Ahora que ha implementado el modelo, puede usarlo con el SDK de Voz. En esta sección, se usa un código de ejemplo para llamar al servicio de voz mediante la autenticación de Azure Active Directory (Azure AD).

Echemos un vistazo al código que usará para llamar al SDK de Voz en C#. En este ejemplo, el reconocimiento de voz se realiza mediante un modelo de inquilino. En esta guía se da por supuesto que la plataforma ya está configurada. Si necesita ayuda para el programa de configuración, consulte [Inicio rápido: Reconocimiento de voz, C# (.NET Core)](quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=dotnetcore).

Copie este código en el proyecto:

```csharp
namespace PrincetonSROnly.FrontEnd.Samples
{
    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Net.Http;
    using System.Text;
    using System.Text.RegularExpressions;
    using System.Threading.Tasks;
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.IdentityModel.Clients.ActiveDirectory;
    using Newtonsoft.Json.Linq;

    // ServiceApplicationId is a fixed value. No need to change it.

    public class TenantLMSample
    {
        private const string EndpointUriArgName = "EndpointUri";
        private const string SubscriptionKeyArgName = "SubscriptionKey";
        private const string UsernameArgName = "Username";
        private const string PasswordArgName = "Password";
        private const string ClientApplicationId = "f87bc118-1576-4097-93c9-dbf8f45ef0dd";
        private const string ServiceApplicationId = "18301695-f99d-4cae-9618-6901d4bdc7be";

        public static async Task ContinuousRecognitionWithTenantLMAsync(Uri endpointUri, string subscriptionKey, string audioDirPath, string username, string password)
        {
            var config = SpeechConfig.FromEndpoint(endpointUri, subscriptionKey);

            // Passing client specific information for obtaining LM
            if (string.IsNullOrEmpty(username) || string.IsNullOrEmpty(password))
            {
                config.AuthorizationToken = await AcquireAuthTokenWithInteractiveLoginAsync().ConfigureAwait(false);
            }
            else
            {
                config.AuthorizationToken = await AcquireAuthTokenWithUsernamePasswordAsync(username, password).ConfigureAwait(false);
            }

            var stopRecognition = new TaskCompletionSource<int>();

            // Creates a speech recognizer using file as audio input.
            // Replace with your own audio file name.
            using (var audioInput = AudioConfig.FromWavFileInput(audioDirPath))
            {
                using (var recognizer = new SpeechRecognizer(config, audioInput))
                {
                    // Subscribes to events
                    recognizer.Recognizing += (s, e) =>
                    {
                        Console.WriteLine($"RECOGNIZING: Text={e.Result.Text}");
                    };

                    recognizer.Recognized += (s, e) =>
                    {
                        if (e.Result.Reason == ResultReason.RecognizedSpeech)
                        {
                            Console.WriteLine($"RECOGNIZED: Text={e.Result.Text}");
                        }
                        else if (e.Result.Reason == ResultReason.NoMatch)
                        {
                            Console.WriteLine($"NOMATCH: Speech could not be recognized.");
                        }
                    };

                    recognizer.Canceled += (s, e) =>
                    {
                        Console.WriteLine($"CANCELED: Reason={e.Reason}");
                        if (e.Reason == CancellationReason.Error)
                        {
                            Exception exp = new Exception(string.Format("Error Code: {0}\nError Details{1}\nIs your subscription information updated?", e.ErrorCode, e.ErrorDetails));
                            throw exp;
                        }

                        stopRecognition.TrySetResult(0);
                    };

                    recognizer.SessionStarted += (s, e) =>
                    {
                        Console.WriteLine("\n    Session started event.");
                    };

                    recognizer.SessionStopped += (s, e) =>
                    {
                        Console.WriteLine("\n    Session stopped event.");
                        Console.WriteLine("\nStop recognition.");
                        stopRecognition.TrySetResult(0);
                    };

                    // Starts continuous recognition. Uses StopContinuousRecognitionAsync() to stop recognition.
                    await recognizer.StartContinuousRecognitionAsync().ConfigureAwait(false);

                    // Waits for completion.
                    // Use Task.WaitAny to keep the task rooted.
                    Task.WaitAny(new[] { stopRecognition.Task });

                    // Stops recognition.
                    await recognizer.StopContinuousRecognitionAsync().ConfigureAwait(false);
                }
            }
        }

        public static void Main(string[] args)
        {
            var arguments = new Dictionary<string, string>();
            string inputArgNamePattern = "--";
            Regex regex = new Regex(inputArgNamePattern);
            if (args.Length > 0)
            {
                foreach (var arg in args)
                {
                    var userArgs = arg.Split("=");
                    arguments[regex.Replace(userArgs[0], string.Empty)] = userArgs[1];
                }
            }

            var endpointString = arguments.GetValueOrDefault(EndpointUriArgName, $"wss://westus.online.princeton.customspeech.ai/msgraphcustomspeech/conversation/v1");
            var endpointUri = new Uri(endpointString);

            if (!arguments.ContainsKey(SubscriptionKeyArgName))
            {
                Exception exp = new Exception("Subscription Key missing! Please pass in a Cognitive services subscription Key using --SubscriptionKey=\"your_subscription_key\"" +
                    "Find more information on creating a Cognitive services resource and accessing your Subscription key here: https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account?tabs=multiservice%2Cwindows");
                throw exp;
            }

            var subscriptionKey = arguments[SubscriptionKeyArgName];
            var username = arguments.GetValueOrDefault(UsernameArgName, null);
            var password = arguments.GetValueOrDefault(PasswordArgName, null);

            var audioDirPath = Path.Combine(Path.GetDirectoryName(System.Reflection.Assembly.GetExecutingAssembly().Location), "../../../AudioSamples/DictationBatman.wav");
            if (!File.Exists(audioDirPath))
            {
                Exception exp = new Exception(string.Format("Audio File does not exist at path: {0}", audioDirPath));
                throw exp;
            }

            ContinuousRecognitionWithTenantLMAsync(endpointUri, subscriptionKey, audioDirPath, username, password).GetAwaiter().GetResult();
        }

        private static async Task<string> AcquireAuthTokenWithUsernamePasswordAsync(string username, string password)
        {
            var tokenEndpoint = "https://login.microsoftonline.com/common/oauth2/token";
            var postBody = $"resource={ServiceApplicationId}&client_id={ClientApplicationId}&grant_type=password&username={username}&password={password}";
            var stringContent = new StringContent(postBody, Encoding.UTF8, "application/x-www-form-urlencoded");
            using (HttpClient httpClient = new HttpClient())
            {
                var response = await httpClient.PostAsync(tokenEndpoint, stringContent).ConfigureAwait(false);

                if (response.IsSuccessStatusCode)
                {
                    var result = await response.Content.ReadAsStringAsync().ConfigureAwait(false);

                    JObject jobject = JObject.Parse(result);
                    return jobject["access_token"].Value<string>();
                }
                else
                {
                    throw new Exception($"Requesting token from {tokenEndpoint} failed with status code {response.StatusCode}: {await response.Content.ReadAsStringAsync().ConfigureAwait(false)}");
                }
            }
        }

        private static async Task<string> AcquireAuthTokenWithInteractiveLoginAsync()
        {
            var authContext = new AuthenticationContext("https://login.windows.net/microsoft.onmicrosoft.com");
            var deviceCodeResult = await authContext.AcquireDeviceCodeAsync(ServiceApplicationId, ClientApplicationId).ConfigureAwait(false);

            Console.WriteLine(deviceCodeResult.Message);

            var authResult = await authContext.AcquireTokenByDeviceCodeAsync(deviceCodeResult).ConfigureAwait(false);

            return authResult.AccessToken;
        }
    }
}
```

Después tendrá que volver a compilar y ejecutar el proyecto desde la línea de comandos. Antes de ejecutar el comando, actualice algunos parámetros, para lo que debe seguir estos pasos:

1. Reemplace `<Username>` y `<Password>` por los valores de un usuario de inquilino válido.
1. Reemplace `<Subscription-Key>` por la clave de suscripción del recurso de Voz. Este valor está disponible en la hoja de **información general** del recurso de Voz de [Azure Portal](https://aka.ms/azureportal).
1. Reemplace `<Endpoint-Uri>` por el siguiente punto de conexión. Asegúrese de que reemplaza `{your region}` por la región donde se creó el recurso de Voz. Se admiten estas regiones: `westus`, `westus2`y `eastus`. La información de la región está disponible en la sección de **información general** del recurso de Voz de [Azure Portal](https://aka.ms/azureportal).
   ```
   "wss://{your region}.online.princeton.customspeech.ai/msgraphcustomspeech/conversation/v1".
   ```
1. Ejecute el siguiente comando:

   ```bash
   dotnet TenantLMSample.dll --Username=<Username> --Password=<Password> --SubscriptionKey=<Subscription-Key> --EndpointUri=<Endpoint-Uri>
   ```

En este tutorial, ha aprendido a usar los datos de Office 365 para crear un modelo de reconocimiento de voz personalizado, a implementarlo y a usarlo con el SDK de Voz.

## <a name="next-steps"></a>Pasos siguientes

* [Speech Studio](https://speech.microsoft.com/)
* [Acerca del SDK de Voz](speech-sdk.md)
