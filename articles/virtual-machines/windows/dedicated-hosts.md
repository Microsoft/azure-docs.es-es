---
title: Información general de hosts dedicados de Azure para máquinas virtuales
description: Obtenga más información sobre cómo se pueden usar los hosts dedicados de Azure para implementar máquinas virtuales.
author: cynthn
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.workload: infrastructure
ms.date: 07/28/2020
ms.author: cynthn
ms.reviewer: zivr
ms.openlocfilehash: 08d5f88ab018f9852da5bba5fe2230ef8148e914
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87386541"
---
# <a name="azure-dedicated-hosts"></a>Hosts dedicados de Azure

Azure Dedicated Host es un servicio que proporciona servidores físicos (capaces de hospedar una o varias máquinas virtuales) dedicados a una suscripción a Azure. Los hosts dedicados son los mismos servidores físicos que se usan en nuestros centros de datos y se proporcionan como un recurso. Puede aprovisionar hosts dedicados dentro de una región, zona de disponibilidad y dominio de error. Después, puede colocar las máquinas virtuales directamente en los hosts aprovisionados, en la configuración que más se ajuste a sus necesidades.


[!INCLUDE [virtual-machines-common-dedicated-hosts](../../../includes/virtual-machines-common-dedicated-hosts.md)]

## <a name="next-steps"></a>Pasos siguientes

- Puede implementar un host dedicado mediante [Azure PowerShell](dedicated-hosts-powershell.md), el [portal](dedicated-hosts-portal.md) y la [CLI de Azure](../linux/dedicated-hosts-cli.md).

- [Aquí](https://github.com/Azure/azure-quickstart-templates/blob/master/201-vm-dedicated-hosts/README.md) encontrará una plantilla de ejemplo en la que se usan zonas y dominios de error para obtener la máxima resistencia en una región.

- También puede ahorrar costos con una [instancia reservada de Azure Dedicated Host](../prepay-dedicated-hosts-reserved-instances.md).
