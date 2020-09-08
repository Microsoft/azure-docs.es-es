---
title: Cmdlets de Azure AD PowerShell para informes | Microsoft Docs
description: Referencia de los cmdlets de Azure AD PowerShell para informes
services: active-directory
documentationcenter: ''
author: MarkusVi
manager: daveba
editor: ''
ms.assetid: a1f93126-77d1-4345-ab7d-561066041161
ms.service: active-directory
ms.devlang: na
ms.topic: reference
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: report-monitor
ms.date: 08/07/2020
ms.author: markvi
ms.reviewer: dhanyahk
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 4353ed5413f76b13425a59d31bb6108542e3bd23
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89231136"
---
# <a name="azure-ad-powershell-cmdlets-for-reporting"></a>Cmdlets de Azure AD PowerShell para informes

> [!NOTE] 
> En la actualidad, estos cmdlets de PowerShell solo funcionan con el módulo de la [versión preliminar de Azure AD](/powershell/module/azuread/?view=azureadps-2.0-preview#directory_auditing). Tenga en cuenta que no se recomienda utilizar el módulo de la versión preliminar para producción. 

Para instalar la versión preliminar pública, use lo siguiente. 

```powershell
Install-module AzureADPreview
```

Para más información sobre cómo conectarse a Azure AD con PowerShell, consulte el artículo [Azure AD PowerShell for Graph](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0).  

Con los informes de Azure Active Directory (Azure AD), puede obtener detalles sobre las actividades de todas las operaciones de escritura en su dirección (registros de auditoría) y los datos de autenticación (registros de inicio de sesión). Aunque la información está disponible mediante MS Graph API, ahora puede recuperar los mismos datos usando los cmdlets de PowerShell de Azure AD para la generación de informes.

En este artículo se ofrece información general sobre los cmdlets de PowerShell que se usan para generar los registros de auditoría y los registros de inicio de sesión.

## <a name="audit-logs"></a>Registros de auditoría

Los [registros de auditoría](concept-audit-logs.md) proporcionan rastreabilidad mediante los registros de todos los cambios realizados por diversas características de Azure AD. Algunos ejemplos de registros de auditoría incluyen los cambios realizados en cualquier recurso de Azure AD, como agregar o quitar usuarios, aplicaciones, grupos, roles y directivas.

Puede acceder a los registros de auditoría con el cmdlet "Get-AzureADAuditDirectoryLogs".


| Escenario                      | Comando de PowerShell |
| :--                           | :--                |
| Nombre para mostrar de la aplicación      | Get-AzureADAuditDirectoryLogs -Filter "initiatedBy/app/displayName eq 'Azure AD Cloud Sync'" |
| Category                      | Get-AzureADAuditDirectoryLogs -Filter "category eq 'ApplicationManagement'" |
| Fecha y hora de la actividad            | Get-AzureADAuditDirectoryLogs -Filter "activityDateTime gt 2019-04-18" |
| Todo lo anterior              | Get-AzureADAuditDirectoryLogs -Filter "initiatedBy/app/displayName eq 'Azure AD Cloud Sync' and category eq 'ApplicationManagement' and activityDateTime gt 2019-04-18"|


En la imagen siguiente se muestra un ejemplo de este comando. 

![Botón "Data Summary" (Resumen de datos)](./media/reference-powershell-reporting/get-azureadauditdirectorylogs.png)



## <a name="sign-in-logs"></a>Registros de inicio de sesión

Los registros de [inicio de sesión](concept-sign-ins.md) proporcionan información sobre el uso de aplicaciones administradas y actividades de inicio de sesión de usuario.

Puede acceder a los registros de inicio de sesión con el cmdlet "Get-AzureADAuditSignInLogs".


| Escenario                      | Comando de PowerShell |
| :--                           | :--                |
| Nombre para mostrar del usuario             | Get-AzureADAuditSignInLogs -Filter "userDisplayName eq 'Timothy Perkins'" |
| Fecha y hora de creación              | Get-AzureADAuditSignInLogs -Filter "createdDateTime gt 2019-04-18T17:30:00.0Z" (Everything since 5:30 pm on 4/18) |
| Estado                        | Get-AzureADAuditSignInLogs -Filter "status/errorCode eq 50105" |
| Nombre para mostrar de la aplicación      | Get-AzureADAuditSignInLogs -Filter "appDisplayName eq 'StoreFrontStudio [wsfed enabled]'" |
| Todo lo anterior              | Get-AzureADAuditSignInLogs -Filter "userDisplayName eq 'Timothy Perkins' and status/errorCode ne 0 and appDisplayName eq 'StoreFrontStudio [wsfed enabled]'" |


En la imagen siguiente se muestra un ejemplo de este comando. 

![Botón "Data Summary" (Resumen de datos)](./media/reference-powershell-reporting/get-azureadauditsigninlogs.png)



## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre informes de Azure AD](overview-reports.md).
- [Informe de registros de auditoría](concept-audit-logs.md) 
- [Acceso mediante programación a los informes de Azure AD](concept-reporting-api.md)