---
title: 'Referencia de la API de conversión de texto a voz (REST): servicio de voz'
titleSuffix: Azure Cognitive Services
description: Aprenda a usar la API de REST Text-to-Speech. En este artículo, obtendrá más información sobre las opciones de autorización y de consulta y sobre cómo estructurar una solicitud y recibir una respuesta.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/23/2020
ms.author: trbye
ms.openlocfilehash: 6fafb668ecc2ae36dbe5a6bbc3d1e1d501545b50
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88056812"
---
# <a name="text-to-speech-rest-api"></a>Text-to-speech REST API

El servicio de voz le permite [convertir texto en voz sintetizada](#convert-text-to-speech) y [obtener una lista de voces admitidas](#get-a-list-of-voices) para una región con un conjunto de API REST. Cada punto de conexión disponible se asocia con una región. Se requiere una clave de suscripción para el punto de conexión o región que se va a usar.

Text to Speech REST API admite voces neuronales y de texto a voz estándar, y cada una de ellas admite un idioma y un dialecto específicos, que se identifican mediante la configuración regional.

* Para ver una lista completa de voces, consulte [compatibilidad con idiomas](language-support.md#text-to-speech).
* Para obtener información acerca de la disponibilidad regional, consulte las [regiones](regions.md#text-to-speech).

> [!IMPORTANT]
> Los costes varían para voces estándar, personalizadas y neuronales. Para obtener más información, consulte el apartado [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

Antes de utilizar esta API, comprenda:

* La API REST de texto a voz requiere un encabezado de autorización. Esto significa que tiene que completar un intercambio de tokens para acceder al servicio. Para más información, consulte [Autenticación](#authentication).

[!INCLUDE [](../../../includes/cognitive-services-speech-service-rest-auth.md)]

## <a name="get-a-list-of-voices"></a>Obtener una lista de voces

El punto de conexión `voices/list` le permite obtener una lista completa de las voces de una región o punto de conexión en concreto.

### <a name="regions-and-endpoints"></a>Regiones y puntos de conexión

| Region | Punto de conexión |
|--------|----------|
| Este de Australia | `https://australiaeast.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Sur de Brasil | `https://brazilsouth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro de Canadá | `https://canadacentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro de EE. UU. | `https://centralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Este de Asia | `https://eastasia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Este de EE. UU. | `https://eastus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Este de EE. UU. 2 | `https://eastus2.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro de Francia | `https://francecentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| India central | `https://centralindia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Japón Oriental | `https://japaneast.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro de Corea del Sur | `https://koreacentral.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro-Norte de EE. UU | `https://northcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Norte de Europa | `https://northeurope.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Centro-sur de EE. UU. | `https://southcentralus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Sudeste de Asia | `https://southeastasia.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Sur de Reino Unido 2 | `https://uksouth.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Oeste de Europa | `https://westeurope.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Oeste de EE. UU. | `https://westus.tts.speech.microsoft.com/cognitiveservices/voices/list` |
| Oeste de EE. UU. 2 | `https://westus2.tts.speech.microsoft.com/cognitiveservices/voices/list` |

### <a name="request-headers"></a>Encabezados de solicitud

En esta tabla se enumeran los encabezados obligatorios y opcionales para las solicitudes de texto a voz.

| Encabezado | Descripción | Obligatorio u opcional |
|--------|-------------|---------------------|
| `Authorization` | Un token de autorización precedido por la palabra `Bearer`. Para más información, consulte [Autenticación](#authentication). | Obligatorio |

### <a name="request-body"></a>Cuerpo de la solicitud

No es necesario un cuerpo para las solicitudes `GET` a este punto de conexión.

### <a name="sample-request"></a>Solicitud de ejemplo

Esta solicitud solo requiere un encabezado de autorización.

```http
GET /cognitiveservices/voices/list HTTP/1.1

Host: westus.tts.speech.microsoft.com
Authorization: Bearer [Base64 access_token]
```

### <a name="sample-response"></a>Respuesta de muestra

Esta respuesta se ha truncado para ilustrar la estructura de una respuesta.

> [!NOTE]
> La disponibilidad de voces varía según la región o el punto de conexión.

```json
[
    {
        "Name": "Microsoft Server Speech Text to Speech Voice (ar-EG, Hoda)",
        "ShortName": "ar-EG-Hoda",
        "Gender": "Female",
        "Locale": "ar-EG",
        "SampleRateHertz": "16000",
        "VoiceType": "Standard"
    },
    {
        "Name": "Microsoft Server Speech Text to Speech Voice (ar-SA, Naayf)",
        "ShortName": "ar-SA-Naayf",
        "Gender": "Male",
        "Locale": "ar-SA",
        "SampleRateHertz": "16000",
        "VoiceType": "Standard"
    },
    {
        "Name": "Microsoft Server Speech Text to Speech Voice (bg-BG, Ivan)",
        "ShortName": "bg-BG-Ivan",
        "Gender": "Male",
        "Locale": "bg-BG",
        "SampleRateHertz": "16000",
        "VoiceType": "Standard"
    },
    {
        "Name": "Microsoft Server Speech Text to Speech Voice (ca-ES, HerenaRUS)",
        "ShortName": "ca-ES-HerenaRUS",
        "Gender": "Female",
        "Locale": "ca-ES",
        "SampleRateHertz": "16000",
        "VoiceType": "Standard"
    },
    {
        "Name": "Microsoft Server Speech Text to Speech Voice (zh-CN, XiaoxiaoNeural)",
        "ShortName": "zh-CN-XiaoxiaoNeural",
        "Gender": "Female",
        "Locale": "zh-CN",
        "SampleRateHertz": "24000",
        "VoiceType": "Neural"
    },

    ...
]
```

### <a name="http-status-codes"></a>Códigos de estado HTTP

El estado HTTP de cada respuesta indica estados de corrección o error comunes.

| Código de estado HTTP | Descripción | Posible motivo |
|------------------|-------------|-----------------|
| 200 | Aceptar | La solicitud fue correcta. |
| 400 | Bad Request | Falta un parámetro requerido, está vacío o es nulo. O bien, el valor pasado a un parámetro obligatorio u opcional no es válido. Un problema común es que el encabezado sea demasiado largo. |
| 401 | No autorizado | La solicitud no está autenticada. Asegúrese de que la clave de suscripción o el token sean válidos y de la región correcta. |
| 429 | Demasiadas solicitudes | Ha superado la cuota o la tasa de solicitudes permitidas para su suscripción. |
| 502 | Puerta de enlace incorrecta    | Problema de red o de servidor. Podría indicar también encabezados no válidos. |


## <a name="convert-text-to-speech"></a>Conversión de texto a voz

El punto de conexión `v1` le permite convertir texto a voz usando [lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md).

### <a name="regions-and-endpoints"></a>Regiones y puntos de conexión

Estas regiones son compatibles con la conversión de texto a voz mediante la API REST. Asegúrese de que selecciona el punto de conexión que coincida con la región de su suscripción.

[!INCLUDE [](../../../includes/cognitive-services-speech-service-endpoints-text-to-speech.md)]

### <a name="request-headers"></a>Encabezados de solicitud

En esta tabla se enumeran los encabezados obligatorios y opcionales para las solicitudes de texto a voz.

| Encabezado | Descripción | Obligatorio u opcional |
|--------|-------------|---------------------|
| `Authorization` | Un token de autorización precedido por la palabra `Bearer`. Para más información, consulte [Autenticación](#authentication). | Obligatorio |
| `Content-Type` | Especifica el tipo de contenido para el texto proporcionado. Valor aceptable: `application/ssml+xml`. | Obligatorio |
| `X-Microsoft-OutputFormat` | Especifica el formato de salida del audio. Para obtener una lista completa de los valores aceptados, consulte [salidas de audio](#audio-outputs). | Obligatorio |
| `User-Agent` | Nombre de la aplicación. El valor proporcionado debe tener menos de 255 caracteres. | Obligatorio |

### <a name="audio-outputs"></a>Salidas de audio

Esta es una lista de formatos de audio admitidos que se envían en cada solicitud como encabezado `X-Microsoft-OutputFormat`. Cada uno de ellos incorpora una velocidad de bits y el tipo de codificación. El servicio de voz admite salidas de audio de 24 kHz, 16 kHz y 8 kHz.

```output
raw-16khz-16bit-mono-pcm            raw-8khz-8bit-mono-mulaw
riff-8khz-8bit-mono-alaw            riff-8khz-8bit-mono-mulaw
riff-16khz-16bit-mono-pcm           audio-16khz-128kbitrate-mono-mp3
audio-16khz-64kbitrate-mono-mp3     audio-16khz-32kbitrate-mono-mp3
raw-24khz-16bit-mono-pcm            riff-24khz-16bit-mono-pcm
audio-24khz-160kbitrate-mono-mp3    audio-24khz-96kbitrate-mono-mp3
audio-24khz-48kbitrate-mono-mp3     ogg-24khz-16bit-mono-opus
```

> [!NOTE]
> Si la voz y el formato de salida seleccionados tienen velocidades de bits diferentes, se vuelve a muestrear el audio según sea necesario. ogg-24khz-16bit-mono-opus se puede descodificar con el [códec opus](https://opus-codec.org/downloads/)

### <a name="request-body"></a>Cuerpo de la solicitud

El cuerpo de cada solicitud `POST` se envía como [lenguaje de marcado de síntesis de voz (SSML)](speech-synthesis-markup.md). SSML le permite elegir la voz y el idioma de la voz sintetizada que devuelve el servicio de texto a voz. Para ver una lista completa de voces compatibles, consulte [compatibilidad con idiomas](language-support.md#text-to-speech).

> [!NOTE]
> Si usa una voz personalizada, el cuerpo de una solicitud puede enviarse como texto sin formato (ASCII o UTF-8).

### <a name="sample-request"></a>Solicitud de ejemplo

Esta solicitud HTTP utiliza SSML para especificar el idioma y la voz. Si la longitud del cuerpo es larga y el audio resultante supera los 10 minutos, se trunca a 10 minutos. En otras palabras, la longitud del audio no puede superar los 10 minutos.

```http
POST /cognitiveservices/v1 HTTP/1.1

X-Microsoft-OutputFormat: raw-16khz-16bit-mono-pcm
Content-Type: application/ssml+xml
Host: westus.tts.speech.microsoft.com
Content-Length: 225
Authorization: Bearer [Base64 access_token]

<speak version='1.0' xml:lang='en-US'><voice xml:lang='en-US' xml:gender='Female'
    name='en-US-AriaRUS'>
        Microsoft Speech Service Text-to-Speech API
</voice></speak>
```

Consulte nuestras guías de inicio rápido para ver ejemplos específicos del idioma:

* [.NET Core, C#](~/articles/cognitive-services/Speech-Service/quickstarts/text-to-speech.md?pivots=programming-language-csharp&tabs=dotnetcore)
* [Python](~/articles/cognitive-services/Speech-Service/quickstarts/text-to-speech.md?pivots=programming-language-python)
* [Node.js](quickstart-nodejs-text-to-speech.md)

### <a name="http-status-codes"></a>Códigos de estado HTTP

El estado HTTP de cada respuesta indica estados de corrección o error comunes.

| Código de estado HTTP | Descripción | Posible motivo |
|------------------|-------------|-----------------|
| 200 | Aceptar | La solicitud es correcta; el cuerpo de la respuesta es un archivo de audio. |
| 400 | Bad Request | Falta un parámetro requerido, está vacío o es nulo. O bien, el valor pasado a un parámetro obligatorio u opcional no es válido. Un problema común es que el encabezado sea demasiado largo. |
| 401 | No autorizado | La solicitud no está autenticada. Asegúrese de que la clave de suscripción o el token sean válidos y de la región correcta. |
| 413 | Entidad de solicitud demasiado larga | La entrada de SSML tiene más de 1024 caracteres. |
| 415 | Tipo de medio no compatible | Es posible que se haya proporcionado el `Content-Type` incorrecto. `Content-Type` se debe establecer en `application/ssml+xml`. |
| 429 | Demasiadas solicitudes | Ha superado la cuota o la tasa de solicitudes permitidas para su suscripción. |
| 502 | Puerta de enlace incorrecta    | Problema de red o de servidor. Podría indicar también encabezados no válidos. |

Si el estado HTTP es `200 OK`, el cuerpo de la respuesta contiene un archivo de audio en el formato solicitado. Este archivo se puede reproducir mientras se transfiere o guardarse en un búfer o en un archivo.

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una cuenta de Azure gratuita](https://azure.microsoft.com/free/cognitive-services/)
- [Síntesis asincrónica para audio de formato largo](quickstarts/text-to-speech/async-synthesis-long-form-audio.md)
- [Introducción a Voz personalizada](how-to-custom-voice.md)
