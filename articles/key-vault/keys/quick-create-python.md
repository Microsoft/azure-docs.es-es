---
title: 'Inicio rápido: administración de claves con la biblioteca cliente de Azure Key Vault para Python'
description: Aprenda a crear, recuperar y eliminar claves desde una instancia de Azure Key Vault mediante la biblioteca cliente de Python
author: msmbaldwin
ms.author: mbaldwin
ms.date: 3/30/2020
ms.service: key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: 18ba00b39d8ffd703eb31b95d373e5b89e51c59b
ms.sourcegitcommit: 3246e278d094f0ae435c2393ebf278914ec7b97b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89376832"
---
# <a name="quickstart-azure-key-vault-keys-client-library-for-python"></a>Inicio rápido: biblioteca cliente de claves de Azure Key Vault para Python

Introducción a la biblioteca cliente de Azure Key Vault para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas.

Azure Key Vault ayuda a proteger claves criptográficas y secretos usados por servicios y aplicaciones en la nube. Use la biblioteca cliente de Key Vault para Python para:

- Aumentar la seguridad y el control sobre claves y contraseñas
- Crear e importar claves de cifrado en minutos
- Reducir la latencia con escala en la nube y redundancia global
- Simplificar y automatizar tareas para certificados TLS/SSL
- Utilizar módulos HSM con certificación FIPS 140-2 nivel 2

[Documentación de referencia de API](/python/api/overview/azure/keyvault-keys-readme?view=azure-python) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault) | [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-keyvault/)

## <a name="prerequisites"></a>Requisitos previos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Python 2.7, 3.5.3 o versiones posteriores
- [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) o [Azure PowerShell](/powershell/azure/)

En esta guía de inicio rápido se supone que está ejecutando la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) en una ventana de terminal de Linux.

## <a name="setting-up"></a>Instalación

### <a name="install-the-package"></a>Instalar el paquete

En la ventana de la consola, instale la biblioteca de claves de Azure Key Vault para Python.

```console
pip install azure-keyvault-keys
```

Para este inicio rápido, deberá instalar también el paquete azure.identity:

```console
pip install azure.identity
```

### <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves

En este inicio rápido se usa un almacén de claves de Azure creado previamente. Puede crear un almacén de claves siguiendo los pasos descritos en el [inicio rápido de CLI de Azure](quick-create-cli.md), [inicio rápido de Azure PowerShell](quick-create-powershell.md) o [inicio rápido de Azure Portal](quick-create-portal.md). Como alternativa, puede ejecutar los siguientes comandos de la CLI de Azure.

> [!Important]
> Cada almacén de claves debe tener un nombre único. Reemplace <nombre-almacén de claves-único> por el nombre del almacén de claves en los ejemplos siguientes.

```azurecli
az group create --name "myResourceGroup" -l "EastUS"

az keyvault create --name <your-unique-keyvault-name> -g "myResourceGroup"
```

### <a name="create-a-service-principal"></a>Creación de una entidad de servicio

La forma más sencilla de autenticar una aplicación basada en la nube es con una identidad administrada; para más información, consulte [Autenticación en Azure Key Vault](../general/authentication.md). 

Sin embargo, en aras de la simplicidad, en este inicio rápido se crea una aplicación de escritorio, que requiere el uso de una entidad de servicio y una directiva de control de acceso. La entidad de servicio requiere un nombre único con el formato "http://&lt;my-unique-service-principal-name&gt;".

Cree una entidad de servicio mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) de la CLI de Azure:

```azurecli
az ad sp create-for-rbac -n "http://&lt;my-unique-service-principal-name&gt;" --sdk-auth
```

Esta operación devolverá una serie de pares clave-valor. 

```console
{
  "clientId": "7da18cae-779c-41fc-992e-0527854c6583",
  "clientSecret": "b421b443-1669-4cd7-b5b1-394d5c945002",
  "subscriptionId": "443e30da-feca-47c4-b68f-1636b75e16b3",
  "tenantId": "35ad10f1-7799-4766-9acf-f2d946161b77",
  "activeDirectoryEndpointUrl": "https://login.microsoftonline.com",
  "resourceManagerEndpointUrl": "https://management.azure.com/",
  "sqlManagementEndpointUrl": "https://management.core.windows.net:8443/",
  "galleryEndpointUrl": "https://gallery.azure.com/",
  "managementEndpointUrl": "https://management.core.windows.net/"
}
```

Tome nota de los valores de clientId y clientSecret, ya que los usaremos en el paso [Establecimiento de variables de entorno](#set-environmental-variables) a continuación.

#### <a name="give-the-service-principal-access-to-your-key-vault"></a>Acceso de la entidad de servicio al almacén de claves

Cree una directiva de acceso para el almacén de claves que conceda permiso a la entidad de servicio pasando clientId al comando [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy). Conceda a la entidad de servicio los permisos get, list y create para las claves.

```azurecli
az keyvault set-policy -n <your-unique-keyvault-name> --spn <clientId-of-your-service-principal> --key-permissions delete get list create 
```

#### <a name="set-environmental-variables"></a>Establecimiento de variables de entorno

El método DefaultAzureCredential de nuestra aplicación depende de tres variables de entorno: `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET` y `AZURE_TENANT_ID`. Establezca estas variables en los valores clientId, clientSecret y tenantId que anotó en el paso [Creación de una entidad de servicio](#create-a-service-principal) anterior con el formato `export VARNAME=VALUE`. (Este método solo establece las variables para el shell actual y los procesos creados desde el shell. Para agregar de forma permanente estas variables a su entorno, edite el archivo `/etc/environment `). 

También tendrá que guardar el nombre del almacén de claves como una variable de entorno llamada `KEY_VAULT_NAME`.

```console
export AZURE_CLIENT_ID=<your-clientID>

export AZURE_CLIENT_SECRET=<your-clientSecret>

export AZURE_TENANT_ID=<your-tenantId>

export KEY_VAULT_NAME=<your-key-vault-name>
````

## <a name="object-model"></a>Modelo de objetos

La biblioteca cliente de Azure Key Vault para Python le permite administrar las claves y los recursos relacionados, como certificados y secretos. En los ejemplos de código siguientes se muestra cómo crear un cliente y cómo crear, recuperar y eliminar una clave.

## <a name="code-examples"></a>Ejemplos de código

### <a name="add-directives"></a>Adición de directivas

Agregue las siguientes directivas al principio del código:

```python
import os
from azure.keyvault.keys import KeyClient
from azure.identity import DefaultAzureCredential
```

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

La autenticación en el almacén de claves y la creación de un cliente de almacén de claves depende de las variables de entorno en el paso [Establecimiento de variables de entorno](#set-environmental-variables) anterior. El nombre del almacén de claves se expande al URI del almacén de claves, con el formato "https://<nombre-de-su-almacén-de-claves>.vault.azure.net".

```python
credential = DefaultAzureCredential()

client = KeyClient(vault_url=KVUri, credential=credential)
```

### <a name="save-a-key"></a>Guardar una clave

Ahora que la aplicación se ha autenticado, puede colocar una clave en el almacén de claves 

```python
rsa_key = client.create_rsa_key(myKey,size=2048)
```

Puede comprobar que la clave se ha establecido con el comando [az keyvault key show](/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-show):

```azurecli
az keyvault key show --vault-name <your-unique-keyvault-name> --name myKey
```

### <a name="retrieve-a-key"></a>Recuperación de una clave

Ahora puede recuperar la clave que ha creado anteriormente

```python
retrieved_key = client.get_key(keyName)
print(retrieve_key.name)

 ```

La clave se guarda como `retrieved_key`.

### <a name="delete-a-key"></a>Eliminación de una clave

Por último, vamos a eliminar la clave del almacén de claves

```python
client.delete_key(keyName)
```

Puede comprobar que la clave ya no está igual que el comando [az keyvault key show](/cli/azure/keyvault/key?view=azure-cli-latest#az-keyvault-key-show):

```azurecli
az keyvault key show --vault-name <your-unique-keyvault-name> --name myKey
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
from azure.keyvault.key import KeyClient
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = "https://" + keyVaultName + ".vault.azure.net"

credential = DefaultAzureCredential()
client = KeyClient(vault_url=KVUri, credential=credential)

keyName = "myKey"

print("Creating a key in " + keyVaultName + " called '" + keyName  + "` ...")

rsa_key = client.create_rsa_key(myKey,size=2048)

print(" done.")

print("Retrieving your key from " + keyVaultName + ".")

retrieved_key = client.get_key(keyName)

print("Key with name '{0}' was found'.".format(retrieved_key.name))
print("Deleting your key from " + keyVaultName + " ...")

client.begin_delete_key(keyName).result()

print(" done.")
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de claves y ha almacenado una clave y recuperado dicha clave. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- [Información general de Azure Key Vault](../general/overview.md)
- [Guía del desarrollador de Azure Key Vault](../general/developers-guide.md)
- [Procedimientos recomendados de Azure Key Vault](../general/best-practices.md)
