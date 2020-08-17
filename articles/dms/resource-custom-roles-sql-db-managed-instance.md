---
title: 'Roles personalizados: Migraciones de SQL Server en línea a instancias administradas de SQL'
titleSuffix: Azure Database Migration Service
description: Obtenga información sobre cómo usar roles personalizados para las migraciones en línea de SQL Server a una instancia administrada de Azure SQL.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: article
ms.date: 10/25/2019
ms.openlocfilehash: 6d720eee668edf2de968f0ce6955a5a586a92419
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087686"
---
# <a name="custom-roles-for-sql-server-to-azure-sql-managed-instance-online-migrations"></a>Roles personalizados para migraciones en línea de SQL Server a una instancia administrada de Azure SQL

Azure Database Migration Service usa un id. de aplicación para interactuar con los servicios de Azure. El id. de aplicación requiere el rol de colaborador en el nivel de suscripción (que muchos departamentos de seguridad corporativos no permitirán) o la creación de roles personalizados que concedan los permisos específicos que el servicio Azure Database Migrations requiere. Dado que hay un límite de 2000 roles personalizados en Azure Active Directory, es aconsejable combinar todos los permisos que requiere específicamente el id. de aplicación en uno o dos roles personalizados y, a continuación, conceder al id. de aplicación el rol personalizado en objetos o grupos de recursos específicos (frente al nivel de suscripción). Si el número de roles personalizados no es un problema, puede dividir los roles personalizados por tipo de recurso para crear tres roles personalizados en total, como se describe a continuación.

La sección AssignableScopes de la cadena JSON de definición de roles permite controlar dónde aparecen los permisos en la interfaz de usuario **Agregar asignación de roles** del portal. Probablemente querrá definir el rol en el grupo de recursos o incluso en el nivel de recurso para evitar la saturación de la interfaz de usuario con roles adicionales. Tenga en cuenta que esto no realiza la asignación de roles real.

## <a name="minimum-number-of-roles"></a>Número mínimo de roles

Actualmente se recomienda crear un mínimo de dos roles personalizados para el id. de aplicación, uno en el nivel de recurso y el otro en el nivel de suscripción.

> [!NOTE]
> Es posible que con el tiempo se quite el último requisito de rol personalizado cuando se implemente nuevo código de instancia administrada de SQL en Azure.

**Rol personalizado para el id. de aplicación**. Este rol es necesario para la migración de Azure Database Migration Service en el nivel de *recurso* o *grupo de recursos* (para más información sobre el id. de aplicación, consulte el artículo [Uso del portal para crear una aplicación de Azure AD y una entidad de servicio con acceso a los recursos](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal)).

```json
{
  "Name": "DMS Role - App ID",
  "IsCustom": true,
  "Description": "DMS App ID access to complete MI migrations",
  "Actions": [
        "Microsoft.Storage/storageAccounts/read",
        "Microsoft.Storage/storageAccounts/listKeys/action",
        "Microsoft.Storage/storageaccounts/blobservices/read",
        "Microsoft.Storage/storageaccounts/blobservices/write",
        "Microsoft.Sql/managedInstances/read",
        "Microsoft.Sql/managedInstances/write",
        "Microsoft.Sql/managedInstances/databases/read",
        "Microsoft.Sql/managedInstances/databases/write",
        "Microsoft.Sql/managedInstances/databases/delete",
        "Microsoft.Sql/managedInstances/metrics/read",
        "Microsoft.DataMigration/locations/*",
        "Microsoft.DataMigration/services/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>/ResourceGroups/<StorageAccount_rg_name>",
    "/subscriptions/<subscription_id>/ResourceGroups/<ManagedInstance_rg_name>",
    "/subscriptions/<subscription_id>/ResourceGroups/<DMS_rg_name>",
  ]
}
```

**Rol personalizado para el id. de aplicación: suscripción**. Este rol es necesario para la migración de Azure Database Migration Service en el nivel de *suscripción*.

```json
{
  "Name": "DMS Role - App ID - Sub",
  "IsCustom": true,
  "Description": "DMS App ID access at subscription level to complete MI migrations",
  "Actions": [
        "Microsoft.Sql/locations/managedDatabaseRestoreAzureAsyncOperation/*"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>"
  ]
}
```

El código JSON anterior debe almacenarse en tres archivos de texto y puede usar los cmdlets AzureRM y AZ PowerShell o la CLI de Azure para crear los roles mediante **New-AzureRmRoleDefinition (AzureRM)** o **New-AzRoleDefinition (AZ)** .

Para más información, consulte el artículo [Roles personalizados de Azure](https://docs.microsoft.com/azure/role-based-access-control/custom-roles).

Después de crear estos roles personalizados, debe agregar las asignaciones de roles a los usuarios y los id. de aplicación a los recursos o grupos de recursos adecuados:

* Se debe conceder el rol "DMS Role - App ID" al id. de aplicación que se usará para las migraciones, y también en la cuenta de almacenamiento, la instancia de Azure Database Migration Service y los niveles de recurso de instancia administrada de SQL.
* El rol "DMS Role - App ID - Sub" debe concederse al id. de aplicación en el nivel de suscripción (se producirá un error si se concede en el recurso o grupo de recursos). Este requisito es temporal hasta que se implemente una actualización de código.

## <a name="expanded-number-of-roles"></a>Número expandido de roles

Si el número de roles personalizados en Azure Active Directory no es un problema, se recomienda crear un total de tres roles. Seguirá necesitando el rol "DMS Role - App ID – Sub", pero el rol "DMS Role - App ID" anterior se divide por tipo de recurso en dos roles diferentes.

**Rol personalizado para el id. de aplicación de la instancia administrada de SQL**

```json
{
  "Name": "DMS Role - App ID - SQL MI",
  "IsCustom": true,
  "Description": "DMS App ID access to complete MI migrations",
  "Actions": [
        "Microsoft.Sql/managedInstances/read",
        "Microsoft.Sql/managedInstances/write",
        "Microsoft.Sql/managedInstances/databases/read",
        "Microsoft.Sql/managedInstances/databases/write",
        "Microsoft.Sql/managedInstances/databases/delete",
        "Microsoft.Sql/managedInstances/metrics/read"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>/resourceGroups/<ManagedInstance_rg_name>"
  ]
}
```

**Rol personalizado para el id. de aplicación de almacenamiento**

```json
{
  "Name": "DMS Role - App ID - Storage",
  "IsCustom": true,
  "Description": "DMS App ID storage access to complete MI migrations",
  "Actions": [
"Microsoft.Storage/storageAccounts/read",
        "Microsoft.Storage/storageAccounts/listKeys/action",
        "Microsoft.Storage/storageaccounts/blobservices/read",
        "Microsoft.Storage/storageaccounts/blobservices/write"
  ],
  "NotActions": [
  ],
  "AssignableScopes": [
    "/subscriptions/<subscription_id>/resourceGroups/<StorageAccount_rg_name>"
  ]
}
```

## <a name="role-assignment"></a>Asignación de roles

Para asignar un rol a los usuarios o al id. de aplicación, abra Azure Portal y siga estos pasos:

1. Navegue hasta el grupo de recursos o el recurso (excepto para el rol que debe concederse en la suscripción), vaya a **Access Control** y desplácese para buscar los roles personalizados que acaba de crear.

2. Seleccione el rol adecuado, seleccione el id. de aplicación y, a continuación, guarde los cambios.

  Los id. de aplicación aparecen ahora en la pestaña **Asignaciones de roles**.

## <a name="next-steps"></a>Pasos siguientes

* Revise las directrices de migración para su escenario en la [Guía de migración de bases de datos](https://datamigration.microsoft.com/) de Microsoft.
