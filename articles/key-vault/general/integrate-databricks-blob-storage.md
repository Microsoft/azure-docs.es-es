---
title: Acceso a Azure Blob Storage mediante Azure Databricks y Azure Key Vault
description: En este tutorial, aprenderá cómo acceder a Azure Blob Storage desde Azure Databricks mediante un secreto almacenado en Azure Key Vault
author: msmbaldwin
ms.author: mbaldwin
ms.service: key-vault
ms.topic: tutorial
ms.date: 06/16/2020
ms.openlocfilehash: bb574bb3dd000682090c6c3f861e885761753e19
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88588524"
---
# <a name="tutorial-access-azure-blob-storage-using-azure-databricks-and-azure-key-vault"></a>Tutorial: Acceso a Azure Blob Storage mediante Azure Databricks y Azure Key Vault

En este tutorial, aprenderá cómo acceder a Azure Blob Storage desde Azure Databricks mediante un secreto almacenado en Azure Key Vault. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un contenedor de blobs con la CLI de Azure.
> * Crear una instancia de Key Vault y establecer un secreto.
> * Crear un área de trabajo de Azure Databricks y agregar un ámbito de secreto de Key Vault.
> * Acceder al contenedor de blobs desde el área de trabajo de Azure Databricks.

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Antes de iniciar este tutorial, instale la [CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli-windows?view=azure-cli-latest).

## <a name="create-a-storage-account-and-blob-container-with-azure-cli"></a>Creación de un contenedor de blobs con la CLI de Azure

Primero tendrá que crear una cuenta de almacenamiento de uso general para usar blobs. Si no tiene un [grupo de recursos](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-create), cree uno antes de ejecutar el comando. El siguiente comando crea y muestra los metadatos del contenedor de almacenamiento. Copie el **identificador**.

```azurecli
az storage account create --name contosoblobstorage5 --resource-group contosoResourceGroup --location eastus --sku Standard_ZRS --encryption-services blob
```

![Salida de consola del comando anterior. El identificador está resaltado en verde para que lo vea el usuario final.](../media/databricks-command-output-1.png)

Para poder crear un contenedor en el que cargar el blob, tendrá que asignar el rol de [Colaborador de datos de Storage Blob](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-contributor) a sí mismo. En este ejemplo, el rol se asignará a la cuenta de almacenamiento que ha creado anteriormente.

```azurecli
az role assignment create --role "Storage Blob Data Contributor" --assignee t-trtr@microsoft.com --scope "/subscriptions/885e24c8-7a36-4217-b8c9-eed31e110504/resourceGroups/contosoResourceGroup5/providers/Microsoft.Storage/storageAccounts/contosoblobstorage5
```

Ahora que ha asignado el rol a la cuenta de almacenamiento, puede crear un contenedor para el blob.

```azurecli
az storage container create --account-name contosoblobstorage5 --name contosocontainer5 --auth-mode login
```

Una vez creado el contenedor, puede cargar un blob (archivo de su elección) en ese contenedor. En este ejemplo, se carga un archivo .txt con helloworld.

```azurecli
az storage blob upload --account-name contosoblobstorage5 --container-name contosocontainer5 --name helloworld --file helloworld.txt --auth-mode login
```

Obtenga una lista de los blobs del contenedor para comprobar que el archivo se encuentra en el contenedor.

```azurecli
az storage blob list --account-name contosoblobstorage5 --container-name contosocontainer5 --output table --auth-mode login
```

![Salida de consola del comando anterior. Muestra el archivo que se acaba de almacenar en el contenedor.](../media/databricks-command-output-2.png)

Obtenga el valor **key1** del contenedor de almacenamiento mediante el siguiente comando. Copie el valor.

```azurecli
az storage account keys list -g contosoResourceGroup5 -n contosoblobstorage5
```

![Salida de consola del comando anterior. El valor de key1 se resalta en un cuadro verde.](../media/databricks-command-output-3.png)

## <a name="create-a-key-vault-and-set-a-secret"></a>Creación de una instancia de Key Vault y establecimiento de un secreto

Va a crear una instancia de Key Vault mediante el siguiente comando. Este comando mostrará también los metadatos de la instancia de Key Vault. Copie los valores del **identificador** y de **vaultUri.**

```azurecli
az keyvault create --name contosoKeyVault10 --resource-group contosoResourceGroup5 --location eastus
```

![Imagen](../media/databricks-command-output-4.png)
![salida de consola del comando anterior. El identificador y el vaultUri están resaltados en verde para que el usuario los vea.](../media/databricks-command-output-5.png)

Para crear el secreto, use el comando siguiente. Establezca el valor del secreto en el valor **key1** de la cuenta de almacenamiento.

```azurecli
az keyvault secret set --vault-name contosoKeyVault10 --name storageKey --value "value of your key1"
```

## <a name="create-an-azure-databricks-workspace-and-add-key-vault-secret-scope"></a>Creación de un área de trabajo de Azure Databricks e incorporación de un ámbito de secreto de Key Vault

Esta sección no se puede completar mediante la línea de comandos. Siga esta [guía](https://docs.microsoft.com/azure/azure-databricks/store-secrets-azure-key-vault#create-an-azure-databricks-workspace-and-add-a-secret-scope). Tendrá que acceder a [Azure Portal](https://ms.portal.azure.com/#home) para:

1. Crear el recurso de Azure Databricks.
1. Iniciar el área de trabajo.
1. Crear un ámbito de secreto compatible con Key Vault.

## <a name="access-your-blob-container-from-azure-databricks-workspace"></a>Acceder al contenedor de blobs desde el área de trabajo de Azure Databricks.

Esta sección no se puede completar mediante la línea de comandos. Siga esta [guía](https://docs.microsoft.com/azure/azure-databricks/store-secrets-azure-key-vault#access-your-blob-container-from-azure-databricks). Tendrá que usar el área de trabajo Azure Databricks para:

1. Crear un **nuevo clúster**.
1. Crear un **Nuevo Notebook**.
1. Rellene los campos correspondientes en el script de Python.
1. Ejecute el script de Python.

```python
dbutils.fs.mount(
source = "wasbs://<your-container-name>@<your-storage-account-name>.blob.core.windows.net",
mount_point = "/mnt/<mount-name>",
extra_configs = {"<conf-key>":dbutils.secrets.get(scope = "<scope-name>", key = "<key-name>")})

df = spark.read.text("/mnt/<mount-name>/<file-name>")

df.show()
```

## <a name="next-steps"></a>Pasos siguientes

Asegúrese de que el almacén de claves se pueda recuperar:
> [!div class="nextstepaction"]
> [Limpieza de los recursos](https://docs.microsoft.com/azure/azure-resource-manager/management/delete-resource-group?tabs=azure-powershell)
