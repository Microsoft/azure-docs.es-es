---
title: Habilitación de TDE en SQL con Azure Key Vault
titleSuffix: Azure SQL Database & SQL Managed Instance & Azure Synapse Analytics
description: Aprenda a configurar una instancia de Azure SQL Database y Azure Synapse Analytics para comenzar a usar Cifrado de datos transparente (TDE) para el cifrado en reposo mediante PowerShell o la CLI de Azure.
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: seo-lt-2019 sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviewer: vanto
ms.date: 03/12/2019
ms.openlocfilehash: 84166e5523cdbdb9ccebf9a0cbfc5e4dee0eb9e8
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387139"
---
# <a name="powershell-and-the-azure-cli-enable-transparent-data-encryption-with-customer-managed-key-from-azure-key-vault"></a>PowerShell y la CLI de Azure: Habilitación de Cifrado de datos transparente con una clave administrada por el cliente de Azure Key Vault
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

En este artículo se explica cómo usar una clave de Azure Key Vault para Cifrado de datos transparente (TDE) en una instancia de Azure SQL Database o Azure Synapse Analytics (anteriormente, SQL Data Warehouse). Para obtener más información sobre TDE con la integración de Azure Key Vault - compatibilidad con Traiga su propia clave (BYOK), visite [TDE with customer-managed keys in Azure Key Vault](transparent-data-encryption-byok-overview.md) (TDE con claves administradas por el cliente de Azure Key Vault).

## <a name="prerequisites-for-powershell"></a>Requisitos previos para PowerShell

- Debe tener una suscripción de Azure y ser un administrador en esa suscripción.
- [Recomendado, pero opcional] Debe tener un módulo de seguridad de hardware (HSM) o un almacén de claves locales para crear una copia local del material de claves del protector de TDE.
- Es preciso tener instalado y en ejecución Azure PowerShell.
- Cree una instancia de Azure Key Vault y una clave para usar para TDE.
  - [Instrucciones para usar un módulo de seguridad de hardware (HSM) y Key Vault](../../key-vault/keys/hsm-protected-keys.md)
    - El almacén de claves debe tener la siguiente propiedad que se usará para TDE:
  - [eliminación temporal](../../key-vault/general/soft-delete-overview.md) y protección de purgas
- La clave debe tener los siguientes atributos que se usarán para TDE:
  - Sin fecha de expiración
  - No deshabilitado
  - Poder realizar las operaciones *get*, *wrap key*, *unwrap key*

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para obtener instrucciones sobre la instalación del módulo Az, consulte [Instalación de Azure PowerShell](/powershell/azure/install-az-ps). Para los cmdlets concretos, consulte [AzureRM.Sql](https://docs.microsoft.com/powershell/module/AzureRM.Sql/).

Para obtener información específica sobre Key Vault, consulte las [instrucciones de PowerShell de Key Vault](../../key-vault/secrets/quick-create-powershell.md) y [Uso de la eliminación temporal de Key Vault con PowerShell](../../key-vault/general/soft-delete-powershell.md).

> [!IMPORTANT]
> El módulo de Azure Resource Manager (RM) para PowerShell todavía es compatible, pero todo el desarrollo futuro se realizará para el módulo Az.Sql. El módulo de AzureRM continuará recibiendo correcciones de errores hasta diciembre de 2020 como mínimo.  Los argumentos para los comandos del módulo Az y los módulos AzureRm son esencialmente idénticos. Para obtener más información sobre la compatibilidad, vea [Presentación del nuevo módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

## <a name="assign-an-azure-active-directory-azure-ad-identity-to-your-server"></a>Asignar una identidad de Azure Active Directory (Azure AD) al servidor

Si ya tiene un [servidor](logical-servers.md), use lo siguiente para agregar una identidad de Azure Active Directory (Azure AD) al servidor:

   ```powershell
   $server = Set-AzSqlServer -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> -AssignIdentity
   ```

Si va a crear un servidor, utilice el cmdlet [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) con la etiqueta -Identity para agregar una identidad de Azure AD durante la creación del servidor:

   ```powershell
   $server = New-AzSqlServer -ResourceGroupName <SQLDatabaseResourceGroupName> -Location <RegionName> `
       -ServerName <LogicalServerName> -ServerVersion "12.0" -SqlAdministratorCredentials <PSCredential> -AssignIdentity
   ```

## <a name="grant-key-vault-permissions-to-your-server"></a>Conceder permisos de Key Vault al servidor

Utilice el cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) para conceder al servidor acceso al almacén de claves antes de usar una clave para TDE.

   ```powershell
   Set-AzKeyVaultAccessPolicy -VaultName <KeyVaultName> `
       -ObjectId $server.Identity.PrincipalId -PermissionsToKeys get, wrapKey, unwrapKey
   ```

## <a name="add-the-key-vault-key-to-the-server-and-set-the-tde-protector"></a>Agregar la clave de Key Vault al servidor y establecer el protector de TDE

- Use el cmdlet [Get-AzKeyVaultKey](/powershell/module/az.keyvault/get-azkeyvaultkey?view=azps-2.4.0) para recuperar el identificador de clave del almacén de claves
- Utilice el cmdlet [Add-AzSqlServerKeyVaultKey](/powershell/module/az.sql/add-azsqlserverkeyvaultkey) para agregar la clave de Key Vault al servidor.
- Utilice el cmdlet [Set-AzSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/set-azsqlservertransparentdataencryptionprotector) para establecer la clave como protector de TDE para todos los recursos del servidor.
- Utilice el cmdlet [Get-AzureRmSqlServerTransparentDataEncryptionProtector](/powershell/module/az.sql/get-azsqlservertransparentdataencryptionprotector) para confirmar que el protector de TDE se ha configurado según lo previsto.

> [!NOTE]
> La longitud combinada para el nombre del almacén de claves y el nombre de la clave no puede superar los 94 caracteres.

> [!TIP]
> KeyId de ejemplo desde Key Vault: https://contosokeyvault.vault.azure.net/keys/Key1/1a1a2b2b3c3c4d4d5e5e6f6f7g7g8h8h

```powershell
# add the key from Key Vault to the server
Add-AzSqlServerKeyVaultKey -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> -KeyId <KeyVaultKeyId>

# set the key as the TDE protector for all resources under the server
Set-AzSqlServerTransparentDataEncryptionProtector -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName> `
   -Type AzureKeyVault -KeyId <KeyVaultKeyId>

# confirm the TDE protector was configured as intended
Get-AzSqlServerTransparentDataEncryptionProtector -ResourceGroupName <SQLDatabaseResourceGroupName> -ServerName <LogicalServerName>
```

## <a name="turn-on-tde"></a>Activar TDE

Utilice el cmdlet [Set-AzSqlDatabaseTransparentDataEncryption](/powershell/module/az.sql/set-azsqldatabasetransparentdataencryption) para activar el TDE.

```powershell
Set-AzSqlDatabaseTransparentDataEncryption -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> -DatabaseName <DatabaseName> -State "Enabled"
```

Ahora, la base de datos o el almacén de datos tienen TDE habilitado con una clave de cifrado en Key Vault.

## <a name="check-the-encryption-state-and-encryption-activity"></a>Comprobar el estado de cifrado y la actividad de cifrado

Utilice el cmdlet [Get-AzSqlDatabaseTransparentDataEncryption](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryption) para obtener el estado de cifrado y el cmdlet [Get-AzSqlDatabaseTransparentDataEncryptionActivity](/powershell/module/az.sql/get-azsqldatabasetransparentdataencryptionactivity) para comprobar el progreso de cifrado para un almacén de datos o una base de datos.

```powershell
# get the encryption state
Get-AzSqlDatabaseTransparentDataEncryption -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> -DatabaseName <DatabaseName> `

# check the encryption progress for a database or data warehouse
Get-AzSqlDatabaseTransparentDataEncryptionActivity -ResourceGroupName <SQLDatabaseResourceGroupName> `
   -ServerName <LogicalServerName> -DatabaseName <DatabaseName>  
```

# <a name="the-azure-cli"></a>[La CLI de Azure](#tab/azure-cli)

Para instalar la versión necesaria de la CLI de Azure (2.0 o posterior) y conectarla con la suscripción a Azure, consulte [Instalación y configuración de la Interfaz de la línea de comandos de Azure 2.0 multiplataforma](https://docs.microsoft.com/cli/azure/install-azure-cli).

Para obtener información específica sobre Key Vault, consulte [Administración de Key Vault mediante la CLI 2.0](../../key-vault/general/manage-with-cli2.md) y [Uso de la eliminación temporal de Key Vault con la CLI](../../key-vault/general/soft-delete-cli.md).

## <a name="assign-an-azure-ad-identity-to-your-server"></a>Asignar una identidad de Azure AD al servidor

```azurecli
# create server (with identity) and database
az sql server create --name <servername> --resource-group <rgname>  --location <location> --admin-user <user> --admin-password <password> --assign-identity
az sql db create --name <dbname> --server <servername> --resource-group <rgname>
```

> [!TIP]
> Mantenga el valor de "principalID" de la creación del servidor; es el identificador de objeto usado para asignar permisos de almacén de claves en el siguiente paso.

## <a name="grant-key-vault-permissions-to-your-server"></a>Conceder permisos de Key Vault al servidor

```azurecli
# create key vault, key and grant permission
az keyvault create --name <kvname> --resource-group <rgname> --location <location> --enable-soft-delete true
az keyvault key create --name <keyname> --vault-name <kvname> --protection software
az keyvault set-policy --name <kvname>  --object-id <objectid> --resource-group <rgname> --key-permissions wrapKey unwrapKey get
```

> [!TIP]
> Mantenga el URI de clave o el valor de keyID de la nueva clave para el próximo paso; por ejemplo: https://contosokeyvault.vault.azure.net/keys/Key1/1a1a2b2b3c3c4d4d5e5e6f6f7g7g8h8h.

## <a name="add-the-key-vault-key-to-the-server-and-set-the-tde-protector"></a>Agregar la clave de Key Vault al servidor y establecer el protector de TDE

```azurecli
# add server key and update encryption protector
az sql server key create --server <servername> --resource-group <rgname> --kid <keyID>
az sql server tde-key set --server <servername> --server-key-type AzureKeyVault  --resource-group <rgname> --kid <keyID>
```

> [!NOTE]
> La longitud combinada para el nombre del almacén de claves y el nombre de la clave no puede superar los 94 caracteres.

## <a name="turn-on-tde"></a>Activar TDE

```azurecli
# enable encryption
az sql db tde set --database <dbname> --server <servername> --resource-group <rgname> --status Enabled
```

Ahora, la base de datos o el almacén de datos tienen TDE habilitado con una clave de cifrado administrada por el cliente en Azure Key Vault.

## <a name="check-the-encryption-state-and-encryption-activity"></a>Comprobar el estado de cifrado y la actividad de cifrado

```azurecli
# get encryption scan progress
az sql db tde list-activity --database <dbname> --server <servername> --resource-group <rgname>  

# get whether encryption is on or off
az sql db tde show --database <dbname> --server <servername> --resource-group <rgname>
```

* * *

## <a name="useful-powershell-cmdlets"></a>Cmdlets de PowerShell útiles

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

- Utilice el cmdlet [Set-AzSqlDatabaseTransparentDataEncryption](/powershell/module/az.sql/set-azsqldatabasetransparentdataencryption) para desactivar TDE.

   ```powershell
   Set-AzSqlDatabaseTransparentDataEncryption -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName> `
      -DatabaseName <DatabaseName> -State "Disabled"
   ```

- Utilice el cmdlet [Get-AzSqlServerKeyVaultKey](/powershell/module/az.sql/get-azsqlserverkeyvaultkey) para devolver la lista de claves de Key Vault agregadas al servidor.

   ```powershell
   # KeyId is an optional parameter, to return a specific key version
   Get-AzSqlServerKeyVaultKey -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```

- Utilice el cmdlet [Remove-AzSqlServerKeyVaultKey](/powershell/module/az.sql/remove-azsqlserverkeyvaultkey) para quitar una clave de Key Vault del servidor.

   ```powershell
   # the key set as the TDE Protector cannot be removed
   Remove-AzSqlServerKeyVaultKey -KeyId <KeyVaultKeyId> -ServerName <LogicalServerName> -ResourceGroupName <SQLDatabaseResourceGroupName>
   ```

# <a name="the-azure-cli"></a>[La CLI de Azure](#tab/azure-cli)

- Para obtener la configuración general de la base de datos, vea [az sql](/cli/azure/sql).

- Para consultar la configuración de la clave de almacén, vea [az sql server key](/cli/azure/sql/server/key).

- Para consultar la configuración del TDE, vea [az sql server tde-key](/cli/azure/sql/server/tde-key) y [az sql db tde](/cli/azure/sql/db/tde).

* * *

## <a name="troubleshooting"></a>Solución de problemas

Compruebe lo siguiente si se produce un problema:

- Si no se encuentra el almacén de claves, asegúrese de que está en la suscripción correcta.

   # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

   ```powershell
   Get-AzSubscription -SubscriptionId <SubscriptionId>
   ```

   # <a name="the-azure-cli"></a>[La CLI de Azure](#tab/azure-cli)

   ```powershell
   az account show - s <SubscriptionId>
   ```

   * * *

- Si la nueva clave no se puede agregar al servidor, o la nueva clave no se puede actualizar como protector de TDE, compruebe lo siguiente:
   - La clave no debe tener una fecha de expiración
   - La clave debe tener las operaciones *get*, *wrap key* y *unwrap key*.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a cambiar el protector de TDE de un servidor para cumplir con los requisitos de seguridad: [Rotación del protector de Cifrado de datos transparente (TDE) mediante PowerShell](transparent-data-encryption-byok-key-rotation.md).
- En caso de riesgo de seguridad, aprenda a quitar un protector de TDE potencialmente comprometido: [Eliminación de una clave potencialmente comprometida](transparent-data-encryption-byok-remove-tde-protector.md).
