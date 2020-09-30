---
title: 'Inicio rápido: biblioteca cliente de Azure Key Vault para Python, administración de secretos'
description: Obtenga información sobre cómo crear, recuperar y eliminar secretos desde una instancia de Azure Key Vault mediante la biblioteca cliente de Python
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/03/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: cd8a5751c018b9b3b3b2ef96765545f2edab685b
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89489211"
---
# <a name="quickstart-azure-key-vault-secrets-client-library-for-python"></a>Inicio rápido: Biblioteca cliente d secretos de Azure Key Vault para Python

Introducción a la biblioteca cliente de Azure Key Vault para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Si usa Key Vault para almacenar secretos, no tendrá que almacenarlos en el código, lo que aumenta la seguridad de las aplicaciones.

[Documentación de referencia de API](/python/api/overview/azure/keyvault-secrets-readme?view=azure-python) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-secrets) | [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-keyvault-secrets/)

## <a name="set-up-your-local-environment"></a>Configuración de un entorno local

[!INCLUDE [Set up your local environment](../../../includes/key-vault-python-qs-setup.md)]

7. Instale la biblioteca de secretos de Key Vault:

    ```terminal
    pip install azure-keyvault-secrets
    ```

## <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-python-qs-rg-kv-creation.md)]

## <a name="give-the-service-principal-access-to-your-key-vault"></a>Acceso de la entidad de servicio al almacén de claves

Ejecute el siguiente comando [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy) para autorizar a la entidad de servicio para realizar las operaciones get, list y set en los secretos. Este comando utiliza las variables de entorno `KEY_VAULT_NAME` y `AZURE_CLIENT_ID` creadas en los pasos anteriores.

# <a name="cmd"></a>[cmd](#tab/cmd)

```azurecli
az keyvault set-policy --name %KEY_VAULT_NAME% --spn %AZURE_CLIENT_ID% --resource-group KeyVault-PythonQS-rg --secret-permissions delete get list set 
```

# <a name="bash"></a>[bash](#tab/bash)

```azurecli
az keyvault set-policy --name $KEY_VAULT_NAME --spn $AZURE_CLIENT_ID --resource-group KeyVault-PythonQS-rg --secret-permissions delete get list set 
```

---

Este comando utiliza las variables de entorno `KEY_VAULT_NAME` y `AZURE_CLIENT_ID` creadas en los pasos anteriores.

Para más información, consulte [Asignación de una directiva de acceso: CLI ](../general/assign-access-policy-cli.md).

## <a name="create-the-sample-code"></a>Creación del código de ejemplo

La biblioteca cliente de Azure Key Vault para Python permite administrar tanto secretos como recursos similares, como certificados y claves criptográficas. El siguiente código de ejemplo muestra cómo crear un cliente y cómo establecer, recuperar y eliminar un secreto.

Cree un archivo llamado *kv_secrets.py* que contenga este código.

```python
import os
import cmd
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = f"https://{keyVaultName}.vault.azure.net"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)

secretName = input("Input a name for your secret > ")
secretValue = input("Input a value for your secret > ")

print(f"Creating a secret in {keyVaultName} called '{secretName}' with the value '{secretValue}' ...")

client.set_secret(secretName, secretValue)

print(" done.")

print(f"Retrieving your secret from {keyVaultName}.")

retrieved_secret = client.get_secret(secretName)

print(f"Your secret is '{retrieved_secret.value}'.")
print(f"Deleting your secret from {keyVaultName} ...")

poller = client.begin_delete_secret(secretName)
deleted_secret = poller.result()

print(" done.")
```

## <a name="run-the-code"></a>Ejecución del código

Asegúrese de que el código de la sección anterior se encuentra en un archivo llamado *kv_secrets.py*. Luego, ejecute el código con el siguiente comando:

```terminal
python kv_secrets.py
```

- Si se producen errores de permisos, asegúrese de que ha ejecutado el comando [`az keyvault set-policy`](#give-the-service-principal-access-to-your-key-vault).
- Al volver a ejecutar el código con el mismo nombre de secreto, se puede producir el error "(Conflict) Secret <name> is currently in a deleted but recoverable state" [(Conflicto) El secreto <name> se ha eliminado, pero se puede recuperar]. Use otro nombre de secreto.

## <a name="code-details"></a>Detalles del código

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

En el código anterior, el objeto [`DefaultAzureCredential`](/python/api/azure-identity/azure.identity.defaultazurecredential?view=azure-python) usa las variables de entorno que ha creado para la entidad de servicio. Esta credencial se proporciona siempre que se crea un objeto de cliente desde una biblioteca de Azure, como [`SecretClient`](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?view=azure-python), junto con el identificador URI del recurso con el que desea trabajar a través de ese cliente:

```python
credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)
```

### <a name="save-a-secret"></a>Almacenamiento de un secreto

Una vez que haya obtenido el objeto de cliente para el almacén de claves, puede almacenar un secreto mediante el método [set_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?view=azure-python#set-secret-name--value----kwargs-): 

```python
client.set_secret(secretName, secretValue)
```

La llamada a `set_secret` genera una llamada a la API REST de Azure para el almacén de claves.

Durante el control de la solicitud, Azure autentica la identidad del autor de la llamada (la entidad de servicio) mediante el objeto de credencial que proporcionó al cliente.

También comprueba que el autor de la llamada tiene la autorización necesaria para realizar la acción solicitada. Esta autorización se la concedió a la entidad de servicio anteriormente mediante el comando [`az keyvault set-policy`](#give-the-service-principal-access-to-your-key-vault).

### <a name="retrieve-a-secret"></a>Recuperación de un secreto

Para leer un secreto de Key Vault, use el método [get_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?view=azure-python#get-secret-name--version-none----kwargs-):

```python
retrieved_secret = client.get_secret(secretName)
 ```

El valor de secreto se incluye en `retrieved_secret.value`.

Los secretos también se pueden recuperar con el comando [az keyvault secret show](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show) de la CLI de Azure.

### <a name="delete-a-secret"></a>Eliminación de un secreto

Para eliminar un secreto, use el método [begin_delete_secret](/python/api/azure-keyvault-secrets/azure.keyvault.secrets.secretclient?view=azure-python#begin-delete-secret-name----kwargs-):

```python
poller = client.begin_delete_secret(secretName)
deleted_secret = poller.result()
```

El método `begin_delete_secret` es asincrónico y devuelve un objeto de sondeador. La llamada al método `result` del sondeador espera hasta su finalización.

Para comprobar que el secreto se ha establecido, utilice el comando [az keyvault secret show](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show) de la CLI de Azure.

Aun después de haberse eliminado, los secretos permanecen en estado eliminado, pero recuperable, durante un tiempo. Si vuelve a ejecutar el código, use otro nombre de secreto.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si también desea experimentar con [certificados](../certificates/quick-create-python.md) y [claves](../keys/quick-create-python.md), puede volver a usar la instancia de Key Vault que se ha creado en este artículo.

De lo contrario, cuando haya terminado con los recursos creados en este artículo, utilice el siguiente comando para eliminar el grupo de recursos y todos los recursos que contiene:

```azurecli
az group delete --resource-group KeyVault-PythonQS-rg
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Azure Key Vault](../general/overview.md)
- [Guía del desarrollador de Azure Key Vault](../general/developers-guide.md)
- [Procedimientos recomendados de Azure Key Vault](../general/best-practices.md)
- [Autenticación con Key Vault](../general/authentication.md)
