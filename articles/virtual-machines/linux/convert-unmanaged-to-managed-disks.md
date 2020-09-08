---
title: Conversión de una máquina virtual Linux de discos no administrados a discos administrados
description: Cómo convertir una máquina virtual Linux de discos no administrados a discos administrados mediante la CLI de Azure.
author: roygara
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 12/15/2017
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: 89035d26e05ca4b93dc271f02fef1afea24b0f20
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815341"
---
# <a name="convert-a-linux-virtual-machine-from-unmanaged-disks-to-managed-disks"></a>Conversión de una máquina virtual Linux con discos no administrados en discos administrados

Si ya dispone de máquinas virtuales (VM) con Linux que usan Unmanaged Disks, puede convertirlas para usar [Azure Managed Disks](../managed-disks-overview.md). Este proceso convierte el disco del SO y los discos de datos conectados.

En este artículo se muestra cómo convertir máquinas virtuales con la CLI de Azure. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

## <a name="before-you-begin"></a>Antes de empezar
* Revise las [preguntas frecuentes sobre la migración a Managed Disks](../faq-for-disks.md#migrate-to-managed-disks).

[!INCLUDE [virtual-machines-common-convert-disks-considerations](../../../includes/virtual-machines-common-convert-disks-considerations.md)]

* No se eliminan los discos duros virtuales originales ni la cuenta de almacenamiento usada por la máquina virtual antes de la conversión. Seguirán acumulando cargos. Para evitar que se le facture por estos artefactos, elimine los blobs de los discos duros virtuales originales después de comprobar que la conversión esté completa. Si tiene que buscar estos discos no conectados con el fin de eliminarlos, consulte nuestro artículo [Búsqueda y eliminación de discos administrados y no administrados de Azure no conectados](find-unattached-disks.md).

## <a name="convert-single-instance-vms"></a>Conversión de máquinas virtuales de instancia única
En esta sección se explica cómo convertir máquinas virtuales de Azure de instancia única de Unmanaged Disks a Managed Disks. (Si las máquinas virtuales se encuentran en un conjunto de disponibilidad, consulte la sección siguiente). Puede usar este proceso para convertir las máquinas virtuales de discos no administrados premium (SDD) a discos administrados premium, o bien de discos no administrados estándar (HDD) a discos administrados estándar.

1. Desasigne la máquina virtual mediante [az vm deallocate](/cli/azure/vm). En el ejemplo siguiente se desasigna la VM `myVM` en el grupo de recursos denominado `myResourceGroup`:

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

2. Convierta la máquina virtual en discos administrados con [az vm convert](/cli/azure/vm). El proceso siguiente convierte la máquina virtual denominada `myVM`, incluidos el disco del SO y todos los discos de datos:

    ```azurecli
    az vm convert --resource-group myResourceGroup --name myVM
    ```

3. Inicie la máquina virtual después de la conversión a discos administrados con [az vm start](/cli/azure/vm). En el ejemplo siguiente se inicia la VM llamada `myVM` en el grupo de recursos `myResourceGroup`.

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```

## <a name="convert-vms-in-an-availability-set"></a>Conversión de VM de un conjunto de disponibilidad

Si las VM que desea convertir en discos administrados se encuentran en un conjunto de disponibilidad, primero debe convertir el conjunto de disponibilidad en un conjunto de disponibilidad administrado.

Todas las VM del conjunto de disponibilidad deben desasignarse antes de convertir el conjunto de disponibilidad. Planee la conversión de todas las máquinas virtuales a discos administrados después de haber convertido el propio conjunto de disponibilidad en un conjunto de disponibilidad administrado. Después, inicie todas las máquinas virtuales y siga trabajando con normalidad.

1. Enumere todas las máquinas virtuales de un conjunto de disponibilidad con [az vm availability-set list](/cli/azure/vm/availability-set). En el ejemplo siguiente se enumeran todas las VM del conjunto de disponibilidad `myAvailabilitySet` en el grupo de recursos denominado `myResourceGroup`:

    ```azurecli
    az vm availability-set show \
        --resource-group myResourceGroup \
        --name myAvailabilitySet \
        --query [virtualMachines[*].id] \
        --output table
    ```

2. Desasigne todas las máquinas virtuales con [az vm deallocate](/cli/azure/vm). En el ejemplo siguiente se desasigna la VM `myVM` en el grupo de recursos denominado `myResourceGroup`:

    ```azurecli
    az vm deallocate --resource-group myResourceGroup --name myVM
    ```

3. Convierta el conjunto de disponibilidad con [az vm availability-set convert](/cli/azure/vm/availability-set). En el ejemplo siguiente se convierte el conjunto de disponibilidad `myAvailabilitySet` en el grupo de recursos denominado `myResourceGroup`:

    ```azurecli
    az vm availability-set convert \
        --resource-group myResourceGroup \
        --name myAvailabilitySet
    ```

4. Convierta todas las máquinas virtuales a discos administrados con [az vm convert](/cli/azure/vm). El proceso siguiente convierte la máquina virtual denominada `myVM`, incluidos el disco del SO y todos los discos de datos:

    ```azurecli
    az vm convert --resource-group myResourceGroup --name myVM
    ```

5. Inicie todas las máquinas virtuales después de la conversión a discos administrados con [az vm start](/cli/azure/vm). En el ejemplo siguiente se inicia la máquina virtual llamada `myVM` en el grupo de recursos `myResourceGroup`:

    ```azurecli
    az vm start --resource-group myResourceGroup --name myVM
    ```

## <a name="convert-using-the-azure-portal"></a>Conversión mediante Azure Portal

Puede convertir discos no administrados en discos administrados mediante Azure Portal.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione la máquina virtual en la lista de máquinas virtuales en el portal.
3. En la hoja de la máquina virtual, seleccione **Discos** en el menú.
4. En la parte superior de la hoja **Discos**, seleccione **Migrar a discos administrados**.
5. Si la máquina virtual está en un conjunto de disponibilidad, habrá una advertencia en la hoja **Migrar a discos administrados** indicándole que debe convertir el conjunto de disponibilidad primero. La advertencia debería tener un vínculo en el que puede hacer clic para convertir el conjunto de disponibilidad. Una vez que se convierta el conjunto de disponibilidad o la máquina virtual no esté en un conjunto de disponibilidad, haga clic en **Migrar** para iniciar el proceso de migración de los discos a discos administrados.

La máquina virtual se detendrá y se reiniciará una vez completada la migración.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las opciones de almacenamiento, vea [Introducción a Azure Managed Disks](../managed-disks-overview.md).
