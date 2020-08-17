---
title: 'Traslado de datos a una base de datos de Azure SQL: proceso de ciencia de datos en equipos'
description: Mueva datos de archivos planos (formatos CSV o TSV) o de datos almacenados en un servidor con SQL Server a Azure SQL Database.
services: machine-learning
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 01/10/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: 1f1120c78ef2a634c079705c85170d9742eabd1f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087431"
---
# <a name="move-data-to-an-azure-sql-database-for-azure-machine-learning"></a>Traslado de datos a una base de datos de Azure SQL para Azure Machine Learning

En este artículo se describen las opciones para mover datos de archivos planos (formatos CSV o TSV) o de datos almacenados en un servidor con SQL Server a Azure SQL Database. Estas tareas para mover datos a la nube forman parte del proceso de ciencia de datos en equipos.

Para ver un tema que describa las opciones para mover datos a un servidor con SQL Server para Machine Learning, consulte [Mover datos a un servidor SQL Server en una máquina virtual de Azure](move-sql-server-virtual-machine.md).

En la tabla siguiente se resumen las opciones para mover datos a una base de datos de Azure SQL.

| <b>ORIGEN</b> | <b>DESTINO: Azure SQL Database</b> |
| --- | --- |
| <b>Archivo plano (formatos CSV o TSV)</b> |[Consulta SQL de inserción masiva](#bulk-insert-sql-query) |
| <b>SQL Server local</b> |1.[Exportación a un archivo plano](#export-flat-file)<br> 2. [SQL Database Migration Wizard](#insert-tables-bcp)<br> 3. [Copia de seguridad y restauración de una base de datos](#db-migration)<br> 4. [Azure Data Factory](#adf) |

## <a name="prerequisites"></a><a name="prereqs"></a>Requisitos previos
El procedimiento aquí descrito requiere disponer de:

* Una **suscripción de Azure**. Si no tiene una suscripción, puede registrarse para obtener una [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* Una **cuenta de almacenamiento de Azure**. En este tutorial se usa una cuenta de Azure Storage para almacenar los datos. Si no dispone de una cuenta de almacenamiento de Azure, vea el artículo [Creación de una cuenta de almacenamiento](../../storage/common/storage-account-create.md) . Tras crear la cuenta de almacenamiento, tendrá que obtener la clave de cuenta que se usa para tener acceso al almacenamiento. Vea [Administración de las claves de acceso de la cuenta de almacenamiento](../../storage/common/storage-account-keys-manage.md).
* Acceso a **Azure SQL Database**. Si debe configurar una base de datos de Azure SQL, [Introducción a Microsoft Azure SQL Database](../../sql-database/sql-database-get-started.md) proporciona información sobre cómo aprovisionar una nueva base de datos de Azure SQL.
* **Azure PowerShell** instalado y configurado de forma local. Para obtener instrucciones, consulte [Instalación y configuración de Azure PowerShell](/powershell/azure/).

**Data**: los procesos de migración se demuestran con el [conjunto de datos de taxis de Nueva York](https://chriswhong.com/open-data/foil_nyc_taxi/). El conjunto de datos de taxis de Nueva York contiene información sobre los datos de carreras y las tarifas, y está disponible en Azure Blob Storage: [Datos de taxis de Nueva York](https://www.andresmh.com/nyctaxitrips/). En [Descripción del conjunto de datos de carreras de taxi de Nueva York](sql-walkthrough.md#dataset), se proporciona un ejemplo y una descripción de estos archivos.

Puede adaptar los procedimientos que se describen aquí para un conjunto de datos propios o seguir los pasos descritos para el uso del conjunto de datos de taxis de Nueva York. Para cargar el conjunto de datos de taxis de Nueva York en la base de datos de SQL Server, siga el procedimiento descrito en [Importación masiva de datos en una base de datos de SQL Server](sql-walkthrough.md#dbload).

## <a name="moving-data-from-a-flat-file-source-to-an-azure-sql-database"></a><a name="file-to-azure-sql-database"></a> Traslado de datos desde un origen de archivo plano a una instancia de Azure SQL Database
Los datos de archivos planos (formatos CSV o TSV) se pueden mover a una base de datos de Azure SQL Database mediante una consulta SQL de inserción masiva.

### <a name="bulk-insert-sql-query"></a><a name="bulk-insert-sql-query"></a> Consulta SQL de inserción masiva
Los pasos del procedimiento en el que se usa la consulta SQL de inserción masiva son similares a los del traslado de datos desde un origen de archivo plano a un servidor SQL Server en una máquina virtual de Azure. Para detalles, vea [Consulta SQL de inserción masiva](move-sql-server-virtual-machine.md#insert-tables-bulkquery).

## <a name="moving-data-from-sql-server-to-an-azure-sql-database"></a><a name="sql-on-prem-to-sazure-sql-database">Traslado de datos desde un servidor con SQL Server a una base de datos de Azure SQL Database</a>
Si los datos de origen están almacenados en SQL Server, hay varias alternativas para mover los datos a una base de datos de Azure SQL Database:

1. [Exportación a un archivo plano](#export-flat-file)
2. [SQL Database Migration Wizard](#insert-tables-bcp)
3. [Copia de seguridad y restauración de una base de datos](#db-migration)
4. [Azure Data Factory](#adf)

Los pasos para las tres primeras opciones son similares a los de la sección [Mover datos a un servidor SQL Server en una máquina virtual de Azure](move-sql-server-virtual-machine.md), en la que se explican los mismos procedimientos. En las instrucciones siguientes se proporcionan vínculos a las secciones correspondientes de ese tema.

### <a name="export-to-flat-file"></a><a name="export-flat-file"></a>Exportación a un archivo plano
Los pasos para esta exportación a un archivo plano son similares a los que se explican en [Exportación a un archivo plano](move-sql-server-virtual-machine.md#export-flat-file).

### <a name="sql-database-migration-wizard"></a><a name="insert-tables-bcp"></a>SQL Database Migration Wizard
Los pasos para usar el Asistente para migración de SQL Database son similares a los que se explican en [Asistente para migración de SQL Database](move-sql-server-virtual-machine.md#sql-migration).

### <a name="database-back-up-and-restore"></a><a name="db-migration"></a>Copia de seguridad y restauración de una base de datos
Los pasos para usar la copia de seguridad y restauración de la base de datos son similares a los que se explican en [Copia de seguridad y restauración de la base de datos](move-sql-server-virtual-machine.md#sql-backup).

### <a name="azure-data-factory"></a><a name="adf"></a>Azure Data Factory
Aprenda a mover datos a una base de datos de Azure SQL Database con Azure Data Factory (ADF) en este tema, [Movimiento de datos desde un servidor con SQL Server hasta SQL Azure con Azure Data Factory](move-sql-azure-adf.md). En este tema se muestra cómo usar ADF para mover datos desde una base de datos de SQL Server a una base de datos de Azure SQL Database a través de Azure Blob Storage.

Considere la posibilidad de usar ADF cuando los datos deban migrarse continuamente con orígenes híbridos locales y en la nube.  ADF también resulta útil si los datos necesitan transformaciones o si requiere una nueva lógica de negocios durante la migración. La ADF permite la programación y supervisión de trabajos mediante scripts JSON sencillos que administran el movimiento de datos de forma periódica. La ADF también tiene otras capacidades como la compatibilidad con operaciones complejas.
