---
title: ¿Qué es Text Analytics API? - Funcionalidades -
titleSuffix: Azure Cognitive Services
description: Use Text Analytics API de Azure Cognitive Services para el análisis de sentimiento, la extracción de frases clave, la detección del idioma y el reconocimiento de entidades.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: overview
ms.date: 08/27/2020
ms.author: aahi
ms.openlocfilehash: a3c538f3a9e7a2d8d71fff38fb927dbcdf725732
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000964"
---
# <a name="what-is-the-text-analytics-api"></a>¿Qué es Text Analytics API?

API Text Analytics es un servicio basado en la nube que proporciona un procesamiento avanzado de idioma natural sobre texto sin formato e incluye cuatro funciones principales: análisis de sentimiento, extracción de frases clave, detección de lenguaje y reconocimiento de entidades de caracteres.

La API forma parte de [Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/), una colección de algoritmos de aprendizaje automático y de inteligencia artificial en la nube para los proyectos de desarrollo.

> [!VIDEO https://channel9.msdn.com/Shows/AI-Show/Understanding-Text-using-Cognitive-Services/player]

El análisis de texto puede significar diferentes cosas, pero en Cognitive Services, Text Analytics API proporciona cuatro tipos de análisis, que se describen a continuación. Puede usar estas características con la [API de REST](https://westus.dev.cognitive.microsoft.com/docs/services/TextAnalytics-V2-1/) o la [biblioteca cliente](quickstarts/text-analytics-sdk.md).

## <a name="sentiment-analysis"></a>Análisis de sentimiento
Use el [análisis de sentimiento](how-tos/text-analytics-how-to-sentiment-analysis.md) para averiguar qué piensan los clientes de su marca o de un tema concreto mediante el análisis de texto sin formato, con el fin de obtener pistas acerca de si sus opiniones son positivas o negativas. Esta API devuelve una puntuación de la opción, que oscila entre 0 y 1, con respecto a cada documento, donde 1 es la más positiva.<br /> Los modelos de análisis se entrenan previamente con una gran cantidad de cuerpo de texto y tecnologías de idioma natural de Microsoft. Para [idiomas seleccionados](text-analytics-supported-languages.md), la API puede analizar y puntuar cualquier texto sin formato que se proporcione, y devolver los resultados directamente a la aplicación que realiza la llamada.

## <a name="key-phrase-extraction"></a>Extracción de frases clave
[Extracción automática de las frases clave](how-tos/text-analytics-how-to-keyword-extraction.md) para identificar rápidamente los puntos principales. Por ejemplo, si el texto de entrada es "La comida estaba deliciosa y el personal era maravilloso", la API devuelve los principales puntos de conversación: "comida" y "personal maravilloso".

## <a name="language-detection"></a>Detección de idiomas
Puede [detectar en qué idioma está escrito el texto de entrada](how-tos/text-analytics-how-to-language-detection.md) y utilizar un código de idioma único para informar acerca de cada documento enviado en la solicitud para una amplia gama de idiomas, variantes, dialectos y algunos idiomas regionales o culturales. El código de idioma se empareja con una puntuación que indica la intensidad de esta.

## <a name="named-entity-recognition"></a>Reconocimiento de entidades con nombre
[Identifique y clasifique las entidades](how-tos/text-analytics-how-to-entity-linking.md) en el texto como personas, lugares, organizaciones, fecha y hora, cantidades, porcentajes, divisas, etc. Las entidades más conocidas también se reconocen y vinculan a más información en la web.

## <a name="use-containers"></a>Uso de contenedores

[Use los contenedores de Text Analytics](how-tos/text-analytics-how-to-install-containers.md) para extraer frases clave, detectar el idioma y analizar opiniones localmente, mediante la instalación de contenedores de Docker estándar más cercanos a los datos.

## <a name="typical-workflow"></a>Flujo de trabajo típico

El flujo de trabajo es sencillo: se envían datos para su análisis y se controla el resultado en el código. Los analizadores se consumen tal cual, sin configuración ni personalización adicionales.

1. [Cree un recurso de Azure](../cognitive-services-apis-create-account.md) para Text Analytics. Después, [obtenga la clave](../cognitive-services-apis-create-account.md#get-the-keys-for-your-resource) generada para autenticar las solicitudes.

2. [Formule una solicitud](how-tos/text-analytics-how-to-call-api.md#json-schema) en JSON que contenga los datos como texto sin estructura.

3. Publique la solicitud en el punto de conexión establecido durante el registro y anexe el recurso deseado: análisis de sentimiento, extracción de frases clave, detección de idioma o identificación de entidad de caracteres.

4. Transmita la respuesta en secuencias o almacénela localmente. En función de cuál sea la solicitud, los resultados son una puntuación de opinión, una colección de frases clave extraídas o un código de idioma.

La salida se devuelve como documento JSON individual, con los resultados de cada documento de texto que ha publicado, en función del identificador. Posteriormente puede analizar, visualizar o clasificar los resultados hasta convertirlos en conclusiones que requieren que se realice alguna acción.

No se almacenan datos en su cuenta. Las operaciones que realiza Text Analytics API no tienen estado, lo que significa que se procesa el texto que se proporciona y los resultados se devuelven de inmediato.

## <a name="text-analytics-for-multiple-programming-experience-levels"></a>Text Analytics para varios niveles de experiencia de programación

Puede empezar a usar Text Analytics API en sus procesos, aunque no tenga mucha experiencia en programación. Use estos tutoriales para averiguar cómo puede usar la API para analizar texto de distintas formas que se ajusten a su nivel de experiencia. 

* Conocimientos mínimos de programación necesarios:
    * [Extracción de información en Excel mediante Text Analytics y Power Automate](tutorials/extract-excel-information.md)
    * [Uso de Text Analytics API y MS Flow para identificar el sentimiento de los comentarios en un grupo de Yammer](https://docs.microsoft.com/Yammer/integrate-yammer-with-other-apps/sentiment-analysis-flow-azure?toc=%2F%2Fazure%2Fcognitive-services%2Ftext-analytics%2Ftoc.json&bc=%2F%2Fazure%2Fbread%2Ftoc.json)
    * [Integración de Power BI con Text Analytics API para analizar los comentarios de clientes](tutorials/tutorial-power-bi-key-phrases.md)
* Experiencia de programación recomendada:
    * [Análisis de sentimiento sobre los datos de streaming con Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/databricks-sentiment-analysis-cognitive-services?toc=%2F%2Fazure%2Fcognitive-services%2Ftext-analytics%2Ftoc.json&bc=%2F%2Fazure%2Fbread%2Ftoc.json)
    * [Compilación de una aplicación de Flask para traducir texto, analizar el sentimiento y sintetizar la voz](https://docs.microsoft.com/azure/cognitive-services/translator/tutorial-build-flask-app-translation-synthesis?toc=%2F%2Fazure%2Fcognitive-services%2Ftext-analytics%2Ftoc.json&bc=%2F%2Fazure%2Fbread%2Ftoc.json)


<a name="supported-languages"></a>

## <a name="supported-languages"></a>Idiomas compatibles

Esta sección se ha movido a otro artículo para facilitar su detección. Para ver este contenido, consulte [Compatibilidad de idiomas para Text Analytics API](text-analytics-supported-languages.md).

<a name="data-limits"></a>

## <a name="data-limits"></a>Límites de datos

Todos los puntos de conexión de Text Analytics API aceptan datos de texto sin formato. Para más información, consulte el artículo acerca de los [límites de datos](concepts/data-limits.md).

## <a name="unicode-encoding"></a>Codificación Unicode

Text Analytics API utiliza la codificación Unicode para la representación del texto y los cálculos del número de caracteres. Las solicitudes se pueden enviar tanto en UTF-8 como en UTF-16 sin que haya diferencias apreciables en el número de caracteres. Los puntos de código Unicode se utilizan como heurística de longitud de caracteres y se consideran equivalentes a efectos de los límites de datos del análisis de texto. Si usa [`StringInfo.LengthInTextElements`](https://docs.microsoft.com/dotnet/api/system.globalization.stringinfo.lengthintextelements) para obtener el número de caracteres, está empleando el mismo método que nosotros para medir el tamaño de los datos.

## <a name="next-steps"></a>Pasos siguientes

+ [Cree un recurso de Azure](../cognitive-services-apis-create-account.md) para Text Analytics para obtener una clave y un punto de conexión para las aplicaciones.

+ Use la [guía de inicio rápido](quickstarts/text-analytics-sdk.md) para empezar a enviar llamadas API. Aprenda a enviar texto, elegir un análisis y ver los resultados con una cantidad mínima de código.

+ Consulte las [novedades de Text Analytics API](whats-new.md) para información sobre las nuevas versiones y características.

+ Profundice un poco más con este [tutorial sobre análisis de sentimiento](https://docs.microsoft.com/azure/azure-databricks/databricks-sentiment-analysis-cognitive-services) mediante Azure Databricks.

+ Consulte nuestra lista de entradas de blog y más vídeos sobre cómo usar Text Analytics API con otras herramientas y tecnologías en nuestra [página de contenido externo y de la comunidad](text-analytics-resource-external-community.md).
