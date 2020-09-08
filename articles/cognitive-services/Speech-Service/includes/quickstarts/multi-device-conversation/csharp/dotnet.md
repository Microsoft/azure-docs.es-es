---
services: cognitive-services
author: ralphe
manager: cpoulain
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 04/04/2020
ms.author: ralphe
ms.custom: devx-track-csharp
ms.openlocfilehash: dff5c7c7018945bb93ce492ee3205f3567dff1d7
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88925914"
---
## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, compruebe lo siguiente:

> [!div class="checklist"]
> * [Ha creado un recurso de Voz de Azure](../../../../get-started.md)
> * [Configuración del entorno de desarrollo y creación de un proyecto vacío](../../../../quickstarts/setup-platform.md?tabs=dotnet&pivots=programming-language-csharp)

## <a name="add-sample-code"></a>Incorporación de código de ejemplo

1. Abra el archivo **Program.cs** y sustituya todo el código por el siguiente:

    ```csharp
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Transcription;
    using System;
    using System.Threading.Tasks;
    
    namespace HelloWorld
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                await CreateConversationAsync();
            }
    
            static async Task CreateConversationAsync()
            {
                // Replace these values with the details of your Cognitive Speech subscription
                string subscriptionKey = "YourSubscriptionKey";

                // Replace below with your region identifier from here: https://aka.ms/speech/sdkregion
                string region = "YourServiceRegion";
    
                // Sets source and target languages.
                // Replace with the languages of your choice, from list found here: https://aka.ms/speech/sttt-languages
                string fromLanguage = "en-US";
                string toLanguage = "de";
    
                // Set this to the display name you want for the conversation host
                string displayName = "The host";
    
                // Create the task completion source that will be used to wait until the user presses Ctrl + C
                var completionSource = new TaskCompletionSource<bool>();
    
                // Register to listen for Ctrl + C
                Console.CancelKeyPress += (s, e) =>
                {
                    completionSource.TrySetResult(true);
                    e.Cancel = true; // don't terminate the current process
                };
    
                // Create an instance of the speech translation config
                var config = SpeechTranslationConfig.FromSubscription(subscriptionKey, region);
                config.SpeechRecognitionLanguage = fromLanguage;
                config.AddTargetLanguage(toLanguage);
    
                // Create the conversation
                using (var conversation = await Conversation.CreateConversationAsync(config).ConfigureAwait(false))
                {
                    // Start the conversation so the host user and others can join
                    await conversation.StartConversationAsync().ConfigureAwait(false);
    
                    // Get the conversation ID. It will be up to your scenario to determine how this is shared with other participants.
                    string conversationId = conversation.ConversationId;
                    Console.WriteLine($"Created '{conversationId}' conversation");
    
                    // At this point, you can use the conversation object to manage the conversation. 
                    // For example, to mute everyone else in the room you can call this method:
                    await conversation.MuteAllParticipantsAsync().ConfigureAwait(false);
    
                    // Configure which audio source you want to use. If you are using a text only language, you 
                    // can use the other overload of the constructor that takes no arguments
                    var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
                    using (var conversationTranslator = new ConversationTranslator(audioConfig))
                    {
                        // You should connect all the event handlers you need at this point
                        conversationTranslator.SessionStarted += (s, e) =>
                        {
                            Console.WriteLine($"Session started: {e.SessionId}");
                        };
                        conversationTranslator.SessionStopped += (s, e) =>
                        {
                            Console.WriteLine($"Session stopped: {e.SessionId}");
                        };
                        conversationTranslator.Canceled += (s, e) =>
                        {
                            switch (e.Reason)
                            {
                                case CancellationReason.EndOfStream:
                                    Console.WriteLine($"End of audio reached");
                                    break;
    
                                case CancellationReason.Error:
                                    Console.WriteLine($"Canceled due to error. {e.ErrorCode}: {e.ErrorDetails}");
                                    break;
                            }
                        };
                        conversationTranslator.ConversationExpiration += (s, e) =>
                        {
                            Console.WriteLine($"Conversation will expire in {e.ExpirationTime.TotalMinutes} minutes");
                        };
                        conversationTranslator.ParticipantsChanged += (s, e) =>
                        {
                            Console.Write("The following participant(s) have ");
                            switch (e.Reason)
                            {
                                case ParticipantChangedReason.JoinedConversation:
                                    Console.Write("joined");
                                    break;
    
                                case ParticipantChangedReason.LeftConversation:
                                    Console.Write("left");
                                    break;
    
                                case ParticipantChangedReason.Updated:
                                    Console.Write("been updated");
                                    break;
                            }
    
                            Console.WriteLine(":");
    
                            foreach (var participant in e.Participants)
                            {
                                Console.WriteLine($"\t{participant.DisplayName}");
                            }
                        };
                        conversationTranslator.TextMessageReceived += (s, e) =>
                        {
                            Console.WriteLine($"Received an instant message from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                            foreach (var entry in e.Result.Translations)
                            {
                                Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                            }
                        };
                        conversationTranslator.Transcribed += (s, e) =>
                        {
                            Console.WriteLine($"Received a transcription from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                            foreach (var entry in e.Result.Translations)
                            {
                                Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                            }
                        };
                        conversationTranslator.Transcribing += (s, e) =>
                        {
                            Console.WriteLine($"Received a partial transcription from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                            foreach (var entry in e.Result.Translations)
                            {
                                Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                            }
                        };
    
                        // Enter the conversation to start receiving events
                        await conversationTranslator.JoinConversationAsync(conversation, displayName).ConfigureAwait(false);
    
                        // You can now send an instant message to all other participants in the room
                        await conversationTranslator.SendTextMessageAsync("The instant message to send").ConfigureAwait(false);
    
                        // If specified a speech-to-text language, you can start capturing audio
                        await conversationTranslator.StartTranscribingAsync().ConfigureAwait(false);
                        Console.WriteLine("Started transcribing. Press Ctrl + c to stop");
    
                        // At this point, you should start receiving transcriptions for what you are saying using the default microphone. Press Ctrl+c to stop audio capture
                        await completionSource.Task.ConfigureAwait(false);
    
                        // Stop audio capture
                        await conversationTranslator.StopTranscribingAsync().ConfigureAwait(false);
    
                        // Leave the conversation. After this you will no longer receive events
                        await conversationTranslator.LeaveConversationAsync().ConfigureAwait(false);
                    }
    
                    // End the conversation
                    await conversation.EndConversationAsync().ConfigureAwait(false);
    
                    // Delete the conversation. Any other participants that are still in the conversation will be removed
                    await conversation.DeleteConversationAsync().ConfigureAwait(false);
                }
            }
        }
    }
    ```

1. En el mismo archivo, reemplace la cadena `YourSubscriptionKey` por su clave de suscripción de Cognitive Speech.

1. Reemplace la cadena `YourServiceRegion` por la [región](~/articles/cognitive-services/Speech-Service/regions.md) asociada a su suscripción.

1. En la barra de menús, elija **Archivo** > **Guardar todo**.

## <a name="build-and-run-the-application-to-create-a-new-conversation"></a>Compilación y ejecución de la aplicación para crear una conversación

1. En la barra de menús, seleccione **Compilar** > **Compilar solución** para compilar la aplicación. El código se debería compilar sin errores ahora.

1. Elija **Depurar** > **Iniciar depuración** o presione **F5** para iniciar la aplicación **HelloWorld**.

1. Cuando aparezca el mensaje `Started transcribing`, puede empezar a hablar. Verá que mientras habla aparecen las transcripciones.
    - Si comparte el código de la conversación con los demás y estos se unen a la conversación, verá también sus transcripciones.

1. Cuando haya terminado de hablar, presione <kbd>Ctrl+C</kbd> para detener la captura de audio y finalizar la conversación.

## <a name="build-and-run-the-application-to-join-an-existing-conversation"></a>Compilación y ejecución de la aplicación para unirse a una conversación existente

1. Copie y pegue la siguiente función en su archivo **Program.cs**:

    ```csharp
    static async Task JoinConversationAsync(string conversationId)
    {
        // Set this to the display name you want for the participant
        string displayName = "participant";

        // Set the speech to text, or text language you want to use
        string language = "en-US";

        // Create the task completion source that will be used to wait until the user presses Ctrl + c
        var completionSource = new TaskCompletionSource<bool>();

        // Register to listen for Ctrl+C
        Console.CancelKeyPress += (s, e) =>
        {
            completionSource.TrySetResult(true);
            e.Cancel = true; // don't terminate the current process
        };

        // As a participant, you don't need to specify any subscription key, or region. You can directly create
        // the conversation translator object
        var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
        using (var conversationTranslator = new ConversationTranslator(audioConfig))
        {
            // Register for any events you are interested here. For now let's just register for
            // transcription, and instant message events
            conversationTranslator.TextMessageReceived += (s, e) =>
            {
                Console.WriteLine($"Received an instant message from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                foreach (var entry in e.Result.Translations)
                {
                    Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                }
            };
            conversationTranslator.Transcribed += (s, e) =>
            {
                Console.WriteLine($"Received a transcription from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                foreach (var entry in e.Result.Translations)
                {
                    Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                }
            };
            conversationTranslator.Transcribing += (s, e) =>
            {
                Console.WriteLine($"Received a partial transcription from '{e.Result.ParticipantId}': '{e.Result.Text}'");
                foreach (var entry in e.Result.Translations)
                {
                    Console.WriteLine($"\tTranslated into '{entry.Key}': '{entry.Value}'");
                }
            };
    
            // To start receiving events, you will need to join the conversation
            await conversationTranslator.JoinConversationAsync(conversationId, displayName, language).ConfigureAwait(false);

            // You can now send an instant message
            await conversationTranslator.SendTextMessageAsync("Message from participant").ConfigureAwait(false);

            // Start capturing audio if you specified a speech-to-text language
            await conversationTranslator.StartTranscribingAsync().ConfigureAwait(false);
            Console.WriteLine("Started transcribing. Press Ctrl-C to stop");

            // At this point, you should start receiving transcriptions for what you are saying using
            // the default microphone. Press Ctrl+C to stop audio capture
            await completionSource.Task.ConfigureAwait(false);

            // Stop audio capture
            await conversationTranslator.StopTranscribingAsync().ConfigureAwait(false);

            // Leave the conversation. You will stop receiving events after this
            await conversationTranslator.LeaveConversationAsync().ConfigureAwait(false);
        }
    }
    ```

2. Reemplace `CreateConversationAsync();` en la función `public static async Task Main(string[] args)` por:

    ```csharp
    // Set this to the conversation you want to join
    JoinConversationAsync("YourConversationId");
    ```

[!INCLUDE [create-from-web](../create-from-web.md)]

4. Vuelva a Visual Studio y reemplace la cadena `YourConversationId` por el identificador de conversación que creó en el paso anterior.

5. En la barra de menús, seleccione **Compilar** > **Compilar solución** para compilar la aplicación. El código se debería compilar sin errores ahora.

6. Elija **Depurar** > **Iniciar depuración** o presione **F5** para iniciar la aplicación **HelloWorld**.

7. Cuando aparezca el mensaje `Started transcribing`, puede empezar a hablar. Verá que mientras habla aparecen las transcripciones.
    - Si vuelve al explorador, verá que las transcripciones aparecen ahí mientras habla.

8. Cuando haya terminado de hablar, presione <kbd>Ctrl+C</kbd> para detener la captura de audio y finalizar la conversación.

9. Vuelva al explorador y salga de la conversación con el botón de salida situado en la esquina superior derecha.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [footer](./footer.md)]
