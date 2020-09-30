---
title: 'Inicio rápido de Speaker Recognition: servicio de voz'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar Speaker Recognition del SDK de voz para responder a la pregunta "¿quién habla?". En este inicio rápido, obtendrá información sobre los patrones de diseño que se usan con más frecuencia para trabajar con la identificación y verificación del hablante, que usan la biometría de voz para identificar voces únicas.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: quickstart
ms.date: 09/02/2020
ms.author: trbye
ms.custom: devx-track-csharp, cog-serv-seo-aug-2020
keywords: speaker recognition, reconocimiento del hablante, biometría de voz
ms.openlocfilehash: f22eb02e6812bef73133b1778484f9f4640245cd
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89410550"
---
# <a name="get-started-with-speaker-recognition"></a>Introducción a Speaker Recognition

En este inicio rápido, aprenderá los patrones de diseño básicos de Speaker Recognition mediante el SDK de Voz, que incluyen:

* Comprobación dependiente e independiente del texto.
* Identificación del hablante para identificar una muestra de voz entre un grupo de voces.
* Eliminación de perfiles de voz.

Para obtener una visión general de los conceptos de reconocimiento de voz, consulte el artículo de [información general](speaker-recognition-overview.md).

## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](get-started.md).

> [!IMPORTANT]
> Speaker Recognition *solo* se admite actualmente en los recursos de Voz de Azure creados en la región de `westus`.

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar Speech SDK. Utilice las siguientes instrucciones en función de la plataforma:

* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=dotnet&pivots=programming-language-csharp" target="_blank">.NET Framework <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=dotnetcore&pivots=programming-language-csharp" target="_blank">.NET Core <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=unity&pivots=programming-language-csharp" target="_blank">Unity <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=uwps&pivots=programming-language-csharp" target="_blank">UWP <span class="docon docon-navigate-external x-hidden-focus"></span></a>
* <a href="https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?tabs=xaml&pivots=programming-language-csharp" target="_blank">Xamarin <span class="docon docon-navigate-external x-hidden-focus"></span></a>

## <a name="import-dependencies"></a>Dependencias de importación

Para ejecutar los ejemplos de este artículo, incluya las siguientes instrucciones de `using` al principio del script.

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;
```

## <a name="create-a-speech-configuration"></a>Creación de una configuración de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechConfig`](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig?view=azure-dotnet). En este ejemplo, se crea un elemento [`SpeechConfig`](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig?view=azure-dotnet) mediante una clave de suscripción y una región. Cree también código reutilizable básico para usarlo en el resto del artículo y que modificará cuando realice distintas personalizaciones.

Tenga en cuenta que la región está establecida en `westus`, ya que es la única región admitida para el servicio.

```csharp
public class Program 
{
    static async Task Main(string[] args)
    {
        // replace with your own subscription key 
        string subscriptionKey = "YourSubscriptionKey";
        string region = "westus";
        var config = SpeechConfig.FromSubscription(subscriptionKey, region);
    }
}
```

## <a name="text-dependent-verification"></a>Comprobación dependiente del texto

Speaker Verification es el acto de confirmar que un hablante coincide con una voz conocida o **inscrita**. El primer paso es **inscribir** un perfil de voz, de modo que el servicio tenga algo para comparar futuras muestras de voz. En este ejemplo, se inscribe el perfil mediante una estrategia **dependiente del texto**, que requiere que se use una frase de contraseña específica para la inscripción y la comprobación. Vea los [documentos de referencia](https://docs.microsoft.com/rest/api/speakerrecognition/) para obtener una lista de frases de contraseña admitidas.

Empiece por crear la siguiente función en la clase `Program` para inscribir un perfil de voz.

```csharp
public static async Task VerificationEnroll(SpeechConfig config, Dictionary<string, string> profileMapping)
{
    using (var client = new VoiceProfileClient(config))
    using (var profile = await client.CreateProfileAsync(VoiceProfileType.TextDependentVerification, "en-us"))
    {
        using (var audioInput = AudioConfig.FromDefaultMicrophoneInput())
        {
            Console.WriteLine($"Enrolling profile id {profile.Id}.");
            // give the profile a human-readable display name
            profileMapping.Add(profile.Id, "Your Name");

            VoiceProfileEnrollmentResult result = null;
            while (result is null || result.RemainingEnrollmentsCount > 0)
            {
                Console.WriteLine("Speak the passphrase, \"My voice is my passport, verify me.\"");
                result = await client.EnrollProfileAsync(profile, audioInput);
                Console.WriteLine($"Remaining enrollments needed: {result.RemainingEnrollmentsCount}");
                Console.WriteLine("");
            }
            
            if (result.Reason == ResultReason.EnrolledVoiceProfile)
            {
                await SpeakerVerify(config, profile, profileMapping);
            }
            else if (result.Reason == ResultReason.Canceled)
            {
                var cancellation = VoiceProfileEnrollmentCancellationDetails.FromResult(result);
                Console.WriteLine($"CANCELED {profile.Id}: ErrorCode={cancellation.ErrorCode} ErrorDetails={cancellation.ErrorDetails}");
            }
        }
    }
}
```

En esta función, `await client.CreateProfileAsync()` es lo que crea realmente el nuevo perfil de voz. Una vez creada, debe especificar cómo va a introducir las muestras de audio. En este ejemplo, se usa `AudioConfig.FromDefaultMicrophoneInput()` para capturar audio del dispositivo de entrada predeterminado. A continuación, debe inscribir muestras de audio en un bucle `while` que realice un seguimiento del número de muestras restantes y necesarias para la inscripción. En cada iteración, `client.EnrollProfileAsync(profile, audioInput)` le pedirá que diga la frase de contraseña a través del micrófono y agregará la muestra al perfil de voz.

Una vez completada la inscripción, debe llamar a `await SpeakerVerify(config, profile, profileMapping)` para comprobarla con el perfil que acaba de crear. Agregue otra función para definir `SpeakerVerify`.

```csharp
public static async Task SpeakerVerify(SpeechConfig config, VoiceProfile profile, Dictionary<string, string> profileMapping)
{
    var speakerRecognizer = new SpeakerRecognizer(config, AudioConfig.FromDefaultMicrophoneInput());
    var model = SpeakerVerificationModel.FromProfile(profile);

    Console.WriteLine("Speak the passphrase to verify: \"My voice is my passport, please verify me.\"");
    var result = await speakerRecognizer.RecognizeOnceAsync(model);
    Console.WriteLine($"Verified voice profile for speaker {profileMapping[result.ProfileId]}, score is {result.Score}");
}
```

En esta función, se pasa el objeto `VoiceProfile` que acaba de crear para inicializar un modelo con el que realizar la comprobación. A continuación, `await speakerRecognizer.RecognizeOnceAsync(model)` le pide que diga de nuevo la frase de contraseña, pero esta vez la validará con el perfil de voz y devolverá una puntuación de similitud comprendida entre 0,0 y 1,0. El objeto `result` también devuelve `Accept` o `Reject`, en función de si la frase de contraseña coincide o no.

A continuación, modifique la función `Main()` para llamar a las nuevas funciones que ha creado. Además, tenga en cuenta que crea un objeto `Dictionary<string, string>` para pasar por referencia en las llamadas de función. El motivo es que el servicio no permite almacenar un nombre legible con un objeto `VoiceProfile` creado y solo almacena un número de identificación con fines de privacidad. En la función `VerificationEnroll`, agrega a este diccionario una entrada con el identificador recién creado, junto con un nombre de texto. En escenarios de desarrollo de aplicaciones en los que es necesario mostrar un nombre legible, **debe almacenar esta asignación en algún lugar, ya que el servicio no puede almacenarla**.

```csharp
static async Task Main(string[] args)
{
    string subscriptionKey = "YourSubscriptionKey";
    string region = "westus";
    var config = SpeechConfig.FromSubscription(subscriptionKey, region);

    // persist profileMapping if you want to store a record of who the profile is
    var profileMapping = new Dictionary<string, string>();
    await VerificationEnroll(config, profileMapping);

    Console.ReadLine();
}
```

Ejecute el script y se le pedirá que diga la frase *My voice is my passport, verify me* (Mi voz es mi pasaporte, verifícame) tres veces para la inscripción y una vez más para la comprobación. El resultado devuelto es la puntuación de similitud, que puede usar para crear sus propios umbrales personalizados para la comprobación.

```shell
Enrolling profile id 87-2cef-4dff-995b-dcefb64e203f.
Speak the passphrase, "My voice is my passport, verify me."
Remaining enrollments needed: 2

Speak the passphrase, "My voice is my passport, verify me."
Remaining enrollments needed: 1

Speak the passphrase, "My voice is my passport, verify me."
Remaining enrollments needed: 0

Speak the passphrase to verify: "My voice is my passport, verify me."
Verified voice profile for speaker Your Name, score is 0.915581
```

## <a name="text-independent-verification"></a>Comprobación independiente del texto

A diferencia de la comprobación **dependiente del texto**, la comprobación **independiente del texto**:

* No requiere que se diga una frase de contraseña determinada; se puede decir cualquier cosa.
* No requiere tres muestras de audio, pero *sí* 20 segundos de audio total.

Realice un par de cambios sencillos en la función `VerificationEnroll` para cambiar a la comprobación **independiente del texto**. En primer lugar, cambie el tipo de comprobación a `VoiceProfileType.TextIndependentVerification`. A continuación, cambie el bucle `while` para realizar un seguimiento de `result.RemainingEnrollmentsSpeechLength`, que le seguirá solicitando que hable hasta que se hayan capturado 20 segundos de audio.

```csharp
public static async Task VerificationEnroll(SpeechConfig config, Dictionary<string, string> profileMapping)
{
    using (var client = new VoiceProfileClient(config))
    using (var profile = await client.CreateProfileAsync(VoiceProfileType.TextIndependentVerification, "en-us"))
    {
        using (var audioInput = AudioConfig.FromDefaultMicrophoneInput())
        {
            Console.WriteLine($"Enrolling profile id {profile.Id}.");
            // give the profile a human-readable display name
            profileMapping.Add(profile.Id, "Your Name");

            VoiceProfileEnrollmentResult result = null;
            while (result is null || result.RemainingEnrollmentsSpeechLength > TimeSpan.Zero)
            {
                Console.WriteLine("Continue speaking to add to the profile enrollment sample.");
                result = await client.EnrollProfileAsync(profile, audioInput);
                Console.WriteLine($"Remaining enrollment audio time needed: {result.RemainingEnrollmentsSpeechLength}");
                Console.WriteLine("");
            }
            
            if (result.Reason == ResultReason.EnrolledVoiceProfile)
            {
                await SpeakerVerify(config, profile, profileMapping);
            }
            else if (result.Reason == ResultReason.Canceled)
            {
                var cancellation = VoiceProfileEnrollmentCancellationDetails.FromResult(result);
                Console.WriteLine($"CANCELED {profile.Id}: ErrorCode={cancellation.ErrorCode} ErrorDetails={cancellation.ErrorDetails}");
            }
        }
    }
}
```

Vuelva a ejecutar el programa y diga cualquier cosa durante la fase de comprobación, ya que no se necesita una frase de contraseña. De nuevo, se devuelve la puntuación de similitud.

```shell
Enrolling profile id 4tt87d4-f2d3-44ae-b5b4-f1a8d4036ee9.
Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:15.3200000

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:09.8100008

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:05.1900000

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:00.8700000

Continue speaking to add to the profile enrollment sample.
Remaining enrollment audio time needed: 00:00:00

Speak the passphrase to verify: "My voice is my passport, please verify me."
Verified voice profile for speaker Your Name, score is 0.849409
```

## <a name="speaker-identification"></a>Identificación del hablante

Speaker Identification se utiliza para determinar **quién** está hablando de un grupo determinado de voces inscritas. El proceso es muy similar al de **comprobación independiente del texto**. La diferencia principal es que se puede realizar la comprobación con varios perfiles de voz a la vez, en lugar de con uno solo.

Cree una función `IdentificationEnroll` para inscribir varios perfiles de voz. El proceso de inscripción de cada perfil es el mismo que el proceso de inscripción de **comprobación independiente del texto** y requiere 20 segundos de audio para cada perfil. Esta función acepta una lista de cadenas `profileNames` y creará un nuevo perfil de voz para cada nombre de la lista. La función devuelve una lista de objetos `VoiceProfile`, que se usan en la siguiente función para identificar a un hablante.

```csharp
public static async Task<List<VoiceProfile>> IdentificationEnroll(SpeechConfig config, List<string> profileNames, Dictionary<string, string> profileMapping)
{
    List<VoiceProfile> voiceProfiles = new List<VoiceProfile>();
    using (var client = new VoiceProfileClient(config))
    {
        foreach (string name in profileNames)
        {
            using (var audioInput = AudioConfig.FromDefaultMicrophoneInput())
            {
                var profile = await client.CreateProfileAsync(VoiceProfileType.TextIndependentIdentification, "en-us");
                Console.WriteLine($"Creating voice profile for {name}.");
                profileMapping.Add(profile.Id, name);

                VoiceProfileEnrollmentResult result = null;
                while (result is null || result.RemainingEnrollmentsSpeechLength > TimeSpan.Zero)
                {
                    Console.WriteLine($"Continue speaking to add to the profile enrollment sample for {name}.");
                    result = await client.EnrollProfileAsync(profile, audioInput);
                    Console.WriteLine($"Remaining enrollment audio time needed: {result.RemainingEnrollmentsSpeechLength}");
                    Console.WriteLine("");
                }
                voiceProfiles.Add(profile);
            }
        }
    }
    return voiceProfiles;
}
```

Cree la siguiente función `SpeakerIdentification` para enviar una solicitud de identificación. La diferencia principal de esta función con respecto a una solicitud de **verificación del hablante** es el uso de `SpeakerIdentificationModel.FromProfiles()`, que acepta una lista de objetos `VoiceProfile`. 

```csharp
public static async Task SpeakerIdentification(SpeechConfig config, List<VoiceProfile> voiceProfiles, Dictionary<string, string> profileMapping) 
{
    var speakerRecognizer = new SpeakerRecognizer(config, AudioConfig.FromDefaultMicrophoneInput());
    var model = SpeakerIdentificationModel.FromProfiles(voiceProfiles);

    Console.WriteLine("Speak some text to identify who it is from your list of enrolled speakers.");
    var result = await speakerRecognizer.RecognizeOnceAsync(model);
    Console.WriteLine($"The most similiar voice profile is {profileMapping[result.ProfileId]} with similiarity score {result.Score}");
}
```

Cambie la función `Main()` por lo siguiente. Se crea una lista de cadenas `profileNames`, que debe pasar a la función `IdentificationEnroll()`. Se le pedirá que cree un nuevo perfil de voz para cada nombre de la lista, de modo que puede agregar más nombres para crear perfiles adicionales para amigos o compañeros.

```csharp
static async Task Main(string[] args)
{
    // replace with your own subscription key 
    string subscriptionKey = "YourSubscriptionKey";
    string region = "westus";
    var config = SpeechConfig.FromSubscription(subscriptionKey, region);

    // persist profileMapping if you want to store a record of who the profile is
    var profileMapping = new Dictionary<string, string>();
    var profileNames = new List<string>() { "Your name", "A friend's name" };
    
    var enrolledProfiles = await IdentificationEnroll(config, profileNames, profileMapping);
    await SpeakerIdentification(config, enrolledProfiles, profileMapping);

    foreach (var profile in enrolledProfiles)
    {
        profile.Dispose();
    }
    Console.ReadLine();
}
```

Ejecute el script y se le pedirá que hable para inscribir las muestras de voz para el primer perfil. Una vez completada la inscripción, se le pedirá que repita este proceso para cada nombre de la lista `profileNames`. Una vez finalizada la inscripción, se le pedirá que **cualquier** persona hable y el servicio intentará identificarla entre los perfiles de voz inscritos.

Este ejemplo devuelve solo la coincidencia más cercana y su puntuación de similitud, pero puede obtener la respuesta completa que incluye las cinco mejores puntuaciones de similitud agregando `string json = result.Properties.GetProperty(PropertyId.SpeechServiceResponse_JsonResult)` a la función `SpeakerIdentification`.

## <a name="changing-audio-input-type"></a>Cambio del tipo de entrada de audio

En los ejemplos de este artículo se usa el micrófono del dispositivo predeterminado como entrada para las muestras de audio. Sin embargo, en escenarios en los que es necesario usar archivos de audio en lugar de entradas de micrófono, basta con cambiar cualquier instancia de `AudioConfig.FromDefaultMicrophoneInput()` a `AudioConfig.FromWavFileInput(path/to/your/file.wav)` para cambiar a una entrada de archivo. También puede tener entradas mixtas, si usa un micrófono para la inscripción y archivos para la comprobación, por ejemplo.

## <a name="deleting-voice-profile-enrollments"></a>Eliminación de inscripciones de perfiles de voz

Para eliminar un perfil inscrito, utilice la función `DeleteProfileAsync()` en el objeto `VoiceProfileClient`. En la función de ejemplo siguiente se muestra cómo eliminar un perfil de voz de un identificador de perfil de voz conocido.

```csharp
public static async Task DeleteProfile(SpeechConfig config, string profileId) 
{
    using (var client = new VoiceProfileClient(config))
    {
        var profile = new VoiceProfile(profileId);
        await client.DeleteProfileAsync(profile);
    }
}
```

## <a name="next-steps"></a>Pasos siguientes

* Consulte la [documentación de referencia](https://docs.microsoft.com/rest/api/speakerrecognition/) de Speaker Recognition para obtener más información sobre las clases y funciones.

* Consulte los ejemplos de [C#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnet/speaker-recognition) y [C++](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows/speaker-recognition) en GitHub.

