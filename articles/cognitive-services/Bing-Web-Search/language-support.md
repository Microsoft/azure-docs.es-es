---
title: 'Idiomas admitidos: Bing Web Search API'
titleSuffix: Azure Cognitive Services
description: Una lista de los idiomas naturales, países y regiones admitidos por Bing News Search API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 05/15/2019
ms.author: aahi
ms.openlocfilehash: 9425de6e75a9a46d71ff85ce49b0650c8e7a9a16
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2020
ms.locfileid: "68882685"
---
# <a name="language-and-region-support-for-the-bing-web-search-api"></a>Compatibilidad de idiomas y regiones para Bing Web Search API

Bing Web Search API admite más de 36 países o regiones, muchos de ellos con varios idiomas. Cuando se realiza una consulta, puede resultar útil especificar un país o una región para refinar los resultados de búsqueda en función de los intereses de ese país o región. Los resultados pueden contener vínculos a Bing, y estos vínculos pueden localizar la experiencia del usuario de Bing con arreglo a la región o el país especificados o al idioma.

Puede especificar un país o región utilizando el parámetro de consulta `cc`. Cuando se especifica un país o región, debe especificarse también uno o varios códigos de idioma con el [encabezado `Accept-Language`](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#headers). Utilice la [tabla de Mercados](#markets) para ver una lista de los idiomas que se admiten en cada mercado.

Si lo desea, también puede especificar el mercado con el parámetro de consulta `mkt` y un código de la tabla de **Mercados**. Cuando se especifica un mercado, también se indica un país o región y un idioma preferido. Puede establecer explícitamente el idioma con el parámetro de consulta `setLang`.

## <a name="countriesregions"></a>Países o regiones

|País/región|Código|
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
|Francia|VF|
|Alemania|DE|
|RAE de Hong Kong|HK|
|India|IN|
|Indonesia|ID|
|Italia|IT|
|Japón|JP|
|Corea|KR|
|Malasia|MY|
|México|MX|
|Países Bajos|NL|
|Nueva Zelanda|NZ|
|Noruega|No|
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

|País/región|Idioma|Código de mercado|
|-------|--------|-----------|
|Argentina|Español|es-AR|
|Australia|Inglés|en-AU|
|Austria|Alemán|de-AT|
|Bélgica|Neerlandés|nl-BE|
|Bélgica|Francés|fr-BE|
|Brasil|Portugués|pt-BR|
|Canadá|Inglés|en-CA|
|Canadá|Francés|fr-CA|
|Chile|Español|es-CL|
|Dinamarca|Danés|da-DK|
|Finlandia|Finés|fi-FI|
|Francia|Francés|fr-FR|
|Alemania|Alemán|de-DE|
|RAE de Hong Kong|Chino tradicional|zh-HK|
|India|Inglés|en-IN|
|Indonesia|Inglés|en-ID|
|Italia|Italiano|it-IT|
|Japón|Japonés|ja-JP|
|Corea|Coreano|ko-KR|
|Malasia|Inglés|en-MY|
|México|Español|es-MX|
|Países Bajos|Neerlandés|nl-NL|
|Nueva Zelanda|Inglés|en-NZ|
|Noruega|Noruego|no-NO|
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

## <a name="next-steps"></a>Pasos siguientes

* [Referencia de Bing Image Search API](//docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)
