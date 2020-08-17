---
title: Cifrado de datos transparente (Portal)
description: Cifrado de datos transparente (TDE) en Azure Synapse Analytics
services: synapse-analytics
author: julieMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/30/2019
ms.author: jrasnick
ms.reviewer: rortloff
ms.custom: seo-lt-2019
ms.openlocfilehash: b530b3a049f41a54ab98cc7d1454018cfc990f75
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86495676"
---
# <a name="get-started-with-transparent-data-encryption-tde-in-azure-synapse-analytics"></a>Introducción al cifrado de datos transparente (TDE) en Azure Synapse Analytics

> [!div class="op_single_selector"]
>
> * [Información general sobre seguridad](sql-data-warehouse-overview-manage-security.md)
> * [Autenticación](sql-data-warehouse-authentication.md)
> * [Cifrado (Portal)](sql-data-warehouse-encryption-tde.md)
> * [Cifrado (T-SQL)](sql-data-warehouse-encryption-tde-tsql.md)

## <a name="required-permissions"></a>Permisos necesarios

Para habilitar el Cifrado de datos transparente (TDE), debe ser un administrador o un miembro del rol dbmanager.

## <a name="enabling-encryption"></a>Habilitar el cifrado

Para habilitar TDE, siga los pasos que aparecen a continuación:

1. Abra la base de datos en el [Portal de Azure](https://portal.azure.com)
2. En la hoja de la base de datos, haga clic en el botón **Configuración**
3. Seleccione la opción **Cifrado de datos transparente**.![configuración del portal](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings.png)
4. Seleccione la opción **Activado**.![configuración del portal: activado](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-on.png)
5. Seleccione **Guardar**.
   ![configuración del portal: guardar](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save.png)  

## <a name="disabling-encryption"></a>Deshabilitar el cifrado

Para deshabilitar TDE, siga los pasos que aparecen a continuación:

1. Abra la base de datos en el [Portal de Azure](https://portal.azure.com)
2. En la hoja de la base de datos, haga clic en el botón **Configuración**
3. Seleccione la opción **Cifrado de datos transparente**.![configuración del portal](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings.png)
4. Seleccione la opción **Desactivado**.![configuración del portal: desactivado](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-off.png)
5. Seleccione **Guardar**.
   ![configuración del portal: guardar 2](./media/sql-data-warehouse-security-tde/sql-data-warehouse-security-tde-portal-settings-save2.png)  

## <a name="encryption-dmvs"></a>DMV de cifrado

El cifrado se puede confirmar con las siguientes DMV:

* [sys.databases](/sql/relational-databases/system-catalog-views/sys-databases-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
* [sys.dm_pdw_nodes_database_encryption_keys](/sql/relational-databases/system-dynamic-management-views/sys-dm-pdw-nodes-database-encryption-keys-transact-sql?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json&view=azure-sqldw-latest)
