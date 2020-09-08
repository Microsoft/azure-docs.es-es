---
title: Opciones de disponibilidad
description: Obtención de información sobre las características de disponibilidad para ejecutar máquinas virtuales en Azure
author: cynthn
ms.author: cynthn
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 05/10/2019
ms.openlocfilehash: 3826019a19aa744cb03737ed7f0c985f8063a1aa
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815613"
---
# <a name="availability-options-for-virtual-machines-in-azure"></a>Opciones de disponibilidad para las máquinas virtuales de Azure

En este artículo proporciona una visión general de las características de disponibilidad de las máquinas virtuales (VM) de Azure.

## <a name="high-availability"></a>Alta disponibilidad

Por lo general, las cargas de trabajo se distribuyen entre máquinas virtuales distintas para obtener un alto rendimiento y crear redundancia en caso de que una máquina virtual se vea afectada debido a una actualización u otro evento. 

Hay algunas opciones que Azure proporciona para lograr alta disponibilidad. En primer lugar, hablemos sobre las construcciones básicas. 

### <a name="availability-zones"></a>Zonas de disponibilidad

Las [zonas de disponibilidad](../availability-zones/az-overview.md) expanden el nivel de control que tiene para mantener la disponibilidad de las aplicaciones y los datos en las máquinas virtuales. Una zona de disponibilidad es una zona separada físicamente dentro de una región de Azure. Hay tres zonas de disponibilidad por cada región de Azure admitida. 

Cada zona de disponibilidad tiene una fuente de alimentación, una red y un sistema de refrigeración distintos. Si diseña las soluciones para que utilicen máquinas virtuales replicadas en zonas, podrá proteger sus datos y aplicaciones frente a la pérdida de un centro de datos. Aunque una zona esté en peligro, las aplicaciones y los datos replicados estarán disponibles instantáneamente en otra zona. 

![Zonas de disponibilidad](./media/virtual-machines-common-regions-and-availability/three-zones-per-region.png)

Obtenga más información acerca de cómo implementar una máquina virtual [Windows](./windows/create-powershell-availability-zone.md) o [Linux](./linux/create-cli-availability-zone.md) en una zona de disponibilidad.


### <a name="fault-domains"></a>Dominios de error

Un dominio de error es un grupo lógico de hardware subyacente que comparte la fuente de alimentación y el conmutador de red, similar a un bastidor dentro de un centro de datos local. 

### <a name="update-domains"></a>Dominios de actualización

Un dominio de actualización es un grupo lógico de hardware subyacente que puede someterse a mantenimiento o reiniciarse al mismo tiempo. 

Este enfoque garantiza que al menos una instancia de la aplicación sigue ejecutándose cuando se realiza el mantenimiento periódico de la plataforma Azure. Es posible que el orden en que se reinician los dominios de actualización no siga una secuencia durante un mantenimiento, pero se reinician de uno en uno.


## <a name="virtual-machines-scale-sets"></a>Conjuntos de escalado de máquinas virtuales 

Los conjuntos de escalado de máquinas virtuales de Azure permiten crear y administrar un grupo de máquinas virtuales con equilibrio de carga. El número de instancias de máquina virtual puede aumentar o disminuir automáticamente según la demanda, o de acuerdo a una programación definida. Los conjuntos de escalado proporcionan una alta disponibilidad a las aplicaciones y le permiten administrar, configurar y actualizar de forma centralizada muchas máquinas virtuales. Se recomienda la creación de dos o más máquinas virtuales en un conjunto de escalado para proporcionar una aplicación de alta disponibilidad y para cumplir el [99,95 % del Acuerdo de Nivel de Servicio de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/). No hay ningún costo asociado con el conjunto de escalado propiamente dicho, solo se paga por cada instancia de máquina virtual que cree. Cuando una sola máquina virtual usa [discos SSD Premium de Azure](./disks-types.md#premium-ssd), se aplica el Acuerdo de Nivel de Servicio de Azure para los eventos de mantenimiento no planeados. Las máquinas virtuales de un conjunto de escalado se pueden implementar en varios dominios de actualización y dominios de error para maximizar la disponibilidad y la resistencia a las interrupciones debidas a interrupciones del centro de datos y a eventos de mantenimiento planeados o no planeados. Las máquinas virtuales de un conjunto de escalado también se pueden implementar en una sola zona de disponibilidad o en regiones. Las opciones de implementación de zona de disponibilidad pueden diferir según el modo de orquestación.

**Dominios de error y dominios de actualización**

Los conjuntos de escalado de máquinas virtuales simplifican el diseño para lograr alta disponibilidad al alinear los dominios de error y los dominios de actualización. Solo tendrá que definir el número de dominios de error del conjunto de escalado. El número de dominios de error disponibles para los conjuntos de escalado puede variar en función de la región. Consulte [Administración de la disponibilidad de las máquinas virtuales Windows en Azure](./windows/manage-availability.md).


## <a name="availability-sets"></a>Conjuntos de disponibilidad
Un conjunto de disponibilidad es una agrupación lógica de máquinas virtuales dentro de un centro de datos que permite a Azure conocer cómo se crea su aplicación para proporcionar redundancia y disponibilidad. Se recomienda la creación de dos, o más, máquinas virtuales en un conjunto de disponibilidad no solo para proporcionar una aplicación de alta disponibilidad sino también para cumplir el [99,95 % del Acuerdo de Nivel de Servicio de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/). No hay ningún costo asociado con el conjunto de disponibilidad propiamente dicho, solo se paga por cada instancia de máquina virtual que cree. Cuando una sola máquina virtual usa [discos SSD Premium de Azure](./disks-types.md#premium-ssd), se aplica el Acuerdo de Nivel de Servicio de Azure para los eventos de mantenimiento no planeados.

En un conjunto de disponibilidad, las máquinas virtuales se distribuyen automáticamente entre estos dominios de error. Este enfoque limita el impacto de potenciales errores de hardware físico, interrupciones de red o cortes de alimentación eléctrica.

Para las máquinas virtuales que usen [Azure Managed Disks](./faq-for-disks.md), las máquinas virtuales se alinean con los dominios de error de disco administrado cuando se usa un conjunto de disponibilidad administrada. Esta alineación garantiza que todos los discos administrados conectados a una máquina virtual se encuentran en el mismo dominio de error de disco administrado. 

Solo se pueden crear máquinas virtuales con discos administrados en un conjunto de disponibilidad administrada. El número de dominios de error de disco administrado varía según la región: dos o tres dominios de error de disco administrado por región. Para más información sobre estos dominios de error de Managed Disks consulte [máquinas virtuales Linux](./linux/manage-availability.md?#use-managed-disks-for-vms-in-an-availability-set) o [máquinas virtuales Windows](./windows/manage-availability.md?#use-managed-disks-for-vms-in-an-availability-set).

![Conjunto de disponibilidad administrado](./media/virtual-machines-common-manage-availability/md-fd-updated.png)


Las máquinas virtuales dentro de un conjunto de disponibilidad también se distribuyen automáticamente entre los dominios de actualización. 

![Conjuntos de disponibilidad](./media/virtual-machines-common-manage-availability/ud-fd-configuration.png)

## <a name="next-steps"></a>Pasos siguientes
Ya puede empezar a usar estas características de disponibilidad y redundancia para crear un entorno de Azure. Para información sobre los procedimientos recomendados, consulte [Lista de comprobación de disponibilidad](/azure/architecture/checklist/resiliency-per-service).