---
title: Ejemplos de escenarios de usuario de Text Analytics API
titleSuffix: Azure Cognitive Services
description: Use este artículo para ver algunos escenarios comunes para integrar Text Analytics API en sus servicios y procesos.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: conceptual
ms.date: 05/13/2020
ms.author: aahi
ms.openlocfilehash: 0e60ba0517fb3b9388045762853c651869cffb6a
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87986464"
---
# <a name="example-user-scenarios-for-the-text-analytics-api"></a>Ejemplos de escenarios de usuario de Text Analytics API

Text Analytics API es un servicio basado en la nube que realiza el procesamiento del lenguaje natural avanzado en texto. En este artículo se describen algunos ejemplos de casos de uso de la integración de la API en sus soluciones y procesos empresariales. 

## <a name="analyze-survey-results"></a>Analizar resultados de encuestas

Procese las respuestas de texto sin formato mediante el Análisis de sentimiento y extraiga conclusiones a partir de los resultados de las encuestas a los empleados y clientes. Agregue los resultados para realizar análisis y un seguimiento, así como para aumentar la involucración de las personas.

![Imagen que describe cómo realizar análisis de sentimiento sobre las encuestas a clientes y empleados.](media/use-cases/survey-results.svg)

## <a name="analyze-recorded-inbound-customer-calls"></a>Analizar las llamadas entrantes grabadas de los clientes

Extraiga estadísticas de las llamadas de los servicios al cliente mediante conversión de voz en texto, Análisis de sentimiento y Extracción de frases clave. Muestre los resultados en el panel de Power BI o un portal para comprender mejor a los clientes, resalte las tendencias del servicio al cliente y aumente la involucración del cliente. Envíe solicitudes a la API en forma de lote para generar informes o en tiempo real para la intervención. Consulte el código de ejemplo [en GitHub](https://github.com/rlagh2/callcenteranalytics).

![Imagen que describe cómo automatizar la obtención de estadísticas de llamadas del servició al cliente mediante el análisis de sentimiento](media/use-cases/azure-inbound.svg)

## <a name="process-and-categorize-support-incidents"></a>Procesar y clasificar los incidentes de soporte técnico

Use la Extracción de frases clave y el reconocimiento de entidades para procesar solicitudes de soporte técnico enviadas en formato de texto no estructurado. Use las entidades y las frases extraídas para clasificar las solicitudes para el análisis de tendencias y el planeamiento de recursos.

![Imagen que se describe cómo usar el reconocimiento de entidades y la Extracción de frases clave para clasificar los informes de incidentes y las tendencias](media/use-cases/support-incidents.svg)

## <a name="monitor-your-products-social-media-feeds"></a>Supervisión de las fuentes de sus productos en las redes sociales

Supervise los comentarios de los usuarios sobre productos en la página de Twitter o Facebook de su producto. Use los datos para analizar el sentimiento del cliente acerca del lanzamiento de nuevos productos, extraiga frases clave sobre características y solicitudes de características, o bien aborde las quejas de los clientes a medida que surgen. Consulte el ejemplo de la [plantilla de Microsoft Flow](https://flow.microsoft.com/galleries/public/templates/2680d2227d074c4d901e36c66e68f6f9/run-sentiment-analysis-on-tweets-and-push-results-to-a-power-bi-dataset/).

![Imagen que describe cómo supervisar los comentarios sobre su empresa y producto en las redes sociales mediante la Extracción de frases clave](media/use-cases/social-feed.svg)

## <a name="classify-and-redact-documents-that-have-sensitive-information"></a>Clasificación y redacción de documentos con información confidencial

Use el reconocimiento de entidades con nombre para identificar información personal y confidencial en los documentos. Use los datos para clasificar o redactar documentos para compartirlos de forma segura.

![Imagen en la que se describe cómo usar NER para detectar información personal, y clasificar y redactar documentos](media/use-cases/sensitive-docs.jpg)

## <a name="perform-opinion-mining"></a>Realización de minería de opiniones

Agrupe las opiniones relacionadas con aspectos concretos de un producto o servicio de encuestas, comentarios de los clientes o cualquier lugar en el que el texto represente una opinión sobre un aspecto. Úselo para ayudar a guiar los lanzamientos y mejoras del producto, los esfuerzos de marketing o resaltar el rendimiento del producto o servicio. 

> [!div class="mx-imgBorder"] 
> :::image type="content" source="media/use-cases/aspect-based-sentiment.png" alt-text="Opiniones de ejemplo sobre un hotel.":::

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Text Analytics API?](overview.md)
* [Envío de una solicitud a Text Analytics API mediante la biblioteca cliente](quickstarts/text-analytics-sdk.md)
