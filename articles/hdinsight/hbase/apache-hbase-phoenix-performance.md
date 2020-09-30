---
title: Rendimiento de Phoenix en Azure HDInsight
description: Procedimientos recomendados para optimizar el rendimiento de Apache Phoenix para clústeres de Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/27/2019
ms.openlocfilehash: 0dfb93db1af807459c37653189a90b754c933aa4
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89504798"
---
# <a name="apache-phoenix-performance-best-practices"></a>Procedimientos recomendados para mejorar el rendimiento de Apache Phoenix

El aspecto más importante del rendimiento de [Apache Phoenix](https://phoenix.apache.org/) es optimizar la instancia de [Apache HBase](https://hbase.apache.org/) subyacente. Phoenix crea un modelo de datos relacionales sobre HBase que convierte las consultas SQL en operaciones de HBase, como recorridos. Tanto el diseño del esquema de tabla como la selección y el orden de los campos en la clave principal y el uso que realiza de los índices afectan el rendimiento de Phoenix.

## <a name="table-schema-design"></a>Diseño de esquemas de tabla

Cuando crea una tabla en Phoenix, se almacena en una tabla HBase. La tabla HBase contiene grupos de columnas (familias de columnas) a las que se accede en conjunto. Una fila de la tabla Phoenix es una fila de la tabla HBase, donde cada fila consta de celdas con versión asociadas con una o varias columnas. Lógicamente, una fila HBase única es una colección de pares clave-valor, donde cada uno tiene el mismo valor de clave de fila. Es decir, cada par clave-valor tiene un atributo de clave de fila y el valor de ese atributo es igual para una fila determinada.

El diseño de esquemas de una tabla Phoenix incluye el diseño de la clave principal, el diseño de la familia de columnas, el diseño de las columnas individuales y la manera de particionar los datos.

### <a name="primary-key-design"></a>Diseño de la clave principal

La clave principal que se define en una tabla en Phoenix determina cómo se almacenan los datos dentro de la clave de fila de la tabla HBase subyacente. En HBase, la única manera de acceder a una fila determinada es con la clave de fila. Además, los datos almacenados en una tabla HBase se ordenan según la clave de fila. Phoenix construye el valor de clave de fila al concatenar los valores de cada una de las columnas de la fila, en el orden en que están definidos en la clave principal.

Por ejemplo, una tabla para los contactos tiene el nombre, el apellido, el número de teléfono y la dirección, todo en la misma familia de columnas. Puede definir una clave principal en función de un número de secuencia creciente:

|clave de fila|       address|   phone| firstName| lastName|
|------|--------------------|--------------|-------------|--------------|
|  1000|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole|
|  8396|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji|

Sin embargo, si habitualmente realiza consultas por lastName, es posible que esta clave principal no funcione correctamente, porque cada consulta requiere un recorrido de tabla completo para leer el valor de cada columnas lastName. En su lugar, puede definir una clave principal en las columnas lastName, firstName y número del seguro social. La última columna se usa para eliminar la ambigüedad entre dos residentes de la misma dirección que tengan el mismo nombre como, por ejemplo, padre e hijo.

|clave de fila|       address|   phone| firstName| lastName| socialSecurityNum |
|------|--------------------|--------------|-------------|--------------| ---|
|  1000|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole| 111 |
|  8396|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji| 222 |

Con esta clave principal nueva, las claves de fila que Phoenix genera serían:

|clave de fila|       address|   phone| firstName| lastName| socialSecurityNum |
|------|--------------------|--------------|-------------|--------------| ---|
|  Dole-John-111|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole| 111 |
|  Raji-Calvin-222|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji| 222 |

En la primera fila, los datos de la clave de fila se representan de la siguiente manera:

|clave de fila|       key|   value|
|------|--------------------|---|
|  Dole-John-111|address |1111 San Gabriel Dr.|  
|  Dole-John-111|phone |1-425-000-0002|  
|  Dole-John-111|firstName |John|  
|  Dole-John-111|lastName |Dole|  
|  Dole-John-111|socialSecurityNum |111|

Esta clave de fila ahora almacena una copia duplicada de los datos. Tenga en cuenta el tamaño y el número de las columnas que incluye en la clave principal, porque este valor se incluye con cada celda en la tabla HBase subyacente.

Además, si la clave principal tiene valores que aumentan de manera continuada, debe crear la tabla con *cubos de sal* para evitar crear zonas activas de escritura. Consulte las [particiones de datos](#partition-data).

### <a name="column-family-design"></a>Diseño de familia de columnas

Si se accede a algunas columnas con más frecuencia que a otras, debe crear varias familias de columnas para separar las columnas a las que se accede con frecuencia de las columnas a las que se accede con menos frecuencia.

Además, si se tiende a acceder a ciertas columnas en conjunto, colóquelas en la misma familia de columnas.

### <a name="column-design"></a>Diseño de columnas

* Mantenga las columnas VARCHAR aproximadamente por debajo de 1 MB por los costos de E/S de las columnas de gran tamaño. Cuando se procesan las consultas, HBase materializa en su totalidad las celdas antes de enviarlas al cliente y el cliente las recibe completas antes de entregarlas al código de la aplicación.
* Almacene los valores de columna con un formato compacto como protobuf, Avro, msgpack o BSON. No se recomienda usar JSON, porque es más grande.
* Considere la posibilidad de comprimir los datos antes del almacenamiento para reducir la latencia y los costos de E/S.

### <a name="partition-data"></a>Partición de datos

Phoenix le permite controlar el número de regiones en las que se distribuyen los datos, lo que puede aumentar considerablemente el rendimiento de lectura/escritura. Cuando se crea una tabla Phoenix, puede cifrar con sal los datos o dividirlos previamente.

Para cifrar con sal una tabla durante la creación, especifique el número de cubos de sal:

```sql
CREATE TABLE CONTACTS (...) SALT_BUCKETS = 16
```

Este cifrado con sal divide la tabla según los valores de las claves principales, mediante la elección automática de los valores. 

Para controlar dónde se producen las divisiones de la tabla, puede dividirla previamente proporcionando los valores de intervalo según los cuales se produce la división. Por ejemplo, para crear una división de tabla en tres regiones:

```sql
CREATE TABLE CONTACTS (...) SPLIT ON ('CS','EU','NA')
```

## <a name="index-design"></a>Diseño del índice

Un índice de Phoenix es una tabla HBase que almacena una copia de algunos o de todos los datos de la tabla indexada. Un índice mejora el rendimiento de tipos específicos de consultas.

Cuando tiene varios índices definidos y, a continuación, se consulta una tabla, Phoenix selecciona automáticamente el mejor índice para la consulta. El índice principal se crea automáticamente en función de las claves principales que selecciona.

En el caso de las consultas anticipadas, también puede crear índices secundarios si especifica sus columnas.

Cuando diseñe los índices:

* Cree solo los índices que necesita.
* Limite el número de índices en las tablas que se actualizan con frecuencia. Las actualizaciones de una tabla se traducen en escrituras tanto en la tabla principal como en las tablas de índice.

## <a name="create-secondary-indexes"></a>Creación de índices secundarios

Los índices secundarios pueden mejorar el rendimiento de lectura al convertir lo que sería un recorrido de tabla completo en una búsqueda puntual, a costa del espacio de almacenamiento y de la velocidad de escritura. Los índices secundarios se pueden agregar o quitar después de crear la tabla y no requieren cambios en las consultas existentes: simplemente, las consultas se ejecutan más rápido. En función de cuáles sean sus necesidades, considere la posibilidad de crear índices cubiertos, índices funcionales o ambos.

### <a name="use-covered-indexes"></a>Uso de índices cubiertos

Los índices cubiertos son los índices que incluyen datos de la fila además de los valores que se indexan. Después de encontrar la entrada de índice deseada, no es necesario acceder a la tabla principal.

Por ejemplo, en la tabla de contactos de ejemplo, podría crear un índice secundario solo en la columna socialSecurityNum. Este índice secundario aceleraría las consultas que filtran por los valores socialSecurityNum, pero la recuperación de los otros valores de campo requerirá otra lectura en la tabla principal.

|clave de fila|       address|   phone| firstName| lastName| socialSecurityNum |
|------|--------------------|--------------|-------------|--------------| ---|
|  Dole-John-111|1111 San Gabriel Dr.|1-425-000-0002|    John|Dole| 111 |
|  Raji-Calvin-222|5415 San Gabriel Dr.|1-230-555-0191|  Calvin|Raji| 222 |

Sin embargo, si habitualmente desea buscar el valor firstName y lastName si se proporciona el valor socialSecurityNum, puede crear un índice cubierto que incluya firstName y lastName como datos reales en la tabla de índice:

```sql
CREATE INDEX ssn_idx ON CONTACTS (socialSecurityNum) INCLUDE(firstName, lastName);
```

Este índice cubierto permite que la consulta siguiente adquiera todos los datos simplemente al leer de la tabla que contiene el índice secundario:

```sql
SELECT socialSecurityNum, firstName, lastName FROM CONTACTS WHERE socialSecurityNum > 100;
```

### <a name="use-functional-indexes"></a>Uso de índices funcionales

Los índices funcionales permiten crear un índice en una expresión arbitraria que espera que se use en las consultas. Una vez que existe un índice funcional y una consulta usa esa expresión, el índice se puede usar para recuperar los resultados en lugar de la tabla de datos.

Por ejemplo, podría crear un índice que le permita realizar búsquedas sin distinción de mayúsculas y minúsculas en la combinación de nombre y apellido de una persona:

```sql
CREATE INDEX FULLNAME_UPPER_IDX ON "Contacts" (UPPER("firstName"||' '||"lastName"));
```

## <a name="query-design"></a>Diseño de consultas

Las principales consideraciones en el diseño de consultas son:

* Comprender el plan de consulta y comprobar que se comporta según lo esperado.
* Combinar de manera eficaz.

### <a name="understand-the-query-plan"></a>Comprensión del plan de consulta

En [SQLLine](http://sqlline.sourceforge.net/), use EXPLAIN seguido de la consulta SQL para ver el plan de operaciones que realizará Phoenix. Compruebe que el plan:

* Usa la clave principal cuando corresponde.
* Usa los índices secundarios adecuados, en lugar de la tabla de datos.
* Usa RANGE SCAN o SKIP SCAN siempre que sea posible, en lugar de TABLE SCAN.

#### <a name="plan-examples"></a>Ejemplos de plan

Como ejemplo, supongamos que tiene una tabla denominada FLIGHTS que almacena información sobre los retrasos de los vuelos.

Para seleccionar todos los vuelos con un valor de airlineid de `19805`, donde airlineid es un campo que no se encuentra en la clave principal ni en ningún índice:

```sql
select * from "FLIGHTS" where airlineid = '19805';
```

Ejecute el comando explain de la siguiente manera:

```sql
explain select * from "FLIGHTS" where airlineid = '19805';
```

El plan de consulta tiene este aspecto:

```sql
CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN FULL SCAN OVER FLIGHTS
   SERVER FILTER BY AIRLINEID = '19805'
```

En este plan, observe la frase FULL SCAN OVER FLIGHTS. Esta frase indica que la ejecución hace una operación TABLE SCAN de todas las filas de la tabla en lugar de usar la opción más eficaz RANGE SCAN o SKIP SCAN.

Ahora, supongamos que desea consultar los vuelos del 2 de enero de 2014 con el operador `AA` con valor flightnum mayor que 1. Supongamos que las columnas year, month, dayofmonth, carrier y flightnum existen en la tabla de ejemplo y que todas forman parte de la clave principal compuesta. La consulta tendría el aspecto siguiente:

```sql
select * from "FLIGHTS" where year = 2014 and month = 1 and dayofmonth = 2 and carrier = 'AA' and flightnum > 1;
```

Examinemos el plan para esta consulta con:

```sql
explain select * from "FLIGHTS" where year = 2014 and month = 1 and dayofmonth = 2 and carrier = 'AA' and flightnum > 1;
```

El plan resultante es:

```sql
CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN RANGE SCAN OVER FLIGHTS [2014,1,2,'AA',2] - [2014,1,2,'AA',*]
```

Los valores entre corchetes son el intervalo de los valores para las claves principales. En este caso, los valores del intervalo se fijan en relación con year 2014, month 1 y dayofmonth 2, pero permiten valores para flightnum que parten en 2 y van hasta (`*`). Este plan de consulta confirma que la clave principal se usa según lo previsto.

A continuación, cree un índice en la tabla FLIGHTS denominada `carrier2_idx` que solo está en el campo carrier. Este índice también incluye flightdate, tailnum, origin y flightnum como columnas cubiertas cuyos datos también se almacenan en el índice.

```sql
CREATE INDEX carrier2_idx ON FLIGHTS (carrier) INCLUDE(FLIGHTDATE,TAILNUM,ORIGIN,FLIGHTNUM);
```

Suponga que desea obtener el valor carrier junto con el valor flightdate y tailnum, como se muestra en la columna siguiente:

```sql
explain select carrier,flightdate,tailnum from "FLIGHTS" where carrier = 'AA';
```

Debe ver que se usa este índice:

```sql
CLIENT 1-CHUNK PARALLEL 1-WAY ROUND ROBIN RANGE SCAN OVER CARRIER2_IDX ['AA']
```

Para una lista completa de los elementos que pueden aparecen en los resultados del plan explain, consulte la sección sobre los planes Explain en [Apache Phoenix Tuning Guide](https://phoenix.apache.org/tuning_guide.html) (Guía de ajuste de Apache Phoenix).

### <a name="join-efficiently"></a>Combinación eficaz

Por lo general, desea evitar las combinaciones a menos que un lado sea pequeño, en particular en las consultas frecuentes.

Si es necesario, puede realizar combinaciones de gran tamaño con la sugerencia `/*+ USE_SORT_MERGE_JOIN */`, pero una combinación de gran tamaño es una operación costosa que abarca enormes cantidades de filas. Si el tamaño total de todas las tablas del lado derecho superara la memoria disponible, use la sugerencia `/*+ NO_STAR_JOIN */`.

## <a name="scenarios"></a>Escenarios

Las instrucciones siguientes describen algunos patrones comunes.

### <a name="read-heavy-workloads"></a>Cargas de trabajo con mucha actividad de lectura

En casos de uso con mucha actividad de lectura, asegúrese de usar índices. Además, para ahorrar la sobrecarga en el tiempo de lectura, considere la posibilidad de crear índices cubiertos.

### <a name="write-heavy-workloads"></a>Cargas de trabajo con mucha actividad de escritura

En el caso de las cargas de trabajo con mucha actividad de escritura en las que la clave principal aumenta de manera uniforme, cree cubos de sal para ayudar a evitar las zonas activas de escritura, a costa del rendimiento de lectura general debido a los recorridos adicionales que se requieren. Además, cuando use UPSERT para escribir una gran cantidad de registros, desactive autoCommit y procese por lote los registros.

### <a name="bulk-deletes"></a>Eliminaciones masivas

Cuando elimine un conjunto de datos de gran tamaño, active autoCommit antes de emitir la consulta DELETE; de este modo, el cliente no necesita recordar las claves de fila de todas las filas eliminadas. AutoCommit evita que el cliente almacene en búfer las filas afectadas por la consulta DELETE, de manera que Phoenix puede eliminarlas directamente en los servidores de región sin el gasto que implica devolverlas al cliente.

### <a name="immutable-and-append-only"></a>Inalterable y de solo anexar

Si el escenario favorece la velocidad de escritura por sobre la integrar de datos, considere deshabilitar el registro de escritura previa cuando cree las tablas:

```sql
CREATE TABLE CONTACTS (...) DISABLE_WAL=true;
```

Para conocer detalles sobre esta y otras opciones, consulte [Apache Phoenix Grammar](https://phoenix.apache.org/language/index.html#options) (Gramática de Apache Phoenix).

## <a name="next-steps"></a>Pasos siguientes

* [Apache Phoenix Tuning Guide](https://phoenix.apache.org/tuning_guide.html) (Guía de ajuste de Apache Phoenix)
* [Secondary Indexes](https://phoenix.apache.org/secondary_indexing.html) (Índices secundarios)
