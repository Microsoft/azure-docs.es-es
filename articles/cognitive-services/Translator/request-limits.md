---
title: 'Límites de solicitudes: Translator'
titleSuffix: Azure Cognitive Services
description: En este artículo se enumeran los límites de solicitudes de Translator. Los cargos se generan en función del número de caracteres, y no por la frecuencia de solicitud, con un límite de 5000 caracteres por solicitud. Los límites de caracteres se basan en la suscripción, con F0 limitado a 2 millones de caracteres por hora.
services: cognitive-services
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 08/06/2020
ms.author: swmachan
ms.openlocfilehash: dff69f00833cc3f8004befd330d144f4da0e332d
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89566592"
---
# <a name="request-limits-for-translator"></a>Límites de solicitudes de Translator

En este artículo se proporcionan los valores de limitación de Translator. Los servicios incluyen traducción, transliteración, detección de la longitud de oraciones, detección de idiomas y traducciones alternativas.

## <a name="character-and-array-limits-per-request"></a>Límites de caracteres y matriz por solicitud

Cada solicitud de traducción está limitada a 10 000 caracteres, en todos los idiomas de destino a los que se va a traducir. Por ejemplo, si se envía una solicitud de traducción de 3000 caracteres para traducir a 3 idiomas diferentes, se obtiene un tamaño de solicitud de 3000 x 3 = 9000 caracteres, que satisface el límite de solicitudes. Se le cobrará por cada carácter, no por el número de solicitudes. Se recomienda enviar solicitudes más cortas.

En la siguiente tabla se muestran los límites de caracteres y elementos de matriz correspondientes a cada una de las operaciones de Translator.

| Operación | Tamaño máximo del elemento de matriz |    Número máximo de elementos de matriz |    Tamaño máximo de la solicitud (caracteres) |
|:----|:----|:----|:----|
| Translate | 10 000    | 100   | 10 000 |
| Transliterar | 5\.000 | 10    | 5\.000 |
| Detect | 50.000 | 100 |   50.000 |
| BreakSentence | 50.000    | 100 | 50.000 |
| Búsqueda en diccionario| 100 |  10  | 1,000 |
| Ejemplos de diccionario | 100 para texto y 100 para traducción (200 en total)| 10|   2\.000 |

## <a name="character-limits-per-hour"></a>Límites de caracteres por hora

El límite de caracteres por hora se basa en el nivel de suscripción de Translator. 

La cuota por hora debe consumirse de forma uniforme a lo largo de la hora. Por ejemplo, en el límite del nivel F0 de 2 millones de caracteres por hora, los caracteres deben consumirse a una velocidad no superior a unos 33 300 caracteres por ventana deslizante de un minuto (2 millones de caracteres divididos por 60 minutos).

Si alcanza o sobrepasa estos límites, o bien envía una parte demasiado grande de la cuota en un breve período de tiempo, es probable que reciba una respuesta de superación de cuota. No hay ningún límite en las solicitudes simultáneas.

| Nivel | Límite de caracteres |
|------|-----------------|
| F0 | 2 millones de caracteres por hora |
| S1 | 40 millones de caracteres por hora |
| S2 / C2 | 40 millones de caracteres por hora |
| S3 / C3 | 120 millones de caracteres por hora |
| S4 / C4 | 200 millones de caracteres por hora |

Los límites de las [suscripciones a varios servicios](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication) son los mismos que los del nivel S1.

Estos límites están restringidos a modelos de traducción estándar de Microsoft. Los modelos de traducción personalizada que usan Traductor personalizado están limitados a 1800 caracteres por segundo.

## <a name="latency"></a>Latencia

Translator tiene una latencia máxima de 15 segundos con los modelos estándar y de 120 segundos al usar modelos personalizados. Normalmente, las respuestas para el *texto de 100 caracteres* tardan entre 150 milisegundos y 300 milisegundos en devolverse. Los modelos de traductor personalizados tienen características de latencia similares en la velocidad de solicitudes sostenidas y pueden tener una latencia mayor cuando la tasa de solicitudes es intermitente. Los tiempos de respuesta variarán según el tamaño de la solicitud y el par de idiomas. Si no recibe una traducción o una [respuesta de error](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#errors) en ese período de tiempo, compruebe el código, la conexión de red y vuelva a intentarlo. 

## <a name="sentence-length-limits"></a>Límites de longitud de oraciones

Cuando se usa la función [BreakSentence](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-break-sentence), la longitud de la oración se limita a 275 caracteres. Existen excepciones para estos idiomas:

| Idioma | Código | Límite de caracteres |
|----------|------|-----------------|
| Chino | zh | 166 |
| Alemán | de | 800 |
| Italiano | it | 800 |
| Japonés | ja | 166 |
| Portugués | pt | 800 |
| Español | es | 800 |
| Tailandés | th | 180 |

> [!NOTE]
> Este límite no se aplica a las traducciones.

## <a name="next-steps"></a>Pasos siguientes

* [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)
* [Disponibilidad regional](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services)
* [Referencia de Translator v3](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
