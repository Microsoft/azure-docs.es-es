---
title: Características admitidas de Azure SQL Edge
description: Información sobre los detalles de las características compatibles con Azure SQL Edge.
keywords: introducción a SQL Edge, qué es SQL Edge, información general sobre SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/03/2020
ms.openlocfilehash: 646ce94587a9aa1bb8fd20a28b84658994b25cf1
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90886536"
---
# <a name="supported-features-of-azure-sql-edge"></a>Características admitidas de Azure SQL Edge 

Azure SQL Edge se basa en la versión más reciente del Motor de base de datos de SQL. Admite un subconjunto de características compatibles con SQL Server 2019 en Linux, además de algunas características que actualmente no se admiten o no están disponibles en SQL Server 2019 en Linux (o en SQL Server en Windows).

Para obtener una lista completa de las características admitidas en SQL Server en Linux, consulte [Ediciones y características admitidas de SQL Server 2019 en Linux](https://docs.microsoft.com/sql/linux/sql-server-linux-editions-and-components-2019). Para conocer las ediciones y las características admitidas de SQL Server en Windows, consulte [Ediciones y características admitidas de SQL Server 2019 (15.x)](https://docs.microsoft.com/sql/sql-server/editions-and-components-of-sql-server-version-15).

## <a name="azure-sql-edge-editions"></a>Ediciones de Azure SQL Edge

Azure SQL Edge está disponible en dos ediciones o planes de software diferentes. Estas ediciones tienen conjuntos de características idénticos y solo difieren en cuanto a sus derechos de uso, y la cantidad de memoria y los núcleos a los que pueden obtener acceso en el sistema del host.

   |**Plan**  |**Descripción**  |
   |---------|---------|
   |Desarrollador de Azure SQL Edge  |  Solo para desarrollo. Cada contenedor de desarrollador de Azure SQL Edge está limitado a hasta 4 núcleos y 32 GB de memoria.  |
   |Azure SQL Edge    |  Para producción. Cada contenedor de Azure SQL Edge está limitado a hasta 8 núcleos y 64 GB de memoria.  |

## <a name="operating-system"></a>Sistema operativo

Los contenedores de Azure SQL Edge se basan en Ubuntu 18.04 y, como tal, solo se pueden ejecutar en hosts de Docker con Ubuntu 18.04 LTS (recomendado) o Ubuntu 20.04 LTS. Es posible ejecutar contenedores de Azure SQL Edge en otros hosts de sistemas operativos, por ejemplo, se puede ejecutar en otras distribuciones de Linux o en Windows (mediante Docker CE o Docker EE); sin embargo, Microsoft no recomienda hacerlo, ya que es posible que esta configuración no se haya probado de manera exhaustiva.

La configuración recomendada para ejecutar Azure SQL Edge en Windows consiste en configurar una máquina virtual de Ubuntu en el host de Windows y, a continuación, ejecutar Azure SQL Edge dentro de la máquina virtual de Linux.

Los sistemas de archivos recomendados y compatibles para Azure SQL Edge son EXT4 y XFS. Si se usan volúmenes persistentes para realizar la copia de seguridad del almacenamiento de bases de datos de Azure SQL Edge, el sistema de archivos de host subyacente debe ser EXT4 y XFS.

## <a name="hardware-support"></a>Compatibilidad de hardware

Azure SQL Edge requiere un procesador de 64 bits (ya sea x64 o ARM64), con un mínimo de un procesador y 1 GB de RAM en el host. Mientras que la superficie de memoria de inicio de Azure SQL Edge está próxima a 450 MB, se necesita memoria adicional para otros módulos o procesos de IoT Edge que se ejecutan en el dispositivo perimetral. Los requisitos de CPU y memoria reales para Azure SQL Edge variarán en función de la complejidad de la carga de trabajo y el volumen de datos que se estén procesando. Al elegir un hardware para la solución, Microsoft recomienda ejecutar amplias pruebas de rendimiento para asegurarse de que se cumplan las características de rendimiento necesarias para la solución.  

## <a name="azure-sql-edge-components"></a>Componentes de Azure SQL Edge

Azure SQL Edge solo admite el motor de base de datos. No incluye compatibilidad con otros componentes disponibles con SQL Server 2019 en Windows o con SQL Server 2019 en Linux. En concreto, Azure SQL Edge no admite componentes de SQL Server, como Analysis Services, Reporting Services, Integration Services, Master Data Services, Machine Learning Services (en base de datos) y Machine Learning Server (independiente).

## <a name="supported-features"></a>Características admitidas

Además de admitir un subconjunto de características de SQL Server en Linux, Azure SQL Edge incluye compatibilidad con las siguientes características nuevas: 

- Streaming de SQL, que se basa en el mismo motor que sustenta Azure Stream Analytics, proporciona funcionalidades de streaming de datos en tiempo real en Azure SQL Edge. 
- La función de T-SQL denominada `Date_Bucket` para el análisis de datos de serie temporal.
- Funcionalidades de Machine Learning a través del runtime de ONNX, incluido con el motor de SQL.

## <a name="unsupported-features"></a>Características no admitidas

La lista siguiente incluye las características de SQL Server 2019 en Linux que no se admiten actualmente en Azure SQL Edge.

| Área | Característica o servicio no admitido |
|-----|-----|
| **Diseño de la base de datos** | OLTP en memoria y comandos DDL y funciones de Transact-SQL relacionados, vistas de catálogo y vistas de administración dinámica. |
| &nbsp; | Tipo de datos `HierarchyID` y comandos DDL y funciones de Transact-SQL relacionados, vistas de catálogo y vistas de administración dinámica. |
| &nbsp; | Tipo de datos `Spatial` y comandos DDL y funciones de Transact-SQL relacionados, vistas de catálogo y vistas de administración dinámica. |
| &nbsp; | Stretch DB y comandos DDL y funciones de Transact-SQL relacionados, vistas de catálogo y vistas de administración dinámica. |
| &nbsp; | Búsqueda e índices de texto completo, y comandos DDL y funciones de Transact-SQL relacionados, vistas de catálogo y vistas de administración dinámica.|
| &nbsp; | `FileTable`, `FILESTREAM` y comandos DDL y funciones de Transact-SQL relacionados, vistas de catálogo y vistas de administración dinámica.|
| **Motor de base de datos** | Replicación. Tenga en cuenta que puede configurar Azure SQL Edge como suscriptor de inserción de una topología de replicación. |
| &nbsp; | PolyBase. Tenga en cuenta que puede configurar Azure SQL Edge como destino para tablas externas en Polybase. |
| &nbsp; | Extensibilidad de lenguaje a través de Java y Spark. |
| &nbsp; | Integración con Active Directory. |
| &nbsp; | Reducción automática de la base de datos. La propiedad de reducción automática de una base de datos se puede establecer mediante el comando `ALTER DATABASE <database_name> SET AUTO_SHRINK ON`, pero ese cambio no tiene ningún efecto. La tarea de reducción automática no se ejecutará en la base de datos. Los usuarios todavía pueden reducir los archivos de base de datos mediante los comandos DBCC. |
| &nbsp; | Instantáneas de base de datos. |
| &nbsp; | Compatibilidad con memoria persistente. |
| &nbsp; | Microsoft DTC (Coordinador de transacciones distribuidas). |
| &nbsp; | Resource Governor y regulación de recursos de E/S. |
| &nbsp; | Buffer Pool Extension. |
| &nbsp; | Consulta distribuida con conexiones de terceros. |
| &nbsp; | Servidores vinculados. |
| &nbsp; | Procedimientos almacenados extendidos del sistema (como `XP_CMDSHELL`). |
| &nbsp; | Ensamblados de CLR y comandos DDL y funciones de Transact-SQL relacionados, vistas de catálogo y vistas de administración dinámica. |
| &nbsp; | Funciones de T-SQL dependientes de CLR, como `ASSEMBLYPROPERTY`, `FORMAT`, `PARSE` y `TRY_PARSE`. |
| &nbsp; | Vistas de catálogo, funciones y cláusulas de consulta de fecha y hora dependientes de CLR. |
| &nbsp; | Extensión del grupo de búferes. |
| &nbsp; | Correo electrónico de base de datos. |
| &nbsp; | Service Broker |
| &nbsp; | Administración basada en directivas |
| &nbsp; | almacén de administración de datos |
| &nbsp; | Bases de datos independientes |
| **Agente SQL Server** |  Subsistemas: CmdExec, PowerShell, Agente de lectura de cola, SSIS, SSAS y SSRS. |
| &nbsp; | Alertas. |
| &nbsp; | Copia de seguridad administrada. |
| **Alta disponibilidad** | Grupos de disponibilidad Always On.  |
| &nbsp; | Grupos de disponibilidad básica. |
| &nbsp; | Instancias de clúster de conmutación por error de Always On. |
| &nbsp; | Creación de reflejo de base de datos. |
| &nbsp; | Agregar memoria y CPU sin interrupción. |
| **Seguridad** | Administración extensible de claves. |
| &nbsp; | Integración con Active Directory.|
| &nbsp; | Compatibilidad con enclaves seguros.|
| **Servicios** | SQL Server Browser. |
| &nbsp; | Machine Learning a través de R y Python. |
| &nbsp; | StreamInsight. |
| &nbsp; | Analysis Services. |
| &nbsp; | Reporting Services. |
| &nbsp; | Data Quality Services. |
| &nbsp; | Master Data Services. |
| &nbsp; | Distributed Replay. |
| **Facilidad de uso** | Punto de control de la utilidad de SQL Server. |

## <a name="next-steps"></a>Pasos siguientes

- [Implementación de Azure SQL Edge](deploy-portal.md)
- [Configuración de Azure SQL Edge](configure.md)
- [Conexión a una instancia de Azure SQL Edge mediante herramientas de cliente de SQL Server](connect.md)
- [Copia de seguridad y restauración de bases de datos en Azure SQL Edge](backup-restore.md)
