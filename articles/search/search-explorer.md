---
title: Herramienta de consulta Explorador de búsqueda de Azure Portal
titleSuffix: Azure Cognitive Search
description: En esta guía de inicio rápido de Azure Portal, use el Explorador de búsqueda para aprender la sintaxis de las consultas, probar expresiones de consulta o inspeccionar un documento de búsqueda. El Explorador de búsqueda consulta los índices en Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: quickstart
ms.date: 06/07/2020
ms.openlocfilehash: 8f004ff8118829ebf2d245a136cf22cdde23accc
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929795"
---
# <a name="quickstart-use-search-explorer-to-run-queries-in-the-portal"></a>Inicio rápido: Uso del Explorador de búsqueda para ejecutar consultas en el portal

El **Explorador de búsqueda** es una herramienta de consulta integrada que se usa para ejecutar consultas en un índice de búsqueda en Azure Cognitive Search. Esta herramienta facilita aprender la sintaxis de las consultas, probar una expresión de consulta o filtro, o verificar si existe contenido nuevo en el índice para confirmar la actualización de datos.

En esta guía de inicio rápido se usa un índice existente para hacer una demostración del Explorador de búsqueda. Las solicitudes se formulan mediante la [API de REST de Search](/rest/api/searchservice/), donde las respuestas se devuelven como documentos JSON.

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar, debe disponer de lo siguiente:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/).

+ Un servicio Azure Cognitive Search. [Cree un servicio](search-create-service-portal.md) o [busque uno existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) en su suscripción actual. Puede usar un servicio gratuito para este inicio rápido. 

+ En esta guía de inicio rápido se usa *realestate-us-sample-index*. Use el asistente para la [**importación de datos**](search-import-data-portal.md) para crear el índice. En el primer paso, cuando se le pida el origen de datos, elija **Ejemplos** y, después, seleccione el origen de datos **realestate-us-sample**. Acepte todos los valores predeterminados del asistente para crear el índice.

## <a name="start-search-explorer"></a>Inicio del Explorador de búsqueda

1. En [Azure Portal](https://portal.azure.com), abra la página del servicio de búsqueda desde el panel o [busque el servicio](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices).

1. Abra el Explorador de búsqueda desde la barra de comandos:

   ![Comando del Explorador de búsqueda en el portal](./media/search-explorer/search-explorer-cmd2.png "Comando del explorador de búsqueda en el portal")

    O bien, use la pestaña **Explorador de búsqueda** insertada en un índice abierto:

   ![Pestaña Explorador de búsqueda](./media/search-explorer/search-explorer-tab.png "Pestaña Explorador de búsqueda")

## <a name="unspecified-query"></a>Consulta sin especificar

Para un echar un primer vistazo al contenido, ejecute una búsqueda vacía haciendo clic en **Buscar** sin especificar ningún término. Una búsqueda vacía resulta útil como primera consulta porque devuelve documentos completos para que pueda revisar la composición del documento. En una búsqueda vacía, no hay ninguna clasificación de búsqueda y los documentos se devuelven en orden arbitrario (`"@search.score": 1` para todos los documentos). De forma predeterminada, en una solicitud de búsqueda se devuelven 50 documentos.

La sintaxis equivalente para una búsqueda vacía es `*` o `search=*`.
   
   ```http
   search=*
   ```

   **Resultados**
   
   ![Ejemplo de consulta vacío](./media/search-explorer/search-explorer-example-empty.png "Ejemplo de consulta vacía o incompleta")

## <a name="free-text-search"></a>Búsqueda de texto libre

Las consultas de forma libre, con o sin operadores, resultan útiles para simular consultas definidas por el usuario enviadas desde una aplicación personalizada a Azure Cognitive Search. Solo se examinan las coincidencias de los campos atribuidos como **Buscable** en la definición del índice. 

Tenga en cuenta que, al proporcionar criterios de búsqueda, como expresiones o términos de consulta, entra en juego la clasificación de búsqueda. El ejemplo siguiente ilustra una búsqueda de texto libre.

   ```http
   Seattle apartment "Lake Washington" miele OR thermador appliance
   ```

   **Resultados**

   Puede usar Ctrl-F para buscar términos específicos de interés en los resultados.

   ![Ejemplo de consulta de texto libre](./media/search-explorer/search-explorer-example-freetext.png "Ejemplo de consulta de texto libre")

## <a name="count-of-matching-documents"></a>Recuento de documentos coincidentes 

Agregue **$count=true** para obtener el número de coincidencias encontradas en un índice. En una búsqueda vacía, el recuento corresponde al número total de documentos en el índice. En una búsqueda completa, corresponde al número de documentos que coinciden con la entrada de la consulta.

   ```http
   $count=true
   ```

   **Resultados**

   ![Ejemplo del número de documentos](./media/search-explorer/search-explorer-example-count.png "Número de documentos coincidentes en el índice")

## <a name="limit-fields-in-search-results"></a>Limitación de campos en los resultados de la búsqueda

Agregue [ **$select**](search-query-odata-select.md) para limitar los resultados a los campos designados de manera explícita para obtener una salida más legible en el **Explorador de búsqueda**. Para mantener la cadena de búsqueda y **$count = true**, anteponga **&** a los argumentos. 

   ```http
   search=seattle condo&$select=listingId,beds,baths,description,street,city,price&$count=true
   ```

   **Resultados**

   ![Ejemplo de campos de límite](./media/search-explorer/search-explorer-example-selectfield.png "Restricción de campos en los resultados de la búsqueda")

## <a name="return-next-batch-of-results"></a>Devolución del lote siguiente de resultados

Azure Cognitive Search devuelve las primeras 50 coincidencias en función de la clasificación de búsqueda. Para obtener el siguiente conjunto de documentos coincidentes, anexe **$top=100,&$skip=50** para aumentar el conjunto de resultados a 100 documentos (el valor predeterminado es 50, el máximo es 1000), lo que omite los primeros 50 documentos. Recuerde que debe proporcionar criterios de búsqueda, como una expresión o un término de consulta, para obtener los resultados clasificados. Tenga en cuenta que las puntuaciones de búsqueda disminuyen cuanto más profundamente se llega en los resultados de la búsqueda.

   ```http
   search=seattle condo&$select=listingId,beds,baths,description,street,city,price&$count=true&$top=100&$skip=50
   ```

   **Resultados**

   ![Resultados de la búsqueda por lotes](./media/search-explorer/search-explorer-example-topskip.png "Devolución del lote siguiente de los resultados de búsqueda")

## <a name="filter-expressions-greater-than-less-than-equal-to"></a>Expresiones de filtro (mayor que, menor que, igual a)

Utilice el parámetro [ **$filter**](search-query-odata-filter.md) cuando quiera especificar criterios precisos en lugar de búsqueda de texto libre. El campo se debe atribuir como **Filtrable** en el índice. En este ejemplo se buscan más de tres dormitorios:

   ```http
   search=seattle condo&$filter=beds gt 3&$count=true
   ```
   
   **Resultados**

   ![Expresión de filtro](./media/search-explorer/search-explorer-example-filter.png "Filtrar por criterios")

## <a name="order-by-expressions"></a>Expresiones OrderBy

Agregue [ **$orderby**](search-query-odata-orderby.md) para ordenar los resultados por otro campo además de la puntuación de búsqueda. El campo se debe atribuir como **Ordenable** en el índice. Una expresión de ejemplo que puede usar para probar este caso es:

   ```http
   search=seattle condo&$select=listingId,beds,price&$filter=beds gt 3&$count=true&$orderby=price asc
   ```
   
   **Resultados**

   ![Expresión OrderBy](./media/search-explorer/search-explorer-example-ordery.png "Cambio del criterio de ordenación")

Ambas expresiones, **$filter** y **$orderby** son construcciones de OData. Para más información, consulte la [sintaxis de filtro de OData](/rest/api/searchservice/odata-expression-syntax-for-azure-search).

<a name="start-search-explorer"></a>

## <a name="takeaways"></a>Puntos clave

En esta guía de inicio rápido, ha usado el **Explorador de búsqueda** para consultar un índice mediante la API de REST.

+ Los resultados se devuelven como documentos JSON detallados para que pueda ver la construcción y el contenido del documento en su totalidad. Puede usar expresiones de consulta, que se muestran en los ejemplos, para limitar los campos que se devuelven.

+ Los documentos se componen de todos los campos marcados como **Recuperable** en el índice. Para ver los atributos del índice en el portal, haga clic en *realestate-us-sample* en la lista **Índices** de la página de información general de búsqueda.

+ Las consultas de forma libre, similares a las que se pueden escribir en un explorador web comercial, resultan útiles para probar una experiencia de usuario final. Por ejemplo, si partimos del índice realestate de ejemplo integrado, podría escribir "Apartamentos Seattle Lake Washington" y luego usar Ctrl-F para buscar términos dentro de los resultados de búsqueda. 

+ Las expresiones de consulta y de filtro se articulan en una sintaxis admitida por Azure Cognitive Search. El valor predeterminado es una [sintaxis simple](/rest/api/searchservice/simple-query-syntax-in-azure-search), pero también puede usar la sintaxis de [Lucene completa](/rest/api/searchservice/lucene-query-syntax-in-azure-search) para realizar consultas más eficaces. Las [expresiones de filtro](/rest/api/searchservice/odata-expression-syntax-for-azure-search) son una sintaxis de OData.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

Si está usando un servicio gratuito, recuerde que está limitado a tres índices, indexadores y orígenes de datos. Puede eliminar elementos individuales en el portal para mantenerse por debajo del límite. 

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre las estructuras y la sintaxis de consulta, use Postman o una herramienta equivalente para crear expresiones de consulta que aprovechen más partes de la API. La [API de REST de Search](/rest/api/searchservice/) es especialmente útil para el aprendizaje y la exploración.

> [!div class="nextstepaction"]
> [Creación de una consulta básica en Postman](search-query-simple-examples.md)