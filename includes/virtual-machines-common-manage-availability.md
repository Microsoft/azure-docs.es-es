---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 03/27/2018
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: f57c2cacca9bb3e4526ec6261b8aa0ff6c18448a
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546694"
---
## <a name="understand-vm-reboots---maintenance-vs-downtime"></a>Información sobre los reinicios de máquinas virtuales: mantenimiento frente a tiempo de inactividad
Hay tres escenarios que pueden afectar a la máquina virtual de Azure: mantenimiento de hardware no planeado, tiempo de inactividad inesperado y mantenimiento planeado.

* **Un evento de mantenimiento de hardware no planeado** se produce cuando la plataforma Azure predice que en el hardware o en cualquier componente de plataforma asociado a una máquina física está a punto de producirse un error. Cuando la plataforma predice un error, se emitirá un evento de mantenimiento de hardware no planeado para reducir el efecto en las máquinas virtuales hospedadas en ese hardware. Azure usa [migración en vivo](https://docs.microsoft.com/azure/virtual-machines/linux/maintenance-and-updates) tecnología para migrar las máquinas virtuales del hardware con errores a una máquina física en buen estado. La migración en vivo es una operación de conservación de máquinas virtuales que solo detiene la máquina virtual durante un breve período. Se mantienen la memoria, los archivos abiertos y las conexiones de red pero el rendimiento puede verse reducido antes o después del evento. En los casos en los que no se puede usar la migración en vivo, la máquina virtual experimentará tiempos de inactividad inesperados, tal y como se describe a continuación.


* **Un evento de tiempo de inactividad inesperado** se produce cuando el hardware o la infraestructura física de la máquina virtual produce un error de forma imprevista. Entre estos podemos encontrar errores de la red local, errores de los discos locales u otras errores a nivel de bastidor. Cuando se detecta, la plataforma Azure migra (recupera) automáticamente la máquina virtual a una máquina física en buen estado en el mismo centro de datos. Durante el procedimiento de recuperación, las máquinas virtuales experimentan tiempos de inactividad (reinicio) y, en algunos casos, pérdidas de la unidad temporal. El sistema operativo y los discos de datos asociados siempre se conservan. 

  Las máquinas virtuales también pueden experimentar tiempos de inactividad en el improbable caso de que una interrupción o desastre afecte a todo un centro de datos, o incluso a toda una región. Para estos casos, Azure proporciona opciones de protección que incluyen [zonas de disponibilidad](../articles/availability-zones/az-overview.md) y [regiones emparejadas](../articles/best-practices-availability-paired-regions.md#what-are-paired-regions).

* **Eventos de mantenimiento planeado** son actualizaciones periódicas realizadas por Microsoft en la plataforma Azure subyacente para mejorar en general la fiabilidad, el rendimiento y la seguridad de la infraestructura de la plataforma sobre las que se funcionan sus máquinas virtuales. La mayoría de estas actualizaciones se realizan sin que Virtual Machines ni Cloud Services resulten afectados (consulte [Mantenimiento de conservación de máquinas virtuales](https://docs.microsoft.com/azure/virtual-machines/windows/preserving-maintenance)). Aunque la plataforma Azure intente utilizar el mantenimiento de conservación de máquinas virtuales en todas las ocasiones posibles, existen algunos casos poco frecuentes en los que estas actualizaciones requieren un reinicio de la máquina virtual para aplicar las actualizaciones necesarias a la infraestructura subyacente. En este caso, puede realizar un mantenimiento planeado de Azure con la operación de reimplementación de mantenimiento mediante el inicio del mantenimiento para sus máquinas virtuales en el período de tiempo adecuado. Para más información, consulte [Mantenimiento planeado de máquinas virtuales](https://docs.microsoft.com/azure/virtual-machines/windows/planned-maintenance/).


Para reducir el impacto del tiempo de parada debido a uno o más de estos eventos, recomendamos las siguientes mejores prácticas de alta disponibilidad para las máquinas virtuales:

* [Configure varias máquinas virtuales en un conjunto de disponibilidad para la redundancia]
* [Uso de Managed Disks para las máquinas virtuales de un conjunto de disponibilidad]
* [Uso de eventos programados para responder de manera proactiva a eventos que afectan a la máquina virtual](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-scheduled-events)
* [Configure cada nivel de aplicación en conjuntos separados de disponibilidad]
* [Combinación de un equilibrador de carga con conjuntos de disponibilidad]
* [Uso de zonas de disponibilidad para protegerse frente a errores en el nivel de centro de datos]

## <a name="configure-multiple-virtual-machines-in-an-availability-set-for-redundancy"></a>Configure varias máquinas virtuales en un conjunto de disponibilidad para la redundancia
Para proporcionar redundancia a la aplicación, recomendamos que agrupe dos máquinas virtuales o más en un conjunto de disponibilidad. Esta configuración en un centro de datos garantiza que, durante un evento de mantenimiento planeado o no planeado, hay al menos una máquina virtual disponible y cumple el 99,95 % del Acuerdo de Nivel de Servicio de Azure. Para obtener más información, consulte [Acuerdo de Nivel de Servicio para máquinas virtuales](https://azure.microsoft.com/support/legal/sla/virtual-machines/).

> [!IMPORTANT]
> Evite dejar una máquina virtual de instancia única sola en un conjunto de disponibilidad. En esta configuración, las máquinas virtuales no reciben la garantía del Acuerdo de Nivel de Servicio y sufrirán un tiempo de inactividad durante los eventos de mantenimiento planeado de Azure, excepto cuando solo una máquina virtual use [discos SSD Premium de Azure](../articles/virtual-machines/windows/disks-types.md#premium-ssd). El Acuerdo de Nivel de Servicio de Azure sí se aplica a las máquinas virtuales únicas que usen discos SSD Premium.

La plataforma Azure subyacente asigna a cada máquina virtual del conjunto de disponibilidad un **dominio de actualización** y un **dominio de error**. Para un conjunto de disponibilidad dado, se asignan de forma predeterminada cinco dominios de actualización que el usuario no puede configurar (las implementaciones de Resource Manager pueden aumentarse para proporcionar un máximo de veinte dominios de actualización), con el fin de indicar grupos de máquinas virtuales y el hardware físico subyacente que se pueden reiniciar simultáneamente. Cuando se configuran más de cinco máquinas virtuales en un único conjunto de disponibilidad, la sexta máquina virtual se coloca en el mismo dominio de actualización que la primera, la séptima en el mismo que la segunda, y así sucesivamente. Es posible que el orden en que se reinician los dominios de actualización no siga una secuencia durante un mantenimiento planeado, pero se reinician de uno en uno. Un dominio de actualización reiniciado tiene 30 minutos para recuperar antes de que el mantenimiento se inicie en un dominio de actualización diferente.

Los dominios de error definen un grupo de máquinas virtuales que comparten un origen de alimentación y un interruptor de red comunes. De manera predeterminada, las máquinas virtuales configuradas en un conjunto de disponibilidad se separan en hasta 3 dominios de error en las implementaciones con Resource Manager (dos dominios de error en las implementaciones con el método clásico). Aunque colocar las máquinas virtuales en un conjunto de disponibilidad no protege su aplicación contra errores del sistema operativo ni específicos de aplicaciones, limita el impacto de posibles errores de hardware físico, interrupciones de red o cortes de alimentación.

<!--Image reference-->
   ![Dibujo conceptual de la configuración del dominio de error y dominio de actualización](./media/virtual-machines-common-manage-availability/ud-fd-configuration.png)

## <a name="use-managed-disks-for-vms-in-an-availability-set"></a>Uso de Managed Disks para las máquinas virtuales de un conjunto de disponibilidad
Si actualmente está usando máquinas virtuales con discos no administrados, es muy recomendable [convertir las máquinas virtuales del conjunto de disponibilidad para que usen Managed Disks](../articles/virtual-machines/windows/convert-unmanaged-to-managed-disks.md).

[Managed Disks](../articles/virtual-machines/windows/managed-disks-overview.md) proporciona una mayor confiabilidad para los conjuntos de disponibilidad, ya que garantiza que los discos de las máquinas virtuales de un conjunto de disponibilidad estén suficientemente aislados entre sí para evitar puntos únicos de error. Para ello, coloca automáticamente los discos en dominios de error de almacenamiento diferentes (clústeres de almacenamiento) y los alinea con el dominio de error de la máquina virtual. Si se produce un error en un dominio de error de almacenamiento debido a un error de hardware o software, solo presentará errores la instancia de máquina virtual que tenga discos en el dominio de error de almacenamiento.
![FDS de discos administrados](./media/virtual-machines-common-manage-availability/md-fd-updated.png)

> [!IMPORTANT]
> El número de dominios de error para conjuntos de disponibilidad administrados varía según la región, entre dos y tres por región. En la tabla siguiente se muestra el número por región.

[!INCLUDE [managed-disks-common-fault-domain-region-list](managed-disks-common-fault-domain-region-list.md)]

Si tiene previsto usar máquinas virtuales con discos no administrados, siga los procedimientos recomendados que aparecen a continuación para las cuentas de almacenamiento donde se almacenan los discos duros virtuales (VHD) de las máquinas virtuales como [blobs en páginas](https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs#about-page-blobs).

1. **Mantenga todos los discos (sistema operativo y datos) asociados a una máquina virtual en la misma cuenta de almacenamiento.**
2. **Revise los [límites](../articles/storage/common/storage-scalability-targets.md) en el número de discos no administrados de una cuenta de almacenamiento** antes de agregar más discos duros virtuales a esta
3. **Utilice una cuenta de almacenamiento independiente para cada máquina virtual de un conjunto de disponibilidad.** No comparta las cuentas de almacenamiento con varias máquinas virtuales del mismo conjunto de disponibilidad. Las máquinas virtuales de distintos conjuntos de disponibilidad pueden compartir las cuentas de almacenamiento si se siguen los procedimientos recomendados anteriores. ![Dominios de error de discos no administrados](./media/virtual-machines-common-manage-availability/umd-updated.png)

## <a name="use-scheduled-events-to-proactively-respond-to-vm-impacting-events"></a>Uso de eventos programados para responder de manera proactiva a eventos que afectan a la máquina virtual

Cuando se suscribe a [eventos programados](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-scheduled-events), se le notifica a la máquina virtual sobre próximos eventos de mantenimiento que pueden afectar a su funcionamiento. Si los eventos programados están habilitados, se le da a la máquina virtual una cantidad mínima de tiempo antes de que se lleve a cabo la actividad de mantenimiento. Por ejemplo, las actualizaciones del sistema operativo del host que pueden afectar a la máquina virtual se ponen en cola como eventos en los que se especifica el impacto, así como la hora en que se efectuará el mantenimiento si no se realiza ninguna acción. Los eventos de programación también se ponen en cola cuando Azure detecta un error inminente del hardware que podría afectar a la máquina virtual, lo cual permite al usuario decidir cuándo se debe realizar la recuperación. Los clientes pueden usar el evento para realizar determinadas tareas antes del mantenimiento, como guardar el estado, conmutar por error a la base de datos secundaria, etc. Después de completar la lógica para controlar correctamente el evento de mantenimiento, puede aprobar el evento programado pendiente para permitir que la plataforma continúe con el mantenimiento.

## <a name="configure-each-application-tier-into-separate-availability-sets"></a>Configure cada nivel de aplicación en conjuntos separados de disponibilidad
Si las máquinas virtuales son casi idénticas y tienen la misma función en su aplicación, recomendamos que configure un conjunto de disponibilidad para cada nivel de la aplicación.  Si coloca dos niveles diferentes en el mismo conjunto de disponibilidad, todas las máquinas virtuales en un mismo nivel de aplicación se podrían reiniciar simultáneamente. Al configurar al menos dos máquinas virtuales en un conjunto de disponibilidad para cada nivel, se garantiza que al menos haya disponible una en cada nivel.

Por ejemplo, podría poner todas las máquinas virtuales en el front-end de la aplicación que ejecuta IIS, Apache, Nginx, etc. en un solo conjunto de disponibilidad. Asegúrese de que solo las máquinas virtuales de front-end se colocan en el mismo conjunto de disponibilidad. De la misma manera, asegúrese de que solo las máquinas virtuales de niveles de datos se colocan en su propio conjunto de disponibilidad, por ejemplo, las máquinas virtuales replicadas de SQL Server o las de MySQL.

<!--Image reference-->
   ![Niveles de aplicación](./media/virtual-machines-common-manage-availability/application-tiers.png)

## <a name="combine-a-load-balancer-with-availability-sets"></a>Combinación de un equilibrador de carga con conjuntos de disponibilidad
Combine [Azure Load Balancer](../articles/load-balancer/load-balancer-overview.md) con un conjunto de disponibilidad para aprovechar al máximo la resistencia de la aplicación. El equilibrador de carga de Azure distribuye el tráfico entre varias máquinas virtuales. El equilibrador de carga de Azure está incluido en nuestras máquinas virtuales de niveles estándar. No todos los niveles de las máquinas virtuales incluyen Azure Load Balancer. Para obtener más información sobre el equilibrio de carga en máquinas virtuales, consulte [Equilibrio de carga de máquinas virtuales](../articles/virtual-machines/virtual-machines-linux-load-balance.md).

Si el equilibrador de carga no está configurado para equilibrar el tráfico entre varias máquinas virtuales, cualquier evento de mantenimiento planeado afectará a la única máquina virtual dedicada al tráfico, lo que provocará una interrupción en el nivel de la aplicación. Si se colocan varias máquinas virtuales del mismo nivel en el mismo equilibrador de carga y conjunto de disponibilidad, se permitirá tener un tráfico continuamente disponible asistido por, al menos, una instancia.

## <a name="use-availability-zones-to-protect-from-datacenter-level-failures"></a>Uso de zonas de disponibilidad para protegerse frente a errores en el nivel de centro de datos

Las [zonas de disponibilidad](../articles/availability-zones/az-overview.md) son una alternativa a los conjuntos de disponibilidad que permite ampliar el nivel de control que tiene para mantener la disponibilidad de las aplicaciones y los datos en las máquinas virtuales. Una zona de disponibilidad es una zona separada físicamente dentro de una región de Azure. Hay tres zonas de disponibilidad por cada región de Azure admitida. Cada zona de disponibilidad tiene una fuente de alimentación, una red y un dispositivo de enfriamiento independientes y está separada lógicamente de otras zonas de disponibilidad dentro de la región de Azure. Si diseña las soluciones para que utilicen máquinas virtuales replicadas en zonas, podrá proteger sus datos y aplicaciones frente a la pérdida de un centro de datos. Aunque una zona esté en peligro, las aplicaciones y los datos replicados estarán disponibles instantáneamente en otra zona. 

![Zonas de disponibilidad](./media/virtual-machines-common-regions-and-availability/three-zones-per-region.png)

Obtenga más información acerca de cómo implementar una máquina virtual [Windows](../articles/virtual-machines/windows/create-powershell-availability-zone.md) o [Linux](../articles/virtual-machines/linux/create-cli-availability-zone.md) en una zona de disponibilidad.


<!-- Link references -->
[Configure varias máquinas virtuales en un conjunto de disponibilidad para la redundancia]: #configure-multiple-virtual-machines-in-an-availability-set-for-redundancy
[Configure cada nivel de aplicación en conjuntos separados de disponibilidad]: #configure-each-application-tier-into-separate-availability-sets
[Combinación de un equilibrador de carga con conjuntos de disponibilidad]: #combine-a-load-balancer-with-availability-sets
[Avoid single instance virtual machines in availability sets]: #avoid-single-instance-virtual-machines-in-availability-sets
[Uso de Managed Disks para las máquinas virtuales de un conjunto de disponibilidad]: #use-managed-disks-for-vms-in-an-availability-set
[Uso de zonas de disponibilidad para protegerse frente a errores en el nivel de centro de datos]: #use-availability-zones-to-protect-from-datacenter-level-failures
