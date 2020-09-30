---
title: 'Inicio rápido: administración de certificados con la biblioteca cliente de Azure Key Vault para Python'
description: Aprenda a crear, recuperar y eliminar certificados desde una instancia de Azure Key Vault mediante la biblioteca cliente de Python
author: msmbaldwin
ms.author: mbaldwin
ms.date: 09/03/2020
ms.service: key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: devx-track-python
ms.openlocfilehash: b9ff7397ad29ac681e21c32608ade9c6ce557c37
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89488633"
---
# <a name="quickstart-azure-key-vault-certificates-client-library-for-python"></a>Inicio rápido: Biblioteca cliente de certificados de Azure Key Vault para Python

Introducción a la biblioteca cliente de Azure Key Vault para Python. Siga estos pasos para instalar el paquete y probar el código de ejemplo para realizar tareas básicas. Si usa Key Vault para almacenar certificados, no tendrá que almacenarlos en el código, lo que aumenta la seguridad de las aplicaciones.

[Documentación de referencia de API](/python/api/overview/azure/keyvault-certificates-readme?view=azure-python) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-certificates) | [Paquete (índice de paquetes de Python)](https://pypi.org/project/azure-keyvault-certificates)

## <a name="set-up-your-local-environment"></a>Configuración de un entorno local

[!INCLUDE [Set up your local environment](../../../includes/key-vault-python-qs-setup.md)]

7. Instale la biblioteca de certificados de Key Vault:

    ```terminal
    pip install azure-keyvault-certificates
    ```

## <a name="create-a-resource-group-and-key-vault"></a>Creación de un grupo de recursos y de un almacén de claves

[!INCLUDE [Create a resource group and key vault](../../../includes/key-vault-python-qs-rg-kv-creation.md)]

## <a name="give-the-service-principal-access-to-your-key-vault"></a>Acceso de la entidad de servicio al almacén de claves

Ejecute el siguiente comando [az keyvault set-policy](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-set-policy) para autorizar a la entidad de servicio para realizar las operaciones get, list y create en los certificados.

# <a name="cmd"></a>[cmd](#tab/cmd)

```azurecli
az keyvault set-policy --name %KEY_VAULT_NAME% --spn %AZURE_CLIENT_ID% --resource-group KeyVault-PythonQS-rg --certificate-permissions delete get list create
```

# <a name="bash"></a>[bash](#tab/bash)

```azurecli
az keyvault set-policy --name $KEY_VAULT_NAME --spn $AZURE_CLIENT_ID --resource-group KeyVault-PythonQS-rg --certificate-permissions delete get list create 
```

---

Este comando utiliza las variables de entorno `KEY_VAULT_NAME` y `AZURE_CLIENT_ID` creadas en los pasos anteriores.

Para más información, consulte [Asignación de una directiva de acceso: CLI ](../general/assign-access-policy-cli.md).

## <a name="create-the-sample-code"></a>Creación del código de ejemplo

La biblioteca cliente de Azure Key Vault para Python permite administrar certificados y recursos similares, como secretos y claves criptográficas. El siguiente código de ejemplo muestra cómo crear un cliente y cómo establecer, recuperar y eliminar un secreto.

Cree un archivo llamado *kv_certificates.py* que contenga este código.

```python
import os
from azure.keyvault.certificates import CertificateClient, CertificatePolicy,CertificateContentType, WellKnownIssuerNames 
from azure.identity import DefaultAzureCredential

keyVaultName = os.environ["KEY_VAULT_NAME"]
KVUri = "https://" + keyVaultName + ".vault.azure.net"

credential = DefaultAzureCredential()
client = CertificateClient(vault_url=KVUri, credential=credential)

certificateName = input("Input a name for your certificate > ")

print(f"Creating a certificate in {keyVaultName} called '{certificateName}' ...")

policy = CertificatePolicy.get_default()
poller = client.begin_create_certificate(certificate_name=certificateName, policy=policy)
certificate = poller.result()

print(" done.")

print(f"Retrieving your certificate from {keyVaultName}.")

retrieved_certificate = client.get_certificate(certificateName)

print(f"Certificate with name '{retrieved_certificate.name}' was found'.")
print(f"Deleting your certificate from {keyVaultName} ...")

poller = client.begin_delete_certificate(certificateName)
deleted_certificate = poller.result()

print(" done.")
```

## <a name="run-the-code"></a>Ejecución del código

Asegúrese de que el código de la sección anterior se encuentra en un archivo llamado *kv_certificates.py*. Luego, ejecute el código con el siguiente comando:

```terminal
python kv_certificates.py
```

- Si se producen errores de permisos, asegúrese de que ha ejecutado el comando [`az keyvault set-policy`](#give-the-service-principal-access-to-your-key-vault).
- Al volver a ejecutar el código con el mismo nombre de clave, se puede producir el error "(Conflict) Certificate <name> is currently in a deleted but recoverable state" [(Conflicto) El certificado <name> se ha eliminado, pero se puede recuperar]. Use otro nombre de clave.

## <a name="code-details"></a>Detalles del código

### <a name="authenticate-and-create-a-client"></a>Autenticación y creación de un cliente

En el código anterior, el objeto [`DefaultAzureCredential`](/python/api/azure-identity/azure.identity.defaultazurecredential?view=azure-python) usa las variables de entorno que ha creado para la entidad de servicio. Esta credencial se proporciona siempre que se crea un objeto de cliente desde una biblioteca de Azure, como [`CertificateClient`](/python/api/azure-keyvault-certificates/azure.keyvault.certificates.certificateclient?view=azure-python), junto con el identificador URI del recurso con el que desea trabajar a través de ese cliente:

```python
credential = DefaultAzureCredential()
client = CertificateClient(vault_url=KVUri, credential=credential)
```

### <a name="save-a-certificate"></a>Guardar certificado

Una vez que haya obtenido el objeto de cliente del almacén de claves, puede crear un certificado con el método [begin_create_certificate](/python/api/azure-keyvault-certificates/azure.keyvault.certificates.certificateclient?view=azure-python#begin-create-certificate-certificate-name--policy----kwargs-): 

```python
policy = CertificatePolicy.get_default()
poller = client.begin_create_certificate(certificate_name=certificateName, policy=policy)
certificate = poller.result()
```

Aquí, el certificado requiere una directiva obtenida con el método [CertificatePolicy.get_default](/python/api/azure-keyvault-certificates/azure.keyvault.certificates.certificatepolicy?view=azure-python#get-default--).

La llamada a un método `begin_create_certificate` genera una llamada asincrónica a la API REST de Azure para el almacén de claves. Y dicha llamada devuelve un objeto sondeador. Para esperar el resultado de la operación, llame al método `result` del sondeador.

Durante la administración de la solicitud, Azure autentica la identidad del autor de la llamada (la entidad de servicio) mediante el objeto de credencial que proporcionó al cliente.

También comprueba que el autor de la llamada tiene la autorización necesaria para realizar la acción solicitada. Esta autorización se la concedió a la entidad de servicio anteriormente mediante el comando [`az keyvault set-policy`](#give-the-service-principal-access-to-your-key-vault).

### <a name="retrieve-a-certificate"></a>Recuperación de un certificado

Para leer un certificado de Key Vault, use el método [get_certificate](/python/api/azure-keyvault-certificates/azure.keyvault.certificates.certificateclient?view=azure-python#get-certificate-certificate-name----kwargs-):

```python
retrieved_certificate = client.get_certificate(certificateName)
 ```

También puede comprobar que el certificado se ha establecido con el comando [az keyvault certificate show](/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-show) de la CLI de Azure.

### <a name="delete-a-certificate"></a>Eliminación de un certificado

Para eliminar un certificado, use el método [begin_delete_certificate](/python/api/azure-keyvault-certificates/azure.keyvault.certificates.certificateclient?view=azure-python#begin-delete-certificate-certificate-name----kwargs-):

```python
poller = client.begin_delete_certificate(certificateName)
deleted_certificate = poller.result()
```

El método `begin_delete_certificate` es asincrónico y devuelve un objeto sondeador. La llamada al método `result` del sondeador espera hasta su finalización.

Puede comprobar que el certificado se elimina con el comando [az keyvault certificate show](/cli/azure/keyvault/certificate?view=azure-cli-latest#az-keyvault-certificate-show) de la CLI de Azure.

Aun después de haberse eliminado, los certificados permanecen en estado eliminado, pero recuperable, durante un tiempo. Si vuelve a ejecutar el código, use otro nombre de certificado.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si también desea experimentar con [secretos](../secrets/quick-create-python.md) y [claves](../keys/quick-create-python.md), puede volver a usar la instancia de Key Vault que se ha creado en este artículo.

De lo contrario, cuando haya terminado con los recursos creados en este artículo, utilice el siguiente comando para eliminar el grupo de recursos y todos los recursos que contiene:

```azurecli
az group delete --resource-group KeyVault-PythonQS-rg
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Azure Key Vault](../general/overview.md)
- [Guía del desarrollador de Azure Key Vault](../general/developers-guide.md)
- [Procedimientos recomendados de Azure Key Vault](../general/best-practices.md)
- [Autenticación con Key Vault](../general/authentication.md)
