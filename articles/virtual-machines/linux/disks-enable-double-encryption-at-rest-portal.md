---
title: Habilitación del cifrado doble en reposo con discos administrados - Azure Portal
description: Habilite el cifrado doble en reposo para los datos de discos administrados mediante Azure Portal.
author: roygara
ms.date: 07/23/2020
ms.topic: how-to
ms.author: rogarana
ms.service: virtual-machines-linux
ms.subservice: disks
ms.custom: references_regions
ms.openlocfilehash: 7fefaa3b13b73db119da696738af66e84f8bdd63
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87136001"
---
# <a name="azure-portal---enable-double-encryption-at-rest-on-your-managed-disks"></a>Azure Portal: habilitación del cifrado doble en reposo en discos administrados

Azure Disk Storage admite el cifrado doble en reposo para los discos administrados. Para obtener información conceptual sobre el cifrado doble en reposo, así como otros tipos de cifrado de discos administrados, consulte la sección [Doble cifrado en reposo](disk-encryption.md#double-encryption-at-rest) de nuestro artículo sobre el cifrado de discos.

[!INCLUDE [virtual-machines-double-encryption-at-rest-portal](../../../includes/virtual-machines-disks-double-encryption-at-rest-portal.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Ejemplos de plantillas de Azure Resource Manager](https://github.com/Azure-Samples/managed-disks-powershell-getting-started/tree/master/DoubleEncryption)
- [Habilitación de claves administradas por el cliente con el cifrado del lado servidor: ejemplos](disks-enable-customer-managed-keys-cli.md#examples)