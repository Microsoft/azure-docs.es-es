---
title: Control de acceso basado en rol en Azure Cosmos DB
description: Obtenga información sobre cómo Azure Cosmos DB proporciona protección de bases de datos con la integración de Active Directory (RBAC).
author: markjbrown
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 06/03/2020
ms.author: mjbrown
ms.openlocfilehash: 6edf5de852ea836de8be02636dd8a971ccebb86d
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87530578"
---
# <a name="role-based-access-control-in-azure-cosmos-db"></a>Control de acceso basado en rol en Azure Cosmos DB

Azure Cosmos DB proporciona un control de acceso basado en role (RBAC) integrado para escenarios de administración comunes en Azure Cosmos DB. Los usuarios con un perfil en Azure Active Directory pueden asignar estos roles de Azure a usuarios, grupos, entidades de servicio o identidades administradas para conceder o denegar el acceso a recursos y operaciones en los recursos de Azure Cosmos DB. Las asignaciones de roles están dirigidas únicamente al acceso al plano de control, que incluye el acceso a las cuentas, bases de datos, contenedores y ofertas de Azure Cosmos (rendimiento).

## <a name="built-in-roles"></a>Roles integrados

Los siguientes son los roles integrados compatibles que se admiten en Azure Cosmos DB:

|**Rol integrado**  |**Descripción**  |
|---------|---------|
|[Colaborador de cuenta de DocumentDB](../role-based-access-control/built-in-roles.md#documentdb-account-contributor)|Puede administrar cuentas de Azure Cosmos DB.|
|[Lector de cuentas de Cosmos DB](../role-based-access-control/built-in-roles.md#cosmos-db-account-reader-role)|Puede leer los datos de cuentas de Azure Cosmos DB.|
|[Operador de copias de seguridad de Cosmos](../role-based-access-control/built-in-roles.md#cosmosbackupoperator)|Puede enviar una solicitud de restauración para una base de datos de Azure Cosmos o un contenedor.|
|[Operador de Cosmos DB](../role-based-access-control/built-in-roles.md#cosmos-db-operator)|Puede aprovisionar cuentas, bases de datos y contenedores de Azure Cosmos, pero no puede acceder a las claves necesarias para acceder a los datos.|

> [!IMPORTANT]
> La compatibilidad con RBAC en Azure Cosmos DB se aplica solo a las operaciones del plano de control. Las operaciones del plano de datos se protegen mediante las claves maestras o tokens de recursos. Para más información, consulte [Protección del acceso a los datos de Azure Cosmos DB](secure-access-to-data.md).

## <a name="identity-and-access-management-iam"></a>Administración de identidad y acceso (IAM)

El panel **Control de acceso (IAM)** de Azure Portal se utiliza para configurar el control de acceso basado en rol en los recursos de Azure Cosmos. Los roles se aplican a usuarios, grupos, entidades de servicio e identidades administradas en Active Directory. Puede usar roles integrados o personalizados para usuarios y grupos. En la captura de pantalla siguiente se muestra la integración de Active Directory (RBAC) con control de acceso (IAM) en Azure Portal:

:::image type="content" source="./media/role-based-access-control/database-security-identity-access-management-rbac.png" alt-text="Control de acceso (IAM) en Azure Portal: demostración de la seguridad de base de datos":::

## <a name="custom-roles"></a>Roles personalizados

Además de los roles integrados, los usuarios también pueden crear [roles personalizados](../role-based-access-control/custom-roles.md) en Azure y aplicar estos roles a las entidades de servicio en todas las suscripciones dentro de su inquilino de Active Directory. Los roles personalizados proporcionan a los usuarios una forma de crear definiciones de roles de Azure con un conjunto personalizado de operaciones de los proveedores de recursos. Para saber qué operaciones están disponibles para crear roles personalizados para Azure Cosmos DB, consulte [Operaciones del proveedor de recursos de Azure Cosmos DB](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb).

## <a name="preventing-changes-from-the-azure-cosmos-db-sdks"></a><a id="prevent-sdk-changes"></a>Bloqueo en los SDK de Azure Cosmos DB para evitar cambios

El proveedor de recursos de Azure Cosmos DB se puede bloquear para evitar cambios en los recursos desde clientes que se conecten mediante claves de cuenta (es decir, aplicaciones que se conecten mediante el SDK de Azure Cosmos), cambios desde Azure Portal incluidos. Esta característica puede resultar útil para los usuarios que deseen mayores grados de control y gobernanza para los entornos de producción. Evitar los cambios desde el SDK también habilita características como los bloqueos de recurso y de registro de diagnóstico para las operaciones en el plano de control. Los clientes que se conecten mediante el SDK de Azure Cosmos DB no podrán cambiar ninguna propiedad de las cuentas, las bases de datos, los contenedores y el rendimiento de Azure Cosmos. Esto no afecta a las operaciones que implican la lectura y la escritura de datos en contenedores de Cosmos.

Cuando esta característica está habilitada, solo pueden realizar cambios en los recursos los usuarios con el rol de Azure correspondiente y credenciales de Azure Active Directory, incluidas las instancias de Managed Service Identity.

> [!WARNING]
> La habilitación de esta característica puede afectar la aplicación. Asegúrese de que entiende el riesgo antes de habilitarlo.

### <a name="check-list-before-enabling"></a>Lista de comprobación anterior a la habilitación

Esta configuración impedirá que se realicen cambios en cualquier recurso de Cosmos desde cualquier cliente que se conecte mediante claves de cuenta, como cualquier SDK de Cosmos DB, cualquier herramienta que se conecte a través de claves de cuenta o desde Azure Portal. Para evitar problemas o errores de aplicaciones después de habilitar esta característica, compruebe si las aplicaciones o los usuarios de Azure Portal realizan alguna de las siguientes acciones antes de habilitar esta característica; por ejemplo:

- Cualquier cambio en la cuenta de Cosmos, incluidas las propiedades o la adición o eliminación de regiones.

- Crear o eliminar recursos secundarios, como bases de datos y contenedores. Esto incluye recursos para otras API, como Cassandra, MongoDB, Gremlin y recursos de tabla.

- Actualizar el rendimiento de los recursos de nivel de base de datos o de contenedor.

- Modificar las propiedades de los contenedores, incluida la directiva de índice, TTL y las claves únicas.

- Modificar procedimientos almacenados, desencadenadores o funciones definidas por el usuario.

Si las aplicaciones (o usuarios mediante Azure Portal) realizan alguna de estas acciones, deberán migrarse para ejecutarse mediante [Plantillas ARM](manage-sql-with-resource-manager.md), [PowerShell](manage-with-powershell.md), la [CLI de Azure](manage-with-cli.md), REST o la [Biblioteca de administración de Azure](https://github.com/Azure-Samples/cosmos-management-net). Tenga en cuenta que la administración de Azure está disponible en [varios idiomas](https://docs.microsoft.com/azure/?product=featured#languages-and-tools).

### <a name="set-via-arm-template"></a>Definición mediante una plantilla de ARM

Para establecer esta propiedad mediante una plantilla de ARM, actualice la plantilla existente o exporte una nueva para la implementación actual e incluya `"disableKeyBasedMetadataWriteAccess": true` en las propiedades de los recursos de `databaseAccounts`. A continuación se muestra un ejemplo básico de una plantilla de Azure Resource Manager con este valor de propiedad.

```json
{
    {
      "type": "Microsoft.DocumentDB/databaseAccounts",
      "name": "[variables('accountName')]",
      "apiVersion": "2020-04-01",
      "location": "[parameters('location')]",
      "kind": "GlobalDocumentDB",
      "properties": {
        "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
        "locations": "[variables('locations')]",
        "databaseAccountOfferType": "Standard",
        "disableKeyBasedMetadataWriteAccess": true
        }
    }
}
```

> [!IMPORTANT]
> Asegúrese de incluir las otras propiedades de la cuenta y los recursos secundarios al volver a implementar esta propiedad. No implemente esta plantilla tal cual o se restablecerán todas las propiedades de la cuenta.

### <a name="set-via-azure-cli"></a>Definir mediante la CLI de Azure

Para la habilitación mediante la CLI de Azure, use el siguiente comando:

```azurecli-interactive
az cosmosdb update  --name [CosmosDBAccountName] --resource-group [ResourceGroupName]  --disable-key-based-metadata-write-access true

```

### <a name="set-via-powershell"></a>Definir mediante PowerShell

Para habilitar el uso de Azure PowerShell, use el siguiente comando:

```azurepowershell-interactive
Update-AzCosmosDBAccount -ResourceGroupName [ResourceGroupName] -Name [CosmosDBAccountName] -DisableKeyBasedMetadataWriteAccess true
```

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es el control de acceso basado en rol de Azure (RBAC)?](../role-based-access-control/overview.md)
- [Roles personalizados en los recursos de Azure](../role-based-access-control/custom-roles.md)
- [Operaciones del proveedor de recursos de Azure Cosmos DB](../role-based-access-control/resource-provider-operations.md#microsoftdocumentdb)
