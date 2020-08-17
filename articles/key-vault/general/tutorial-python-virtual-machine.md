---
title: 'Tutorial: Uso de Azure Key Vault con una máquina virtual en Python | Microsoft Docs'
description: En este tutorial, va a configurar una aplicación ASP.NET Core para que lea un secreto de su almacén de claves.
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 07/20/2020
ms.author: mbaldwin
ms.custom: mvc, devx-track-python
ms.openlocfilehash: c0f98f8c77e4a9cd4271511e0169f07da1c52baa
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87875956"
---
# <a name="tutorial-use-azure-key-vault-with-a-virtual-machine-in-python"></a>Tutorial: Uso de Azure Key Vault con una máquina virtual en Python

Azure Key Vault le ayuda a proteger las claves, los secretos y los certificados, como las claves de API y las cadenas de conexión de base de datos.

En este tutorial, configurará una aplicación de Python para leer información de Azure Key Vault mediante identidades administradas para recursos de Azure. Aprenderá a:

> [!div class="checklist"]
> * Creación de un Almacén de claves
> * Almacenar un secreto en Key Vault
> * Crear una máquina virtual Linux de Azure
> * Habilitar una [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md) para la máquina virtual
> * Conceder los permisos necesarios para que la aplicación de consola lea datos de Key Vault
> * Recuperar un secreto del almacén de claves

Antes de empezar, lea los [conceptos básicos de Key Vault](basic-concepts.md). 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="prerequisites"></a>Requisitos previos

Para Windows, Mac y Linux:
  * [Git](https://git-scm.com/downloads)
  * Este tutorial requiere que se ejecute localmente la CLI de Azure. Debe tener instalada la versión 2.0.4 de la CLI de Azure o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure 2.0](/cli/azure/install-azure-cli).

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Para iniciar sesión en Azure mediante la CLI de Azure, escriba:

```azurecli
az login
```

## <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-rg-kv-creation.md)]

## <a name="populate-your-key-vault-with-a-secret"></a>Rellenado del almacén de claves con un secreto

[!INCLUDE [Create a secret](../../../includes/key-vault-create-secret.md)]

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Create una máquina virtual llamada **myVM**, para lo que debe usar uno de los siguientes métodos:

| Linux | Windows |
|--|--|
| [CLI de Azure](../../virtual-machines/linux/quick-create-cli.md) | [CLI de Azure](../../virtual-machines/windows/quick-create-cli.md) |
| [PowerShell](../../virtual-machines/linux/quick-create-powershell.md) | [PowerShell](../../virtual-machines/windows/quick-create-powershell.md) |
| [Azure Portal](../../virtual-machines/linux/quick-create-portal.md) | [Portal de Azure](../../virtual-machines/windows/quick-create-portal.md) |

Para crear una máquina virtual Linux mediante la CLI de Azure, use el comando [az vm create](/cli/azure/vm).  En el ejemplo siguiente se agrega una cuenta de usuario llamada *azureuser*. El parámetro `--generate-ssh-keys` se usa para generar automáticamente una clave SSH y colocarla en la ubicación de la clave predeterminada ( *~/.ssh*). 

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

Anote el valor de `publicIpAddress` en la salida.

## <a name="assign-an-identity-to-the-vm"></a>Asignación de una identidad a la máquina virtual

Cree una identidad asignada por el sistema para la máquina virtual mediante el comando [az vm identity assign](/cli/azure/vm/identity?view=azure-cli-latest#az-vm-identity-assign) de la CLI de Azure:

```azurecli
az vm identity assign --name "myVM" --resource-group "myResourceGroup"
```

Tenga en cuenta la identidad asignada por el sistema que se muestra en el código siguiente. La salida del comando anterior sería: 

```output
{
  "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "userAssignedIdentities": {}
}
```

## <a name="assign-permissions-to-the-vm-identity"></a>Asignación de permisos a la identidad de máquina virtual

Ahora puede asignar los permisos de la identidad creada anteriormente al almacén de claves mediante la ejecución del comando siguiente:

```azurecli
az keyvault set-policy --name "<your-unique-keyvault-name>" --object-id "<systemAssignedIdentity>" --secret-permissions get list
```

## <a name="log-in-to-the-vm"></a>Inicio de sesión en la máquina virtual

Para iniciar sesión en la máquina virtual, siga las instrucciones que encontrará en el artículo en el que se explica la [conexión y el inicio de sesión en una máquina virtual Linux en Azure](../../virtual-machines/linux/login-using-aad.md), o bien en el que se explica la [conexión y el inicio de sesión en una máquina virtual Windows en Azure](../../virtual-machines/windows/connect-logon.md).


Para iniciar sesión en una máquina virtual Linux, puede usar el comando ssh con el "<publicIpAddress>" proporcionado en el paso de [Creación de una máquina virtual](#create-a-virtual-machine):

```terminal
ssh azureuser@<PublicIpAddress>
```

## <a name="install-python-libraries-on-the-vm"></a>Instalación de bibliotecas de Python en la máquina virtual

En la máquina virtual, instale las dos bibliotecas de Python que vamos a usar en el script de Python: `azure-keyvault-secrets` y `azure.identity`.  

En una máquina virtual Linux, por ejemplo, se puede usar `pip3` para instalarlas:

```bash
pip3 install azure-keyvault-secrets

pip3 install azure.identity
```

## <a name="create-and-edit-the-sample-python-script"></a>Creación y edición del script de Python de ejemplo

En la máquina virtual, cree un archivo de Python llamado **sample.py**. Posteriormente, edítelo para que contenga el siguiente código y reemplace <nombre-almacén de claves-único> por el nombre de su almacén de claves:

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential

keyVaultName = "<your-unique-keyvault-name>"
KVUri = f"https://{keyVaultName}.vault.azure.net"
secretName = "mySecret"

credential = DefaultAzureCredential()
client = SecretClient(vault_url=KVUri, credential=credential)
retrieved_secret = client.get_secret(secretName)

print(f"The value of secret '{secretName}' in '{keyVaultName}' is: '{retrieved_secret.value}'")
```

## <a name="run-the-sample-python-app"></a>Ejecución de la aplicación de Python de ejemplo

Por último, ejecute **sample.py**. Si todo ha ido bien, debería devolver el valor de su secreto:

```bash
python3 sample.py

The value of secret 'mySecret' in '<your-unique-keyvault-name>' is: 'Success!'
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no son necesarios, elimine la máquina virtual y el almacén de claves.  Puede hacerlo rápidamente, solo debe eliminar el grupo de recursos al que pertenecen:

```azurecli
az group delete -g myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

[API REST de Azure Key Vault](https://docs.microsoft.com/rest/api/keyvault/)
