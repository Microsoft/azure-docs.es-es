---
title: Características y capacidades de SQL Managed Instance para Azure Arc
description: Características y capacidades de SQL Managed Instance para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: vin-yu
ms.author: vinsonyu
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 810a08d2f72359b385d2a7567b796aa222c6ab14
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932601"
---
# <a name="features-and-capabilities-of-azure-arc-enabled-sql-managed-instance"></a>Características y capacidades de SQL Managed Instance para Azure Arc

SQL Managed Instance para Azure Arc comparte una base de código común con la versión estable más reciente de SQL Server. La mayoría de las características estándar de lenguaje SQL, procesamiento de consultas y administración de bases de datos son idénticas. Las características que son comunes entre SQL Server y SQL Database o Instancia administrada de SQL son:

- Características del lenguaje: [Palabras clave de lenguaje de control de flujo](/sql/t-sql/language-elements/control-of-flow), [Cursores](/sql/t-sql/language-elements/cursors-transact-sql), [Tipos de datos](/sql/t-sql/data-types/data-types-transact-sql), [Instrucciones DML](/sql/t-sql/queries/queries), [Predicados](/sql/t-sql/queries/predicates), [Números de secuencia](/sql/relational-databases/sequence-numbers/sequence-numbers), [Procedimientos almacenados](/sql/relational-databases/stored-procedures/stored-procedures-database-engine) y [Variables](/sql/t-sql/language-elements/variables-transact-sql).
- Características de base de datos: [Ajuste automático (forzado del plan)](/sql/relational-databases/automatic-tuning/automatic-tuning), [Seguimiento de cambios](/sql/relational-databases/track-changes/about-change-tracking-sql-server), [Intercalación de bases de datos](/sql/relational-databases/collations/set-or-change-the-database-collation), [Bases de datos independientes](/sql/relational-databases/databases/contained-databases), [Usuarios contenidos](/sql/relational-databases/security/contained-database-users-making-your-database-portable), [Compresión de datos](/sql/relational-databases/data-compression/data-compression), [Configuración de base de datos](/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql), [Operaciones de índice en línea](/sql/relational-databases/indexes/perform-index-operations-online), [Creación de particiones](/sql/relational-databases/partitions/partitioned-tables-and-indexes) y [Tablas temporales](/sql/relational-databases/tables/temporal-tables) ([consulte la guía de introducción](/sql/relational-databases/tables/getting-started-with-system-versioned-temporal-tables)).
- Características de seguridad: [Roles de aplicación](/sql/relational-databases/security/authentication-access/application-roles), [Enmascaramiento dinámico de datos](/sql/relational-databases/security/dynamic-data-masking) ([Introducción al enmascaramiento dinámico de datos de SQL Database con Azure Portal](../../azure-sql/database/dynamic-data-masking-configure-portal.md)), [Seguridad de nivel de fila](/sql/relational-databases/security/row-level-security).
- Capacidades de varios modelos: [Procesamiento de gráficos](/sql/relational-databases/graphs/sql-graph-overview), [Datos JSON](/sql/relational-databases/json/json-data-sql-server), [OPENXML](/sql/t-sql/functions/openxml-transact-sql), [Spatial](/sql/relational-databases/spatial/spatial-data-sql-server), [OPENJSON](/sql/t-sql/functions/openjson-transact-sql) e [Índices XML](/sql/t-sql/statements/create-xml-index-transact-sql).



[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="features-of-azure-arc-enabled-sql-managed-instance"></a>Características de SQL Managed Instance para Azure Arc

###  <a name="rdbms-high-availability"></a><a name="RDBMSHA"></a> RDBMS High Availability  
  
|Característica|SQL Managed Instance para Azure Arc|
|-------------|----------------|
|Trasvase de registros|Sí| 
|Compresión de copia de seguridad|Sí|
|Instantáneas de base de datos|Sí|
|Instancia de clúster de conmutación por error AlwaysOn<sup>1</sup>| No es aplicable. Capacidades similares disponibles |
|Grupos de disponibilidad AlwaysOn<sup>2</sup>|Se han planeado capacidades de alta disponibilidad.|
|Grupos de disponibilidad básica <sup>2</sup>|Se han planeado capacidades de alta disponibilidad.|
|Grupo de disponibilidad de confirmación de réplica mínima<sup>2</sup>|Se han planeado capacidades de alta disponibilidad.|
|Grupo de disponibilidad sin clúster|Sí|
|Restauración de archivos y páginas en línea|Sí|
|Índices en línea|Sí|
|Recompilaciones de índices en línea reanudables|Sí|
|Cambio de esquema en línea|Sí|
|Recuperación rápida|Sí|
|Copias de seguridad reflejadas|Sí|
|Agregar memoria y CPU sin interrupción|Sí|
|Copia de seguridad cifrada|Sí|
|Copia de seguridad híbrida en Azure (copia de seguridad en dirección URL)|Sí|

<sup>1</sup> En un escenario en el que hay un error de pod, se inicia una nueva instancia de SQL Managed Instance y se vuelve a conectar al volumen persistente que contiene los datos. [Obtenga más información sobre volúmenes persistentes de Kubernetes aquí](https://kubernetes.io/docs/concepts/storage/persistent-volumes).

<sup>2</sup> Las versiones futuras van a proporcionar capacidades de AG 

###  <a name="rdbms-scalability-and-performance"></a><a name="RDBMSSP"></a> RDBMS Scalability and Performance  

|Característica|SQL Managed Instance para Azure Arc|
|-------------|----------------|
|columnstore|   Sí|
|Archivos binarios de objetos de gran tamaño en índices de almacén de columnas en clúster|    Sí|
|Recompilación de índices de almacén de columnas no en clúster en línea| Sí|
|OLTP en memoria|    Sí|
|Memoria principal persistente|    Sí|
|Particiones de tabla e índice|  Sí
|Compresión de datos|  Sí|
|regulador de recursos| Sí|
|Paralelismo de tabla con particiones| Sí|
|Asignación de memoria de página grande habilitada para NUMA y matriz de búferes|  Sí|
|Regulación de recursos de E/S|    Sí|
|Perdurabilidad diferida|    Sí|
|Ajuste automático|  Sí|
|Combinaciones adaptables de modo de proceso por lotes| Sí|
|Comentarios de concesión de memoria de modo de proceso por lotes|  Sí|
|Ejecución intercalada de funciones con valores de tabla de múltiples instrucciones|  Sí|
|Mejoras de inserción masiva   |Sí|

###  <a name="rdbms-security"></a><a name="RDBMSS"></a> RDBMS Security  
|Característica|SQL Managed Instance para Azure Arc|
|-------------|----------------|
|Seguridad de nivel de fila|    Sí|
|Always Encrypted|  Sí|
|Always Encrypted con enclaves seguros| No|
|Enmascaramiento de datos dinámicos|  Sí|
|Auditoría básica|    Sí|
|Auditoría específica| Sí|
|Cifrado de base de datos transparente|   Sí|
|Roles definidos por el usuario|    Sí|
|Bases de datos independientes|   Sí|
|Cifrado para copias de seguridad|    Sí|

###  <a name="rdbms-manageability"></a><a name="RDBMSM"></a> RDBMS Manageability  

|Característica|SQL Managed Instance para Azure Arc|
|-------------|----------------|
|Conexión de administración dedicada|    Sí|
|Compatibilidad con PowerShell scripting|  Sí|
|Compatibilidad con las operaciones de componentes de aplicación de capa de datos: extracción, implementación, actualización, eliminación| Sí
|Automatización de directivas (comprobar en la programación y cambio)   |Sí|
|Recopilador de datos de rendimiento|    Sí|
|Informes de rendimiento estándar   |Sí|
|Guías del plan y congelación del plan para las guías del plan| Sí|
|Consulta directa de vistas indexadas (mediante la sugerencia NOEXPAND)|   Sí|
|Mantenimiento automático de vistas indexadas    |Sí|
|Vistas distribuidas con particiones| Sí|
|Operaciones indizadas en paralelo    |Sí|
|Uso automático de vistas indexadas por el optimizador de consultas|  Sí|
|Comprobación de coherencia en paralelo |Sí|


### <a name="programmability"></a><a name="Programmability"></a> Programmability  

|Característica|SQL Managed Instance para Azure Arc|
|-------------|----------------|
|JSON|  Sí |       |
|Almacén de consultas    |Sí    |       
|Temporal|  Sí |       
|Compatibilidad con XML nativo|    Sí |       
|Índices XML   |Sí    |       
|Funciones MERGE y UPSERT|   Sí |       
|Tipos de datos de fecha y hora    |Sí    |       
|Compatibilidad para internacionalización|  Sí |       
|Búsqueda de texto completo y semántica |    No      |
|Especificación de idioma en la consulta |Sí        |   
|Service Broker (mensajería)|    Sí     |   
|Transact-SQL, extremos|    Sí |       
|Grafo| Sí |   
|Machine Learning Services| No  |   
|PolyBase| No   |


### <a name="tools"></a>Herramientas

SQL Managed Instance para Azure Arc admite diversas herramientas de datos que pueden ayudar a administrar los datos.

| **Herramienta** | SQL Managed Instance para Azure Arc|
| --- | --- | --- |
| Azure Portal<sup>1</sup> | No |
| CLI de Azure | No |
| [Azure Data Studio](/sql/azure-data-studio/what-is) | Sí |
| Azure PowerShell | Sí |
| [Archivo BACPAC (exportar)](/sql/relational-databases/data-tier-applications/export-a-data-tier-application) | Sí |
| [Archivo BACPAC (importar)](/sql/relational-databases/data-tier-applications/import-a-bacpac-file-to-create-a-new-user-database) | Sí |
| [SQL Server Data Tools (SSDT)](/sql/ssdt/download-sql-server-data-tools-ssdt) | Sí |
| [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) | Sí |
| [SQL Server PowerShell](/sql/relational-databases/scripting/sql-server-powershell) | Sí |
| [SQL Server Profiler](/sql/tools/sql-server-profiler/sql-server-profiler) | Sí |

<sup>1</sup> Azure Portal solo se usa para ver instancias de SQL Managed Instance para Azure Arc en modo de solo lectura durante la versión preliminar.


### <a name="unsupported-features--services"></a><a name="Unsupported"></a> Características y servicios no admitidos

Las siguientes características y servicios no están disponibles para SQL Managed Instance para Azure Arc. La compatibilidad de estas características se incrementará con el tiempo.

| Área | Característica o servicio no admitido |
|-----|-----|
| **Motor de base de datos** | Replicación de mezcla |
| &nbsp; | Stretch DB |
| &nbsp; | Consulta distribuida con conexiones de terceros |
| &nbsp; | Servidores vinculados a orígenes de datos distintos de productos de SQL Server y Azure SQL |
| &nbsp; | Procedimientos extendidos almacenados del sistema (XP_CMDSHELL, etc.) |
| &nbsp; | FileTable, FILESTREAM |
| &nbsp; | Ensamblados de CLR con el conjunto de permisos EXTERNAL_ACCESS o UNSAFE |
| &nbsp; | Buffer Pool Extension |
| **Agente SQL Server** |  Subsistemas: CmdExec, PowerShell, Agente de lectura de cola, SSIS, SSAS, SSRS |
| &nbsp; | Alertas |
| &nbsp; | Copia de seguridad administrada |
| **Alta disponibilidad** | Creación de reflejo de la base de datos  |
| **Seguridad** | Administración extensible de claves |
| &nbsp; | Autenticación de AD para servidores vinculados | 
| &nbsp; | Autenticación de AD para grupos de disponibilidad | 