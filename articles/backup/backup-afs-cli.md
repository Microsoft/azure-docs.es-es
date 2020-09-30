---
title: Copia de seguridad de recursos compartidos de archivos de Azure con la CLI de Azure
description: Aprenda a utilizar la CLI de Azure para hacer copias de seguridad de recursos compartidos de archivos de Azure en el almacén de Recovery Services.
ms.topic: conceptual
ms.date: 01/14/2020
ms.openlocfilehash: cc4422a7d20dbd231729922bd013549d5276deb1
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89182214"
---
# <a name="back-up-azure-file-shares-with-cli"></a>Copia de seguridad de recursos compartidos de archivos de Azure con la CLI

La interfaz de la línea de comandos (CLI) de Azure ofrece una experiencia de línea de comandos para administrar los recursos de Azure. Es una herramienta excelente para personalizar la automatización del uso de los recursos de Azure. En este artículo se detalla cómo realizar una copia de seguridad de los recursos compartidos de archivos de Azure con la CLI de Azure. Estos pasos también se pueden llevar a cabo con [Azure PowerShell](./backup-azure-afs-automation.md) o en [Azure Portal](backup-afs.md).

Al final de este tutorial, habrá aprendido a realizar las siguientes operaciones mediante la CLI de Azure:

* Creación de un almacén de Recovery Services
* Habilitación de la copia de seguridad de los recursos compartidos de archivos de Azure
* Desencadenamiento de una copia de seguridad a petición para recursos compartidos de archivos

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Para instalar y usar la CLI localmente, debe ejecutar la versión 2.0.18 de la CLI de Azure o una posterior. Para averiguar la versión de la CLI, `run az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Recovery Services

Un almacén de Recovery Services es una entidad que proporciona funcionalidad de administración y una vista consolidada de todos los elementos de copia de seguridad. Cuando se ejecuta el trabajo de copia de seguridad para un recurso protegido, crea un punto de recuperación en el almacén de Recovery Services. Posteriormente, se puede usar uno de estos puntos de recuperación para restaurar los datos a un momento dado en el tiempo.

Siga estos pasos para crear un almacén de Recovery Services:

1. Un almacén se coloca en un grupo de recursos. Si no tiene un grupo de recursos, cree uno con [az group create](/cli/azure/group?view=azure-cli-latest#az-group-create). En este tutorial, se creará el nuevo grupo de recursos *azurefiles* en la región Este de EE. UU.

    ```azurecli-interactive
    az group create --name AzureFiles --location eastus --output table
    ```

    ```output
    Location    Name
    ----------  ----------
    eastus      AzureFiles
    ```

1. Use el cmdlet [az backup vault create](/cli/azure/backup/vault?view=azure-cli-latest#az-backup-vault-create) para crear el almacén. Especifique para el almacén la misma ubicación del grupo de recursos.

    En el ejemplo siguiente se crea un almacén de Recovery Services denominado *azurefilesvault* en la región del este de EE. UU.

    ```azurecli-interactive
    az backup vault create --resource-group azurefiles --name azurefilesvault --location eastus --output table
    ```

    ```output
    Location    Name                ResourceGroup
    ----------  ----------------    ---------------
    eastus      azurefilesvault     azurefiles
    ```

## <a name="enable-backup-for-azure-file-shares"></a>Habilitación de la copia de seguridad de los recursos compartidos de archivos de Azure

En esta sección se supone que ya tiene un recurso compartido de archivos de Azure para el que desea configurar la copia de seguridad. Si no lo tiene, cree un recurso compartido de archivos de Azure mediante el comando [az storage share create](/cli/azure/storage/share?view=azure-cli-latest#az-storage-share-create).

Para habilitar la copia de seguridad de recursos compartidos de archivos, debe crear una directiva de protección que defina cuándo se ejecuta un trabajo de copia de seguridad y durante cuánto tiempo se almacenan los puntos de recuperación. Puede crear una directiva de copia de seguridad mediante el cmdlet [az backup policy create](/cli/azure/backup/policy?view=azure-cli-latest#az-backup-policy-create).

En el ejemplo siguiente se usa el cmdlet [az backup protection enable-for-azurefileshare](/cli/azure/backup/protection?view=azure-cli-latest#az-backup-protection-enable-for-azurefileshare) para habilitar la copia de seguridad para el recurso compartido de archivos *azurefiles* en la cuenta de almacenamiento *afsaccount* mediante la directiva de copia de seguridad *schedule 1*.

```azurecli-interactive
az backup protection enable-for-azurefileshare --vault-name azurefilesvault --resource-group  azurefiles --policy-name schedule1 --storage-account afsaccount --azure-file-share azurefiles  --output table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
0caa93f4-460b-4328-ac1d-8293521dd928  azurefiles
```

El atributo **Name** de la salida corresponde al nombre del trabajo creado por el servicio de copia de seguridad para la operación de **habilitación de copia de seguridad**. Para realizar el seguimiento del estado del trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-show).

## <a name="trigger-an-on-demand-backup-for-file-share"></a>Desencadenamiento de una copia de seguridad a petición para recursos compartidos de archivos

Si desea desencadenar una copia de seguridad a petición para el recurso compartido de archivos en lugar de esperar a que la directiva de copia de seguridad ejecute el trabajo a la hora programada, use el cmdlet [az backup protection backup-now](/cli/azure/backup/protection?view=azure-cli-latest#az-backup-protection-backup-now).

Debe definir los parámetros siguientes para desencadenar una copia de seguridad a petición:

* **--container-name** es el nombre de la cuenta de almacenamiento que hospeda el recurso compartido de archivos. Para recuperar el **nombre** o **nombre descriptivo** del contenedor, use el comando [az backup container list](/cli/azure/backup/container?view=azure-cli-latest#az-backup-container-list).
* **--item-name** es el nombre del recurso compartido de archivos para el que desea desencadenar una copia de seguridad a petición. Para recuperar el **nombre** o **nombre descriptivo** del elemento de copia de seguridad, use el comando [az backup item list](/cli/azure/backup/item?view=azure-cli-latest#az-backup-item-list).
* **--retain-until** especifica la fecha hasta la que desea conservar el punto de recuperación. El valor debe establecerse en formato de hora UTC (dd-mm-aaaa).

En el ejemplo siguiente se desencadena una copia de seguridad a petición para el recurso compartido de archivos *azuresfiles* en la cuenta de almacenamiento *afsaccount* con retención hasta el *20-01-2020*.

```azurecli-interactive
az backup protection backup-now --vault-name azurefilesvault --resource-group azurefiles --container-name "StorageContainer;Storage;AzureFiles;afsaccount" --item-name "AzureFileShare;azurefiles" --retain-until 20-01-2020 --output table
```

```output
Name                                  ResourceGroup
------------------------------------  ---------------
9f026b4f-295b-4fb8-aae0-4f058124cb12  azurefiles
```

El atributo **Name** de la salida corresponde al nombre del trabajo creado por el servicio de copia de seguridad para la operación de "copia de seguridad a petición". Para realizar el seguimiento del estado de un trabajo, use el cmdlet [az backup job show](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-show).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre cómo [restaurar recursos compartidos de archivos de Azure con la CLI](restore-afs-cli.md)
* Más información sobre cómo [administrar copias de seguridad de recursos compartidos de archivos de Azure con la CLI](manage-afs-backup-cli.md)
