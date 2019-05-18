---
title: 'Idiomas admitidos: Bing Custom Search API'
titleSuffix: Azure Cognitive Services
description: Una lista de los idiomas y las regiones admitidos para Bing Custom Search API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-custom-search
ms.topic: conceptual
ms.date: 09/25/2018
ms.author: aahi
ms.openlocfilehash: 56870a63f42c10b48cc2d8f0ae2995862be46d8f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65790242"
---
# <a name="language-and-region-support-for-the-bing-custom-search-api"></a>Compatibilidad de idiomas y regiones para Bing Custom Search API

Bing Custom Search API admite más de 36 países o regiones, muchos de ellos con más de un idioma.

Aunque es opcional, la solicitud debe especificar el parámetro de consulta [mkt](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-search-api-v7-reference#mkt), que identifica el mercado de donde desea que procedan los resultados. Para ver una lista de parámetros de consulta opcionales, consulte [Query Parameters](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-search-api-v7-reference#query-parameters) (Parámetros de consulta).

Puede especificar un país o región utilizando el parámetro de consulta `cc`. Si especifica un país o región, también debe especificar uno o más códigos de idioma con el encabezado `Accept-Language`. Los idiomas admitidos varían según el país o región; se facilitan para cada país en la tabla **Mercados**.

El encabezado `Accept-Language` y el parámetro de consulta `setLang` son mutuamente excluyentes, no especifique ambos. Para obtener más información, consulte [Accept-Language](https://docs.microsoft.com/rest/api/cognitiveservices/bing-custom-search-api-v7-reference#acceptlanguage).

## <a name="countriesregions"></a>Países/regiones

|País o región|Código|
|-------|----|
|Argentina|AR|
|Australia|AU|
|Austria|AT|
|Bélgica|BE|
|Brasil|BR|
|Canadá|CA|
|Chile|CL|
|Dinamarca|DK|
|Finlandia|FI|
|Francia|FR|
|Alemania|DE|
|Hong Kong RAE|HK|
|India|IN|
|Indonesia|ID|
|Italia|IT|
|Japón|JP|
|Corea|KR|
|Malasia|MY|
|México|MX|
|Países bajos|NL|
|Nueva Zelanda|NZ|
|Noruega|NO|
|China|CN|
|Polonia|PL|
|Portugal|PT|
|Filipinas|PH|
|Rusia|RU|
|Arabia Saudí|SA|
|Sudáfrica|ZA|
|España|ES|
|Suecia|SE|
|Suiza|CH|
|Taiwán|TW|
|Turquía|TR|
|Reino Unido|GB|
|Estados Unidos|US|


## <a name="markets"></a>Mercados

|País o región|Idioma|Código de mercado|
|-------|--------|-----------|
|Argentina|Español|es-AR|
|Australia|Inglés|en-AU|
|Austria|Alemán|de-AT|
|Bélgica|Holandés|nl-BE|
|Bélgica|Francés|fr-BE|
|Brasil|Portugués|pt-BR|
|Canadá|Inglés|en-CA|
|Canadá|Francés|fr-CA|
|Chile|Español|es-CL|
|Dinamarca|Danés|da-DK|
|Finlandia|Finlandés|fi-FI|
|Francia|Francés|fr-FR|
|Alemania|Alemán|de-DE|
|Hong Kong, SAR|Chino tradicional|zh-HK|
|India|Inglés|en-IN|
|Indonesia|Inglés|en-ID|
|Italia|Italiano|it-IT|
|Japón|Japonés|ja-JP|
|Corea|Coreano|ko-KR|
|Malasia|Inglés|en-MY|
|México|Español|es-MX|
|Países bajos|Holandés|nl-NL|
|Nueva Zelanda|Inglés|en-NZ|
|Noruega|Noruego (Bokmal)|no-NO|
|China|Chino|zh-CN|
|Polonia|Polaco|pl-PL|
|Portugal|Portugués|pt-PT|
|Filipinas|Inglés|en-PH|
|Rusia|Ruso|ru-RU|
|Arabia Saudí|Árabe|ar-SA|
|Sudáfrica|Inglés|en-ZA|
|España|Español|es-ES|
|Suecia|Sueco|sv-SE|
|Suiza|Francés|fr-CH|
|Suiza|Alemán|de-CH|
|Taiwán|Chino tradicional|zh-TW|
|Turquía|Turco|tr-TR|
|Reino Unido|Inglés|en-GB|
|Estados Unidos|Inglés|es-ES|
|Estados Unidos|Español|es-US|
