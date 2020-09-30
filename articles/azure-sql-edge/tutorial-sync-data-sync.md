---
title: Sincronización de datos desde Azure SQL Edge mediante SQL Data Sync
description: Información sobre la sincronización de datos desde Azure SQL Edge mediante Azure SQL Data Sync
keywords: SQL Edge, sincronización de datos desde SQL Edge, sincronización de datos de SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: tutorial
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 5830cedfdce54c4cd91eb60a5cd2145309e965b0
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904107"
---
# <a name="tutorial-sync-data-from-sql-edge-to-azure-sql-database-by-using-sql-data-sync"></a>Tutorial: Sincronización de datos desde SQL Edge a Azure SQL Database mediante SQL Data Sync

En este tutorial, aprenderá a usar un *grupo de sincronización* de Azure SQL Data Sync para sincronizar incrementalmente los datos de Azure SQL Edge con Azure SQL Database. SQL Data Sync es un servicio basado en Azure SQL Database que permite sincronizar los datos seleccionados de manera bidireccional entre varias bases de datos en instancias de SQL Server y Azure SQL Database. Para más información sobre SQL Data Sync, consulte [Azure SQL Data Sync](../azure-sql/database/sql-data-sync-data-sql-server-sql-database.md).

Dado que SQL Edge se basa en las versiones más recientes del [Motor de base de datos de SQL Server](/sql/sql-server/sql-server-technical-documentation/), cualquier mecanismo de sincronización de datos que sea aplicable a una instancia de SQL Server también se puede usar para sincronizar datos con la instancia de SQL Edge que se ejecute en un dispositivo perimetral.

## <a name="prerequisites"></a>Requisitos previos

Para este tutorial se requiere un equipo Windows configurado con [Data Sync Agent para Azure SQL Data Sync](../azure-sql/database/sql-data-sync-agent-overview.md).

## <a name="before-you-begin"></a>Antes de empezar

* Cree una base de datos en Azure SQL Database. Para información sobre cómo crear una base de datos desde Azure Portal, consulte [Creación de una base de datos única en Azure SQL Database](../azure-sql/database/single-database-create-quickstart.md?tabs=azure-portal).

* Cree las tablas y otros objetos necesarios en la implementación de Azure SQL Database.

* Cree las tablas y objetos necesarios en la implementación de Azure SQL Edge. Para más información, vea [Uso de paquetes DAC de SQL Database con SQL Edge](deploy-dacpac.md).

* Registre la instancia de Azure SQL Edge con Data Sync Agent para Azure SQL Data Sync. Para más información, consulte [Incorporación de una base de datos de SQL Server](../azure-sql/database/sql-data-sync-sql-server-configure.md#add-on-prem).

## <a name="sync-data-between-a-database-in-azure-sql-database-and-sql-edge"></a>Sincronización de datos entre una base de datos de Azure SQL Database y SQL Edge

La configuración de la sincronización entre una base de datos de Azure SQL Database y una instancia de SQL Edge mediante SQL Data Sync implica tres pasos clave:  


1. Use Azure Portal para crear un grupo de sincronización. Para más información, consulte [Creación de un área de trabajo](../azure-sql/database/sql-data-sync-sql-server-configure.md#create-sync-group). Puede usar una sola base de datos *centro* para crear varios grupos de sincronización para sincronizar datos de distintas instancias de SQL Edge con una o varias bases de datos de Azure SQL Database. 

2. Agregue miembros de sincronización al grupo de sincronización. Para más información, consulte [Adición de miembros de sincronización](../azure-sql/database/sql-data-sync-sql-server-configure.md#add-sync-members).

3. Configure el grupo de sincronización para seleccionar las tablas que formarán parte de la sincronización. Para más información, consulte [Configuración de un grupo de sincronización](../azure-sql/database/sql-data-sync-sql-server-configure.md#add-sync-members).

Después de completar los pasos anteriores, tendrá un grupo de sincronización que incluye una base de datos en Azure SQL Database y una instancia de SQL Edge.

Para más información sobre SQL Data Sync, consulte estos artículos:

* [Data Sync Agent para Azure SQL Data Sync](../azure-sql/database/sql-data-sync-agent-overview.md)

* [Procedimientos recomendados](../azure-sql/database/sql-data-sync-best-practices.md) y [Solución de problemas de Azure SQL Data Sync](../azure-sql/database/sql-data-sync-troubleshoot.md)

* [Monitor SQL Data Sync with Azure Monitor logs](../azure-sql/database/sql-data-sync-monitor-sync.md) (Supervisión de SQL Data Sync con registros de Azure Monitor)

* [Actualización del esquema de sincronización con Transact-SQL](../azure-sql/database/sql-data-sync-update-sync-schema.md) o [PowerShell](../azure-sql/database/scripts/update-sync-schema-in-sync-group.md)

## <a name="next-steps"></a>Pasos siguientes


* [Uso de PowerShell para sincronizar entre Azure SQL Database y Azure SQL Edge](../azure-sql/database/scripts/sql-data-sync-sync-data-between-azure-onprem.md). En el tutorial, reemplace los detalles de la base de datos `OnPremiseServer` por los detalles de Azure SQL Edge.
