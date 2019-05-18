---
title: 'Límites de solicitudes: Translator Text API'
titleSuffix: Azure Cognitive Services
description: En este artículo se enumeran los límites de solicitudes de Translator Text API. Los cargos se generan en función del número de caracteres, y no por la frecuencia de solicitud, con un límite de 5000 caracteres por solicitud. Los límites de caracteres se basan en la suscripción, con F0 limitado a 2 millones de caracteres por hora.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.topic: conceptual
ms.date: 02/21/2019
ms.author: erhopf
ms.openlocfilehash: 91cc002f373318e5124fc21f76edbfd000d17238
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796908"
---
# <a name="request-limits-for-translator-text"></a>Límites de solicitudes de Translator Text

En este artículo se proporcionan los valores de limitación de Translator Text API. Los servicios incluyen traducción, transliteración, detección de la longitud de oraciones, detección de idiomas y traducciones alternativas.

## <a name="character-and-array-limits-per-request"></a>Límites de carácter y la matriz por solicitud

Cada solicitud de traducción está limitada a 5000 caracteres. Se le cobrará por cada carácter, no por el número de solicitudes. Se recomienda para enviar solicitudes más cortas.

La siguiente tabla se enumeran elemento y el carácter límites de la matriz para cada operación de Translator Text API.

| Operación | Tamaño máximo del elemento de matriz |   Número máximo de elementos de matriz |  Tamaño máximo de solicitud (caracteres) |
|:----|:----|:----|:----|
| Trasladar | 5.000 | 100   | 5.000 |
| Transliterar | 5.000 | 10    | 5.000 |
| Detectar | 10 000 | 100 |   50,000 |
| BreakSentence | 10 000    | 100 | 5,0000 |
| Búsqueda en diccionario| 100 |  10  | 1000 |
| Ejemplos de diccionario | 100 para texto y 100 para la traducción (total de 200)| 10|   2.000 |

## <a name="character-limits-per-hour"></a>Límites de caracteres por hora

El límite de caracteres por hora se basa en el nivel de suscripción de Translator Text. La cuota por hora se debe consumir uniformemente a lo largo de la hora. Si alcanzar o superar estos límites o enviar demasiado grande de una parte de la cuota en un breve período de tiempo, probablemente recibirá una salida de respuesta de la cuota. 

| Nivel: | Límite de caracteres |
|------|-----------------|
| F0 | 2 millones de caracteres por hora |
| S1 | 40 millones de caracteres por hora |
| S2 / C2 | 40 millones de caracteres por hora |
| S3 / C3 | 120 millones de caracteres por hora |
| S4 / C4 | 200 millones de caracteres por hora |

Límites para [suscripciones múltiples servicios](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication) son los mismos que el nivel S1.

Estos límites están restringidos a los modelos estándar traducción de Microsoft. Los modelos de traducción personalizada que utilice Translator personalizado se limitan a caracteres 1.800 por segundo.

## <a name="latency"></a>Latencia

Translator Text API tiene una latencia máxima de 15 segundos con los modelos estándar. Conversión mediante modelos personalizados tiene una latencia máxima de 25 segundos. Llegados a este punto, habrá recibido un resultado o una respuesta de tiempo de espera agotado. Normalmente, las respuestas tardan entre 150 milisegundos y 300 milisegundos en devolverse. Tiempos de respuesta variará en función del tamaño del par de solicitud y de idioma. Si no recibe una traducción o un [respuesta de error](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#errors) en ese período de tiempo, debe comprobar la conexión de red y vuelva a intentar.

## <a name="sentence-length-limits"></a>Límites de longitud de oraciones

Cuando se usa la función [BreakSentence](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-break-sentence), la longitud de la oración se limita a 275 caracteres. Existen excepciones para estos idiomas:

| Idioma | Código | Límite de caracteres |
|----------|------|-----------------|
| Chino | zh | 132 |
| Alemán | de | 290 |
| Italiano | it | 280 |
| Japonés | ja | 150 |
| Portugués | pt | 290 |
| Español | es | 280 |
| Italiano | it | 280 |
| Tailandés | .º | 258 |

> [!NOTE]
> Este límite no se aplica a las traducciones.

## <a name="next-steps"></a>Pasos siguientes

* [Precios](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)
* [Disponibilidad regional](https://azure.microsoft.com/global-infrastructure/services/?products=cognitive-services)
* [Referencia de Translator Text API v3](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
