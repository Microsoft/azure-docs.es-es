---
title: Puntos de conexión de Bing News Search
titlesuffix: Azure Cognitive Services
description: Resumen del punto de conexión de News Search API.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: conceptual
ms.date: 1/10/2019
ms.author: aahi
ms.openlocfilehash: b680f476345bde61797fd1cc137cad6e5653d0a5
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65798218"
---
# <a name="bing-news-search-api-endpoints"></a>Puntos de conexión de Bing News Search API

**News Search API** devuelve artículos de noticias, páginas web, imágenes, vídeos y [entidades](https://docs.microsoft.com/azure/cognitive-services/bing-entities-search/search-the-web). Las entidades contienen información de resumen sobre una persona, lugar o tema.

## <a name="endpoints"></a>Puntos de conexión

Para obtener nuevos resultados de búsqueda con Bing News Search API, envíe una solicitud `GET` a uno de los siguientes puntos de conexión. Los encabezados y parámetros de dirección URL definen especificaciones adicionales.

### <a name="news-items-by-search-query"></a>Elementos nuevos por consulta de búsqueda

```
GET https://api.cognitive.microsoft.com/bing/v7.0/news/search
```

Devuelve nuevos elementos según una consulta de búsqueda. Si la consulta de búsqueda está vacía, la API devolverá los principales elementos de noticias de categorías diferentes. Para enviar una consulta por URL, codifique el término de búsqueda y anéxelo al parámetro `q=""`. Para lograr una disponibilidad, consulte [países o regiones y mercados admitidos](language-support.md#supported-markets-for-news-search-endpoint).

### <a name="top-news-items-by-category"></a>Principales elementos de noticias por categoría

```
GET https://api.cognitive.microsoft.com/bing/v7.0/news  
```

Devuelve los principales elementos de noticias por categoría. En concreto, puede solicitar los artículos principales sobre negocios, deportes o entretenimiento mediante `category=business`, `category=sports` o `category=entertainment`.  El parámetro `category` solo se puede usar con la dirección URL `/news`. Existen algunos requisitos formales para especificar las categorías; consulte `category` en la documentación de [parámetros de consulta](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference#query-parameters). Para enviar una consulta por URL, codifique el término de búsqueda y anéxelo al parámetro `q=""`. Para lograr una disponibilidad, consulte [países o regiones y mercados admitidos](language-support.md#supported-markets-for-news-endpoint).

### <a name="trending-news-topics"></a>Temas de noticias de tendencia 

```
GET https://api.cognitive.microsoft.com/bing/v7.0/news/trendingtopics
```

Devuelve los temas de noticias que actualmente son tendencia en las redes sociales. Cuando se incluye la opción `/trendingtopics`, la búsqueda de Bing omite otros parámetros, como `freshness` y `?q=""`. Para lograr una disponibilidad, consulte [países o regiones y mercados admitidos](language-support.md#supported-markets-for-news-trending-endpoint).

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre encabezados, parámetros, códigos de mercado, objetos de respuesta, errores etc., consulte la referencia [Bing News Search API v7](https://docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference).

Para una información completa acerca de los parámetros admitidos por cada punto de conexión, consulte las páginas de referencia de cada tipo.
Para obtener ejemplos de solicitudes básicas con News Search API, consulte los [inicios rápidos de Bing News Search](https://docs.microsoft.com/azure/cognitive-services/bing-news-search).
