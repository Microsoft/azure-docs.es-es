---
title: Consultas de búsqueda en registros de Azure Monitor | Microsoft Docs
description: En este artículo se proporciona un tutorial de introducción a la búsqueda de consultas de registro en Azure Monitor.
services: log-analytics
documentationcenter: ''
author: bwren
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: log-analytics
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 08/06/2018
ms.author: bwren
ms.openlocfilehash: b5860243b6b605288aaf8d6dffe2260bc031a592
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65560840"
---
# <a name="search-queries-in-azure-monitor-logs"></a>Consultas de búsqueda en registros de Azure Monitor

> [!NOTE]
> Debe completar [Get started with Azure Monitor log queries](get-started-queries.md) (Introducción a las consultas de registro de Azure Monitor) antes de completar esta lección.

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

Las consultas de registro de Azure Monitor pueden comenzar por un nombre de tabla o un comando de búsqueda. Este tutorial trata las consultas de búsqueda. Cada método tiene sus ventajas.

Las consultas basadas en tablas comienzan por definir el ámbito de la consulta y, por tanto, tienden a ser más eficaces que las de búsqueda. Las consultas de búsqueda son menos estructuradas, lo que las convierte en la mejor opción cuando se busca un valor específico en columnas o tablas. Con **search** se pueden examinar todas las columnas de una tabla determinada o en todas las tablas para encontrar el valor especificado. La cantidad de datos que se procesa puede ser enorme, por lo que estas consultas podrían tardar más en completarse y devolver conjuntos de resultados muy grandes.

## <a name="search-a-term"></a>Búsqueda de un término
El comando **search** normalmente se usa para buscar un término específico. En el ejemplo siguiente se examinan todas las columnas de todas las tablas en busca del término "error":

```Kusto
search "error"
| take 100
```

Aunque sean fáciles de usar, las consultas sin ámbito como la anterior no son eficaces y pueden devolver numerosos resultados irrelevantes. Lo más recomendable sería buscar en la tabla pertinente o incluso en una columna específica.

### <a name="table-scoping"></a>Ámbito de tabla
Para buscar un término en una tabla específica, agregue `in (table-name)` justo después del operador **search**:

```Kusto
search in (Event) "error"
| take 100
```

o en varias tablas:
```Kusto
search in (Event, SecurityEvent) "error"
| take 100
```

### <a name="table-and-column-scoping"></a>Ámbito de tabla y columna
De forma predeterminada, **search** evalúa todas las columnas del conjunto de datos. Para buscar solo una columna específica, use esta sintaxis:

```Kusto
search in (Event) Source:"error"
| take 100
```

> [!TIP]
> Si usa `==` en lugar de `:`, los resultados incluirían los registros en los que la columna *Source* tienen el valor exacto "error" y únicamente eso. Uso de ':' incluirá los registros donde *origen* tiene valores como "código de error 404" o "Error".

## <a name="case-sensitivity"></a>Diferenciación entre mayúsculas y minúsculas
De forma predeterminada, en la búsqueda de términos no se distingue entre mayúsculas y minúsculas, por lo que al buscar "dns" se pueden producir resultados como "DNS", "dns" o "Dns". Para que en la búsqueda se distinga entre mayúsculas y minúsculas, use la opción `kind`:

```Kusto
search kind=case_sensitive in (Event) "DNS"
| take 100
```

## <a name="use-wild-cards"></a>Uso de caracteres comodín
El comando **search** admite caracteres comodín, al principio, al final o en el medio de un término.

Para buscar términos que comienzan por "win":
```Kusto
search in (Event) "win*"
| take 100
```

Para buscar términos que terminan en ".com":
```Kusto
search in (Event) "*.com"
| take 100
```

Para buscar términos que contienen "www":
```Kusto
search in (Event) "*www*"
| take 100
```

Para buscar términos que comienzan por "corp" y terminan en ".com", por ejemplo, "corp.mydomain.com":

```Kusto
search in (Event) "corp*.com"
| take 100
```

También puede obtener todo el contenido de una tabla utilizando solo un carácter comodín: `search in (Event) *`, pero que sería lo mismo que escribir simplemente `Event`.

> [!TIP]
> Aunque puede usar `search *` para obtener todas las columnas de todas las tablas, se recomienda siempre dirigir las consultas a tablas específicas. Las consultas generales pueden tardar en completarse y puede que se devuelvan demasiados resultados.

## <a name="add-and--or-to-search-queries"></a>Agregue *and* / *or* a las consultas de búsqueda
Use **and** para buscar los registros que contengan varios términos:

```Kusto
search in (Event) "error" and "register"
| take 100
```

Use **or** para obtener los registros que contengan al menos uno de los términos:

```Kusto
search in (Event) "error" or "register"
| take 100
```

Si tiene varias condiciones de búsqueda, puede combinarlas en la misma consulta mediante paréntesis:

```Kusto
search in (Event) "error" and ("register" or "marshal*")
| take 100
```

Los resultados de este ejemplo serían los registros que contienen el término "error" y también "register" o algo que empieza por "marshal".

## <a name="pipe-search-queries"></a>Canalización de consultas de búsqueda
Al igual que cualquier otro comando, **search** se puede canalizar para filtrar, ordenar y agregar los resultados de la búsqueda. Por ejemplo, para obtener el número de registros *Event* que contienen "win":

```Kusto
search in (Event) "win"
| count
```




## <a name="next-steps"></a>Pasos siguientes

- Consulte más tutoriales en el [sitio del lenguaje de consulta de Kusto](/azure/kusto/query/).
