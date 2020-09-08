---
title: Diseño de tablas de Azure Cosmos DB para el escalado y el rendimiento
description: 'Guía de diseño de Azure Table Storage: tablas escalables y eficaces en Azure Cosmos DB y Azure Table Storage'
ms.service: cosmos-db
ms.subservice: cosmosdb-table
ms.topic: how-to
ms.date: 06/19/2020
author: sakash279
ms.author: akshanka
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 05a469dbeb093c41b45be278aec42cc930223c72
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002183"
---
# <a name="azure-table-storage-table-design-guide-scalable-and-performant-tables"></a>Guía de diseño de tablas de Azure Table Storage: Tablas escalables y eficaces

[!INCLUDE [storage-table-cosmos-db-tip-include](../../includes/storage-table-cosmos-db-tip-include.md)]

Para diseñar tablas escalables y eficaces, debe tener en cuenta diversos factores, como el costo. Si ha diseñado previamente esquemas para bases de datos relacionales, estas consideraciones le resultarán familiares. Pero aunque hay algunas similitudes entre Azure Table Storage y los modelos relacionales, hay también muchas diferencias importantes. Normalmente, estas diferencias provocan diseños diferentes que pueden parecer no intuitivos o incorrectos a alguien que esté familiarizado con las bases de datos relacionales, pero que sí tienen sentido si va a diseñar un almacén de claves/valores de NoSQL, como Table Storage.

Table Storage está diseñado para admitir aplicaciones de escala de nube que pueden contener miles de millones de entidades ("filas" en terminología de bases de datos relacionales) de datos o conjuntos de datos que deben admitir volúmenes elevados de transacciones. Por lo tanto, debe pensar de forma diferente en cómo almacenar los datos y comprender cómo funciona Table Storage. Un almacén de datos NoSQL bien diseñado puede permitir a su solución escalar mucho más (y a un costo más bajo) que una solución que utiliza una base de datos relacional. Esta guía le ayuda con estos temas.  

## <a name="about-azure-table-storage"></a>Acerca de Azure Table Storage
En esta sección se resaltan algunas de las características clave de Table Storage que son especialmente importantes para obtener un diseño que confiera rendimiento y escalabilidad. Si no está familiarizado con Azure Storage y Table Storage, consulte primero [Introducción a Microsoft Azure Storage](../storage/common/storage-introduction.md) e [Introducción a Azure Table Storage mediante .NET](table-storage-how-to-use-dotnet.md) antes de leer el resto de este artículo. Aunque esta guía se centra en Table Storage, incluye información sobre los servicios Azure Queue Storage y Azure Blob Storage, y cómo usarlos junto con Table Storage en una solución.  

Table Storage usa un formato tabular para almacenar los datos. En la terminología estándar, cada fila de la tabla representa una entidad y las columnas almacenan las distintas propiedades de la entidad. Cada entidad tiene un par de claves que la identifican de forma única, y una columna de marca de tiempo que usa Table Storage para realizar el seguimiento de cuándo se actualizó la entidad por última vez. El campo de la marca de tiempo se agrega automáticamente, y no se puede sobrescribir manualmente con un valor arbitrario. Table Storage usa esta última marca de tiempo modificada (LMT) para administrar la simultaneidad optimista.  

> [!NOTE]
> Las operaciones de API REST de Table Storage también devuelven un valor `ETag` que se obtiene de la LMT. En este documento se utilizan los términos ETag y LMT indistintamente porque hacen referencia a los mismos datos subyacentes.  
> 
> 

En el ejemplo siguiente se muestra el diseño de una tabla sencilla para almacenar las entidades employee y department. Muchos de los ejemplos que se muestran más adelante en esta guía se basan en este diseño simple.  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td>Marketing</td>
<td>00001</td>
<td>2014-08-22T00:50:32Z</td>
<td>
<table>
<tr>
<th>Nombre</th>
<th>Apellidos</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Don</td>
<td>Hall</td>
<td>34</td>
<td>donh@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>00002</td>
<td>2014-08-22T00:50:34Z</td>
<td>
<table>
<tr>
<th>Nombre</th>
<th>Apellidos</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Jun</td>
<td>Cao</td>
<td>47</td>
<td>junc@contoso.com</td>
</tr>
</table>
</tr>
<tr>
<td>Marketing</td>
<td>department</td>
<td>2014-08-22T00:50:30Z</td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>Marketing</td>
<td>153</td>
</tr>
</table>
</td>
</tr>
<tr>
<td>Sales</td>
<td>00010</td>
<td>2014-08-22T00:50:44Z</td>
<td>
<table>
<tr>
<th>Nombre</th>
<th>Apellidos</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Ken</td>
<td>Kwok</td>
<td>23</td>
<td>kenk@contoso.com</td>
</tr>
</table>
</td>
</tr>
</table>


Hasta ahora, este diseño es similar a una tabla en una base de datos relacional. Las principales diferencias son las columnas obligatorias y la capacidad de almacenar varios tipos de entidad en la misma tabla. Además, cada una de las propiedades definidas por el usuario, como **FirstName** o **Age**, tienen un tipo de datos, como un número entero o una cadena, al igual que una columna en una base de datos relacional. Aunque, a diferencia de una base de datos relacional, la naturaleza sin esquema de Table Storage significa que una propiedad no necesita tener los mismos tipos de datos en cada entidad. Para almacenar tipos de datos complejos en una sola propiedad, debe utilizar un formato serializado como JSON o XML. Para obtener más información, consulte [Descripción del modelo de datos de Table Storage](https://msdn.microsoft.com/library/azure/dd179338.aspx).

La elección de `PartitionKey` y `RowKey` es fundamental para un buen diseño de tabla. Todas las entidades almacenadas en una tabla deben tener una combinación única de `PartitionKey` y `RowKey`. Al igual que con las claves de una tabla de base de datos relacional, los valores `PartitionKey` y `RowKey` se indexan para crear un índice agrupado para permitir búsquedas rápidas. El servicio Table Storage, sin embargo, no crea ningún índice secundario, por lo que estas son las dos únicas propiedades indexadas (algunos de los patrones que se describen más adelante muestran cómo puede solucionar esta limitación aparente).  

Una tabla consta de una o varias particiones, y muchas de las decisiones de diseño que tome estarán relacionadas con la elección de un valor adecuado de `PartitionKey` y `RowKey` para optimizar la solución. Una solución puede constar de una única tabla que contenga todas las entidades organizadas en particiones, pero normalmente una solución tiene varias tablas. Las tablas le ayudan a organizar lógicamente las entidades y a administrar el acceso a los datos mediante listas de control de acceso. Puede quitar una tabla completa mediante una sola operación de almacenamiento.  

### <a name="table-partitions"></a>Particiones de tabla
El nombre de la cuenta, el nombre de la tabla y `PartitionKey` juntos identifican la partición dentro del servicio de almacenamiento donde Table Storage almacena la entidad. Además de ser parte del esquema de direccionamiento de las entidades, las particiones definen un ámbito para las transacciones (vea [Transacciones des grupo de entidades](#entity-group-transactions) más adelante en este mismo artículo) y forman la base de cómo escala Table Storage. Para más información sobre las particiones de tabla, consulte [Lista de comprobación de escalabilidad y rendimiento para Table Storage](../storage/tables/storage-performance-checklist.md).  

En Table Storage, un nodo individual da servicio a una o más particiones completas, y el servicio se escala equilibrando dinámicamente la carga de las particiones entre nodos. Si un nodo está bajo carga, Table Storage puede dividir el intervalo de particiones atendidas por ese nodo en nodos diferentes. Cuando el tráfico disminuye, Table Storage puede combinar los intervalos de la partición de nodos silenciosos en un único nodo.  

Para más información sobre los detalles internos de Table Storage, y en particular cómo administra las particiones, consulte [Microsoft Azure Storage: servicio de almacenamiento en nube altamente disponible y de gran coherencia](https://docs.microsoft.com/archive/blogs/windowsazurestorage/sosp-paper-windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency).  

### <a name="entity-group-transactions"></a>Transacciones de grupos de entidades
En Table Storage, las transacciones de grupo de entidades (EGT) son el único mecanismo integrado para realizar actualizaciones atómicas en varias entidades. Las EGT se conocen también como *transacciones por lotes*. Las EGT funcionan únicamente en entidades almacenadas en la misma partición (comparten la misma clave de partición en una tabla determinada), por lo que siempre que necesite un comportamiento transaccional atómico a través de varias entidades, asegúrese de que las entidades se encuentren en la misma partición. Este suele ser un motivo para mantener varios tipos de entidad en la misma tabla (y partición) y no utilizar varias tablas para diferentes tipos de entidad. Una sola EGT puede operar en 100 entidades como máximo.  Si envía varias EGT simultáneas para procesamiento, es importante asegurarse de que esas EGT no se usan en las entidades que son comunes a todas las EGT, ya que de lo contrario se puede retrasar el procesamiento.

Las EGT también presentan posibles ventajas e inconvenientes que es necesario evaluar en el diseño. El uso de más particiones aumenta la escalabilidad de la aplicación, porque Azure tiene más oportunidades para equilibrar la carga de solicitudes entre los nodos. Sin embargo, también podría limitar la capacidad de la aplicación para realizar transacciones atómicas y mantener una fuerte coherencia de los datos. Además, existen objetivos de escalabilidad específicos en el nivel de una partición que podrían limitar el rendimiento de las transacciones que se puede esperar de un único nodo.

Para obtener más información sobre los objetivos de escalabilidad de cuentas de almacenamiento de Azure, consulte [Objetivos de escalabilidad para cuentas de almacenamiento estándar](../storage/common/scalability-targets-standard-account.md). Para más información acerca de los objetivos de escalabilidad de Table Storage, consulte [Objetivos de escalabilidad y rendimiento para Table Storage](../storage/tables/scalability-targets.md). En secciones posteriores de esta guía se trata sobre diversas estrategias de diseño que le ayudarán a administrar los inconvenientes de este, y se explica cómo elegir mejor su clave de partición según los requisitos específicos de su aplicación cliente.  

### <a name="capacity-considerations"></a>Consideraciones de capacidad
En la tabla siguiente se incluyen algunos de los valores de clave que se deben tener en cuenta al diseñar una solución de Table Storage:  

| Capacidad total de una cuenta de Azure Storage | 500 TB |
| --- | --- |
| Número de tablas en una cuenta de Azure Storage |Solo limitadas por la capacidad de la cuenta de almacenamiento. |
| Número de particiones en una tabla |Solo limitadas por la capacidad de la cuenta de almacenamiento. |
| Número de entidades de una partición |Solo limitadas por la capacidad de la cuenta de almacenamiento. |
| Tamaño de una entidad individual |Hasta 1 MB con un máximo de 255 propiedades (incluyendo `PartitionKey`, `RowKey` y `Timestamp`). |
| Tamaño de `PartitionKey` |Una cadena de hasta 1 KB. |
| Tamaño de `RowKey` |Una cadena de hasta 1 KB. |
| Tamaño de una transacción de un grupo de entidades |Una transacción puede incluir como máximo 100 entidades, y la carga debe ser inferior a 4 MB. Un EGT solo puede actualizar una entidad una vez. |

Para obtener más información, consulte [Descripción del modelo de datos de Table service](https://msdn.microsoft.com/library/azure/dd179338.aspx).  

### <a name="cost-considerations"></a>Consideraciones sobre el costo
Table Storage es relativamente económico, pero debe incluir las estimaciones de costes para el uso de la capacidad y la cantidad de transacciones como parte de la evaluación de cualquier solución que utilice Table Storage. Sin embargo, en muchos escenarios, el almacenamiento de datos duplicados o sin normalizar para mejorar el rendimiento o la escalabilidad de su solución es un enfoque válido que se puede tomar. Para más información sobre los precios, consulte [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/).  

## <a name="guidelines-for-table-design"></a>Directrices para el diseño de tablas
Estas listas resumen algunas de las directrices clave que debe tener en cuenta al diseñar las tablas. En esta guía se abordan todas al detalle más adelante. Estas instrucciones son diferentes de las directrices que seguiría normalmente para el diseño de la base de datos relacional.  

Diseño del almacenamiento Table Storage de manera que su *lectura* sea eficaz:

* **Diseño para realizar consultas en aplicaciones con muchas lecturas.** Al diseñar las tablas, piense en las consultas (especialmente las sensibles a la latencia) que ejecutará antes de pensar cómo actualizará las entidades. Normalmente esto produce una solución eficiente y de rendimiento.  
* **Especifique `PartitionKey` y `RowKey` en las consultas.** *Consultas puntuales* como estas son las consultas más eficaces de Table Storage.  
* **Tenga en cuenta la posibilidad de almacenar copias duplicadas de las entidades.** El almacenamiento de Table Storage es barato, por lo que puede almacenar la misma entidad varias veces (con claves diferentes) para permitir que se realicen consultas más eficaces.  
* **Considere la posibilidad de desnormalizar sus datos.** El almacenamiento de Table Storage es barato, por tanto, piense en desnormalizar sus datos. Por ejemplo, almacene entidades de resumen para que las consultas para datos agregados solo necesiten acceder a una única entidad.  
* **Use valores de clave compuestos.** Las únicas claves que tiene son `PartitionKey` y `RowKey`. Por ejemplo, s los valores de clave compuestos para habilitar rutas de acceso con clave alternativas a las entidades.  
* **Use la proyección de consultas.** Puede reducir la cantidad de datos que se transfieren a través de la red mediante el uso de consultas que seleccionen solo los campos que necesite.  

Diseño del almacenamiento de Table Storage de manera que su *escritura* sea eficaz:  

* **No cree particiones activas.** Elija claves que le permitan distribuir las solicitudes en varias particiones en cualquier momento.  
* **Evite picos de tráfico.** Distribuya el tráfico en un período de tiempo razonable y evite picos de tráfico.
* **No es preciso crear necesariamente una tabla independiente para cada tipo de entidad.** Cuando necesite transacciones atómicas en los tipos de entidad, puede almacenar estos distintos tipos de entidad en la misma partición de la misma tabla.
* **Tenga en cuenta el rendimiento máximo que debe alcanzar.** Debe tener en cuenta los objetivos de escalabilidad de Table Storage y asegurarse de que su diseño no los superará.  

Más adelante en esta guía, verá ejemplos en los que se ponen en práctica todos estos principios.  

## <a name="design-for-querying"></a>Diseño de consulta
Las soluciones de Table Storage pueden requerir mucha lectura, escritura o una combinación de ambas. En esta sección se aborda la manera de configurar un diseño que admita operaciones de lectura de forma eficaz. Normalmente, un diseño que admite operaciones de lectura eficazmente también es eficaz para las operaciones de escritura. Sin embargo, existen consideraciones adicionales para un diseño que admita las operaciones de escritura. Se describen en la sección siguiente, [Diseño para la modificación de datos](#design-for-data-modification).

Un buen punto de partida para que pueda leer los datos de manera eficiente es preguntar "¿Qué consultas necesitará ejecutar mi aplicación para recuperar los datos que necesita?"  

> [!NOTE]
> Con Table Storage, es importante obtener el diseño correcto por adelantado porque resulta difícil y caro cambiarlo más adelante. Por ejemplo, en una base de datos relacional a menudo resulta posible resolver problemas de rendimiento agregando simplemente los índices en una base de datos existente. Esto no es una opción con Table Storage.  

### <a name="how-your-choice-of-partitionkey-and-rowkey-affects-query-performance"></a>De qué manera afecta al rendimiento de las consultas la elección de `PartitionKey` y `RowKey`
Los ejemplos siguientes asumen que Table Storage almacena las entidades de empleado con la estructura siguiente (la mayoría de los ejemplos omiten la propiedad `Timestamp` para mayor claridad):  

| Nombre de la columna | Tipo de datos |
| --- | --- |
| `PartitionKey` (Nombre de departamento) |String |
| `RowKey` (Id. de empleado) |String |
| `FirstName` |String |
| `LastName` |String |
| `Age` |Entero |
| `EmailAddress` |String |

Estas son algunas directrices generales para diseñar consultas de Table Storage. La sintaxis de filtro utilizada en los ejemplos siguientes procede de la API REST de Table Storage. Para más información, consulte [Consulta de entidades](https://msdn.microsoft.com/library/azure/dd179421.aspx).  

* Una *consulta puntual* es la búsqueda más eficaz que puede usar y se recomienda para búsquedas de gran volumen o búsquedas que requieren la menor latencia. Este tipo de consulta puede utilizar los índices para localizar una entidad individual con gran eficacia si se especifican los valores `PartitionKey` y `RowKey`. Por ejemplo: `$filter=(PartitionKey eq 'Sales') and (RowKey eq '2')`.  
* La segunda mejor opción es una *consulta por rango*. Utiliza el `PartitionKey` y filtra en un intervalo de valores `RowKey` para devolver más de una entidad. El valor `PartitionKey` identifica una partición específica y los valores `RowKey` identifican un subconjunto de las entidades de esa partición. Por ejemplo: `$filter=PartitionKey eq 'Sales' and RowKey ge 'S' and RowKey lt 'T'`.  
* La tercera mejor opción es un *examen de partición*. Utiliza `PartitionKey` y filtra en función de otra propiedad que no sea clave y podría devolver más de una entidad. El valor `PartitionKey` identifica una partición específica y los valores de propiedad seleccionan un subconjunto de las entidades de esa partición. Por ejemplo: `$filter=PartitionKey eq 'Sales' and LastName eq 'Smith'`.  
* Un *recorrido de tabla* no incluye `PartitionKey` y es ineficaz, ya que busca en todas las particiones que componen la tabla todas las entidades coincidentes. Realiza un recorrido de tabla independientemente de si su filtro usa `RowKey`. Por ejemplo: `$filter=LastName eq 'Jones'`.  
* Las consultas de Azure Table Storage que devuelven varias entidades las clasifican en orden `PartitionKey` y `RowKey`. Para evitar reordenar las entidades del cliente, seleccione un valor `RowKey` que defina el criterio de ordenación más común. Los resultados de consulta devueltos por la Table API de Azure en Azure Cosmos DB no se ordenan por clave de fila ni por clave de partición. Para obtener una lista detallada de las diferencias entre características, consulte las [diferencias entre Table API de Azure Cosmos DB y Azure Table Storage](table-api-faq.md#table-api-vs-table-storage).

Al usar "**or**" para especificar un filtro basado en valores `RowKey`, se generará un examen de partición y no se tratará como una consulta por rango. Por lo tanto, evite las consultas que utilizan filtros como `$filter=PartitionKey eq 'Sales' and (RowKey eq '121' or RowKey eq '322')`.  

Para obtener ejemplos de código de cliente que utilizan la biblioteca cliente de Storage para ejecutar consultas eficaces, consulte:  

* [Ejecución de una consulta puntual mediante la biblioteca cliente de Storage](#run-a-point-query-by-using-the-storage-client-library)
* [Recuperación de varias entidades mediante LINQ](#retrieve-multiple-entities-by-using-linq)
* [Proyección de servidor](#server-side-projection)  

Para obtener ejemplos de código de cliente que pueda administrar varios tipos de entidad almacenados en la misma tabla, consulte:  

* [Work with heterogeneous entity types](#work-with-heterogeneous-entity-types) (Uso de tipos de entidad heterogéneos)  

### <a name="choose-an-appropriate-partitionkey"></a>Selección de un valor `PartitionKey` apropiado
La elección del `PartitionKey` debe equilibrar la necesidad de habilitar el uso de EGT (para garantizar la coherencia) frente a la necesidad de distribuir las entidades en varias particiones (para asegurar una solución escalable).  

En un extremo, puede almacenar todas las entidades en una sola partición. Pero esto puede limitar la escalabilidad de la solución e impedir que Table Storage equilibre la carga de las solicitudes. En el otro extremo, puede almacenar una entidad por partición. Esta opción es muy escalable y permite a Table Storage equilibrar la carga de las solicitudes, pero impide el uso de transacciones de grupo de entidades.  

Un valor `PartitionKey` idóneo le permite usar consultas eficaces y tiene suficientes particiones para asegurar que la solución es escalable. Normalmente, encontrará que las entidades tendrán una propiedad adecuada que distribuye las entidades entre particiones suficientes.

> [!NOTE]
> Por ejemplo, en un sistema que almacena información acerca de los usuarios o empleados, `UserID` puede ser un buen valor `PartitionKey`. Puede que tenga varias entidades que utilizan un determinado `UserID` como clave de partición. Cada entidad que almacena datos sobre un usuario se agrupa en una sola partición. Se puede acceder a estas entidades a través de EGT, y además sigue siendo muy escalable.
> 
> 

Existen otros aspectos adicionales que se deben considerar al elegir `PartitionKey`, que están relacionados con la forma de insertar, actualizar y eliminar entidades. Para más información, consulte [Diseño para la modificación de datos](#design-for-data-modification) más adelante en este mismo artículo.  

### <a name="optimize-queries-for-table-storage"></a>Optimización de consultas para Table Storage
Table Storage indexa automáticamente las entidades mediante los valores `PartitionKey` y `RowKey` de un índice agrupado único. Este es el motivo por el que las consultas puntuales son la opción más eficaz. Sin embargo, no hay ningún índice distinto del índice agrupado en `PartitionKey` y `RowKey`.

Muchos diseños deben cumplir los requisitos para habilitar la búsqueda de entidades según varios criterios. Por ejemplo, localizar las entidades de empleado en función de correo electrónico, identificador de empleado o apellido. Los siguientes patrones de la sección [Patrones de diseño de tabla](#table-design-patterns) tratan estos tipos de requisitos. Los patrones también describen formas de solucionar el hecho de que Table Storage no proporcione índices secundarios.  

* [Patrón de índice secundario dentro de la partición](#intra-partition-secondary-index-pattern): Almacene varias copias de cada entidad usando valores `RowKey` diferentes (en la misma partición). Esto permite realizar búsquedas rápidas y eficaces, así como alternar criterios de ordenación utilizando valores `RowKey` diferentes.  
* [Patrón de índice secundario entre particiones](#inter-partition-secondary-index-pattern): Almacene varias copias de cada entidad usando valores `RowKey` diferentes en particiones separadas o tablas separadas. Esto permite realizar búsquedas rápidas y eficaces, así como alternar criterios de ordenación utilizando valores `RowKey` diferentes.  
* [Patrón de entidades de índice](#index-entities-pattern): Mantenga entidades de índice para poder efectuar búsquedas eficaces que devuelvan listas de entidades.  

### <a name="sort-data-in-table-storage"></a>Ordenación de datos en Table Storage

Table Storage devuelve los resultados de la consulta ordenados de forma ascendente, en función de `PartitionKey` y, luego, `RowKey`.

> [!NOTE]
> Los resultados de consulta devueltos por la Table API de Azure en Azure Cosmos DB no se ordenan por clave de fila ni por clave de partición. Para obtener una lista detallada de las diferencias entre características, consulte las [diferencias entre Table API de Azure Cosmos DB y Azure Table Storage](table-api-faq.md#table-api-vs-table-storage).

Las claves de Table Storage son valores de cadena. Para asegurarse de que los valores numéricos se ordenen correctamente, debe convertirlos a una longitud fija y rellenarlos con ceros. Por ejemplo, si el valor de identificador de empleado que utiliza como `RowKey` es un valor entero, debe convertir el identificador de empleado **123** en **00000123**. 

Muchas aplicaciones tienen requisitos para utilizar datos ordenados en distintos órdenes: por ejemplo, ordenar los empleados por su nombre o por su fecha de contratación. Los patrones siguientes de la sección [Patrones de diseño de tabla](#table-design-patterns) tratan cómo alternar criterios de ordenación para sus entidades:  

* [Patrón de índice secundario dentro de la partición](#intra-partition-secondary-index-pattern): Almacene varias copias de cada entidad usando valores `RowKey` diferentes (en la misma partición). Esto permite realizar búsquedas rápidas y eficaces, así como alternar criterios de ordenación utilizando valores `RowKey` diferentes.  
* [Patrón de índice secundario entre particiones](#inter-partition-secondary-index-pattern): Almacene varias copias de cada entidad usando valores `RowKey` diferentes en particiones separadas o tablas separadas. Esto permite realizar búsquedas rápidas y eficaces, así como alternar criterios de ordenación utilizando valores `RowKey` diferentes.
* [Patrón final del registro](#log-tail-pattern): Recupere las entidades *n* agregadas recientemente a una partición utilizando un valor `RowKey` que se ordene en orden de fecha y hora inverso.  

## <a name="design-for-data-modification"></a>Diseño para la modificación de datos
Esta sección se centra en las consideraciones de diseño para optimizar las inserciones, actualizaciones y eliminaciones. En algunos casos, necesitará evaluar el equilibrio entre los diseños que se optimizan para las consultas y los diseños que se optimizan para la modificación de datos. Esta evaluación es similar a lo que se hace en los diseños de las bases de datos relacionales (aunque las técnicas para administrar las ventajas e inconvenientes del diseño son diferentes en una base de datos relacional). En la sección [Patrones de diseño de tabla](#table-design-patterns) se describen algunos modelos de diseño detallados para Table Storage y se destacan algunas de estas ventajas e inconvenientes. En la práctica, encontrará que muchos diseños optimizados para consultar entidades también funcionan adecuadamente para la modificación de entidades.  

### <a name="optimize-the-performance-of-insert-update-and-delete-operations"></a>Optimización del rendimiento de las operaciones de inserción, actualización y eliminación
Para actualizar o eliminar una entidad, debe poder identificarla mediante el uso de los valores `PartitionKey` y `RowKey`. En este sentido, la elección de `PartitionKey` y `RowKey` para modificar entidades debe seguir criterios similares a los que elija para admitir consultas puntuales. Desea identificar las entidades de la forma más eficaz posible. No desea utilizar un recorrido ineficaz de partición o de tabla para buscar una entidad con el fin de detectar los valores `PartitionKey` y `RowKey` que necesita actualizar o eliminar.  

Los patrones siguientes de la sección [Patrones de diseño de tablas](#table-design-patterns) abordan la optimización del rendimiento de las operaciones de inserción, actualización y eliminación:  

* [Patrón de eliminación de gran volumen](#high-volume-delete-pattern): Habilite la eliminación de un gran volumen de entidades mediante el almacenamiento de todas las entidades para su eliminación simultánea en su propia tabla independiente. Elimine las entidades mediante la eliminación de la tabla.  
* [Patrón de serie de datos](#data-series-pattern): Almacene una serie de datos completa en una sola entidad para minimizar el número de solicitudes que realice.  
* [Patrón de entidades amplio](#wide-entities-pattern): Use varias entidades físicas para almacenar entidades lógicas con más de 252 propiedades.  
* [Patrón de entidades de gran tamaño](#large-entities-pattern): Use Blob Storage para almacenar valores de propiedad de gran tamaño.  

### <a name="ensure-consistency-in-your-stored-entities"></a>Garantía de la coherencia en las entidades almacenadas
El otro factor clave que afecta a su elección de claves para optimizar las modificaciones de datos es cómo garantizar la coherencia mediante el uso de transacciones atómicas. Solo puede utilizar un EGT para operar en las entidades almacenadas en la misma partición.  

Los siguientes patrones de la sección [Patrones de diseño de tabla](#table-design-patterns) tratan la administración de coherencia:  

* [Patrón de índice secundario dentro de la partición](#intra-partition-secondary-index-pattern): Almacene varias copias de cada entidad usando valores `RowKey` diferentes (en la misma partición). Esto permite realizar búsquedas rápidas y eficaces, así como alternar criterios de ordenación utilizando valores `RowKey` diferentes.  
* [Patrón de índice secundario entre particiones](#inter-partition-secondary-index-pattern): Almacene varias copias de cada entidad usando valores `RowKey` diferentes en particiones separadas o tablas separadas. Esto permite realizar búsquedas rápidas y eficaces, así como alternar criterios de ordenación utilizando valores `RowKey` diferentes.  
* [Patrón final coherente de transacciones](#eventually-consistent-transactions-pattern): Habilitar el comportamiento final coherente a través de límites de partición o los límites del sistema de almacenamiento mediante el uso de las colas de Azure.
* [Patrón de entidades de índice](#index-entities-pattern): Mantenga entidades de índice para poder efectuar búsquedas eficaces que devuelvan listas de entidades.  
* [Patrón de desnormalización](#denormalization-pattern): Combine datos relacionados entre sí en una sola entidad para recuperar todos los datos que necesita con una consulta puntual única.  
* [Patrón de serie de datos](#data-series-pattern): Almacene una serie de datos completa en una sola entidad para minimizar el número de solicitudes que realice.  

Para más información, consulte [Transacciones de grupos de entidades](#entity-group-transactions) más adelante en este artículo.  

### <a name="ensure-your-design-for-efficient-modifications-facilitates-efficient-queries"></a>Garantía de modificaciones eficientes en el diseño para facilitar la eficiencia de las consultas
En muchos casos, un diseño para los resultados de consultas eficaces en modificaciones eficaces, pero siempre debe evaluar si este es el caso para su escenario concreto. Algunos de los patrones de la sección [Patrones de diseño de tabla](#table-design-patterns) evalúan explícitamente equilibrios entre entidades de consulta y modificación, y deben tener siempre en cuenta el número de cada tipo de operación.  

Los siguientes patrones de la sección [Patrones de diseño de tabla](#table-design-patterns) tratan el equilibrio entre diseños para efectuar consultas eficaces y diseños para la modificación eficaz de datos:  

* [Patrón de clave compuesta](#compound-key-pattern): Use valores `RowKey` compuestos para permitir a los clientes buscar datos relacionados con una consulta puntual única.  
* [Patrón final del registro](#log-tail-pattern): Recupere las entidades *n* agregadas recientemente a una partición utilizando un valor `RowKey` que se ordene en orden de fecha y hora inverso.  

## <a name="encrypt-table-data"></a>Cifrado de datos de tabla
La biblioteca cliente de Azure Storage para .NET admite el cifrado de propiedades de entidades de cadena en operaciones de insertar y reemplazar. Las cadenas cifradas se almacenan en el servicio como propiedades binarias y se convierten de nuevo en cadenas después del descifrado.    

Para las tablas, además de la directiva de cifrado, los usuarios deben especificar las propiedades que se van a cifrar. Especifique un atributo `EncryptProperty` (para las entidades POCO que se derivan de `TableEntity`) o una resolución de cifrado en las opciones de solicitud. Una resolución de cifrado es un delegado que toma una clave de partición, una clave de fila y un nombre de propiedad y devuelve un valor booleano que indica si se debe cifrar dicha propiedad. Durante el cifrado, la biblioteca de cliente usa esta información para decidir si se debe cifrar una propiedad mientras se escribe en la conexión. El delegado también proporciona la posibilidad de lógica con respecto a la forma de cifrar las propiedades. (Por ejemplo, si el valor es X, hay que cifrar la propiedad A; en caso contrario, hay que cifrar las propiedades A y B). No es necesario proporcionar esta información para leer o consultar entidades.

La combinación no se admite actualmente. Puesto que un subconjunto de propiedades puede haberse cifrado previamente con una clave distinta, si simplemente se combinan las nuevas propiedades y se actualizan los metadatos, se producirá una pérdida de datos. Para realizar una combinación es necesario realizar llamadas de servicio adicionales para leer la entidad existente desde el servicio. También puede usar una nueva clave por propiedad. Ninguno de estos procedimientos es adecuado por motivos de rendimiento.     

Para información acerca del cifrado de datos de tablas, consulte [Cifrado del lado cliente y Azure Key Vault para Microsoft Azure Storage](../storage/common/storage-client-side-encryption.md).  

## <a name="model-relationships"></a>Relaciones de modelo
La creación de modelos de dominio es un paso clave en el diseño de sistemas complejos. Normalmente, usará el proceso de modelado para identificar las entidades y las relaciones entre ellas como manera de entender el dominio de negocio e informar del diseño del sistema. Esta sección se centra en cómo puede convertir algunos de los tipos de relación comunes encontrados en los modelos de dominio en diseños para Table Storage. El proceso de asignación de un modelo de datos lógico a uno físico basado en NoSQL es muy diferente del que se usa cuando se diseña una base de datos relacional. El diseño de bases de datos relacionales presupone normalmente un proceso de normalización de datos optimizado para minimizar la redundancia. Dicho diseño también presupone una funcionalidad de consulta declarativa que abstrae la implementación de cómo funciona la base de datos.  

### <a name="one-to-many-relationships"></a>Relaciones uno a varios
Las relaciones uno a varios entre los objetos de dominio de negocio se producen con mucha frecuencia: por ejemplo, cuando un departamento tiene muchos empleados. Hay varias formas de implementar relaciones uno a varios en Table Storage, cada una con las correspondientes ventajas y desventajas para el escenario en concreto.  

Piense en el ejemplo de una corporación multinacional de gran tamaño con decenas de miles de departamentos y entidades de empleados. Cada departamento tiene muchos empleados y cada empleado está asociado a un departamento específico. Un enfoque consiste en almacenar el departamento y las entidades de empleado por separado, de este modo:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE01.png" alt-text="Gráfico que muestra una entidad de departamento y una entidad de empleado":::

En este ejemplo se muestra una relación de uno a varios implícita entre los tipos basados en el valor `PartitionKey`. Cada departamento puede tener muchos empleados.  

En este ejemplo también se muestra una entidad de departamento y sus entidades relacionadas de empleado relacionadas en la misma partición. Puede optar por usar distintas particiones, tablas o incluso cuentas de almacenamiento para los diferentes tipos de entidad.  

Un enfoque alternativo es desnormalizar los datos y almacenar solo las entidades de empleado con datos sin normalizar de departamentos tal como se muestra en el ejemplo siguiente. En este escenario concreto, este enfoque sin normalizar puede que no sea el mejor si tiene un requisito para poder cambiar los detalles de un administrador de departamento. Para ello, deberá actualizar a todos los empleados del departamento.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE02.png" alt-text="Gráfico de una entidad de empleado":::

Para obtener más información, consulte más adelante en esta guía el [Patrón de desnormalización](#denormalization-pattern) .  

En la tabla siguiente se resumen las ventajas y desventajas de cada uno de los métodos para almacenar entidades de departamento y empleado que tienen una relación uno a varios. También debe tener en cuenta la frecuencia con la que espera realizar varias operaciones. Podría ser aceptable tener un diseño que incluya una operación costosa si esa operación solo se produce con poca frecuencia.  

<table>
<tr>
<th>Enfoque</th>
<th>Ventajas</th>
<th>Desventajas</th>
</tr>
<tr>
<td>Tipos de entidad independientes, la misma partición, la misma tabla</td>
<td>
<ul>
<li>Puede actualizar una entidad de departamento con una sola operación.</li>
<li>Puede usar un EGT para mantener la coherencia si tiene un requisito para modificar una entidad department siempre que se actualice, inserte o elimine una entidad de empleado. Por ejemplo, si mantiene un recuento de empleados de departamento para cada departamento.</li>
</ul>
</td>
<td>
<ul>
<li>Puede que necesite recuperar una entidad de empleado y de departamento para algunas actividades de cliente.</li>
<li>Las operaciones de almacenamiento se producen en la misma partición. Con volúmenes de transacciones elevados, esto puede producir un punto de conflicto.</li>
<li>No puede mover a un empleado a un nuevo departamento mediante una EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Tipos de entidad independientes, particiones o tablas distintas o cuentas de almacenamiento</td>
<td>
<ul>
<li>Puede actualizar una entidad de departamento o de empleado con una sola operación.</li>
<li>Con volúmenes de transacciones altos, esto puede ayudar a distribuir la carga entre más particiones.</li>
</ul>
</td>
<td>
<ul>
<li>Puede que necesite recuperar una entidad de empleado y de departamento para algunas actividades de cliente.</li>
<li>No se pueden usar EGT para mantener la coherencia cuando se actualiza, inserta y elimina un empleado y se actualiza un departamento. Por ejemplo, actualizando un recuento de empleados en una entidad de departamento.</li>
<li>No puede mover a un empleado a un nuevo departamento mediante una EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Desnormalizar en un único tipo de entidad</td>
<td>
<ul>
<li>Puede recuperar toda la información que necesita con una única solicitud.</li>
</ul>
</td>
<td>
<ul>
<li>Puede ser costoso de mantener la coherencia si necesita actualizar la información de departamento (esto requerirá actualizar a todos los empleados de un departamento).</li>
</ul>
</td>
</tr>
</table>

La elección entre estas opciones y la determinación de cuáles de las ventajas y desventajas son más importantes dependen de los escenarios de aplicación concretos. Por ejemplo, ¿con qué frecuencia se modifican las entidades de departamento? ¿Todas las consultas de los empleados requieren la información adicional del departamento? ¿Cuál es la proximidad de los límites de escalabilidad de sus particiones o de su cuenta de almacenamiento?  

### <a name="one-to-one-relationships"></a>Relaciones uno a uno
Es posible que los modelos de dominio incluyan relaciones uno a uno entre las entidades. Si necesita implementar una relación uno a uno en Table Storage, también debe elegir cómo vincular las dos entidades relacionadas cuando se necesita para recuperar ambas. Este vínculo puede ser implícito, en función de una convención en los valores de clave, o explícito, almacenando un vínculo en el formulario de los valores `PartitionKey` y `RowKey` de cada entidad con su entidad relacionada. Para obtener una explicación sobre si debe almacenar las entidades relacionadas en la misma partición, consulte la sección [Relaciones uno a varios](#one-to-many-relationships).  

También hay que considerar algunos aspectos sobre la implementación que podrían llevarle a implementar relaciones uno a uno en Table Storage:  

* Administración de entidades de gran tamaño (para obtener más información, consulte [Patrón de entidades de gran tamaño](#large-entities-pattern)).  
* Implementación de controles de acceso (para más información, consulte [Control de acceso con firmas de acceso compartido](#control-access-with-shared-access-signatures)).  

### <a name="join-in-the-client"></a>Únase al cliente
Aunque hay formas de modelar las relaciones en Table Storage, no olvide que las dos razones principales para utilizar Table Storage son la escalabilidad y el rendimiento. Si observa que está modelando muchas relaciones que ponen en peligro el rendimiento y la escalabilidad de su solución, debe preguntarse si es necesario crear todas las relaciones de datos en el diseño de tabla. Es posible que pueda simplificar el diseño y mejorar la escalabilidad y el rendimiento de la solución si permite que la aplicación cliente realice las uniones necesarias.  

Por ejemplo, si tiene tablas pequeñas que contienen datos que no cambian muy a menudo, puede recuperar estos datos una vez y almacenarlos en la caché en el cliente. Esto puede evitar repetidas idas y vueltas para recuperar los mismos datos. En los ejemplos que hemos examinado en esta guía, es probable que el conjunto de departamentos de una organización pequeña sea pequeño y cambie con poca frecuencia. Esto lo convierte en un buen candidato para los datos que una aplicación cliente puede descargar una vez y almacenar en caché como datos de búsqueda.  

### <a name="inheritance-relationships"></a>Relaciones de herencia
Si la aplicación cliente usa un conjunto de clases que forman parte de una relación de herencia para representar entidades empresariales, puede conservar fácilmente las entidades en Table Storage. Por ejemplo, el siguiente conjunto de clases puede estar definido en la aplicación cliente, donde `Person` es una clase abstracta.

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE03.png" alt-text="Diagrama de relaciones de herencia":::

Puede conservar las instancias de las dos clases concretas en Table Storage mediante una sola tabla de `Person`. Use entidades que tengan un aspecto similar al siguiente:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE04.png" alt-text="Gráfico que muestra una entidad de cliente y una entidad de empleado":::

Para más información acerca de cómo trabajar con varios tipos de entidad en la misma tabla en el código de cliente, consulte [Uso de tipos de entidad heterogéneos](#work-with-heterogeneous-entity-types) en esta misma guía. Esto proporciona ejemplos de cómo reconocer el tipo de entidad en el código de cliente.  

## <a name="table-design-patterns"></a>Patrones de diseño de tabla
En las secciones anteriores, ha visto cómo optimizar su diseño de tabla para recuperar datos de entidad mediante consultas y para insertar, actualizar y eliminar datos de la entidad. En esta sección se describen algunos modelos adecuados para usarse con soluciones de Table Storage. Además, verá cómo puede prácticamente abordar algunos de los problemas y las ventajas e inconvenientes generados anteriormente en esta guía. En el diagrama siguiente se resumen las relaciones entre los distintos patrones:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE05.png" alt-text="Diagrama de patrones de diseño de tablas":::

La asignación de patrones resalta algunas relaciones entre patrones (azules) y antipatrones (naranja) que se documentan en esta guía. Por supuesto, existen muchos otros patrones que merece la pena tener en cuenta. Por ejemplo, uno de los escenarios clave de Table Storage es almacenar el [patrón de vistas materializadas](https://msdn.microsoft.com/library/azure/dn589782.aspx) desde el patrón de [segregación de responsabilidades de comandos y consultas](https://msdn.microsoft.com/library/azure/jj554200.aspx).  

### <a name="intra-partition-secondary-index-pattern"></a>Patrón de índice secundario dentro de la partición
Almacene varias copias de cada entidad usando valores `RowKey` diferentes (en la misma partición). Esto permite realizar búsquedas rápidas y eficaces, así como alternar criterios de ordenación utilizando valores `RowKey` diferentes. La coherencia de las actualizaciones entre copias se puede mantener mediante las EGT.  

#### <a name="context-and-problem"></a>Contexto y problema
Table Storage indexa automáticamente las entidades mediante los valores `PartitionKey` y `RowKey`. Esto permite que una aplicación cliente recupere una entidad eficazmente con estos valores. Por ejemplo, si se usa la estructura de tabla siguiente, una aplicación cliente puede usar una consulta puntual para recuperar una entidad de empleado individual mediante el nombre del departamento y el identificador del empleado (los valores `PartitionKey` y `RowKey`). Un cliente también puede recuperar las entidades ordenadas por identificador de empleado dentro de cada departamento.

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE06.png" alt-text="Gráfico de una entidad de empleado":::

Si desea buscar una entidad de empleado basada en el valor de otra propiedad, como la dirección de correo electrónico, debe usar un examen de la partición menos eficiente para encontrar una coincidencia. Esto se debe a que Table Storage no proporciona índices secundarios. Además, no hay ninguna opción para solicitar una lista de empleados ordenados en un orden diferente a `RowKey`.  

#### <a name="solution"></a>Solución
Para solucionar la falta de índices secundarios, puede almacenar varias copias de cada entidad con cada copia mediante un valor `RowKey` diferente. Si almacena una entidad con las estructuras siguientes, puede recuperar eficazmente las entidades de empleado en función de un identificador de empleado o una dirección de correo electrónico. Los valores de prefijo de `RowKey`, `empid_` y `email_` permiten consultar un solo empleado, o un intervalo de empleados, mediante un intervalo de direcciones de correo electrónico o identificadores de empleado.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE07.png" alt-text="Gráfico que muestra una entidad de empleado con diferentes valores de RowKey":::

Los dos criterios de filtro siguientes (uno de búsqueda por identificador de empleado y uno de búsqueda por dirección de correo electrónico) especifican consultas puntuales:  

* $filter=(PartitionKey eq 'Sales') y (RowKey eq 'empid_000223')  
* $filter=(PartitionKey eq 'Sales') y (RowKey eq 'email_jonesj@contoso.com')  

Si consulta un intervalo de entidades de empleado, puede especificar un intervalo ordenado por identificador de empleado o un intervalo ordenado por dirección de correo electrónico. Consulte entidades con el prefijo adecuado en `RowKey`.  

* Para buscar todos los empleados del departamento de ventas con un identificador de empleado en el intervalo de 000100 a 000199, use: $filter=(PartitionKey eq 'Sales') y (RowKey ge 'empid_000100') y (RowKey le 'empid_000199').  
* Para buscar todos los empleados del departamento de ventas con una dirección de correo electrónico que empiece por la letra "a", use: $filter=(PartitionKey eq 'Sales') y (RowKey ge 'email_a') y (RowKey lt 'email_b').  
  
La sintaxis de filtro utilizada en los ejemplos anteriores procede de la API REST de Table Storage. Para más información, consulte [Consulta de entidades](https://msdn.microsoft.com/library/azure/dd179421.aspx).  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* El almacenamiento en tablas es relativamente barato, por lo que la sobrecarga de costos de almacenamiento de datos duplicados no debe ser una preocupación importante. Sin embargo, debe evaluar siempre el costo del diseño según los requisitos de almacenamiento previstos y solo agregar entidades duplicadas para admitir las consultas que ejecutará la aplicación cliente.  
* Dado que las entidades de índice secundario se almacenan en la misma partición que las entidades originales, asegúrese de que no superen los objetivos de escalabilidad para una partición individual.  
* Puede mantener la coherencia de las entidades duplicadas utilizando EGT para actualizar las dos copias de la entidad de forma atómica. Esto implica que debe almacenar todas las copias de una entidad en la misma partición. Para más información, consulte [Transacciones de grupos de entidades](#entity-group-transactions).  
* El valor que se usa para `RowKey` debe ser único para cada entidad. Considere la posibilidad de usar valores de clave compuestos.  
* Al rellenar los valores numéricos de `RowKey` (por ejemplo, el identificador de empleado 000223) es posible corregir los criterios de ordenación y filtrado en función de los límites inferior y superior.  
* No es necesario duplicar todas las propiedades de su entidad. Por ejemplo, si las consultas que realizan búsquedas en las entidades mediante la dirección de correo electrónico de `RowKey` nunca necesitan la edad del empleado, dichas entidades podrían tener la siguiente estructura:

  :::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE08.png" alt-text="Gráfico de una entidad de empleado":::

* Normalmente, es mejor almacenar los datos duplicados y asegurarse de que puede recuperar todos los datos que necesita con una sola consulta que usar una consulta para buscar una entidad y otra para buscar los datos necesarios.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Use este patrón en los siguientes supuestos:

- La aplicación cliente debe recuperar entidades mediante una variedad de claves diferentes.
- El cliente debe recuperar entidades en distintos criterios de ordenación.
- Puede identificar cada entidad mediante una variedad de valores únicos.

Sin embargo, asegúrese de no superar los límites de escalabilidad de partición al realizar búsquedas de entidad utilizando los diferentes valores `RowKey`.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Patrón de índice secundario entre particiones](#inter-partition-secondary-index-pattern)
* [Patrón de clave compuesta](#compound-key-pattern)
* [Transacciones de grupos de entidades](#entity-group-transactions)
* [Work with heterogeneous entity types](#work-with-heterogeneous-entity-types) (Uso de tipos de entidad heterogéneos)

### <a name="inter-partition-secondary-index-pattern"></a>Patrón de índice secundario entre particiones
Almacene varias copias de cada entidad usando valores `RowKey` diferentes en particiones separadas o tablas separadas. Esto permite realizar búsquedas rápidas y eficaces, así como alternar criterios de ordenación utilizando valores `RowKey` diferentes.  

#### <a name="context-and-problem"></a>Contexto y problema
Table Storage indexa automáticamente las entidades mediante los valores `PartitionKey` y `RowKey`. Esto permite que una aplicación cliente recupere una entidad eficazmente con estos valores. Por ejemplo, si se usa la estructura de tabla siguiente, una aplicación cliente puede usar una consulta puntual para recuperar una entidad de empleado individual mediante el nombre del departamento y el identificador del empleado (los valores `PartitionKey` y `RowKey`). Un cliente también puede recuperar las entidades ordenadas por identificador de empleado dentro de cada departamento.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE09.png" alt-text="Gráfico de una entidad de empleado":::[9]

Si desea ser capaz de encontrar una entidad de empleado basada en el valor de otra propiedad, como la dirección de correo electrónico, debe usar un examen de la partición menos eficiente para encontrar a coincidencia. Esto se debe a que Table Storage no proporciona índices secundarios. Además, no hay ninguna opción para solicitar una lista de empleados ordenados en un orden diferente a `RowKey`.  

Prevé un gran volumen de transacciones en estas entidades y desea minimizar el riesgo de que Table Storage limite la velocidad del cliente.  

#### <a name="solution"></a>Solución
Para solucionar la falta de índices secundarios, puede almacenar varias copias de cada entidad con cada copia mediante valores `PartitionKey` y `RowKey`. Si almacena una entidad con las estructuras siguientes, puede recuperar eficazmente las entidades de empleado en función de un identificador de empleado o una dirección de correo electrónico. Los valores de prefijo de `PartitionKey`, `empid_` y `email_` le permiten identificar qué índice desea utilizar para una consulta.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE10.png" alt-text="Gráfico que muestra una entidad de empleado con índice principal y una entidad de empleado con índice secundario":::

Los dos criterios de filtro siguientes (uno de búsqueda por identificador de empleado y uno de búsqueda por dirección de correo electrónico) especifican consultas puntuales:  

* $filter=(PartitionKey eq 'empid_Sales') y (RowKey eq '000223')
* $filter=(PartitionKey eq 'email_Sales') y (RowKey eq 'jonesj@contoso.com')  

Si consulta un intervalo de entidades de empleado, puede especificar un intervalo ordenado por identificador de empleado o un intervalo ordenado por dirección de correo electrónico. Consulte entidades con el prefijo adecuado en `RowKey`.  

* Para buscar todos los empleados del departamento de ventas con un identificador de empleado en el intervalo de **000100** a **000199**, clasificados en orden de identificador de empleado, use: $filter=(PartitionKey eq 'empid_Sales') y (RowKey ge '000100') y (RowKey le '000199').  
* Para buscar todos los empleados del departamento de ventas con una dirección de correo electrónico que empiece por "a", ordenados en el orden de dirección de correo electrónico, use: $filter=(PartitionKey eq 'email_Sales') y (RowKey ge 'a') y (RowKey lt 'b').  

Observe que la sintaxis de filtro utilizada en los ejemplos anteriores procede de la API REST de Table Storage. Para más información, consulte [Consulta de entidades](https://msdn.microsoft.com/library/azure/dd179421.aspx).  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* Puede mantener las entidades duplicadas coherentes entre sí en última instancia con el [patrón de transacciones coherentes en última instancia](#eventually-consistent-transactions-pattern) para conservar las entidades de índice principal y secundaria.  
* Table Storage es relativamente barato, por lo que la sobrecarga de costos de almacenamiento de datos duplicados no debe ser una preocupación importante. Sin embargo, evalúe siempre el costo del diseño según los requisitos de almacenamiento previstos y solo agregue entidades duplicadas para admitir las consultas que ejecutará la aplicación cliente.  
* El valor que se usa para `RowKey` debe ser único para cada entidad. Considere la posibilidad de usar valores de clave compuestos.  
* Al rellenar los valores numéricos de `RowKey` (por ejemplo, el identificador de empleado 000223) es posible corregir los criterios de ordenación y filtrado en función de los límites inferior y superior.  
* No es necesario duplicar todas las propiedades de su entidad. Por ejemplo, si las consultas que realizan búsquedas en las entidades mediante la dirección de correo electrónico de `RowKey` nunca necesitan la edad del empleado, dichas entidades podrían tener la siguiente estructura:
  
  :::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE11.png" alt-text="Gráfico que muestra una entidad de empleado con índice secundario":::

* Normalmente es mejor almacenar los datos duplicados y asegurarse de que puede recuperar todos los datos que necesita con una sola consulta que usar una consulta para buscar una entidad mediante el índice secundario y otra para buscar los datos necesarios en el índice principal.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Use este patrón en los siguientes supuestos:

- La aplicación cliente debe recuperar entidades mediante una variedad de claves diferentes.
- El cliente debe recuperar entidades en distintos criterios de ordenación.
- Puede identificar cada entidad mediante una variedad de valores únicos.

Utilice este patrón cuando desee no exceder los límites de escalabilidad de la partición al realizar búsquedas de entidades que usen los diferentes valores `RowKey`.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Patrón final coherente de transacciones](#eventually-consistent-transactions-pattern)  
* [Patrón de índice secundario dentro de la partición](#intra-partition-secondary-index-pattern)  
* [Patrón de clave compuesta](#compound-key-pattern)  
* [Transacciones de grupos de entidades](#entity-group-transactions)  
* [Work with heterogeneous entity types](#work-with-heterogeneous-entity-types) (Uso de tipos de entidad heterogéneos)  

### <a name="eventually-consistent-transactions-pattern"></a>Patrón final coherente de transacciones
Habilitar el comportamiento final coherente a través de límites de partición o los límites del sistema de almacenamiento mediante el uso de las colas de Azure.  

#### <a name="context-and-problem"></a>Contexto y problema
Los EGT permiten transacciones atómicas a través de varias entidades que comparten la misma clave de partición. Por motivos de escalabilidad y rendimiento, puede decidir almacenar entidades con requisitos de coherencia en particiones independientes o en un sistema de almacenamiento independiente. En este escenario, no puede utilizar las EGT para mantener la coherencia. Por ejemplo, podría tener un requisito de mantener la coherencia eventual entre:  

* Entidades almacenadas en dos particiones diferentes de la misma tabla, en tablas diferentes y en diferentes cuentas de almacenamiento.  
* Una entidad almacenada en Table Storage y un blob almacenado en Blob Storage.  
* Una entidad almacenada en Table Storage y un archivo en un sistema de archivos.  
* Una entidad almacenada en Table Storage, pero indexada mediante Azure Cognitive Search.  

#### <a name="solution"></a>Solución
Mediante el uso de las colas de Azure, puede implementar una solución que ofrece coherencia final entre dos o más particiones o sistemas de almacenamiento.

Para ilustrar este enfoque, suponga que tiene un requisito para poder archivar entidades de empleado antiguas. Las entidades de empleado antiguas rara vez se consultan y deben excluirse de las actividades relacionadas con los empleados actuales. Para implementar este requisito, almacene los empleados activos en la tabla **Current** y los empleados antiguos en la tabla **Archive**. Para archivar un empleado, es preciso eliminar la entidad de la tabla **Current** y agregarla a la tabla **Archive**.

Sin embargo, no se puede usar una EGT para realizar estas dos operaciones. Para evitar el riesgo de que un error provoque la aparición de una entidad en las dos tablas o en ninguna, la operación de almacenamiento debe ser coherente con el tiempo. En el diagrama de secuencia siguiente se describen los pasos de esta operación.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE12.png" alt-text="Diagrama de la solución para mantener la coherencia final":::

Un cliente inicia la operación de almacenamiento mediante la colocación de un mensaje en una cola de Azure, en este ejemplo para archivar el empleado #456. Un rol de trabajador sondea la cola de mensajes nuevos; si encuentra alguno, lee el mensaje y deja una copia oculta en la cola. A continuación, el rol de trabajo busca una copia de la entidad en la tabla **Current**, inserta una copia en la tabla **Archive** y, seguidamente, elimina la original de la tabla **Current**. Por último, si no ha habido errores en los pasos anteriores, el rol de trabajador elimina el mensaje oculto de la cola.  

En este ejemplo, el paso 4 del diagrama inserta el empleado en la tabla **Archive**. Puede añadir el empleado a un blob en Blob Storage o un archivo en un sistema de archivos.  

#### <a name="recover-from-failures"></a>Recuperación de errores
Es importante que las operaciones de los pasos 4-5 sean *idempotentes*, por si el rol de trabajo necesita reiniciar la operación de archivado. Si va a utilizar Table Storage para el paso 4, debe utilizar una operación de "insertar o reemplazar"; en el paso 5, debe usar una operación de "eliminar si existe" en la biblioteca cliente que esté usando. Si está utilizando otro sistema de almacenamiento, debe utilizar una operación idempotente adecuada.  

Si el rol de trabajo no completa el paso 6 del diagrama, después de un tiempo de expiración el mensaje volverá a aparecer en la cola listo para que el rol de trabajo intente volver a procesarlo. El rol de trabajo puede comprobar cuántas veces se ha leído un mensaje de la cola y, si es necesario, marcarlo como mensaje "dudoso" para investigarlo mediante el envío a una cola independiente. Para obtener más información acerca de cómo leer mensajes de la cola y comprobar el número de eliminaciones de cola, consulte [Obtener mensajes](https://msdn.microsoft.com/library/azure/dd179474.aspx).  

Algunos errores de Table Storage y Queue Storage son errores transitorios, y la aplicación cliente debe incluir una lógica de reintento adecuada para controlarlos.  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* Esta solución no permite el aislamiento de las transacciones. Por ejemplo, un cliente pudo leer las tablas **Current** y **Archive** cuando el rol de trabajo estaba entre los pasos 4-5 y 5 del diagrama, y tener una vista incoherente de los datos. Los datos serán coherentes con el tiempo.  
* Debe asegurarse de que los pasos 4 y 5 sean idempotentes para garantizar la coherencia final.  
* Puede escalar la solución mediante el uso de varias colas e instancias de rol de trabajador.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Utilice este patrón cuando desee garantizar la coherencia eventual entre las entidades que existen en diferentes particiones o tablas. Puede extender este patrón para garantizar la coherencia final de las operaciones en Table Storage y Blob Storage, y otros orígenes de datos que no sean de Azure Storage, tales como bases de datos o el sistema de archivos.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Transacciones de grupos de entidades](#entity-group-transactions)  
* [Combinar o reemplazar](#merge-or-replace)  

> [!NOTE]
> Si el aislamiento de transacciones es importante para su solución, considere la posibilidad de volver a diseñar las tablas para poder utilizar las EGT.  
> 
> 

### <a name="index-entities-pattern"></a>Patrón de entidades de índice
Mantenga entidades de índice para poder efectuar búsquedas eficaces que devuelvan listas de entidades.  

#### <a name="context-and-problem"></a>Contexto y problema
Table Storage indexa automáticamente las entidades mediante los valores `PartitionKey` y `RowKey`. Esto permite que una aplicación cliente recupere una entidad eficazmente mediante una consulta puntual. Por ejemplo, si se usa la estructura de tabla siguiente, una aplicación cliente puede recuperar eficazmente una entidad de empleado individual mediante el nombre del departamento y el identificador del empleado (los valores `PartitionKey` y `RowKey`).  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE13.png" alt-text="Gráfico de una entidad de empleado":::

Si también desea poder recuperar una lista de las entidades employee en función del valor de otra propiedad no exclusiva, por ejemplo, su apellido, debe utilizar un examen de partición menos eficaz. Este examen busca coincidencias en lugar de utilizar un índice para buscarlas directamente. Esto se debe a que Table Storage no proporciona índices secundarios.  

#### <a name="solution"></a>Solución
Para habilitar la búsqueda por apellido con la estructura de entidad anterior, debe mantener listas de identificadores de empleado. Si quiere recuperar las entidades "employee" con un apellido determinado, como Jones, debe encontrar primero la lista de identificadores de empleado para los empleados con Jones como apellido y, después, recuperar las entidades "employee". Hay tres opciones principales para almacenar las listas de identificadores de empleado:  

* Uso de Blob Storage.  
* Cree entidades de índice en la misma partición que las entidades employee.  
* Cree entidades de índice en una tabla o una partición independiente.  

Opción 1: Uso de Blob Storage  

Cree un blob para cada apellido único y, en cada almacén de blobs, una lista de valores `PartitionKey` (departamento) y `RowKey` (identificador de empleado) para los empleados que tienen ese apellido. Al agregar o eliminar a un empleado, asegúrese de que el contenido del blob relevante es coherente con las entidades de empleado.  

Opción 2: Creación de entidades de índice en la misma partición  

Utilice las entidades de índice que almacenan los datos siguientes:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE14.png" alt-text="Gráfico que muestra una entidad de empleado con una cadena que contiene una lista de identificadores de empleado con el mismo apellido":::

La propiedad `EmployeeIDs` contiene una lista de identificadores de empleado para los empleados cuyo apellido está almacenado en `RowKey`.  

Los siguientes pasos describen el proceso que debe seguir al agregar un nuevo empleado. En este ejemplo, agregamos un empleado con el identificador 000152 y el apellido Jones en el departamento de ventas:  

1. Recupere la entidad de índice con el valor `PartitionKey` "Sales" y el valor `RowKey` "Jones". Guarde el valor ETag de esta entidad para usar en el paso 2.  
2. Cree una transacción de grupo de entidades (es decir, una operación por lotes) que inserte la nueva entidad de empleado (valor `PartitionKey` "Sales" y valor `RowKey` "000152") y actualice la entidad de índice (valor `PartitionKey` "Sales" y valor `RowKey` "Jones"). Para ello, la EGT agrega el nuevo identificador de empleado a la lista en el campo EmployeeIDs. Para más información sobre las EGT, consulte [Transacciones de grupos de entidades](#entity-group-transactions).  
3. Si la EGT experimenta un error de simultaneidad optimista (esto es, alguien ha modificado la entidad de índice), necesitará comenzar de nuevo en el paso 1.  

Puede usar un enfoque similar a la eliminación de un empleado si utiliza la segunda opción. Cambiar el apellido de un empleado es ligeramente más complejo porque necesitará ejecutar una EGT que actualice tres entidades: la entidad de empleado, la entidad de índice para el apellido antiguo y la entidad de índice para el apellido nuevo. Debe recuperar cada entidad antes de realizar cambios para recuperar los valores de ETag que puede utilizar para realizar las actualizaciones mediante la simultaneidad optimista.  

Los siguientes pasos describen el proceso que debe llevar a cabo cuando se necesita buscar todos los empleados con un apellido determinado en un departamento. En este ejemplo se buscan todos los empleados con el apellido Jones en el departamento de ventas:  

1. Recupere la entidad de índice con el valor `PartitionKey` "Sales" y el valor `RowKey` "Jones".  
2. Analice la lista de identificadores de empleado en el campo `EmployeeIDs`.  
3. Si necesita información adicional sobre cada uno de los empleados (por ejemplo, sus direcciones de correo electrónico), recupere cada una de las entidades de empleado mediante los valores `PartitionKey` "Sales" y `RowKey` de la lista de empleados que obtuvo en el paso 2.  

Opción 3: Creación de entidades de índice en una tabla o partición independiente  

Para esta opción, utilice las entidades de índice que almacenan los datos siguientes:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE15.png" alt-text="Gráfico que muestra una entidad de empleado con una cadena que contiene una lista de identificadores de empleado con el mismo apellido":::

La propiedad `EmployeeIDs` contiene una lista de identificadores de empleado para los empleados cuyo apellido está almacenado en `RowKey` y `PartitionKey`.  

No puede utilizar las EGT para mantener la coherencia porque las entidades del índice están en una partición distinta que las entidades de empleado. Asegúrese de que las entidades de índice son coherentes finalmente con las entidades "employee".  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* Esta solución requiere al menos dos consultas para recuperar las entidades coincidentes: una para consultar las entidades de índice con el fin de obtener la lista de valores `RowKey` y, luego, las consultas para recuperar cada entidad de la lista.  
* Dado que una entidad individual tiene un tamaño máximo de 1 MB, la opción 2 y la opción 3 de la solución dan por hecho que la lista de identificadores de empleado de cualquier apellido determinado nunca es mayor que 1 MB. Si es probable que la lista de identificadores de empleado tenga un tamaño superior a 1 MB, use la opción 1 y almacene los datos del índice en Blob Storage.  
* Si utiliza la opción 2 (el uso de EGT para controlar la adición y eliminación de empleados, y el cambio de los apellidos de un empleado), debe evaluar si el volumen de transacciones se aproximará a los límites de escalabilidad de una partición determinada. Si este es el caso, debe considerar una solución coherente en última instancia (opción 1 o bien opción 3). Estas usan colas para controlar las solicitudes de actualización y permiten almacenar las entidades de índice en una partición independiente de las entidades de empleado.  
* La opción 2 de esta solución presupone que desea buscar por apellido dentro de un departamento. Por ejemplo, desea recuperar una lista de empleados con el apellido Jones en el departamento de ventas. Si desea buscar todos los empleados con apellido Jones en toda la organización, utilice opción 1 o la opción 3.
* Puede implementar una solución basada en cola que ofrezca coherencia final. Para obtener más información, consulte [Patrón final coherente de transacciones](#eventually-consistent-transactions-pattern).  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Use este patrón cuando quiera buscar un conjunto de entidades que compartan un valor de propiedad común, como todos los empleados con el apellido Jones.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Patrón de clave compuesta](#compound-key-pattern)  
* [Patrón final coherente de transacciones](#eventually-consistent-transactions-pattern)  
* [Transacciones de grupos de entidades](#entity-group-transactions)  
* [Work with heterogeneous entity types](#work-with-heterogeneous-entity-types) (Uso de tipos de entidad heterogéneos)  

### <a name="denormalization-pattern"></a>Patrón de desnormalización
Combine datos relacionados entre sí en una sola entidad para recuperar todos los datos que necesita con una consulta de punto único.  

#### <a name="context-and-problem"></a>Contexto y problema
En una base de datos relacional, normalmente normaliza datos para eliminar datos duplicados que se generan en las consultas que recuperan datos de varias tablas. Si normaliza los datos de tablas de Azure, debe realizar varias acciones de ida y vuelta desde el cliente al servidor para recuperar los datos relacionados. Por ejemplo, con la siguiente estructura de tabla, necesita dos recorridos de ida y vuelta para recuperar los detalles de un departamento. Un recorrido captura la entidad de departamento que incluye el identificador del administrador y el segundo paso captura los detalles del administrador en una entidad de empleado.  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE16.png" alt-text="Gráfico de una entidad de departamento y una entidad de empleado":::

#### <a name="solution"></a>Solución
En lugar de almacenar los datos en dos entidades independientes, desnormalice los datos y conserve una copia de los detalles del administrador en la entidad department. Por ejemplo:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE17.png" alt-text="Gráfico de una entidad de departamento combinada y sin normalizar":::

Ahora con las entidades de departamento almacenadas con estas propiedades, puede recuperar todos los detalles que necesita acerca de un departamento mediante una consulta puntual.  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* Hay algunos costes de sobrecarga asociados al almacenamiento de datos dos veces. El mejor rendimiento resultante de tener menos solicitudes en Table Storage es normalmente superior al aumento marginal de los costos de almacenamiento. Además, este costo se compensa parcialmente por una reducción en el número de transacciones que necesita para capturar los detalles de un departamento.  
* Debe mantener la coherencia de las dos entidades que almacenan información acerca de los administradores. Puede controlar el problema de coherencia mediante el uso de EGT para actualizar varias entidades en una única transacción atómica. En este caso, la entidad de departamento y la entidad de empleado para el administrador del departamento se almacenan en la misma partición.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Utilice este patrón cuando necesite buscar información relacionada con frecuencia. Este patrón reduce el número de consultas que el cliente debe realizar para recuperar los datos que necesita.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Patrón de clave compuesta](#compound-key-pattern)  
* [Transacciones de grupos de entidades](#entity-group-transactions)  
* [Work with heterogeneous entity types](#work-with-heterogeneous-entity-types) (Uso de tipos de entidad heterogéneos)

### <a name="compound-key-pattern"></a>Patrón de clave compuesta
Use valores `RowKey` compuestos para permitir a los clientes buscar datos relacionados con una consulta puntual única.  

#### <a name="context-and-problem"></a>Contexto y problema
En una base de datos relacional, resulta natural usar combinaciones en las consultas para devolver datos relacionados al cliente en una sola consulta. Por ejemplo, podría usar el identificador de empleado para buscar una lista de entidades relacionadas que contengan datos de rendimiento y revisión de ese empleado.  

Suponga que está almacenando entidades de empleado en Table Storage utilizando la siguiente estructura:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE18.png" alt-text="Gráfico de una entidad de empleado":::

También necesita almacenar datos históricos relacionados con las revisiones y el rendimiento de cada año que el empleado ha trabajado para su organización y necesitará tener acceso a esta información por año. Una opción consiste en crear otra tabla que almacene las entidades con la estructura siguiente:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE19.png" alt-text="Gráfico de una entidad de revisión de empleado":::

Con este enfoque, podría decidir duplicar parte de la información (por ejemplo, nombre y apellidos) en la nueva entidad, lo que le permite recuperar los datos con una única solicitud. Sin embargo, no puede mantener la homogeneidad porque no puede utilizar una EGT para actualizar las dos entidades de forma atómica.  

#### <a name="solution"></a>Solución
Almacene un nuevo tipo de entidad en la tabla original mediante entidades con la estructura siguiente:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE20.png" alt-text="Gráfico de una entidad de empleado con clave compuesta":::

Observe cómo el valor `RowKey` es ahora una clave compuesta, formada por el identificador de empleado y el año de los datos de revisión. Esto le permite recuperar el rendimiento del empleado y revisar los datos con una única solicitud para una sola entidad.  

El ejemplo siguiente describe cómo se pueden recuperar todos los datos de revisión para un empleado concreto (como employee 000123 en el departamento de ventas):  

$filter=(PartitionKey eq 'Ventas') y (RowKey ge 'empid_000123') y (RowKey lt 'empid_000124')&$select=RowKey,Manager Rating,Peer Rating,Comments  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* Debe usar un carácter separador adecuado que facilite el análisis del valor `RowKey`: por ejemplo, **000123_2012**.  
* También almacena esta entidad en la misma partición que otras entidades que contienen datos relacionados correspondientes al mismo empleado. Esto significa que puede usar EGT para mantener una coherencia segura.
* Debe considerar la frecuencia con la que consultará los datos para determinar si este patrón es adecuado. Por ejemplo, si tendrá acceso a los datos de revisión con poca frecuencia y a menudo a los datos de empleados principales debe guardarlos como entidades independientes.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Utilice este patrón cuando necesite almacenar una o más entidades relacionadas que consulte con frecuencia.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Transacciones de grupos de entidades](#entity-group-transactions)  
* [Work with heterogeneous entity types](#work-with-heterogeneous-entity-types) (Uso de tipos de entidad heterogéneos)  
* [Patrón final coherente de transacciones](#eventually-consistent-transactions-pattern)  

### <a name="log-tail-pattern"></a>Patrón final del registro
Recupere las entidades *n* agregadas recientemente a una partición utilizando un valor `RowKey` que se ordene en orden de fecha y hora inverso.  

> [!NOTE]
> Los resultados de consulta devueltos por la Table API de Azure en Azure Cosmos DB no se ordenan por clave de fila ni por clave de partición. Por lo tanto, si bien este patrón es adecuado para Table Storage, no lo es para Azure Cosmos DB. Para ver una lista detallada de las diferencias de las características, consulte las [diferencias de Table API en Azure Cosmos DB y Azure Table Storage](table-api-faq.md#table-api-vs-table-storage).

#### <a name="context-and-problem"></a>Contexto y problema
Un requisito común es poder recuperar las entidades creadas más recientemente, por ejemplo, las últimas diez reclamaciones de gastos enviadas por un empleado. Las consultas de tabla admiten una operación de consulta `$top` para devolver las primeras *n* entidades de un conjunto. No hay ninguna operación de consulta equivalente para devolver las últimas *n* entidades de un conjunto.  

#### <a name="solution"></a>Solución
Almacene las entidades mediante un valor `RowKey` que ordene naturalmente en orden inverso de fecha y hora de modo que la entrada más reciente sea siempre la primera de la tabla.  

Por ejemplo, para poder recuperar las diez reclamaciones de gastos más recientes enviadas por un empleado, puede utilizar un valor de marca inversa derivado de la fecha y hora actuales. El siguiente ejemplo de código de C# muestra una forma de crear un valor de "marcas invertidas" adecuado para un valor `RowKey` que ordene de más reciente a más antiguo:  

`string invertedTicks = string.Format("{0:D19}", DateTime.MaxValue.Ticks - DateTime.UtcNow.Ticks);`  

Puede volver al valor de fecha y hora utilizando el código siguiente:  

`DateTime dt = new DateTime(DateTime.MaxValue.Ticks - Int64.Parse(invertedTicks));`  

La consulta de la tabla tiene este aspecto:  

`https://myaccount.table.core.windows.net/EmployeeExpense(PartitionKey='empid')?$top=10`  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* Debe rellenar el valor de tic inverso con ceros a la izquierda para asegurarse de que el valor de cadena se ordene según lo esperado.  
* Debe ser consciente de los objetivos de escalabilidad en el nivel de una partición. Tenga cuidado de no crear particiones en la zona activa.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Utilice este patrón cuando necesite tener acceso a entidades en orden inverso de fecha y hora o cuando necesite tener acceso a las entidades que haya agregado más recientemente.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Anteponer/anexar antipatrón](#prepend-append-anti-pattern)  
* [Recuperación de entidades](#retrieve-entities)  

### <a name="high-volume-delete-pattern"></a>Patrón de eliminación de gran volumen
Habilite la eliminación de un gran volumen de entidades mediante el almacenamiento de todas las entidades para su eliminación simultánea en su propia tabla independiente. Elimine las entidades mediante la eliminación de la tabla.  

#### <a name="context-and-problem"></a>Contexto y problema
Muchas aplicaciones eliminarán datos antiguos que ya no necesita que estén disponibles para una aplicación cliente o que la aplicación haya archivado en otro medio de almacenamiento. Normalmente, estos datos se identifican por fecha. Por ejemplo, tiene un requisito para eliminar registros de todas las solicitudes de inicio de sesión que tengan más de 60 días.  

Un diseño posible es utilizar la fecha y hora de la solicitud de inicio de sesión en el valor `RowKey`:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE21.png" alt-text="Gráfico de una entidad de intento de inicio de sesión":::

Este enfoque evita las zonas activas de las particiones, porque la aplicación puede insertar y eliminar entidades de inicio de sesión para cada usuario en una partición independiente. Sin embargo, este enfoque puede ser costoso y lento si tiene un gran número de entidades. En primer lugar, debe realizar un recorrido de tabla para identificar todas las entidades que se van a eliminar y, a continuación, debe eliminar cada entidad antigua. Puede reducir el número de viajes de ida y vuelta al servidor necesarios para eliminar las entidades antiguas almacenando por lotes varias solicitudes de eliminación en EGT.  

#### <a name="solution"></a>Solución
Utilice una tabla independiente para cada día de intentos de inicio de sesión. Puede usar el diseño de entidad anterior para evitar las zonas activas cuando inserte entidades. La eliminación de entidades anteriores ahora es simplemente una cuestión de eliminar una tabla todos los días (una operación de almacenamiento único) en lugar de buscar y eliminar cientos de miles de entidades de inicio de sesión individuales cada día.  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* ¿Su diseño admite otras formas de uso por parte de su aplicación de los datos como la búsqueda de entidades específicas, vinculación con otros datos o generar información de agregado?  
* ¿Evita el diseño problemas cuando se insertan nuevas entidades?  
* Espere un retraso si desea reutilizar el mismo nombre de tabla después de eliminarlo. Es mejor utilizar siempre nombres de tabla únicos.  
* Espere ciertas limitaciones de velocidad al primero una tabla nueva mientras Table Storage aprende los patrones de acceso y distribuye las particiones entre los nodos. Debe considerar la frecuencia con la que necesita crear nuevas tablas.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Utilice este patrón cuando tenga un gran volumen de entidades que deba eliminar al mismo tiempo.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Transacciones de grupos de entidades](#entity-group-transactions)
* [Modificar entidades](#modify-entities)  

### <a name="data-series-pattern"></a>Patrón de serie de datos
Almacene una serie de datos completa en una sola entidad para minimizar el número de solicitudes que realice.  

#### <a name="context-and-problem"></a>Contexto y problema
Un escenario común para una aplicación es almacenar una serie de datos que normalmente necesite recuperar al mismo tiempo. Por ejemplo, la aplicación podría registrar el número de mensajes de MI que envía cada hora cada empleado y, a continuación, utilizar esta información para trazar cuántos mensajes envió cada usuario durante las 24 horas anteriores. Un diseño podría ser almacenar 24 entidades para cada empleado:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE22.png" alt-text="Gráfico de una entidad de estadísticas de mensajes":::

Con este diseño, puede localizar y actualizar fácilmente la entidad que se va a actualizar para cada empleado, siempre que la aplicación necesite actualizar el valor de recuento de mensajes. Sin embargo, para recuperar la información para trazar un gráfico de la actividad durante las 24 horas anteriores, debe recuperar 24 entidades.  

#### <a name="solution"></a>Solución
Utilice el siguiente diseño con una propiedad independiente para almacenar el número de mensajes de cada hora:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE23.png" alt-text="Gráfico que muestra una entidad de estadísticas de mensajes con propiedades independientes":::

Con este diseño, puede utilizar una operación de combinación para actualizar el número de mensajes de un empleado para una hora concreta. Ahora puede recuperar toda la información que necesita para trazar el gráfico mediante una solicitud para una entidad única.  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* Si la serie de datos completa no cabe en una única entidad (una entidad puede tener hasta 252 propiedades), utilice un almacén de datos alternativo, como un blob.  
* Si tiene varios clientes actualizando una entidad simultáneamente, utilice el valor de **ETag** para implementar la simultaneidad optimista. Si tiene muchos clientes, puede experimentar un alto nivel de contención.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Utilice este patrón cuando necesite actualizar y recuperar una serie de datos asociada con una entidad individual.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Patrón de entidades de gran tamaño](#large-entities-pattern)  
* [Combinar o reemplazar](#merge-or-replace)  
* [Patrón final coherente de transacciones](#eventually-consistent-transactions-pattern) (si va a almacenar la serie de datos en un blob)  

### <a name="wide-entities-pattern"></a>Patrón de entidades amplio
Use varias entidades físicas para almacenar entidades lógicas con más de 252 propiedades.  

#### <a name="context-and-problem"></a>Contexto y problema
Una entidad individual no puede tener más de 252 propiedades (excepto las propiedades del sistema obligatorias) y no puede almacenar más de 1 MB de datos en total. En una base de datos relacional, normalmente se encontrará con límites en el tamaño de una fila al agregar una nueva tabla e imponer una relación de 1 a 1 entre ellas.  

#### <a name="solution"></a>Solución
Con Table Storage, puede almacenar varias entidades para representar un objeto único de gran empresa con más de 252 propiedades. Por ejemplo, si desea almacenar un recuento del número de mensajes de mensajería instantánea enviados por cada empleado durante los últimos 365 días, podría utilizar el siguiente diseño que usa dos entidades con distintos esquemas:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE24.png" alt-text="Gráfico que muestra una entidad de estadísticas de mensajes con Rowkey 01 y una entidad de estadísticas de mensajes con Rowkey 02":::

Si necesita realizar un cambio que requiere la actualización de ambas entidades para mantenerlas sincronizadas entre sí puede utilizar un EGT. De lo contrario, puede utilizar una única operación de combinación para actualizar el número de mensajes para un día concreto. Para recuperar todos los datos de un empleado individual, debe recuperar ambas entidades. Puede hacerlo con dos solicitudes eficientes que usan un valor `PartitionKey` y `RowKey`.  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta el punto siguiente al decidir cómo implementar este patrón:  

* Recuperar una entidad lógica completa implica al menos dos transacciones de almacenamiento: una para recuperar cada entidad física.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Utilice este patrón cuando necesite almacenar entidades cuyo tamaño o número de propiedades supere los límites de una entidad individual en Table Storage.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Transacciones de grupos de entidades](#entity-group-transactions)
* [Combinar o reemplazar](#merge-or-replace)

### <a name="large-entities-pattern"></a>Patrón de entidades de gran tamaño
Use Blob Storage para almacenar valores de propiedad de gran tamaño.  

#### <a name="context-and-problem"></a>Contexto y problema
Una entidad individual no puede almacenar más de 1 MB de datos en total. Si una o varias de sus propiedades almacenan valores que provocan que el tamaño total de la entidad supere este valor, no puede almacenar toda la entidad en Table Storage.  

#### <a name="solution"></a>Solución
Si la entidad supera 1 MB de tamaño porque una o varias propiedades contienen una gran cantidad de datos, puede almacenar datos en Blob Storage y, a continuación, almacenar la dirección del blob en una propiedad de la entidad. Por ejemplo, puede almacenar la foto de un empleado en Blob Storage y almacenar un vínculo a la foto en la propiedad `Photo` de la entidad de empleado:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE25.png" alt-text="Gráfico que muestra una entidad de empleado con cadena de foto que apunta a Blob Storage":::

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* Para mantener la coherencia final entre la entidad de Table Storage y los datos de Blob Storage, utilice el [patrón final coherente de transacciones](#eventually-consistent-transactions-pattern) para mantener las entidades.
* Recuperar una entidad completa implica al menos dos transacciones de almacenamiento: una para recuperar la entidad y otra para recuperar los datos del blob.  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Utilice este patrón cuando necesite almacenar entidades cuyo tamaño supere los límites para una entidad individual en Table Storage.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Patrón final coherente de transacciones](#eventually-consistent-transactions-pattern)  
* [Patrón de entidades amplio](#wide-entities-pattern)

<a name="prepend-append-anti-pattern"></a>

### <a name="prependappend-anti-pattern"></a>Antipatrón de anteponer/anexar
Cuando tenga un alto volumen de inserciones, aumente la escalabilidad repartiéndolas en varias particiones.  

#### <a name="context-and-problem"></a>Contexto y problema
Anteponer o anexar las entidades a las entidades almacenadas normalmente provoca en la aplicación la adición de nuevas entidades a la primera o última partición de una secuencia de particiones. En este caso, todas las inserciones en un momento determinado están teniendo lugar en la misma partición, creando una zona activa. Esto evita que Table Storage realice inserciones de equilibrio de carga entre varios nodos y, posiblemente, hace que la aplicación alcance los objetivos de escalabilidad de la partición. Por ejemplo, piense en una aplicación que registre el acceso a la red y los recursos por parte de los empleados. Una estructura de entidad como la siguiente puede dar lugar a que la partición de la hora actual se convierta en una zona activa, si el volumen de transacciones alcanza el objetivo de escalabilidad para una partición individual:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE26.png" alt-text="Gráfico de una entidad de empleado":::

#### <a name="solution"></a>Solución
La siguiente estructura de una entidad alternativa evita puntos de conflicto en una partición determinada a medida que la aplicación registra eventos:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE27.png" alt-text="Gráfico que muestra una entidad de empleado con RowKey que se compone de año, mes, día, hora e identificador de evento":::

Observe en este ejemplo que tanto `PartitionKey` como `RowKey` son claves compuestas. `PartitionKey` usa tanto el departamento como el identificador de empleado para distribuir el registro entre varias particiones.  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los puntos siguientes al decidir cómo implementar este patrón:  

* ¿Admite la estructura de clave alternativa que evita la creación de particiones activas en inserciones eficazmente las consultas que realiza la aplicación cliente?  
* ¿El volumen de transacciones previstas significa que es probable alcanzar los objetivos de escalabilidad para una partición individual y que esté limitada por Table Storage?  

#### <a name="when-to-use-this-pattern"></a>Cuándo usar este patrón
Evite el antipatrón anteponer/anexar cuando es posible que el volumen de transacciones provoque una limitación de la velocidad por parte de Table Storage cuando acceda a una partición activa.  

#### <a name="related-patterns-and-guidance"></a>Orientación y patrones relacionados
Los patrones y las directrices siguientes también pueden ser importantes a la hora de implementar este modelo:  

* [Patrón de clave compuesta](#compound-key-pattern)  
* [Patrón de cola de registro](#log-tail-pattern)  
* [Modificar entidades](#modify-entities)  

### <a name="log-data-anti-pattern"></a>Antipatrón de datos de registro
Normalmente, debe utilizar Blob Storage en lugar de Table Storage para almacenar los datos de registro.  

#### <a name="context-and-problem"></a>Contexto y problema
Un caso de uso común para los datos de registro consiste en recuperar una selección de entradas de registro para un intervalo de fecha y hora específico. Por ejemplo, desea buscar todos los mensajes de error y críticos que la aplicación registró entre las 15:04 y las 15:06 en una fecha concreta. No desea utilizar la fecha y hora del mensaje de registro para determinar la partición en la que se guardan las entidades de registro. Eso da como resultado una partición activa porque, en un momento determinado, todas las entidades de registro compartirán el mismo valor `PartitionKey` (consulte el [antipatrón de anteponer/anexar](#prepend-append-anti-pattern)). Por ejemplo, el siguiente esquema de entidad para un mensaje de registro produce una partición activa debido a que la aplicación escribe todos los mensajes de registro en la partición en la fecha y la hora actuales:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE28.png" alt-text="Gráfico de una entidad de mensaje de registro":::

En este ejemplo, el valor `RowKey` incluye la fecha y hora del mensaje de registro para asegurarse de que los mensajes de registro se ordenan teniendo en cuenta la fecha y la hora. El valor `RowKey` también incluye un identificador de mensaje, en caso de que varios mensajes de registro compartan la misma fecha y hora.  

Otro enfoque consiste en utilizar un valor `PartitionKey` que garantice que la aplicación escriba los mensajes en un intervalo de particiones. Por ejemplo, si el origen del mensaje de registro proporciona una manera de distribuir los mensajes entre muchas particiones, puede utilizar el siguiente esquema de entidad:  

:::image type="content" source="./media/storage-table-design-guide/storage-table-design-IMAGE29.png" alt-text="Gráfico de una entidad de mensaje de registro":::

Sin embargo, el problema con este esquema es que para recuperar todos los mensajes de registro de un intervalo de tiempo específico debe buscar todas las particiones de la tabla.

#### <a name="solution"></a>Solución
En la sección anterior se resaltó el problema de intentar utilizar Table Storage para almacenar las entradas del registro, y se sugirieron dos diseños no satisfactorios. Una solución provocó una partición activa con el riesgo de obtener un bajo rendimiento de escritura de mensajes de registro. La otra solución ocasionó un bajo rendimiento de consultas debido a la necesidad de examinar cada partición de la tabla para recuperar los mensajes de registro de un intervalo de tiempo específico. Blob Storage ofrece una mejor solución para este tipo de escenario y así es como almacena Azure Storage Analytics los datos de registro que recopila.  

En esta sección se describe cómo almacena Storage Analytics los datos de registro en Blob Storage para ilustrar este método de almacenamiento de datos que se suele consultar por intervalo.  

Storage Analytics almacena los mensajes de registro en un formato delimitado en varios blobs. El formato delimitado facilita a una aplicación cliente analizar los datos del mensaje de registro.  

Storage Analytics utiliza una convención de nomenclatura para los blobs que le permite localizar el blob (o blobs) que contienen los mensajes de registro que está buscando. Por ejemplo, un blob denominado "queue/2014/07/31/1800/000001.log" contiene los mensajes de registro relacionados con el servicio de cola con hora de inicio a las 18:00 del 31 de julio de 2014. El "000001" indica que se trata del primer archivo de registro de este período. Storage Analytics también registra las marcas de tiempo del primer y último mensaje de registro almacenados en el archivo como parte de los metadatos del blob. La API para Blob Storage permite buscar blobs en un contenedor basado en un prefijo de nombre. Para buscar todos los blobs que contienen datos de registro de cola durante la hora que comienza a las 18:00, puede usar el prefijo "queue/2014/07/31/1800".  

Storage Analytics almacena en búfer los mensajes de registro internamente y, a continuación, actualiza periódicamente el blob adecuado o crea uno nuevo con el último lote de entradas de registro. Esto reduce el número de escrituras que se debe realizar en Blob Storage.  

Si está implementando una solución similar en su propia aplicación, recuerde que es necesario administrar el equilibrio entre la confiabilidad y el costo y la escalabilidad. En otras palabras, evalúe el efecto de escribir cada entrada de registro en Blob Storage a medida que se produzca en comparación con almacenar en búfer las actualizaciones en la aplicación y escribirlas en Blob Storage en lotes.  

#### <a name="issues-and-considerations"></a>Problemas y consideraciones
Tenga en cuenta los siguientes puntos cuando decida cómo almacenar los datos del registro:  

* Si crea un diseño de tabla que evite posibles particiones activas, observará que no tiene acceso a los datos del registro de forma eficaz.  
* Para procesar los datos de registro, un cliente a menudo necesita cargar muchos registros.  
* Aunque a menudo se estructuran datos del registro, Blob Storage puede ser una solución mejor.  

### <a name="implementation-considerations"></a>Consideraciones de implementación
En esta sección se describen algunas de las consideraciones a tener en cuenta al implementar los modelos descritos en las secciones anteriores. En la mayor parte de esta sección se utilizan ejemplos escritos en C# que utilizan la biblioteca de clientes de Storage (versión 4.3.0 en el momento de escribir).  

### <a name="retrieve-entities"></a>Recuperación de entidades
Como se describe en la sección [Diseño para consultas](#design-for-querying), la consulta más eficaz es una puntual. Sin embargo, en algunos casos puede que necesite recuperar varias entidades. En esta sección se describen algunos enfoques comunes para recuperar entidades mediante la biblioteca cliente de Storage.  

#### <a name="run-a-point-query-by-using-the-storage-client-library"></a>Ejecución de una consulta puntual mediante la biblioteca cliente de Storage
La manera más sencilla de ejecutar una consulta puntual es usar la operación de tabla **Retrieve**. Como se muestra en el siguiente fragmento de código de C#, esta operación recupera una entidad con un `PartitionKey` de valor "Sales" y un `RowKey` de valor "212":  

```csharp
TableOperation retrieveOperation = TableOperation.Retrieve<EmployeeEntity>("Sales", "212");
var retrieveResult = employeeTable.Execute(retrieveOperation);
if (retrieveResult.Result != null)
{
    EmployeeEntity employee = (EmployeeEntity)retrieveResult.Result;
    ...
}  
```

Observe cómo este ejemplo espera que la entidad que recupera sea del tipo `EmployeeEntity`.  

#### <a name="retrieve-multiple-entities-by-using-linq"></a>Recuperación de varias entidades mediante LINQ
Puede recuperar varias entidades mediante LINQ con la biblioteca cliente de Storage y especificar una consulta con una cláusula **where**. Para evitar un recorrido de tabla, debe incluir siempre el valor `PartitionKey` en la cláusula where y, si es posible, el valor `RowKey` para evitar exámenes de tablas y de particiones. Table Storage admite un conjunto limitado de operadores de comparación (mayor que, mayor o igual que, menor que, menor o igual que, igual y no igual a) para utilizar en la cláusula where. El siguiente fragmento de código de C# busca todos los empleados cuyo apellido empieza por "B" (suponiendo que `RowKey` almacene el apellido) del departamento Sales (suponiendo que `PartitionKey` almacene el nombre del departamento):  

```csharp
TableQuery<EmployeeEntity> employeeQuery = employeeTable.CreateQuery<EmployeeEntity>();
var query = (from employee in employeeQuery
            where employee.PartitionKey == "Sales" &&
            employee.RowKey.CompareTo("B") >= 0 &&
            employee.RowKey.CompareTo("C") < 0
            select employee).AsTableQuery();
var employees = query.Execute();  
```

Observe que la consulta especifica un valor `RowKey` y un valor `PartitionKey` para asegurar un mejor rendimiento.  

El siguiente ejemplo de código muestra una funcionalidad equivalente mediante la API fluida (para obtener más información acerca de las API fluidas en general, consulte [Procedimientos recomendados para diseñar una API fluida](https://visualstudiomagazine.com/articles/2013/12/01/best-practices-for-designing-a-fluent-api.aspx)):  

```csharp
TableQuery<EmployeeEntity> employeeQuery = new TableQuery<EmployeeEntity>().Where(
    TableQuery.CombineFilters(
    TableQuery.CombineFilters(
        TableQuery.GenerateFilterCondition(
    "PartitionKey", QueryComparisons.Equal, "Sales"),
    TableOperators.And,
    TableQuery.GenerateFilterCondition(
    "RowKey", QueryComparisons.GreaterThanOrEqual, "B")
),
TableOperators.And,
TableQuery.GenerateFilterCondition("RowKey", QueryComparisons.LessThan, "C")
    )
);
var employees = employeeTable.ExecuteQuery(employeeQuery);  
```

> [!NOTE]
> El ejemplo anida varios métodos `CombineFilters` para incluir las tres condiciones de filtro.  
> 
> 

#### <a name="retrieve-large-numbers-of-entities-from-a-query"></a>Recuperación de una gran cantidad de entidades de una consulta
Una consulta óptima devuelve una entidad individual basada en un valor `PartitionKey` y un valor `RowKey`. Sin embargo, en algunos escenarios puede tener el requisito de devolver varias entidades de la misma partición o incluso de varias particiones. Siempre se debe probar a fondo el rendimiento de la aplicación en estas situaciones.  

Una consulta en Table Storage puede devolver un máximo de 1000 entidades al mismo tiempo, y se puede ejecutar durante un máximo de 5 segundos. Table Storage devuelve un token de continuación para permitir que la aplicación cliente solicite el siguiente conjunto de entidades, si se cumple alguna de las siguientes condiciones:

- El conjunto de resultados contiene más de 1000 entidades.
- La consulta no se completó en 5 segundos.
- La consulta supera el límite de la partición. 

Para más información sobre el funcionamiento de los tokens de continuación, consulte [Tiempo de espera de consulta y paginación](https://msdn.microsoft.com/library/azure/dd135718.aspx).  

Si utiliza la biblioteca cliente de Storage, puede controlar automáticamente los tokens de continuación cuando devuelve entidades de Table Storage. Por ejemplo, el siguiente ejemplo de código de C# controla automáticamente tokens de continuación si Table Storage los devuelve en una respuesta:  

```csharp
string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

var employees = employeeTable.ExecuteQuery(employeeQuery);
foreach (var emp in employees)
{
        ...
}  
```

El siguiente código C# administra los tokens de continuación explícitamente:  

```csharp
string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

TableContinuationToken continuationToken = null;

do
{
        var employees = employeeTable.ExecuteQuerySegmented(
        employeeQuery, continuationToken);
    foreach (var emp in employees)
    {
    ...
    }
    continuationToken = employees.ContinuationToken;
} while (continuationToken != null);  
```

Mediante el uso de tokens de continuación explícitamente, puede controlar cuando recupera la aplicación el siguiente segmento de datos. Por ejemplo, si la aplicación cliente permite a los usuarios desplazarse por las entidades almacenadas en una tabla, un usuario podría decidir no desplazarse por todas las entidades recuperadas por la consulta. La aplicación solo usaría un token de continuación para recuperar el siguiente segmento cuando el usuario haya terminado de desplazarse por todas las entidades del segmento actual. Este enfoque tiene varias ventajas:  

* Puede limitar la cantidad de datos que desea recuperar de Table Storage y desplazarse a través de la red.  
* Puede realizar operaciones de E/S asincrónicas en .NET.  
* Puede serializar el token de continuación en un almacenamiento persistente para que pueda continuar en caso de un bloqueo de la aplicación.  

> [!NOTE]
> Normalmente, un token de continuación devuelve un segmento que contiene 1000 entidades, aunque pueden ser menos. Este también es el caso si limita el número de entradas que devuelve una consulta mediante **Take** para devolver las n primeras entidades que coincidan con los criterios de búsqueda. Table Storage puede devolver un segmento que contenga menos de n entidades, junto con un token de continuación que permita recuperar las entidades restantes.  
> 
> 

El siguiente código de C# muestra cómo modificar el número de entidades devueltas dentro de un segmento:  

```csharp
employeeQuery.TakeCount = 50;  
```

#### <a name="server-side-projection"></a>Proyección de servidor
Una sola entidad puede tener hasta 255 propiedades y ocupar hasta 1 MB. Al consultar la tabla y recuperar las entidades, puede que no necesite todas las propiedades, y puede evitar la transferencia de datos innecesaria (para ayudar a reducir la latencia y el coste). Puede usar proyección de servidor para transferir solo las propiedades que necesita. En el ejemplo siguiente se recupera solo la propiedad `Email` (junto con `PartitionKey`, `RowKey`, `Timestamp` y `ETag`) de las entidades seleccionadas por la consulta.  

```csharp
string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
List<string> columns = new List<string>() { "Email" };
TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter).Select(columns);

var entities = employeeTable.ExecuteQuery(employeeQuery);
foreach (var e in entities)
{
        Console.WriteLine("RowKey: {0}, EmployeeEmail: {1}", e.RowKey, e.Email);
}  
```

Observe que el valor `RowKey` está disponible incluso si no se incluyó en la lista de propiedades para recuperar.  

### <a name="modify-entities"></a>Modificar entidades
La biblioteca cliente de Storage permite modificar las entidades almacenadas en Table Storage insertando, eliminando y actualizando entidades. Puede usar las EGT para procesar por lotes varias operaciones de inserción, actualización y eliminación conjuntamente para reducir el número de recorridos de ida y vuelta requeridos y mejorar el rendimiento de la solución.  

Entre las excepciones que se producen cuando la biblioteca cliente de Storage ejecuta una EGT normalmente se incluye el índice de la entidad que ha provocado el error del lote. Esto resulta útil cuando se depura código que usa EGT.  

También debe considerar cómo afecta su diseño a la forma en que la aplicación cliente trata las operaciones de simultaneidad y actualización.  

#### <a name="managing-concurrency"></a>Administrar la simultaneidad
De forma predeterminada, Table Storage implementa comprobaciones de simultaneidad optimista en el nivel de entidades individuales para las operaciones de inserción, combinación y eliminación, aunque es posible que un cliente fuerce a Table Storage a omitir estas comprobaciones. Para obtener más información, consulte [Administración de la simultaneidad en Microsoft Azure Storage](../storage/common/storage-concurrency.md).  

#### <a name="merge-or-replace"></a>Combinar o reemplazar
El método `Replace` de la clase `TableOperation` siempre reemplaza la entidad completa en Table Storage. Si no incluye una propiedad en la solicitud cuando esa propiedad existe en la entidad almacenada, la solicitud quita esa propiedad de la entidad almacenada. A menos que desee quitar una propiedad de forma explícita de entidad almacenada, debe incluir todas las propiedades en la solicitud.  

Puede utilizar el método `Merge` de la clase `TableOperation` para reducir la cantidad de datos que envía a Table Storage si desea actualizar una entidad. El método `Merge` reemplaza cualquier propiedad de la entidad almacenada por los valores de propiedad de la entidad incluida en la solicitud. Este método deja intactas las propiedades de la entidad almacenada que no se incluyen en la solicitud. Esto resulta útil si tiene entidades de gran tamaño y solo tiene que actualizar un pequeño número de propiedades en una solicitud.  

> [!NOTE]
> Los métodos `*Replace` y `Merge` producen un error si la entidad no existe. Como alternativa, puede usar los métodos `InsertOrReplace` o `InsertOrMerge`, que crean una nueva entidad si todavía no existe.  
> 
> 

### <a name="work-with-heterogeneous-entity-types"></a>Uso de tipos de entidad heterogéneos
Table Storage es un almacén de tabla *sin esquema*. Esto significa que una sola tabla puede almacenar entidades de varios tipos, lo que proporciona gran flexibilidad en el diseño. En el ejemplo siguiente se muestra una tabla que almacena entidades de empleado y de departamento:  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Nombre</th>
<th>Apellidos</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Nombre</th>
<th>Apellidos</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>Nombre</th>
<th>Apellidos</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

Cada entidad debe seguir teniendo valores `PartitionKey`, `RowKey` y `Timestamp`, pero puede tener cualquier conjunto de propiedades. Además, no hay nada que indique el tipo de una entidad, a menos que elija almacenar esa información en algún lugar. Hay dos opciones para identificar el tipo de entidad:  

* Anteponer el tipo de entidad al valor `RowKey` (o posiblemente `PartitionKey`). Por ejemplo, `EMPLOYEE_000123` o `DEPARTMENT_SALES` como valores `RowKey`.  
* Utilice una propiedad independiente para registrar el tipo de entidad, como se muestra en la tabla siguiente.  

<table>
<tr>
<th>PartitionKey</th>
<th>RowKey</th>
<th>Timestamp</th>
<th></th>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Nombre</th>
<th>Apellidos</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Employee</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Nombre</th>
<th>Apellidos</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Employee</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>DepartmentName</th>
<th>EmployeeCount</th>
</tr>
<tr>
<td>department</td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
<tr>
<td></td>
<td></td>
<td></td>
<td>
<table>
<tr>
<th>EntityType</th>
<th>Nombre</th>
<th>Apellidos</th>
<th>Age</th>
<th>Email</th>
</tr>
<tr>
<td>Employee</td>
<td></td>
<td></td>
<td></td>
<td></td>
</tr>
</table>
</td>
</tr>
</table>

La primera opción, anteponer el tipo de entidad a `RowKey`, resulta útil si existe la posibilidad de que dos entidades de tipos diferentes tengan el mismo valor de clave. También agrupa las entidades del mismo tipo juntas en la partición.  

Las técnicas descritas en esta sección son especialmente pertinentes para la explicación sobre las [relaciones de herencia](#inheritance-relationships).  

> [!NOTE]
> Considere la posibilidad de incluir un número de versión en el valor de tipo de entidad para permitir a las aplicaciones de cliente evolucionar objetos POCO y trabajar con distintas versiones.  
> 
> 

En el resto de esta sección se describen algunas de las características de la biblioteca de clientes de Storage que facilitan el trabajo con varios tipos de entidad en la misma tabla.  

#### <a name="retrieve-heterogeneous-entity-types"></a>Recuperación de tipos de entidad heterogéneos
Si utiliza la biblioteca cliente de Storage, tiene tres opciones para trabajar con varios tipos de entidad.  

Si conoce el tipo de la entidad almacenada con valores específicos de `RowKey` y `PartitionKey`, puede especificar el tipo de entidad al recuperar la entidad. Vio esto en los dos ejemplos anteriores que recuperan entidades de tipo `EmployeeEntity`: [Ejecución de una consulta puntual mediante la biblioteca cliente de Storage](#run-a-point-query-by-using-the-storage-client-library) y [Recuperación de varias entidades mediante LINQ](#retrieve-multiple-entities-by-using-linq).  

La segunda opción consiste en usar el tipo `DynamicTableEntity` (un contenedor de propiedades), en lugar de un tipo concreto de entidad POCO. Esta opción también puede mejorar el rendimiento, ya que no es necesario serializar y deserializar la entidad en tipos de .NET. Potencialmente, el siguiente código de C# recupera varias entidades de distintos tipos de la tabla, pero devuelve todas las entidades como instancias de `DynamicTableEntity`. A continuación, usa la propiedad `EntityType` para determinar el tipo de cada entidad:  

```csharp
string filter = TableQuery.CombineFilters(
    TableQuery.GenerateFilterCondition("PartitionKey",
    QueryComparisons.Equal, "Sales"),
    TableOperators.And,
    TableQuery.CombineFilters(
    TableQuery.GenerateFilterCondition("RowKey",
                    QueryComparisons.GreaterThanOrEqual, "B"),
        TableOperators.And,
        TableQuery.GenerateFilterCondition("RowKey",
        QueryComparisons.LessThan, "F")
    )
);
TableQuery<DynamicTableEntity> entityQuery =
    new TableQuery<DynamicTableEntity>().Where(filter);
var employees = employeeTable.ExecuteQuery(entityQuery);

IEnumerable<DynamicTableEntity> entities = employeeTable.ExecuteQuery(entityQuery);
foreach (var e in entities)
{
EntityProperty entityTypeProperty;
if (e.Properties.TryGetValue("EntityType", out entityTypeProperty))
{
    if (entityTypeProperty.StringValue == "Employee")
    {
        // Use entityTypeProperty, RowKey, PartitionKey, Etag, and Timestamp
        }
    }
}  
```

Para recuperar otras propiedades, debe utilizar el método `TryGetValue` en la propiedad `Properties` de la clase `DynamicTableEntity`.  

Una tercera opción es combinar utilizando el tipo `DynamicTableEntity` y una instancia de `EntityResolver`. Esto le permite resolver en varios tipos POCO en la misma consulta. En este ejemplo, el delegado `EntityResolver` usa la propiedad `EntityType` para distinguir entre los dos tipos de entidad que devuelve la consulta. El método `Resolve` usa el delegado `resolver` para resolver instancias `DynamicTableEntity` en instancias `TableEntity`.  

```csharp
EntityResolver<TableEntity> resolver = (pk, rk, ts, props, etag) =>
{

        TableEntity resolvedEntity = null;
        if (props["EntityType"].StringValue == "Department")
        {
        resolvedEntity = new DepartmentEntity();
        }
        else if (props["EntityType"].StringValue == "Employee")
        {
        resolvedEntity = new EmployeeEntity();
        }
        else throw new ArgumentException("Unrecognized entity", "props");

        resolvedEntity.PartitionKey = pk;
        resolvedEntity.RowKey = rk;
        resolvedEntity.Timestamp = ts;
        resolvedEntity.ETag = etag;
        resolvedEntity.ReadEntity(props, null);
        return resolvedEntity;
};

string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, "Sales");
TableQuery<DynamicTableEntity> entityQuery =
        new TableQuery<DynamicTableEntity>().Where(filter);

var entities = employeeTable.ExecuteQuery(entityQuery, resolver);
foreach (var e in entities)
{
        if (e is DepartmentEntity)
        {
    ...
        }
        if (e is EmployeeEntity)
        {
    ...
        }
}  
```

#### <a name="modify-heterogeneous-entity-types"></a>Modificación de tipos de entidad heterogéneos
No es necesario conocer el tipo de una entidad para eliminarla, y siempre sabe el tipo de una entidad al insertarla. Sin embargo, puede usar el tipo `DynamicTableEntity` para actualizar una entidad sin conocer su tipo y sin utilizar una clase de entidad POCO. En el código de ejemplo siguiente se recupera una entidad individual y se comprueba que la propiedad `EmployeeCount`existe antes de actualizarla.  

```csharp
TableResult result =
        employeeTable.Execute(TableOperation.Retrieve(partitionKey, rowKey));
DynamicTableEntity department = (DynamicTableEntity)result.Result;

EntityProperty countProperty;

if (!department.Properties.TryGetValue("EmployeeCount", out countProperty))
{
        throw new
        InvalidOperationException("Invalid entity, EmployeeCount property not found.");
}
countProperty.Int32Value += 1;
employeeTable.Execute(TableOperation.Merge(department));  
```

### <a name="control-access-with-shared-access-signatures"></a>Control de acceso con firmas de acceso compartido
Puede utilizar tokens de firma de acceso compartido (SAS) para permitir a las aplicaciones cliente modificar (y consultar) entidades de tabla directamente, sin necesidad de autenticarse directamente con Table Storage. Normalmente, hay tres ventajas principales de utilizar SAS en su aplicación:  

* No es necesario distribuir la clave de la cuenta de almacenamiento a una plataforma no segura (por ejemplo, un dispositivo móvil) para permitir que ese dispositivo acceda y modifique entidades en Table Storage.  
* Puede descargar parte del trabajo que realizan los roles web y de trabajo en la administración de las entidades. Puede descargar los dispositivos cliente, como los equipos de los usuarios finales y los dispositivos móviles.  
* Puede asignar un conjunto de permisos restringido y con limitación de tiempo a un cliente (por ejemplo, para permitir el acceso de solo lectura a recursos específicos).  

Para más información sobre el uso de tokens de SAS con Table Storage, consulte [Uso de firmas de acceso compartido (SAS)](../storage/common/storage-dotnet-shared-access-signature-part-1.md).  

Sin embargo, todavía debe generar los tokens de SAS que conceden una aplicación cliente a las entidades en Table Storage. Hágalo en un entorno que tenga acceso seguro a las claves de la cuenta de almacenamiento. Normalmente, se utiliza un rol web o de trabajador para generar los tokens SAS y entregarlos a las aplicaciones cliente que necesitan tener acceso a las entidades. Dado que todavía hay una sobrecarga implicada en la generación y entrega tokens SAS a los clientes, debería considerar cómo reducir mejor esta sobrecarga, especialmente en escenarios de gran volumen.  

Es posible generar un token SAS que conceda acceso a un subconjunto de las entidades de una tabla. De forma predeterminada, se crea un token de SAS para una tabla completa. Pero también es posible especificar que el token de SAS conceda acceso a un intervalo de valores `PartitionKey`, o a un intervalo de valores `PartitionKey` y `RowKey`. Puede elegir generar tokens de SAS para usuarios individuales del sistema, de forma que el token de SAS de cada usuario solo les permita acceder a sus propias entidades en Table Storage.  

### <a name="asynchronous-and-parallel-operations"></a>Operaciones asincrónicas y paralelas
En caso de que reparta las solicitudes entre varias particiones, puede mejorar el rendimiento y la capacidad de respuesta del cliente mediante consultas asincrónicas o paralelas.
Por ejemplo, podría tener dos o más instancias de rol de trabajo con acceso a las tablas en paralelo. Podría tener roles de trabajo individuales responsables de determinados conjuntos de particiones, o simplemente tener varias instancias de rol de trabajo, cada una de ellas con acceso a todas las particiones de una tabla.  

Dentro de una instancia de cliente, puede mejorar el rendimiento mediante la ejecución de operaciones de almacenamiento de forma asincrónica. La biblioteca de clientes de Storage facilita la escritura de modificaciones y consultas asíncronas. Por ejemplo, puede comenzar con el método sincrónico que recupera todas las entidades de una partición. como se muestra en el siguiente código de C#:  

```csharp
private static void ManyEntitiesQuery(CloudTable employeeTable, string department)
{
        string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, department);
        TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);

        TableContinuationToken continuationToken = null;

        do
        {
        var employees = employeeTable.ExecuteQuerySegmented(
                employeeQuery, continuationToken);
        foreach (var emp in employees)
    {
        ...
    }
        continuationToken = employees.ContinuationToken;
        } while (continuationToken != null);
}  
```

Este código puede se modificar fácilmente para que la consulta se ejecute de forma asincrónica del modo siguiente:  

```csharp
private static async Task ManyEntitiesQueryAsync(CloudTable employeeTable, string department)
{
        string filter = TableQuery.GenerateFilterCondition(
        "PartitionKey", QueryComparisons.Equal, department);
        TableQuery<EmployeeEntity> employeeQuery =
        new TableQuery<EmployeeEntity>().Where(filter);
        TableContinuationToken continuationToken = null;

        do
        {
        var employees = await employeeTable.ExecuteQuerySegmentedAsync(
                employeeQuery, continuationToken);
        foreach (var emp in employees)
        {
            ...
        }
        continuationToken = employees.ContinuationToken;
            } while (continuationToken != null);
}  
```

En este ejemplo asincrónico, puede ver los cambios siguientes desde la versión sincrónica:  

* La firma del método incluye ahora el modificador `async` y devuelve una instancia de `Task`.  
* En lugar de llamar al método `ExecuteSegmented` para recuperar los resultados, el método llama ahora al método `ExecuteSegmentedAsync`. El método usa el modificador `await` para recuperar los resultados de forma asincrónica.  

La aplicación cliente puede llamar a este método varias veces, con valores diferentes en el parámetro `department`. Cada consulta se ejecuta en un subproceso independiente.  

No hay ninguna versión asincrónica del método `Execute` en la clase `TableQuery` porque la interfaz `IEnumerable` no admite la enumeración asincrónica.  

También puede insertar, actualizar y eliminar entidades de forma asincrónica. En el ejemplo de C# siguiente se muestra un método sencillo y sincrónico para insertar o reemplazar una entidad de empleado:  

```csharp
private static void SimpleEmployeeUpsert(CloudTable employeeTable,
        EmployeeEntity employee)
{
        TableResult result = employeeTable
        .Execute(TableOperation.InsertOrReplace(employee));
        Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
}  
```

Este código se puede modificar fácilmente para que la actualización se ejecute de forma asincrónica del modo siguiente:  

```csharp
private static async Task SimpleEmployeeUpsertAsync(CloudTable employeeTable,
        EmployeeEntity employee)
{
        TableResult result = await employeeTable
        .ExecuteAsync(TableOperation.InsertOrReplace(employee));
        Console.WriteLine("HTTP Status: {0}", result.HttpStatusCode);
}  
```

En este ejemplo asincrónico, puede ver los cambios siguientes desde la versión sincrónica:  

* La firma del método incluye ahora el modificador `async` y devuelve una instancia de `Task`.  
* En lugar de llamar al método `Execute` para actualizar la entidad, el método llama ahora al método `ExecuteAsync`. El método usa el modificador `await` para recuperar los resultados de forma asincrónica.  

La aplicación cliente puede llamar a varios métodos asincrónicos como este, y cada invocación de método se ejecuta en un subproceso independiente.  

