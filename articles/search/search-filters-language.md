---
title: Filtro por idioma en un índice de búsqueda
titleSuffix: Azure Cognitive Search
description: Criterios de filtro para admitir la búsqueda en varios idiomas, de ámbito de ejecución de consultas en campos específicos del idioma.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/22/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 120c30803d1423dc8b1255c65703e84d5938f235
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002528"
---
# <a name="how-to-filter-by-language-in-azure-cognitive-search"></a>Filtro por idioma en Azure Cognitive Search 

Un requisito fundamental de una aplicación de búsqueda multilingüe es la posibilidad de buscar y recuperar los resultados en el idioma del usuario. En Azure Cognitive Search, una manera de satisfacer los requisitos de idioma de una aplicación multilingüe es crear una serie de campos dedicados al almacenamiento de cadenas en un idioma determinado y luego restringir la búsqueda de texto completo a solo esos campos en el momento de la consulta.

Los parámetros de consulta de la solicitud se usan para establecer el ámbito de la operación de búsqueda y para recortar los resultados de los campos que no proporcionan contenido compatible con la experiencia de búsqueda que quiere proporcionar.

| Parámetros | Propósito |
|-----------|--------------|
| **searchFields** | Limita la búsqueda de texto completo a la lista de campos con nombre. |
| **$select** | Recorta la respuesta para incluir solo los campos que especifique. De forma predeterminada, se devuelven todos los campos recuperables. El parámetro **$select** le permite elegir cuáles devolver. |

El éxito de esta técnica depende de la integridad del contenido del campo. Azure Cognitive Search no traduce las cadenas ni realiza la detección del idioma. Es responsabilidad suya asegurarse de que los campos contienen las cadenas que espera.

## <a name="define-fields-for-content-in-different-languages"></a>Definición de campos para el contenido en distintos idiomas

En Azure Cognitive Search, las consultas tienen como destino un índice único. Los desarrolladores que quieran proporcionar cadenas específicas del idioma en una única experiencia de búsqueda suele definen campos dedicados para almacenar los valores: un campo para las cadenas en inglés, otro para las de francés, etc. 

El ejemplo siguiente procede del [ejemplo de una inmobiliaria](search-get-started-portal.md) que tiene varios campos de cadena que incluyen contenido en distintos idiomas. Observe las asignaciones del analizador de idioma en los campos de este índice. Los campos que contienen cadenas funcionan mejor en la búsqueda de texto completo cuando se emparejan con un analizador diseñado para administrar las reglas lingüísticas del idioma de destino.

  ![Captura de pantalla que muestra la pantalla Campos del ejemplo de inmobiliaria real. Un grupo de campos se resalta para indicar cómo se corresponden las asignaciones del analizador de idioma con los idiomas de los campos resaltados.](./media/search-filters-language/lang-fields.png)

> [!Note]
> Para ver ejemplos de código que muestran definiciones de campo con analizadores de idiomas, consulte [Definición de un índice (. NET)](./search-get-started-dotnet.md) y [Definición de un índice (REST)](./search-get-started-powershell.md).

## <a name="build-and-load-an-index"></a>Creación y carga de un índice

Un paso intermedio (y, quizás, obvio) es que tiene que [crear y completar el índice](./search-get-started-dotnet.md) antes de formular una consulta. Hemos mencionado este paso aquí para mayor integridad. Una manera de determinar si el índice está disponible es comprobar la lista de índices en el [portal](https://portal.azure.com).

## <a name="constrain-the-query-and-trim-results"></a>Restricción de la consulta y recorte de los resultados

Los parámetros de la consulta se usan para limitar la búsqueda a campos específicos y luego recortar los resultados de los campos que no resulten útiles para su escenario. Dado un objetivo de restricción de la búsqueda a los campos que contienen cadenas en francés, usaría **searchFields** como destino de la consulta en los campos que contienen cadenas en ese idioma. 

De forma predeterminada, una búsqueda devuelve todos los campos que están marcados como recuperables. Por lo tanto, puede que quiera excluir los campos que no se ajustan a la experiencia de búsqueda específica del idioma que desea proporcionar. En concreto, si limita la búsqueda a un campo con cadenas en francés, probablemente desee excluir los campos con cadenas en inglés de los resultados. Mediante el parámetro de consulta **$select** puede controlar qué campos se devuelven a la aplicación que realiza la llamada.

```csharp
parameters =
    new SearchParameters()
    {
        searchFields = "description_fr" 
        Select = new[] { "description_fr"  }
    };
```
> [!Note]
> Aunque no hay ningún argumento $filter en la consulta, este caso de uso está fuertemente vinculado a los conceptos de filtro, por lo que se presenta como un escenario de filtrado.

## <a name="see-also"></a>Consulte también

+ [Filtros de Azure Cognitive Search](search-filters.md)
+ [Analizadores de idiomas](/rest/api/searchservice/language-support)
+ [Funcionamiento de la búsqueda de texto completo en Azure Cognitive Search](search-lucene-query-architecture.md)
+ [API de REST de documentos de búsqueda](/rest/api/searchservice/search-documents)