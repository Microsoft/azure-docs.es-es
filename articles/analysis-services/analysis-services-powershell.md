---
title: Administración de Azure Analysis Services con PowerShell | Microsoft Docs
description: Describe los cmdlets de PowerShell de Azure Analysis Services para tareas administrativas comunes, como la creación de servidores, la suspensión de operaciones o el cambio del nivel de servicio.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 05/19/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 3e64ffe5007d27a44167f08807a9694875fe48c4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050449"
---
# <a name="manage-azure-analysis-services-with-powershell"></a>Administración de Azure Analysis Services con PowerShell

En este artículo se describen los cmdlets de PowerShell que se usan para realizar tareas de administración de bases de datos y del servidor de Azure Analysis Services. 

Las tareas de administración del servidor como crear o eliminar un servidor, suspender o reanudar las operaciones del servidor o cambiar el nivel de servicio, usan cmdlets de Azure Analysis Services. Otras tareas de administración de bases de datos, como la incorporación y eliminación de miembros de roles, el procesamiento o la creación de particiones, usan los cmdlets incluidos en el mismo módulo de SqlServer que SQL Server Analysis Services.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="permissions"></a>Permisos

Para la mayoría de las tareas de PowerShell, es necesario disponer de privilegios de administración en el servidor de Analysis Services que esté administrando. Las tareas programadas de PowerShell son operaciones desatendidas. La cuenta o entidad de servicio que ejecuta Scheduler debe tener privilegios de administrador en el servidor de Analysis Services. 

En el caso de operaciones de servidor mediante cmdlets de Azure PowerShell, su cuenta o la cuenta que ejecuta Scheduler también deben pertenecer al rol de propietario del recurso en el [control de acceso basado en rol (RBAC) de Azure](../role-based-access-control/overview.md). 

## <a name="resource-and-server-operations"></a>Recursos y operaciones del servidor 

Instalación del módulo: [Az.AnalysisServices](https://www.powershellgallery.com/packages/Az.AnalysisServices)   
Documentación: [referencia de Az.AnalysisServices](/powershell/module/az.analysisservices)

## <a name="database-operations"></a>Operaciones de la base de datos

Las operaciones de la base de datos de Azure Analysis Services utilizan el mismo módulo SqlServer que SQL Server Analysis Services. Pero no todos los cmdlets son compatibles con Azure Analysis Services. 

El módulo SqlServer proporciona cmdlets de administración de base de datos específicos de la tarea, así como el cmdlet de uso general Invoke-ASCmd que acepta una consulta o script de Tabular Model Scripting Language (TMSL). Los siguientes cmdlets del módulo SqlServer son compatibles con Azure Analysis Services.

Instalación del módulo: [SqlServer](https://www.powershellgallery.com/packages/SqlServer)   
Documentation: [referencia de SqlServer](/powershell/module/sqlserver)

### <a name="supported-cmdlets"></a>Cmdlets compatibles

|Cmdlet|Descripción|
|------------|-----------------| 
|[Add-RoleMember](https://docs.microsoft.com/powershell/module/sqlserver/Add-RoleMember)|Agrega un miembro a un rol de base de datos.| 
|[Backup-ASDatabase](https://docs.microsoft.com/powershell/module/sqlserver/backup-asdatabase)|Realiza una copia de seguridad de una base de datos de Analysis Services.|  
|[Remove-RoleMember](https://docs.microsoft.com/powershell/module/sqlserver/remove-rolemember)|Quita un miembro de un rol de base de datos.|   
|[Invoke-ASCmd](https://docs.microsoft.com/powershell/module/sqlserver/invoke-ascmd)|Ejecuta un script de TMSL.|
|[Invoke-ProcessASDatabase](https://docs.microsoft.com/powershell/module/sqlserver/invoke-processasdatabase)|Procesa una base de datos.|  
|[Invoke-ProcessPartition](https://docs.microsoft.com/powershell/module/sqlserver/invoke-processpartition)|Procesa una partición.| 
|[Invoke-ProcessTable](https://docs.microsoft.com/powershell/module/sqlserver/invoke-processtable)|Procesa una tabla.|  
|[Merge-Partition](https://docs.microsoft.com/powershell/module/sqlserver/merge-partition)|Combina una partición.|  
|[Restore-ASDatabase](https://docs.microsoft.com/powershell/module/sqlserver/restore-asdatabase)|Restaura una base de datos de Analysis Services.| 
  

## <a name="related-information"></a>Información relacionada

* [SQL Server PowerShell](https://docs.microsoft.com/sql/powershell/sql-server-powershell)      
* [Descarga del módulo de PowerShell de SQL Server](https://docs.microsoft.com/sql/ssms/download-sql-server-ps-module)   
* [Descarga de SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)   
* [Módulo de SqlServer en Galería de PowerShell](https://www.powershellgallery.com/packages/SqlServer)    
* [Programación de modelo tabular para el nivel de compatibilidad 1200 y superior](https://docs.microsoft.com/analysis-services/tabular-model-programming-compatibility-level-1200/tabular-model-programming-for-compatibility-level-1200)
