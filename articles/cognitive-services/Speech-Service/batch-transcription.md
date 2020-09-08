---
title: 'Uso de la transcripción por lotes: servicio de voz'
titleSuffix: Azure Cognitive Services
description: La transcripción de lotes es ideal si desea transcribir una gran cantidad de audio en el almacenamiento, como los blobs de Azure. Mediante la API REST dedicada, puede apuntar a archivos de audio con un identificador URI de firma de acceso compartido (SAS) y recibir las transcripciones de forma asincrónica.
services: cognitive-services
author: wolfma61
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/20/2020
ms.author: wolfma
ms.custom: devx-track-csharp
ms.openlocfilehash: 5be3b4ce5d89a15009f2b9b31183400890dbefb5
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88918951"
---
# <a name="how-to-use-batch-transcription"></a>Uso de la transcripción por lotes

La transcripción por lotes es un conjunto de operaciones de API REST que permite transcribir una gran cantidad de audio en el almacenamiento. Puede apuntar a archivos de audio mediante un identificador URI de firma de acceso compartido (SAS) y recibir los resultados de las transcripciones de forma asincrónica. Con la API v3.0, puede transcribir uno o varios archivos de audio o procesar un contenedor de almacenamiento completo.

Puede usar las API REST de transcripción por lotes para llamar a los métodos siguientes:

|    Operación de transcripción por lotes                                             |    Método    |    Llamada a API REST                                   |
|------------------------------------------------------------------------------|--------------|----------------------------------------------------|
|    Crea una transcripción nueva.                                              |    POST      |    speechtotext/v3.0/transcriptions            |
|    Recupera una lista de transcripciones para la suscripción autenticada.    |    GET       |    speechtotext/v3.0/transcriptions            |
|    Obtiene una lista de configuraciones regionales admitidas para las transcripciones sin conexión.              |    GET       |    speechtotext/v3.0/transcriptions/locales    |
|    Actualiza los detalles mutables de la transcripción identificada por su id.    |    PATCH     |    speechtotext/v3.0/transcriptions/{id}       |
|    Elimina la tarea de transcripción especificada.                                 |    Delete    |    speechtotext/v3.0/transcriptions/{id}       |
|    Obtiene la transcripción identificada por el id. especificado.                        |    GET       |    speechtotext/v3.0/transcriptions/{id}       |
|    Obtiene los archivos de resultado de la transcripción identificada por el id. especificado.    |    GET       |    speechtotext/v3.0/transcriptions/{id}/files |

Puede revisar y probar la API detallada, que está disponible como un [documento de Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0).

Esta API no requiere puntos de conexión personalizados y no tiene requisitos de simultaneidad.

Los trabajos de transcripción por lotes se programan de la mejor manera posible.
No es posible determinar el momento en que un trabajo cambiará al estado en ejecución, pero debería suceder en cuestión de minutos con una carga normal del sistema. Una vez que la transcripción tiene el estado "En ejecución", se produce más rápido que la velocidad de reproducción de audio en tiempo de ejecución.

## <a name="prerequisites"></a>Requisitos previos

Como sucede con todas las características del servicio Voz, puede crear una clave de suscripción en [Azure Portal](https://portal.azure.com) siguiendo nuestra [guía de inicio](get-started.md).

>[!NOTE]
> Se requiere una suscripción estándar (S0) para el servicio de voz para usar la transcripción de lotes. Las claves de suscripción gratuita (F0) no funcionan. Para obtener más información, consulte los [precios y límites](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

Si tiene previsto personalizar los modelos, siga los pasos descritos en [Personalización acústica](how-to-customize-acoustic-models.md) y [Personalización de idioma](how-to-customize-language-model.md). Para usar los modelos creados en la transcripción por lotes, necesita sus ubicaciones. Puede recuperar la ubicación del modelo al inspeccionar los detalles del modelo (propiedad `self`). *No es necesario* un punto de conexión personalizado implementado para el servicio de transcripción por lotes.

## <a name="batch-transcription-api"></a>API de transcripciones de Azure Batch

La API Batch Transcription admite los siguientes formatos:

| Formato | Códec | Bits por muestra | Velocidad de muestreo             |
|--------|-------|---------|---------------------------------|
| WAV    | PCM   | 16 bits  | 8 kHz o 16 kHz, mono o estéreo |
| MP3    | PCM   | 16 bits  | 8 kHz o 16 kHz, mono o estéreo |
| OGG    | OPUS  | 16 bits  | 8 kHz o 16 kHz, mono o estéreo |

En el caso de las secuencias de audio estéreo, los canales izquierdo y derecho se dividen durante la transcripción. Para cada canal se crea un archivo de resultados JSON.
Para crear una transcripción final ordenada, utilice las marcas de tiempo generadas por expresión.

### <a name="configuration"></a>Configuración

Los parámetros de configuración se proporcionan como JSON (uno o más archivos individuales):

```json
{
  "contentUrls": [
    "<URL to an audio file to transcribe>",
  ],
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "displayName": "Transcription of file using default model for en-US"
}
```

Los parámetros de configuración se proporcionan como JSON (se procesa un contenedor de almacenamiento completo):

```json
{
  "contentContainerUrl": "<SAS URL to the Azure blob container to transcribe>",
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "displayName": "Transcription of container using default model for en-US"
}
```

El siguiente archivo JSON especifica un modelo entrenado personalizado para usarlo en una transcripción por lotes:

```json
{
  "contentUrls": [
    "<URL to an audio file to transcribe>",
  ],
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "model": {
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/models/{id}"
  },
  "displayName": "Transcription of file using default model for en-US"
}
```


### <a name="configuration-properties"></a>Propiedades de configuración

Utilice estas propiedades opcionales para configurar la transcripción:

:::row:::
   :::column span="1":::
      **Parámetro**
   :::column-end:::
   :::column span="2":::
      **Descripción**
:::row-end:::
:::row:::
   :::column span="1":::
      `profanityFilterMode`
   :::column-end:::
   :::column span="2":::
      (Opcional) El valor predeterminado es `Masked`. Especifica cómo controlar las palabras soeces en los resultados del reconocimiento. Los valores aceptados son `None`, para deshabilitar el filtrado de palabras soeces; `Masked`, para reemplazar las palabras soeces con asteriscos; `Removed`, para quitar todas las palabras soeces del resultado; o `Tags`, para agregar etiquetas de "palabras soeces".
:::row-end:::
:::row:::
   :::column span="1":::
      `punctuationMode`
   :::column-end:::
   :::column span="2":::
      (Opcional) El valor predeterminado es `DictatedAndAutomatic`. Especifica cómo controlar la puntuación en los resultados del reconocimiento. Los valores aceptados son `None`, para deshabilitar la puntuación; `Dictated`, para implicar signos de puntuación explícitos (hablados); `Automatic`, para permitir que el descodificador trate con signos de puntuación o `DictatedAndAutomatic`, para usar la puntuación dictada y automática.
:::row-end:::
:::row:::
   :::column span="1":::
      `wordLevelTimestampsEnabled`
   :::column-end:::
   :::column span="2":::
      (Opcional) `false` es el valor predeterminado. Especifica si las marcas de tiempo de nivel de palabra se deben agregar a la salida.
:::row-end:::
:::row:::
   :::column span="1":::
      `diarizationEnabled`
   :::column-end:::
   :::column span="2":::
      (Opcional) `false` es el valor predeterminado. Especifica que el análisis de la diarización se debe llevar a cabo en la entrada que se espera sea un canal mono que contenga dos voces. Nota: Se requiere que `wordLevelTimestampsEnabled` se establezca en `true`.
:::row-end:::
:::row:::
   :::column span="1":::
      `channels`
   :::column-end:::
   :::column span="2":::
      (Opcional) `0` y `1` se transcriben de forma predeterminada. Matriz de números de canal para procesar. Aquí se puede especificar un subconjunto de los canales disponibles en el archivo de audio para procesarse (por ejemplo, solo `0`).
:::row-end:::
:::row:::
   :::column span="1":::
      `timeToLive`
   :::column-end:::
   :::column span="2":::
      (Opcional) No se elimina de forma predeterminada. Plazo para eliminar automáticamente las transcripciones después de completar el proceso de transcripción. `timeToLive` resulta útil en el procesamiento en masa de transcripciones, para asegurarse de que se eliminarán (por ejemplo, `PT12H` para 12 horas).
:::row-end:::
:::row:::
   :::column span="1":::
      `destinationContainerUrl`
   :::column-end:::
   :::column span="2":::
      URL opcional con [SAS de servicio](../../storage/common/storage-sas-overview.md) en un contenedor grabable de Azure. El resultado se almacena en este contenedor. Cuando no se especifica, Microsoft almacena los resultados en un contenedor de almacenamiento administrado por Microsoft. Cuando se elimina la transcripción mediante una llamada a [Delete transcription](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/DeleteTranscription), también se eliminan los datos del resultado.
:::row-end:::

### <a name="storage"></a>Storage

La transcripción por lotes puede leer audio desde un URI de Internet visible públicamente, además de leer transcripciones de audio o de escritura mediante un URI de SAS con [Azure Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blobs-overview).

## <a name="batch-transcription-result"></a>Resultado de la transcripción por lotes

Para cada entrada de audio, se crea un archivo de resultado de transcripción.
La operación [Get transcripciones files](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetTranscriptionFiles) devuelve una lista de archivos de resultados para esta transcripción. Para buscar el archivo de transcripción de un archivo de entrada específico, filtre todos los archivos devueltos con `kind` == `Transcription` y `name` == `{originalInputName.suffix}.json`.

Cada uno de los archivos de resultado de transcripción tiene el siguiente formato:

```json
{
  "source": "...",                      // sas url of a given contentUrl or the path relative to the root of a given container
  "timestamp": "2020-06-16T09:30:21Z",  // creation time of the transcription, ISO 8601 encoded timestamp, combined date and time
  "durationInTicks": 41200000,          // total audio duration in ticks (1 tick is 100 nanoseconds)
  "duration": "PT4.12S",                // total audio duration, ISO 8601 encoded duration
  "combinedRecognizedPhrases": [        // concatenated results for simple access in single string for each channel
    {
      "channel": 0,                     // channel number of the concatenated results
      "lexical": "hello world",
      "itn": "hello world",
      "maskedITN": "hello world",
      "display": "Hello world."
    }
  ],
  "recognizedPhrases": [                // results for each phrase and each channel individually
    {
      "recognitionStatus": "Success",   // recognition state, e.g. "Success", "Failure"
      "channel": 0,                     // channel number of the result
      "offset": "PT0.07S",              // offset in audio of this phrase, ISO 8601 encoded duration 
      "duration": "PT1.59S",            // audio duration of this phrase, ISO 8601 encoded duration
      "offsetInTicks": 700000.0,        // offset in audio of this phrase in ticks (1 tick is 100 nanoseconds)
      "durationInTicks": 15900000.0,    // audio duration of this phrase in ticks (1 tick is 100 nanoseconds)
      
      // possible transcriptions of the current phrase with confidences
      "nBest": [
        {
          "confidence": 0.898652852,    // confidence value for the recognition of the whole phrase
          "speaker": 1,                 // if `diarizationEnabled` is `true`, this is the identified speaker (1 or 2), otherwise this property is not present
          "lexical": "hello world",
          "itn": "hello world",
          "maskedITN": "hello world",
          "display": "Hello world.",
          
          // if wordLevelTimestampsEnabled is `true`, there will be a result for each word of the phrase, otherwise this property is not present
          "words": [
            {
              "word": "hello",
              "offset": "PT0.09S",
              "duration": "PT0.48S",
              "offsetInTicks": 900000.0,
              "durationInTicks": 4800000.0,
              "confidence": 0.987572
            },
            {
              "word": "world",
              "offset": "PT0.59S",
              "duration": "PT0.16S",
              "offsetInTicks": 5900000.0,
              "durationInTicks": 1600000.0,
              "confidence": 0.906032
            }
          ]
        }
      ]    
    }
  ]
}
```

El resultado contiene los siguientes campos:

:::row:::
   :::column span="1":::
      **Campo**
   :::column-end:::
   :::column span="2":::
      **Contenido**
:::row-end:::
:::row:::
   :::column span="1":::
      `lexical`
   :::column-end:::
   :::column span="2":::
      Las palabras reales reconocidas.
:::row-end:::
:::row:::
   :::column span="1":::
      `itn`
   :::column-end:::
   :::column span="2":::
      El formato de normalización inversa de texto del texto reconocido. Se aplican las abreviaturas ("doctor Pérez" a "Dr Pérez"), los números de teléfono y otras transformaciones.
:::row-end:::
:::row:::
   :::column span="1":::
      `maskedITN`
   :::column-end:::
   :::column span="2":::
      El formato ITN con enmascaramiento de palabras soeces aplicado.
:::row-end:::
:::row:::
   :::column span="1":::
      `display`
   :::column-end:::
   :::column span="2":::
      El formato mostrado del texto reconocido. Se incluyen el uso de mayúsculas y minúsculas y la puntuación que se agrega.
:::row-end:::

## <a name="speaker-separation-diarization"></a>Separación de altavoces (diarización)

La diarización es el proceso de separación de los altavoces en una parte del audio. La canalización por lotes admite la diarización y es capaz de reconocer dos altavoces en grabaciones monocanal. La característica no está disponible en las grabaciones estéreo.

La salida de la transcripción con la opción de diarización habilitada contiene una entrada `Speaker` para cada frase transcrita. Si no se usa la diarización, la propiedad `Speaker` no está presente en la salida del archivo JSON. Para la diarización, se admiten dos voces, por lo que los oradores se identifican como `1` o `2`.

Para solicitar la diarización, establezca la propiedad `diarizationEnabled` en `true`, tal y como se muestra en la solicitud HTTP a continuación.

 ```json
{
  "contentUrls": [
    "<URL to an audio file to transcribe>",
  ],
  "properties": {
    "diarizationEnabled": true,
    "wordLevelTimestampsEnabled": true,
    "punctuationMode": "DictatedAndAutomatic",
    "profanityFilterMode": "Masked"
  },
  "locale": "en-US",
  "displayName": "Transcription of file using default model for en-US"
}
```

Las marcas de tiempo de nivel de palabra deben habilitarse, como indican los parámetros de la solicitud anterior.

## <a name="best-practices"></a>Procedimientos recomendados

El servicio transcripción por lotes puede manejar un elevado número de transcripciones enviadas. Puede consultar el estado de las transcripciones mediante [Get transcriptions](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetTranscriptions).
Realice una llamada a [Delete transcription](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/DeleteTranscription) periódicamente desde el servicio cuando se hayan recuperado los resultados. También puede establecer la propiedad `timeToLive` para garantizar que los resultados se eliminen eventualmente.

## <a name="sample-code"></a>Código de ejemplo

Hay ejemplos completos disponibles en el [repositorio de ejemplos de GitHub](https://aka.ms/csspeech/samples) dentro del subdirectorio `samples/batch`.

Si utiliza un modelo personalizado, actualice el código de ejemplo con la información de suscripción, la región del servicio, el URI que apunta al archivo de audio que se va a transcribir y la ubicación del modelo.

[!code-csharp[Configuration variables for batch transcription](~/samples-cognitive-services-speech-sdk/samples/batch/csharp/program.cs#transcriptiondefinition)]

El código de ejemplo configura el cliente y envía la solicitud de transcripción. A continuación, sondea la información de estado e imprime los detalles sobre el progreso de la transcripción.

[!code-csharp[Code to check batch transcription status](~/samples-cognitive-services-speech-sdk/samples/batch/csharp/program.cs#transcriptionstatus)]

Para más información sobre las llamadas anteriores, consulte nuestro [documento de Swagger](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0). Para ver el ejemplo completo aquí mostrado, vaya a [GitHub](https://aka.ms/csspeech/samples) en el subdirectorio `samples/batch`.

En este ejemplo se usa una configuración asincrónica para publicar audio y recibir el estado de la transcripción.
El método `PostTranscriptions` envía los detalles del archivo de audio y el método `GetTranscriptions` recibe los estados.
`PostTranscriptions` devuelve un identificador y `GetTranscriptions` lo usa para crear un identificador y obtener el estado de la transcripción.

Este código de ejemplo no especifica un modelo personalizado. El servicio usa el modelo base de referencia para transcribir los archivos. Para especificar el modelo, puede pasar la referencia de modelo del modelo personalizado en el mismo método.

> [!NOTE]
> En el caso de las transcripciones de base de referencia, no es necesario declarar los identificadores del modelo de base de referencia.

## <a name="next-steps"></a>Pasos siguientes

- [Referencia de la API Conversión de voz en texto v3](https://centralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/CopyModelToSubscription)
