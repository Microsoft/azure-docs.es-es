---
title: Migración de cargas de trabajo de SSIS locales a SSIS en Azure Data Factory
description: Migre las cargas de trabajo de SSIS locales a SSIS en ADF.
services: data-factory
documentationcenter: ''
author: chugugrace
ms.author: chugu
ms.reviewer: ''
manager: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 9/3/2019
ms.openlocfilehash: 53085544be9477c03fdbbc27e709bd80dea25b92
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88186070"
---
# <a name="migrate-on-premises-ssis-workloads-to-ssis-in-adf"></a>Migración de cargas de trabajo de SSIS locales a SSIS en ADF

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

## <a name="overview"></a>Información general

Al migrar las cargas de trabajo de base de datos de SQL Server local a servicios de base de datos de Azure, concretamente Azure SQL Database o Instancia administrada de Azure SQL, también se deben migrar las cargas de trabajo de ETL de SQL Server Integration Services (SSIS), como uno de los principales servicios de valor añadido.

Azure-SSIS Integration Runtime (IR) en Azure Data Factory (ADF) admite la ejecución de paquetes de SSIS. Una vez aprovisionado Azure-SSIS IR, puede usar herramientas familiares, como SQL Server Data Tools (SSDT)/ SQL Server Management Studio (SSMS) y utilidades de la línea de comandos, como dtinstall, dtutil o dtexec, para implementar y ejecutar los paquetes en Azure. Para más información, consulte [Migrar cargas de trabajo de SQL Server Integration Services a la nube mediante lift-and-shift](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview).

En este artículo se describe el proceso de migración de las cargas de trabajo de ETL de SSIS local a SSIS en ADF. El proceso de migración consta de dos fases: **Evaluación** y **Migración**.

## <a name="assessment"></a>Evaluación

Para establecer un plan de migración completo, una evaluación exhaustiva le ayudará a identificar problemas con los paquetes de SSIS de origen que impedirían una migración correcta.

Data Migration Assistant (DMA) es una herramienta que se puede descargar de forma gratuita para este fin que se puede instalar y ejecutar localmente. Se puede crear un proyecto de evaluación de DMA de tipo **Integration Services** para evaluar paquetes de SSIS en lotes e identificar los problemas de compatibilidad que se presentan en las siguientes categorías:

- Bloqueadores de migración: Estos son problemas de compatibilidad que bloquean la ejecución de los paquetes de origen de la migración en Azure-SSIS IR. DMA proporciona guía para ayudarle a resolver estos problemas.

- Problemas informativos: Se trata de características parcialmente admitidas o en desuso que se usan en los paquetes de origen. DMA proporciona un conjunto completo de recomendaciones, enfoques alternativos disponibles en Azure y pasos de mitigación para solucionarlo.

### <a name="four-storage-types-for-ssis-packages"></a>Cuatro tipos de almacenamiento para paquetes de SSIS

- Catálogo de SSIS (SSISDB). Se presentó con SQL Server 2012 y contiene un conjunto de procedimientos almacenados, vistas y funciones con valores de tablas que se usan para trabajar con paquetes o proyectos de SSIS.
- Sistema de archivos.
- Base de datos del sistema de SQL Server (MSDB).
- Almacén de paquetes de SSIS. Se trata de una capa de administración de paquetes sobre dos subtipos:
  - MSDB, que es una base de datos del sistema de SQL Server que se usa para almacenar paquetes de SSIS.
  - Sistema de archivos administrado, que es una carpeta específica en la ruta de acceso de la instalación de SQL Server que se usa para almacenar paquetes de SSIS.

DMA admite actualmente la evaluación por lotes de paquetes almacenados en **Sistema de archivos**, **Almacén de paquetes** y **Catálogo de SSIS** desde la **versión v5.0 de DMA**.

Obtenga [DMA](https://docs.microsoft.com/sql/dma/dma-overview) y [realice su evaluación de paquetes con él](https://docs.microsoft.com/sql/dma/dma-assess-ssis).

## <a name="migration"></a>Migración

En función de los [tipos de almacenamiento](#four-storage-types-for-ssis-packages) de los paquetes de SSIS de origen y el destino de la migración de las cargas de trabajo de base de datos, los pasos para migrar los **paquetes de SSIS** y los **trabajos del Agente SQL Server** que realizan la programación de las ejecuciones de paquetes de SSIS pueden variar. Hay dos escenarios:

- [**Instancia administrada de Azure SQL** como destino de la carga de trabajo de base de datos](#azure-sql-managed-instance-as-database-workload-destination)
- [**Azure SQL Database** como destino de la carga de trabajo de base de datos](#azure-sql-database-as-database-workload-destination)

### <a name="azure-sql-managed-instance-as-database-workload-destination"></a>**Instancia administrada de Azure SQL** como destino de la carga de trabajo de base de datos

| **Tipo de almacenamiento de los paquetes** |Procedimiento de migración por lotes de paquetes de SSIS|Procedimiento de migración por lotes de trabajos de SSIS|
|-|-|-|
|SSISDB|[Migración de **SSISDB**](scenario-ssis-migration-ssisdb-mi.md)|<li>[Migración de trabajos de SSIS al agente de Instancia administrada de Azure SQL](scenario-ssis-migration-ssisdb-mi.md#ssis-jobs-to-sql-managed-instance-agent) <li>Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Sistema de archivos|Vuelva a implementarlos en recursos compartidos de archivos o Azure Files mediante dtinstall, dtutil o copia manual o manténgalos en los sistemas de archivos para su acceso mediante red virtual o IR autohospedado. Para más información, consulte [Utilidad dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility).|<li>[Migración de trabajos de SSIS al agente de Instancia administrada de Azure SQL](scenario-ssis-migration-ssisdb-mi.md#ssis-jobs-to-sql-managed-instance-agent) <li> Migración con [Asistente para migración de trabajos de SSIS en SSMS](how-to-migrate-ssis-job-ssms.md) <li>Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|SQL Server (MSDB)|Expórtelos a sistemas de archivos, recursos compartidos de archivos o Azure Files mediante SSMS o dtutil. Para más información, consulte [Exportación de paquetes de SSIS](https://docs.microsoft.com/sql/integration-services/service/package-management-ssis-service#import-and-export-packages).|Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Almacén de paquetes|Expórtelos a sistemas de archivos, recursos compartidos de archivos o Azure Files mediante SSMS o dtutil o vuelva a implementarlos en recursos compartidos de archivos o Azure Files mediante dtinstall, dtutil o copia manual o manténgalos en los sistemas de archivos para su acceso mediante red virtual o IR autohospedado. Para más información, consulte Utilidad dtutil. Para más información, consulte [Utilidad dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility).|<li>[Migración de trabajos de SSIS al agente de Instancia administrada de Azure SQL](scenario-ssis-migration-ssisdb-mi.md#ssis-jobs-to-sql-managed-instance-agent) <li> Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|

### <a name="azure-sql-database-as-database-workload-destination"></a>**Azure SQL Database** como destino de la carga de trabajo de base de datos

| **Tipo de almacenamiento de los paquetes** |Procedimiento de migración por lotes de paquetes de SSIS|Procedimiento de migración por lotes de trabajos|
|-|-|-|
|SSISDB|Vuelva a implementar en Azure-SSISDB mediante SSDT o SSMS. Para más información, consulte [Implementación de paquetes de SSIS en Azure](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-deploy-run-monitor-tutorial).|Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Sistema de archivos|Vuelva a implementarlos en recursos compartidos de archivos o Azure Files mediante dtinstall, dtutil o copia manual o manténgalos en los sistemas de archivos para su acceso mediante red virtual o IR autohospedado. Para más información, consulte [Utilidad dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility).|<li> Migración con [Asistente para migración de trabajos de SSIS en SSMS](how-to-migrate-ssis-job-ssms.md) <li> Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|SQL Server (MSDB)|Expórtelos a sistemas de archivos, recursos compartidos de archivos o Azure Files mediante SSMS o dtutil. Para más información, consulte [Exportación de paquetes de SSIS](https://docs.microsoft.com/sql/integration-services/service/package-management-ssis-service#import-and-export-packages).|Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|
|Almacén de paquetes|Expórtelos a sistemas de archivos, recursos compartidos de archivos o Azure Files mediante SSMS o dtutil o vuelva a implementarlos en recursos compartidos de archivos o Azure Files mediante dtinstall, dtutil o copia manual o manténgalos en los sistemas de archivos para su acceso mediante red virtual o IR autohospedado. Para más información, consulte Utilidad dtutil. Para más información, consulte [Utilidad dtutil](https://docs.microsoft.com/sql/integration-services/dtutil-utility).|Conviértalos en canalizaciones, actividades o desencadenadores de ADF mediante scripts, SSMS o el portal de ADF. Para más información, consulte [Característica de programación de SSMS](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages-ssms).|

## <a name="additional-resources"></a>Recursos adicionales

- [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction)
- [Database Migration Assistant](https://docs.microsoft.com/sql/dma/dma-overview)
- [Migrar cargas de trabajo de SQL Server Integration Services a la nube mediante lift-and-shift](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-lift-shift-ssis-packages-overview?view=sql-server-2017)
- [Migración de paquetes de SSIS a Instancia administrada de Azure SQL](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages-managed-instance)
- [Volver a implementar paquetes en Azure SQL Database](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages)

## <a name="next-steps"></a>Pasos siguientes

- [Validación de paquetes de SSIS implementados en Azure](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-validate-packages)
- [Ejecución de paquetes de SSIS implementados en Azure](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-run-packages)
- [Supervisión de Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/monitor-integration-runtime#azure-ssis-integration-runtime)
- [Programar la ejecución de paquetes de SQL Server Integration Services (SSIS) implementados en Azure](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-schedule-packages)
