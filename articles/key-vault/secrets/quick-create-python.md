---
title: 'Inicio rápido: biblioteca cliente de Azure Key Vault para Python, administración de secretos'
description: Obtenga información sobre cómo crear, recuperar y eliminar secretos desde una instancia de Azure Key Vault mediante la biblioteca cliente de Python
author: msmbaldwin
ms.author: mbaldwin
ms.date: 10/20/2019
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: b31f22f9a3909df308fdcfa994833887828f2539
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87876366"
---
# <a name="quickstart-azure-key-vault-secrets-client-library-for-python"></a>Inicio rápido: Biblioteca cliente d secretos de Azure Key Vault para Python

Introducción a la biblioteca cliente de Azure Key Vault para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Azure Key Vault ayuda a proteger claves criptográficas y secretos usados por servicios y aplicaciones en la nube. Use la biblioteca cliente de Key Vault para Python para:

- Aumentar la seguridad y el control sobre claves y contraseñas
- Crear e importar claves de cifrado en minutos
- Reducir la latencia con escala en la nube y redundancia global
- Simplificar y automatizar tareas para certificados TLS/SSL
- Utilizar módulos HSM con certificación FIPS 140-2 nivel 2

[Documentación de referencia de API](/python/api/overview/azure/keyvault-secrets-readme?view=azure-python) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault) | [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-keyvault/)

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Python 2.7, 3.5.3 o versiones posteriores
- [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) o [Azure PowerShell](/powershell/azure/)

En esta guía de inicio rápido se supone que está ejecutando la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) en una ventana de terminal de Linux.

## <a name="setting-up"></a>Instalación

### <a name="install-the-package"></a>Instalar el paquete

En la ventana de la consola, instale la biblioteca de secretos de Azure Key Vault para Python.

```console
pip install azure-keyvault-secrets
```

Para este inicio rápido, deberá instalar también el paquete azure.identity:

```console
pip install azure.identity
```

### <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-rg-kv-creation.md)]

### <a name="create-a-service-principal"></a>Creación de una entidad de servicio

[!INCLUDE [Create a service principal](../../../includes/key-vault-sp-creation.md)]

#### <a name="give-the-service-principal-access-to-your-key-vault"></a>Acceso de la entidad de servicio al almacén de claves

[!INCLUDE [Give the service principal access to your key vault](../../../includes/key-vault-sp-kv-access.md)]

#### <a name="set-environmental-variables"></a>Establecimiento de variables de entorno

[!INCLUDE [Set environmental variables](../../../includes/key-vault-set-environmental-variables.md)]

## <a name="object-model"></a>Modelo de objetos

La biblioteca cliente de Azure Key Vault para Python le permite administrar las claves y los recursos relacionados, como certificados y secretos. Los ejemplos de código siguientes le mostrarán cómo crear un cliente y cómo establecer, recuperar y eliminar un secreto.

Toda la aplicación de consola está disponible en https://github.com/Azure-Samples/key-vault-dotnet-core-quickstart/tree/master/key-vault-console-app.

## <a name="code-examples"></a>Ejemplos de código

### <a name="add-directives"></a>Adición de directivas

Agregue las siguientes directivas al principio del código:

```python
import os
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential
```

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

La autenticación en el almacén de claves y la creación de un cliente de almacén de claves depende de las variables de entorno en el paso [Establecimiento de variables de entorno](#set-environmental-variables) anterior. El nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://<nombre-de-su-almacén-de-claves>.vault.azure.net".

```python
credential = DefaultAzureCredential()

client = SecretClient(vault_url=KVUri, credential=credential)
```

### <a name="save-a-secret"></a>Almacenamiento de un secreto

Ahora que la aplicación se ha autenticado, puede colocar un secreto en el almacén de claves mediante el método client.SetSecret (/dotnet/api/microsoft.azure.keyvault.keyvaultclientextensions.setsecretasync). Esto requiere establecer un nombre para el secreto; en este ejemplo, usaremos "mySecret".  

```python
client.set_secret(secretName, secretValue)
```

Puede comprobar que el secreto se ha establecido con el comando [az keyvault secret show](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show):

```azurecli
az keyvault secret show --vault-name <your-unique-keyvault-name> --name mySecret
```

### <a name="retrieve-a-secret"></a>Recuperación de un secreto

Ahora puede recuperar el valor previamente establecido con el [método client.GetSecret](/dotnet/api/microsoft.azure.keyvault.keyvaultclientextensions.getsecretasync).

```python
retrieved_secret = client.get_secret(secretName)
 ```

El secreto se guarda ahora como `retrieved_secret.value`.

### <a name="delete-a-secret"></a>eliminar un secreto

Por último, vamos a eliminar el secreto del almacén de claves con el [método client.DeleteSecret](/dotnet/api/microsoft.azure.keyvault.keyvaultclientextensions.getsecretasync).

```python
client.delete_secret(secretName)
```

Puede comprobar que el secreto ya no está con el comando [az keyvault secret show](/cli/azure/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show):

```azurecli
az keyvault secret show --vault-name <your-unique-keyvault-name> --name mySecret
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar la CLI de Azure o Azure PowerShell para quitar el almacén de claves y el grupo de recursos correspondiente.

```azurecli
az group delete -g "myResourceGroup"
```

```azurepowershell
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="sample-code"></a>Código de ejemplo

```python
import os
import cmd
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = f"https://{keyVaultName}.vault.azure.net"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)

secretName = "mySecret"

print("Input the value of your secret > ")
secretValue = raw_input()

print(f"Creating a secret in {keyVaultName} called '{secretName}' with the value '{secretValue}` ...")

client.set_secret(secretName, secretValue)

print(" done.")

print("Forgetting your secret.")
secretValue = ""
print(f"Your secret is {secretValue}.")

print(f"Retrieving your secret from {keyVaultName}.")

retrieved_secret = client.get_secret(secretName)

print(f"Your secret is '{retrieved_secret.value}'.")
print(f"Deleting your secret from {keyVaultName} ...")

client.delete_secret(secretName)

print(" done.")
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado un almacén de claves y ha almacenado un secreto en él. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Consulte la [guía del desarrollador de Azure Key Vault](../general/developers-guide.md).
- Consulte los [procedimientos recomendados de Azure Key Vault](../general/best-practices.md).
