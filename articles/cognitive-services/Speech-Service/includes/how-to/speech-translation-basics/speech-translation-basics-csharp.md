---
author: trevorbye
ms.service: cognitive-services
ms.topic: include
ms.date: 04/13/2020
ms.author: trbye
ms.custom: devx-track-csharp
ms.openlocfilehash: 1042383e285780f5ee9297b1c87615bbdd2fe1b6
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88925271"
---
## <a name="prerequisites"></a>Requisitos previos

En este artículo se da por sentado que tiene una cuenta de Azure y una suscripción al servicio de voz. Si no dispone de una cuenta y una suscripción, [pruebe el servicio de voz de forma gratuita](../../../get-started.md).

## <a name="install-the-speech-sdk"></a>Instalación de Speech SDK

En primer lugar, deberá instalar Speech SDK. Dependiendo de la plataforma, siga las instrucciones de la sección <a href="https://docs.microsoft.com/en-us/azure/cognitive-services/speech-service/speech-sdk#get-the-speech-sdk" target="_blank">Obtención del SDK de Voz<span class="docon docon-navigate-external x-hidden-focus"></span></a> del artículo _Acerca del SDK de Voz_.

## <a name="import-dependencies"></a>Dependencias de importación

Para ejecutar los ejemplos de este artículo, incluya las siguientes instrucciones `using` al principio del archivo *Program.cs*.

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Text;
using System.Threading.Tasks;
using Microsoft.CognitiveServices.Speech;
using Microsoft.CognitiveServices.Speech.Audio;
using Microsoft.CognitiveServices.Speech.Translation;
```

## <a name="sensitive-data-and-environment-variables"></a>Datos confidenciales y variables de entorno

El código fuente de ejemplo de este artículo depende de las variables de entorno para almacenar datos confidenciales, como la región y la clave de suscripción del recurso de Voz. La clase `Program` contiene dos valores `static readonly string` que se asignan a las variables de entorno de las máquinas host, es decir, `SPEECH__SUBSCRIPTION__KEY` y `SPEECH__SERVICE__REGION`. Ambos campos se encuentran en el ámbito de clase, lo que les permite tener acceso a ellos en los cuerpos de método de la clase. Para más información sobre las variables de entorno, consulte [Configuración de las variables de entorno y de la aplicación](../../../../cognitive-services-security.md#environment-variables-and-application-configuration).

```csharp
public class Program
{
    static readonly string SPEECH__SUBSCRIPTION__KEY =
        Environment.GetEnvironmentVariable(nameof(SPEECH__SUBSCRIPTION__KEY));
    
    static readonly string SPEECH__SERVICE__REGION =
        Environment.GetEnvironmentVariable(nameof(SPEECH__SERVICE__REGION));

    static Task Main() => Task.CompletedTask;
}
```

## <a name="create-a-speech-translation-configuration"></a>Creación de una configuración de traducción de voz

Para llamar al servicio de voz con Speech SDK, debe crear un elemento [`SpeechTranslationConfig`][config]. Esta clase incluye información sobre la suscripción, como la clave, la región asociada, el punto de conexión, el host o el token de autorización.

> [!TIP]
> Debe crear siempre una configuración, independientemente de si va a realizar reconocimiento de voz, síntesis de voz, traducción o reconocimiento de intenciones.

Existen diversas maneras para inicializar un elemento [`SpeechTranslationConfig`][config]:

* Con una suscripción: pase una clave y la región asociada.
* Con un punto de conexión: pase un punto de conexión del servicio de voz. La clave y el token de autorización son opcionales.
* Con un host: pase una dirección de host. La clave y el token de autorización son opcionales.
* Con un token de autorización: pase el token de autorización y la región asociada.

Vamos a echar un vistazo al procedimiento de creación de un elemento [`SpeechTranslationConfig`][config] con una clave y una región. Consulte la página de [soporte por regiones](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#speech-sdk) para buscar el identificador de región.

```csharp
public class Program
{
    static readonly string SPEECH__SUBSCRIPTION__KEY =
        Environment.GetEnvironmentVariable(nameof(SPEECH__SUBSCRIPTION__KEY));
    
    static readonly string SPEECH__SERVICE__REGION =
        Environment.GetEnvironmentVariable(nameof(SPEECH__SERVICE__REGION));

    static Task Main() => TranslateSpeechAsync();

    static async Task TranslateSpeechAsync()
    {
        var translationConfig =
            SpeechTranslationConfig.FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    }
}
```

## <a name="change-source-language"></a>Cambio del idioma de origen

Una tarea común en la traducción de voz es especificar el idioma de entrada (u origen). Echemos un vistazo a cómo se cambiaría el idioma de entrada a italiano. En el código, interactúe con la instancia [`SpeechTranslationConfig`][config], asignándola a la propiedad `SpeechRecognitionLanguage`.

```csharp
static async Task TranslateSpeechAsync()
{
    var translationConfig =
        SpeechTranslationConfig.FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    // Source (input) language
    translationConfig.SpeechRecognitionLanguage = "it-IT";
}
```

La propiedad [`SpeechRecognitionLanguage`][recognitionlang] espera una cadena con formato de configuración regional de idioma. En la columna **Locale**, puede proporcionar cualquier valor de la lista de idiomas o configuraciones regionales [compatibles](../../../language-support.md).

## <a name="add-translation-language"></a>Incorporación del idioma de traducción

Otra tarea común de la traducción de voz es especificar los idiomas de traducción de destino; se requiere al menos uno, pero se admiten varios. En el fragmento de código siguiente, se han especificado el francés y el alemán como destinos del idioma de traducción.

```csharp
static async Task TranslateSpeechAsync()
{
    var translationConfig =
        SpeechTranslationConfig.FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    translationConfig.SpeechRecognitionLanguage = "it-IT";
    
    // Translate to languages. See, https://aka.ms/speech/sttt-languages
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("de");
}
```

Con cada llamada a [`AddTargetLanguage`][addlang], se especifica un nuevo idioma de traducción de destino. En otras palabras, cuando se reconoce la voz del lenguaje de origen, cada traducción de destino está disponible como parte de la operación de traducción resultante.

## <a name="initialize-a-translation-recognizer"></a>Inicialización de un reconocedor de traducción

Una vez creado un elemento [`SpeechTranslationConfig`][config], el paso siguiente consiste en inicializar un elemento [`TranslationRecognizer`][recognizer]. Al inicializar un elemento [`TranslationRecognizer`][recognizer], deberá pasar el elemento `translationConfig`. El objeto de configuración proporciona las credenciales que necesita el servicio de voz para validar la solicitud.

Si va a realizar un reconocimiento de voz con el micrófono predeterminado del dispositivo, el elemento [`TranslationRecognizer`][recognizer] debería ser similar a:

```csharp
static async Task TranslateSpeechAsync()
{
    var translationConfig =
        SpeechTranslationConfig.FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);

    var fromLanguage = "en-US";
    var toLanguages = new List<string> { "it", "fr", "de" };
    translationConfig.SpeechRecognitionLanguage = fromLanguage;
    toLanguages.ForEach(translationConfig.AddTargetLanguage);

    using var recognizer = new TranslationRecognizer(translationConfig);
}
```

Si desea especificar el dispositivo de entrada de audio, deberá crear un elemento [`AudioConfig`][audioconfig] y proporcionar el parámetro `audioConfig` al inicializar el elemento [`TranslationRecognizer`][recognizer].

> [!TIP]
> [Obtenga información sobre cómo obtener el identificador de dispositivo del dispositivo de entrada de audio](../../../how-to-select-audio-input-devices.md).

En primer lugar, haga referencia al objeto `AudioConfig` como se indica a continuación:

```csharp
static async Task TranslateSpeechAsync()
{
    var translationConfig =
        SpeechTranslationConfig.FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    var fromLanguage = "en-US";
    var toLanguages = new List<string> { "it", "fr", "de" };
    translationConfig.SpeechRecognitionLanguage = fromLanguage;
    toLanguages.ForEach(translationConfig.AddTargetLanguage);

    using var audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using var recognizer = new TranslationRecognizer(translationConfig, audioConfig);
}
```

Si desea proporcionar un archivo de audio en lugar de usar un micrófono, deberá proporcionar un elemento `audioConfig`. Sin embargo, cuando se crea un elemento [`AudioConfig`][audioconfig], en lugar de llamar a `FromDefaultMicrophoneInput`, llamará a `FromWavFileInput` y pasará el parámetro `filename`.

```csharp
static async Task TranslateSpeechAsync()
{
    var translationConfig =
        SpeechTranslationConfig.FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    var fromLanguage = "en-US";
    var toLanguages = new List<string> { "it", "fr", "de" };
    translationConfig.SpeechRecognitionLanguage = fromLanguage;
    toLanguages.ForEach(translationConfig.AddTargetLanguage);

    using var audioConfig = AudioConfig.FromWavFileInput("YourAudioFile.wav");
    using var recognizer = new TranslationRecognizer(translationConfig, audioConfig);
}
```

## <a name="translate-speech"></a>Traducir voz

Para traducir la voz, el SDK de Voz depende de un micrófono o en una entrada de archivo de audio. El reconocimiento de voz tiene lugar antes de la traducción de voz. Cuando todos los objetos se hayan inicializado, llame a la función recognize-once y obtenga el resultado.

```csharp
static async Task TranslateSpeechAsync()
{
    var translationConfig =
        SpeechTranslationConfig.FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    var fromLanguage = "en-US";
    var toLanguages = new List<string> { "it", "fr", "de" };
    translationConfig.SpeechRecognitionLanguage = fromLanguage;
    toLanguages.ForEach(translationConfig.AddTargetLanguage);

    using var recognizer = new TranslationRecognizer(translationConfig);

    Console.Write($"Say something in '{fromLanguage}' and ");
    Console.WriteLine($"we'll translate into '{string.Join("', '", toLanguages)}'.\n");
    
    var result = await recognizer.RecognizeOnceAsync();
    if (result.Reason == ResultReason.TranslatedSpeech)
    {
        Console.WriteLine($"Recognized: \"{result.Text}\":");
        foreach (var (language, translation) in result.Translations)
        {
            Console.WriteLine($"Translated into '{language}': {translation}");
        }
    }
}
```

Para más información acerca de la conversión de voz a texto, consulte [Aspectos básicos del reconocimiento de voz](../../../speech-to-text-basics.md).

## <a name="synthesize-translations"></a>Síntesis de traducciones

Después de un reconocimiento de voz y una traducción correctos, el resultado contiene todas las traducciones de un diccionario. La clave del diccionario [`Translations`][translations] es el idioma de traducción de destino y el valor es el texto traducido. La voz reconocida se puede traducir y sintetizar en otro idioma (de voz a voz).

### <a name="event-based-synthesis"></a>Síntesis basada en eventos

El objeto `TranslationRecognizer` expone un evento `Synthesizing`. El evento se activa varias veces y proporciona un mecanismo para recuperar el audio sintetizado del resultado del reconocimiento de la traducción. Si va a traducir a varios idiomas, consulte [Síntesis manual](#manual-synthesis). Especifique la voz de síntesis asignando un [`VoiceName`][voicename] y proporcione un controlador de eventos para el evento de `Synthesizing` y obtenga el audio. En el ejemplo siguiente se guarda el audio traducido como un archivo *.wav*.

> [!IMPORTANT]
> La síntesis basada en eventos solo funciona con una sola traducción, **no** agrega varios idiomas de traducción de destino. Además, el objeto [`VoiceName`][voicename] debe ser el mismo idioma que el idioma de traducción de destino; por ejemplo, `"de"` podría asignarse a `"de-DE-Hedda"`.

```csharp
static async Task TranslateSpeechAsync()
{
    var translationConfig =
        SpeechTranslationConfig.FromSubscription(SPEECH__SUBSCRIPTION__KEY, SPEECH__SERVICE__REGION);
    
    var fromLanguage = "en-US";
    var toLanguage = "de";
    translationConfig.SpeechRecognitionLanguage = fromLanguage;
    translationConfig.AddTargetLanguage(toLanguage);

    // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
    translationConfig.VoiceName = "de-DE-Hedda";

    using var recognizer = new TranslationRecognizer(translationConfig);

    recognizer.Synthesizing += (_, e) =>
    {
        var audio = e.Result.GetAudio();
        Console.WriteLine($"Audio synthesized: {audio.Length:#,0} byte(s) {(audio.Length == 0 ? "(Complete)" : "")}");

        if (audio.Length > 0)
        {
            File.WriteAllBytes("YourAudioFile.wav", audio);
        }
    };

    Console.Write($"Say something in '{fromLanguage}' and ");
    Console.WriteLine($"we'll translate into '{toLanguage}'.\n");

    var result = await recognizer.RecognizeOnceAsync();
    if (result.Reason == ResultReason.TranslatedSpeech)
    {
        Console.WriteLine($"Recognized: \"{result.Text}\"");
        Console.WriteLine($"Translated into '{toLanguage}': {result.Translations[toLanguage]}");
    }
}
```

### <a name="manual-synthesis"></a>Síntesis manual

El diccionario [`Translations`][translations] se puede usar para sintetizar el audio desde el texto de traducción. Recorra en iteración cada traducción y sintetice la traducción. Al crear una instancia de `SpeechSynthesizer`, el objeto `SpeechConfig` debe tener su propiedad [`SpeechSynthesisVoiceName`][speechsynthesisvoicename] establecida en la voz deseada. El siguiente ejemplo se traduce a cinco idiomas y, a continuación, cada conversión se sintetiza en un archivo de audio en el idioma neuronal correspondiente.

```csharp
static async Task TranslateSpeechAsync()
{
    var translationConfig =
        SpeechTranslationConfig.FromSubscription(SPEECH__SERVICE__KEY, SPEECH__SERVICE__REGION);

    var fromLanguage = "en-US";
    var toLanguages = new List<string> { "de", "en", "it", "pt", "zh-Hans" };
    translationConfig.SpeechRecognitionLanguage = fromLanguage;
    toLanguages.ForEach(translationConfig.AddTargetLanguage);

    using var recognizer = new TranslationRecognizer(translationConfig);

    Console.Write($"Say something in '{fromLanguage}' and ");
    Console.WriteLine($"we'll translate into '{string.Join("', '", toLanguages)}'.\n");

    var result = await recognizer.RecognizeOnceAsync();
    if (result.Reason == ResultReason.TranslatedSpeech)
    {
        // See: https://aka.ms/speech/sdkregion#standard-and-neural-voices
        var languageToVoiceMap = new Dictionary<string, string>
        {
            ["de"] = "de-DE-KatjaNeural",
            ["en"] = "en-US-AriaNeural",
            ["it"] = "it-IT-ElsaNeural",
            ["pt"] = "pt-BR-FranciscaNeural",
            ["zh-Hans"] = "zh-CN-XiaoxiaoNeural"
        };

        Console.WriteLine($"Recognized: \"{result.Text}\"");

        foreach (var (language, translation) in result.Translations)
        {
            Console.WriteLine($"Translated into '{language}': {translation}");

            var speechConfig =
                SpeechConfig.FromSubscription(
                    SPEECH__SERVICE__KEY, SPEECH__SERVICE__REGION);
            speechConfig.SpeechSynthesisVoiceName = languageToVoiceMap[language];

            using var audioConfig = AudioConfig.FromWavFileOutput($"{language}-translation.wav");
            using var synthesizer = new SpeechSynthesizer(speechConfig, audioConfig);
            
            await synthesizer.SpeakTextAsync(translation);
        }
    }
}
```

Para más información acerca de la síntesis de voz, consulte [Aspectos básicos de la síntesis de voz](../../../text-to-speech-basics.md).

[config]: https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechtranslationconfig?view=azure-dotnet
[audioconfig]: https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.audio.audioconfig?view=azure-dotnet
[recognizer]: https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.translation.translationrecognizer?view=azure-dotnet
[recognitionlang]: https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.speechrecognitionlanguage?view=azure-dotnet
[addlang]: https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechtranslationconfig.addtargetlanguage?view=azure-dotnet
[translations]: https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.translation.translationrecognitionresult.translations?view=azure-dotnet
[voicename]: https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechtranslationconfig.voicename?view=azure-dotnet
[speechsynthesisvoicename]: https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.speechsynthesisvoicename?view=azure-dotnet
