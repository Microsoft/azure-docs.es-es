---
title: 'Inicio rápido: administración de claves con la biblioteca cliente de Azure Key Vault para Python'
description: Aprenda a crear, recuperar y eliminar claves desde una instancia de Azure Key Vault mediante la biblioteca cliente de Python
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/03/2020
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: 44942067756f82c224decc218de17bf7dbc69734
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89482148"
---
# <a name="quickstart-azure-key-vault-keys-client-library-for-python"></a>Inicio rápido: biblioteca cliente de claves de Azure Key Vault para Python

Introducción a la biblioteca cliente de Azure Key Vault para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Si usa Key Vault para almacenar claves criptográficas, no tendrá que almacenarlas en el código, lo que aumenta la seguridad de las aplicaciones.

[Documentación de referencia de API](/python/api/overview/azure/keyvault-keys-readme?view=azure-python) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-keys) | [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-keyvault-keys/)

## <a name="set-up-your-local-environment"></a>Configuración de un entorno local

[!INCLUDE [Set up your local environment](../../../includes/key-vault-python-qs-setup.md)]

7. Instale la biblioteca de claves de Key Vault:

    ```terminal
    pip install azure-keyvault-keys
    ```

## <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-python-qs-rg-kv-creation.md)]

## <a name="give-the-service-principal-access-to-your-key-vault"></a>Acceso de la entidad de servicio al almacén de claves

Ejecute el siguiente comando [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy) para autorizar a la entidad de servicio a realizar las operaciones delete, get, list y create en las claves. 

# <a name="cmd"></a>[cmd](#tab/cmd)

```azurecli
az keyvault set-policy --name %KEY_VAULT_NAME% --spn %AZURE_CLIENT_ID% --resource-group KeyVault-PythonQS-rg --key-permissions delete get list create
```

# <a name="bash"></a>[bash](#tab/bash)

```azurecli
az keyvault set-policy --name $KEY_VAULT_NAME --spn $AZURE_CLIENT_ID --resource-group KeyVault-PythonQS-rg --key-permissions delete get list create
```

---

Este comando utiliza las variables de entorno `KEY_VAULT_NAME` y `AZURE_CLIENT_ID` creadas en los pasos anteriores.

Para más información, consulte [Asignación de una directiva de acceso: CLI ](../general/assign-access-policy-cli.md).

## <a name="create-the-sample-code"></a>Creación del código de ejemplo

La biblioteca cliente de Azure Key Vault para Python le permite administrar las claves criptográficas y los recursos relacionados, como certificados y secretos. El siguiente código de ejemplo muestra cómo crear un cliente y cómo establecer, recuperar y eliminar un secreto.

Cree un archivo llamado *kv_keys.py* que contenga este código.

```python
import os
from azure.keyvault.keys import KeyClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = "https://" + keyVaultName + ".vault.azure.net"

credential = DefaultAzureCredential()
client = KeyClient(vault_url=KVUri, credential=credential)

keyName = input("Input a name for your key > ")

print(f"Creating a key in {keyVaultName} called '{keyName}' ...")

rsa_key = client.create_rsa_key(keyName, size=2048)

print(" done.")

print(f"Retrieving your key from {keyVaultName}.")

retrieved_key = client.get_key(keyName)

print(f"Key with name '{retrieved_key.name}' was found.")
print(f"Deleting your key from {keyVaultName} ...")

poller = client.begin_delete_key(keyName)
deleted_key = poller.result()

print(" done.")
```

## <a name="run-the-code"></a>Ejecución del código

Asegúrese de que el código de la sección anterior se encuentra en un archivo llamado *kv_keys.py*. Luego ejecute el código con el siguiente comando:

```terminal
python kv_keys.py
```

- Si se producen errores de permisos, asegúrese de que ha ejecutado el comando [`az keyvault set-policy`](#give-the-service-principal-access-to-your-key-vault).
- Al volver a ejecutar el código con el mismo nombre de clave, se puede producir el error "(Conflict) Key <name> is currently in a deleted but recoverable state" [(Conflicto) La clave <name> se ha eliminado, pero se puede recuperar]. Use otro nombre de clave.

## <a name="code-details"></a>Detalles del código

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

En el código anterior, el objeto [`DefaultAzureCredential`](/python/api/azure-identity/azure.identity.defaultazurecredential?view=azure-python) usa las variables de entorno que ha creado para la entidad de servicio. Esta credencial se proporciona siempre que se crea un objeto de cliente desde una biblioteca de Azure, como [`KeyClient`](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?view=azure-python), junto con el identificador URI del recurso con el que se desea trabajar a través de ese cliente:

```python
credential = DefaultAzureCredential()
client = KeyClient(vault_url=KVUri, credential=credential)
```

## <a name="save-a-key"></a>Guardar una clave

Una vez que haya obtenido el objeto de cliente para el almacén de claves, puede almacenar una clave mediante el método [create_rsa_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?view=azure-python#create-rsa-key-name----kwargs-): 

```python
rsa_key = client.create_rsa_key(keyName, size=2048)
```

También se pueden usar [create_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?view=azure-python#create-key-name--key-type----kwargs-) o [create_ec_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?view=azure-python#create-ec-key-name----kwargs-).

La llamada a un método `create` genera una llamada a la API REST de Azure para el almacén de claves.

Durante el control de la solicitud, Azure autentica la identidad del autor de la llamada (la entidad de servicio) mediante el objeto de credencial que proporcionó al cliente.

También comprueba que el autor de la llamada tiene la autorización necesaria para realizar la acción solicitada. Esta autorización se la concedió a la entidad de servicio anteriormente mediante el comando [`az keyvault set-policy`](#give-the-service-principal-access-to-your-key-vault).

## <a name="retrieve-a-key"></a>Recuperación de una clave

Para leer una clave de Key Vault, utilice el método [get_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?view=azure-python#get-key-name--version-none----kwargs-):

```python
retrieved_key = client.get_key(keyName)
 ```

También puede comprobar que la clave se ha establecido con el comando [az keyvault key show](/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-show) de la CLI de Azure.

### <a name="delete-a-key"></a>Eliminación de una clave

Para eliminar una clave, use el método [begin_delete_key](/python/api/azure-keyvault-keys/azure.keyvault.keys.keyclient?view=azure-python#begin-delete-key-name----kwargs-):

```python
poller = client.begin_delete_key(keyName)
deleted_key = poller.result()
```

El método `begin_delete_key` es asincrónico y devuelve un objeto de sondeador. La llamada al método `result` del sondeador espera hasta su finalización.

Puede comprobar que la clave se elimina con el comando [az keyvault key show](/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-show) de la CLI de Azure.

Aun después de haberse eliminado, las claves permanecen en estado eliminado, pero recuperable, durante un tiempo. Si vuelve a ejecutar el código, use otro nombre de clave.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si también desea experimentar con [certificados](../certificates/quick-create-python.md) y [secretos](../secrets/quick-create-python.md), puede volver a usar la instancia de Key Vault que se ha creado en este artículo.

De lo contrario, cuando haya terminado con los recursos creados en este artículo, utilice el siguiente comando para eliminar el grupo de recursos y todos los recursos que contiene:

```azurecli
az group delete --resource-group KeyVault-PythonQS-rg
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Azure Key Vault](../general/overview.md)
- [Guía del desarrollador de Azure Key Vault](../general/developers-guide.md)
- [Procedimientos recomendados de Azure Key Vault](../general/best-practices.md)
- [Autenticación con Key Vault](../general/authentication.md)
