---
title: 'Idiomas admitidos: Bing Visual Search API'
titleSuffix: Azure Cognitive Services
description: Una lista de los idiomas naturales, países y regiones admitidos por Bing Visual Search API. Bing Visual Search API admite más de 36 países o regiones, muchos de ellos con más de un idioma.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: conceptual
ms.date: 09/25/2018
ms.author: scottwhi
ms.openlocfilehash: b17341bc234ff3dfecc2c6dcd84ef77116a95d61
ms.sourcegitcommit: 58faa9fcbd62f3ac37ff0a65ab9357a01051a64f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2020
ms.locfileid: "68883549"
---
# <a name="language-and-region-support-for-the-bing-visual-search-api"></a>Compatibilidad de idiomas y regiones para Bing Visual Search API

Bing Visual Search API admite más de 36 países o regiones, muchos de ellos con más de un idioma. Cada solicitud debe incluir el país o región y el idioma preferido del usuario. Conocer el mercado del usuario ayuda a Bing a devolver resultados adecuados. Si no se especifican un país o región y un idioma, Bing realiza un mayor esfuerzo para determinar el país y el idioma del usuario. Dado que los resultados pueden contener vínculos a Bing, conocer el país o región y el idioma puede proporcionar una experiencia de usuario de Bing localizada y preferida si el usuario hace clic en los vínculos de Bing.

Para especificar el país o región y el idioma, establezca el parámetro de consulta `mkt` (mercado) en un código de la tabla **Mercados** siguiente. El mercado especifica un país o región y un idioma. Si el usuario prefiere ver el texto de la pantalla en un idioma diferente, establezca el parámetro de consulta `setLang`en el código de idioma adecuado.

También puede especificar el país o región utilizando el parámetro de consulta `cc`. Si especifica un país o región, también debe especificar uno o más códigos de idioma con el encabezado HTTP `Accept-Language`. Los idiomas admitidos varían según el país o región; se facilitan para cada país en la tabla Mercados.



> [!NOTE]
> Se aplican las restricciones de mercado que se indican a continuación:
>
> - Las anotaciones de reconocimiento de imagen solo están disponibles en inglés.
> - Las páginas de recetas, compras y conclusiones solo están disponibles en el mercado de en-US.


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
