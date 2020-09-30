---
title: Uso de sys_schema en Azure Database for MySQL
description: Aprenda a usar sys_schema para detectar problemas de rendimiento y realizar el mantenimiento de bases de datos en Azure Database for MySQL.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: troubleshooting
ms.date: 3/30/2020
ms.openlocfilehash: 62a34a2dba459c6f65729cd5c6804378ee7f8b52
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90902773"
---
# <a name="how-to-use-sys_schema-for-performance-tuning-and-database-maintenance-in-azure-database-for-mysql"></a>Uso de sys_schema para operaciones de optimización del rendimiento y mantenimiento de bases de datos en Azure Database for MySQL

La característica performance_schema de MySQL, disponible por primera vez en MySQL 5.5, proporciona instrumentación para muchos recursos importantes del servidor como la asignación de memoria, programas almacenados, bloqueo de metadatos, etc. No obstante, performance_schema contiene más de 80 tablas y obtener la información necesaria a menudo requiere unir las tablas de performance_schema, así como las tablas procedentes de information_schema. Tomando como base performance_schema e information_schema, la característica sys_schema proporciona un conjunto eficaz de [vistas fáciles de usar](https://dev.mysql.com/doc/refman/5.7/en/sys-schema-views.html) en una base de datos de solo lectura y está completamente habilitada en Azure Database for MySQL versión 5.7.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/sys-schema-views.png" alt-text="vistas de sys_schema":::

Hay 52 vistas en sys_schema y cada vista tiene uno de los siguientes prefijos:

- Host_summary o IO: latencias relacionadas con E/S.
- InnoDB: estado y bloqueos del búfer InnoDB.
- Memoria: uso de la memoria por parte del host y de los usuarios.
- Esquema: información relacionada con los esquemas, como el incremento automático, los índices, etc.
- Statement: información sobre las instrucciones SQL. Puede tratarse de una instrucción que dio como resultado un recorrido de tabla completo o un tiempo de consulta largo.
- User: los recursos que consumen y agrupan los usuarios. Algunos ejemplos son: operaciones de E/S de archivo, conexiones y memoria.
- Wait: eventos de espera agrupados por host o usuario.

Echemos un vistazo a algunos patrones de uso habituales de sys_schema. Para empezar, vamos a agrupar los patrones de uso en dos categorías: **Ajuste de rendimiento** y **Mantenimiento de la base de datos**.

## <a name="performance-tuning"></a>Optimización del rendimiento

### <a name="sysuser_summary_by_file_io"></a>*sys.user_summary_by_file_io*

E/S es la operación más costosa en la base de datos. Podemos averiguar el promedio de latencia de E/S consultando la vista *sys.user_summary_by_file_io*. Con el valor predeterminado de 125 GB de almacenamiento aprovisionado, la latencia de E/S es aproximadamente de 15 segundos.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/io-latency-125GB.png" alt-text="Latencia de E/S: 125 GB":::

Dado que Azure Database for MySQL escala E/S en relación al almacenamiento, después de aumentar mi almacenamiento aprovisionado a 1 TB, la latencia de E/S se reduce a 571 ms.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/io-latency-1TB.png" alt-text="Latencia de E/S: 1 TB":::

### <a name="sysschema_tables_with_full_table_scans"></a>*sys.schema_tables_with_full_table_scans*

A pesar de efectuar un planeamiento cuidadoso, puede que muchas consultas aún resulten en recorridos de tabla completos. Para obtener información adicional acerca de los tipos de índices y cómo optimizarlos, puede consultar este artículo: [Solución de problemas relacionados con el rendimiento de consultas](./howto-troubleshoot-query-performance.md). Los recorridos de tabla completos requieren un uso intensivo de los recursos y reducen el rendimiento de la base de datos. La forma más rápida de buscar tablas con recorridos de tabla completos es consultar la vista *sys.schema_tables_with_full_table_scans*.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/full-table-scans.png" alt-text="recorridos de tabla completos":::

### <a name="sysuser_summary_by_statement_type"></a>*sys.user_summary_by_statement_type*

Para solucionar los problemas de rendimiento de bases de datos, puede resultar útil identificar los eventos que se producen dentro de la base de datos y mediante la vista *sys.user_summary_by_statement_type* puede lograrlo.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/summary-by-statement.png" alt-text="resumen por instrucción":::

En este ejemplo Azure Database for MySQL empleó 53 minutos en vaciar el registro de consultas slog 44579 veces. Eso es mucho tiempo y muchas E/S. Puede reducir esta actividad si deshabilita el registro de consultas lentas o reduce la frecuencia de este registro en Azure Portal.

## <a name="database-maintenance"></a>Mantenimiento de base de datos

### <a name="sysinnodb_buffer_stats_by_table"></a>*sys.innodb_buffer_stats_by_table*

[!IMPORTANT]
> La consulta de esta vista puede afectar al rendimiento. Se recomienda llevar a cabo esta solución de problemas durante el horario comercial fuera de horas punta.

El grupo de búferes InnoDB reside en la memoria y es el principal mecanismo de memoria caché entre el sistema de administración de bases de datos y el almacenamiento. El tamaño del grupo de búferes InnoDB está vinculado al nivel de rendimiento y no se puede cambiar a menos que se elija una SKU de producto diferente. Al igual que con la memoria del sistema operativo, se intercambiaron páginas antiguas para dejar espacio a los datos más recientes. Para averiguar qué tablas consumen la mayor parte de la memoria del grupo de búferes InnoDB, puede consultar la vista *sys.innodb_buffer_stats_by_table*.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/innodb-buffer-status.png" alt-text="Estado de los búferes InnoDB":::

En el gráfico anterior, se puede ver que aparte de las tablas del sistema y las vistas, cada tabla de la base de datos mysqldatabase033, que hospeda uno de mis sitios de WordPress, ocupa 16 KB, o 1 página, de datos en memoria.

### <a name="sysschema_unused_indexes--sysschema_redundant_indexes"></a>*Sys.schema_unused_indexes* & *sys.schema_redundant_indexes*

Los índices son unas herramientas estupendas para mejorar el rendimiento de lectura, pero suponen costos adicionales por las inserciones y el almacenamiento. *Sys.schema_unused_indexes* and *sys.schema_redundant_indexes* proporcionan información sobre los índices sin usar o los duplicados.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/unused-indexes.png" alt-text="índices sin usar":::

:::image type="content" source="./media/howto-troubleshoot-sys-schema/redundant-indexes.png" alt-text="índices redundantes":::

## <a name="conclusion"></a>Conclusión

En resumen, sys_schema es una herramienta excelente para la optimización de rendimiento y el mantenimiento de base de datos. Asegúrese de sacarle el máximo partido a esta característica en Azure Database for MySQL. 

## <a name="next-steps"></a>Pasos siguientes
- Para buscar respuestas de otros usuarios a sus preguntas o publicar una nueva pregunta o respuesta, visite la [Página de pregunta y respuesta de Microsoft](https://docs.microsoft.com/answers/topics/azure-database-mysql.html) o [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-database-mysql).
