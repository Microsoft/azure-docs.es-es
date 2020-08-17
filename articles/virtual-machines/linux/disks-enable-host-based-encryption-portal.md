---
title: Habilitación del cifrado de un extremo a otro mediante el cifrado en host con discos administrados - Azure Portal
description: Use el cifrado en el host para habilitar el cifrado de un extremo a otro en los discos administrados de Azure mediante Azure Portal.
author: roygara
ms.service: virtual-machines
ms.topic: how-to
ms.date: 07/23/2020
ms.author: rogarana
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: e969fe34e7e7723218586f24807ad70944ba5716
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87171522"
---
# <a name="enable-end-to-end-encryption-using-encryption-at-host---azure-portal"></a>Habilitación del cifrado de un extremo a otro mediante el cifrado en host - Azure Portal

Cuando se habilita el cifrado en el host, los datos almacenados en el host de máquina virtual se cifran en reposo y se transmiten cifrados al servido Storage. Para obtener información conceptual sobre el cifrado en el host, así como otros tipos de cifrado de disco administrado, consulte [Cifrado en host: cifrado de un extremo a otro para los datos de la VM](disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).

[!INCLUDE [virtual-machines-disks-encryption-at-host-portal](../../../includes/virtual-machines-disks-encryption-at-host-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

[Ejemplos de plantillas de Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/EncryptionAtHost)