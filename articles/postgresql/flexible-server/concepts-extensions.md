---
title: 'Extensiones en Azure Database for PostgreSQL: Servidor flexible'
description: 'Obtenga información sobre las extensiones de Postgres disponibles en Azure Database for PostgreSQL: Servidor flexible'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/22/2020
ms.openlocfilehash: 1126c218f8e80b7d89183746890a3fae1357d29d
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932109"
---
# <a name="postgresql-extensions-in-azure-database-for-postgresql---flexible-server"></a>Extensiones de PostgreSQL en Azure Database for PostgreSQL: Servidor flexible

> [!IMPORTANT]
> Azure Database for PostgreSQL: Servidor flexible está en versión preliminar.

PostgreSQL ofrece la capacidad de ampliar la funcionalidad de su base de datos mediante extensiones. Las extensiones agrupan varios objetos SQL relacionados en un solo paquete que se puede cargar o quitar de la base de datos con un comando. Después de cargarse en la base de datos, las extensiones funcionan como características integradas.

## <a name="how-to-use-postgresql-extensions"></a>¿Cómo se utilizan las extensiones de PostgreSQL?
Para poder usar las extensiones de PostgreSQL, es preciso que estén instaladas en su base de datos. Para instalar una extensión determinada, ejecute el comando [CREATE EXTENSION](https://www.postgresql.org/docs/current/sql-createextension.html). Este comando carga los objetos empaquetados en la base de datos.

Azure Database for PostgreSQL admite un subconjunto de extensiones de claves, como se enumeran a continuación. Esta información también está disponible al ejecutar `SHOW azure.extensions;`. Las extensiones no enumeradas en este documento no se admiten en Azure Database for PostgreSQL: Servidor flexible. No puede crear o cargar su propia extensión en el servicio Azure Database for PostgreSQL.


## <a name="postgres-12-extensions"></a>Extensiones de Postgres 12

Las extensiones siguientes están disponibles en los servidores flexibles de Azure Database for PostgreSQL que tienen la versión 12 de Postgres. 

> [!div class="mx-tableFixed"]
> | **Extensión**| **Versión de la extensión** | **Descripción** |
> |---|---|---|
> |[amcheck](https://www.postgresql.org/docs/12/amcheck.html)                    | 1.2             | Funciones de comprobación de la integridad de la relación.|
> |[bloom](https://www.postgresql.org/docs/12/bloom.html)                    | 1,0             | Método de acceso de bloom: índice basado en archivos de firma.|
> |[btree_gin](https://www.postgresql.org/docs/12/btree-gin.html)                    | 1.3             | Compatibilidad con la indexación de tipos de datos comunes en GIN|
> |[btree_gist](https://www.postgresql.org/docs/12/btree-gist.html)                   | 1.5             | Compatibilidad con la indexación de tipos de datos comunes en GiST|
> |[citext](https://www.postgresql.org/docs/12/citext.html)                       | 1.6             | Tipo de datos para cadenas de caracteres que no distinguen mayúsculas de minúsculas|
> |[cube](https://www.postgresql.org/docs/12/cube.html)                         | 1.4             | Tipo de datos para los cubos multidimensionales|
> |[dblink](https://www.postgresql.org/docs/12/dblink.html)                       | 1.2             | Se conecta a otras bases de datos de PostgreSQL desde una base de datos|
> |[dict_int](https://www.postgresql.org/docs/12/dict-int.html)                     | 1.0             | Plantilla de diccionario de búsqueda de texto para números enteros|
> |[dict_xsyn](https://www.postgresql.org/docs/12/dict-xsyn.html)                     | 1,0             | Plantilla de diccionario de búsqueda de texto para el procesamiento de sinónimos extendido.|
> |[earthdistance](https://www.postgresql.org/docs/12/earthdistance.html)                | 1.1             | Calcula distancias de círculo máximo en la superficie de la Tierra|
> |[fuzzystrmatch](https://www.postgresql.org/docs/12/fuzzystrmatch.html)                | 1.1             | Determina las similitudes y la distancia entre las cadenas|
> |[hstore](https://www.postgresql.org/docs/12/hstore.html)                       | 1.6             | Tipo de datos para almacenar conjuntos de pares (clave/valor)|
> |[intagg](https://www.postgresql.org/docs/12/intagg.html)                     | 1.1             | Agregador y enumerador de enteros. (Obsoleto)|
> |[intarray](https://www.postgresql.org/docs/12/intarray.html)                     | 1.2             | Funciones, operadores e índices compatibles con matrices 1D de números enteros|
> |[isn](https://www.postgresql.org/docs/12/isn.html)                          | 1.2             | Tipos de datos para los estándares internacionales de numeración de productos|
> |[ltree](https://www.postgresql.org/docs/12/ltree.html)                        | 1.1             | Tipo de datos para las estructuras de árbol jerárquicas|
> |[pageinspect](https://www.postgresql.org/docs/12/pageinspect.html)                        | 1.7             | Inspección del contenido de páginas de bases de datos a un nivel bajo.|
> |[pg_buffercache](https://www.postgresql.org/docs/12/pgbuffercache.html)               | 1.3             | Examina la caché del búfer compartido|
> |[pg_freespacemap](https://www.postgresql.org/docs/12/pgfreespacemap.html)               | 1.2             | Examen de la asignación de espacio libre (FSM).|
> |[pg_prewarm](https://www.postgresql.org/docs/12/pgprewarm.html)                   | 1.2             | Prepara los datos de relación|
> |[pg_stat_statements](https://www.postgresql.org/docs/12/pgstatstatements.html)           | 1.7             | Realiza un seguimiento de las estadísticas de ejecución de todas las instrucciones SQL ejecutadas|
> |[pg_trgm](https://www.postgresql.org/docs/12/pgtrgm.html)                      | 1.4             | Medición de similitud de texto y búsqueda de índice basada en trigramas|
> |[pg_visibility](https://www.postgresql.org/docs/12/pgvisibility.html)                      | 1.2             | Examen del mapa de visibilidad (VM) y la información de visibilidad de nivel de página.|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.4             | Proporciona funcionalidad de auditoría|
> |[pgcrypto](https://www.postgresql.org/docs/12/pgcrypto.html)                     | 1.3             | Funciones de cifrado|
> |[pgrowlocks](https://www.postgresql.org/docs/12/pgrowlocks.html)                   | 1.2             | Muestra información de bloqueo de nivel de fila|
> |[pgstattuple](https://www.postgresql.org/docs/12/pgstattuple.html)                  | 1.5             | Muestra estadísticas de nivel de tupla|
> |[plpgsql](https://www.postgresql.org/docs/12/plpgsql.html)                      | 1.0             | Lenguaje de procedimientos de PL/pgSQL|
> |[postgis](https://www.postgis.net/)                      | 3.0.0           | Geometría, geografía y funciones y tipos espaciales de trama de PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/12/postgres-fdw.html)                 | 1.0             | Contenedor de datos externos para servidores PostgreSQL remotos|
> |[sslinfo](https://www.postgresql.org/docs/12/sslinfo.html)                    | 1.2             | Información sobre los certificados SSL.|
> |[tsm_system_rows](https://www.postgresql.org/docs/12/tsm-system-rows.html)                    | 1.0             |  Método TABLESAMPLE que acepta el número de filas como un límite.|
> |[tsm_system_time](https://www.postgresql.org/docs/12/tsm-system-time.html)                    | 1.0             |  Método TABLESAMPLE que acepta el tiempo en milisegundos como un límite.|
> |[unaccent](https://www.postgresql.org/docs/12/unaccent.html)                     | 1.1             | Diccionario de búsqueda de texto que quita los acentos|
> |[uuid-ossp](https://www.postgresql.org/docs/12/uuid-ossp.html)                    | 1.1             | Genera identificadores únicos universales (UUID)|

## <a name="postgres-11-extensions"></a>Extensiones de Postgres 11

Las extensiones siguientes están disponibles en los servidores flexibles de Azure Database for PostgreSQL que tienen la versión 11 de Postgres. 

> [!div class="mx-tableFixed"]
> | **Extensión**| **Versión de la extensión** | **Descripción** |
> |---|---|---|
> |[amcheck](https://www.postgresql.org/docs/11/amcheck.html)                    | 1.1             | Funciones de comprobación de la integridad de la relación.|
> |[bloom](https://www.postgresql.org/docs/11/bloom.html)                    | 1,0             | Método de acceso de bloom: índice basado en archivos de firma.|
> |[btree_gin](https://www.postgresql.org/docs/11/btree-gin.html)                    | 1.3             | Compatibilidad con la indexación de tipos de datos comunes en GIN|
> |[btree_gist](https://www.postgresql.org/docs/11/btree-gist.html)                   | 1.5             | Compatibilidad con la indexación de tipos de datos comunes en GiST|
> |[citext](https://www.postgresql.org/docs/11/citext.html)                       | 1.5             | Tipo de datos para cadenas de caracteres que no distinguen mayúsculas de minúsculas|
> |[cube](https://www.postgresql.org/docs/11/cube.html)                         | 1.4             | Tipo de datos para los cubos multidimensionales|
> |[dblink](https://www.postgresql.org/docs/11/dblink.html)                       | 1.2             | Se conecta a otras bases de datos de PostgreSQL desde una base de datos|
> |[dict_int](https://www.postgresql.org/docs/11/dict-int.html)                     | 1.0             | Plantilla de diccionario de búsqueda de texto para números enteros|
> |[dict_xsyn](https://www.postgresql.org/docs/11/dict-xsyn.html)                     | 1,0             | Plantilla de diccionario de búsqueda de texto para el procesamiento de sinónimos extendido.|
> |[earthdistance](https://www.postgresql.org/docs/11/earthdistance.html)                | 1.1             | Calcula distancias de círculo máximo en la superficie de la Tierra|
> |[fuzzystrmatch](https://www.postgresql.org/docs/11/fuzzystrmatch.html)                | 1.1             | Determina las similitudes y la distancia entre las cadenas|
> |[hstore](https://www.postgresql.org/docs/11/hstore.html)                       | 1.5             | Tipo de datos para almacenar conjuntos de pares (clave/valor)|
> |[intagg](https://www.postgresql.org/docs/11/intagg.html)                     | 1.1             | Agregador y enumerador de enteros. (Obsoleto)|
> |[intarray](https://www.postgresql.org/docs/11/intarray.html)                     | 1.2             | Funciones, operadores e índices compatibles con matrices 1D de números enteros|
> |[isn](https://www.postgresql.org/docs/11/isn.html)                          | 1.2             | Tipos de datos para los estándares internacionales de numeración de productos|
> |[ltree](https://www.postgresql.org/docs/11/ltree.html)                        | 1.1             | Tipo de datos para las estructuras de árbol jerárquicas|
> |[pageinspect](https://www.postgresql.org/docs/11/pageinspect.html)                        | 1.7             | Inspección del contenido de páginas de bases de datos a un nivel bajo.|
> |[pg_buffercache](https://www.postgresql.org/docs/11/pgbuffercache.html)               | 1.3             | Examina la caché del búfer compartido|
> |[pg_freespacemap](https://www.postgresql.org/docs/11/pgfreespacemap.html)               | 1.2             | Examen de la asignación de espacio libre (FSM).|
> |[pg_prewarm](https://www.postgresql.org/docs/11/pgprewarm.html)                   | 1.2             | Prepara los datos de relación|
> |[pg_stat_statements](https://www.postgresql.org/docs/11/pgstatstatements.html)           | 1.6             | Realiza un seguimiento de las estadísticas de ejecución de todas las instrucciones SQL ejecutadas|
> |[pg_trgm](https://www.postgresql.org/docs/11/pgtrgm.html)                      | 1.4             | Medición de similitud de texto y búsqueda de índice basada en trigramas|
> |[pg_visibility](https://www.postgresql.org/docs/11/pgvisibility.html)                      | 1.2             | Examen del mapa de visibilidad (VM) y la información de visibilidad de nivel de página.|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.3.1             | Proporciona funcionalidad de auditoría|
> |[pgcrypto](https://www.postgresql.org/docs/11/pgcrypto.html)                     | 1.3             | Funciones de cifrado|
> |[pgrowlocks](https://www.postgresql.org/docs/11/pgrowlocks.html)                   | 1.2             | Muestra información de bloqueo de nivel de fila|
> |[pgstattuple](https://www.postgresql.org/docs/11/pgstattuple.html)                  | 1.5             | Muestra estadísticas de nivel de tupla|
> |[plpgsql](https://www.postgresql.org/docs/11/plpgsql.html)                      | 1.0             | Lenguaje de procedimientos de PL/pgSQL|
> |[postgis](https://www.postgis.net/)                      | 2.5.1           | Geometría, geografía y funciones y tipos espaciales de trama de PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/11/postgres-fdw.html)                 | 1.0             | Contenedor de datos externos para servidores PostgreSQL remotos|
> |[sslinfo](https://www.postgresql.org/docs/11/sslinfo.html)                    | 1.2             | Información sobre los certificados SSL.|
> |[tablefunc](https://www.postgresql.org/docs/11/tablefunc.html)                    | 1.0             | Funciones que manipulan la totalidad del contenido de las tablas, incluidas tablas de referencias cruzadas|
> |[tsm_system_rows](https://www.postgresql.org/docs/11/tsm-system-rows.html)                    | 1,0             |  Método TABLESAMPLE que acepta el número de filas como un límite.|
> |[tsm_system_time](https://www.postgresql.org/docs/11/tsm-system-time.html)                    | 1,0             |  Método TABLESAMPLE que acepta el tiempo en milisegundos como un límite.|
> |[unaccent](https://www.postgresql.org/docs/11/unaccent.html)                     | 1.1             | Diccionario de búsqueda de texto que quita los acentos|
> |[uuid-ossp](https://www.postgresql.org/docs/11/uuid-ossp.html)                    | 1.1             | Genera identificadores únicos universales (UUID)|


## <a name="pg_prewarm"></a>pg_prewarm

La extensión pg_prewarm carga los datos relacionales en la memoria caché. El precalentamiento de las memorias caché significa que las consultas tengan mejores tiempos de respuesta en su primera ejecución después de un reinicio. La función de precalentamiento automático no está disponible actualmente en Azure Database for PostgreSQL: Servidor flexible.


## <a name="next-steps"></a>Pasos siguientes

Si no ve una extensión que le gustaría utilizar, háganoslo saber. Vote por las solicitudes existentes o cree nuevos comentarios y solicitudes en nuestro [foro de comentarios](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).