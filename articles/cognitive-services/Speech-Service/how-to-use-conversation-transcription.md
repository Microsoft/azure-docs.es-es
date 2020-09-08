---
title: 'Transcripción de conversaciones en tiempo real (versión preliminar): servicio de voz'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar la transcripción de conversaciones en tiempo real con el SDK de voz. Disponible para C++, C# y Java.
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: weixu
ms.custom: devx-track-csharp
ms.openlocfilehash: a425c75dfd57f0d5f9c98b73d8b351972b70703a
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88918662"
---
# <a name="real-time-conversation-transcription-preview"></a>Transcripción de conversaciones en tiempo real (versión preliminar)

La API **ConversationTranscriber** del SDK de voz le permite transcribir reuniones y conversaciones con la posibilidad de agregar, quitar e identificar varios participantes haciendo streaming de audio al servicio de voz mediante `PullStream` o `PushStream`. Este tema requiere que sepa cómo usar la conversión de voz en texto con el SDK de voz (versión 1.8.0 o posterior). Para más información, consulte [Qué es el servicio de voz](overview.md).

## <a name="limitations"></a>Limitaciones

- Se admite la API ConversationTranscriber para C++, C# y Java en Windows, Linux y Android.
- Actualmente disponible para los idiomas "en-US" y "zh-CN" en las siguientes regiones: _centralus_ y _eastasia_.
- Requiere una matriz con varios micrófonos circular de siete micrófonos con un flujo de referencia de reproducción. La matriz de micrófonos debe cumplir [nuestra especificación](https://aka.ms/sdsdk-microphone).
- El [SDK de dispositivos de voz](speech-devices-sdk.md) proporciona dispositivos adecuados y una aplicación de ejemplo que muestra la transcripción de conversaciones.

## <a name="optional-sample-code-resources"></a>Recursos de código de ejemplo opcionales

El SDK de dispositivos de voz proporciona código de ejemplo en Java para la captura de audio en tiempo real mediante ocho canales.

- [ROOBO device sample code](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK/blob/master/Samples/Java/Android/Speech%20Devices%20SDK%20Starter%20App/example/app/src/main/java/com/microsoft/cognitiveservices/speech/samples/sdsdkstarterapp/ConversationTranscription.java) (Código de ejemplo de dispositivo ROOBO)
- [Azure Kinect Dev Kit sample code](https://github.com/Azure-Samples/Cognitive-Services-Speech-Devices-SDK/blob/master/Samples/Java/Windows_Linux/SampleDemo/src/com/microsoft/cognitiveservices/speech/samples/Cts.java) (Código de ejemplo de Azure Kinect Dev Kit)

## <a name="prerequisites"></a>Requisitos previos

Una suscripción al servicio de voz. Puede crear una [cuenta gratuita de Azure](https://azure.microsoft.com/free/cognitive-services/) si no tiene ninguna.

## <a name="create-voice-signatures"></a>Creación de firmas de voz

El primer paso es crear firmas de voz para los participantes de la conversación para una identificación eficaz del hablante.

### <a name="audio-input-requirements"></a>Requisitos de entrada de audio

- El archivo WAVE de audio de entrada para la creación de firmas de voz debe aparecer en las muestras de 16 bits, con frecuencia de muestreo de 16 kHz y un formato de canal único (mono).
- La longitud recomendada para cada muestra de audio está entre 30 segundos y dos minutos.

### <a name="sample-code"></a>Código de ejemplo

El ejemplo siguiente muestra dos maneras diferentes de crear la firma de voz [mediante la API de REST](https://aka.ms/cts/signaturegenservice) de C#. Tenga en cuenta que necesitará sustituir la información real de "YourSubscriptionKey", su nombre de archivo WAVE de "speakerVoice.wav" y su región de `{region}` y "YourServiceRegion" (_centralus_ o _eastasia_).

```csharp
class Program
{
    static async Task CreateVoiceSignatureByUsingFormData()
    {
        // Replace with your own region
        var region = "YourServiceRegion";
        // Change the name of the wave file to match yours
        byte[] fileBytes = File.ReadAllBytes(@"speakerVoice.wav");
        var form = new MultipartFormDataContent();
        var content = new ByteArrayContent(fileBytes);
        form.Add(content, "file", "file");
        var client = new HttpClient();
        // Add your subscription key to the header Ocp-Apim-Subscription-Key directly
        // Replace "YourSubscriptionKey" with your own subscription key
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "YourSubscriptionKey");
        // Edit with your desired region for `{region}`
        var response = await client.PostAsync($"https://signature.{region}.cts.speech.microsoft.com/api/v1/Signature/GenerateVoiceSignatureFromFormData", form);
        // A voice signature contains Version, Tag and Data key values from the Signature json structure from the Response body.
        // Voice signature format example: { "Version": <Numeric string or integer value>, "Tag": "string", "Data": "string" }
        var jsonData = await response.Content.ReadAsStringAsync();
    }

    static async Task CreateVoiceSignatureByUsingBody()
    {
        // Replace with your own region
        var region = "YourServiceRegion";
        // Change the name of the wave file to match yours
        byte[] fileBytes = File.ReadAllBytes(@"speakerVoice.wav");
        var content = new ByteArrayContent(fileBytes);

        var client = new HttpClient();
        // Add your subscription key to the header Ocp-Apim-Subscription-Key directly
        // Replace "YourSubscriptionKey" with your own subscription key
        client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "YourSubscriptionKey");
        // Edit with your desired region for `{region}`
        var response = await client.PostAsync($"https://signature.{region}.cts.speech.microsoft.com/api/v1/Signature/GenerateVoiceSignatureFromByteArray", content);
        // A voice signature contains Version, Tag and Data key values from the Signature json structure from the Response body.
        // Voice signature format example: { "Version": <Numeric string or integer value>, "Tag": "string", "Data": "string" }
        var jsonData = await response.Content.ReadAsStringAsync();
    }

    static void Main(string[] args)
    {
        CreateVoiceSignatureByUsingFormData().Wait();
        CreateVoiceSignatureByUsingBody().Wait();
    }
}
```

## <a name="transcribe-conversations"></a>Transcripción de conversaciones

El siguiente código de ejemplo muestra cómo transcribir conversaciones en tiempo real para tres hablantes. Se da por supuesto que ya ha creado firmas de voz para cada hablante como se muestra arriba. Sustituya la información real de "YourSubscriptionKey" y "YourServiceRegion" al crear el objeto SpeechConfig.

Estos son aspectos destacados de código de ejemplo:

- Creación de un objeto `Conversation` a partir del objeto `SpeechConfig` mediante un identificador de reunión generado con `Guid.NewGuid()`
- Creación de un objeto `ConversationTranscriber` y participación en la conversación con `JoinConversationAsync()` para iniciar la transcripción
- Registro de los eventos de interés
- Incorporación de participantes a la conversación o eliminación de los mismos mediante el objeto Conversation
- Transmisión del audio
- A partir de la versión 1.9.0 en adelante del SDK de Voz, se admiten los tipos de valor `int` y `string` en el campo Versión de la firma de voz.

El identificador del hablante y la transcripción vuelven en los eventos registrados.

```csharp
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;
using Microsoft.CognitiveServices.Speech.Transcription;

public class MyConversationTranscriber
{
    public static async Task ConversationWithPullAudioStreamAsync()
    {
        // Creates an instance of a speech config with specified subscription key and service region
        // Replace with your own subscription key and region
        var config = SpeechConfig.FromSubscription("YourSubscriptionKey", "YourServiceRegion");
        config.SetProperty("ConversationTranscriptionInRoomAndOnline", "true");
        var stopTranscription = new TaskCompletionSource<int>();

        // Create an audio stream from a wav file or from the default microphone if you want to stream live audio from the supported devices
        // Replace with your own audio file name and Helper class which implements AudioConfig using PullAudioInputStreamCallback
        using (var audioInput = Helper.OpenWavFile(@"8channelsOfRecordedPCMAudio.wav"))
        {
            var meetingId = Guid.NewGuid().ToString();
            using (var conversation = await Conversation.CreateConversationAsync(config, meetingId).ConfigureAwait(false))
            {
                // Create a conversation transcriber using audio stream input
                using (var conversationTranscriber = new ConversationTranscriber(audioInput))
                {
                    await conversationTranscriber.JoinConversationAsync(conversation);

                    // Subscribe to events
                    conversationTranscriber.Transcribing += (s, e) =>
                    {
                            Console.WriteLine($"TRANSCRIBING: Text={e.Result.Text}");
                    };

                    conversationTranscriber.Transcribed += (s, e) =>
                    {
                        if (e.Result.Reason == ResultReason.RecognizedSpeech)
                        {
                            Console.WriteLine($"TRANSCRIBED: Text={e.Result.Text}, UserID={e.Result.UserId}");
                        }
                        else if (e.Result.Reason == ResultReason.NoMatch)
                        {
                            Console.WriteLine($"NOMATCH: Speech could not be recognized.");
                        }
                    };

                    conversationTranscriber.Canceled += (s, e) =>
                    {
                        Console.WriteLine($"CANCELED: Reason={e.Reason}");

                        if (e.Reason == CancellationReason.Error)
                        {
                            Console.WriteLine($"CANCELED: ErrorCode={e.ErrorCode}");
                            Console.WriteLine($"CANCELED: ErrorDetails={e.ErrorDetails}");
                            Console.WriteLine($"CANCELED: Did you update the subscription info?");
                            stopTranscription.TrySetResult(0);
                        }
                    };

                    conversationTranscriber.SessionStarted += (s, e) =>
                    {
                        Console.WriteLine("\nSession started event.");
                    };

                    conversationTranscriber.SessionStopped += (s, e) =>
                    {
                        Console.WriteLine("\nSession stopped event.");
                        Console.WriteLine("\nStop recognition.");
                        stopTranscription.TrySetResult(0);
                    };

                    // Add participants to the conversation.
                    // Create voice signatures using REST API described in the earlier section in this document.
                    // Voice signature needs to be in the following format:
                    // { "Version": <Numeric string or integer value>, "Tag": "string", "Data": "string" }

                    var speakerA = Participant.From("Speaker_A", "en-us", signatureA);
                    var speakerB = Participant.From("Speaker_B", "en-us", signatureB);
                    var speakerC = Participant.From("SPeaker_C", "en-us", signatureC);
                    await conversation.AddParticipantAsync(speakerA);
                    await conversation.AddParticipantAsync(speakerB);
                    await conversation.AddParticipantAsync(speakerC);

                    // Starts transcribing of the conversation. Uses StopTranscribingAsync() to stop transcribing when all participants leave.
                    await conversationTranscriber.StartTranscribingAsync().ConfigureAwait(false);

                    // Waits for completion.
                    // Use Task.WaitAny to keep the task rooted.
                    Task.WaitAny(new[] { stopTranscription.Task });

                    // Stop transcribing the conversation.
                    await conversationTranscriber.StopTranscribingAsync().ConfigureAwait(false);
                 }
            }
       }
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Transcripción de conversaciones asincrónica](how-to-async-conversation-transcription.md)
