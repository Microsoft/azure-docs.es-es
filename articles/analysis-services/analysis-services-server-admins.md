---
title: Administración de administradores de servidor en Azure Analysis Services | Microsoft Docs
description: En este artículo se describe cómo administrar los administradores de servidor para un servidor de Azure Analysis Services mediante Azure Portal, PowerShell o las API REST.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: bc17d27837d5b96f06b5172fb019db873418db94
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87922956"
---
# <a name="manage-server-administrators"></a>Administración de administradores de servidor

Los administradores del servidor deben ser un usuario, entidad de servicio o grupo de seguridad válido en Azure Active Directory (Azure AD) para el inquilino en el que reside el servidor. Puede usar **Administradores de Analysis Services** en el servidor de Azure Portal, Propiedades del servidor en SSMS, PowerShell o la API de REST para administrar los administradores del servidor. 

Al agregar un **grupo de seguridad**, use `obj:groupid@tenantid`. Las entidades de servicio no se admiten en los grupos de seguridad que se agregan al rol de administrador del servidor.

## <a name="to-add-server-administrators-by-using-azure-portal"></a>Incorporación de administradores de servidor mediante Azure Portal

1. En el portal, en su servidor, haga clic en **Administradores de Analysis Services**.
2. En **\<servername> - Administradores de Analysis Services**, haga clic en **Agregar**.
3. En **Agregar administradores de servidor**, seleccione las cuentas de usuario de Azure AD o invite usuarios externos por dirección de correo electrónico.

    ![Administradores del servidor en Azure Portal](./media/analysis-services-server-admins/aas-manage-users-admins.png)

## <a name="to-add-server-administrators-by-using-ssms"></a>Incorporación de administradores de servidor mediante SSMS

1. Haga clic con el botón derecho en el servidor > **Propiedades**.
2. En **Propiedades de Analysis Server**, haga clic en **Seguridad**.
3. Haga clic en **Agregar** y escriba la dirección de correo electrónico de un usuario o grupo de Azure AD.
   
    ![Incorporación de administradores de servidor en SSMS](./media/analysis-services-server-admins/aas-manage-users-ssms.png)

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Use el cmdlet [New-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/new-azanalysisservicesserver) para especificar el parámetro de administrador al crear un servidor. <br>
Use el cmdlet [Set-AzAnalysisServicesServer](https://docs.microsoft.com/powershell/module/az.analysisservices/set-azanalysisservicesserver) para modificar el parámetro de administrador de un servidor existente.

## <a name="rest-api"></a>API DE REST

Use [Crear](https://docs.microsoft.com/rest/api/analysisservices/servers/create) para especificar la propiedad asAdministrator al crear un servidor. <br>
Use [Actualizar](https://docs.microsoft.com/rest/api/analysisservices/servers/update) para especificar la propiedad asAdministrator al modificar un servidor existente. <br>



## <a name="next-steps"></a>Pasos siguientes 

[Permisos de usuario y autenticación](analysis-services-manage-users.md)  
[Administración de usuarios y roles de base de datos](analysis-services-database-users.md)  
[Control de acceso basado en roles de Azure (Azure RBAC)](../role-based-access-control/overview.md)  
