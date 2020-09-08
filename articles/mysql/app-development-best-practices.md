---
title: 'Procedimientos recomendados de desarrollo de aplicaciones: Azure Database for MySQL'
description: Más información sobre los procedimientos recomendados al compilar una aplicación con Azure Database for MySQL.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: conceptual
ms.date: 08/11/2020
ms.openlocfilehash: 93bd6972a89065832a20fbd66949cde5b7510534
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88794193"
---
# <a name="best-practices-for-building-an-application-with-azure-database-for-mysql"></a>Procedimientos recomendados para compilar una aplicación con Azure Database for MySQL 

Estos son algunos procedimientos recomendados que le ayudarán a compilar una aplicación preparada para la nube mediante Azure Database for MySQL. Estos procedimientos recomendados pueden reducir el tiempo de desarrollo de la aplicación. 

## <a name="configuration-of-application-and-database-resources"></a>Configuración de recursos de aplicación y de base de datos

### <a name="keep-the-application-and-database-in-the-same-region"></a>Mantenga la aplicación y la base de datos en la misma región.
Al implementar la aplicación en Azure, asegúrese de que todas las dependencias están en la misma región. La propagación de instancias entre regiones o zonas de disponibilidad crea latencia de red, lo que puede afectar al rendimiento general de la aplicación. 

### <a name="keep-your-mysql-server-secure"></a>Mantenimiento de la seguridad del servidor MySQL
Configure el servidor MySQL como [seguro](https://docs.microsoft.com/azure/mysql/concepts-security) y no se pueda acceder a él de forma pública. Use cualquiera de estas opciones para proteger el servidor: 
- [Reglas de firewall](https://docs.microsoft.com/azure/mysql/concepts-firewall-rules)
- [Redes virtuales](https://docs.microsoft.com/azure/mysql/concepts-data-access-and-security-vnet) 
- [Azure Private Link](https://docs.microsoft.com/azure/mysql/concepts-data-access-security-private-link)

Por seguridad, siempre debe conectarse al servidor MySQL a través de SSL y configurar el servidor MySQL y la aplicación para que usen TLS 1.2. Vea [Conectividad SSL/TLS](https://docs.microsoft.com/azure/mysql/concepts-ssl-connection-security). 

### <a name="tune-your-server-parameters"></a>Ajuste de los parámetros del servidor
En el caso de cargas de trabajo con muchas operaciones de lectura, el ajuste de los parámetros `tmp_table_size` y `max_heap_table_size` puede ayudar a optimizar el rendimiento. Para calcular los valores necesarios para estas variables, examine los valores de memoria total por conexión y la memoria base. La suma de los parámetros de memoria por conexión, excluido `tmp_table_size`, junto con la memoria base, da como resultado la memoria total del servidor.

Para calcular el mayor tamaño posible de `tmp_table_size` y `max_heap_table_size`, use la siguiente fórmula:

```(total memory - (base memory + (sum of per-connection memory * # of connections)) / # of connections```

>[!NOTE]
> La memoria total indica la cantidad total de memoria que el servidor tiene en los núcleos virtuales aprovisionados.  Por ejemplo, en un servidor de Azure Database for MySQL con dos núcleos virtuales de uso general, la memoria total será 5 GB * 2. Puede encontrar más información sobre la memoria en cada nivel en la documentación del [plan de tarifa](https://docs.microsoft.com/azure/mysql/concepts-pricing-tiers).
>
> La memoria base indica las variables de memoria, como `query_cache_size` y `innodb_buffer_pool_size`, que MySQL inicializará y asignará al inicio del servidor. La memoria por conexión, como `sort_buffer_size` y `join_buffer_size`, es la memoria que se asigna solo cuando una consulta la necesita.

### <a name="create-non-admin-users"></a>Creación de usuarios no administradores 
[Cree usuarios que no sean administradores](https://docs.microsoft.com/azure/mysql/howto-create-users) para cada una de las bases de datos. Normalmente, los nombres de usuario se identifican como nombres de base de datos.

### <a name="reset-your-password"></a>Restablecimiento de la contraseña
Puede [restablecer la contraseña](https://docs.microsoft.com/azure/mysql/howto-create-manage-server-portal#update-admin-password) del servidor MySQL mediante Azure Portal. 

El restablecimiento de la contraseña del servidor de una base de datos de producción puede desactivar la aplicación. Una buena práctica es restablecer la contraseña de cualquier carga de trabajo de producción en horas de poca actividad para reducir los efectos en los usuarios de la aplicación.

## <a name="performance-and-resiliency"></a>Rendimiento y resistencia 
Estas son algunas herramientas y prácticas que puede usar para depurar problemas de rendimiento con la aplicación.

### <a name="enable-slow-query-logs-to-identify-performance-issues"></a>Habilitación de los registros de consultas lentas para identificar problemas de rendimiento
Puede habilitar los [registros de consultas lentas](https://docs.microsoft.com/azure/mysql/concepts-server-logs) y los [registros de auditoría](https://docs.microsoft.com/azure/mysql/concepts-audit-logs) en el servidor. El análisis de consultas lentas puede ayudar a identificar cuellos de botella que afectan al rendimiento a fin de solucionar el problema. 

Los registros de auditoría también están disponibles mediante los registros de Azure Diagnostics en los registros de Azure Monitor, Event Hubs y las cuentas de almacenamiento. Consulte [Solución de problemas del rendimiento de las consultas](https://docs.microsoft.com/azure/mysql/howto-troubleshoot-query-performance).

### <a name="use-connection-pooling"></a>Use agrupar conexiones
La administración de conexiones de base de datos puede tener un impacto significativo en el rendimiento de la aplicación en su conjunto. Para optimizar el rendimiento, debe reducir el número de veces que se establecen las conexiones y el tiempo para establecerlas en las rutas de acceso al código principal. Use [Agrupación de conexiones](https://docs.microsoft.com/azure/mysql/concepts-connectivity#access-databases-by-using-connection-pooling-recommended) para conectarse a Azure Database for MySQL, a fin de mejorar la resistencia y el rendimiento. 

Puede usar el agrupador de conexiones [ProxySQL](https://proxysql.com/) para administrar de forma eficaz las conexiones. Con un agrupador de conexiones, puede reducir las conexiones inactivas y reutilizar las conexiones existentes, lo que ayuda a evitar problemas. Consulte [Configuración de ProxySQL](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/connecting-efficiently-to-azure-database-for-mysql-with-proxysql/ba-p/1279842) para más información. 

### <a name="retry-logic-to-handle-transient-errors"></a>Lógica de reintento para controlar errores transitorios
La aplicación podría experimentar [errores transitorios](https://docs.microsoft.com/azure/mysql/concepts-connectivity#handling-transient-errors) en los que las conexiones a la base de datos se rechazan o se pierden de manera intermitente. En tales situaciones, el servidor se pone en funcionamiento al cabo de uno o dos reintentos en 5 o 10 segundos. 

Una buena práctica es esperar 5 segundos antes del primer reintento. Después, aumente gradualmente la espera entre cada reintento, hasta 60 segundos. Para investigar el problema más a fondo, limite el número máximo de reintentos en cuyo momento la aplicación considera que se produjo un error en la operación. Consulte [Solución de problemas de conexión](https://docs.microsoft.com/azure/mysql/howto-troubleshoot-common-connection-issues) para más información. 

### <a name="enable-read-replication-to-mitigate-failovers"></a>Habilitar la replicación de lectura para mitigar las conmutaciones por error
Se puede usar la [Replicación de datos de entrada](https://docs.microsoft.com/azure/mysql/howto-data-in-replication) en los escenarios de conmutación por error. Cuando se usan réplicas de lectura, no se produce una conmutación por error automatizada entre servidores maestros y réplicas. 

Observará un retraso entre el servidor maestro y la réplica, ya que la replicación es asincrónica. El retraso de red puede verse afectado por muchos factores, por ejemplo, el tamaño de la carga de trabajo que se ejecuta en el servidor maestro y la latencia entre los centros de datos. En la mayoría de los casos, el retraso de la réplica oscila entre unos segundos y un par de minutos.

## <a name="database-deployment"></a>Implementación de bases de datos 

### <a name="configure-an-azure-database-for-mysql-task-in-your-cicd-deployment-pipeline"></a>Configuración de una tarea de Azure Database for MySQL en la canalización de implementación de CI/CD
En ocasiones, es necesario implementar cambios en la base de datos. En tales casos, puede usar la integración continua (CI) y la entrega continua (CD) mediante [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) y emplear una tarea para que el [servidor MySQL](https://docs.microsoft.com/azure/devops/pipelines/tasks/deploy/azure-mysql-deployment?view=azure-devops) actualice la base de datos mediante la ejecución de un script personalizado en ella.

### <a name="use-an-effective-process-for-manual-database-deployment"></a>Uso de un proceso eficaz para la implementación manual de bases de datos 
Durante la implementación manual de bases de datos, siga estos pasos para minimizar el tiempo de inactividad o reducir el riesgo de implementaciones con errores: 

1. Cree una copia de una base de datos de producción en una nueva base de datos mediante [mysqldump](https://dev.mysql.com/doc/refman/8.0/en/mysqldump.html) o [MySQL Workbench](https://dev.mysql.com/doc/workbench/en/wb-admin-export-import-management.html). 
2. Actualice la nueva base de datos con los nuevos cambios de esquema o las actualizaciones necesarias de la base de datos. 
3. Ponga la base de datos de producción en estado de solo lectura. No debe tener operaciones de escritura en la base de datos de producción hasta que se complete la implementación. 
4. Pruebe la aplicación con la base de datos recién actualizada del paso 1.
5. Implemente los cambios de la aplicación y asegúrese de que la aplicación usa ahora la nueva base de datos que tiene las actualizaciones más recientes. 
6. Mantenga la antigua base de datos de producción para poder revertir los cambios. Después, puede valorar eliminar la base de datos de producción antigua o exportarla en Azure Storage, si es necesario. 

>[!NOTE]
>Si la aplicación es como una aplicación de comercio electrónico y no puede ponerla en estado de solo lectura, implemente los cambios directamente en la base de datos de producción después de realizar una copia de seguridad. Estos cambios deben realizarse durante las horas de poca actividad y poco tráfico a la aplicación, para que tengan el menor efecto posible, ya que algunos usuarios pueden experimentar solicitudes con error. 
>
>Asegúrese de que el código de aplicación también controla las solicitudes con error.

### <a name="use-mysql-native-metrics-to-see-if-your-workload-is-exceeding-in-memory-temporary-table-sizes"></a>Uso de las métricas nativas de MySQL para ver si la carga de trabajo supera los tamaños de tablas temporales en memoria
Cuando hay cargas de trabajo con muchas operaciones de lectura, las consultas que se ejecutan en el servidor MySQL pueden superar los tamaños de tablas temporales en memoria. Una carga de trabajo de este tipo puede hacer que el servidor pase a escribir tablas temporales en el disco, lo que afecta al rendimiento de la aplicación. Para determinar si el servidor está escribiendo en el disco como resultado de superar el tamaño de la tabla temporal, examine las métricas siguientes:

```
show global status like 'created_tmp_disk_tables';
show global status like 'created_tmp_tables';
```
La métrica `created_tmp_disk_tables` indica el número de tablas que se crearon en el disco. La métrica `created_tmp_table` indica cuántas tablas temporales se deben crear en la memoria, dada la carga de trabajo. Para determinar si la ejecución de una consulta específica usará tablas temporales, ejecute la instrucción [EXPLAIN](https://dev.mysql.com/doc/refman/8.0/en/explain.html) en la consulta. Los detalles de la columna `extra` indican `Using temporary` si la consulta se ejecutará con tablas temporales.

Para calcular el porcentaje de la carga de trabajo con consultas que desbordan los discos, use los valores de las métricas en la fórmula siguiente:

```(created_tmp_disk_tables / (created_tmp_disk_tables + created_tmp_tables)) * 100```

Idealmente, este porcentaje debe ser inferior al 25 %. Si ve que el porcentaje es del 25 % o superior, se recomienda modificar dos parámetros de servidor, tmp_table_size y max_heap_table_size.

## <a name="database-schema-and-queries"></a>Consultas y esquema de base de datos

Estos son algunos consejos que hay que tener en cuenta al compilar el esquema de base de datos y las consultas.

### <a name="use-the-right-datatype-for-your-table-columns"></a>Uso del tipo de datos correcto para las columnas de la tabla
El uso de los tipos de datos correctos en función del tipo de datos que quiere almacenar puede optimizar el almacenamiento y reducir los errores que pueden producirse debido a los tipos de datos incorrectos.

### <a name="use-indexes"></a>Uso de índices
Para evitar consultas lentas, puede utilizar índices. Los índices pueden ayudar a buscar filas con columnas específicas rápidamente. Consulte [Uso de índices en MySQL](https://dev.mysql.com/doc/refman/8.0/en/mysql-indexes.html).

### <a name="use-explain-for-your-select-queries"></a>Uso de instrucciones EXPLAIN en las consultas SELECT
Use la instrucción `EXPLAIN` para obtener información detallada sobre lo que hace MySQL para ejecutar la consulta. Puede ayudarle a detectar cuellos de botella o problemas con ella. Vea [Uso de EXPLAIN para solucionar problemas relacionados con el rendimiento de consultas](https://docs.microsoft.com/azure/mysql/howto-troubleshoot-query-performance).


