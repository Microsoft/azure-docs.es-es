---
title: Optimización del rendimiento con vistas materializadas
description: Recomendaciones y consideraciones que debe conocer al usar vistas materializadas para mejorar el rendimiento de las consultas.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: xiaoyul
ms.reviewer: nibruno; jrasnick
ms.openlocfilehash: d476bef6faa19defad1d2e1ef1a90f7e5d83def5
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495699"
---
# <a name="performance-tuning-with-materialized-views"></a>Optimización del rendimiento con vistas materializadas

En un grupo de SQL de Synapse, las vistas materializadas proporcionan un método con poco mantenimiento para que las consultas analíticas complejas puedan tener un rendimiento rápido sin cambiar la consulta. En este artículo se describe la guía general sobre el uso de vistas materializadas.

## <a name="materialized-views-vs-standard-views"></a>Vistas materializadas frente a vistas estándar

El grupo de SQL admite las vistas estándar y materializadas.  Ambas son tablas virtuales creadas con expresiones SELECT y se presentan a las consultas como tablas lógicas.  Las vistas revelan la complejidad del cálculo común de datos y agregan una capa de abstracción a los cambios en el cálculo, por lo que no es necesario volver a escribir las consultas.  

Una vista estándar calcula sus datos cada vez que se utiliza la vista.  No hay datos almacenados en disco. Normalmente, los usuarios usan las vistas estándar como una herramienta que ayuda a organizar los objetos lógicos y las consultas de una base de datos.  Para utilizar una vista estándar, una consulta debe hacer referencia directa a ella.

Una vista materializada calcula previamente, almacena y mantiene sus datos en el grupo de SQL como una tabla.  No es necesario volver a calcular cada vez que se usa una vista materializada.  Este es el motivo por el que las consultas que utilizan todos los datos o un subconjunto de ellos en vistas materializadas pueden lograr un rendimiento más rápido.  Además, las consultas pueden usar una vista materializada sin hacer referencia directa a ella, por lo que no es necesario cambiar el código de la aplicación.  

La mayoría de los requisitos de una vista estándar se aplican a una vista materializada. Para más información sobre la sintaxis de la vista materializada y otros requisitos, consulte [CREATE MATERIALIZED VIEW AS SELECT](/sql/t-sql/statements/create-materialized-view-as-select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

| De comparación                     | Ver                                         | Vista materializada
|:-------------------------------|:---------------------------------------------|:--------------------------------------------------------------|
|Ver definición                 | Almacenada en Azure Data Warehouse.              | Almacenada en Azure Data Warehouse.
|Contenido de la vista                    | Se genera cada vez que se utiliza la vista.   | Se procesa previamente y se almacena en Azure Data Warehouse durante la creación de la vista. Se actualiza a medida que se agregan datos a las tablas subyacentes.
|Actualización de datos                    | Siempre actualizados                               | Siempre actualizados
|Velocidad de recuperación de los datos de la vista a partir de consultas complejas     | Lenta                                         | Fast (rápido)  
|Almacenamiento adicional                   | No                                           | Sí
|Sintaxis                          | CREATE VIEW                                  | CREATE MATERIALIZED VIEW AS SELECT

## <a name="benefits-of-materialized-views"></a>Ventajas de las vistas materializadas

Una vista materializada diseñada correctamente proporciona las siguientes ventajas:

- Menor tiempo de ejecución de las consultas complejas con instrucciones JOIN y funciones de agregado. Cuanto más compleja sea la consulta, mayor será la posibilidad de reducir el tiempo de ejecución. La mayor ventaja se obtiene cuando el costo de cálculo de una consulta es elevado y el conjunto de datos resultante es pequeño.  

- El optimizador en el grupo de SQL puede usar automáticamente las vistas materializadas implementadas para mejorar los planes de ejecución de consultas.  Este proceso es transparente para los usuarios y proporciona un rendimiento de las consultas más rápido y no requiere que las consultas hagan referencia directa a las vistas materializadas.

- Bajos requisitos de mantenimiento en las vistas.  Una vista materializada almacena los datos en dos lugares, un índice de almacén de columnas agrupado para los datos iniciales en el momento de creación de la vista y un almacén incremental para los cambios de datos incrementales.  Todos los cambios de datos de las tablas base se agregan automáticamente al almacén incremental de forma sincrónica.  Un proceso en segundo plano (motor de tupla) mueve periódicamente los datos del almacén incremental al índice de almacén de columnas de la vista.  Este diseño permite que las consultas de las vistas materializadas devuelvan los mismos datos que las consultas directas de las tablas base.
- Los datos de una vista materializada se pueden distribuir de forma diferente de las tablas base.  
- Los datos de las vistas materializadas tienen las mismas ventajas de alta disponibilidad y resistencia que los datos de las tablas normales.  

En comparación con otros proveedores de almacenamiento de datos, las vistas materializadas implementadas en un grupo de SQL también proporcionan las siguientes ventajas adicionales:

- Actualización automática y sincrónica de los datos con los cambios de los datos de las tablas base. No se requiere ninguna acción del usuario.
- Amplia compatibilidad con funciones de agregado. Consulte [CREATE MATERIALIZED VIEW AS SELECT (Transact-SQL)](/sql/t-sql/statements/create-materialized-view-as-select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).
- Compatibilidad con la recomendación de vista materializada específica de la consulta.  Consulte [EXPLAIN (Transact-SQL)](/sql/t-sql/queries/explain-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

## <a name="common-scenarios"></a>Escenarios frecuentes  

Las vistas materializadas se suelen usar en los siguientes escenarios:

**Necesidad de mejorar el rendimiento de las consultas analíticas complejas en datos de gran tamaño**

Las consultas analíticas complejas suelen usar más funciones de agregación y combinaciones de tablas, lo que provoca operaciones de proceso más intensivo, como las combinaciones y las uniones en la ejecución de las consultas.  Este es el motivo por el que las consultas tardan más tiempo en completarse, en particular en tablas grandes.  

Los usuarios pueden crear vistas materializadas para los datos devueltos por los cálculos comunes de las consultas, por lo que no se necesita ningún recálculo cuando las consultas necesitan los datos, lo que permite reducir el costo de proceso y obtener una respuesta de las consultas más rápida.

**Necesidad de un rendimiento más rápido sin cambios o con cambios mínimos en las consultas**

Normalmente, los cambios de esquema y de consulta en los almacenamientos de datos se conservan como mínimo para admitir las operaciones de ETL normales y los informes.  Los usuarios pueden usar vistas materializadas para optimizar el rendimiento de las consultas si la ganancia en el rendimiento de las consultas compensa el costo en el que incurren las vistas.

En comparación con otras opciones de optimización, como el escalado y la administración de estadísticas, crear y mantener una vista materializada es un cambio de producción con mucho menor impacto, y su posible ganancia de rendimiento también es mayor.

- La creación y el mantenimiento de las vistas materializadas no afectan a las consultas que se ejecutan en las tablas base.
- El optimizador de consultas puede usar automáticamente las vistas materializadas implementadas sin referencia directa a las vistas en una consulta. Esta funcionalidad reduce la necesidad de cambiar las consultas en el ajuste del rendimiento.

**Necesidad de una estrategia de distribución de datos diferente para un rendimiento más rápido de las consultas**

Azure Data Warehouse es un sistema de procesamiento masivo en paralelo (MPP) y distribuido.   Los datos de una tabla del almacenamiento de datos se distribuyen entre 60 nodos mediante una de estas tres [estrategias de distribución](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) (hash, round_robin o replicada).  

La distribución de datos se especifica en el momento de creación de la tabla y permanece invariable hasta que se elimina la tabla. Puesto que la vista materializada es una tabla virtual en disco, posibilita las distribuciones de datos hash y round_robin.  Los usuarios pueden elegir una distribución de datos diferente a la de las tablas base, pero que sea óptima para el rendimiento de las consultas que utilizan las vistas con frecuencia.  

## <a name="design-guidance"></a>Guía de diseño

Esta es la guía general sobre el uso de vistas materializadas para mejorar el rendimiento de las consultas:

**Diseño en función de la carga de trabajo**

Antes de empezar a crear vistas materializadas, es importante tener una comprensión profunda de la carga de trabajo en cuanto a los patrones de consulta, la importancia, la frecuencia y el tamaño de los datos resultantes.  

Los usuarios pueden ejecutar la instrucción SQL EXPLAIN WITH_RECOMMENDATIONS para las vistas materializadas recomendadas por el optimizador de consultas.  Dado que estas recomendaciones son específicas de la consulta, una vista materializada que beneficia a una sola consulta puede no ser óptima para otras consultas de la misma carga de trabajo.  

Evalúe estas recomendaciones teniendo en cuenta las necesidades de la carga de trabajo.  Las vistas materializadas ideales son las que benefician el rendimiento de la carga de trabajo.  

**Tenga en cuenta el equilibrio entre consultas más rápidas y el costo**

Para cada vista materializada, hay un costo de almacenamiento de datos y un costo por el mantenimiento de la vista.  A medida que cambian los datos en las tablas base, el tamaño de la vista materializada aumenta y su estructura física también cambia.  

Para evitar la degradación del rendimiento de las consultas, el motor de almacenamiento de datos mantiene por separado cada vista materializada, lo que incluye el movimiento de filas del almacén delta a los segmentos de índice de almacén de columnas y la consolidación de los cambios de datos.  

La carga de trabajo de mantenimiento aumenta cuando crece el número de vistas materializadas y de cambios de la tabla base.   Los usuarios deben comprobar si el costo producido por todas las vistas materializadas se puede compensar con la ganancia de rendimiento de las consultas.  

Puede ejecutar esta consulta para obtener la lista de vistas materializadas de una base de datos:

```sql
SELECT V.name as materialized_view, V.object_id
FROM sys.views V
JOIN sys.indexes I ON V.object_id= I.object_id AND I.index_id < 2;
```

Opciones para reducir el número de vistas materializadas:

- Identifique los conjuntos de datos comunes que se usan con frecuencia en las consultas complejas de la carga de trabajo.  Cree vistas materializadas para almacenar esos conjuntos de datos, de modo que el optimizador pueda usarlos como bloques de creación al crear planes de ejecución.  

- Elimine las vistas materializadas que tienen poco uso o que ya no son necesarias.  Una vista materializada deshabilitada no se mantiene, pero sigue incurriendo en el costo de almacenamiento.  

- Combine las vistas materializadas creadas en las mismas o similares tablas base, incluso si sus datos no se superponen.  La combinación de las vistas materializadas podría generar una vista de tamaño mayor que la suma de las vistas independientes, pero el costo de mantenimiento de las vistas debería reducirse.  Por ejemplo:

```sql
-- Query 1 would benefit from having a materialized view created with this SELECT statement
SELECT A, SUM(B)
FROM T
GROUP BY A

-- Query 2 would benefit from having a materialized view created with this SELECT statement
SELECT C, SUM(D)
FROM T
GROUP BY C

-- You could create a single mateiralized view of this form
SELECT A, C, SUM(B), SUM(D)
FROM T
GROUP BY A, C

```

**No todas las optimizaciones de rendimiento requieren cambios en las consultas**

El optimizador del almacenamiento de datos puede utilizar automáticamente las vistas materializadas implementadas para mejorar el rendimiento de las consultas.  Esta compatibilidad se aplica de forma transparente a las consultas que no hacen referencia a las vistas y a las consultas que usan agregados no admitidos en la creación de vistas materializadas.  No se necesita ningún cambio en la consulta. Puede comprobar el plan de ejecución estimado de una consulta para confirmar si se usa una vista materializada.  

**Supervisión de vistas materializadas**

Una vista materializada se almacena en el almacenamiento de datos igual que una tabla con un índice de almacén de columnas agrupado (CCI).  La lectura de los datos de una vista materializada incluye el examen del índice y la aplicación de cambios desde el almacén incremental.  Cuando el número de filas del almacén incremental es demasiado alto, la resolución de una consulta a partir de una vista materializada puede tardar más que la consulta directa de las tablas base.  Para evitar la degradación del rendimiento de las consultas, se recomienda ejecutar [DBCC PDW_SHOWMATERIALIZEDVIEWOVERHEAD](/sql/t-sql/database-console-commands/dbcc-pdw-showmaterializedviewoverhead-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) para supervisar la relación de sobrecarga de la vista (filas totales/filas de la vista base).  Si la relación de sobrecarga es demasiado alta, considere la posibilidad de volver a generar la vista materializada para que todas las filas del almacén incremental se muevan al índice de almacén de columnas.  

**Vista materializada y almacenamiento en caché de conjuntos de resultados**

Estas dos características se introducen en el grupo de SQL aproximadamente al mismo tiempo para el ajuste del rendimiento de las consultas. El almacenamiento en caché de conjuntos de resultados se usa para obtener una alta simultaneidad y tiempos de respuesta menores a consultas repetitivas sobre datos estáticos.  

Para usar el resultado almacenado en caché, la forma de la consulta de solicitud de almacenamiento en caché debe coincidir con la consulta que generó la memoria caché.  Además de esto, el resultado almacenado en caché debe ser de aplicación para toda la consulta.  Las vistas materializadas permiten cambios en los datos de las tablas base.  Los datos de las vistas materializadas se pueden aplicar a una parte de una consulta.  Esta compatibilidad permite usar las mismas vistas materializadas en consultas diferentes que comparten algún cálculo para obtener un rendimiento más rápido.

## <a name="example"></a>Ejemplo

En este ejemplo se usa una consulta de tipo TPCDS que encuentra a los clientes que gastan más dinero en el catálogo que en las tiendas. También identifica los clientes preferidos y su país o región de origen.   La consulta implica seleccionar los primeros 100 registros del resultado de la instrucción UNION de tres instrucciones sub-SELECT que implican SUM() y GROUP BY.

```sql
WITH year_total AS (
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ss_ext_list_price-ss_ext_wholesale_cost-ss_ext_discount_amt+ss_ext_sales_price, 0)/2) year_total
       ,'s' sale_type
FROM customer
     ,store_sales
     ,date_dim
WHERE c_customer_sk = ss_customer_sk
   AND ss_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
UNION ALL
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(cs_ext_list_price-cs_ext_wholesale_cost-cs_ext_discount_amt+cs_ext_sales_price, 0)/2) year_total
       ,'c' sale_type
FROM customer
     ,catalog_sales
     ,date_dim
WHERE c_customer_sk = cs_bill_customer_sk
   AND cs_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
UNION ALL
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ws_ext_list_price-ws_ext_wholesale_cost-ws_ext_discount_amt+ws_ext_sales_price, 0)/2) year_total
       ,'w' sale_type
FROM customer
     ,web_sales
     ,date_dim
WHERE c_customer_sk = ws_bill_customer_sk
   AND ws_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
         )
  SELECT TOP 100
                  t_s_secyear.customer_id
                 ,t_s_secyear.customer_first_name
                 ,t_s_secyear.customer_last_name
                 ,t_s_secyear.customer_birth_country
FROM year_total t_s_firstyear
     ,year_total t_s_secyear
     ,year_total t_c_firstyear
     ,year_total t_c_secyear
     ,year_total t_w_firstyear
     ,year_total t_w_secyear
WHERE t_s_secyear.customer_id = t_s_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_c_secyear.customer_id
   AND t_s_firstyear.customer_id = t_c_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_w_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_w_secyear.customer_id
   AND t_s_firstyear.sale_type = 's'
   AND t_c_firstyear.sale_type = 'c'
   AND t_w_firstyear.sale_type = 'w'
   AND t_s_secyear.sale_type = 's'
   AND t_c_secyear.sale_type = 'c'
   AND t_w_secyear.sale_type = 'w'
   AND t_s_firstyear.dyear+0 =  1999
   AND t_s_secyear.dyear+0 = 1999+1
   AND t_c_firstyear.dyear+0 =  1999
   AND t_c_secyear.dyear+0 =  1999+1
   AND t_w_firstyear.dyear+0 = 1999
   AND t_w_secyear.dyear+0 = 1999+1
   AND t_s_firstyear.year_total > 0
   AND t_c_firstyear.year_total > 0
   AND t_w_firstyear.year_total > 0
   AND CASE WHEN t_c_firstyear.year_total > 0 THEN t_c_secyear.year_total / t_c_firstyear.year_total ELSE NULL END
           > CASE WHEN t_s_firstyear.year_total > 0 THEN t_s_secyear.year_total / t_s_firstyear.year_total ELSE NULL END
   AND CASE WHEN t_c_firstyear.year_total > 0 THEN t_c_secyear.year_total / t_c_firstyear.year_total ELSE NULL END
           > CASE WHEN t_w_firstyear.year_total > 0 THEN t_w_secyear.year_total / t_w_firstyear.year_total ELSE NULL END
ORDER BY t_s_secyear.customer_id
         ,t_s_secyear.customer_first_name
         ,t_s_secyear.customer_last_name
         ,t_s_secyear.customer_birth_country
OPTION ( LABEL = 'Query04-af359846-253-3');
```

Compruebe el plan de ejecución estimado de la consulta.  Hay 18 combinaciones y 17 operaciones de unión que tardan más tiempo en ejecutarse. Ahora vamos a crear una vista materializada para cada una de las tres instrucciones sub-SELECT.

```sql
CREATE materialized view nbViewSS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ss_ext_list_price-ss_ext_wholesale_cost-ss_ext_discount_amt+ss_ext_sales_price, 0)/2) year_total
          , count_big(*) AS cb
FROM dbo.customer
     ,dbo.store_sales
     ,dbo.date_dim
WHERE c_customer_sk = ss_customer_sk
   AND ss_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
GO
CREATE materialized view nbViewCS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(cs_ext_list_price-cs_ext_wholesale_cost-cs_ext_discount_amt+cs_ext_sales_price, 0)/2) year_total
          , count_big(*) as cb
FROM dbo.customer
     ,dbo.catalog_sales
     ,dbo.date_dim
WHERE c_customer_sk = cs_bill_customer_sk
   AND cs_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year

GO
CREATE materialized view nbViewWS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ws_ext_list_price-ws_ext_wholesale_cost-ws_ext_discount_amt+ws_ext_sales_price, 0)/2) year_total
          , count_big(*) AS cb
FROM dbo.customer
     ,dbo.web_sales
     ,dbo.date_dim
WHERE c_customer_sk = ws_bill_customer_sk
   AND ws_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year

```

Vuelva a comprobar el plan de ejecución de la consulta original.  Ahora, el número de uniones cambia de 17 a 5 y ya no hay ninguna combinación.  Haga clic en el icono de la operación de filtro en el plan. Su lista de salida muestra que los datos se leen de las vistas materializadas en lugar de las tablas base.  

 ![Lista de salida del plan con vistas materializadas](./media/develop-materialized-view-performance-tuning/output-list.png)

Con las vistas materializadas, la misma consulta se ejecuta mucho más rápido sin ningún cambio de código.  

## <a name="next-steps"></a>Pasos siguientes

Para obtener más sugerencias sobre desarrollo, consulte la [información general sobre desarrollo de SQL de Synapse](develop-overview.md).
 