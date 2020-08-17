---
title: 'Inicio rápido: Copia de seguridad de una máquina virtual con la CLI de Azure'
description: En esta guía de inicio rápido, aprenda a crear un almacén de Recovery Services, a habilitar la protección en una VM y a crear el punto de recuperación inicial con la CLI de Azure.
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 01/31/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: b263d5394d642301c5fc53e169186e4aea56ecab
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497467"
---
# <a name="back-up-a-virtual-machine-in-azure-with-the-cli"></a>Copia de seguridad de una máquina virtual en Azure con la CLI

La CLI de Azure se usa para crear y administrar recursos de Azure desde la línea de comandos o en scripts. Para proteger sus datos realice copias de seguridad a intervalos regulares. Azure Backup crea puntos de recuperación que se guardan en almacenes de recuperación con redundancia geográfica. En este artículo se explica cómo realizar una copia de seguridad de una máquina virtual (VM) en Azure con la CLI de Azure. Estos pasos también se pueden llevar a cabo con [Azure PowerShell](quick-backup-vm-powershell.md) o en [Azure Portal](quick-backup-vm-portal.md).

Esta guía de inicio rápido permite realizar copias de seguridad en una máquina virtual de Azure existente. Si necesita crear una máquina virtual, puede [crearla con la CLI de Azure](../virtual-machines/linux/quick-create-cli.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Para instalar y usar la CLI localmente, debe ejecutar la versión 2.0.18 de la CLI de Azure o una posterior. Para averiguar la versión de la CLI, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="create-a-recovery-services-vault"></a>Creación de un almacén de Servicios de recuperación

Un almacén de Recovery Services es un contenedor lógico que almacena los datos de copia de seguridad de los recursos protegidos, como las máquinas virtuales de Azure. Cuando se ejecuta el trabajo de copia de seguridad para un recurso protegido, crea un punto de recuperación en el almacén de Recovery Services. Posteriormente, se puede usar uno de estos puntos de recuperación para restaurar los datos a un momento dado en el tiempo.

Cree un almacén de Recovery Services con [az backup vault create](/cli/azure/backup/vault#az-backup-vault-create). Especifique el mismo grupo de recursos y ubicación que tenga la máquina virtual que desea proteger. Si ha usado el [inicio rápido de la máquina virtual](../virtual-machines/linux/quick-create-cli.md), habrá creado lo siguiente:

- un grupo de recursos denominado *myResourceGroup*;
- una máquina virtual denominada *myVM*;
- recursos en la ubicación *eastus*.

```azurecli-interactive
az backup vault create --resource-group myResourceGroup \
    --name myRecoveryServicesVault \
    --location eastus
```

De forma predeterminada, el almacén de Recovery Services se establece para el almacenamiento con redundancia geográfica. El almacenamiento con redundancia geográfica garantiza que se repliquen los datos de copia de seguridad en una región de Azure secundaria que se encuentra a cientos de kilómetros de distancia de la región primaria. Si es necesario modificar la configuración de la redundancia del almacenamiento, utilice el cmdlet [az backup vault backup-properties set](/cli/azure/backup/vault/backup-properties?view=azure-cli-latest#az-backup-vault-backup-properties-set).

```azurecli
az backup vault backup-properties set \
    --name myRecoveryServicesVault  \
    --resource-group myResourceGroup \
    --backup-storage-redundancy "LocallyRedundant/GeoRedundant"
```

## <a name="enable-backup-for-an-azure-vm"></a>Habilitación de la copia de seguridad de una máquina virtual de Azure

Cree y use directivas para definir cuándo se ejecuta un trabajo de copia de seguridad y durante cuánto tiempo se almacenan los puntos de recuperación. La directiva de protección predeterminada ejecuta un trabajo de copia de seguridad cada día y conserva los puntos de seguridad durante 30 días. Estos valores de la directiva predeterminada se pueden usar para proteger rápidamente la máquina virtual. Para habilitar la protección mediante copia de seguridad de una máquina virtual, use [az backup protection enable-for-vm](/cli/azure/backup/protection#az-backup-protection-enable-for-vm). Especifique el grupo de recursos y la máquina virtual que se van a proteger, y la directiva que se va a usar:

```azurecli-interactive
az backup protection enable-for-vm \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --vm myVM \
    --policy-name DefaultPolicy
```

> [!NOTE]
> Si la máquina virtual no está en el mismo grupo de recursos que el almacén, myResourceGroup hará referencia al grupo de recursos donde se creó el almacén. En lugar del nombre de la máquina virtual, proporcione el identificador de la máquina virtual como se indica a continuación.

```azurecli-interactive
az backup protection enable-for-vm \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --vm $(az vm show -g VMResourceGroup -n MyVm --query id | tr -d '"') \
    --policy-name DefaultPolicy
```

> [!IMPORTANT]
> Al usar la CLI para habilitar la copia de seguridad de varias máquinas virtuales a la vez, asegúrese de que una sola directiva no tenga más de 100 máquinas virtuales asociadas. Este es un [procedimiento recomendado](./backup-azure-vm-backup-faq.md#is-there-a-limit-on-number-of-vms-that-can-beassociated-with-the-same-backup-policy). Actualmente, el cliente de PS no bloquea explícitamente si hay más de 100 máquinas virtuales, pero está previsto que la comprobación se agregue en el futuro.

## <a name="start-a-backup-job"></a>Inicio de un trabajo de copia de seguridad

Para iniciar una copia de seguridad ahora, en lugar de esperar a que la directiva predeterminada ejecute el trabajo en el momento programado, utilice [az backup protection backup-now](/cli/azure/backup/protection#az-backup-protection-backup-now). El primer trabajo de copia de seguridad crea un punto de recuperación completa. Cada uno de los trabajo de copia de seguridad posteriores a esta copia de seguridad inicial crea puntos de recuperación incremental. Los puntos de recuperación incremental ahorran tiempo y espacio de almacenamiento, ya que solo transfieren los cambios realizados desde la última copia de seguridad.

Los siguientes parámetros se utilizan para hacer una copia de seguridad de la máquina virtual:

- `--container-name` es el nombre de la máquina virtual
- `--item-name` es el nombre de la máquina virtual
- El valor `--retain-until` se debe establecerse en la última fecha disponible, en formato de hora UTC (**dd-mm-aaaa**), en que desea que el punto de recuperación esté disponible

En el ejemplo siguiente se realiza una copia de la máquina virtual denominada *myVM* y se establece la expiración del punto de recuperación para el 18 de octubre de 2017:

```azurecli-interactive
az backup protection backup-now \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --container-name myVM \
    --item-name myVM \
    --retain-until 18-10-2017
```

## <a name="monitor-the-backup-job"></a>Supervisión del trabajo de copia de seguridad

Para supervisar el estado de los trabajos de copia de seguridad, use [az backup job list](/cli/azure/backup/job#az-backup-job-list):

```azurecli-interactive
az backup job list \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --output table
```

La salida es similar al ejemplo siguiente, que muestra que el estado del trabajo de copia de seguridad es *InProgress*:

```output
Name      Operation        Status      Item Name    Start Time UTC       Duration
--------  ---------------  ----------  -----------  -------------------  --------------
a0a8e5e6  Backup           InProgress  myvm         2017-09-19T03:09:21  0:00:48.718366
fe5d0414  ConfigureBackup  Completed   myvm         2017-09-19T03:03:57  0:00:31.191807
```

Cuando el apartado *Status* (Estado) del trabajo de copia de seguridad muestre *Completed* (Completado), la máquina virtual estará protegida con Recovery Services y tendrá almacenado un punto de recuperación completa.

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Cuando deje de ser necesaria, puede deshabilitar la protección en la máquina virtual, quitar los puntos de restauración y el almacén de Recovery Services, y eliminar tanto el grupo de recursos como los recursos de la máquina virtual asociados. Si ha usado una máquina virtual existente, puede omitir el última comando [az group delete](/cli/azure/group?view=azure-cli-latest#az-group-delete) para dejar tanto el grupo de recursos como la máquina virtual en su lugar.

Si desea seguir un tutorial de Backup en el que se explique cómo restaurar los datos en una máquina virtual, vaya a [Pasos siguientes](#next-steps).

```azurecli-interactive
az backup protection disable \
    --resource-group myResourceGroup \
    --vault-name myRecoveryServicesVault \
    --container-name myVM \
    --item-name myVM \
    --delete-backup-data true
az backup vault delete \
    --resource-group myResourceGroup \
    --name myRecoveryServicesVault \
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado un almacén de Recovery Services, ha habilitado la protección en una máquina virtual y ha creado el punto de recuperación inicial. Para más información acerca de Azure Backup, y Recovery Services, continúe con los tutoriales.

> [!div class="nextstepaction"]
> [Copia de seguridad de varias máquinas virtuales de Azure](./tutorial-backup-vm-at-scale.md)
