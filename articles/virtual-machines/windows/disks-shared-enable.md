---
title: Habilitación de los discos compartidos para Azure Managed Disks
description: Configuración de un disco administrado de Azure con discos compartidos, con el fin de poder compartirlo entre varias máquinas virtuales
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 07/30/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: cc365f6eef4da61ab7de5113b8f4b2b185360740
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460069"
---
# <a name="enable-shared-disk"></a>Habilitación del disco compartido

En este artículo se describe cómo habilitar la característica de discos compartidos en Azure Managed Disks. Los discos compartidos de Azure son una nueva característica de Azure Managed Disks que permite conectar un disco administrado a varias máquinas virtuales (VM) al mismo tiempo. Si adjunta un disco administrado en varias VM, podrá implementar nuevas aplicaciones en clúster o migrar las existentes a Azure. 

Si busca información conceptual sobre los discos administrados que tienen habilitados discos compartidos, consulte [Discos compartidos de Azure](disks-shared.md).
[!INCLUDE [virtual-machines-enable-shared-disk](../../../includes/virtual-machines-enable-shared-disk.md)]