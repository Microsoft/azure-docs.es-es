---
title: Cifrado de datos de Azure Database for MySQL en la CLI de Azure
description: Aprenda a configurar y administrar el cifrado de datos de Azure Database for MySQL mediante la CLI de Azure.
author: kummanish
ms.author: manishku
ms.service: mysql
ms.topic: how-to
ms.date: 03/30/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: eb83cd4fe7e98b1cde6dcee5d3f25fa5e35f1d2c
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87799826"
---
# <a name="data-encryption-for-azure-database-for-mysql-by-using-the-azure-cli"></a>Cifrado de datos para Azure Database for MySQL mediante la CLI de Azure

Aprenda a usar la CLI de Azure para configurar y administrar el cifrado de datos de Azure Database for MySQL.

## <a name="prerequisites-for-azure-cli"></a>Requisitos previos de la CLI de Azure

* Debe tener una suscripción de Azure y ser un administrador en esa suscripción.
* Cree un almacén de claves y una clave que se usará como clave administrada por el cliente. Habilite también la protección de purga y la eliminación temporal en el almacén de claves.

  ```azurecli-interactive
  az keyvault create -g <resource_group> -n <vault_name> --enable-soft-delete true -enable-purge-protection true
  ```

* En la instancia de Azure Key Vault creada, cree la clave que se usará para el cifrado de datos de Azure Database for MySQL.

  ```azurecli-interactive
  az keyvault key create --name <key_name> -p software --vault-name <vault_name>
  ```

* Para usar un almacén de claves existente, este debe tener las siguientes propiedades para usarse como una clave administrada por el cliente:

  * [Eliminación temporal](../key-vault/general/soft-delete-overview.md)

    ```azurecli-interactive
    az resource update --id $(az keyvault show --name \ <key_vault_name> -o tsv | awk '{print $1}') --set \ properties.enableSoftDelete=true
    ```

  * [Protegido contra purgas](../key-vault/general/soft-delete-overview.md#purge-protection)

    ```azurecli-interactive
    az keyvault update --name <key_vault_name> --resource-group <resource_group_name>  --enable-purge-protection true
    ```

* La clave debe tener los siguientes atributos para que se pueda usar como clave administrada por el cliente:
  * Sin fecha de expiración
  * No deshabilitado
  * Poder realizar las operaciones **get**, **wrap** y **unwrap**

## <a name="set-the-right-permissions-for-key-operations"></a>Establecer los permisos adecuados para las operaciones de clave

1. Hay dos maneras de obtener la identidad administrada para Azure Database for MySQL.

   ### <a name="create-an-new-azure-database-for-mysql-server-with-a-managed-identity"></a>Cree un servidor de Azure Database for MySQL con una identidad administrada.

   ```azurecli-interactive
   az mysql server create --name -g <resource_group> --location <locations> --storage-size size>  -u <user>-p <pwd> --backup-retention <7> --sku-name <sku name> -geo-redundant-backup <Enabled/Disabled>  --assign-identity
   ```

   ### <a name="update-an-existing-the-azure-database-for-mysql-server-to-get-a-managed-identity"></a>Actualice un servidor de Azure Database for MySQL existente para obtener una identidad administrada.

   ```azurecli-interactive
   az mysql server update --name  <server name>  -g <resource_group> --assign-identity
   ```

2. Establezca los **Permisos de clave** (**Get**, **Wrap** y **Unwrap**) para la **Entidad de seguridad**, que es el nombre del servidor MySQL.

    ```azurecli-interactive
    az keyvault set-policy --name -g <resource_group> --key-permissions get unwrapKey wrapKey --object-id <principal id of the server>
    ```

## <a name="set-data-encryption-for-azure-database-for-mysql"></a>Establecimiento del cifrado de datos para Azure Database for MySQL

1. Permita el cifrado de datos para Azure Database for MySQL mediante la clave creada en Azure Key Vault.

    ```azurecli-interactive
    az mysql server key create –name  <server name>  -g <resource_group> --kid <key url>
    ```

    URL de clave: `https://YourVaultName.vault.azure.net/keys/YourKeyName/01234567890123456789012345678901>`

## <a name="using-data-encryption-for-restore-or-replica-servers"></a>Uso del cifrado de datos para servidores de restauración o réplica

Después de cifrar Azure Database for MySQL con la clave administrada de un cliente almacenada en Key Vault, también se cifra cualquier copia recién creada del servidor. Puede crear esta nueva copia mediante una operación de restauración local o geográfica, o por medio de una operación de réplica (local o entre regiones). De modo que, con un servidor MySQL cifrado, puede usar los siguientes pasos para crear un servidor restaurado cifrado.

### <a name="creating-a-restoredreplica-server"></a>Creación de un servidor restaurado o de réplica

* [Creación de un servidor de restauración](howto-restore-server-cli.md) 
* [Creación de un servidor de réplica de lectura](howto-read-replicas-cli.md) 

### <a name="once-the-server-is-restored-revalidate-data-encryption-the-restored-server"></a>Una vez restaurado el servidor, vuelva a validar el cifrado de datos del servidor restaurado.

*   Asignación de identidades para el servidor Réplica
```azurecli-interactive
az mysql server update --name  <server name>  -g <resoure_group> --assign-identity
```

*   Obtenga la clave existente que se debe usar para el servidor restaurado o el servidor Réplica.

```azurecli-interactive
az mysql server key list --name  '<server_name>'  -g '<resource_group_name>'
```

*   Establezca la directiva de la nueva identidad del servidor restaurado o el servidor Réplica.
  
```azurecli-interactive
az keyvault set-policy --name <keyvault> -g <resoure_group> --key-permissions get unwrapKey wrapKey --object-id <principl id of the server returned by the step 1>
```

* Vuelva a validar el servidor restaurado o el servidor Réplica con la clave de cifrado

```azurecli-interactive
az mysql server key create –name  <server name> -g <resource_group> --kid <key url>
```

## <a name="additional-capability-for-the-key-being-used-for-the-azure-database-for-mysql"></a>Funcionalidad adicional para la clave que usará para Azure Database for MySQL

### <a name="get-the-key-used"></a>Obtención de la clave usada

```azurecli-interactive
az mysql server key show --name  <server name>  -g <resource_group> --kid <key url>
```

Dirección URL de la clave: `https://YourVaultName.vault.azure.net/keys/YourKeyName/01234567890123456789012345678901>`

### <a name="list-the-key-used"></a>Enumeración de la clave usada

```azurecli-interactive
az mysql server key list --name  <server name>  -g <resource_group>
```

### <a name="drop-the-key-being-used"></a>Eliminación de la clave usada

```azurecli-interactive
az mysql server key delete -g <resource_group> --kid <key url>
```

## <a name="using-an-azure-resource-manager-template-to-enable-data-encryption"></a>Uso de una plantilla de Azure Resource Manager para habilitar el cifrado de datos

Además de en Azure Portal, también puede habilitar el cifrado de datos en el servidor de Azure Database for MySQL mediante plantillas de Azure Resource Manager para los servidores nuevos o ya existentes.

### <a name="for-a-new-server"></a>Para un nuevo servidor

Use una de las plantillas de Azure Resource Manager creadas anteriormente para aprovisionar el servidor con el cifrado de datos habilitado: [Ejemplo con cifrado de datos](https://github.com/Azure/azure-mysql/tree/master/arm-templates/ExampleWithDataEncryption)

Esta plantilla de Azure Resource Manager crea un servidor de Azure Database for MySQL y usa **KeyVault** y **Key** como parámetros para habilitar el cifrado de datos en el servidor.

### <a name="for-an-existing-server"></a>Para un servidor existente

Además, puede usar plantillas de Azure Resource Manager para habilitar el cifrado de datos en los servidores existentes de Azure Database for MySQL.

* Pase el id. de recurso de la clave de Azure Key Vault que copió anteriormente en la propiedad `Uri` en el objeto properties.

* Use *2020-01-01-preview* como la versión de API.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string"
    },
    "serverName": {
      "type": "string"
    },
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Key vault name where the key to use is stored"
      }
    },
    "keyVaultResourceGroupName": {
      "type": "string",
      "metadata": {
        "description": "Key vault resource group name where it is stored"
      }
    },
    "keyName": {
      "type": "string",
      "metadata": {
        "description": "Key name in the key vault to use as encryption protector"
      }
    },
    "keyVersion": {
      "type": "string",
      "metadata": {
        "description": "Version of the key in the key vault to use as encryption protector"
      }
    }
  },
  "variables": {
    "serverKeyName": "[concat(parameters('keyVaultName'), '_', parameters('keyName'), '_', parameters('keyVersion'))]"
  },
  "resources": [
    {
      "type": "Microsoft.DBforMySQL/servers",
      "apiVersion": "2017-12-01",
      "kind": "",
      "location": "[parameters('location')]",
      "identity": {
        "type": "SystemAssigned"
      },
      "name": "[parameters('serverName')]",
      "properties": {
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-05-01",
      "name": "addAccessPolicy",
      "resourceGroup": "[parameters('keyVaultResourceGroupName')]",
      "dependsOn": [
        "[resourceId('Microsoft.DBforMySQL/servers', parameters('serverName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.KeyVault/vaults/accessPolicies",
              "name": "[concat(parameters('keyVaultName'), '/add')]",
              "apiVersion": "2018-02-14-preview",
              "properties": {
                "accessPolicies": [
                  {
                    "tenantId": "[subscription().tenantId]",
                    "objectId": "[reference(resourceId('Microsoft.DBforMySQL/servers/', parameters('serverName')), '2017-12-01', 'Full').identity.principalId]",
                    "permissions": {
                      "keys": [
                        "get",
                        "wrapKey",
                        "unwrapKey"
                      ]
                    }
                  }
                ]
              }
            }
          ]
        }
      }
    },
    {
      "name": "[concat(parameters('serverName'), '/', variables('serverKeyName'))]",
      "type": "Microsoft.DBforMySQL/servers/keys",
      "apiVersion": "2020-01-01-preview",
      "dependsOn": [
        "addAccessPolicy",
        "[resourceId('Microsoft.DBforMySQL/servers', parameters('serverName'))]"
      ],
      "properties": {
        "serverKeyType": "AzureKeyVault",
        "uri": "[concat(reference(resourceId(parameters('keyVaultResourceGroupName'), 'Microsoft.KeyVault/vaults/', parameters('keyVaultName')), '2018-02-14-preview', 'Full').properties.vaultUri, 'keys/', parameters('keyName'), '/', parameters('keyVersion'))]"
      }
    }
  ]
}

```

## <a name="next-steps"></a>Pasos siguientes

 Para más información sobre el cifrado de datos, consulte [Cifrado de datos de Azure Database for MySQL con clave administrada por el cliente](concepts-data-encryption-mysql.md).
