---
title: Administración de adaptadores de red para la recuperación ante desastres con Azure Site Recovery
description: Se describe cómo administrar las interfaces de red para la recuperación ante desastres local en Azure con Azure Site Recovery.
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 4/9/2019
ms.author: harshacs
ms.openlocfilehash: 4dad7f76edf34782131c7c844978763cda53acc7
ms.sourcegitcommit: 1fe5127fb5c3f43761f479078251242ae5688386
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/14/2020
ms.locfileid: "90068122"
---
# <a name="manage-vm-network-interfaces-for-on-premises-disaster-recovery-to-azure"></a>Administración de interfaces de red de máquina virtual para la recuperación ante desastres local en Azure

Una máquina virtual (VM) en Azure debe tener al menos una interfaz de red conectada. Puede tener tantas interfaces de red conectadas como admita el tamaño de la máquina virtual.

De forma predeterminada, la primera interfaz de red de una máquina virtual de Azure se define como la interfaz de red principal. Todas las demás interfaces de red de la máquina virtual son interfaces de red secundarias. También de forma predeterminada, todo el tráfico saliente de la máquina virtual se envía a la dirección IP asignada a la configuración de IP principal de la interfaz de red principal.

En un entorno local, las máquinas virtuales o los servidores pueden tener varias interfaces de red para redes diferentes dentro del entorno. Se suelen usar redes diferentes para realizar operaciones específicas, como actualizaciones, mantenimiento y acceso a Internet. Durante la migración o la conmutación por error a Azure desde un entorno local, tenga en cuenta que las interfaces de red en la misma máquina virtual deben estar todas conectadas a la misma red virtual.

De forma predeterminada, Azure Site Recovery crea tantas interfaces de red en una máquina virtual de Azure como unidades conectadas al servidor local. Puede evitar la creación de interfaces de red redundantes durante la migración o la conmutación por error mediante la edición de la configuración de la interfaz de red en la configuración de la máquina virtual replicada.

## <a name="select-the-target-network"></a>Seleccionar la red de destino

Para VMware y las máquinas físicas, así como para las máquinas virtuales de Hyper-V (sin System Center Virtual Machine Manager), puede especificar la red virtual de destino para máquinas virtuales individuales. Para las máquinas virtuales de Hyper-V administradas con Virtual Machine Manager, use [asignación de red](./hyper-v-vmm-network-mapping.md) para asignar redes de máquinas virtuales en un servidor de Virtual Machine Manager de origen y redes de Azure de destino.

1. En **Elementos replicados** en un almacén de Recovery Services, seleccione cualquier elemento replicado para acceder a la configuración de ese elemento replicado.

2. Seleccione la pestaña **Proceso y red** para acceder a la configuración de red del elemento replicado.

3. En **Propiedades de red**, elija una red virtual en la lista de interfaces de red disponibles.

    ![Configuración de red](./media/site-recovery-manage-network-interfaces-on-premises-to-azure/compute-and-network.png)

La modificación de la red de destino afecta a todas las interfaces de red para esa máquina virtual específica.

Para las nubes de Virtual Machine Manager, la modificación de la asignación de red afecta a todas las máquinas virtuales y sus interfaces de red.

## <a name="select-the-target-interface-type"></a>Seleccionar el tipo de interfaz de destino

En la sección **Interfaces de red** del panel **Proceso y red**, puede ver y editar la configuración de la interfaz de red. También puede especificar el tipo de interfaz de red de destino.

- Una interfaz de red **principal** es necesaria para la conmutación por error.
- Todas las demás interfaces de red seleccionadas, si hay alguna, son interfaces de red **secundarias**.
- Seleccione **No usar** para excluir la creación de una interfaz de red en la conmutación por error.

De forma predeterminada, al habilitar la replicación, Site Recovery selecciona todas las interfaces de red detectadas en el servidor local. Marca una como **principal** y todas las demás como **secundarias**. Las interfaces siguientes que se agregan al servidor local se marcan con **No usar** de forma predeterminada. Cuando agregue más interfaces de red, asegúrese de que está seleccionado el tamaño de destino de la máquina virtual de Azure correcto para dar cabida a todas las interfaces de red necesarias.

## <a name="modify-network-interface-settings"></a>Modificar la configuración de la interfaz de red

Puede modificar la subred y la dirección IP de las interfaces de red de un elemento replicado. Si no se especifica una dirección IP, Site Recovery asignará la siguiente dirección IP disponible de la subred a la interfaz de red en la conmutación por error.

1. Seleccione cualquier interfaz de red disponible para abrir la configuración de la interfaz de red.

2. Elija la subred deseada en la lista de subredes disponibles.

3. Escriba la dirección IP deseada (según sea necesario).

    ![Configuración de la interfaz de red](./media/site-recovery-manage-network-interfaces-on-premises-to-azure/network-interface-settings.png)

4. Seleccione **Aceptar** para finalizar la edición y volver al panel **Proceso y red**.

5. Repita los pasos 1 a 4 para otras interfaces de red.

6. Seleccione **Guardar** para guardar todos los cambios.

## <a name="next-steps"></a>Pasos siguientes
  [Obtenga más información](../virtual-network/virtual-network-network-interface-vm.md) sobre las interfaces de red para máquinas virtuales de Azure.
