---
title: Incorporación de una entidad de servicio al rol de administrador de Azure Analysis Services | Microsoft Docs
description: Aprenda cómo agregar una entidad de servicio de automatización al rol de administrador del servidor de Azure Analysis Services
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 07/07/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: fasttrack-edit
ms.openlocfilehash: ce8f7347e4813e72ede426ab17f09221ab859136
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87015415"
---
# <a name="add-a-service-principal-to-the-server-administrator-role"></a>Incorporación de una entidad de servicio al rol de administrador del servidor 

 Para automatizar las tareas de PowerShell desatendidas, una entidad de servicio debe tener privilegios de **administrador del servidor** en el servidor de Analysis Services que se está administrando. En este artículo se describe cómo agregar una entidad de servicio al rol de administrador de servidor en un servidor de Azure AS. Puede hacerlo mediante SQL Server Management Studio o una plantilla de Resource Manager. 

> [!NOTE]
> Las entidades de servicio deben agregarse directamente al rol Administrador del servidor. La incorporación de una entidad de servicio a un grupo de seguridad y, posteriormente, de ese grupo de seguridad al rol Administrador del servidor, no se admite. 

## <a name="before-you-begin"></a>Antes de empezar
Antes de completar esta tarea, debe tener registrada una entidad de servicio en Azure Active Directory.

[Creación de una entidad de servicio: Azure Portal](../active-directory/develop/howto-create-service-principal-portal.md)   
[Creación de una entidad de servicio: PowerShell](../active-directory/develop/howto-authenticate-service-principal-powershell.md)

## <a name="using-sql-server-management-studio"></a>Uso de SQL Server Management Studio

Puede configurar administradores de servidor mediante SQL Server Management Studio (SSMS). Para completar esta tarea, debe tener permisos de [administrador del servidor](analysis-services-server-admins.md) en el servidor Azure AS. 

1. En SSMS, conéctese a su servidor Azure AS.
2. En **Propiedades del servidor** > **Seguridad**, haga clic en **Agregar**.
3. En **Seleccione un usuario o grupo**, busque la aplicación registrada por su nombre, selecciónela y, a continuación, haga clic en **Agregar**.

    ![Búsqueda de la cuenta de la entidad de servicio](./media/analysis-services-addservprinc-admins/aas-add-sp-ssms-picker.png)

4. Compruebe el identificador de la cuenta de la entidad de servicio y, a continuación, haga clic en **Aceptar**.
    
    ![Búsqueda de la cuenta de la entidad de servicio](./media/analysis-services-addservprinc-admins/aas-add-sp-ssms-add.png)

## <a name="using-a-resource-manager-template"></a>Uso de una plantilla de Resource Manager

También puede configurar administradores de servidor si implementa el servidor de Analysis Services mediante una plantilla de Azure Resource Manager. La identidad que ejecuta la implementación debe pertenecer al rol **Colaborador** del recurso en el [control de acceso basado en rol (RBAC) de Azure](../role-based-access-control/overview.md).

> [!IMPORTANT]
> La entidad de servicio se debe agregar con el formato `app:{service-principal-client-id}@{azure-ad-tenant-id}`.

La siguiente plantilla de Resource Manager implementa un servidor de Analysis Services con una entidad de servicio especificada agregada al rol Administrador de Analysis Services:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "analysisServicesServerName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "analysisServicesSkuName": {
            "type": "string"
        },
        "analysisServicesCapacity": {
            "type": "int"
        },
        "servicePrincipalClientId": {
            "type": "string"
        },
        "servicePrincipalTenantId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('analysisServicesServerName')]",
            "type": "Microsoft.AnalysisServices/servers",
            "apiVersion": "2017-08-01",
            "location": "[parameters('location')]",
            "sku": {
                "name": "[parameters('analysisServicesSkuName')]",
                "capacity": "[parameters('analysisServicesCapacity')]"
            },
            "properties": {
                "asAdministrators": {
                    "members": [
                        "[concat('app:', parameters('servicePrincipalClientId'), '@', parameters('servicePrincipalTenantId'))]"
                    ]
                }
            }
        }
    ]
}
```

## <a name="using-managed-identities"></a>Uso de identidades administradas

También se puede agregar una identidad administrada a la lista de administradores de Analysis Services. Por ejemplo, puede tener una [aplicación lógica con una identidad administrada asignada por el sistema](../logic-apps/create-managed-service-identity.md) y querer concederle la capacidad de administrar su servidor de Analysis Services.

En la mayoría de las partes de Azure Portal y las API, las identidades administradas se identifican mediante el identificador de objeto de la entidad de servicio. Sin embargo, Analysis Services requiere que se identifiquen mediante su identificador de cliente. Para obtener el identificador de cliente de una entidad de servicio, puede usar la CLI de Azure:

```bash
az ad sp show --id <ManagedIdentityServicePrincipalObjectId> --query appId -o tsv
```

También puede usar la PowerShell:

```powershell
(Get-AzureADServicePrincipal -ObjectId <ManagedIdentityServicePrincipalObjectId>).AppId
```

A continuación, puede usar este identificador de cliente junto con el identificador de inquilino para agregar la identidad administrada a la lista de administradores de Analysis Services, como se describe más arriba.

## <a name="related-information"></a>Información relacionada

* [Descarga del módulo de PowerShell de SQL Server](https://docs.microsoft.com/sql/ssms/download-sql-server-ps-module)   
* [Descarga de SSMS](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms)   
