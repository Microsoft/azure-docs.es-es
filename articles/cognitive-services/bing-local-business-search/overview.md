---
title: ¿Qué es Bing Local Business Search API? | Microsoft Docs
titleSuffix: Azure Cognitive Services
description: Bing Local Business Search API es un servicio RESTful que permite a las aplicaciones buscar información acerca de lugares y empresas locales en función de las consultas de búsquedas.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: article
ms.date: 11/01/2018
ms.author: rosh
ms.openlocfilehash: 3cc9ed4dd108e76da6430a450876b709be514356
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796766"
---
# <a name="what-is-bing-local-business-search"></a>¿Qué es Bing Local Business Search?
Bing Local Business Search API es un servicio RESTful que permite a las aplicaciones buscar información acerca de empresas locales en función de las consultas de búsquedas. Por ejemplo, `q=<business-name> in Redmond, Washington` o `q=Italian restaurants near me`. 

## <a name="features"></a>Características
| Característica | DESCRIPCIÓN |  
| -- | -- | 
| [Búsqueda de ubicaciones y empresas locales](quickstarts/local-quickstart.md) | Bing Local Business Search API obtiene resultados localizados a partir de una consulta. Los resultados incluyen la dirección URL del sitio web de la empresa y muestran texto, el número de teléfono y la ubicación geográfica, lo que incluye: las coordenadas de GPS, la ciudad y la dirección postal |  
| [Filtrado de los resultados locales por límites geográficos](specify-geographic-search.md) | Agregue las coordenadas como parámetros de búsqueda para limitar los resultados a un área geográfica determinada que se especifica mediante un área circular o un rectángulo delimitador. | 
| [Filtrado de los resultados de empresas locales por categoría](local-categories.md) | Busque resultados de empresas locales por categorías. Esta opción utiliza las coordenadas de GPS o la dirección IP inversa del autor de la llamada para devolver resultados localizados de diversas categorías de empresas.|

## <a name="workflow"></a>Flujo de trabajo
Llame a Bing Local Business Search API con cualquier lenguaje de programación que pueda realizar solicitudes HTTP y procesar respuestas JSON. Se puede acceder a este servicio con la API REST.
 
1. Cree una [cuenta de Cognitive Services API](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) con acceso a Bing Search APIs. Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api).   
2. Codifique como una dirección URL los términos de búsqueda del parámetro de consulta `q=""`. Por ejemplo, `q=nearby+restaurant` o `q=nearby%20restaurant`. Si es necesario, establezca también la paginación. 
3. Envíe una [solicitud a Bing Local Business Search API](quickstarts/local-quickstart.md). 
4. Procese la respuesta JSON. 

> [!NOTE]
> En la actualidad, Local Business Search solo puede usarse en el mercado `en-US`. 
> [!NOTE]
> En la actualidad, Local Business Search no admite Autosuggest. 

## <a name="next-steps"></a>Pasos siguientes
- [Consultas y respuestas](local-search-query-response.md)
- [Guía de inicio rápido de Local Business Search](quickstarts/local-quickstart.md)
- [Referencia de Local Business Search API](local-search-reference.md)
- [Requisitos de uso y visualización](use-display-requirements.md)
