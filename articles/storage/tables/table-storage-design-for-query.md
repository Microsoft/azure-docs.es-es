---
title: Diseño de almacenamiento de tablas de Azure para consultas | Microsoft Docs
description: Diseñe tablas para consultas en el almacenamiento de tablas de Azure. Elija una clave de partición adecuada, optimice las consultas y ordene los datos de Table service.
services: storage
author: tamram
ms.author: tamram
ms.service: storage
ms.topic: article
ms.date: 04/23/2018
ms.subservice: tables
ms.openlocfilehash: a15415ab7f5e01619a4a022d7254ef3995a825b0
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88236342"
---
# <a name="design-for-querying"></a>Diseño de consulta
Las soluciones de Table service pueden requerir mucha lectura, escritura o una combinación de ambas. Este artículo se centra en los aspectos que se deben tener en cuenta al diseñar su instancia de Table service para admitir operaciones de lectura de forma eficaz. Normalmente, un diseño que admite operaciones de lectura eficazmente también es eficaz para las operaciones de escritura. Sin embargo, hay algunos otros aspectos que hay que tener en cuenta durante el diseño para admitir operaciones de escritura y que se explican en el artículo [Diseño para la modificación de datos](table-storage-design-for-modification.md).

Un buen punto de partida para diseñar la solución de Table service para que pueda leer los datos de manera eficiente es preguntar "¿Qué consultas necesitará ejecutar mi aplicación para recuperar los datos que necesita de Table service?"  

> [!NOTE]
> Con Table service, es importante obtener el diseño correcto por adelantado, ya que resulta difícil y caro cambiarlo posteriormente. Por ejemplo, en una base de datos relacional a menudo resulta posible resolver problemas de rendimiento agregando índices a una base de datos existente: esto no es una opción con Table service.  
> 
> 

Esta sección se centra en los problemas clave que se deben solucionar al diseñar las tablas para las consultas. Entre los temas tratados en esta sección se incluyen:

* [Cómo afecta al rendimiento de las consultas su elección de PartitionKey y RowKey](#how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance)
* [Elegir un PartitionKey apropiado](#choosing-an-appropriate-partitionkey)
* [Optimización de consultas para Table service](#optimizing-queries-for-the-table-service)
* [Ordenación de los datos de Table service](#sorting-data-in-the-table-service)

## <a name="how-your-choice-of-partitionkey-and-rowkey-impacts-query-performance"></a>Cómo afecta al rendimiento de las consultas su elección de PartitionKey y RowKey
Los ejemplos siguientes asumen que Table service almacena las entidades employee con la estructura siguiente (la mayoría de los ejemplos omiten la propiedad **Timestamp** para mayor claridad):  

| *Nombre de la columna* | *Tipo de datos* |
| --- | --- |
| **PartitionKey** (nombre de departamento) |String |
| **RowKey** (Identificación de empleado) |String |
| **Nombre** |String |
| **Apellidos** |String |
| **Age** |Entero |
| **EmailAddress** |String |

En el artículo [Introducción a Azure Table Storage](table-storage-overview.md) se describen algunas de las características clave de Azure Table service que influyen directamente en el diseño de consultas. Estos dan como resultado las siguientes directrices generales para diseñar consultas de Table service. Tenga en cuenta que la sintaxis de filtro utilizada en los ejemplos siguientes es de la API de REST de Table service. Para más información, consulte [Entidades de consulta](https://docs.microsoft.com/rest/api/storageservices/Query-Entities).  

* Una ***consulta de punto*** es la búsqueda más eficaz que puede usar y se recomienda para búsquedas de gran volumen o búsquedas que requieren menor latencia. Este tipo de consulta puede utilizar los índices para localizar una entidad individual con gran eficacia si se especifican los valores **PartitionKey** y **RowKey**. Por ejemplo: $filter=(PartitionKey eq 'Sales') y (RowKey eq '2')  
* La segunda opción más eficaz es una ***Consulta por rango*** que use **PartitionKey** y filtre un rango de valores **RowKey** para devolver más de una entidad. El valor **PartitionKey** identifica una partición específica y los valores **RowKey** identifican un subconjunto de las entidades de esa partición. Por ejemplo: $filter=PartitionKey eq 'Sales”, RowKey ge 'S' y RowKey lt 'T'  
* En tercer lugar, tiene un ***Examen de partición*** que usa **PartitionKey** y filtra otra propiedad no clave y que puede devolver más de una entidad. El valor **PartitionKey** identifica una partición específica y los valores de propiedad seleccionan un subconjunto de las entidades de esa partición. Por ejemplo: $filter=PartitionKey eq 'Sales' y LastName eq 'Smith'  
* Un ***Examen de tabla*** no incluye la **PartitionKey** y es bastante ineficaz, ya que busca en todas las particiones que componen la tabla todas las entidades coincidentes. Realizará un recorrido de tabla independientemente de si su filtro usa **RowKey**. Por ejemplo: $filter=LastName eq 'Jones'  
* Las consultas que devuelven varias entidades las devuelven ordenadas en orden **PartitionKey** y **RowKey**. Para evitar reordenar las entidades del cliente, seleccione un **RowKey** que defina el criterio de ordenación más común.  

Tenga en cuenta que si usa un "**or**" para especificar un filtro basado en valores **RowKey**, se generará un examen de partición y no se tratará como una consulta de intervalo. Por lo tanto, debe evitar las consultas que usan filtros como: $filter=PartitionKey eq 'Sales' y (RowKey eq '121' o RowKey eq '322')  

Para obtener ejemplos de código de cliente que utilizan la biblioteca de clientes de Storage para ejecutar consultas eficaces, consulte:  

* [Execute a point query using the Storage Client Library](table-storage-design-patterns.md#executing-a-point-query-using-the-storage-client-library) (Ejecución de una consulta de punto mediante la biblioteca de clientes de Storage)
* [Retrieve multiple entities using LINQ](table-storage-design-patterns.md#retrieving-multiple-entities-using-linq) (Recuperación de varias entidades mediante LINQ)
* [Proyección de servidor](table-storage-design-patterns.md#server-side-projection)  

Para obtener ejemplos de código de cliente que pueda administrar varios tipos de entidad almacenados en la misma tabla, consulte:  

* [Work with heterogeneous entity types](table-storage-design-patterns.md#working-with-heterogeneous-entity-types) (Uso de tipos de entidad heterogéneos)  

## <a name="choosing-an-appropriate-partitionkey"></a>Elegir un PartitionKey apropiado
La elección del **PartitionKey** debe equilibrar la necesidad de habilitar el uso de EGT (para garantizar la coherencia) frente a la necesidad de distribuir las entidades en varias particiones (para asegurar una solución escalable).  

En un extremo, puede almacenar todas las entidades en una sola partición, pero esto puede limitar la escalabilidad de su solución y podría impedir a Table service equilibrar la carga de las solicitudes. En el otro extremo, puede almacenar una entidad por cada partición que sería muy escalable y que permite a Table service equilibrar la carga de las solicitudes, pero que podrían impedir que se utilicen transacciones de grupo de la entidad.  

Un **PartitionKey** idóneo es el que permite utilizar consultas eficaces y que tiene suficientes particiones para asegurarse de que su solución es escalable. Normalmente, encontrará que las entidades tendrán una propiedad adecuada que distribuye las entidades entre particiones suficientes.

> [!NOTE]
> Por ejemplo, en un sistema que almacena información acerca de los usuarios o empleados, el identificador del usuario puede ser un buen PartitionKey. Puede que tenga varias entidades que utilizan un determinado identificador de usuario como clave de partición. Cada entidad que almacena los datos de un usuario se agrupa en una sola partición y, de esta manera estas entidades están accesibles a través de las transacciones de grupo de entidad, y continúan siendo altamente escalables.
> 
> 

Existen otros aspectos adicionales que se deben considerar al elegir **PartitionKey**, que están relacionados con la forma de insertar, actualizar y eliminar entidades. Para más información, consulte [Diseño para la modificación de datos](table-storage-design-for-modification.md).  

## <a name="optimizing-queries-for-the-table-service"></a>Optimización de consultas para Table service
Table service indexará automáticamente las entidades mediante los valores **PartitionKey** y **RowKey** en un índice agrupado único, por lo tanto, este es el motivo por el que las consultas de punto son las más eficaces. Sin embargo, no hay ningún índice distinto del índice agrupado en **PartitionKey** y **RowKey**.

Muchos diseños deben cumplir los requisitos para habilitar la búsqueda de entidades según varios criterios. Por ejemplo, localizar las entidades de empleado en función de correo electrónico, identificador de empleado o apellido. Los patrones descritos en [Patrones de diseño de tablas](table-storage-design-patterns.md) abordan estos tipos de requisitos y describen formas de trabajar en torno al hecho de que Table service no proporciona índices secundarios:  

* [Patrón de índice secundario dentro de la partición](table-storage-design-patterns.md#intra-partition-secondary-index-pattern): almacenar varias copias de cada entidad con diferentes valores **RowKey** (en la misma partición) para habilitar búsquedas rápidas y eficaces y ordenaciones alternativas mediante el uso de diferentes valores **RowKey**.  
* [Patrón de índice secundario entre particiones](table-storage-design-patterns.md#inter-partition-secondary-index-pattern): almacenar varias copias de cada entidad con diferentes valores **RowKey** en particiones o en tablas independientes para habilitar búsquedas rápidas y eficaces y ordenaciones alternativas mediante el uso de diferentes valores **RowKey**.  
* [Patrón de entidades de índice](table-storage-design-patterns.md#index-entities-pattern) : mantener las entidades de índice para habilitar búsquedas eficaces que devuelvan listas de entidades.  

## <a name="sorting-data-in-the-table-service"></a>Ordenación de los datos de Table service
Table service devuelve entidades ordenadas en orden ascendente según **PartitionKey** y, a continuación, por **RowKey**. Estas claves son valores de cadena y para asegurarse de que los valores numéricos se ordenen correctamente, debe convertirlos a una longitud fija y rellenarlos con ceros. Por ejemplo, si el valor de identificador de empleado que utiliza como **RowKey** es un valor entero, debe convertir el identificador de empleado **123** en **00000123**.  

Muchas aplicaciones tienen requisitos para utilizar datos ordenados en distintos órdenes: por ejemplo, ordenar los empleados por su nombre o por su fecha de contratación. Los patrones siguientes abordan el modo de alternar órdenes de clasificación para las entidades:  

* [Patrón de índice secundario dentro de la partición](table-storage-design-patterns.md#intra-partition-secondary-index-pattern): almacenar varias copias de cada entidad con diferentes valores RowKey (en la misma partición) para habilitar búsquedas rápidas y eficaces y ordenaciones alternativas mediante el uso de diferentes valores RowKey.  
* [Patrón de índice secundario entre particiones](table-storage-design-patterns.md#inter-partition-secondary-index-pattern): almacenar varias copias de cada entidad con diferentes valores RowKey en particiones en tablas independientes para habilitar búsquedas rápidas y eficaces y ordenaciones alternativas mediante el uso de diferentes valores RowKey.
* [Patrón final del registro](table-storage-design-patterns.md#log-tail-pattern) : recupere las entidades *n* agregadas recientemente a una partición utilizando un valor **RowKey** que se ordene en orden de fecha y hora inverso.  

## <a name="next-steps"></a>Pasos siguientes

- [Patrones de diseño de tablas](table-storage-design-patterns.md)
- [Modelado de relaciones](table-storage-design-modeling.md)
- [Cifrado de datos de tabla](table-storage-design-encrypt-data.md)
- [Diseño para la modificación de datos](table-storage-design-for-modification.md)
