---
title: 'Tutorial: Bot de Language Understanding Node.js v4'
description: Use Node.js para crear un bot de chat que se integre con Language Understanding (LUIS) en este tutorial. Este bot de chat utiliza la aplicación de recursos humanos para implementar con rapidez una solución de bot. El bot se compila con la versión 4 de Bot Framework y el bot de aplicación web de Azure.
ms.topic: tutorial
ms.date: 06/23/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 522502a043778d6f482c1756506063dfa6515ca3
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405031"
---
# <a name="tutorial-use-a-web-app-bot-enabled-with-language-understanding-in-nodejs"></a>Tutorial: Uso de un bot de aplicación web habilitado con Language Understanding en Node.js

Use Node.js para crear un bot de chat que se integre con Language Understanding (LUIS). El bot se ha compilado con el recurso [bot de aplicación web](https://docs.microsoft.com/azure/bot-service/) de Azure y [Bot Framework versión](https://github.com/Microsoft/botbuilder-dotnet) V4.

**En este tutorial, aprenderá a:**

> [!div class="checklist"]
> * Crear un bot de aplicación web. Este proceso crea una nueva aplicación de LUIS.
> * Descargar el proyecto de bot creado por el servicio de bot web
> * Iniciar el emulador y el bot localmente en el equipo.
> * Ver los resultados de expresiones del bot.

## <a name="prerequisites"></a>Requisitos previos

* [Bot Emulator](https://aka.ms/abs/build/emulatordownload)
* [Visual Studio Code](https://code.visualstudio.com/Download)

## <a name="create-a-web-app-bot-resource"></a>Crear un nuevo recurso de bot de aplicación web

1. En [Azure Portal](https://portal.azure.com), haga clic en **Crear un recurso**.

1. En el cuadro de búsqueda, busque y seleccione **Bot de aplicación web**. Seleccione **Crear**.

1. En **Bot Service**, proporcione la información necesaria:

    |Configuración|Propósito|Configuración sugerida|
    |--|--|--|
    |Bot handle (Identificador de bot)|Nombre del recurso|`luis-nodejs-bot-` + `<your-name>`, por ejemplo, `luis-nodejs-bot-johnsmith`|
    |Suscripción|Suscripción donde se creará el bot.|Suscripción principal.
    |Resource group|Grupo lógico de recursos de Azure|Crear un nuevo grupo para almacenar todos los recursos utilizados con este bot, denominando el grupo `luis-nodejs-bot-resource-group`.|
    |Location|Región de Azure: no tiene que ser la misma que la región de creación o publicación de LUIS.|`westus`|
    |Plan de tarifa|Se usa para los límites de solicitud de servicio y facturación.|`F0` es el nivel Gratis.
    |Nombre de la aplicación|El nombre se usa como subdominio cuando el bot se implementa en la nube (por ejemplo, humanresourcesbot.azurewebsites.net).|`luis-nodejs-bot-` + `<your-name>`, por ejemplo, `luis-nodejs-bot-johnsmith`|
    |Plantilla de bot|Configuración de Bot Framework: ver la tabla siguiente|
    |Ubicación de la aplicación de LUIS|Debe ser la misma que la región de recursos de LUIS.|`westus`|
    |Plan de App Service/ubicación|No cambie el valor predeterminado proporcionado.|
    |Application Insights|No cambie el valor predeterminado proporcionado.|
    |Id. y contraseña de aplicación de Microsoft|No cambie el valor predeterminado proporcionado.|

1. En **Bot template** (Plantilla del bot), seleccione lo siguiente y luego elija el botón **Seleccionar** en esta configuración:

    |Configuración|Propósito|Número de selección|
    |--|--|--|
    |Lenguaje de SDK|Lenguaje de programación del bot|**Node.js**|
    |Bot|Tipo de bot|**Bot básico**|

1. Seleccione **Crear**. Esto crea e implementa el servicio de bots en Azure. Parte de este proceso crea una aplicación de LUIS denominada `luis-nodejs-bot-XXXX`. Este nombre se basa en el nombre de la aplicación /Azure Bot Service.

    > [!div class="mx-imgBorder"]
    > [![Creación de un bot de aplicación web](./media/bfv4-nodejs/create-web-app-service.png)](./media/bfv4-nodejs/create-web-app-service.png#lightbox)

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

    Puede usar la funcionalidad de prueba para probar rápidamente el bot. Si lo que desea es realizar unas pruebas completas, que incluyan la depuración, descargue el código del bot y use Visual Studio Code.

## <a name="download-the-web-app-bot-source-code"></a>Descarga del código fuente del bot de aplicación web
Con el fin de desarrollar el código del bot de aplicación web, descargue el código y úselo en el equipo local.

1. En Azure Portal, seleccione **Compilar** en la sección **Bot Management** (Administración del bot).

1. Seleccione **Download Bot source code** (Descargar el código fuente del bot).

    [![Descargar el código fuente del bot de aplicación web para el bot básico](../../../includes/media/cognitive-services-luis/bfv4/download-code.png)](../../../includes/media/cognitive-services-luis/bfv4/download-code.png#lightbox)

1. Cuando el cuadro de diálogo emergente le pregunte **Include app settings in the downloaded zip file?** (¿Incluir la configuración de la aplicación en el archivo zip descargado?), seleccione **Sí**. Así se proporciona la configuración de LUIS.

1. Cuando el código fuente se haya comprimido, un mensaje proporcionará un vínculo para descargar el código. Seleccione el vínculo.

1. Guarde el archivo ZIP en el equipo local y extraiga los ficheros. Abra la carpeta del proyecto con Visual Studio Code.

## <a name="review-code-to-send-utterance-to-luis-and-get-response"></a>Revisión del código para enviar la expresión a LUIS y obtener respuesta

1. Para enviar la expresión del usuario al punto de conexión de predicción LUIS, abra el archivo **dialogs -> flightBookingRecognizer.cs**. Aquí es donde la expresión del usuario especificada en el bot se envía a LUIS. La respuesta de LUIS se devuelve desde el método **executeLuisQuery**.

    ```javascript
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License.

    const { LuisRecognizer } = require('botbuilder-ai');

    class FlightBookingRecognizer {
        constructor(config) {
            const luisIsConfigured = config && config.applicationId && config.endpointKey && config.endpoint;
            if (luisIsConfigured) {
                // Set the recognizer options depending on which endpoint version you want to use e.g v2 or v3.
                // More details can be found in https://docs.microsoft.com/en-gb/azure/cognitive-services/luis/luis-migration-api-v3
                const recognizerOptions = {
                    apiVersion: 'v3'
                };

                this.recognizer = new LuisRecognizer(config, recognizerOptions);
            }
        }

        get isConfigured() {
            return (this.recognizer !== undefined);
        }

        /**
         * Returns an object with preformatted LUIS results for the bot's dialogs to consume.
         * @param {TurnContext} context
         */
        async executeLuisQuery(context) {
            return await this.recognizer.recognize(context);
        }

        getFromEntities(result) {
            let fromValue, fromAirportValue;
            if (result.entities.$instance.From) {
                fromValue = result.entities.$instance.From[0].text;
            }
            if (fromValue && result.entities.From[0].Airport) {
                fromAirportValue = result.entities.From[0].Airport[0][0];
            }

            return { from: fromValue, airport: fromAirportValue };
        }

        getToEntities(result) {
            let toValue, toAirportValue;
            if (result.entities.$instance.To) {
                toValue = result.entities.$instance.To[0].text;
            }
            if (toValue && result.entities.To[0].Airport) {
                toAirportValue = result.entities.To[0].Airport[0][0];
            }

            return { to: toValue, airport: toAirportValue };
        }

        /**
         * This value will be a TIMEX. And we are only interested in a Date so grab the first result and drop the Time part.
         * TIMEX is a format that represents DateTime expressions that include some ambiguity. e.g. missing a Year.
         */
        getTravelDate(result) {
            const datetimeEntity = result.entities.datetime;
            if (!datetimeEntity || !datetimeEntity[0]) return undefined;

            const timex = datetimeEntity[0].timex;
            if (!timex || !timex[0]) return undefined;

            const datetime = timex[0].split('T')[0];
            return datetime;
        }
    }

    module.exports.FlightBookingRecognizer = FlightBookingRecognizer;
    ```

1. El archivo **dialogs -> MainDialog** captura la expresión y la envía a executeLuisQuery en el método actStep.

    ```javascript
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License.

    const { TimexProperty } = require('@microsoft/recognizers-text-data-types-timex-expression');
    const { MessageFactory, InputHints } = require('botbuilder');
    const { LuisRecognizer } = require('botbuilder-ai');
    const { ComponentDialog, DialogSet, DialogTurnStatus, TextPrompt, WaterfallDialog } = require('botbuilder-dialogs');

    const MAIN_WATERFALL_DIALOG = 'mainWaterfallDialog';

    class MainDialog extends ComponentDialog {
        constructor(luisRecognizer, bookingDialog) {
            super('MainDialog');

            if (!luisRecognizer) throw new Error('[MainDialog]: Missing parameter \'luisRecognizer\' is required');
            this.luisRecognizer = luisRecognizer;

            if (!bookingDialog) throw new Error('[MainDialog]: Missing parameter \'bookingDialog\' is required');

            // Define the main dialog and its related components.
            // This is a sample "book a flight" dialog.
            this.addDialog(new TextPrompt('TextPrompt'))
                .addDialog(bookingDialog)
                .addDialog(new WaterfallDialog(MAIN_WATERFALL_DIALOG, [
                    this.introStep.bind(this),
                    this.actStep.bind(this),
                    this.finalStep.bind(this)
                ]));

            this.initialDialogId = MAIN_WATERFALL_DIALOG;
        }

        /**
         * The run method handles the incoming activity (in the form of a TurnContext) and passes it through the dialog system.
         * If no dialog is active, it will start the default dialog.
         * @param {*} turnContext
         * @param {*} accessor
         */
        async run(turnContext, accessor) {
            const dialogSet = new DialogSet(accessor);
            dialogSet.add(this);

            const dialogContext = await dialogSet.createContext(turnContext);
            const results = await dialogContext.continueDialog();
            if (results.status === DialogTurnStatus.empty) {
                await dialogContext.beginDialog(this.id);
            }
        }

        /**
         * First step in the waterfall dialog. Prompts the user for a command.
         * Currently, this expects a booking request, like "book me a flight from Paris to Berlin on march 22"
         * Note that the sample LUIS model will only recognize Paris, Berlin, New York and London as airport cities.
         */
        async introStep(stepContext) {
            if (!this.luisRecognizer.isConfigured) {
                const messageText = 'NOTE: LUIS is not configured. To enable all capabilities, add `LuisAppId`, `LuisAPIKey` and `LuisAPIHostName` to the .env file.';
                await stepContext.context.sendActivity(messageText, null, InputHints.IgnoringInput);
                return await stepContext.next();
            }

            const messageText = stepContext.options.restartMsg ? stepContext.options.restartMsg : 'What can I help you with today?\nSay something like "Book a flight from Paris to Berlin on March 22, 2020"';
            const promptMessage = MessageFactory.text(messageText, messageText, InputHints.ExpectingInput);
            return await stepContext.prompt('TextPrompt', { prompt: promptMessage });
        }

        /**
         * Second step in the waterfall.  This will use LUIS to attempt to extract the origin, destination and travel dates.
         * Then, it hands off to the bookingDialog child dialog to collect any remaining details.
         */
        async actStep(stepContext) {
            const bookingDetails = {};

            if (!this.luisRecognizer.isConfigured) {
                // LUIS is not configured, we just run the BookingDialog path.
                return await stepContext.beginDialog('bookingDialog', bookingDetails);
            }

            // Call LUIS and gather any potential booking details. (Note the TurnContext has the response to the prompt)
            const luisResult = await this.luisRecognizer.executeLuisQuery(stepContext.context);
            switch (LuisRecognizer.topIntent(luisResult)) {
            case 'BookFlight': {
                // Extract the values for the composite entities from the LUIS result.
                const fromEntities = this.luisRecognizer.getFromEntities(luisResult);
                const toEntities = this.luisRecognizer.getToEntities(luisResult);

                // Show a warning for Origin and Destination if we can't resolve them.
                await this.showWarningForUnsupportedCities(stepContext.context, fromEntities, toEntities);

                // Initialize BookingDetails with any entities we may have found in the response.
                bookingDetails.destination = toEntities.airport;
                bookingDetails.origin = fromEntities.airport;
                bookingDetails.travelDate = this.luisRecognizer.getTravelDate(luisResult);
                console.log('LUIS extracted these booking details:', JSON.stringify(bookingDetails));

                // Run the BookingDialog passing in whatever details we have from the LUIS call, it will fill out the remainder.
                return await stepContext.beginDialog('bookingDialog', bookingDetails);
            }

            case 'GetWeather': {
                // We haven't implemented the GetWeatherDialog so we just display a TODO message.
                const getWeatherMessageText = 'TODO: get weather flow here';
                await stepContext.context.sendActivity(getWeatherMessageText, getWeatherMessageText, InputHints.IgnoringInput);
                break;
            }

            default: {
                // Catch all for unhandled intents
                const didntUnderstandMessageText = `Sorry, I didn't get that. Please try asking in a different way (intent was ${ LuisRecognizer.topIntent(luisResult) })`;
                await stepContext.context.sendActivity(didntUnderstandMessageText, didntUnderstandMessageText, InputHints.IgnoringInput);
            }
            }

            return await stepContext.next();
        }

        /**
         * Shows a warning if the requested From or To cities are recognized as entities but they are not in the Airport entity list.
         * In some cases LUIS will recognize the From and To composite entities as a valid cities but the From and To Airport values
         * will be empty if those entity values can't be mapped to a canonical item in the Airport.
         */
        async showWarningForUnsupportedCities(context, fromEntities, toEntities) {
            const unsupportedCities = [];
            if (fromEntities.from && !fromEntities.airport) {
                unsupportedCities.push(fromEntities.from);
            }

            if (toEntities.to && !toEntities.airport) {
                unsupportedCities.push(toEntities.to);
            }

            if (unsupportedCities.length) {
                const messageText = `Sorry but the following airports are not supported: ${ unsupportedCities.join(', ') }`;
                await context.sendActivity(messageText, messageText, InputHints.IgnoringInput);
            }
        }

        /**
         * This is the final step in the main waterfall dialog.
         * It wraps up the sample "book a flight" interaction with a simple confirmation.
         */
        async finalStep(stepContext) {
            // If the child dialog ("bookingDialog") was cancelled or the user failed to confirm, the Result here will be null.
            if (stepContext.result) {
                const result = stepContext.result;
                // Now we have all the booking details.

                // This is where calls to the booking AOU service or database would go.

                // If the call to the booking service was successful tell the user.
                const timeProperty = new TimexProperty(result.travelDate);
                const travelDateMsg = timeProperty.toNaturalLanguage(new Date(Date.now()));
                const msg = `I have you booked to ${ result.destination } from ${ result.origin } on ${ travelDateMsg }.`;
                await stepContext.context.sendActivity(msg, msg, InputHints.IgnoringInput);
            }

            // Restart the main dialog with a different message the second time around
            return await stepContext.replaceDialog(this.initialDialogId, { restartMsg: 'What else can I do for you?' });
        }
    }

    module.exports.MainDialog = MainDialog;
    ```

<a name="ask-bot-a-question-for-the-book-flight-intent"></a>

## <a name="start-the-bot-code"></a>Inicio del código del bot

1. Abra una consola de Windows o un terminal de Linux o macOS.

1. Cambie al directorio que contiene el código del bot y escriba el siguiente comando para instalar la dependencia dotenv:

    ```console
    npm install dotenv --save
    ```

1. Escriba el siguiente comando para iniciar el bot:

    ```console
    node index.js
    ```

Se abre una ventana del explorador con el sitio web del bot de aplicación web en `http://localhost:3978/`. Se muestra una página principal con información sobre su bot.

![Se muestra una página principal con información sobre su bot.](./media/bfv4-csharp/running-bot-web-home-page-success.png)

## <a name="use-the-bot-emulator-to-test-the-bot"></a>Uso del emulador de bot para probar el bot

Formule una pregunta al bot para la intención de reservar vuelo.

1. Inicie el emulador de bots y seleccione **Open Bot** (Abrir bot).
1. En el cuadro de diálogo emergente **Open a bot** (Abrir un bot), escriba la dirección URL del bot, por ejemplo, `http://localhost:3978/api/messages`. La ruta `/api/messages` es la dirección web para el bot.
1. Escriba valores en **Microsoft App ID** (Id. de aplicación de Microsoft) y **Microsoft App password** (Contraseña de aplicación de Microsoft), que se encuentran en el archivo **.env** en la raíz del código de bot que descargó.

1. En el emulador de bot, escriba `Book a flight from Seattle to Berlin tomorrow` y obtenga la misma respuesta para el bot básico que la que recibió en el **Test in Web Chat** (Probar en chat web).

    [![Respuesta de bot básico en el emulador](./media/bfv4-nodejs/ask-bot-emulator-a-question-and-get-response.png)](./media/bfv4-nodejs/ask-bot-emulator-a-question-and-get-response.png#lightbox)

1. Seleccione **Sí**. El bot responde con un resumen de sus acciones.
1. En el registro del emulador bot, seleccione la línea que incluye `<- trace LuisV3 Trace`. Esto muestra la respuesta JSON de LUIS para la intención y las entidades de la expresión.

    [![Respuesta de bot básico en el emulador](./media/bfv4-nodejs/ask-luis-book-flight-question-get-json-response-in-bot-emulator.png)](./media/bfv4-nodejs/ask-luis-book-flight-question-get-json-response-in-bot-emulator.png#lightbox)

[!INCLUDE [Bot Information](../../../includes/cognitive-services-qnamaker-luis-bot-info.md)]

## <a name="next-steps"></a>Pasos siguientes

Consulte más [ejemplos](https://github.com/microsoft/botframework-solutions) con bots de conversación.

> [!div class="nextstepaction"]
> [Build a Language Understanding app with a custom subject domain](luis-quickstart-intents-only.md) (Compilación de una aplicación Language Understanding con un dominio de sujeto personalizado)
