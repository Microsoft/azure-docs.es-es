---
title: Auditorías en una cuenta de almacenamiento detrás de una red virtual y un firewall
description: Configure auditorías para escribir eventos de base de datos en una cuenta de almacenamiento detrás de una red virtual y un firewall.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.topic: conceptual
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 06/17/2020
ms.custom: azure-synapse
ms.openlocfilehash: 6ba0a599bcb0b058ce4902882df9459b177fb6b5
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87530421"
---
# <a name="write-audit-to-a-storage-account-behind-vnet-and-firewall"></a>Escritura de auditorías en una cuenta de almacenamiento detrás de una red virtual y un firewall
[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]


La auditoría de [Azure SQL Database](sql-database-paas-overview.md) y [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) admite la escritura de eventos de base de datos en una [cuenta de Azure Storage](../../storage/common/storage-account-overview.md) detrás de un firewall y una red virtual.

En este artículo se explican dos maneras de configurar Azure SQL Database y la cuenta de almacenamiento de Azure para admitir esta opción. La primera usa Azure Portal y la segunda usa REST.

## <a name="background"></a>Información previa

[Azure Virtual Network (VNet)](../../virtual-network/virtual-networks-overview.md) es el bloque de creación fundamental de una red privada en Azure. VNet permite muchos tipos de recursos de Azure, como Azure Virtual Machines (máquinas virtuales), para comunicarse de forma segura entre usuarios, con Internet y con las redes locales. VNet es similar a una red tradicional en su propio centro de datos, pero aporta las ventajas adicionales de la infraestructura de Azure, como la escala, la disponibilidad y el aislamiento.

Consulte el artículo [¿Qué es Azure Virtual Network?](../../virtual-network/virtual-networks-overview.md) para obtener más información sobre los conceptos de VNet.

Para obtener más información sobre cómo crear una red virtual, vea [Inicio rápido: Creación de una red virtual mediante Azure Portal](../../virtual-network/quick-create-portal.md).

## <a name="prerequisites"></a>Requisitos previos

Para que una auditoría escriba eventos en una cuenta de almacenamiento detrás de un firewall o una red virtual, se necesitan los siguientes requisitos previos:

> [!div class="checklist"]
>
> * Una cuenta de almacenamiento de uso general v2. Si tiene una cuenta de almacenamiento de uso general v1 o una cuenta de almacenamiento de blobs, [actualice a una cuenta de almacenamiento de uso general v2](../../storage/common/storage-account-upgrade.md). Para obtener más información, consulte la sección [Tipos de cuentas de almacenamiento](../../storage/common/storage-account-overview.md#types-of-storage-accounts).
> * La cuenta de almacenamiento debe estar en la misma suscripción y en la misma ubicación que el [servidor SQL lógico](logical-servers.md).
> * La cuenta de Azure Storage requiere `Allow trusted Microsoft services to access this storage account`. Establezca esta configuración en **Firewalls y redes virtuales** en la cuenta de almacenamiento.
> * Debe tener el permiso `Microsoft.Authorization/roleAssignments/write` en la cuenta de almacenamiento seleccionada. Para más información, consulte [Roles integrados en Azure](../../role-based-access-control/built-in-roles.md).

## <a name="configure-in-azure-portal"></a>Configuración en Azure Portal

Conéctese a [Azure Portal](https://portal.azure.com) con su suscripción. Desplácese al grupo de recursos y al servidor.

1. Haga clic en **Auditoría** en el encabezado Seguridad. Seleccione **Activado**.

2. Seleccione **Storage**. Seleccione la cuenta de almacenamiento en la que se guardarán los registros. La cuenta de almacenamiento debe cumplir los requisitos enumerados en la sección [Requisitos previos](#prerequisites).

3. Abra **Detalles de almacenamiento**.

  > [!NOTE]
  > Si la cuenta de Azure Storage seleccionada está detrás de una red virtual, verá el mensaje siguiente:
  >
  >`You have selected a storage account that is behind a firewall or in a virtual network. Using this storage requires to enable 'Allow trusted Microsoft services to access this storage account' on the storage account and creates a server managed identity with 'storage blob data contributor' RBAC.`
  >
  >Si no ve este mensaje, la cuenta de almacenamiento no está detrás de una red virtual.

4. Seleccione el número de días para el período de retención. A continuación, haga clic en **Aceptar**. Los registros anteriores al período de retención se eliminarán.

5. Seleccione **Guardar** en la configuración de auditoría.

Ha configurado correctamente la auditoría para que escriba en una cuenta de almacenamiento detrás de un firewall o una red virtual.

## <a name="configure-with-rest-commands"></a>Configuración con comandos de REST

Como alternativa al uso de Azure Portal, puede usar comandos de REST para configurar la auditoría con el fin de escribir eventos de base de datos en una cuenta de almacenamiento detrás de una red virtual y un firewall.

Antes de ejecutar los scripts de ejemplo de esta sección, es necesario que los actualice. Reemplace los valores siguientes en los scripts:

|Valor de ejemplo|Descripción de ejemplo|
|:-----|:-----|
|`<subscriptionId>`| Identificador de suscripción de Azure|
|`<resource group>`| Resource group|
|`<logical SQL server>`| Nombre de servidor|
|`<administrator login>`| Cuenta de administrador |
|`<complex password>`| Contraseña compleja para la cuenta de administrador|

Para configurar una auditoría de SQL para que escriba eventos en una cuenta de almacenamiento detrás de un firewall o una red virtual:

1. Registre su servidor con Azure Active Directory (Azure AD). Use PowerShell o la API REST.

   **PowerShell**

   ```powershell
   Connect-AzAccount
   Select-AzSubscription -SubscriptionId <subscriptionId>
   Set-AzSqlServer -ResourceGroupName <your resource group> -ServerName <azure server name> -AssignIdentity
   ```

   [**API REST**](https://docs.microsoft.com/rest/api/sql/servers/createorupdate):

   Solicitud de ejemplo

   ```html
   PUT https://management.azure.com/subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Sql/servers/<azure server name>?api-version=2015-05-01-preview
   ```

   Cuerpo de la solicitud

   ```json
   {
   "identity": {
              "type": "SystemAssigned",
              },
   "properties": {
     "fullyQualifiedDomainName": "<azure server name>.database.windows.net",
     "administratorLogin": "<administrator login>",
     "administratorLoginPassword": "<complex password>",
     "version": "12.0",
     "state": "Ready"
   }
   ```

2. Abra [Azure Portal](https://portal.azure.com). Vaya a la cuenta de almacenamiento. Busque **Control de acceso (IAM)** y haga clic en **Agregar asignación de roles**. Asigne el rol de Azure **Colaborador de datos de blobs de almacenamiento** al servidor que hospeda la base de datos que ha registrado con Azure Active Directory (Azure AD) en el paso anterior.

   > [!NOTE]
   > Solo los miembros con el privilegio Propietario pueden realizar este paso. Para conocer los distintos roles integrados de Azure, consulte [Roles integrados de Azure](../../role-based-access-control/built-in-roles.md).

3. Configure la [directiva de auditoría de blobs del servidor](/rest/api/sql/server%20auditing%20settings/createorupdate) sin especificar un elemento *storageAccountAccessKey*:

   Solicitud de ejemplo

   ```html
     PUT https://management.azure.com/subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Sql/servers/<azure server name>/auditingSettings/default?api-version=2017-03-01-preview
   ```

   Cuerpo de la solicitud

   ```json
   {
     "properties": {
      "state": "Enabled",
      "storageEndpoint": "https://<storage account>.blob.core.windows.net"
     }
   }
   ```

## <a name="using-azure-powershell"></a>Uso de Azure PowerShell

- [Crear o actualizar la directiva de auditoría de base de datos (Set-AzSqlDatabaseAudit)](/powershell/module/az.sql/set-azsqldatabaseaudit)
- [Crear o actualizar la directiva de auditoría de servidor (Set-AzSqlServerAudit)](/powershell/module/az.sql/set-azsqlserveraudit)

## <a name="using-azure-resource-manager-template"></a>Uso de la plantilla de Azure Resource Manager

Puede configurar la auditoría para escribir los eventos de base de datos en una cuenta de almacenamiento detrás de la red virtual y el firewall mediante una plantilla de [Azure Resource Manager](../../azure-resource-manager/management/overview.md), como se muestra en el ejemplo siguiente:

> [!IMPORTANT]
> Para usar una cuenta de almacenamiento detrás de la red virtual y el firewall, debe establecer el parámetro **isStorageBehindVnet** en True.

- [Implementación de un servidor de Azure SQL Server con la auditoría habilitada para escribir los registros de auditoría en un almacenamiento de blobs](https://azure.microsoft.com/resources/templates/201-sql-auditing-server-policy-to-blob-storage)

> [!NOTE]
> El ejemplo vinculado se encuentra en un repositorio público externo y se proporcionan "tal cual", sin ninguna garantía y no se admite en los programas o servicios de soporte técnico de Microsoft.

## <a name="next-steps"></a>Pasos siguientes

* [Use PowerShell para crear un punto de conexión de servicio de red virtual y, a continuación, una regla de red virtual para Azure SQL Database.](scripts/vnet-service-endpoint-rule-powershell-create.md)
* [Reglas de red virtual: Operaciones con API REST](/rest/api/sql/virtualnetworkrules)
* [Uso de reglas y puntos de conexión de servicio de red virtual para servidores](vnet-service-endpoint-rule-overview.md)
