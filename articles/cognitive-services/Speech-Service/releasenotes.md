---
title: 'Notas de la versión: Servicio de Voz'
titleSuffix: Azure Cognitive Services
description: Un registro de las versiones de actualización de características, mejoras, correcciones de errores y problemas conocidos del servicio de voz.
services: cognitive-services
author: oliversc
manager: jhakulin
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/17/2020
ms.author: oliversc
ms.custom: seodec18
ms.openlocfilehash: 94947499452c7f1b8515fee56996b13120232f34
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89462384"
---
# <a name="speech-service-release-notes"></a>Notas de la versión del servicio Voz

## <a name="text-to-speech-2020-august-release"></a>Texto a voz, versión de agosto de 2020

### <a name="new-features"></a>Nuevas características

* **TTS neuronal: nuevo estilo del habla para la voz `en-US` de Aria**. AriaNeural puede parecer un locutor al leer las noticias. El estilo "newscast-formal" suena más serio, mientras que el estilo "newscast-casual" es más flexible e informal. Consulte [cómo usar los estilos del habla en SSML](speech-synthesis-markup.md).

* **Voz personalizada: se presenta una nueva característica para comprobar automáticamente la calidad de los datos de entrenamiento**. Al cargar los datos, el sistema examinará diversos aspectos de los datos de audio y transcripción, y corregirá o filtrará los problemas automáticamente para mejorar la calidad del modelo de voz. Abarca el volumen del audio, el nivel de ruido, la precisión de pronunciación de la voz, la alineación de la voz con el texto normalizado, el silencio en el audio, además del formato de audio y de script. 

* **Creación de contenido de audio: un conjunto de nuevas características para habilitar capacidades de administración de audio y de ajuste de voz más eficaces**.

    * Pronunciación: la característica de optimización de la pronunciación se actualiza con el conjunto de fonemas más reciente. Puede seleccionar el elemento de fonema correcto en la biblioteca y refinar la pronunciación de las palabras que ha seleccionado. 

    * Descargar: La característica "Descargar"/"Exportar" de audio se ha mejorado para admitir la generación de audio por párrafo. Puede editar el contenido en el mismo archivo o SSML, mientras genera varias salidas de audio. La estructura de archivos de "Descargar" también se ha refinado. Ahora, puede colocar fácilmente todos los audios en una carpeta. 

    * Estado de la tarea: Se ha mejorado la experiencia de exportación de varios archivos. Cuando se exportaban varios archivos en el pasado, si se producía un error en uno de los archivos, se producía un error en toda la tarea. Pero ahora todos los demás archivos se exportarán correctamente. El informe de tareas se enriquece con información más detallada y estructurada. Ahora puede comprobar los registros de todos los archivos y oraciones con errores con el informe. 

    * Documentación de SSML: vinculado a un documento SSML para ayudarle a comprobar las reglas sobre cómo usar todas las características de optimización.

* **La API de lista de voces se ha actualizado para incluir un nombre para mostrar del usuario que es descriptivo y los estilos del habla admitidos para las voces neuronales**.

### <a name="general-tts-voice-quality-improvements"></a>Mejoras generales de calidad de voz TTS

* Se ha reducido el porcentaje de errores de pronunciación de nivel de palabra para `ru-RU` (en un 56 %) y para `sv-SE` (en un 49 %).

* Se ha mejorado la lectura de palabras polifónicas en voces neuronales `en-US` en un 40 %. Entre los ejemplos de palabras polifónicas se incluyen "read", "live", "content", "record", "object", etc. 

* Se ha mejorado la naturalidad del tono de la pregunta en `fr-FR`. MOS (puntuación de opinión media): +0,28.

* Se han actualizado los vocoders para las siguientes voces, con mejoras de fidelidad y velocidad de rendimiento general en un 40 %.

    | Configuración regional | Voz |
    |---|---|    
    | `en-GB` | Mia |
    | `es-MX` | Dalia |
    | `fr-CA` | Sylvie |
    | `fr-FR` | Denise |
    | `ja-JP` | Nanami |
    | `ko-KR` | Sun-Hi |

### <a name="bug-fixes"></a>Corrección de errores

* Se han corregido varios errores de la herramienta Creación de contenido de audio 
    * Corrección del problema con la actualización automática. 
    * Corrección de los problemas con los estilos de voz en zh-CN de la región del Sudeste Asiático.
    * Corrección del problema de estabilidad, incluido un error de exportación con la etiqueta "break" y errores en signos de puntuación.    

## <a name="new-speech-to-text-locales-2020-august-release"></a>Nuevas configuraciones regionales de conversión de voz en texto: versión de agosto de 2020
La conversión de voz en texto ha publicado 26 nuevas configuraciones regionales en agosto: 2 idiomas europeos `cs-CZ` y `hu-HU`, 5 configuraciones regionales en inglés y 19 configuraciones regionales en español que cubren la mayoría de los países de Sudamérica. A continuación, se muestra una lista de las nuevas configuraciones regionales. Consulte la lista completa de idiomas [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support).

| Configuración regional  | Idioma                          |
|---------|-----------------------------------|
| `cs-CZ` | Checo (República Checa)            | 
| `en-HK` | Inglés (Hong Kong)               | 
| `en-IE` | Inglés (Irlanda)                 | 
| `en-PH` | Inglés (Filipinas)             | 
| `en-SG` | Inglés (Singapur)               | 
| `en-ZA` | Inglés (Sudáfrica)            | 
| `es-AR` | Español (Argentina)               | 
| `es-BO` | Español (Bolivia)                 | 
| `es-CL` | Español (Chile)                   | 
| `es-CO` | Español (Colombia)                | 
| `es-CR` | Español (Costa Rica)              | 
| `es-CU` | Español (Cuba)                    | 
| `es-DO` | Español (República Dominicana)      | 
| `es-EC` | Español (Ecuador)                 | 
| `es-GT` | Español (Guatemala)               | 
| `es-HN` | Español (Honduras)                | 
| `es-NI` | Español (Nicaragua)               | 
| `es-PA` | Español (Panamá)                  | 
| `es-PE` | Español (Perú)                    | 
| `es-PR` | Español (Puerto Rico)             | 
| `es-PY` | Español (Paraguay)                | 
| `es-SV` | Español (El Salvador)             | 
| `es-US` | Español (EE. UU.)                     | 
| `es-UY` | Español (Uruguay)                 | 
| `es-VE` | Español (Venezuela)               | 
| `hu-HU` | Húngaro (Hungría)               | 


## <a name="speech-sdk-1130-2020-july-release"></a>SDK de voz 1.13.0: versión de julio de 2020

**Nota**: El SDK de voz en Windows depende de Microsoft Visual C++ Redistributable compartido para Visual Studio 2015, 2017 y 2019. Descárguelo e instálelo [aquí](https://support.microsoft.com/help/2977003/the-latest-supported-visual-c-downloads).

**Nuevas características:**
- **C#** : se ha agregado compatibilidad para la transcripción de conversaciones asincrónicas. Consulte la documentación [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-async-conversation-transcription).  
- **JavaScript**: se ha agregado compatibilidad con Speaker Recognition tanto para el [explorador](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/browser/speaker-recognition) como para [node.js](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/speaker-recognition).
- **JavaScript**: se ha agregado compatibilidad para la detección automática del idioma o el identificador del idioma. Consulte la documentación [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-automatic-language-detection?pivots=programming-language-javascript).
- **Objective-C**: Se ha agregado compatibilidad para la [conversación entre varios dispositivos](https://docs.microsoft.com/azure/cognitive-services/speech-service/multi-device-conversation) y la [transcripción de conversaciones](https://docs.microsoft.com/azure/cognitive-services/speech-service/conversation-transcription). 
- **Python**: se ha agregado compatibilidad con audio comprimido para Python en Windows y Linux. Consulte la documentación [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-use-codec-compressed-audio-input-streams). 

**Correcciones de errores**
- **Todos**: se ha corregido un problema que provocaba que KeywordRecognizer no hiciera avanzar los flujos después de un reconocimiento.
- **Todos**: se ha corregido un problema que provocaba que el flujo obtenido de KeywordRecognitionResult no incluyera la palabra clave.
- **Todos**: se ha corregido un problema por el que SendMessageAsync no enviaba realmente el mensaje a través de la conexión después de que hubiese terminado la espera de los usuarios.
- **Todos**: Se ha corregido un bloqueo en las API de Speaker Recognition cuando los usuarios llamaban al método VoiceProfileClient::SpeakerRecEnrollProfileAsync varias veces y no esperaban a que las llamadas finalizaran.
- **Todos**: se ha corregido la habilitación del registro de archivos en las clases VoiceProfileClient y SpeakerRecognizer.
- **JavaScript**: se ha corregido un [problema](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/74) con la limitación al minimizar el explorador.
- **JavaScript**: se ha corregido un [problema](https://github.com/microsoft/cognitive-services-speech-sdk-js/issues/78) con la fuga de memoria en los flujos.
- **JavaScript**: se ha agregado almacenamiento en caché para las respuestas de OCSP de NodeJS.
- **Java**: se ha corregido un problema que provocaba que los campos BigInteger devolvieran siempre 0.
- **iOS**: Se ha corregido un [problema](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/702) con la publicación de aplicaciones basadas en el SDK de Voz en la iOS App Store.

**Muestras**
- **C++** : se ha agregado código de ejemplo para Speaker Recognition [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/cpp/windows/console/samples/speaker_recognition_samples.cpp).

**Pruebas reducidas ante la COVID-19:** Debido al trabajo de forma remota en las últimas semanas, no pudimos realizar tantas pruebas manuales de comprobación de como normalmente hacemos. No hemos hecho ningún cambio que creemos que pueda haber generado algún error y se han superado todas las pruebas automatizadas. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.

## <a name="text-to-speech-2020-july-release"></a>Texto a voz 2020-versión de julio

### <a name="new-features"></a>Nuevas características

* **TTS neuronal, 15 nuevas voces neuronales**: Las nuevas voces agregadas a la cartera de TTS neuronal son Salma en `ar-EG` árabe (Egipto), Zariyah en `ar-SA` árabe (Arabia Saudí), Alba en `ca-ES` catalán (España), Christel en `da-DK` danés (Dinamarca), Neerja en `es-IN` inglés (India), Noora en `fi-FI` finés (Finlandia), Swara en `hi-IN` hindi (India), Colette en `nl-NL` holandés (Países Bajos), Zofia en `pl-PL` polaco (Polonia), Fernanda en `pt-PT` portugués (Portugal), Dariya en `ru-RU` ruso (Rusia), Hillevi en `sv-SE` sueco (Suecia), Achara en `th-TH` tailandés (Tailandia), HiuGaai en `zh-HK` chino (cantonés, tradicional) y HsiaoYu en `zh-TW` chino (mandarín, Taiwán). Comprobación de los [idiomas compatibles](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#neural-voices).  

* **Voz personalizada, pruebas de voz simplificadas con el flujo de aprendizaje para simplificar la experiencia del usuario**: Con la nueva característica de prueba, cada voz se probará automáticamente con un conjunto de pruebas predefinido optimizado para cada idioma con el fin de cubrir los escenarios generales y de los asistentes de voz. Estos conjuntos de pruebas se seleccionan y se prueban cuidadosamente para incluir los casos de uso típicos y fonemas en el idioma. Además, los usuarios todavía pueden seleccionar cargar sus propios scripts de prueba al entrenar un modelo.

* **Creación de contenido de audio: se publica un conjunto de nuevas características para habilitar capacidades de administración de audio y de ajuste de voz más eficaces**.

    * `Pitch`, `rate`y `volume` se han mejorado para admitir la optimización con un valor predefinido, como lento, medio y rápido. Ahora es sencillo para los usuarios seleccionar un valor "constante" para su edición de audio.

    ![Ajuste de audio](media/release-notes/audio-tuning.png)

    * Los usuarios ahora pueden revisar el `Audio history` para su archivo de trabajo. Con esta característica, los usuarios pueden realizar fácilmente un seguimiento de todo el audio generado relacionado con un archivo de trabajo. Pueden comprobar la versión del historial y comparar la calidad durante la optimización al mismo tiempo. 

    ![Historial de audio](media/release-notes/audio-history.png)

    * La característica `Clear` ahora es más flexible. Los usuarios pueden borrar un parámetro de ajuste específico manteniendo otros parámetros disponibles para el contenido seleccionado.  

    * Se ha agregado un vídeo tutorial en la [página de aterrizaje](https://speech.microsoft.com/audiocontentcreation) para ayudar a los usuarios a empezar a trabajar rápidamente con la optimización de voz y el ajuste de audio de TTS. 

### <a name="general-tts-voice-quality-improvements"></a>Mejoras generales de calidad de voz TTS

* Se han mejorado los vocoder de TTS en para mayor fidelidad y menor latencia.

    * Se ha actualizado Elsa en `it-IT` a un nuevo vocoder que logró + 0,464 CMOS (puntuación aproximada de opinión) en la calidad de la voz, un 40% más rápido en la síntesis y una reducción del 30% en la latencia del primer byte. 
    * Se ha actualizado Xiaoxiao en `zh-CN` al nuevo vocoder que logró + 0148 CMOS para el dominio general, + 0,348 para el estilo newscast y + 0,195 para el estilo lírico. 

* Se han actualizado los modelos de voz `de-DE` y `ja-JP` para que la salida de TTS sea más natural.
    
    * Katja actualizado en `de-DE` con el método de modelado de prosodia más reciente, la ganancia de MOS (puntuación media de la opinión) es de + 0,13. 
    * Nanami actualizado en `ja-JP` con un nuevo modelo de énfasis de brea prosodia, la ganancia de MOS (puntuación media de la opinión) es de + 0,19;  

* Precisión de Nivel de pronunciación de palabra mejorada en 5 idiomas.

    | Idioma | Reducción de errores de Pronunciación |
    |---|---|
    | `en-GB` | 51% |
    | `ko-KR` | 17 % |
    | `pt-BR` | 39 % |
    | `pt-PT` | 77% |
    | `id-ID` | 46% |

### <a name="bug-fixes"></a>Corrección de errores

* Lectura de moneda
    * Se corrigió el problema con la lectura de moneda para `es-ES` y `es-MX`
     
    | Idioma | Entrada | Lectura después de la mejora |
    |---|---|---|
    | `es-MX` | 1,58 USD | un peso cincuenta y ocho centavos |
    | `es-ES` | 1,58 USD | un dólar con cincuenta y ocho centavos |

    * Compatibilidad con la moneda negativa (por ejemplo, "-€325") en las siguientes configuraciones regionales: `en-US`, `en-GB`, `fr-FR`, `it-IT`, `en-AU`,`en-CA`.

* Mejora de la lectura de direcciones en `pt-PT`.
* Se han corregido problemas de pronunciación de Natasha (`en-AU`) y Libby (`en-UK`) en la palabra en inglés "for" y "four".  
* Se han corregido errores en la herramienta de creación de contenido de audio.
    * Se corrigió la pausa adicional e inesperada después del segundo párrafo.  
    * La característica 'Sin interrupción' se vuelve a agregar desde un error de regresión. 
    * Se corrigió el problema de actualización aleatoria de Speech Studio.  

### <a name="samplessdk"></a>Muestras/SDK

* JavaScript: Correcciones para el problema de reproducción en Firefox y Safari, tanto en macOS como en iOS. 

## <a name="speech-sdk-1121-2020-june-release"></a>SDK de voz 1.12.1: Versión de junio de 2020
**CLI de voz (también conocida como SPX)**
-   Se han agregado las características de búsqueda en la ayuda en la CLI:
    -   `spx help find --text TEXT`
    -   `spx help find --topic NAME`
-   Se ha actualizado para que funcione con la versión 3.0 de las API Batch y Custom Speech:
    -   `spx help batch examples`
    -   `spx help csr examples`

**Nuevas características:**
-   **C\# y C++** : Versión preliminar de Speaker Recognition: Esta característica permite la identificación del hablante (¿quién habla?) y la verificación del hablante (¿es quien dice ser?). Empiece con la [introducción](https://docs.microsoft.com/azure/cognitive-services/Speech-Service/speaker-recognition-overview), lea el [artículo sobre los aspectos básicos de Speaker Recognition](https://docs.microsoft.com/azure/cognitive-services/speech-service/speaker-recognition-basics) o los [documentos de referencia de las API](https://docs.microsoft.com/rest/api/speakerrecognition/).

**Correcciones de errores**
-   **C\# y C++** : la grabación del micrófono fijo no funcionaba en la versión 1.12 de Speaker Recognition.
-   **JavaScript**: Correcciones para la conversión de texto a voz en Firefox y Safari, tanto en macOS como en iOS.
-   Corrección del bloqueo de la violación del acceso al comprobador de aplicaciones Windows en la transcripción de conversaciones cuando se usa el flujo de 8 canales.
-   Corrección del bloqueo de la violación del acceso al comprobador de aplicaciones Windows en la traducción de conversaciones entre varios dispositivos.

**Muestras**
-   **C#** : [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/dotnet/speaker-recognition) del reconocimiento del hablante.
-   **C++** : [código de ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp/windows/speaker-recognition) del reconocimiento del hablante.
-   **Java**: [ejemplo de código](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android/intent-recognition) para el reconocimiento de la intención en Android. 

**Pruebas reducidas ante la COVID-19:** Debido al trabajo de forma remota en las últimas semanas, no pudimos realizar tantas pruebas manuales de comprobación de como normalmente hacemos. No hemos hecho ningún cambio que creemos que pueda haber generado algún error y se han superado todas las pruebas automatizadas. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.


## <a name="speech-sdk-1120-2020-may-release"></a>SDK de voz 1.12.0: Versión de mayo de 2020
**CLI de voz (también conocida como SPX)**
- **SPX** es una nueva herramienta de línea de comandos que permite realizar el reconocimiento, la síntesis, la traducción, la transcripción por lotes y la administración de Custom Speech desde la línea de comandos. Úselo para probar el servicio de voz o para crear scripts de las tareas del servicio de voz que debe realizar. Descargue la herramienta y lea la documentación [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/spx-overview).

**Nuevas características:**
- **Go**: Nueva compatibilidad con el lenguaje Go para el [reconocimiento de voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/speech-to-text-from-microphone?pivots=programming-language-go) y el [asistente de Voz personalizada](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/voice-assistants?pivots=programming-language-go). Configure el entorno de desarrollo [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/setup-platform?pivots=programming-language-go). Para ver el código de ejemplo, consulte la sección Ejemplos más adelante. 
- **JavaScript**: Compatibilidad del explorador agregada para la conversión de texto a voz. Consulte la documentación [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/text-to-speech-audio-file?pivots=programming-language-JavaScript).
- **C++, C#, Java**: Nuevo objeto `KeywordRecognizer` y las API compatibles con las plataformas Windows, Android, Linux e iOS. Consulte la documentación [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/custom-keyword-overview). Para ver el código de ejemplo, consulte la sección Ejemplos más adelante. 
- **Java**: Se ha agregado la conversación con varios dispositivos con compatibilidad con traducción. Vea el documento de referencia [aquí](https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.transcription).

**Mejoras y optimizaciones**
- **JavaScript**: Implementación optimizada del micrófono del explorador para mejorar la precisión del reconocimiento de voz.
- **Java**: enlaces refactorizados mediante la implementación directa de JNI sin SWIG. Esto reduce en 10 veces el tamaño de los enlaces de todos los paquetes de Java usados para Windows, Android, Linux y Mac, y facilita el desarrollo de la implementación de Java del SDK de voz.
- **Linux**: [Documentación](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=linux) de compatibilidad actualizada con las notas específicas de RHEL 7 más recientes.
- Se ha mejorado la lógica de conexión para intentar conectarse varias veces en caso de errores de servicio y de red.
- Se ha actualizado la página de inicio rápido de Voz de [portal.azure.com](https://portal.azure.com) para ayudar a los desarrolladores a realizar el siguiente paso en el recorrido de Voz de Azure.

**Correcciones de errores**
- **C#, Java**: Se ha corregido un [problema](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/587) con la carga de bibliotecas de SDK en Linux ARM (32 y 64 bits).
- **C#** : Se ha corregido la cancelación explícita de identificadores nativos para los objetos TranslationRecognizer, IntentRecognizer y Connection.
- **C#** : Se ha corregido la administración de la duración de la entrada de audio para el objeto ConversationTranscriber.
- Se ha corregido un problema por el que la razón del resultado de `IntentRecognizer` no se establecía correctamente al reconocer la intención de frases simples.
- Se ha corregido un problema por el que el desplazamiento del resultado de `SpeechRecognitionEventArgs` no se establecía correctamente.
- Se ha corregido una condición de carrera en la que el SDK intentaba enviar un mensaje de red antes de abrir la conexión de WebSocket. Era reproducible para `TranslationRecognizer` al agregar participantes.
- Se han corregido las fugas de memoria en el motor de reconocedor de palabras clave.

**Muestras**
- **Go**: Se han agregado inicios rápidos para el [reconocimiento de voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/speech-to-text-from-microphone?pivots=programming-language-go) y el [asistente de Voz personalizada](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/voice-assistants?pivots=programming-language-go). Encuentre el código de ejemplo [aquí](https://github.com/microsoft/cognitive-services-speech-sdk-go/tree/master/samples). 
- **JavaScript**: Se han agregado inicios rápidos para la [conversión de texto a voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/text-to-speech?pivots=programming-language-javascript), la [traducción de texto a voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/get-started-speech-translation?tabs=script&pivots=programming-language-csharp) y el [reconocimiento de intenciones](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/intent-recognition?pivots=programming-language-javascript).
- Ejemplos de reconocimiento de palabras clave para [C\#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/uwp/keyword-recognizer) y [Java](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android/keyword-recognizer) (Android).  

**Pruebas reducidas ante la COVID-19:** Debido al trabajo de forma remota en las últimas semanas, no pudimos realizar tantas pruebas manuales de comprobación de como normalmente hacemos. No hemos hecho ningún cambio que creemos que pueda haber generado algún error y se han superado todas las pruebas automatizadas. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br>
Y sigan sanos.

## <a name="speech-sdk-1110-2020-march-release"></a>SDK de voz 1.11.0: Versión de marzo de 2020
**Nuevas características:**
- Linux: se ha agregado compatibilidad con Red Hat Enterprise Linux (RHEL)/CentOS 7 para x64 con [instrucciones](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-configure-rhel-centos-7) sobre cómo configurar el sistema para el SDK de voz.
- Linux: se ha agregado compatibilidad con C# de .Net Core en Linux ARM32 y ARM64. Obtenga más información [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/speech-sdk?tabs=linux). 
- C#, C++: se ha agregado `UtteranceId` en `ConversationTranscriptionResult`, un identificador coherente en todos los intermedios y el resultado final del reconocimiento de voz. Detalles de [C#](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.transcription.conversationtranscriptionresult?view=azure-dotnet), [C++](https://docs.microsoft.com/cpp/cognitive-services/speech/transcription-conversationtranscriptionresult).
- Python: se ha agregado compatibilidad con `Language ID` Consulte speech_sample.py en el [repositorio de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/python/console).
- Windows: se ha agregado compatibilidad con el formato de entrada de audio comprimido en la plataforma Windows para todas las aplicaciones de consola win32. Consulte los detalles [aquí](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-use-codec-compressed-audio-input-streams). 
- JavaScript: compatibilidad con síntesis de voz (conversión de texto en voz) en NodeJS. Obtenga más información [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript/node/text-to-speech). 
- JavaScript: se han agregado nuevas API para habilitar la inspección de todos los mensajes enviados y recibidos. Obtenga más información [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/javascript). 
        
**Correcciones de errores**
- C#, C++: se ha corregido un problema y ahora `SendMessageAsync` envía el mensaje binario como tipo binario. Detalles de [C#](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.connection.sendmessageasync?view=azure-dotnet#Microsoft_CognitiveServices_Speech_Connection_SendMessageAsync_System_String_System_Byte___System_UInt32_), [C++](https://docs.microsoft.com/cpp/cognitive-services/speech/connection).
- C#, C++: se ha corregido un problema por el cual el uso del evento `Connection MessageReceived` puede causar un bloqueo si se elimina `Recognizer` antes del objeto `Connection`. Detalles de [C#](https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.connection.messagereceived?view=azure-dotnet), [C++](https://docs.microsoft.com/cpp/cognitive-services/speech/connection#messagereceived).
- Android: el tamaño de búfer del audio desde el micrófono ha disminuido de 800 ms a 100 ms para mejorar la latencia.
- Android: se ha corregido un [problema](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/563) con el emulador de Android para x86 en Android Studio.
- JavaScript: se ha agregado compatibilidad con regiones en China con la API `fromSubscription`. Consulte los detalles [aquí](https://docs.microsoft.com/javascript/api/microsoft-cognitiveservices-speech-sdk/speechconfig?view=azure-node-latest#fromsubscription-string--string-). 
- JavaScript: se ha agregado más información para los errores de conexión de NodeJS.
        
**Muestras**
- Unity: se ha corregido el ejemplo público de reconocimiento de la intención, en el que se produjo un error en la importación de JSON de LUIS. Consulte los detalles [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/369).
- Python: Ejemplo agregado para `Language ID`. Consulte los detalles [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/python/console/speech_sample.py).
    
**Pruebas abreviadas de COVID-19:** Debido al trabajo de forma remota en las últimas semanas, no pudimos realizar tantas pruebas manuales de comprobación de dispositivos como normalmente hacemos. Un ejemplo de esto es la prueba de entrada y salida del micrófono en Linux, iOS y macOS. No hemos hecho ningún cambio que creemos que pudiera haber producido algún error en estas plataformas y todas las pruebas automatizadas han pasado. En el improbable caso de que falte algo, háganoslo saber en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen).<br> Gracias por su asistencia continuada. Como siempre, publique las preguntas o comentarios en [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues?q=is%3Aissue+is%3Aopen) o en [Stack Overflow](https://stackoverflow.microsoft.com/questions/tagged/731).<br>
Y sigan sanos.

## <a name="speech-sdk-1100-2020-february-release"></a>Speech SDK 1.10.0: versión de febrero de 2020

**Nuevas características:**

 - Se han agregado paquetes de Python para admitir la nueva versión 3.8 de Python.
 - Compatibilidad con Red Hat Enterprise Linux (RHEL) y CentOS 8 x64 (C++, C#, Java y Python).
   > [!NOTE] 
   > Los clientes deben configurar OpenSSL según [estas instrucciones](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-configure-openssl-linux).
 - Compatibilidad de Linux ARM32 con Debian y Ubuntu.
 - DialogServiceConnector ahora admite un parámetro opcional "bot ID" en BotFrameworkConfig. Este parámetro permite el uso de varios bots de Direct Line Speech con un solo recurso de voz de Azure. Si no se especifica el parámetro, se utilizará el bot predeterminado (el que se determine en la página de configuración del canal de Direct Line Speech).
 - DialogServiceConnector ahora tiene una propiedad SpeechActivityTemplate. Direct Line Speech usa el contenido de esta cadena JSON para rellenar previamente una gran variedad de campos admitidos en todas las actividades que acceden a un bot de Direct Line Speech, incluidas las actividades generadas automáticamente en respuesta a eventos como el reconocimiento de voz.
 - TTS ahora usa la clave de suscripción para la autenticación, lo que reduce la latencia del primer byte del primer resultado de la síntesis después de crear un sintetizador.
 - Se han actualizado los modelos de reconocimiento de voz de 19 configuraciones regionales, con lo que se ha logrado una reducción media de la tasa de errores de palabras del 18,6 % (es-ES, es-MX, FR-CA, fr-FR, TI-IT, ja-JP, ko-KR, pt-BR, zh-CN, ZH-HK, NB-NO, fi-FL, ru-RU, pl-PL, CA-ES, zh-TW, TH-TH, pt-PT y tr-TR). Los nuevos modelos aportan mejoras significativas en varios dominios, entre los que se incluyen los escenarios de dictado, transcripción del centro de llamadas e indexación de vídeo.

**Correcciones de errores**

 - Se ha corregido el error por el que la transcripción de conversaciones no esperaba correctamente en las API de JAVA 
 - Revisión del emulador de Android x86 para Xamarin [problema de GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/363)
 - Adición de métodos Property (Get|Set) a AudioConfig
 - Corrección de un error de TTS en el que audioDataStream no se puede detener cuando se produce un error en la conexión
 - El uso de un punto de conexión sin una región provocaría errores en el USP para el traductor de conversaciones
 - La generación de identificadores en las aplicaciones universales de Windows ahora usa un algoritmo de GUID único; cuyo valor predeterminado anterior y no intencionado es una implementación con código auxiliar que a menudo producía colisiones en conjuntos de interacciones grandes.
 
 **Muestras**
 
 - Muestra de Unity con la que usar Speech SDK [Micrófono de Unity y streaming en modo de inserción](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/unity/from-unitymicrophone)

**Otros cambios**

 - [Documentación de la configuración de OpenSSL actualizada para Linux](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-configure-openssl-linux)

## <a name="speech-sdk-190-2020-january-release"></a>Acerca del SDK de Voz 1.9.0: Versión de enero de 2020

**Nuevas características**

- Conversación entre varios dispositivos: conecte varios dispositivos a la misma conversación basada en texto o en voz y, opcionalmente, traduzca los mensajes que se envían entre ellos. Más información en [este artículo](multi-device-conversation.md). 
- Se ha agregado compatibilidad con el reconocimiento de palabras clave para el paquete .aar de Android y se ha agregado compatibilidad con las versiones x86 y x64. 
- Objective-C: se han agregado los métodos `SendMessage` y `SetMessageProperty` al objeto `Connection`. Consulte la documentación [aquí](https://docs.microsoft.com/objectivec/cognitive-services/speech/spxconnection).
- La API de C++ para TTS ahora admite `std::wstring` como entrada de texto de síntesis, lo que elimina la necesidad de convertir un valor wstring en string antes de pasarlo al SDK. Consulte los detalles [aquí](https://docs.microsoft.com/cpp/cognitive-services/speech/speechsynthesizer#speaktextasync). 
- C#: El [identificador de idioma](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-automatic-language-detection?pivots=programming-language-csharp) y la [configuración del idioma de origen](https://docs.microsoft.com/azure/cognitive-services/speech-service/how-to-specify-source-language?pivots=programming-language-csharp) ya están disponibles.
- JavaScript: Se ha agregado una característica al objeto `Connection` para el paso a través de mensajes personalizados desde el servicio de voz como objeto `receivedServiceMessage` de devolución de llamada.
- JavaScript: Se ha agregado compatibilidad con `FromHost API` para facilitar su uso con contenedores locales y nubes soberanas. Consulte la documentación [aquí](speech-container-howto.md).
- JavaScript: Ahora se admite `NODE_TLS_REJECT_UNAUTHORIZED` gracias a una contribución de [orgads](https://github.com/orgads). Consulte los detalles [aquí](https://github.com/microsoft/cognitive-services-speech-sdk-js/pull/75).

**Cambios importantes**

- `OpenSSL` se ha actualizado a la versión 1.1.1b y está vinculada estáticamente a la biblioteca principal del SDK de Voz para Linux. Esto puede producir una interrupción si la bandeja de entrada `OpenSSL` no se ha instalado en el directorio `/usr/lib/ssl` del sistema. Consulte [nuestra documentación](how-to-configure-openssl-linux.md) en los documentos sobre el SDK de Voz para solucionar el problema.
- Hemos cambiado el tipo de datos devuelto para C# `WordLevelTimingResult.Offset` de `int` a `long` para permitir el acceso a `WordLevelTimingResults` cuando los datos de voz duren más de 2 minutos.
- `PushAudioInputStream` y `PullAudioInputStream` envían ahora información de los encabezados WAV al servicio de voz basado en `AudioStreamFormat`, que se especificó como opción cuando se crearon. Los clientes deben utilizar ahora el [formato de entrada de audio admitido](how-to-use-audio-input-streams.md). Cualquier otro formato obtendrá resultados de reconocimiento no óptimos o podría causar otros problemas. 

**Correcciones de errores**

- Consulte la actualización de `OpenSSL` en cambios importantes anteriores. Hemos corregido un bloqueo intermitente y un problema de rendimiento (contención de bloqueo bajo carga alta) en Linux y Java. 
- Java: Se han realizado mejoras en la clausura de objetos en escenarios de alta simultaneidad.
- Se ha reestructurado nuestro paquete NuGet. Se han eliminado las tres copias de `Microsoft.CognitiveServices.Speech.core.dll` y `Microsoft.CognitiveServices.Speech.extension.kws.dll` en las carpetas lib, con lo cual el paquete NuGet es ahora más pequeño y su descarga es más rápida, y se han agregado los encabezados necesarios para compilar algunas aplicaciones nativas en C++.
- [Aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/cpp) puede encontrar ejemplos corregidos del inicio rápido. Estos estaban saliendo sin mostrar la excepción "No se encontró el micrófono" en Linux, macOS y Windows.
- Se ha corregido un bloqueo del SDK por el que se producían resultados de reconocimientos de voz largos en determinadas rutas de acceso al código como en [este ejemplo](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/csharp/uwp/speechtotext-uwp).
- Se ha corregido un error en la implementación del SDK en el entorno de Azure Web App para solucionar [este problema del cliente](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/396).
- Se ha corregido un error de TTS al usar varias etiquetas `<voice>` o `<audio>` para solucionar [este problema del cliente](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/433). 
- Se ha corregido un error TTS 401 cuando se recupera el SDK del estado suspendido.
- JavaScript: Se ha corregido una importación circular de datos de audio gracias a una contribución de [euirim](https://github.com/euirim). 
- JavaScript: se ha agregado compatibilidad para establecer las propiedades del servicio, como se hizo en 1.7.
- JavaScript: se ha corregido un problema por el que un error de conexión podría provocar intentos de reconexión de WebSocket continuos e incorrectos.

**Muestras**

- Se ha agregado el ejemplo de reconocimiento de palabras clave para Android [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/java/android/sdkdemo).
- Se ha agregado el ejemplo de TTS para el escenario de servidor [aquí](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/csharp/sharedcontent/console/speech_synthesis_server_scenario_sample.cs).
- Se han agregado inicios rápidos de conversación entre varios dispositivos para C# y C++ [aquí](quickstarts/multi-device-conversation.md).

**Otros cambios**

- Se ha optimizado el tamaño de la biblioteca principal del SDK en Android.
- El SDK de las versiones 1.9.0 y posteriores admite los tipos `int` y `string` en el campo Versión de la firma de voz para la transcripción de conversaciones.

## <a name="speech-sdk-180-2019-november-release"></a>SDK de Voz 1.8.0: Versión de noviembre de 2019

**Nuevas características**

- Se ha agregado una API `FromHost()` para facilitar su uso con contenedores locales y nubes soberanas.
- Se ha agregado la detección automática de idioma de origen para el reconocimiento de voz (en Java y C++)
- Se ha agregado el objeto `SourceLanguageConfig` para el reconocimiento de voz, que se usa para especificar los idiomas de origen esperados (en Java y C++).
- Se ha agregado compatibilidad con `KeywordRecognizer` en Windows (UWP), Android e iOS mediante los paquetes de NuGet y Unity.
- Se ha agregado la API de Java de conversación remota para realizar la transcripción de conversaciones en lotes asincrónicos.

**Cambios importantes**

- Las funcionalidades de transcripción de conversaciones se han movido al espacio de nombres `Microsoft.CognitiveServices.Speech.Transcription`.
- Parte de los métodos de transcripción de conversaciones se han movido a la nueva clase `Conversation`.
- Compatibilidad eliminada para iOS de 32 bits (ARMv7 y x86)

**Correcciones de errores**

- Se ha corregido un bloqueo si se usa `KeywordRecognizer` local sin una clave de suscripción válida al servicio de voz.

**Muestras**

- Ejemplo de Xamarin para `KeywordRecognizer`
- Ejemplo de Unity para `KeywordRecognizer`
- Ejemplos de C++ y Java de detección automática de idioma de origen

## <a name="speech-sdk-170-2019-september-release"></a>SDK de voz 1.7.0: versión de septiembre de 2019

**Nuevas características**

- Compatibilidad con la versión beta agregada para Xamarin en la Plataforma universal de Windows (UWP), Android e iOS
- Compatibilidad con iOS agregada para Unity
- Se ha agregado compatibilidad con entradas `Compressed` para ALaw, Mulaw, FLAC en Android, iOS y Linux.
- Se ha agregado `SendMessageAsync` en la clase `Connection` para enviar un mensaje al servicio.
- Se ha agregado `SetMessageProperty` en la clase `Connection` para establecer la propiedad de un mensaje.
- TTS agregó enlaces para Java (JRE y Android), Python, Swift y Objective-C.
- TTS agregó compatibilidad de reproducción para macOS, iOS y Android
- Se ha agregado información de "límite de palabras" para TTS

**Correcciones de errores**

- Se ha corregido un problema de compilación de IL2CPP en Unity 2019 para Android
- Se ha corregido un problema con los encabezados con formato incorrecto en la entrada de archivo WAV que se procesa de forma incorrecta
- Se ha corregido un problema con UUID que no es único en algunas propiedades de conexión
- Se han corregido algunas advertencias sobre los especificadores de nulabilidad en los enlaces SWIFT (puede que se requieran pequeños cambios en el código)
- Se ha corregido un error que provocaba que las conexiones de WebSocket se cerraran de manera incorrecta en la carga de red
- Se ha corregido un problema en Android que a veces provoca que `DialogServiceConnector` use identificadores de impresión duplicados.
- Se han introducido mejoras en la estabilidad de las conexiones entre interacciones multiproceso y la generación de informes de errores (a través de eventos `Canceled`) cuando se producen con `DialogServiceConnector`.
- Los inicios de sesión de `DialogServiceConnector` ahora proporcionarán eventos correctamente, incluso si se llama a `ListenOnceAsync()` durante una operación `StartKeywordRecognitionAsync()` activa.
- Se ha resuelto un bloqueo asociado a la recepción de actividades `DialogServiceConnector`.

**Muestras**

- Inicio rápido para Xamarin
- Se ha actualizado el inicio rápido de CPP con información de ARM64 de Linux
- Se ha actualizado el inicio rápido de Unity con información de iOS

## <a name="speech-sdk-160-2019-june-release"></a>SDK de Voz 1.6.0: versión de junio de 2019

**Muestras**

- Ejemplos de inicio rápido para Texto a voz en UWP y Unity
- Ejemplo de inicio rápido para Swift en iOS
- Ejemplos de Unity para Traducción y Reconocimiento de la intención comunicativa y Voz
- Ejemplos de inicios rápidos actualizados para `DialogServiceConnector`

**Mejoras y cambios**

- Espacio de nombres de cuadro de diálogo:
  - El nombre de `SpeechBotConnector` ha cambiado a `DialogServiceConnector`
  - El nombre de `BotConfig` ha cambiado a `DialogServiceConfig`
  - `BotConfig::FromChannelSecret()` se ha reasignado a `DialogServiceConfig::FromBotSecret()`
  - Todos los clientes de Voz de Direct Line existentes siguen siendo compatibles después del cambio de nombre
- Actualización del adaptador REST de TTS para admitir una conexión persistente de proxy
- Un mejor mensaje de error cuando se pasa una región no válida
- Swift/Objective-C:
  - Mejores informes de errores: los métodos que pueden generar un error ahora se encuentran en dos versiones: una que expone un objeto `NSError` para el control de errores y una que genera una excepción. La primera se expone a Swift. Este cambio requiere adaptaciones en el código Swift existente.
  - Mejor control de eventos

**Correcciones de errores**

- Corrección de TTS: donde el futuro de `SpeakTextAsync` se devolvió sin esperar al fin de la representación del audio
- Corrección para la serialización de las cadenas en C# para permitir la compatibilidad total con idiomas
- Corrección del problema de las aplicaciones centrales de .NET para cargar la biblioteca principal con un marco de destino net461 en ejemplos
- Corrección de problemas ocasionales para implementar bibliotecas nativas en la carpeta de salida en los ejemplos
- Corrección para cerrar el socket web de manera confiable
- Corrección de un posible bloqueo al abrir una conexión con una carga muy elevada en Linux
- Corrección de metadatos faltantes en el paquete de marcos para macOS
- Corrección de problemas con `pip install --user` en Windows

## <a name="speech-sdk-151"></a>Speech SDK 1.5.1

Se trata de una versión de corrección de errores y solo afecta al SDK nativo o administrado. No afecta a la versión de JavaScript del SDK.

**Correcciones de errores**

- Corrección de FromSubscription cuando se usa con la transcripción de la conversación.
- Corrección de errores en la detección de palabras clave para los asistentes por voz.

## <a name="speech-sdk-150-2019-may-release"></a>Speech SDK 1.5.0 Versión de mayo de 2019

**Nuevas características:**

- La detección de palabras clave (KWS) ahora está disponible para Windows y Linux. La funcionalidad KWS podría funcionar con cualquier tipo de micrófono; no obstante, la compatibilidad oficial de KWS está limitada actualmente a las matrices de micrófonos que se encuentran en el hardware de Azure Kinect DK o el SDK de dispositivos de voz.
- La funcionalidad de sugerencia de frases está disponible a través del SDK. Para más información, consulte [esta página](how-to-phrase-lists.md).
- La funcionalidad de transcripción de conversaciones está disponible a través del SDK. Consulte [aquí](conversation-transcription-service.md).
- Compatibilidad agregada con los asistentes por voz mediante el canal Direct Line Speech.

**Muestras**

- Se han agregado ejemplos para nuevas características o nuevos servicios admitidos por el SDK.

**Mejoras y cambios**

- Se han agregado varias propiedades de reconocimiento para ajustar el comportamiento del servicio o los resultados del servicio (por ejemplo, enmascaramiento de palabras soeces etc.).
- Ahora puede configurar el reconocimiento a través de las propiedades de configuración estándar, incluso si ha creado el valor de `FromEndpoint` del reconocedor.
- Objective-C: se agregó la propiedad `OutputFormat` a `SPXSpeechConfiguration`.
- El SDK ahora admite Debian 9 como una distribución de Linux.

**Correcciones de errores**

- Se ha corregido un problema donde el recurso de altavoz se destruía demasiado pronto en la conversión de texto a voz.

## <a name="speech-sdk-142"></a>Speech SDK 1.4.2

Se trata de una versión de corrección de errores y solo afecta al SDK nativo o administrado. No afecta a la versión de JavaScript del SDK.

## <a name="speech-sdk-141"></a>Speech SDK 1.4.1

Esta es una versión solo para JavaScript. No se agregó ninguna característica. Se realizaron las siguientes correcciones:

- Se impide que el paquete web cargue https-proxy-agent.

## <a name="speech-sdk-140-2019-april-release"></a>Speech SDK 1.4.0 Versión de abril de 2019

**Nuevas características:**

- El SDK admite ahora el servicio de conversión de texto a voz en versión beta. Se admite en Windows y Linux Desktop desde C++ y C#. Para más información, consulte la [información general sobre la conversión de texto a voz](text-to-speech.md#get-started).
- El SDK ahora admite archivos de audio MP3 y Opus/OGG como archivos de entrada de secuencia. Esta característica solo está disponible en Linux desde C++ y C# y está actualmente en versión beta (más detalles [aquí](how-to-use-codec-compressed-audio-input-streams.md)).
- Speech SDK para Java, .NET Core, C++ y Objective-C ha conseguido compatibilidad con macOS. La compatibilidad de Objective-C con macOS está actualmente en versión beta.
- iOS: Speech SDK para iOS (Objective-C) ahora también se publica como una instancia de CocoaPod.
- JavaScript: compatibilidad con micrófono no predeterminada como dispositivo de entrada.
- JavaScript: compatibilidad con servidores proxy para Node.js.

**Muestras**

- se han agregado ejemplos para usar Speech SDK con C++ y con Objective-C en macOS.
- Se han agregado ejemplos que muestran el uso del servicio de conversión de texto a voz.

**Mejoras y cambios**

- Python: ahora se exponen propiedades adicionales de los resultados del reconocimiento mediante la propiedad `properties`.
- Para la compatibilidad adicional con el desarrollo y la depuración, puede redirigir la información de registro y diagnóstico del SDK a un archivo de registro (más información [aquí](how-to-use-logging.md)).
- JavaScript: mejora del rendimiento del procesamiento de audio.

**Correcciones de errores**

- Mac/iOS: se corrigió un error que daba lugar a una larga espera cuando no se podía establecer una conexión con el servicio de voz.
- Python: mejora del control de errores en los argumentos de las devoluciones de llamada de Python.
- JavaScript: se corrigieron los informes de estado erróneos de la voz que finalizaban en RequestSession.

## <a name="speech-sdk-131-2019-february-refresh"></a>Speech SDK 1.3.1 Actualización de febrero de 2019

Se trata de una versión de corrección de errores y solo afecta al SDK nativo o administrado. No afecta a la versión de JavaScript del SDK.

**Corrección de error**

- Se ha corregido una fuga de memoria cuando se usa la entrada de micrófono. No afecta a la entrada de archivos o basada en secuencias.

## <a name="speech-sdk-130-2019-february-release"></a>Speech SDK 1.3.0: versión de febrero de 2019

**Nuevas características**

- El SDK de voz admite la selección del micrófono de entrada mediante la clase `AudioConfig`. Esto permite transmitir datos de audio al servicio de voz desde un micrófono no predeterminado. Para más información, consulte la documentación en la que se describe cómo [seleccionar un dispositivo de entrada de audio](how-to-select-audio-input-devices.md). Esta característica aún no está disponible en JavaScript.
- Speech SDK ahora es compatible con Unity en una versión beta. Proporcione sus comentarios en la sección de problemas en el [repositorio de ejemplos de GitHub](https://aka.ms/csspeech/samples). Esta versión es compatible con Unity en Windows x86 y x64 (aplicaciones de escritorio o de la Plataforma universal de Windows) y Android (ARM32/64, x86). Puede encontrar más información en nuestra [guía de inicio rápido sobre Unity](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=unity).
- El archivo `Microsoft.CognitiveServices.Speech.csharp.bindings.dll` (incluido en versiones anteriores) ya no es necesario. La funcionalidad está ahora integrada en el SDK principal.

**Muestras**

El siguiente contenido nuevo está disponible en nuestro [repositorio de ejemplo](https://aka.ms/csspeech/samples):

- Ejemplos adicionales para `AudioConfig.FromMicrophoneInput`.
- Ejemplos adicionales de Python para traducción y reconocimiento de intenciones.
- Ejemplos adicionales para usar el objeto `Connection` en iOS.
- Ejemplos adicionales de Java para la traducción con la salida de audio.
- Nuevo ejemplo de uso de la [API de REST de transcripción de lotes](batch-transcription.md).

**Mejoras y cambios**

- Python
  - Mensajes de error y verificación de parámetros mejorada en `SpeechConfig`.
  - Adición de compatibilidad para el objeto `Connection`.
  - Compatibilidad con Python (x86) de 32 bits en Windows.
  - Speech SDK para Python ya no está disponible como beta.
- iOS
  - El SDK ahora se compila en función de la versión 12.1 del SDK de iOS.
  - El SDK ahora es compatible con las versiones 9.2 y posteriores de iOS.
  - Documentación de referencia mejorada y corrección de varios nombres de propiedad.
- JavaScript
  - Adición de compatibilidad para el objeto `Connection`.
  - Archivos de definición de tipos agregados para JavaScript agrupado.
  - Compatibilidad e implementación iniciales para sugerencias de frases.
  - Colección de propiedades devuelta con JSON del servicio para reconocimiento.
- Los archivos DLL de Windows contienen ahora un recurso de versión.
- Si crea un valor de `FromEndpoint` de reconocedor, puede agregar parámetros directamente a la dirección URL del punto de conexión. Con `FromEndpoint` no puede configurar el reconocedor mediante las propiedades de configuración estándar.

**Correcciones de errores**

- La contraseña de proxy y el nombre de usuario de proxy vacíos no se administraron correctamente. Con esta versión, si establece el nombre de usuario de proxy y la contraseña de proxy en una cadena vacía, no se enviarán al conectarse al proxy.
- El identificador de sesión creado por el SDK no siempre es realmente aleatorio para algunos lenguajes o&nbsp; entornos. Se ha agregado la inicialización del generador aleatorio para corregir este problema.
- Control mejorado del token de autorización. Si desea usar un token de autorización, especifíquelo en `SpeechConfig` y deje la clave de suscripción vacía. A continuación, cree el reconocedor como de costumbre.
- En algunos casos, el objeto `Connection` no se publicó correctamente. Ahora se ha corregido.
- Se corrigió el ejemplo de JavaScript para admitir la salida de audio para la síntesis de traducción también en Safari.

## <a name="speech-sdk-121"></a>Speech SDK 1.2.1

Esta es una versión solo para JavaScript. No se agregó ninguna característica. Se realizaron las siguientes correcciones:

- Activar el final del flujo en turn.end, y no en speech.end.
- Corregir error de la bomba de audio por el que no se programaba el siguiente envío en caso de error del envío actual.
- Corregir el reconocimiento continuo con el token de autenticación.
- Corrección de errores de diferentes reconocedores y puntos de conexión.
- Mejoras en la documentación.

## <a name="speech-sdk-120-2018-december-release"></a>Speech SDK 1.2.0: Versión de diciembre de 2018

**Nuevas características**

- Python
  - La versión beta de la compatibilidad con Python (3.5 y versiones posteriores) está disponible con esta versión. Para más información, consulte aquí](quickstart-python.md).
- JavaScript
  - Speech SDK para JavaScript ha sido de código abierto. El código fuente está disponible en [GitHub](https://github.com/Microsoft/cognitive-services-speech-sdk-js).
  - Ya se admite Node.js; puede encontrar más información [aquí](quickstart-js-node.md).
  - Se quitó la restricción de longitud para las sesiones de audio; la reconexión se realizará automáticamente en la portada.
- Objecto `Connection`
  - Desde el objeto `Recognizer`, puede acceder al objeto `Connection`. Este objeto le permite iniciar la conexión al servicio y suscribirse para conectar y desconectar eventos explícitamente.
    (Esta característica no está disponible aún ni en JavaScript ni en Python).
- Compatibilidad con Ubuntu 18.04.
- Android
  - Compatibilidad con ProGuard habilitada durante la generación del APK.

**Mejoras**

- Mejoras en el uso de subprocesos internos, lo que reduce el número de subprocesos, bloqueos y exclusiones mutuas.
- Se mejoraron los informes de errores y la información. En algunos casos, los mensajes de error no se propagan totalmente.
- Se actualizaron las dependencias de desarrollo en JavaScript para usar los módulos actualizados.

**Correcciones de errores**

- Se han corregido las fugas de causadas por un error de coincidencia de tipos en `RecognizeAsync`.
- En algunos casos, se perdieron excepciones.
- Corrección de las fugas de memoria en los argumentos de eventos de traducción.
- Se ha corregido un problema de bloqueo al volver a conectar en sesiones de larga ejecución.
- Se ha corregido un problema que podría dar lugar a que faltase el resultado final para las traducciones con errores.
- C#: Si no se esperaba una operación `async` en el subproceso principal, es posible que se pudiese desechar el reconocedor antes de completarse la tarea asincrónica.
- Java: Se ha corregido un problema que provocaba un bloqueo de la VM de Java.
- Objective-C: Se ha corregido la asignación de la enumeración; se devolvió RecognizedIntent en lugar de `RecognizingIntent`.
- JavaScript: Se ha establecido el formato de salida predeterminado en "simple" en `SpeechConfig`.
- JavaScript: Se ha quitado una incoherencia entre las propiedades del objeto de configuración en JavaScript y otros lenguajes.

**Muestras**

- Se han actualizado y corregido varios ejemplos, como las voces de salida para la traducción, etc.
- Se han agregado ejemplos de Node.js en el [repositorio de ejemplo](https://aka.ms/csspeech/samples).

## <a name="speech-sdk-110"></a>Speech SDK 1.1.0

**Nuevas características**

- Compatibilidad con Android x86/x64.
- Compatibilidad con proxy: En el objeto `SpeechConfig`, ahora puede llamar a una función para establecer la información del proxy (nombre de host, puerto, nombre de usuario y contraseña). Esta característica no está disponible aún en iOS.
- Mensajes y códigos de error mejorados. Si un reconocimiento devolvió un error, esto ya ha establecido `Reason` (en el evento cancelado) o `CancellationDetails` (en el resultado del reconocimiento) en `Error`. El evento cancelado ahora contiene dos miembros adicionales, `ErrorCode` y `ErrorDetails`. Si el servidor devolvió información de error adicional con el error notificado, ahora estará disponible en los nuevos miembros.

**Mejoras**

- Verificación adicional agregada en la configuración del reconocedor y mensaje de error adicional agregado.
- Control mejorado del silencio prolongado en medio de un archivo de audio.
- Paquete NuGet: para proyectos de .NET Framework, evita la compilación con la configuración de AnyCPU.

**Correcciones de errores**

- En los reconocedores se han encontrado varias excepciones corregidas. Además, las excepciones se detectan y se convierten en un evento `Canceled`.
- Corrección de una fuga de memoria en la administración de propiedades.
- Se corrigió el error en el que un archivo de entrada de audio podría bloquear el reconocedor.
- Se corrigió un error donde se podrían recibir eventos después de un evento de detención de la sesión.
- Se corrigieron algunas condiciones de subprocesos.
- Se corrigió un problema de compatibilidad de iOS que podría dar lugar a un bloqueo.
- Mejoras de estabilidad para la compatibilidad del micrófono en Android.
- Se corrigió un error donde un reconocedor en JavaScript ignoraría el lenguaje de reconocimiento.
- Se corrigió un error que impedía establecer el valor `EndpointId` (en algunos casos) en JavaScript.
- Se cambió el orden de los parámetros en AddIntent en JavaScript y se agregó la firma de JavaScript `AddIntent` que faltaba.

**Muestras**

- Se han agregado ejemplos de C++ y C# para el uso de transmisiones de inserción y extracción en el [repositorio de ejemplos](https://aka.ms/csspeech/samples).

## <a name="speech-sdk-101"></a>Speech SDK 1.0.1

Mejoras en la confiabilidad y correcciones de errores:

- Corrección de un potencial error grave debido a una condición de carrera al desechar un reconocedor
- Corrección de un potencial error grave en el caso de propiedades sin establecer.
- Comprobación adicional de errores y parámetros.
- Objective-C: corrección de posibles errores graves causados por la invalidación de nombres en NSString.
- Objective-C: ajuste de visibilidad en la API
- JavaScript: corrección con respecto a los eventos y sus cargas.
- Mejoras en la documentación.

Se ha agregado un nuevo ejemplo de Javascript en nuestro [repositorio de ejemplos](https://aka.ms/csspeech/samples).

## <a name="cognitive-services-speech-sdk-100-2018-september-release"></a>SDK de Voz 1.0.0 de Cognitive Services: Versión de septiembre de 2018

**Nuevas características:**

- Compatibilidad con Objective-C en iOS. Consulte la [Guía de inicio rápido de Objective-C para iOS](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone-langs/objectivec-ios.md).
- Se admite JavaScript en el explorador. Consulte la [Guía de inicio rápido de JavaScript](quickstart-js-browser.md).

**Cambios importantes**

- Con esta versión se presentan una serie de cambios importantes.
  Consulte [esta página](https://aka.ms/csspeech/breakingchanges_1_0_0) para más información.

## <a name="cognitive-services-speech-sdk-060-2018-august-release"></a>SDK de Voz 0.6.0 de Cognitive Services: Versión de agosto de 2018

**Nuevas características:**

- Ahora, las aplicaciones de UWP creadas con SDK de Voz superan el Kit para la certificación de aplicaciones en Windows (WACK).
  Consulte la [Guía de inicio rápido de UWP](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-chsarp&tabs=uwp).
- Compatibilidad con .NET Standard 2.0 en Linux (Ubuntu 16.04 x64).
- Experimental: compatibilidad con Java 8 en Windows (64 bits) y Linux (Ubuntu 16.04 x 64).
  Consulte la [Guía de inicio rápido de Java Runtime Environment](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-java&tabs=jre).

**Cambios funcionales**

- Se expone más información detallada sobre los errores de conexión.

**Cambios importantes**

- En Java (Android), la función `SpeechFactory.configureNativePlatformBindingWithDefaultCertificate` ya no requiere un parámetro de ruta de acceso. Ahora, la ruta de acceso se detecta automáticamente en todas las plataformas compatibles.
- En Java y C#, se ha quitado el descriptor de acceso get- de la propiedad `EndpointUrl`.

**Correcciones de errores**

- En Java, se implementa ahora el resultado de la síntesis de audio en el reconocedor de traducción.
- Se ha corregido un error que podía provocar subprocesos inactivos y un mayor número de sockets abiertos y sin usar.
- Se ha corregido un problema por el que un proceso de reconocimiento de larga ejecución podía terminar en mitad de la transmisión.
- Se ha corregido una condición de carrera en el proceso de apagado del reconocedor.

## <a name="cognitive-services-speech-sdk-050-2018-july-release"></a>SDK de Voz 0.5.0 de Cognitive Services: Versión de julio de 2018

**Nuevas características:**

- Compatibilidad con la plataforma Android (API 23: Android Marshmallow 6.0 o posterior). Consulte el [inicio rápido de Android](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-java&tabs=android).
- Compatibilidad con .NET Standard 2.0 en Windows. Consulte el [inicio rápido de .NET Core](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=dotnetcore).
- Experimental: compatibilidad con UWP en Windows (versión 1709 o posterior).
  - Consulte la [Guía de inicio rápido de UWP](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-csharp&tabs=uwp).
  - Nota: Las aplicaciones de UWP creadas con el SDK de Voz no pasan aún el Kit para la certificación de aplicaciones en Windows (WACK).
- Compatibilidad con el reconocimiento de ejecución prolongada con reconexión automática.

**Cambios funcionales**

- `StartContinuousRecognitionAsync()` admite reconocimiento de ejecución prolongada.
- El resultado del reconocimiento contiene más campos. Tienen un desplazamiento desde el principio del audio y la duración (ambos en tics) del texto reconocido y valores adicionales que representan el estado de reconocimiento, por ejemplo, `InitialSilenceTimeout` e `InitialBabbleTimeout`.
- Compatibilidad con AuthorizationToken para la creación de instancias de fábrica.

**Cambios importantes**

- Eventos de reconocimiento: el tipo de evento `NoMatch` se combina con el evento `Error`.
- SpeechOutputFormat en C# se llama ahora `OutputFormat` para concordar con C++.
- El tipo de valor devuelto de algunos métodos de la interfaz `AudioInputStream` se ha modificado ligeramente:
  - En Java, el método `read` ahora devuelve `long` en lugar de `int`.
  - En C#, el método `Read` ahora devuelve `uint` en lugar de `int`.
  - En C++, los métodos `Read` y `GetFormat` ahora devuelven `size_t` en lugar de `int`.
- C++: las instancias de secuencias de entrada de audio ahora solo se pueden pasar como un valor `shared_ptr`.

**Correcciones de errores**

- Se han corregido los valores devueltos incorrectos cuando se agota el tiempo de espera de `RecognizeAsync()`.
- Se ha eliminado la dependencia de las bibliotecas de Media Foundation en Windows. El SDK ahora usa las API de audio básicas.
- Corrección de la documentación: se ha agregado una página de [regiones](regions.md) para describir cuáles son las regiones admitidas.

**Problema conocido**

- SDK de Voz para Android no informa de los resultados de la síntesis de voz para la traducción. Este problema se solucionará en la próxima versión.

## <a name="cognitive-services-speech-sdk-040-2018-june-release"></a>SDK de Voz 0.4.0 de Cognitive Services: Versión de junio de 2018

**Cambios funcionales**

- AudioInputStream

  Un reconocedor ahora puede consumir una secuencia como origen de audio. Para más información, consulte la [guía de procedimientos](how-to-use-audio-input-streams.md) relacionada.

- Formato de salida detallado

  Al crear un elemento `SpeechRecognizer`, puede solicitar el formato de salida `Detailed` o `Simple`. `DetailedSpeechRecognitionResult` contiene una puntuación de confianza, texto reconocido, formato léxico sin formato, formato normalizado y formato normalizado con palabras soeces enmascaradas.

**Cambio importante**

- En C# se cambia de `SpeechRecognitionResult.RecognizedText` a `SpeechRecognitionResult.Text`.

**Correcciones de errores**

- Se ha corregido un posible problema de devolución de llamada en la capa USP durante el apagado.
- Si un reconocedor usaba un archivo de entrada de audio, mantenía el identificador de archivo más tiempo del necesario.
- Se han eliminado varios interbloqueos entre el suministro de mensajes y el reconocedor.
- Se desencadena un resultado `NoMatch` cuando se agota la respuesta del servicio.
- Las bibliotecas de Media Foundation en Windows son de carga retrasada. Esta biblioteca solo es necesaria para la entrada del micrófono.
- La velocidad de carga de los datos de audio se limita al doble de la velocidad de audio original.
- En Windows, los ensamblados .NET de C# ahora son de nombre seguro.
- Corrección de la documentación: `Region` necesita información para crear un reconocedor.

Se han agregado más ejemplos y se actualizan constantemente. Para obtener el conjunto más reciente de ejemplos, consulte el [repositorio de GitHub de ejemplos de SDK de Voz](https://aka.ms/csspeech/samples).

## <a name="cognitive-services-speech-sdk-0212733-2018-may-release"></a>SDK de Voz 0.2.12733 de Cognitive Services: Versión de mayo de 2018

Esta versión es la primera versión preliminar pública de SDK de Voz de Cognitive Services.
