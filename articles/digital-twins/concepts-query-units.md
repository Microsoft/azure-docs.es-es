---
title: Unidades de consulta en Azure Digital Twins
titleSuffix: Azure Digital Twins
description: Descripción del concepto de facturación de las unidades de consulta en Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 8/14/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 5334a1196ac8044c56e615cf8894b44646b48fb4
ms.sourcegitcommit: cd0a1ae644b95dbd3aac4be295eb4ef811be9aaa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88615098"
---
# <a name="query-units-in-azure-digital-twins"></a>Unidades de consulta en Azure Digital Twins 

Una **unidad de consulta** de Azure Digital Twins es una unidad de cálculo a petición que se usa para ejecutar las [consultas de Azure Digital Twins](how-to-query-graph.md) mediante la [API de consulta](https://docs.microsoft.com/rest/api/digital-twins/dataplane/query). 

Abstrae los recursos del sistema, como CPU, IOPS y memoria, que son necesarios para realizar operaciones de consulta admitidas por Azure Digital Twins, lo que le permite realizar un seguimiento del uso en unidades de consulta en su lugar.

La complejidad de la consulta afecta a la cantidad de unidades de consulta que se consumen para ejecutar esa consulta. 

En este artículo se explica cómo comprender las unidades de consulta y realizar el seguimiento del consumo de la unidad de consulta.

## <a name="find-the-query-unit-consumption-in-azure-digital-twins"></a>Búsqueda del consumo de unidades de consulta en Azure Digital Twins 

Al ejecutar una consulta con la [API de consulta](https://docs.microsoft.com/rest/api/digital-twins/dataplane/query) de Azure Digital Twins, puede examinar el encabezado de respuesta para realizar el seguimiento del número de unidades de consulta que la consulta consume. Busque "query-charge" en la respuesta que se devuelve desde Azure Digital Twins. 

Los [SDK](how-to-use-apis-sdks.md) de Azure Digital Twins permiten extraer el encabezado de cargo de consulta de la respuesta paginable. En esta sección se muestra cómo consultar los gemelos digitales y cómo recorrer en iteración la respuesta paginable para extraer el encabezado del cargo de la consulta. 

En el fragmento de código siguiente se muestra cómo se pueden extraer los cargos de consulta en los que se incurre cuando se llama a la API de consulta. Recorre en iteración las páginas de respuesta en primer lugar para acceder al encabezado de cargos de consulta y, a continuación, recorre en iteración los resultados de los gemelos digitales dentro de cada página. 
 
```csharp
AsyncPageable<string> asyncPageableResponseWithCharge = client.QueryAsync("SELECT * FROM digitaltwins"); 
int pageNum = 0; 

// The "await" keyword here is required, as a call is made when fetching a new page. 

await foreach (Page<string> page in asyncPageableResponseWithCharge.AsPages()) 
{ 
    Console.WriteLine($"Page {++pageNum} results:"); 

    // Extract the query-charge header from the page 

    if (QueryChargeHelper.TryGetQueryCharge(page, out float queryCharge)) 
    { 
        Console.WriteLine($"Query charge was: {queryCharge}"); 
    } 

    // Iterate over the twin instances. 

    // The "await" keyword is not required here, as the paged response is local. 

    foreach (string response in page.Values) 
    { 
        BasicDigitalTwin twin = JsonSerializer.Deserialize<BasicDigitalTwin>(response); 
        Console.WriteLine($"Found digital twin '{twin.Id}'"); 
    } 
} 
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo hacer consultas en Azure Digital Twins, consulte:
* [*Conceptos: Lenguaje de consulta*](concepts-query-language.md)
* [*Procedimiento: Consulta del grafo gemelo*](how-to-query-graph.md)
* [Documentación de referencia de la API de consulta](https://docs.microsoft.com/rest/api/digital-twins/dataplane/query/querytwins)

Puede encontrar límites relacionados con las consultas de Azure Digital Twins en [*Referencia: Límites de servicio de la versión preliminar pública*](reference-service-limits.md).