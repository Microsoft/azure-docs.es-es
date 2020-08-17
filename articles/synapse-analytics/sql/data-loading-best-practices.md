---
title: Procedimientos recomendados para la carga de datos
description: Recomendaciones y optimizaciones de rendimiento para cargar datos en SQL de Synapse
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: kevin
ms.reviewer: igorstan
ms.custom: azure-synapse
ms.openlocfilehash: acfb2af7d482f9c0a51596818b1302584277defb
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87486823"
---
# <a name="best-practices-for-loading-data-for-data-warehousing"></a>Procedimientos recomendados para la carga de datos en almacenamientos de datos

Recomendaciones y optimizaciones de rendimiento para cargar datos

## <a name="prepare-data-in-azure-storage"></a>Preparación de datos en Azure Storage

Para minimizar la latencia, reubique la capa de almacenamiento y el almacenamiento de datos.

Al exportar datos en formato de archivo ORC, pueden producirse errores de falta de memoria de Java con las columnas de texto grandes. Para resolver este problema, exporte solo un subconjunto de las columnas.

PolyBase no puede cargar filas que tengan más de un millón de bytes de datos. Cuando ponga datos en los archivos de texto de Azure Blob Storage o Azure Data Lake Store, estos tienen que tener menos de un millón de bytes de datos. Esta limitación de datos es cierta independientemente del esquema de tabla definido.

Todos los formatos de archivo tienen diferentes características de rendimiento. Para lograr la carga más rápida, use archivos de texto delimitados comprimidos. La diferencia entre el rendimiento de UTF-8 y UTF-16 es mínima.

Dividir archivos comprimidos grandes en archivos comprimidos más pequeños.

## <a name="run-loads-with-enough-compute"></a>Ejecución de cargas con suficientes recursos de proceso

Para una velocidad de carga más rápida, ejecute solo una carga de trabajo de cada vez. Si no es factible, ejecute un número mínimo de cargas al mismo tiempo. Si espera un trabajo de carga grande, considere la posibilidad de escalar verticalmente el grupo de SQL antes de la carga.

Para ejecutar cargas con recursos de proceso adecuados, cree usuarios de carga designados para ejecutar cargas. Asigne cada usuario de carga a un grupo de cargas de trabajo o una clase de recurso específicos. Para ejecutar una carga, inicie sesión como uno de los usuarios de carga y, a continuación, ejecute la carga. La carga se ejecuta con la clase de recurso del usuario.  Este método es más sencillo que intentar cambiar la clase de recursos de un usuario para que se ajuste a la necesidad actual de clase de recurso.

### <a name="create-a-loading-user"></a>Creación de un usuario de carga

En este ejemplo se crea un usuario de carga para la clase de recurso staticrc20. El primer paso consiste en **conectarse al servidor principal** y crear un inicio de sesión.

```sql
   -- Connect to master
   CREATE LOGIN LoaderRC20 WITH PASSWORD = 'a123STRONGpassword!';
```

Conéctese al almacenamiento de datos y cree un usuario. El código siguiente da por supuesto que está conectado a la base de datos denominada mySampleDataWarehouse. Muestra cómo crear un usuario llamado LoaderRC20 y concederle el permiso de control de usuario en una base de datos. A continuación, agrega el usuario como miembro del rol de base de datos staticrc20.  

```sql
   -- Connect to the database
   CREATE USER LoaderRC20 FOR LOGIN LoaderRC20;
   GRANT CONTROL ON DATABASE::[mySampleDataWarehouse] to LoaderRC20;
   EXEC sp_addrolemember 'staticrc20', 'LoaderRC20';
```

Para ejecutar una carga con recursos de las clases de recursos staticRC20, inicie sesión como LoaderRC20 y ejecute la carga.

Ejecute cargas en clases de recursos estáticas en lugar de dinámicas. El uso de clases de recursos estáticas garantiza los mismos recursos independientemente de las [unidades de almacenamiento de datos](resource-consumption-models.md). Si utiliza una clase de recursos dinámica, los recursos varían según el nivel de servicio. Para las clases dinámicas, un nivel de servicio inferior significa que probablemente necesita usar una clase de recursos más grande para el usuario de carga.

## <a name="allow-multiple-users-to-load"></a>Posibilidad de que varios usuarios realicen cargas

A menudo, es necesario que varios usuarios puedan cargar datos en un almacén de datos. La carga con [CREATE TABLE AS SELECT (Transact-SQL)](/sql/t-sql/statements/create-table-as-select-azure-sql-data-warehouse?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest) requiere permisos de CONTROL en la base de datos.  El permiso CONTROL ofrece control de acceso a todos los esquemas. Probablemente no desee que todos los usuarios de carga tengan control de acceso en todos los esquemas. Para limitar los permisos, use la instrucción DENY CONTROL.

Por ejemplo: tenemos los esquemas de base de datos schema_A para el departamento A, schema_B para el departamento B y los usuarios de PolyBase user_A y user_B con cargas en los departamentos A y B respectivamente. A ambos se les ha concedido permiso de CONTROL sobre la base de datos. Ahora, los creadores de los esquemas A y B bloquean dichos esquemas utilizando DENY:

```sql
   DENY CONTROL ON SCHEMA :: schema_A TO user_B;
   DENY CONTROL ON SCHEMA :: schema_B TO user_A;
```

User_A y user_B estarán ahora bloqueados en el esquema de los otros departamentos.

## <a name="load-to-a-staging-table"></a>Cargar los datos en una tabla de almacenamiento provisional

Para lograr la velocidad de carga más rápida para mover datos a una tabla de almacenamiento de datos, cargue los datos en una tabla de almacenamiento provisional.  Definir la tabla de almacenamiento provisional como un montón y usar round robin para la opción de distribución.

Tenga en cuenta que la carga suele ser un proceso de dos pasos en el que se carga primero en una tabla de almacenamiento provisional y, a continuación, se insertan los datos en una tabla de almacenamiento de datos de producción. Si la tabla de producción utiliza una distribución hash, el tiempo total para cargar e insertar puede ser más rápido si define la tabla de almacenamiento provisional con la distribución hash. Cargar la tabla de almacenamiento provisional lleva más tiempo, pero el segundo paso de insertar las filas en la tabla de producción no incurre en movimiento de datos a través de las distribuciones.

## <a name="load-to-a-columnstore-index"></a>Carga en un índice de almacén de columnas

Los índices de almacén de columnas necesitan mucha memoria para comprimir los datos en grupos de filas de alta calidad. Para una mejor compresión y eficacia del índice, el índice de almacén de columnas tiene que comprimir el máximo de 1.048.576 filas en cada grupo de filas. Si no hay memoria suficiente, es posible que el índice de almacén de columnas no pueda lograr las tasas de compresión máximas. Esto afecta a su vez al rendimiento de las consultas. Para un análisis detallado, consulte [Maximización de la calidad del grupo de filas del almacén de columnas](data-load-columnstore-compression.md).

- Para asegurarse de que el usuario de carga tenga suficiente memoria para lograr la tasa de compresión máxima, utilice usuarios de carga que sean miembros de una clase de recursos mediana o grande.
- Cargue filas suficientes para rellenar completamente los nuevos grupos de filas. Durante una carga masiva, cada 1.048.576 filas se comprime directamente en el almacén de columnas como un grupo de filas completo. Las cargas con menos de 102.400 filas envían las filas al almacén delta, donde se mantienen las filas en un índice de árbol b. Si carga muy pocas filas, puede que pasen todas al almacén delta y que no se compriman inmediatamente con el formato de almacén de columnas.

## <a name="increase-batch-size-when-using-sqlbulkcopy-api-or-bcp"></a>Aumento del tamaño de lote al usar SQLBulkCopy API o BCP

Como se ha mencionado anteriormente, la carga con PolyBase proporcionará el mayor rendimiento con el grupo de SQL de Synapse. Si no puede usar PolyBase para cargar y debe usar SQLBulkCopy API (o BCP), considere la posibilidad de aumentar el tamaño del lote para mejorar el rendimiento, se recomienda un tamaño de lote entre 100 000 filas y 1 millón.

## <a name="manage-loading-failures"></a>Administración de errores al cargar

Una carga que utiliza una tabla externa puede producir el error *"Consulta anulada: se alcanzó el umbral de rechazo máximo al leer desde un origen externo"* . Este mensaje indica que sus datos externos contienen registros con modificaciones. Un registro de datos se considera "con modificaciones" si los tipos de datos y l número de columnas no coincide con las definiciones de columna de la tabla externa o si los datos no se ajustan al formato de archivo externo especificado.

Para corregir estos registros, asegúrese de que la tabla externa y las definiciones de formato de archivo externos son correctas y que los datos externos se ajustan a estas definiciones. En el caso de que un subconjunto de registros de datos externos contenga registros con modificaciones, puede rechazar estos registros para sus consultas mediante las opciones de rechazo en CREATE EXTERNAL TABLE.

## <a name="insert-data-into-a-production-table"></a>Inserción de datos en una tabla de producción

Una tabla pequeña se puede cargar una sola vez con una [instrucción INSERT](/sql/t-sql/statements/insert-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest), o incluso una recarga periódica de una búsqueda puede funcionar bien con una instrucción del tipo `INSERT INTO MyLookup VALUES (1, 'Type 1')`.  Sin embargo las inserciones simples no son tan eficaces como para realizar una carga masiva.

Si se tienen miles de inserciones simples a lo largo del día, agrúpelas por lotes para que pueda cargarlas masivamente.  Desarrolle los procesos para anexar las inserciones sencillas a un archivo y, a continuación, cree otro proceso que cargue el archivo de forma periódica.

## <a name="create-statistics-after-the-load"></a>Creación de estadísticas después de la carga

Para mejorar el rendimiento de las consultas, es importante crear estadísticas de todas las columnas de todas las tablas después de la primera carga, o bien después de que se realicen cambios importantes en los datos.  Esto se puede realizar manualmente o bien puede habilitar la [creación automática de estadísticas](../sql-data-warehouse/sql-data-warehouse-tables-statistics.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

Para ver una explicación detallada de las estadísticas, consulte [Estadísticas](develop-tables-statistics.md). En el ejemplo siguiente se muestra cómo crear manualmente las estadísticas de cinco columnas de la tabla Customer_Speed.

```sql
create statistics [SensorKey] on [Customer_Speed] ([SensorKey]);
create statistics [CustomerKey] on [Customer_Speed] ([CustomerKey]);
create statistics [GeographyKey] on [Customer_Speed] ([GeographyKey]);
create statistics [Speed] on [Customer_Speed] ([Speed]);
create statistics [YearMeasured] on [Customer_Speed] ([YearMeasured]);
```

## <a name="rotate-storage-keys"></a>Rotación de claves de almacenamiento

Es un buen procedimiento de seguridad cambiar la clave de acceso al almacenamiento de blobs de forma regular. Tiene dos claves de almacenamiento para la cuenta de Blob Storage, lo que le permite la transición de las claves.

Para rotar las cuentas de Azure Storage:

Para cada cuenta de almacenamiento cuya clave haya cambiado, ejecute [ALTER DATABASE SCOPED CREDENTIAL](/sql/t-sql/statements/alter-database-scoped-credential-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest).

Ejemplo:

Se crea la clave original

```sql
CREATE DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key1'
```

Se cambia de la clave 1 a la clave 2.

```sql
ALTER DATABASE SCOPED CREDENTIAL my_credential WITH IDENTITY = 'my_identity', SECRET = 'key2'
```

No es necesario cambiar nada más en los orígenes de datos externos subyacentes.

## <a name="next-steps"></a>Pasos siguientes

- Para más información acerca de PolyBase y del diseño de un proceso de extracción, carga y transformación (ETL), consulte [Design ELT for SQL Data Warehouse](../sql-data-warehouse/design-elt-data-loading.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) (Diseño de ELT para SQL Data Warehouse).
- Si desea un tutorial sobre carga, consulte [Uso de PolyBase para cargar de datos de Azure Blob Storage en Azure SQL Data Warehouse](../sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
- Para supervisar las cargas de datos, consulte [Supervisión de la carga de trabajo mediante DMV](../sql-data-warehouse/sql-data-warehouse-manage-monitor.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).
