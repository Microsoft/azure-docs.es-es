---
title: Detección de orígenes de datos en Azure Data Catalog
description: Este artículo indica cómo detectar recursos de datos registrados con Azure Data Catalog, mediante la búsqueda y el filtrado, y mediante las capacidades de resaltado de referencias del portal de Azure Data Catalog.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: how-to
ms.date: 08/01/2019
ms.openlocfilehash: 4eb689b17df8236a00b5914912d1927804944f25
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87081209"
---
# <a name="how-to-discover-data-sources-in-azure-data-catalog"></a>Detección de orígenes de datos en Azure Data Catalog

## <a name="introduction"></a>Introducción

Azure Data Catalog es un servicio en la nube totalmente administrado que actúa como sistema de registro y de detección para orígenes de datos empresariales. En otras palabras, Data Catalog ayuda a las personas a descubrir, comprender y usar orígenes de datos. Ayuda a las organizaciones a obtener más valor de sus datos existentes. Una vez que se ha registrado un origen de datos con Azure Data Catalog, sus metadatos se indexan según el servicio, de forma que puede realizar búsquedas con facilidad de los datos que necesita.

## <a name="searching-and-filtering"></a>Búsqueda y filtrado

La detección en Azure Data Catalog usa dos mecanismos principales: la búsqueda y el filtrado.

La búsqueda está diseñada para ser intuitiva y eficaz. De forma predeterminada, los términos de la búsqueda se comparan todas las propiedades del catálogo, incluidas las anotaciones proporcionadas por el usuario.

El filtrado está diseñado para complementar la búsqueda. Puede seleccionar características específicas como expertos, tipo de origen de datos, tipo de objeto y etiquetas. Puede ver solo los activos de datos coincidentes y restringir los resultados de búsqueda a los activos coincidentes.

Con una combinación de la búsqueda y el filtrado, puede navegar rápidamente a través de los orígenes de datos registrados con Azure Data Catalog para detectar los que necesita.

## <a name="search-syntax"></a>Sintaxis de búsqueda

Aunque la búsqueda de texto libre predeterminada es sencilla e intuitiva, también puede usar la sintaxis de búsqueda de Azure Data Catalog para tener un mayor control sobre los resultados de la búsqueda. Azure Data Catalog admite las siguientes técnicas:

| Técnica | Uso | Ejemplo |
| --- | --- | --- |
| Búsqueda básica |Búsqueda básica que usa uno o varios términos de búsqueda. Los resultados son los recursos que tienen alguna propiedad que coincide con uno o varios de los términos especificados. |`sales data` |
| Ámbito de propiedad |Devuelve solamente los orígenes de datos para los que el término de búsqueda coincide con la propiedad especificada. |`name:finance` |
| operadores booleanos |Se puede ampliar o reducir una búsqueda mediante operaciones booleanas. |`finance NOT corporate` |
| Agrupación con paréntesis |Use paréntesis para agrupar partes de la consulta y así conseguir aislamiento lógico, especialmente en combinación con los operadores booleanos. |`name:finance AND (tags:Q1 OR tags:Q2)` |
| Operadores de comparación |Use comparaciones distintas de la igualdad de propiedades que tengan tipos de datos numéricos y de fechas. |`modifiedTime > "11/05/2014"` |

Para más información acerca de la búsqueda de Data Catalog, consulte el artículo sobre [Azure Data Catalog](/rest/api/datacatalog/#search-syntax-reference).

## <a name="hit-highlighting"></a>Resaltado de referencias

Al visualizar los resultados de la búsqueda, se resaltarán las propiedades mostradas que coinciden con los términos de búsqueda especificados (como el nombre del recurso de datos, la descripción y las etiquetas), para que sea más fácil identificar por qué se devuelve un recurso de datos concreto a partir de una búsqueda determinada.

> [!NOTE]
> Para desactivar el resaltado, use el conmutador **Resaltar** del portal de Data Catalog.

Al visualizar los resultados de la búsqueda, puede que no siempre sea evidente por qué se incluye un recurso de datos, incluso con el resaltado de visitas habilitado. Dado que se busca en todas las propiedades de forma predeterminada, es posible que se devuelva un recurso de datos debido a una coincidencia en una propiedad de nivel de columna. Y, puesto que distintos usuarios pueden anotar los recursos de datos registrados con sus propias descripciones y etiquetas, no se muestran todos los metadatos en la lista de resultados de la búsqueda.

En la vista de iconos predeterminada, cada icono que se muestre en los resultados de la búsqueda incluirá un icono **Ver coincidencias de los términos de búsqueda**, que le permite ver rápidamente el número de coincidencias y su ubicación, y obtener acceso a ellas si lo desea.

 ![Resaltado de referencias y coincidencias de búsqueda en el portal de Azure Data Catalog](./media/data-catalog-how-to-discover/search-matches.png)

## <a name="summary"></a>Resumen

Puesto que al registrar un origen de datos con Azure Data Catalog se copian los metadatos estructurales y descriptivos del origen de datos en el servicio de catálogo, es más fácil la detección y comprensión de ese origen de datos. Una vez registrado un origen de datos, puede detectarlo mediante el filtrado y la búsqueda desde el portal de Data Catalog.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información paso a paso sobre cómo detectar orígenes de datos, consulte [Introducción a Azure Data Catalog](data-catalog-get-started.md).
