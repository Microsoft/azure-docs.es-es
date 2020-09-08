---
title: 'Idiomas admitidos: Translator'
titleSuffix: Azure Cognitive Services
description: Translator de Cognitive Services admite los siguientes idiomas para la conversión de texto a texto con traducción automática neuronal (NMT).
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: reference
ms.date: 06/10/2020
ms.author: swmachan
ms.openlocfilehash: 9c745395026b8b7e8c58fcb4b7cc67971d971a7c
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89300228"
---
# <a name="language-and-region-support-for-text-and-speech-translation"></a>Compatibilidad de idiomas y regiones con la traducción de texto y voz

Use Translator para traducir a y desde cualquiera de los más de 70 idiomas de traducción de texto. La traducción automática neuronal (NMT) es el nuevo estándar de traducción automática de alta calidad con tecnologías de inteligencia artificial, y está disponible de forma predeterminada con la versión V3 de Translator cuando hay un sistema neuronal disponible.

También puede usar Translator junto con Traductor personalizado para compilar sistemas de traducción neuronal que entiendan la terminología que se usa en su empresa y su sector y con Servicio Voz de Microsoft para agregar traducción de voz a la aplicación.

[Más información sobre cómo funciona la traducción automática](https://www.microsoft.com/translator/mt.aspx)

## <a name="text-translation"></a>Traducción de texto
La traducción de texto está disponible mediante la operación Traducir a o desde cualquiera de los idiomas disponibles en Translator. La API también ofrece detección de idioma mediante la operación Detectar, transliteración mediante la operación Transliterar y diccionarios bilingües mediante las operaciones Buscar en diccionario y Ejemplos de diccionario. A continuación se indican los idiomas disponibles en cada una de estas operaciones. 

### <a name="translate"></a>Translate

Translator admite los siguientes idiomas para la traducción de texto a texto. 

[Vea documentación de referencia de la operación Traducir](reference/v3-0-translate.md)

|Idioma|  Código de lenguaje|
|:-----|:-----:|
|Afrikáans| `af`|
|Árabe|    `ar`    |
|Bengalí|    `bn`    |
|Bosnio (latino)|   `bs`    |
|Búlgaro| `bg`    |
|Cantonés (tradicional)|   `yue`|
|Catalán|   `ca`    |
|Chino simplificado|    `zh-Hans`|
|Chino tradicional|   `zh-Hant`       |
|Croata|  `hr`    |
|Checo| `cs`    |
|Dari|  `prs`   |
|Danés|    `da`        |
|Neerlandés| `nl`|
|Inglés|   `en`    |
|Estonio|  `et`    |
|Fiyiano|    `fj`    |
|Filipino|  `fil`   |
|Finés|   `fi`    |
|Francés|    `fr`    |
|Alemán|    `de`    |
|Griego| `el`    |
|Gujarati|  `gu`    |
|Criollo haitiano|    `ht`        |
|Hebreo |`he`   |
|Hindi| `hi`    |
|Hmong Daw| `mww`   |
|Húngaro| `hu`    |
|Islandés| `is`    |
|Indonesio|    `id`    |
|Irlandés | `ga`|
|Italiano|   `it`    |
|Japonés|  `ja`    |
|Canarés|`kn`|
|Kazajo|`kk`|
|Kiswahili| `sw`    |
|Klingon|   `tlh-Latn`  |
|Klingon (plqaD)|   `tlh-Piqd`  |
|Coreano |`ko`   |
|Kurdo (central)  |`ku`   |
|Kurdo (norte) |`kmr`  |
|Letón|   `lv`    |
|Lituano|    `lt`    |
|Malgache|  `mg`    |
|Malayo| `ms`        |
|Malayalam| `ml` |
|Maltés|   `mt`    |
|Maori| `mi`  |
|Maratí| `mr`  |
|Noruego| `nb`    |
|Odia|  `or`    |
|Pastún|    `ps`    |
|Persa|   `fa`    |
|Polaco|    `pl`    |
|Portugués (Brasil)|   `pt-br` |
|Portugués (Portugal)| `pt-pt` |
|Punjabi|`pa`|
|Otomí Querétaro|   `otq`   |
|Rumano|  `ro`    |
|Ruso|   `ru`    |
|Samoano|    `sm`    |
|Serbio (cirílico)|    `sr-Cyrl`|
|Serbio (latino)|   `sr-Latn`       |
|Eslovaco|    `sk`    |
|Esloveno| `sl`    |
|Español|   `es`    |
|Sueco|   `sv`    |
|Tahitiano|  `ty`    |
|Tamil| `ta`    |
|Telugu|    `te`    |
|Tailandés|  `th`    |
|Tongano|    `to`    |
|Turco|   `tr`        |
|Ucraniano| `uk`    |
|Urdu|  `ur`    |
|Vietnamita|    `vi`    |
|Galés| `cy`    |
|Maya Yucateco|  `yua`   |

> [!NOTE]
> El código de idioma `pt` se establece de forma predeterminada en `pt-br`, Portugués (Brasil).

### <a name="detect"></a>Detect

Translator detecta los siguientes idiomas para traducción y transliteración.

[Vea documentación de referencia de la operación Detectar](reference/v3-0-detect.md)

|Idioma|  Código de lenguaje|
|:-----|:-----:|
|Afrikáans| `af`|
|Árabe|    `ar`    |
|Búlgaro| `bg`    |
|Catalán|   `ca`    |
|Chino simplificado|    `zh-Hans`|
|Chino tradicional|   `zh-Hant`       |
|Croata|  `hr`    |
|Checo| `cs`    |
|Danés|    `da`        |
|Neerlandés| `nl`|
|Inglés|   `en`    |
|Estonio|  `et`    |
|Finés|   `fi`    |
|Francés|    `fr`    |
|Alemán|    `de`    |
|Griego| `el`    |
|Gujarati|  `gu`    |
|Criollo haitiano|    `ht`        |
|Hebreo |`he`   |
|Hindi| `hi`    |
|Húngaro| `hu`    |
|Islandés| `is`    |
|Indonesio|    `id`    |
|Irlandés | `ga`|
|Italiano|   `it`    |
|Japonés|  `ja`    |
|Kiswahili| `sw`    |
|Klingon|   `tlh-Latn`  |
|Coreano |`ko`   |
|Kurdo (central)  |`ku-Arab`  |
|Letón|   `lv`    |
|Lituano|    `lt`    |
|Malayo| `ms`        |
|Maltés|   `mt`    |
|Noruego| `nb`    |
|Pastún|    `ps`    |
|Persa|   `fa`    |
|Polaco|    `pl`    |
|Portugués (Brasil)|   `pt-br` |
|Portugués (Portugal)| `pt-pt` |
|Rumano|  `ro`    |
|Ruso|   `ru`    |
|Serbio (cirílico)|    `sr-Cyrl`|
|Serbio (latino)|   `sr-Latn`       |
|Eslovaco|    `sk`    |
|Esloveno| `sl`    |
|Español|   `es`    |
|Sueco|   `sv`    |
|Tahitiano|  `ty`    |
|Tailandés|  `th`    |
|Turco|   `tr`        |
|Ucraniano| `uk`    |
|Urdu|  `ur`    |
|Vietnamita|    `vi`    |
|Galés| `cy`    |
|Maya Yucateco|  `yua`   |

### <a name="transliterate"></a>Transliterar

El método Transliterate admite los siguientes idiomas. En "Hacia/Desde", "<-->" indica que el idioma se puede transliterar hacia o desde cualquiera de los alfabetos enumerados. "-->" indica que el idioma solo se puede transliterar de un idioma al otro.

[Vea documentación de referencia de la operación Transliterar](reference/v3-0-translate.md)


| Idioma    | Código de lenguaje | Script | Hacia/Desde | Script|
|:----------- |:-------------:|:-------------:|:-------------:|:-------------:|
| Árabe | `ar` | Árabe `Arab` | <--> | Latino `Latn` |
| Bengalí  | `bn` | Bengalí `Beng` | <--> | Latino `Latn` |
| Chino (simplificado) | `zh-Hans` | Chino simplificado `Hans`| <--> | Latino `Latn` |
| Chino (simplificado) | `zh-Hans` | Chino simplificado `Hans`| <--> | Chino tradicional `Hant`|
| Chino (tradicional) | `zh-Hant` | Chino tradicional `Hant`| <--> | Latino `Latn` |
| Chino (tradicional) | `zh-Hant` | Chino tradicional `Hant`| <--> | Chino simplificado `Hans` |
| Gujarati | `gu`  | Guyaratí `Gujr` | <--> | Latino `Latn` |
| Hebreo | `he` | Hebreo `Hebr` | <--> | Latino `Latn` |
| Hindi | `hi` | Devanagari `Deva` | <--> | Latino `Latn` |
| Japonés | `ja` | Japonés `Jpan` | <--> | Latino `Latn` |
| Canarés | `kn` | Canarés `Knda` | <--> | Latino `Latn` |
| Malayalam | `ml` | Malayalam `Mlym` | <--> | Latino `Latn` |
| Maratí | `mr` | Devanagari `Deva` | <--> | Latino `Latn` |
| Odia | `or` | Odia `Orya` | <--> | Latino `Latn` |
| Punjabi | `pa` | Gurmukhi `Guru`  | <--> | Latino `Latn`  |
| Serbio (cirílico) | `sr-Cyrl` | Cirílico `Cyrl`  | --> | Latino `Latn` |
| Serbio (latino) | `sr-Latn` | Latino `Latn` | --> | Cirílico `Cyrl`|
| Tamil | `ta` | Tamil `Taml` | <--> | Latino `Latn` |
| Telugu | `te` | Telugu `Telu` | <--> | Latino `Latn` |
| Tailandés | `th` | Tailandés `Thai` | --> | Latino `Latn` |

### <a name="dictionary"></a>Diccionario

El diccionario admite los siguientes idiomas desde o hacia el inglés con los métodos Lookup y Examples.

Vea documentación de referencia de las operaciones [Buscar en diccionario](reference/v3-0-dictionary-lookup.md) y [Ejemplos de diccionario](reference/v3-0-dictionary-examples.md).

| Idioma    | Código de lenguaje |
|:----------- |:-------------:|
| Afrikáans      | `af`          |
| Árabe       | `ar`          |
| Bengalí      | `bn`          |
| Bosnio (latino)      | `bs`          |
| Búlgaro      | `bg`          |
| Catalán      | `ca`          |
| Chino simplificado      | `zh-Hans`          |
| Croata      | `hr`          |
| Checo      | `cs`          |
| Danés      | `da`          |
| Neerlandés      | `nl`          |
| Estonio      | `et`          |
| Finés      | `fi`          |
| Francés      | `fr`          |
| Alemán      | `de`          |
| Griego      | `el`          |
| Criollo haitiano      | `ht`          |
| Hebreo      | `he`          |
| Hindi      | `hi`          |
| Hmong Daw      | `mww`          |
| Húngaro      | `hu`          |
| Islandés    | `is`  |
| Indonesio      | `id`          |
| Italiano      | `it`          |
| Japonés      | `ja`          |
| Kiswahili      | `sw`          |
| Klingon      | `tlh`          |
| Coreano      | `ko`          |
| Letón      | `lv`          |
| Lituano      | `lt`          |
| Malayo      | `ms`          |
| Maltés      | `mt`          |
| Noruego      | `nb`          |
| Persa      | `fa`          |
| Polaco      | `pl`          |
| Portugués (Brasil)     | `pt-br`          |
| Rumano      | `ro`          |
| Ruso      | `ru`          |
| Serbio (latino)      | `sr-Latn`          |
| Eslovaco     | `sk`          |
| Esloveno      | `sl`          |
| Español      | `es`          |
| Sueco      | `sv`          |
| Tamil      | `ta`          |
| Tailandés      | `th`          |
| Turco      | `tr`          |
| Ucraniano      | `uk`          |
| Urdu      | `ur`          |
| Vietnamita      | `vi`          |
| Galés      | `cy`          |

### <a name="access-the-translator-language-list-programmatically"></a>Acceso a la lista de idiomas de Translator mediante programación

Puede recuperar una lista de los idiomas admitidos por Translator con el método Languages. Puede ver la lista por característica, código de idioma o por nombre del idioma en inglés o en cualquier otro idioma admitido. El servicio Microsoft Translator actualiza esta lista automáticamente cuando hay nuevos idiomas disponibles.

[Ver la documentación de referencia de la operación Languages](reference/v3-0-languages.md)

## <a name="customization"></a>Personalización

Los siguientes idiomas están disponibles para personalización al y del inglés mediante [Traductor personalizado](https://aka.ms/CustomTranslator).

| Idioma    | Código de lenguaje |
|:----------- |:-------------:|
|Afrikáans| `af`|
| Árabe       | `ar`          |
| Bengalí      | `bn`          |
| Bosnio (latino)      | `bs`          |
| Búlgaro      | `bg`          |
|Catalán|   `ca`    |
| Chino simplificado      | `zh-Hans`          |
|Chino tradicional|   `zh-Hant`   |
| Croata      | `hr`          |
| Checo      | `cs`          |
| Danés      | `da`          |
| Neerlandés      | `nl`          |
| Inglés    | `en`     |
| Estonio      | `et`          |
|Fiyiano|    `fj`    |
|Filipino|  `fil`   |
| Finés      | `fi`          |
| Francés      | `fr`          |
| Alemán      | `de`          |
| Griego      | `el`          |
| Gujarati| `gu`    |
| Hebreo      | `he`          |
| Hindi      | `hi`          |
| Húngaro      | `hu`          |
| Islandés | `is` |
| Indonesio|   `id`    |
| Irlandés | `ga`  |
| Italiano      | `it`          |
| Japonés      | `ja`          |
|Canarés|`kn`|
| Kiswahili|    `sw`    |
| Coreano      | `ko`          |
| Letón      | `lv`          |
| Lituano      | `lt`          |
| Malgache| `mg`    |
| Malayo|    `ms`        |
|Maltés|   `mt`    |
| Maori| `mi`  |
| Maratí| `mr`  |
| Noruego      | `nb`          |
| Persa      | `fa`          |
| Polaco      | `pl`          |
| Portugués (Brasil) | `pt-br` |
| Punjabi|`pa`|
| Rumano      | `ro`          |
| Ruso      | `ru`          |
| Samoano|   `sm`    |
| Serbio (latino)      | `sr-Latn`          |
| Eslovaco     | `sk`          |
| Esloveno      | `sl`          |
| Español      | `es`          |
| Sueco      | `sv`          |
|Tahitiano|  `ty`    |
| Tailandés      | `th`          |
|Tongano|    `to`    |
| Turco      | `tr`          |
| Ucraniano      | `uk`          |
| Urdu| `ur`    |
| Vietnamita      | `vi`          |
| Galés | `cy` |

## <a name="speech-translation"></a>Speech Translation
La traducción de voz está disponible mediante Translator con el Servicio Voz de Cognitive Services. Vea [Documentación sobre el servicio de voz](https://docs.microsoft.com/azure/cognitive-services/speech-service/) para obtener más información sobre el uso de la traducción de voz y para ver todas las [opciones de idioma disponibles](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support).

### <a name="speech-to-text"></a>Voz a texto
Convierta la voz en texto para traducir al idioma de texto que prefiera. La conversión de voz en texto se usa para la traducción de voz a texto o para la traducción de voz a voz cuando se usa junto con la síntesis de voz.

| Idioma    |
|:----------- |
|Árabe|
|Cantonés (tradicional)|
|Catalán|
|Chino simplificado|
|Chino tradicional|
|Danés|
|Neerlandés|
|Inglés|
|Finés|
|Francés|
|Alemán|
|Gujarati|
|Hindi|
|Italiano|
|Japonés|
|Coreano|
|Maratí|
|Noruego|
|Polaco|
|Portugués (Brasil)|
|Portugués (Portugal)|
|Ruso|
|Español|
|Sueco|
|Tamil|
|Telugu|
|Tailandés|
|Turco|

### <a name="text-to-speech"></a>Texto a voz
Convertir texto a voz. La conversión de texto en voz se usa para agregar una salida con sonido de los resultados de traducción o para la traducción de voz a voz cuando se usa con la conversión de voz en texto. 

| Idioma    |
|:----------- |
|Árabe|
|Búlgaro|
|Cantonés (tradicional)|
|Catalán|
|Chino simplificado|
|Chino tradicional|
|Croata|
|Checo|
|Danés|
|Neerlandés|
|Inglés|
|Finés|
|Francés|
|Alemán|
|Griego|
|Hebreo|
|Hindi|
|Húngaro|
|Indonesio|
|Italiano|
|Japonés|
|Coreano|
|Malayo|
|Noruego|
|Polaco|
|Portugués (Brasil)|
|Portugués (Portugal)|
|Rumano|
|Ruso|
|Eslovaco|
|Esloveno|
|Español|
|Sueco|
|Tamil|
|Telugu|
|Tailandés|
|Turco|
|Vietnamita|

## <a name="view-the-language-list-on-the-microsoft-translator-website"></a>Vista de la lista de idiomas en el sitio web de Microsoft Translator

Para obtener una vista rápida de los idiomas, el sitio web de Microsoft Translator muestra todos los idiomas compatibles con Translator para la traducción de texto y el Servicio Voz para la traducción de voz. Esta lista no incluye información específica para desarrolladores, por ejemplo, los códigos de idioma.

[Ver la lista de idiomas](https://www.microsoft.com/translator/languages.aspx)
