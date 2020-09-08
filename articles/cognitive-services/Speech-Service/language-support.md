---
title: 'Compatibilidad con idiomas: servicio de voz'
titleSuffix: Azure Cognitive Services
description: El servicio de voz admite un gran número de idiomas para la conversión de texto a voz y voz a texto, junto con la traducción de voz. En este artículo se proporciona una lista completa de idiomas compatibles por servicio.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/26/2020
ms.author: trbye
ms.custom: seodec18
ms.openlocfilehash: 19f4e7b340f0b1ceb64d04a2704a4f6667e1e14a
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88053582"
---
# <a name="language-and-voice-support-for-the-speech-service"></a>Compatibilidad con idiomas y voces en el servicio de voz

La compatibilidad con los idiomas varía según la funcionalidad del servicio de voz. En las tablas siguientes se resume la compatibilidad con idiomas para la [conversión de voz a texto](#speech-to-text), [conversión de texto a voz](#text-to-speech) y las ofertas del servicio [Speech Translation](#speech-translation).

## <a name="speech-to-text"></a>Voz a texto

El SDK de Voz de Microsoft y la API REST admiten los siguientes idiomas (configuraciones regionales). 

Para mejorar la precisión, se ofrece la posibilidad de personalizar un subconjunto de idiomas mediante la carga de **audio y transcripciones etiquetadas por humanos** o **texto relacionado: oraciones**. Para más información sobre la personalización, consulte [Introducción a Custom Speech](how-to-custom-speech.md).

<!--
To get the AM and ML bits:
https://westus.cris.ai/swagger/ui/index#/Custom%20Speech%20models%3A/GetSupportedLocalesForModels

To get pronunciation bits:
https://cris.ai -> Click on Adaptation Data -> scroll down to section "Pronunciation Datasets" -> Click on Import -> Locale: the list of locales there correspond to the supported locales
-->

| Configuración regional  | Idioma                          | Personalizaciones                                    |
|---------|-----------------------------------|---------------------------------------------------|
| `ar-AE` | Árabe (Emiratos Árabes Unidos)                      | No                                                |
| `ar-BH` | Árabe (Bahréin), estándar moderno | Modelo de lenguaje                                    |
| `ar-EG` | Árabe (Egipto)                    | Modelo de lenguaje                                    |
| `ar-IL` | Árabe (Israel)                   | No                                                |
| `ar-JO` | Árabe (Jordania)                   | No                                                |
| `ar-KW` | Árabe (Kuwait)                   | No                                                |
| `ar-LB` | Árabe (Líbano)                  | No                                                |
| `ar-PS` | Árabe (Palestina)                | No                                                |
| `ar-QA` | Árabe (Qatar)                    | No                                                |
| `ar-SA` | Árabe (Arabia Saudí)             | No                                                |
| `ar-SY` | Árabe (Siria)                    | Modelo de lenguaje                                    |
| `ca-ES` | Catalán                           | Modelo de lenguaje                                    |
| `cs-CZ` | Checo (República Checa)            | Modelo de lenguaje                                    | 
| `da-DK` | Danés (Dinamarca)                  | Modelo de lenguaje                                    |
| `de-DE` | Alemán (Alemania)                  | Modelo acústico<br>Modelo de lenguaje<br>Pronunciación |
| `en-AU` | Inglés (Australia)               | Modelo acústico<br>Modelo de lenguaje                  |
| `en-CA` | Inglés (Canadá)                  | Modelo acústico<br>Modelo de lenguaje                  |
| `en-GB` | Inglés (Reino Unido)          | Modelo acústico<br>Modelo de lenguaje<br>Pronunciación |
| `en-HK` | Inglés (RAE de Hong Kong)               | Modelo de lenguaje                                    | 
| `en-IE` | Inglés (Irlanda)                 | Modelo de lenguaje                                    | 
| `en-IN` | Inglés (India)                   | Modelo acústico<br>Modelo de lenguaje                  |
| `en-NZ` | Inglés (Nueva Zelanda)             | Modelo acústico<br>Modelo de lenguaje                  |
| `en-PH` | Inglés (Filipinas)             | Modelo de lenguaje                                    | 
| `en-SG` | Inglés (Singapur)               | Modelo de lenguaje                                    | 
| `en-US` | Spanish (Traditional Sort) - Spain           | Modelo acústico<br>Modelo de lenguaje<br>Pronunciación |
| `en-ZA` | Inglés (Sudáfrica)            | Modelo de lenguaje                                    | 
| `es-AR` | Español (Argentina)               | Modelo de lenguaje                                    | 
| `es-BO` | Español (Bolivia)                 | Modelo de lenguaje                                    | 
| `es-CL` | Español (Chile)                   | Modelo de lenguaje                                    | 
| `es-CO` | Español (Colombia)                | Modelo de lenguaje                                    | 
| `es-CR` | Español (Costa Rica)              | Modelo de lenguaje                                    | 
| `es-CU` | Español (Cuba)                    | Modelo de lenguaje                                    | 
| `es-DO` | Español (República Dominicana)      | Modelo de lenguaje                                    | 
| `es-EC` | Español (Ecuador)                 | Modelo de lenguaje                                    | 
| `es-ES` | Español (España)                   | Modelo acústico<br>Modelo de lenguaje                  |
| `es-GT` | Español (Guatemala)               | Modelo de lenguaje                                    | 
| `es-HN` | Español (Honduras)                | Modelo de lenguaje                                    | 
| `es-MX` | Español (México)                  | Modelo acústico<br>Modelo de lenguaje                  |
| `es-NI` | Español (Nicaragua)               | Modelo de lenguaje                                    | 
| `es-PA` | Español (Panamá)                  | Modelo de lenguaje                                    | 
| `es-PE` | Español (Perú)                    | Modelo de lenguaje                                    | 
| `es-PR` | Español (Puerto Rico)             | Modelo de lenguaje                                    | 
| `es-PY` | Español (Paraguay)                | Modelo de lenguaje                                    | 
| `es-SV` | Español (El Salvador)             | Modelo de lenguaje                                    | 
| `es-US` | Español (EE. UU.)                     | Modelo de lenguaje                                    | 
| `es-UY` | Español (Uruguay)                 | Modelo de lenguaje                                    | 
| `es-VE` | Español (Venezuela)               | Modelo de lenguaje                                    | 
| `fi-FI` | Finés (Finlandia)                 | Modelo de lenguaje                                    |
| `fr-CA` | Francés (Canadá)                   | Modelo acústico<br>Modelo de lenguaje                  |
| `fr-FR` | Francés (Francia)                   | Modelo acústico<br>Modelo de lenguaje<br>Pronunciación |
| `gu-IN` | Gujarati (India)                 | Modelo de lenguaje                                    |
| `hi-IN` | Hindi (India)                     | Modelo acústico<br>Modelo de lenguaje                  |
| `hu-HU` | Húngaro (Hungría)               | Modelo de lenguaje                                    | 
| `it-IT` | Italiano (Italia)                   | Modelo acústico<br>Modelo de lenguaje<br>Pronunciación |
| `ja-JP` | Japonés (Japón)                  | Modelo de lenguaje                                    |
| `ko-KR` | Coreano (Corea)                    | Modelo de lenguaje                                    |
| `mr-IN` | Maratí (India)                   | Modelo de lenguaje                                    |
| `nb-NO` | Noruego, Bokmål (Noruego)       | Modelo de lenguaje                                    |
| `nl-NL` | Neerlandés (Países Bajos)               | Modelo de lenguaje                                    |
| `pl-PL` | Polaco (Polonia)                   | Modelo de lenguaje                                    |
| `pt-BR` | Portugués (Brasil)               | Modelo acústico<br>Modelo de lenguaje<br>Pronunciación |
| `pt-PT` | Portugués (Portugal)             | Modelo de lenguaje                                    |
| `ru-RU` | Ruso (Rusia)                  | Modelo acústico<br>Modelo de lenguaje                  |
| `sv-SE` | Sueco (Suecia)                  | Modelo de lenguaje                                    |
| `ta-IN` | Tamil (India)                     | Modelo de lenguaje                                    |
| `te-IN` | Telugu (India)                    | Modelo de lenguaje                                    |
| `th-TH` | Tailandés (Tailandia)                   | No                                                |
| `tr-TR` | Turco (Turquía)                  | Modelo de lenguaje                                    |
| `zh-CN` | Chino (mandarín, simplificado)    | Modelo acústico<br>Modelo de lenguaje                  |
| `zh-HK` | Chino (cantonés, tradicional)  | Modelo de lenguaje                                    |
| `zh-TW` | Chino (mandarín, Taiwán)      | Modelo de lenguaje                                    |

## <a name="text-to-speech"></a>Texto a voz

Tanto el SDK de Voz de Microsoft como las API REST admiten estas voces, y cada una de ellas admite un idioma y un dialecto específicos, que se identifican mediante la configuración regional. También puede obtener una lista completa de los idiomas y las voces que se admiten para cada región o punto de conexión específico a través de [voices/list API](rest-text-to-speech.md#get-a-list-of-voices). 

> [!IMPORTANT]
> Los precios son distintos para voces estándar, personalizadas y neuronales. Consulte la página de [precios](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/) para más información.

### <a name="neural-voices"></a>Voces neuronales

Texto a voz neuronal es un nuevo tipo de síntesis de voz con tecnología de redes neuronales profundas. Cuando se usa una voz neuronal, es prácticamente imposible distinguir la voz sintetizada de las grabaciones humanas.

Las voces neuronales se pueden usar para que las interacciones con los bots de chat y los asistentes de voz sean más naturales y atractivas, para convertir textos digitales (por ejemplo, los libros electrónicos) en audiolibros y para mejorar los sistemas de navegación de los automóviles. Con su prosodia natural similar a la humana y la clara articulación de las palabras, las voces neuronales reducen considerablemente la fatiga de la escucha que aparece cuando los usuarios interactúan con sistemas de inteligencia artificial.

Para más información acerca de la disponibilidad regional, consulte las [regiones](regions.md#standard-and-neural-voices).

|Configuración regional  | Idioma            | Sexo | Nombre de voz | Compatibilidad de estilo |
|--|--|--|--|--|
| `ar-EG` | Árabe (Egipto)                  | Female | `ar-EG-SalmaNeural`      | General |
| `ar-SA` | Árabe (Arabia Saudí)           | Female | `ar-SA-ZariyahNeural`    | General |
| `ca-ES` | Catalán (España)                 | Female | `ca-ES-AlbaNeural`       | General |
| `da-DK` | Danés (Dinamarca)                | Female | `da-DK-ChristelNeural`   | General |
| `de-DE` | Alemán (Alemania)                | Female | `de-DE-KatjaNeural`      | General |
| `en-AU` | Inglés (Australia)             | Female | `en-AU-NatashaNeural`    | General |
| `en-CA` | Inglés (Canadá)                | Female | `en-CA-ClaraNeural`      | General |
| `en-GB` | English (Reino Unido)                    | Female | `en-GB-LibbyNeural`      | General |
|         |                                 | Female | `en-GB-MiaNeural`        | General |
| `en-IN` | Inglés (India)                 | Female | `en-IN-NeerjaNeural`     | General |
| `en-US` | Inglés (EE. UU.)                    | Female | `en-US-AriaNeural`       | General, varios estilos de voz disponibles |
|         |                                 | Male   | `en-US-GuyNeural`        | General |
| `es-ES` | Español (España)                 | Female | `es-ES-ElviraNeural`     | General |
| `es-MX` | Español (México)                | Female | `es-MX-DaliaNeural`      | General |
| `fi-FI` | Finés (Finlandia)               | Female | `fi-FI-NooraNeural`      | General |
| `fr-CA` | Francés (Canadá)                 | Female | `fr-CA-SylvieNeural`     | General |
| `fr-FR` | Francés (Francia)                 | Female | `fr-FR-DeniseNeural`     | General |
| `hi-IN` | Hindi (India)                   | Female | `hi-IN-SwaraNeural`      | General |
| `it-IT` | Italiano (Italia)                 | Female | `it-IT-ElsaNeural`       | General |
| `ja-JP` | Japonés                        | Female | `ja-JP-NanamiNeural`     | General |
| `ko-KR` | Coreano                          | Female | `ko-KR-SunHiNeural`      | General |
| `nb-NO` | Noruego                       | Female | `nb-NO-IselinNeural`     | General |
| `nl-NL` | Neerlandés (Países Bajos)              | Female | `nl-NL-ColetteNeural`    | General |
| `pl-PL` | Polaco (Polonia)                 | Female | `pl-PL-ZofiaNeural`      | General |
| `pt-BR` | Portugués (Brasil)             | Female | `pt-BR-FranciscaNeural`  | General, varios estilos de voz disponibles |
| `tr-TR` | Turco                         | Female | `tr-TR-EmelNeural`       | General |
| `pt-PT` | Portugués (Portugal)           | Female | `pt-PT-FernandaNeural`   | General |
| `ru-RU` | Ruso (Rusia)                | Female | `ru-RU-DariyaNeural`     | General |
| `sv-SE` | Sueco (Suecia)                | Female | `sv-SE-HilleviNeural`    | General |
| `th-TH` | Tailandés (Tailandia)                 | Female | `th-TH-AcharaNeural`     | General |
| `zh-CN` | Chino (mandarín, simplificado)  | Female | `zh-CN-XiaoxiaoNeural`   | General, varios estilos de voz disponibles |
|         |                                 | Female | `zh-CN-XiaoyouNeural`    | Voz de niño, optimizado para la narración de historias |
|         |                                 | Male   | `zh-CN-YunyangNeural`    | Optimizado para la lectura de noticias, varios estilos de voz disponibles |
|         |                                 | Male   | `zh-CN-YunyeNeural`      | Optimizado para la narración de historias |
| `zh-HK` | Chino (cantonés, tradicional)   | Female | `zh-HK-HiuGaaiNeural`| General |
| `zh-TW` | Chino (mandarín, Taiwán)   | Female | `zh-TW-HsiaoYuNeural`    | General |

> [!IMPORTANT]
> La voz `en-US-JessaNeural` ha cambiado a `en-US-AriaNeural`. Si usaba "Jessa" antes, conviértalo a "Aria".

Para obtener información sobre cómo configurar y ajustar las voces neuronales, consulte [Lenguaje de marcado de síntesis de voz](speech-synthesis-markup.md#adjust-speaking-styles).

> [!TIP]
> Puede seguir usando la asignación de nombre completo de servicio, como "Microsoft Server Speech Text to Speech Voice (en-US, AriaNeural)", en sus solicitudes de síntesis de voz.

### <a name="standard-voices"></a>Voces estándar

Hay más de 75 voces estándar disponibles en más de 45 idiomas y configuraciones regionales, lo que le permite convertir texto en voz sintetizada. Para más información acerca de la disponibilidad regional, consulte las [regiones](regions.md#standard-and-neural-voices).

| Configuración regional | Idioma | Sexo | Nombre de voz |
|--|--|--|--|
| <sup>1</sup>`ar-EG` | Árabe (Egipto) | Female | "ar-EG-Hoda" |
| `ar-SA` | Árabe (Arabia Saudí) | Male | "ar-SA-Naayf" |
| `bg-BG` | Búlgaro | Male |  "bg-BG-Ivan" |
| `ca-ES` | Catalán | Female |  "ca-ES-HerenaRUS" |
| `cs-CZ` | Checo | Male | "cs-CZ-Jakub" |
| `da-DK` | Danés | Female |  "da-DK-HelleRUS" |
| `de-AT` | Alemán (Austria) | Male | "de-AT-Michael" |
| `de-CH` | Alemán (Suiza) | Male |  "de-CH-Karsten" |
| `de-DE` | Alemán (Alemania) | Female |  "de-DE-Hedda" |
|  |  | Female | "de-DE-HeddaRUS" |
|  |  | Male |  "de-DE-Stefan-Apollo" |
| `el-GR` | Griego | Male | "el-GR-Stefanos" |
| `en-AU` | Inglés (Australia) | Female |  "en-AU-Catherine" |
|  |  | Female |  "en-AU-HayleyRUS" |
| `en-CA` | Inglés (Canadá) | Female |  "en-CA-Linda" |
|  |  | Female |  "en-CA-HeatherRUS" |
| `en-GB` | English (Reino Unido) | Female |  "en-GB-Susan-Apollo" |
|  |  | Female |  "en-GB-HazelRUS" |
|  |  | Male |  "en-GB-George-Apollo" |
| `en-IE` | Inglés (Irlanda) | Male | "en-IE-Sean" |
| `en-IN` | Inglés (India) | Female | "en-IN-Heera-Apollo" |
|  |  | Female |  "en-IN-PriyaRUS" |
|  |  | Male |  "en-IN-Ravi-Apollo" |
| `en-US` | Inglés (EE. UU.) | Female |  "en-US-ZiraRUS" |
|  |  | Female | "en-US-AriaRUS" |
|  |  | Male | "en-US-BenjaminRUS" |
|  |  | Male |  "en-US-Guy24kRUS" |
| `es-ES` | Español (España) | Female |  "es-ES-Laura-Apollo" |
|  |  | Female | "es-ES-HelenaRUS" |
|  |  | Male | "es-ES-Pablo-Apollo" |
| `es-MX` | Español (México) | Female |  "es-MX-HildaRUS" |
|  |  | Male | "es-MX-Raul-Apollo" |
| `fi-FI` | Finés | Female | "fi-FI-HeidiRUS" |
| `fr-CA` | Francés (Canadá) | Female | "fr-CA-Caroline" |
|  |  | Female | "fr-CA-HarmonieRUS" |
| `fr-CH` | Francés (Suiza) | Male | "fr-CH-Guillaume" |
| `fr-FR` | Francés (Francia) | Female |  "fr-FR-Julie-Apollo" |
|  |  | Female |"fr-FR-HortenseRUS" |
|  |  | Male |  "fr-FR-Paul-Apollo" |
| `he-IL` | Hebreo (Israel) | Male |  "he-IL-Asaf" |
| `hi-IN` | Hindi (India) | Female | "hi-IN-Kalpana-Apollo" |
|  |  | Female |  "hi-IN-Kalpana" |
|  |  | Male |  "hi-IN-Hemant" |
| `hr-HR` | Croata | Male | "hr-HR-Matej" |
| `hu-HU` | Húngaro | Male |  "hu-HU-Szabolcs" |
| `id-ID` | Indonesio | Male | "id-ID-Andika" |
| `it-IT` | Italiano | Male |  "it-IT-Cosimo-Apollo" |
|  |  | Female |  "it-IT-LuciaRUS" |
| `ja-JP` | Japonés | Female |  "ja-JP-Ayumi-Apollo" |
|  |  | Male | "ja-JP-Ichiro-Apollo" |
|  |  | Female |  "ja-JP-HarukaRUS" |
| `ko-KR` | Coreano | Female | "ko-KR-HeamiRUS" |
| `ms-MY` | Malayo | Male |  "ms-MY-Rizwan" |
| `nb-NO` | Noruego | Female |  "nb-NO-HuldaRUS" |
| `nl-NL` | Neerlandés | Female |  "nl-NL-HannaRUS" |
| `pl-PL` | Polaco | Female |  "pl-PL-PaulinaRUS" |
| `pt-BR` | Portugués (Brasil) | Female | "pt-BR-HeloisaRUS" |
|  |  | Male |  "pt-BR-Daniel-Apollo" |
| `pt-PT` | Portugués (Portugal) | Female | "pt-PT-HeliaRUS" |
| `ro-RO` | Rumano | Male | "ro-RO-Andrei" |
| `ru-RU` | Ruso | Female |  "ru-RU-Irina-Apollo" |
|  |  | Male | "ru-RU-Pavel-Apollo" |
|  |  | Female |  ru-RU-EkaterinaRUS |
| `sk-SK` | Eslovaco | Male | "sk-SK-Filip" |
| `sl-SI` | Esloveno | Male |  "sl-SI-Lado" |
| `sv-SE` | Sueco | Female | "sv-SE-HedvigRUS" |
| `ta-IN` | Tamil (India) | Male |  "ta-IN-Valluvar" |
| `te-IN` | Telugu (India) | Female |  "te-IN-Chitra" |
| `th-TH` | Tailandés | Male |  "th-TH-Pattara" |
| `tr-TR` | Turco (Turquía) | Female | "tr-TR-SedaRUS" |
| `vi-VN` | Vietnamita | Male |  "vi-VN-An" |
| `zh-CN` | Chino (mandarín, simplificado) | Female |  "zh-CN-HuihuiRUS" |
|  |  | Female | "zh-CN-Yaoyao-Apollo" |
|  |  | Male | "zh-CN-Kangkang-Apollo" |
| `zh-HK` | Chino (cantonés, tradicional) | Female |  "zh-HK-Tracy-Apollo" |
|  |  | Female | "zh-HK-TracyRUS" |
|  |  | Male |  "zh-HK-Danny-Apollo" |
| `zh-TW` | Chino (mandarín, Taiwán) | Female |  "zh-TW-Yating-Apollo" |
|  |  | Female | "zh-TW-HanHanRUS" |
|  |  | Male |  "zh-TW-Zhiwei-Apollo" |

**1** *ar-EG admite el árabe estándar moderno (MSA).*

> [!IMPORTANT]
> La voz `en-US-Jessa` ha cambiado a `en-US-Aria`. Si usaba "Jessa" antes, conviértalo a "Aria".

> [!TIP]
> Puede seguir usando la asignación de nombre completo de servicio, como "Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)", en sus solicitudes de síntesis de voz.

### <a name="customization"></a>Personalización

La personalización de la voz está disponible para `de-DE`, `en-GB`, `en-IN`, `en-US`, `es-MX`, `fr-FR`, `it-IT`, `pt-BR` y `zh-CN`. Seleccione la configuración regional adecuada que coincida con los datos de entrenamiento que tiene para entrenar un modelo de voz personalizado. Por ejemplo, si los datos de grabación que tienen que hablar en inglés con acento británico, seleccione `en-GB`.

> [!NOTE]
> No se admite el entrenamiento de modelos bilingües en la funcionalidad de voz personalizada, excepto en el caso de chino-inglés bilingüe. Seleccione la opción de chino-inglés bilingüe si quiere entrenar una voz china que pueda hablar también inglés. El entrenamiento de la voz de todas las configuraciones regionales se inicia con un conjunto de datos de más de 2000 expresiones, excepto en el caso de `en-US` y `zh-CN`, que puede comenzar con cualquier tamaño de datos de entrenamiento.

## <a name="speech-translation"></a>Traducción de voz

**Speech Translation** API admite varios idiomas para la traducción de voz a voz y de texto a voz. El idioma de origen debe incluirse siempre en la siguiente tabla de conversión de voz en texto. Los idiomas de destino admitidos dependen de si el destino de traducción es voz o texto. Puede traducir la voz entrante en más de [60 idiomas](https://www.microsoft.com/translator/business/languages/). Un subconjunto de estos idiomas está disponible para la [síntesis de voz](language-support.md#text-languages).

### <a name="text-languages"></a>Idiomas de texto

| Idioma de texto           | Código de lenguaje |
|:------------------------|:-------------:|
| Afrikáans               | `af`          |
| Árabe                  | `ar`          |
| Bengalí                  | `bn`          |
| Bosnio (latino)         | `bs`          |
| Búlgaro               | `bg`          |
| Cantonés (tradicional) | `yue`         |
| Catalán                 | `ca`          |
| Chino simplificado      | `zh-Hans`     |
| Chino tradicional     | `zh-Hant`     |
| Croata                | `hr`          |
| Checo                   | `cs`          |
| Danés                  | `da`          |
| Neerlandés                   | `nl`          |
| Inglés                 | `en`          |
| Estonio                | `et`          |
| Fiyiano                  | `fj`          |
| Filipino                | `fil`         |
| Finés                 | `fi`          |
| Francés                  | `fr`          |
| Alemán                  | `de`          |
| Griego                   | `el`          |
| Gujarati                | `gu`          |
| Criollo haitiano          | `ht`          |
| Hebreo                  | `he`          |
| Hindi                   | `hi`          |
| Hmong Daw               | `mww`         |
| Húngaro               | `hu`          |
| Indonesio              | `id`          |
| Irlandés                   | `ga`          |
| Italiano                 | `it`          |
| Japonés                | `ja`          |
| Canarés                 | `kn`          |
| Kiswahili               | `sw`          |
| Klingon                 | `tlh-Latn`    |
| Klingon (plqaD)         | `tlh-Piqd`    |
| Coreano                  | `ko`          |
| Letón                 | `lv`          |
| Lituano              | `lt`          |
| Malgache                | `mg`          |
| Malayo                   | `ms`          |
| Malayalam               | `ml`          |
| Maltés                 | `mt`          |
| Maori                   | `mi`          |
| Maratí                 | `mr`          |
| Noruego               | `nb`          |
| Persa                 | `fa`          |
| Polaco                  | `pl`          |
| Portugués (Brasil)     | `pt-br`       |
| Portugués (Portugal)   | `pt-pt`       |
| Punjabi                 | `pa`          |
| Otomí Querétaro         | `otq`         |
| Rumano                | `ro`          |
| Ruso                 | `ru`          |
| Samoano                  | `sm`          |
| Serbio (cirílico)      | `sr-Cyrl`     |
| Serbio (latino)         | `sr-Latn`     |
| Eslovaco                  | `sk`          |
| Esloveno               | `sl`          |
| Español                 | `es`          |
| Sueco                 | `sv`          |
| Tahitiano                | `ty`          |
| Tamil                   | `ta`          |
| Telugu                  | `te`          |
| Tailandés                    | `th`          |
| Tongano                  | `to`          |
| Turco                 | `tr`          |
| Ucraniano               | `uk`          |
| Urdu                    | `ur`          |
| Vietnamita              | `vi`          |
| Galés                   | `cy`          |
| Maya Yucateco            | `yua`         |

## <a name="speaker-recognition"></a>Speaker Recognition

Consulte la tabla siguiente para ver los idiomas compatibles con las distintas API de Speaker Recognition. Consulte la [introducción](speaker-recognition-overview.md) para obtener información adicional sobre Speaker Recognition.

| Configuración regional | Idioma | Comprobación dependiente del texto | Comprobación independiente del texto | Identificación independiente del texto |
|----|----|----|----|----|
| es-ES | Inglés (EE. UU.) | sí | sí | sí |
|zh-CN    |Chino (mandarín, simplificado)|    N/D|    sí|    sí|
|de-DE    |Alemán (Alemania)    |N/D    |sí    |sí|
|en-GB    |English (Reino Unido)    |N/D    |sí    |sí|
|fr-FR    |Francés (Francia)    |N/D    |sí    |sí|
|en-AU    |Inglés (Australia)    |N/D    |sí    |sí|
|en-CA    |Inglés (Canadá)    |N/D|    sí|    sí|
|fr-CA    |Francés (Canadá)    |N/D    |sí|    sí|
|it-IT    |Italiano|    N/D    |sí|    sí|
|es-ES|    Español (España)    |N/D    |sí|    sí|
|es-MX    |Español (México)    |N/D|    sí|    sí|
|ja-JP|    Japonés    |N/D    |sí    |sí|
|pt-BR|    Portugués (Brasil)|    N/D|    sí|    sí|

## <a name="next-steps"></a>Pasos siguientes

* [Creación de una cuenta de Azure gratuita](https://azure.microsoft.com/free/cognitive-services/)
* [Vea cómo funciona el reconocimiento de voz en C#](~/articles/cognitive-services/Speech-Service/quickstarts/speech-to-text-from-microphone.md?pivots=programming-language-chsarp)
