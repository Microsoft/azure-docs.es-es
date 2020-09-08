---
title: 'Tutorial: Bot de Language Understanding C# v4'
description: Con C#, cree un bot de chat que se integre con la descripción de lenguaje (LUIS). El bot se compila con la versión 4 de Bot Framework y el servicio de bot de aplicación web de Azure.
ms.topic: tutorial
ms.date: 06/22/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 820cdeb692dffbf80a0e85c9d3dd86b703a4cae3
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88931274"
---
# <a name="tutorial-use-a-web-app-bot-enabled-with-language-understanding-in-c"></a>Tutorial: usar un bot de aplicación web habilitado con Language Understanding en C#

Use C# para crear un bot de chat que se integre con Language Understanding (LUIS). El bot se ha compilado con el recurso [bot de aplicación web](https://docs.microsoft.com/azure/bot-service/) de Azure y [Bot Framework versión](https://github.com/Microsoft/botbuilder-dotnet) V4.

**En este tutorial, aprenderá a:**

> [!div class="checklist"]
> * Crear un bot de aplicación web. Este proceso crea una nueva aplicación de LUIS.
> * Descargar el proyecto de bot creado por el servicio de bot web
> * Iniciar el emulador y el bot localmente en el equipo.
> * Ver los resultados de expresiones del bot.

## <a name="prerequisites"></a>Requisitos previos

* [Bot Emulator](https://aka.ms/abs/build/emulatordownload)
* [Visual Studio](https://visualstudio.microsoft.com/downloads/)

## <a name="create-a-web-app-bot-resource"></a>Crear un nuevo recurso de bot de aplicación web

1. En [Azure Portal](https://portal.azure.com), haga clic en **Crear un recurso**.

1. En el cuadro de búsqueda, busque y seleccione **Bot de aplicación web**. Seleccione **Crear**.

1. En **Bot Service**, proporcione la información necesaria:

    |Configuración|Propósito|Configuración sugerida|
    |--|--|--|
    |Bot handle (Identificador de bot)|Nombre del recurso|`luis-csharp-bot-` + `<your-name>`, por ejemplo, `luis-csharp-bot-johnsmith`|
    |Suscripción|Suscripción donde se creará el bot.|Suscripción principal.
    |Resource group|Grupo lógico de recursos de Azure|Crear un nuevo grupo para almacenar todos los recursos utilizados con este bot, denominando el grupo `luis-csharp-bot-resource-group`.|
    |Location|Región de Azure: no tiene que ser la misma que la región de creación o publicación de LUIS.|`westus`|
    |Plan de tarifa|Se usa para los límites de solicitud de servicio y facturación.|`F0` es el nivel Gratis.
    |Nombre de la aplicación|El nombre se usa como subdominio cuando el bot se implementa en la nube (por ejemplo, humanresourcesbot.azurewebsites.net).|`luis-csharp-bot-` + `<your-name>`, por ejemplo, `luis-csharp-bot-johnsmith`|
    |Plantilla de bot|Configuración de Bot Framework: ver la tabla siguiente|
    |Ubicación de la aplicación de LUIS|Debe ser la misma que la región de recursos de LUIS.|`westus`|
    |Plan de App Service/ubicación|No cambie el valor predeterminado proporcionado.|
    |Application Insights|No cambie el valor predeterminado proporcionado.|
    |Id. y contraseña de aplicación de Microsoft|No cambie el valor predeterminado proporcionado.|

1. En **Bot template** (Plantilla del bot), seleccione lo siguiente y luego elija el botón **Seleccionar** en esta configuración:

    |Configuración|Propósito|Número de selección|
    |--|--|--|
    |Lenguaje de SDK|Lenguaje de programación del bot|**C#**|
    |Bot|Tipo de bot|**Bot básico**|

1. Seleccione **Crear**. Esto crea e implementa el servicio de bots en Azure. Parte de este proceso crea una aplicación de LUIS denominada `luis-csharp-bot-XXXX`. Este nombre se basa en el nombre de la aplicación /Azure Bot Service.

    > [!div class="mx-imgBorder"]
    > [![Creación de un bot de aplicación web](./media/bfv4-csharp/create-web-app-service.png)](./media/bfv4-csharp/create-web-app-service.png#lightbox)

    Antes de continuar, espere hasta que se crea el servicio bot.

1. Seleccione `Go to resource` en la notificación para ir a la página del bot de la aplicación web.

## <a name="the-bot-has-a-language-understanding-model"></a>El bot tiene un modelo de Language Understanding.

El proceso de creación del servicio de bot también crea una nueva aplicación LUIS con intenciones y expresiones de ejemplo. El bot proporciona asignación de intenciones a la nueva aplicación de LUIS para las intenciones siguientes:

|Intenciones de LUIS del bot básico|Expresión de ejemplo|
|--|--|
|Reservar vuelo|`Travel to Paris`|
|Cancelar|`bye`|
|GetWeather|`what's the weather like?`|
|None|Nada fuera del dominio de la aplicación.|

## <a name="test-the-bot-in-web-chat"></a>Probar el bot en Chat en web

1. Mientras sigue en Azure Portal para el nuevo bot, seleccione **Test in Web Chat** (Probar en chat web).
1. En el cuadro de texto **Type your message** (Escriba su mensaje), escriba el texto `Book a flight from Seattle to Berlin tomorrow`. Al responder, el bot comprueba que desea reservar un vuelo.

    ![Captura de pantalla de Azure Portal, escribir el texto "hello".](./media/bfv4-nodejs/ask-bot-question-in-portal-test-in-web-chat.png)

    Puede usar la funcionalidad de prueba para probar rápidamente su bot. Para pruebas más completas, como la depuración, descargue el código de bot y use Visual Studio.

## <a name="download-the-web-app-bot-source-code"></a>Descarga del código fuente del bot de aplicación web

Con el fin de desarrollar el código del bot de aplicación web, descargue el código y úselo en el equipo local.

1. En Azure Portal, seleccione **Compilar** en la sección **Bot Management** (Administración del bot).

1. Seleccione **Download Bot source code** (Descargar el código fuente del bot).

    [![Descargar el código fuente del bot de aplicación web para el bot básico](../../../includes/media/cognitive-services-luis/bfv4/download-code.png)](../../../includes/media/cognitive-services-luis/bfv4/download-code.png#lightbox)

1. Cuando el cuadro de diálogo emergente le pregunte **Include app settings in the downloaded zip file?** (¿Incluir la configuración de la aplicación en el archivo zip descargado?), seleccione **Sí**.

1. Cuando el código fuente se haya comprimido, un mensaje proporcionará un vínculo para descargar el código. Seleccione el vínculo.

1. Guarde el archivo ZIP en el equipo local y extraiga los ficheros. Abra el proyecto con Visual Studio.

## <a name="review-code-to-send-utterance-to-luis-and-get-response"></a>Revisión del código para enviar la expresión a LUIS y obtener respuesta

1. Para enviar la expresión del usuario al punto de conexión de predicción LUIS, abra el archivo **FlightBookingRecognizer.cs**. Aquí es donde la expresión del usuario especificada en el bot se envía a LUIS. La respuesta de LUIS se devuelve desde el método **RecognizeAsync**.

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License.

    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.AI.Luis;
    using Microsoft.Extensions.Configuration;

    namespace Microsoft.BotBuilderSamples
    {
        public class FlightBookingRecognizer : IRecognizer
        {
            private readonly LuisRecognizer _recognizer;

            public FlightBookingRecognizer(IConfiguration configuration)
            {
                var luisIsConfigured = !string.IsNullOrEmpty(configuration["LuisAppId"]) && !string.IsNullOrEmpty(configuration["LuisAPIKey"]) && !string.IsNullOrEmpty(configuration["LuisAPIHostName"]);
                if (luisIsConfigured)
                {
                    var luisApplication = new LuisApplication(
                        configuration["LuisAppId"],
                        configuration["LuisAPIKey"],
                        "https://" + configuration["LuisAPIHostName"]);

                    _recognizer = new LuisRecognizer(luisApplication);
                }
            }

            // Returns true if luis is configured in the appsettings.json and initialized.
            public virtual bool IsConfigured => _recognizer != null;

            public virtual async Task<RecognizerResult> RecognizeAsync(ITurnContext turnContext, CancellationToken cancellationToken)
                => await _recognizer.RecognizeAsync(turnContext, cancellationToken);

            public virtual async Task<T> RecognizeAsync<T>(ITurnContext turnContext, CancellationToken cancellationToken)
                where T : IRecognizerConvert, new()
                => await _recognizer.RecognizeAsync<T>(turnContext, cancellationToken);
        }
    }
    ```

1. Al abrir **dialogs -> mainDialog.cs**, se captura la expresión y se envía a executeLuisQuery en el método actStep.

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License.

    using System;
    using System.Collections.Generic;
    using System.Linq;
    using System.Threading;
    using System.Threading.Tasks;
    using Microsoft.Bot.Builder;
    using Microsoft.Bot.Builder.Dialogs;
    using Microsoft.Bot.Schema;
    using Microsoft.Extensions.Logging;
    using Microsoft.Recognizers.Text.DataTypes.TimexExpression;

    namespace Microsoft.BotBuilderSamples.Dialogs
    {
        public class MainDialog : ComponentDialog
        {
            private readonly FlightBookingRecognizer _luisRecognizer;
            protected readonly ILogger Logger;

            // Dependency injection uses this constructor to instantiate MainDialog
            public MainDialog(FlightBookingRecognizer luisRecognizer, BookingDialog bookingDialog, ILogger<MainDialog> logger)
                : base(nameof(MainDialog))
            {
                _luisRecognizer = luisRecognizer;
                Logger = logger;

                AddDialog(new TextPrompt(nameof(TextPrompt)));
                AddDialog(bookingDialog);
                AddDialog(new WaterfallDialog(nameof(WaterfallDialog), new WaterfallStep[]
                {
                    IntroStepAsync,
                    ActStepAsync,
                    FinalStepAsync,
                }));

                // The initial child Dialog to run.
                InitialDialogId = nameof(WaterfallDialog);
            }

            private async Task<DialogTurnResult> IntroStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
            {
                if (!_luisRecognizer.IsConfigured)
                {
                    await stepContext.Context.SendActivityAsync(
                        MessageFactory.Text("NOTE: LUIS is not configured. To enable all capabilities, add 'LuisAppId', 'LuisAPIKey' and 'LuisAPIHostName' to the appsettings.json file.", inputHint: InputHints.IgnoringInput), cancellationToken);

                    return await stepContext.NextAsync(null, cancellationToken);
                }

                // Use the text provided in FinalStepAsync or the default if it is the first time.
                var messageText = stepContext.Options?.ToString() ?? "What can I help you with today?\nSay something like \"Book a flight from Paris to Berlin on March 22, 2020\"";
                var promptMessage = MessageFactory.Text(messageText, messageText, InputHints.ExpectingInput);
                return await stepContext.PromptAsync(nameof(TextPrompt), new PromptOptions { Prompt = promptMessage }, cancellationToken);
            }

            private async Task<DialogTurnResult> ActStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
            {
                if (!_luisRecognizer.IsConfigured)
                {
                    // LUIS is not configured, we just run the BookingDialog path with an empty BookingDetailsInstance.
                    return await stepContext.BeginDialogAsync(nameof(BookingDialog), new BookingDetails(), cancellationToken);
                }

                // Call LUIS and gather any potential booking details. (Note the TurnContext has the response to the prompt.)
                var luisResult = await _luisRecognizer.RecognizeAsync<FlightBooking>(stepContext.Context, cancellationToken);
                switch (luisResult.TopIntent().intent)
                {
                    case FlightBooking.Intent.BookFlight:
                        await ShowWarningForUnsupportedCities(stepContext.Context, luisResult, cancellationToken);

                        // Initialize BookingDetails with any entities we may have found in the response.
                        var bookingDetails = new BookingDetails()
                        {
                            // Get destination and origin from the composite entities arrays.
                            Destination = luisResult.ToEntities.Airport,
                            Origin = luisResult.FromEntities.Airport,
                            TravelDate = luisResult.TravelDate,
                        };

                        // Run the BookingDialog giving it whatever details we have from the LUIS call, it will fill out the remainder.
                        return await stepContext.BeginDialogAsync(nameof(BookingDialog), bookingDetails, cancellationToken);

                    case FlightBooking.Intent.GetWeather:
                        // We haven't implemented the GetWeatherDialog so we just display a TODO message.
                        var getWeatherMessageText = "TODO: get weather flow here";
                        var getWeatherMessage = MessageFactory.Text(getWeatherMessageText, getWeatherMessageText, InputHints.IgnoringInput);
                        await stepContext.Context.SendActivityAsync(getWeatherMessage, cancellationToken);
                        break;

                    default:
                        // Catch all for unhandled intents
                        var didntUnderstandMessageText = $"Sorry, I didn't get that. Please try asking in a different way (intent was {luisResult.TopIntent().intent})";
                        var didntUnderstandMessage = MessageFactory.Text(didntUnderstandMessageText, didntUnderstandMessageText, InputHints.IgnoringInput);
                        await stepContext.Context.SendActivityAsync(didntUnderstandMessage, cancellationToken);
                        break;
                }

                return await stepContext.NextAsync(null, cancellationToken);
            }

            // Shows a warning if the requested From or To cities are recognized as entities but they are not in the Airport entity list.
            // In some cases LUIS will recognize the From and To composite entities as a valid cities but the From and To Airport values
            // will be empty if those entity values can't be mapped to a canonical item in the Airport.
            private static async Task ShowWarningForUnsupportedCities(ITurnContext context, FlightBooking luisResult, CancellationToken cancellationToken)
            {
                var unsupportedCities = new List<string>();

                var fromEntities = luisResult.FromEntities;
                if (!string.IsNullOrEmpty(fromEntities.From) && string.IsNullOrEmpty(fromEntities.Airport))
                {
                    unsupportedCities.Add(fromEntities.From);
                }

                var toEntities = luisResult.ToEntities;
                if (!string.IsNullOrEmpty(toEntities.To) && string.IsNullOrEmpty(toEntities.Airport))
                {
                    unsupportedCities.Add(toEntities.To);
                }

                if (unsupportedCities.Any())
                {
                    var messageText = $"Sorry but the following airports are not supported: {string.Join(',', unsupportedCities)}";
                    var message = MessageFactory.Text(messageText, messageText, InputHints.IgnoringInput);
                    await context.SendActivityAsync(message, cancellationToken);
                }
            }

            private async Task<DialogTurnResult> FinalStepAsync(WaterfallStepContext stepContext, CancellationToken cancellationToken)
            {
                // If the child dialog ("BookingDialog") was cancelled, the user failed to confirm or if the intent wasn't BookFlight
                // the Result here will be null.
                if (stepContext.Result is BookingDetails result)
                {
                    // Now we have all the booking details call the booking service.

                    // If the call to the booking service was successful tell the user.

                    var timeProperty = new TimexProperty(result.TravelDate);
                    var travelDateMsg = timeProperty.ToNaturalLanguage(DateTime.Now);
                    var messageText = $"I have you booked to {result.Destination} from {result.Origin} on {travelDateMsg}";
                    var message = MessageFactory.Text(messageText, messageText, InputHints.IgnoringInput);
                    await stepContext.Context.SendActivityAsync(message, cancellationToken);
                }

                // Restart the main dialog with a different message the second time around
                var promptMessage = "What else can I do for you?";
                return await stepContext.ReplaceDialogAsync(InitialDialogId, promptMessage, cancellationToken);
            }
        }
    }
    ```

## <a name="start-the-bot-code-in-visual-studio"></a>Iniciar el código del bot en Visual Studio

En Visual Studio 2019, inicie el bot. Se abre una ventana del explorador con el sitio web del bot de aplicación web en `http://localhost:3978/`. Se muestra una página principal con información sobre su bot.

![Se muestra una página principal con información sobre su bot.](./media/bfv4-csharp/running-bot-web-home-page-success.png)

## <a name="use-the-bot-emulator-to-test-the-bot"></a>Uso del emulador de bot para probar el bot

1. Inicie el emulador de bots y seleccione **Open Bot** (Abrir bot).
1. En el cuadro de diálogo emergente **Open a bot** (Abrir un bot), escriba la dirección URL del bot, por ejemplo, `http://localhost:3978/api/messages`. La ruta `/api/messages` es la dirección web para el bot.
1. Escriba los valores que desee en **Microsoft App ID** (Id. de aplicación de Microsoft) y **Microsoft App password** (Contraseña de aplicación de Microsoft), que se encuentran en el archivo **appsettings.json** de la raíz del código de bot que descargó y seleccione **Conectar**.

1. En el emulador de bot, escriba `Book a flight from Seattle to Berlin tomorrow` y obtenga la misma respuesta para el bot básico que la que recibió en el **Test in Web Chat** (Probar en chat web) en una sección anterior.

    [![Respuesta de bot básico en el emulador](./media/bfv4-nodejs/ask-bot-emulator-a-question-and-get-response.png)](./media/bfv4-nodejs/ask-bot-emulator-a-question-and-get-response.png#lightbox)

1. Seleccione **Sí**. El bot responde con un resumen de sus acciones.
1. En el registro del emulador bot, seleccione la línea que incluye `<- trace LuisV3 Trace`. Esto muestra la respuesta JSON de LUIS para la intención y las entidades de la expresión.

    [![Respuesta de bot básico en el emulador](./media/bfv4-nodejs/ask-luis-book-flight-question-get-json-response-in-bot-emulator.png)](./media/bfv4-nodejs/ask-luis-book-flight-question-get-json-response-in-bot-emulator.png#lightbox)

[!INCLUDE [Bot Information](../../../includes/cognitive-services-qnamaker-luis-bot-info.md)]

## <a name="next-steps"></a>Pasos siguientes

Consulte más [ejemplos](https://github.com/microsoft/botframework-solutions) con bots de conversación.

> [!div class="nextstepaction"]
> [Build a Language Understanding app with a custom subject domain](luis-quickstart-intents-only.md) (Compilación de una aplicación Language Understanding con un dominio de sujeto personalizado)
