---
title: Migración de SSIS con Instancia administrada de Azure SQL como destino de la carga de trabajo de base de datos
description: Migración de SSIS con Instancia administrada de Azure SQL como destino de la carga de trabajo de base de datos.
services: data-factory
documentationcenter: ''
author: chugugrace
ms.author: chugu
ms.reviewer: ''
manager: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 9/12/2019
ms.openlocfilehash: 6de08faee78deeb86117084b420eb5043153f62d
ms.sourcegitcommit: faeabfc2fffc33be7de6e1e93271ae214099517f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88186053"
---
# <a name="ssis-migration-with-azure-sql-managed-instance-as-the-database-workload-destination"></a>Migración de SSIS con Instancia administrada de Azure SQL como destino de la carga de trabajo de base de datos

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Para migrar cargas de trabajo de base de datos de una instancia de SQL Server a Instancia administrada de Azure SQL, debe estar familiarizado con [Azure Data Migration Service](https://docs.microsoft.com/azure/dms/dms-overview) (DMS) y las [topologías de red para migraciones de Instancia administrada de SQL con DMS](https://docs.microsoft.com/azure/dms/resource-network-topologies).

Este artículo se centra en la migración de los paquetes de SQL Server Integration Services (SSIS) almacenados en el catálogo de SSIS (SSISDB) y los trabajos del Agente SQL Server que programan las ejecuciones de paquetes de SSIS.

## <a name="migrate-ssis-catalog-ssisdb"></a>Migración del catálogo de SSIS (SSISDB)

La migración de SSISDB se puede realizar con DMS, tal como se describe en el artículo: [Migración de paquetes de SQL Server Integration Services a una instancia administrada de Azure SQL Database](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages-managed-instance).

## <a name="ssis-jobs-to-sql-managed-instance-agent"></a>Migración de trabajos de SSIS al agente de Instancia administrada de SQL

Instancia administrada de SQL tiene un programador nativo de primera clase como el Agente SQL Server local.  Puede [ejecutar paquetes SSIS mediante el agente de Azure SQL Managed Instance](how-to-invoke-ssis-package-managed-instance-agent.md).

Puesto que aún no hay disponible una herramienta de migración para los trabajos de SSIS, se deben migrar del Agente SQL Server local al agente de Instancia administrada de SQL mediante scripts o copia manual.

## <a name="additional-resources"></a>Recursos adicionales

- [Azure Data Factory](https://docs.microsoft.com/azure/data-factory/introduction)
- [Azure-SSIS Integration Runtime](https://docs.microsoft.com/azure/data-factory/create-azure-ssis-integration-runtime)
- [Azure Database Migration Service](https://docs.microsoft.com/azure/dms/dms-overview)
- [Topologías de red para migraciones de Instancia administrada de Azure SQL Database mediante Azure Database Migration Service](https://docs.microsoft.com/azure/dms/resource-network-topologies)
- [Migración de paquetes de SQL Server Integration Services a una instancia administrada de Azure SQL Database](https://docs.microsoft.com/azure/dms/how-to-migrate-ssis-packages-managed-instance)

## <a name="next-steps"></a>Pasos siguientes

- [Conexión a SSISDB en Azure](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-connect-to-catalog-database)
- [Ejecución de paquetes de SSIS implementados en Azure](https://docs.microsoft.com/sql/integration-services/lift-shift/ssis-azure-run-packages)
