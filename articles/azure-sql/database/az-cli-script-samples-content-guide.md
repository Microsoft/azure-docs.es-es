---
title: Ejemplos de script de la CLI de Azure
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Ejemplos de scripts de la CLI de Azure para crear y administrar Azure SQL Database e Instancia administrada de Azure SQL
services: sql-database
ms.service: sql-db-mi
ms.subservice: service
ms.custom: overview-samples, mvc, sqldbrb=2, devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 02/03/2019
ms.openlocfilehash: 893ea607b322c2f9b2d8a4e8b5b087e21c476d98
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497178"
---
# <a name="azure-cli-samples-for-azure-sql-database-and-sql-managed-instance"></a>Ejemplos de la CLI de Azure para Azure SQL Database e Instancia administrada de SQL 
 
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Puede configurar Azure SQL Database e Instancia administrada de SQL mediante la <a href="/cli/azure">CLI de Azure</a>.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este tema es preciso que ejecute la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

# <a name="azure-sql-database"></a>[Azure SQL Database](#tab/single-database)

En la tabla siguiente se incluyen vínculos a ejemplos de scripts de la CLI de Azure para administrar bases de datos individuales y agrupadas en Azure SQL Database. 

|Área|Descripción|
|---|---|
|**Creación de bases de datos en Azure SQL Database**||
| [Creación de una base de datos única y configuración de una regla de firewall](scripts/create-and-configure-database-cli.md) | Crea una base de datos de SQL Database y configura una regla de firewall en el nivel de servidor. |
| [Creación de grupos elásticos y traslado de bases de datos agrupadas](scripts/move-database-between-elastic-pools-cli.md) | Crea grupos elásticos de SQL, traslada las bases de datos agrupadas y cambia los tamaños de proceso. |
|**Escalado de bases de datos en Azure SQL Database**||
| [Escalado de una base de datos única](scripts/monitor-and-scale-database-cli.md) | Escala una base de datos de SQL Database a un tamaño de proceso distinto después de consultar la información del tamaño de la base de datos. |
| [Escalado de un grupo elástico](scripts/scale-pool-cli.md) | Escala un grupo elástico de SQL a un tamaño de proceso distinto. |
|**Configuración de la replicación geográfica y de la conmutación por error**||
| [Adición de una base de datos única a un grupo de conmutación por error](scripts/add-database-to-failover-group-cli.md)| Crea una base de datos y un grupo de conmutación por error, agrega la base de datos al grupo de conmutación por error y, después, prueba la conmutación por error en el servidor secundario. |
| [Configuración de un grupo de conmutación por error para un grupo elástico](../../sql-database/scripts/sql-database-add-elastic-pool-to-failover-group-cli.md) | Crea una base de datos, lo agrega a un grupo elástico, agrega el grupo elástico al grupo de conmutación por error y, después, prueba la conmutación por error en el servidor secundario. |
| [Configuración y conmutación por error de una base de datos única mediante la replicación geográfica activa](../../sql-database/scripts/sql-database-setup-geodr-and-failover-database-cli.md)| Configura la replicación geográfica activa para una base de datos de Azure SQL Database y la conmuta por error a la réplica secundaria. |
| [Configuración y conmutación por error de una base de datos agrupada mediante la replicación geográfica activa](../../sql-database/scripts/sql-database-setup-geodr-and-failover-pool-cli.md)| Configura la replicación geográfica activa para una base de datos de un grupo elástico y la conmuta por error a la réplica secundaria. |
| **Detección de amenazas y auditoría** |
| [Configuración de detección de amenazas y auditoría](../../sql-database/scripts/sql-database-auditing-and-threat-detection-cli.md)| Configura las directivas de auditoría y detección de amenazas para una base de datos de Azure SQL Database. |
| **Copia de seguridad, restauración, copia e importación de una base de datos**||
| [Copia de seguridad de una base de datos](../../sql-database/scripts/sql-database-backup-database-cli.md)| Realiza una copia de seguridad de una base de datos de SQL Database de Azure en una copia de seguridad de Azure Storage. |
| [Restauración de una base de datos](../../sql-database/scripts/sql-database-restore-database-cli.md)| Restaura una base de datos de SQL Database desde una copia de seguridad con redundancia geográfica y restaura una base de datos eliminada a la copia de seguridad más reciente. |
| [Copia de una base de datos en un nuevo servidor](../../sql-database/scripts/sql-database-copy-database-to-new-server-cli.md) | Crea una copia de una base de datos existente de SQL Database en un servidor nuevo. |
| [Importación de una base de datos desde un archivo BACPAC](../../sql-database/scripts/sql-database-import-from-bacpac-cli.md)| Importa una base de datos a SQL Database desde un archivo BACPAC. |
|||

Obtenga más información sobre la [API de la CLI de Azure de la base de datos única](single-database-manage.md#the-azure-cli).

# <a name="azure-sql-managed-instance"></a>[Instancia administrada de Azure SQL](#tab/managed-instance)

En la tabla siguiente se incluyen vínculos a ejemplos de script de la CLI de Azure para la instancia administrada de Azure SQL.

|Área|Descripción|
|---|---|
| **Creación de una instancia administrada de SQL**||
| [Creación de una instancia administrada de SQL](../../sql-database/scripts/sql-database-create-configure-managed-instance-cli.md)| Crea una instancia administrada de SQL. |
| **Configuración del Cifrado de datos transparente (TDE)**||
| [Administración del Cifrado de datos transparente en Instancia administrada de SQL con Azure Key Vault](../../sql-database/scripts/transparent-data-encryption-byok-sql-managed-instance-cli.md)| Configura el Cifrado de datos transparente (TDE) en Instancia administrada de SQL mediante Azure Key Vault con varios escenarios clave. |
|**Configuración de un grupo de conmutación por error**||
| [Configuración de un grupo de conmutación por error para una instancia administrada de SQL](../../sql-database/scripts/sql-database-add-managed-instance-to-failover-group-cli.md) | Crea dos instancias administradas de SQL, las agrega a un grupo de conmutación por error y, a continuación, prueba la conmutación por error de la instancia administrada principal de SQL en la instancia administrada secundaria. |
|||

Para obtener más ejemplos de instancia administrada de SQL, consulte la información sobre [crear](https://blogs.msdn.microsoft.com/sqlserverstorageengine/20../../create-azure-sql-managed-instance-using-azure-cli/), [actualizar](https://blogs.msdn.microsoft.com/sqlserverstorageengine/20../../modify-azure-sql-database-managed-instance-using-azure-cli/), [mover una base de datos](https://blogs.msdn.microsoft.com/sqlserverstorageengine/20../../cross-instance-point-in-time-restore-in-azure-sql-database-managed-instance/) y [trabajar con](https://medium.com/azure-sqldb-managed-instance/working-with-sql-managed-instance-using-azure-cli-611795fe0b44) scripts.

Más información sobre la [API de la CLI de Azure de Instancia administrada de SQL](../managed-instance/api-references-create-manage-instance.md#azure-cli-create-and-configure-managed-instances).

---
