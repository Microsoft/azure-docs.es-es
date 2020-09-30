---
title: Trabajo con cadenas en consultas de registro de Azure Monitor | Microsoft Docs
description: En este artículo se proporciona un tutorial para usar el análisis de registros de Azure Monitor en Azure Portal para consultar y analizar datos de registro en Azure Monitor.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 08/16/2018
ms.openlocfilehash: 5021b815da1419052ceb4d6114fbb2bc676fe8d6
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90088406"
---
# <a name="working-with-json-and-data-structures-in-azure-monitor-log-queries"></a>Trabajo con JSON y estructuras de datos en consultas de registro de Azure Monitor

> [!NOTE]
> Antes de realizar esta lección, debe seguir los pasos de [Introducción a Log Analytics de Azure Monitor](get-started-portal.md) e [Introducción a las consultas de registro de Azure Monitor](get-started-queries.md).

[!INCLUDE [log-analytics-demo-environment](../../../includes/log-analytics-demo-environment.md)]

Los objetos anidados son objetos que contienen otros objetos en una matriz o un mapa de pares clave-valor. Estos objetos se representan como cadenas JSON. En este artículo se describe cómo se usa JSON para recuperar datos y analizar objetos anidados.

## <a name="working-with-json-strings"></a>Trabajo con cadenas JSON
Use `extractjson` para acceder a un elemento JSON específico en una ruta de acceso conocida. Esta función requiere una expresión de ruta de acceso que use las siguientes convenciones.

- _$_ para hacer referencia a la carpeta raíz
- Use la notación de corchetes o puntos para hacer referencia a los índices y elementos, tal como se muestra en los ejemplos siguientes.


Use corchetes para los índices y puntos para separar los elementos:

```Kusto
let hosts_report='{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}';
print hosts_report
| extend status = extractjson("$.hosts[0].status", hosts_report)
```

Este es el mismo resultado usando solo la notación de corchetes:

```Kusto
let hosts_report='{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}';
print hosts_report 
| extend status = extractjson("$['hosts'][0]['status']", hosts_report)
```

Si hay solo un elemento, puede usar únicamente la notación de puntos:

```Kusto
let hosts_report=dynamic({"location":"North_DC", "status":"running", "rate":5});
print hosts_report 
| extend status = hosts_report.status
```


## <a name="working-with-objects"></a>Trabajar con objetos

### <a name="parsejson"></a>parsejson
Para acceder a varios elementos en la estructura JSON, es más fácil acceder como un objeto dinámico. Use `parsejson` para convertir los datos de texto en un objeto dinámico. Después de la conversión en un tipo dinámico, se pueden usar funciones adicionales para analizar los datos.

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}');
print hosts_object 
| extend status0=hosts_object.hosts[0].status, rate1=hosts_object.hosts[1].rate
```



### <a name="arraylength"></a>arraylength
Use `arraylength` para contar el número de elementos en una matriz:

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}');
print hosts_object 
| extend hosts_num=arraylength(hosts_object.hosts)
```

### <a name="mvexpand"></a>mvexpand
Use `mvexpand` para dividir las propiedades de un objeto en filas independientes.

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}');
print hosts_object 
| mvexpand hosts_object.hosts[0]
```

![Captura de pantalla que muestra hosts_0 con valores para la ubicación, el estado y la tasa.](media/json-data-structures/mvexpand.png)

### <a name="buildschema"></a>buildschema
Use `buildschema` para obtener el esquema que admite todos los valores de un objeto:

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"location":"South_DC", "status":"stopped", "rate":3}]}');
print hosts_object 
| summarize buildschema(hosts_object)
```

El resultado es un esquema en formato JSON:
```json
{
    "hosts":
    {
        "indexer":
        {
            "location": "string",
            "rate": "int",
            "status": "string"
        }
    }
}
```
Esta salida describe los nombres de los campos del objeto y sus tipos de datos coincidentes. 

Los objetos anidados pueden tener esquemas diferentes, como en el ejemplo siguiente:

```Kusto
let hosts_object = parsejson('{"hosts": [{"location":"North_DC", "status":"running", "rate":5},{"status":"stopped", "rate":"3", "range":100}]}');
print hosts_object 
| summarize buildschema(hosts_object)
```


![Compilación de esquema](media/json-data-structures/buildschema.png)

## <a name="next-steps"></a>Pasos siguientes
Consulte otras lecciones de uso de consultas de registro en Azure Monitor:

- [Operaciones de cadena](string-operations.md)
- [Operaciones de fecha y hora](datetime-operations.md)
- [Funciones de agregación](aggregations.md)
- [Agregaciones avanzadas](advanced-aggregations.md)
- [Escritura de consultas avanzadas](advanced-query-writing.md)
- [Combinaciones](joins.md)
- [Gráficos](charts.md)
