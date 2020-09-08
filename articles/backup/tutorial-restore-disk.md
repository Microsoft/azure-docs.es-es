---
title: 'Tutorial: Restauración de una máquina virtual con la CLI de Azure'
description: Obtenga información acerca de cómo restaurar un disco y crear una máquina virtual recuperada en Azure con Backup and Recovery Services.
ms.topic: tutorial
ms.date: 01/31/2019
ms.custom: mvc
ms.openlocfilehash: d93f3d24762f4b9a3da4a9e725d28810f6700fe0
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88890729"
---
# <a name="restore-a-vm-with-azure-cli"></a>Restauración de una máquina virtual con la CLI de Azure

Azure Backup crea puntos de recuperación que se almacenan en almacenes de recuperación con redundancia geográfica. Cuando se realiza una restauración desde un punto de recuperación, se puede restaurar toda una máquina virtual o archivos individuales. En este artículo se explica cómo restaurar una máquina virtual completa mediante la CLI. En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Enumerar y seleccionar puntos de recuperación
> * Restaurar un disco desde un punto de recuperación
> * Crear una máquina virtual a partir del disco restaurado

Para información sobre cómo usar PowerShell para restaurar un disco y crear una máquina virtual recuperada, consulte [Copia de seguridad y restauración de máquinas virtuales de Azure con PowerShell](backup-azure-vms-automation.md#restore-an-azure-vm).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.18 u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="prerequisites"></a>Prerrequisitos

Para este tutorial se necesita una máquina virtual Linux protegida con Azure Backup. Para simular un proceso de recuperación y eliminación de máquina virtual accidental, cree una máquina virtual desde un disco en un punto de recuperación. Si necesita una máquina virtual Linux que esté protegida con Azure Backup, consulte [Copia de seguridad de una máquina virtual en Azure con la CLI](quick-backup-vm-cli.md).

## <a name="backup-overview"></a>Introducción a Backup

Cuando Azure inicia una copia de seguridad, la extensión de copia de seguridad en la máquina virtual toma una instantánea de un momento dado. La extensión de copia de seguridad se instala en la máquina virtual cuando se solicita la primera copia de seguridad. Azure Backup también puede tomar una instantánea del almacenamiento subyacente, si la máquina virtual no se está ejecutando cuando se realiza la copia de seguridad.

De forma predeterminada, Azure Backup toma una copia de seguridad coherente del sistema de archivos. Después de que el servicio Azure Backup tome la instantánea, los datos se transfieren al almacén de Recovery Services. Para que el proceso resulte más eficaz, Azure Backup identifica y transfiere únicamente los bloques de datos que han cambiado desde la última copia de seguridad.

Cuando finaliza la transferencia de datos, se elimina la instantánea y se crea un punto de recuperación.

## <a name="list-available-recovery-points"></a>Lista de puntos de recuperación disponibles

Para restaurar un disco, debe seleccionar un punto de recuperación como el origen de los datos de recuperación. Dado que la directiva predeterminada crea un punto de recuperación cada día y lo mantiene durante 30 días, puede mantener un conjunto de puntos de recuperación que le permita seleccionar un punto concreto a tiempo para la recuperación.

Para ver una lista de los puntos de recuperación disponibles, use [az backup recoverypoint list](/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-list). El punto de recuperación **name** se usa para recuperar discos. En este tutorial, queremos usar el punto de recuperación más reciente disponible. El parámetro `--query [0].name` selecciona el nombre del punto de recuperación más reciente de la siguiente manera:

```azurecli-interactive
az backup recoverypoint list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --backup-management-type AzureIaasVM
    --container-name myVM \
    --item-name myVM \
    --query [0].name \
    --output tsv
```

## <a name="restore-a-vm-disk"></a>Restaurar un disco de máquina virtual

> [!IMPORTANT]
> Se recomienda usar la versión 2.0.74 de Az CLI o una versión posterior para obtener todas las ventajas de una restauración rápida, incluida la restauración de discos administrados. Es mejor usar siempre la versión más reciente.

### <a name="managed-disk-restore"></a>Restauración de un disco administrado

Si la máquina virtual con copia de seguridad tiene discos administrados y la intención es restaurarlos desde el punto de recuperación, primero debe proporcionar una cuenta de almacenamiento de Azure. Esta cuenta de almacenamiento se usa para almacenar la configuración de la máquina virtual y la plantilla de implementación que se pueden usar posteriormente para implementar la máquina virtual desde los discos restaurados. A continuación, también se proporciona un grupo de recursos de destino para los discos administrados en los que se va a realizar la restauración.

1. Para crear una cuenta de almacenamiento, use [az storage account create](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create). El nombre de la cuenta de almacenamiento debe estar en minúsculas y ser único globalmente. Reemplace *mystorageaccount* por su propio nombre único:

    ```azurecli-interactive
    az storage account create \
        --resource-group myResourceGroup \
        --name mystorageaccount \
        --sku Standard_LRS
    ```

2. Restaure el disco desde el punto de recuperación con [az backup restore restore-disks](/cli/azure/backup/restore?view=azure-cli-latest#az-backup-restore-restore-disks). Reemplace *mystorageaccount* por el nombre de la cuenta de almacenamiento que creó en el comando anterior. Reemplace *myRecoveryPointName* por el nombre del punto de recuperación que obtuvo en la salida del comando [az backup recoverypoint list](/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-list) anterior. ***También debe proporcionar el grupo de recursos de destino donde se van a restaurar los discos administrados***.

    ```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName \
        --target-resource-group targetRG
    ```

    > [!WARNING]
    > Si no se proporciona el parámetro **target-resource-group**, los discos administrados se restaurarán como discos no administrados en la cuenta de almacenamiento especificada. Esto tendrá consecuencias significativas en el tiempo de restauración, ya que el tiempo que se tarda en restaurar los discos por completo depende de la cuenta de almacenamiento especificada. Obtendrá la ventaja de la restauración instantánea solo cuando se proporcione el parámetro target-resource-group. Si la intención es restaurar los discos administrados como no administrados, no proporcione el parámetro **target-resource-group** y, en su lugar, use el parámetro **restore-as-unmanaged-disk**, como se muestra a continuación. Este parámetro está disponible desde az 3.4.0 en adelante.

    ```azurecli-interactive
    az backup restore restore-disks \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --container-name myVM \
    --item-name myVM \
    --storage-account mystorageaccount \
    --rp-name myRecoveryPointName \
    --restore-as-unmanaged-disk
    ```

De esta forma se restaurarán los discos administrados como no administrados en la cuenta de almacenamiento dada y no se aprovechará la funcionalidad de restauración instantánea. En versiones futuras de la CLI, será obligatorio proporcionar el parámetro **target-resource-group** o el parámetro **restore-as-unmanaged-disk**.

### <a name="unmanaged-disks-restore"></a>Restauración de discos no administrados

Si la máquina virtual con copia de seguridad tiene discos no administrados y la intención es restaurarlos desde el punto de recuperación, primero debe proporcionar una cuenta de almacenamiento de Azure. Esta cuenta de almacenamiento se usa para almacenar la configuración de la máquina virtual y la plantilla de implementación que se pueden usar posteriormente para implementar la máquina virtual desde los discos restaurados. De forma predeterminada, los discos no administrados se restaurarán a sus cuentas de almacenamiento originales. Si quiere restaurar todos los discos no administrados en un solo lugar, también puede usar la cuenta de almacenamiento especificada como ubicación provisional para esos discos.

En pasos adicionales, el disco restaurado se usa para crear una máquina virtual.

1. Para crear una cuenta de almacenamiento, use [az storage account create](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create). El nombre de la cuenta de almacenamiento debe estar en minúsculas y ser único globalmente. Reemplace *mystorageaccount* por su propio nombre único:

    ```azurecli-interactive
    az storage account create \
        --resource-group myResourceGroup \
        --name mystorageaccount \
        --sku Standard_LRS
    ```

2. Restaure el disco desde el punto de recuperación con [az backup restore restore-disks](/cli/azure/backup/restore?view=azure-cli-latest#az-backup-restore-restore-disks). Reemplace *mystorageaccount* por el nombre de la cuenta de almacenamiento que creó en el comando anterior. Reemplace *myRecoveryPointName* por el nombre del punto de recuperación que obtuvo en la salida del comando [az backup recoverypoint list](/cli/azure/backup/recoverypoint?view=azure-cli-latest#az-backup-recoverypoint-list) anterior:

    ```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName
    ```

Como se mencionó anteriormente, los discos no administrados se restaurarán a su cuenta de almacenamiento original. Esto proporciona el mejor rendimiento de restauración. Pero si todos los discos no administrados deben restaurarse a una cuenta de almacenamiento determinada, use la marca pertinente como se muestra a continuación.

```azurecli-interactive
    az backup restore restore-disks \
        --resource-group myResourceGroup \
        --vault-name myRecoveryServicesVault \
        --container-name myVM \
        --item-name myVM \
        --storage-account mystorageaccount \
        --rp-name myRecoveryPointName \
        --restore-to-staging-storage-account
    ```

## Monitor the restore job

To monitor the status of restore job, use [az backup job list](/cli/azure/backup/job?view=azure-cli-latest#az-backup-job-list):

```azurecli-interactive
az backup job list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --output table
```

El resultado es similar al ejemplo siguiente, que muestra que el estado del trabajo de restauración es *InProgress*:

```output
Name      Operation        Status      Item Name    Start Time UTC       Duration
--------  ---------------  ----------  -----------  -------------------  --------------
7f2ad916  Restore          InProgress  myvm         2017-09-19T19:39:52  0:00:34.520850
a0a8e5e6  Backup           Completed   myvm         2017-09-19T03:09:21  0:15:26.155212
fe5d0414  ConfigureBackup  Completed   myvm         2017-09-19T03:03:57  0:00:31.191807
```

Cuando el valor de *Estado* del trabajo de restauración es *Completado*, la información necesaria (configuración de la máquina virtual y la plantilla de implementación) se ha restaurado en la cuenta de almacenamiento.

## <a name="create-a-vm-from-the-restored-disk"></a>Crear una máquina virtual a partir del disco restaurado

El último paso es crear una máquina virtual a partir de los discos restaurados. Puede usar la plantilla de implementación descargada en la cuenta de almacenamiento especificada para crear la máquina virtual.

### <a name="fetch-the-job-details"></a>Captura de los detalles del trabajo

Los detalles del trabajo resultante ofrecen la plantilla de URI que se puede consultar e implementar. Use el comando job show para obtener más detalles del trabajo restaurado desencadenado.

```azurecli-interactive
az backup job show \
    -v myRecoveryServicesVault \
    -g myResourceGroup \
    -n 1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414
```

La salida de esta consulta proporcionará todos los detalles, pero solo nos interesa el contenido de la cuenta de almacenamiento. Se puede usar la [funcionalidad de consulta](/cli/azure/query-azure-cli?view=azure-cli-latest) de la CLI de Azure para capturar los detalles pertinentes.

```azurecli-interactive
az backup job show \
    -v myRecoveryServicesVault \
    -g myResourceGroup \
    -n 1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414 \
    --query properties.extendedInfo.propertyBag

{
  "Config Blob Container Name": "myVM-daa1931199fd4a22ae601f46d8812276",
  "Config Blob Name": "config-myVM-1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414.json",
  "Config Blob Uri": "https://mystorageaccount.blob.core.windows.net/myVM-daa1931199fd4a22ae601f46d8812276/config-appvm8-1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json",
  "Job Type": "Recover disks",
  "Recovery point time ": "12/25/2019 10:07:11 PM",
  "Target Storage Account Name": "mystorageaccount",
  "Target resource group": "mystorageaccountRG",
  "Template Blob Uri": "https://mystorageaccount.blob.core.windows.net/myVM-daa1931199fd4a22ae601f46d8812276/azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json"
}
```

### <a name="fetch-the-deployment-template"></a>Captura de la plantilla de implementación

La plantilla no es accesible directamente, ya que está en la cuenta de almacenamiento de un cliente y un contenedor concreto. Necesitamos la dirección URL completa (junto con un token de SAS temporal) para acceder a ella.

En primer lugar, extraiga el URI del blob de plantilla de los detalles del trabajo

```azurecli-interactive
az backup job show \
    -v myRecoveryServicesVault \
    -g myResourceGroup \
    -n 1fc2d55d-f0dc-4ca6-ad48-aca0fe5d0414 \
    --query properties.extendedInfo.propertyBag."""Template Blob Uri"""

"https://mystorageaccount.blob.core.windows.net/myVM-daa1931199fd4a22ae601f46d8812276/azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json"
```

El URI del blob de plantilla tendrá este formato y servirá para extraer el nombre de la plantilla

```https
https://<storageAccountName.blob.core.windows.net>/<containerName>/<templateName>
```

Por lo tanto, el nombre de plantilla del ejemplo anterior será ```azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json``` y el del contenedor, ```myVM-daa1931199fd4a22ae601f46d8812276```

Ahora, obtenga el token de SAS para el contenedor y la plantilla como se detalla [aquí](../azure-resource-manager/templates/secure-template-with-sas-token.md?tabs=azure-cli#provide-sas-token-during-deployment)

```azurecli-interactive
expiretime=$(date -u -d '30 minutes' +%Y-%m-%dT%H:%MZ)
connection=$(az storage account show-connection-string \
    --resource-group mystorageaccountRG \
    --name mystorageaccount \
    --query connectionString)
token=$(az storage blob generate-sas \
    --container-name myVM-daa1931199fd4a22ae601f46d8812276 \
    --name azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json \
    --expiry $expiretime \
    --permissions r \
    --output tsv \
    --connection-string $connection)
url=$(az storage blob url \
   --container-name myVM-daa1931199fd4a22ae601f46d8812276 \
    --name azuredeploy1fc2d55d-f0dc-4ca6-ad48-aca0519c0232.json \
    --output tsv \
    --connection-string $connection)
```

### <a name="deploy-the-template-to-create-the-vm"></a>Implementación de la plantilla para crear la máquina virtual

Ahora, implemente la plantilla para crear la máquina virtual como se explica [aquí](../azure-resource-manager/templates/deploy-cli.md).

```azurecli-interactive
az group deployment create \
  --resource-group ExampleGroup \
  --template-uri $url?$token
```

Para comprobar que la máquina virtual se ha creado desde el disco recuperado, enumere las máquinas virtuales del grupo de recursos con [az vm list](/cli/azure/vm?view=azure-cli-latest#az-vm-list) como se indica a continuación:

```azurecli-interactive
az vm list --resource-group myResourceGroup --output table
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha restaurado un disco desde un punto de recuperación y, a continuación, ha creado una máquina virtual desde el disco. Ha aprendido a:

> [!div class="checklist"]
>
> * Enumerar y seleccionar puntos de recuperación
> * Restaurar un disco desde un punto de recuperación
> * Crear una máquina virtual a partir del disco restaurado

Avance hasta el siguiente tutorial para obtener información acerca de cómo restaurar archivos individuales desde un punto de recuperación.

> [!div class="nextstepaction"]
> [Restauración de archivos en una máquina virtual de Azure](tutorial-restore-files.md)
