---
title: Traslado de máquinas virtuales de Azure a otra región con Azure Site Recovery
description: Use Azure Site Recovery para mover máquinas virtuales de Azure de una región de Azure a otra.
author: rajani-janaki-ram
ms.service: site-recovery
ms.topic: tutorial
ms.date: 01/28/2019
ms.author: rajanaki
ms.custom: MVC
ms.openlocfilehash: 0c7efc94bcde18e7b6ff43726602fa87641f3e76
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86130620"
---
# <a name="moving-azure-vms-to-another-azure-region"></a>Traslado de máquinas virtuales de Azure a otra región de Azure

En este artículo se proporciona información general sobre los motivos y los pasos necesarios para mover máquinas virtuales de Azure a otra región de Azure mediante [Azure Site Recovery](site-recovery-overview.md). 


## <a name="reasons-to-move-azure-vms"></a>Razones para trasladar máquinas virtuales de Azure

Posibles motivos para trasladas las máquinas virtuales:

- Que ya se haya implementado en una región y se haya agregado compatibilidad con una nueva, que está más cerca de los usuarios finales de la aplicación o el servicio. En este escenario, quizá quiera trasladas las máquinas virtuales tal cual están a la nueva región para reducir la latencia. Use el mismo enfoque si desea consolidar las suscripciones o si existen reglas de gobernanza o de organización que requieran el traslado.
- Que la máquina virtual se implementara como de instancia única o como parte de un conjunto de disponibilidad. Si desea aumentar la disponibilidad de los Acuerdos de Nivel de Servicio, puede trasladar las máquinas virtuales a una zona de disponibilidad.

## <a name="steps-to-move-azure-vms"></a>Pasos para trasladar máquinas virtuales de Azure

El traslado de máquinas virtuales implica los pasos siguientes:

1. Compruebe los requisitos previos.
2. Prepare las máquinas virtuales de origen.
3. Prepare la región de destino.
4. Copie los datos en la región de destino. Utilice la tecnología de replicación de Azure Site Recovery para copiar los datos de la máquina virtual de origen en la región de destino.
5. Pruebe la configuración. Una vez finalizada la replicación, pruebe la configuración mediante la realización de una conmutación por error de prueba a una red que no sea de producción.
6. Realice el traslado.
7. Descarte los recursos en la región de origen.

> [!NOTE]
> En las secciones siguientes se proporcionan detalles acerca de estos pasos.
> [!IMPORTANT]
> Actualmente Azure Site Recovery permite trasladar máquinas virtuales de una región a otra, pero no admite el traslado dentro de una misma región.

## <a name="typical-architectures-for-a-multi-tier-deployment"></a>Arquitecturas típicas para una implementación de varios niveles

En esta sección se describen las arquitecturas de implementación más comunes para una aplicación de varios niveles en Azure. El ejemplo es de una aplicación de tres niveles con una IP pública. Todos los niveles (web, aplicación y base de datos) tienen dos máquinas virtuales y están conectados por un equilibrador de carga de Azure a los demás niveles. El nivel de base de datos tiene replicación SQL Server Always On entre las máquinas virtuales para alta disponibilidad.

* **Máquinas virtuales de instancia única implementadas en varios niveles**: en los distintos niveles, las máquinas virtuales se configuran como de instancia única y se conectan mediante equilibradores de carga a los demás niveles. Esta configuración es la que se adopta más fácilmente.

     ![Implementación de máquinas virtuales de instancia única en distintos niveles](media/move-vm-overview/regular-deployment.png)

* **Máquinas virtuales en todos los niveles implementadas en conjuntos de disponibilidad**: Las máquinas virtuales de un nivel se configuran en un conjunto de disponibilidad. Los [conjuntos de disponibilidad](../virtual-machines/windows/tutorial-availability-sets.md) garantizan que las máquinas virtuales implementadas en Azure se distribuyan entre varios nodos de hardware aislados en un clúster. Esto garantiza que, si se produce un error de hardware o software en Azure, solo un subconjunto de las máquinas virtuales se vea afectado y que la solución global siga disponible y en funcionamiento.

     ![Implementación de máquinas virtuales en conjuntos de disponibilidad](media/move-vm-overview/avset.png)

* **Máquinas virtuales en todos los niveles implementadas en zonas de disponibilidad**: Las máquinas virtuales de los distintos niveles se configuran en [zonas de disponibilidad](../availability-zones/az-overview.md). Una zona de disponibilidad de una región de Azure es una combinación de un dominio de error y un dominio de actualización. Por ejemplo, si crea tres o más máquinas virtuales en tres zonas de una región de Azure, las máquinas virtuales se distribuyen eficazmente en tres dominios de error y tres dominios de actualización. La plataforma Azure reconoce esta distribución entre dominios de actualización para asegurarse de que las máquinas virtuales de distintas zonas no se actualizan al mismo tiempo.

     ![Implementación en una zona de disponibilidad](media/move-vm-overview/zone.png)

## <a name="move-vms-as-is-to-a-target-region"></a>Traslado de máquinas virtuales tal cual están a una región de destino

En función de las [arquitecturas](#typical-architectures-for-a-multi-tier-deployment) anteriormente mencionadas, aquí se muestra cómo se verán las implementaciones una vez realizado el traslado tal y como está a la región de destino.

* **Máquinas virtuales de instancia única implementadas en varios niveles**

     ![Implementación de máquinas virtuales de instancia única en distintos niveles](media/move-vm-overview/single-zone.png)

* **Máquinas virtuales en todos los niveles implementadas en conjuntos de disponibilidad**

     ![Conjuntos de disponibilidad en distintas regiones](media/move-vm-overview/crossregionaset.png)

* **Máquinas virtuales en todos los niveles implementadas en zonas de disponibilidad**

     ![Implementación de máquinas virtuales en zonas de disponibilidad](media/move-vm-overview/azonecross.png)

## <a name="move-vms-to-increase-availability"></a>Traslado de máquinas virtuales para aumentar la disponibilidad

* **Máquinas virtuales de instancia única implementadas en varios niveles**

     ![Implementación de máquinas virtuales de instancia única en distintos niveles](media/move-vm-overview/single-zone.png)

* **Máquinas virtuales en todos los niveles implementadas en conjuntos de disponibilidad**: Puede configurar las máquinas virtuales en un conjunto de disponibilidad en zonas de disponibilidad independientes al habilitar la replicación de la máquina virtual mediante Azure Site Recovery. El Acuerdo de Nivel de Servicio de disponibilidad será del 99,99 % una vez finalizada la operación de traslado.

     ![Implementación de máquinas virtuales en conjuntos de disponibilidad y zonas de disponibilidad](media/move-vm-overview/aset-azone.png)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> 
> * [Traslado de máquinas virtuales de Azure a otra región](azure-to-azure-tutorial-migrate.md)
> 
> * [Traslado de máquinas virtuales de Azure a zonas de disponibilidad](move-azure-vms-avset-azone.md)

