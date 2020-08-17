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
ms.openlocfilehash: f1517fd577c5e6bd7341e5dde0204456524ba976
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87545203"
---
## <a name="understand-vm-reboots---maintenance-vs-downtime"></a>Información sobre los reinicios de máquinas virtuales: mantenimiento frente a tiempo de inactividad
Hay tres escenarios que pueden afectar a la máquina virtual de Azure: mantenimiento de hardware no planeado, tiempo de inactividad inesperado y mantenimiento planeado.

* **Un evento de mantenimiento de hardware no planeado** se produce cuando la plataforma Azure predice que en el hardware o en cualquier componente de plataforma asociado a una máquina física está a punto de producirse un error. Cuando la plataforma predice un error, se emitirá un evento de mantenimiento de hardware no planeado para reducir el efecto en las máquinas virtuales hospedadas en ese hardware. Azure usa la tecnología [Migración en vivo](https://docs.microsoft.com/azure/virtual-machines/linux/maintenance-and-updates) para migrar las máquinas virtuales del hardware con errores a un equipo físico en buen estado. La migración en vivo es una operación de conservación de máquinas virtuales que solo detiene la máquina virtual durante un breve período. Se mantienen la memoria, los archivos abiertos y las conexiones de red pero el rendimiento puede verse reducido antes o después del evento. En los casos en los que no se puede usar la migración en vivo, la máquina virtual experimentará tiempos de inactividad inesperados, tal y como se describe a continuación.


* **Un evento de tiempo de inactividad inesperado** se produce cuando el hardware o la infraestructura física de la máquina virtual produce un error de forma imprevista. Entre estos podemos encontrar errores de la red local, errores de los discos locales u otras errores a nivel de bastidor. Cuando se detecta, la plataforma Azure migra (recupera) automáticamente la máquina virtual a una máquina física en buen estado en el mismo centro de datos. Durante el procedimiento de recuperación, las máquinas virtuales experimentan tiempos de inactividad (reinicio) y, en algunos casos, pérdidas de la unidad temporal. El sistema operativo y los discos de datos asociados siempre se conservan.

  Las máquinas virtuales también pueden experimentar tiempos de inactividad en el improbable caso de que una interrupción o desastre afecte a todo un centro de datos, o incluso a toda una región. Para estos casos, Azure proporciona opciones de protección que incluyen [zonas de disponibilidad](../articles/availability-zones/az-overview.md) y [regiones emparejadas](../articles/best-practices-availability-paired-regions.md#what-are-paired-regions).

* **Eventos de mantenimiento planeado** son actualizaciones periódicas realizadas por Microsoft en la plataforma Azure subyacente para mejorar en general la fiabilidad, el rendimiento y la seguridad de la infraestructura de la plataforma sobre las que se funcionan sus máquinas virtuales. La mayoría de estas actualizaciones se realizan sin que Virtual Machines ni Cloud Services resulten afectados (consulte [Mantenimiento de conservación de máquinas virtuales](https://docs.microsoft.com/azure/virtual-machines/windows/preserving-maintenance)). Aunque la plataforma Azure intente utilizar el mantenimiento de conservación de máquinas virtuales en todas las ocasiones posibles, existen algunos casos poco frecuentes en los que estas actualizaciones requieren un reinicio de la máquina virtual para aplicar las actualizaciones necesarias a la infraestructura subyacente. En este caso, puede realizar un mantenimiento planeado de Azure con la operación de reimplementación de mantenimiento mediante el inicio del mantenimiento para sus máquinas virtuales en el período de tiempo adecuado. Para más información, consulte [Mantenimiento planeado de máquinas virtuales](https://docs.microsoft.com/azure/virtual-machines/windows/planned-maintenance/).


Para reducir el impacto del tiempo de parada debido a uno o más de estos eventos, recomendamos las siguientes mejores prácticas de alta disponibilidad para las máquinas virtuales:

* [Configure varias máquinas virtuales en un conjunto de disponibilidad para la redundancia]
* [Uso de Managed Disks para las máquinas virtuales de un conjunto de disponibilidad]
* [Uso de eventos programados para responder de manera proactiva a eventos que afectan a la máquina virtual](../articles/virtual-machines/linux/scheduled-events.md)
* [Configuración de cada nivel de aplicación en conjuntos separados de disponibilidad]
* [Combinación de un equilibrador de carga con conjuntos de disponibilidad]
* [Uso de zonas de disponibilidad para protegerse frente a errores en el nivel de centro de datos]

## <a name="use-availability-zones-to-protect-from-datacenter-level-failures"></a>Uso de zonas de disponibilidad para protegerse frente a errores en el nivel de centro de datos

Las [zonas de disponibilidad](../articles/availability-zones/az-overview.md) expanden el nivel de control que tiene para mantener la disponibilidad de las aplicaciones y los datos en las máquinas virtuales. Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay un mínimo de tres zonas independientes en todas las regiones habilitadas. La separación física de las zonas de disponibilidad dentro de una región protege las aplicaciones y los datos frente a los errores del centro de datos. Los servicios con redundancia de zona replican las aplicaciones y los datos entre zonas de disponibilidad para protegerlos frente a puntos de error únicos.

Una zona de disponibilidad de una región de Azure es una combinación de un **dominio de error** y un **dominio de actualización**. Por ejemplo, si crea tres o más máquinas virtuales en tres zonas de una región de Azure, las máquinas virtuales se distribuyen eficazmente en tres dominios de error y tres dominios de actualización. La plataforma Azure reconoce esta distribución entre dominios de actualización para asegurarse de que las máquinas virtuales de distintas zonas no se actualizan al mismo tiempo.

Con las zonas de disponibilidad, Azure ofrece el mejor Acuerdo de Nivel de Servicio del sector de tiempo de actividad de máquina virtual, con un 99,99 %. Si diseña las soluciones para que usen máquinas virtuales replicadas en zonas, puede proteger los datos y las aplicaciones frente a la pérdida de un centro de datos. Aunque una zona esté en peligro, las aplicaciones y los datos replicados estarán disponibles instantáneamente en otra zona.

![Zonas de disponibilidad](./media/virtual-machines-common-manage-availability/three-zones-per-region.png)

Obtenga más información acerca de cómo implementar una máquina virtual [Windows](../articles/virtual-machines/windows/create-powershell-availability-zone.md) o [Linux](../articles/virtual-machines/linux/create-cli-availability-zone.md) en una zona de disponibilidad.

## <a name="configure-multiple-virtual-machines-in-an-availability-set-for-redundancy"></a>Configure varias máquinas virtuales en un conjunto de disponibilidad para la redundancia
Los conjuntos de disponibilidad son otra configuración de centro de datos para proporcionar redundancia y disponibilidad de máquina virtual. Esta configuración en un centro de datos garantiza que, durante un evento de mantenimiento planeado o no planeado, hay al menos una máquina virtual disponible y cumple el 99,95 % del Acuerdo de Nivel de Servicio de Azure. Para obtener más información, consulte [Acuerdo de Nivel de Servicio para máquinas virtuales](https://azure.microsoft.com/support/legal/sla/virtual-machines/).

> [!IMPORTANT]
> Una máquina virtual de una sola instancia en un conjunto de disponibilidad por sí misma debe usar SSD Premium o un disco Ultra para todos los discos del sistema operativo y los discos de datos para poder optar al Acuerdo de Nivel de Servicio para la conectividad de máquina virtual de al menos el 99,9 %. 
> 
> Una máquina virtual de instancia única con un SSD estándar tendrá un Acuerdo de Nivel de Servicio de al menos el 99,5 %, mientras que una máquina virtual de instancia única con un HDD estándar tendrá un Acuerdo de Nivel de Servicio de al menos 95 %.  Consulte el [Acuerdo de Nivel de Servicio para Virtual Machines](https://azure.microsoft.com/support/legal/sla/virtual-machines/)

La plataforma Azure subyacente asigna a cada máquina virtual del conjunto de disponibilidad un **dominio de actualización** y un **dominio de error**. Para un conjunto de disponibilidad dado, se asignan de forma predeterminada cinco dominios de actualización que el usuario no puede configurar (las implementaciones de Resource Manager pueden aumentarse para proporcionar un máximo de veinte dominios de actualización), con el fin de indicar grupos de máquinas virtuales y el hardware físico subyacente que se pueden reiniciar simultáneamente. Cuando se configuran más de cinco máquinas virtuales en un único conjunto de disponibilidad, la sexta máquina virtual se coloca en el mismo dominio de actualización que la primera, la séptima en el mismo que la segunda, y así sucesivamente. Es posible que el orden en que se reinician los dominios de actualización no siga una secuencia durante un mantenimiento planeado, pero se reinician de uno en uno. Un dominio de actualización reiniciado tiene 30 minutos para recuperar antes de que el mantenimiento se inicie en un dominio de actualización diferente.

Los dominios de error definen un grupo de máquinas virtuales que comparten un origen de alimentación y un interruptor de red comunes. De manera predeterminada, las máquinas virtuales configuradas en un conjunto de disponibilidad se separan en hasta 3 dominios de error en las implementaciones con Resource Manager (dos dominios de error en las implementaciones con el método clásico). Aunque colocar las máquinas virtuales en un conjunto de disponibilidad no protege su aplicación contra errores del sistema operativo ni específicos de aplicaciones, limita el impacto de posibles errores de hardware físico, interrupciones de red o cortes de alimentación.

<!--Image reference-->
   ![Dibujo conceptual de la configuración del dominio de actualización y el dominio de error](./media/virtual-machines-common-manage-availability/ud-fd-configuration.png)

## <a name="use-managed-disks-for-vms-in-an-availability-set"></a>Uso de Managed Disks para las máquinas virtuales de un conjunto de disponibilidad
Si actualmente está usando máquinas virtuales con discos no administrados, es muy recomendable [convertir las máquinas virtuales del conjunto de disponibilidad para que usen Managed Disks](../articles/virtual-machines/windows/convert-unmanaged-to-managed-disks.md).

[Managed Disks](../articles/virtual-machines/windows/managed-disks-overview.md) proporciona una mayor confiabilidad para los conjuntos de disponibilidad, ya que garantiza que los discos de las máquinas virtuales de un conjunto de disponibilidad estén suficientemente aislados entre sí para evitar puntos únicos de error. Para ello, coloca automáticamente los discos en dominios de error de almacenamiento diferentes (clústeres de almacenamiento) y los alinea con el dominio de error de la máquina virtual. Si se produce un error en un dominio de error de almacenamiento debido a un error de hardware o software, solo presentará errores la instancia de máquina virtual que tenga discos en el dominio de error de almacenamiento.
![FDS de discos administrados](./media/virtual-machines-common-manage-availability/md-fd-updated.png)

> [!IMPORTANT]
> El número de dominios de error para conjuntos de disponibilidad administrados varía según la región, entre dos y tres por región. Puede ver el dominio de error de cada región ejecutando los scripts siguientes.

```azurepowershell-interactive
Get-AzComputeResourceSku | where{$_.ResourceType -eq 'availabilitySets' -and $_.Name -eq 'Aligned'}
```

```azurecli-interactive 
az vm list-skus --resource-type availabilitySets --query '[?name==`Aligned`].{Location:locationInfo[0].location, MaximumFaultDomainCount:capabilities[0].value}' -o Table
```

> [!NOTE]
> En determinadas circunstancias, dos máquinas virtuales en el mismo conjunto de disponibilidad podrían compartir el mismo dominio de error. Para confirmarlo, vaya al conjunto de disponibilidad y compruebe la columna **Dominio de error**.
> Esto puede deberse a la siguiente secuencia mientras se implementan las máquinas virtuales:
> - Implementar la primera máquina virtual
> - Detener o desasignar la primera máquina virtual
> - Implementar la segunda máquina virtual en estas circunstancias, el disco del sistema operativo de la segunda máquina virtual puede crearse en el mismo dominio de error que la primera máquina virtual, por lo que la segunda máquina virtual también aterrizará en el mismo FaultDomain. 
> Para evitar este problema, se recomienda no detener ni desasignar las máquinas virtuales entre implementaciones.

Si tiene previsto usar máquinas virtuales con discos no administrados, siga los procedimientos recomendados que aparecen a continuación para las cuentas de almacenamiento donde se almacenan los discos duros virtuales (VHD) de las máquinas virtuales como [blobs en páginas](https://docs.microsoft.com/rest/api/storageservices/Understanding-Block-Blobs--Append-Blobs--and-Page-Blobs#about-page-blobs).

1. **Mantenga todos los discos (sistema operativo y datos) asociados a una máquina virtual en la misma cuenta de almacenamiento.**
2. **Revise los [límites](../articles/storage/blobs/scalability-targets-premium-page-blobs.md) en el número de discos no administrados de una cuenta de Azure Storage** antes de agregar más discos duros virtuales a esta
3. **Utilice una cuenta de almacenamiento independiente para cada máquina virtual de un conjunto de disponibilidad.** No comparta las cuentas de almacenamiento con varias máquinas virtuales del mismo conjunto de disponibilidad. Las máquinas virtuales de distintos conjuntos de disponibilidad pueden compartir las cuentas de almacenamiento si se siguen los procedimientos recomendados anteriores. ![Dominios de error de discos no administrados](./media/virtual-machines-common-manage-availability/umd-updated.png)

## <a name="use-scheduled-events-to-proactively-respond-to-vm-impacting-events"></a>Uso de eventos programados para responder de manera proactiva a eventos que afectan a la máquina virtual

Cuando se suscribe a [eventos programados](../articles/virtual-machines/linux/scheduled-events.md), se le notifica a la máquina virtual sobre próximos eventos de mantenimiento que pueden afectar a su funcionamiento. Si los eventos programados están habilitados, se le da a la máquina virtual una cantidad mínima de tiempo antes de que se lleve a cabo la actividad de mantenimiento. Por ejemplo, las actualizaciones del sistema operativo del host que pueden afectar a la máquina virtual se ponen en cola como eventos en los que se especifica el impacto, así como la hora en que se efectuará el mantenimiento si no se realiza ninguna acción. Los eventos de programación también se ponen en cola cuando Azure detecta un error inminente del hardware que podría afectar a la máquina virtual, lo cual permite al usuario decidir cuándo se debe realizar la recuperación. Los clientes pueden usar el evento para realizar determinadas tareas antes del mantenimiento, como guardar el estado, conmutar por error a la base de datos secundaria, etc. Después de completar la lógica para controlar correctamente el evento de mantenimiento, puede aprobar el evento programado pendiente para permitir que la plataforma continúe con el mantenimiento.


## <a name="combine-a-load-balancer-with-availability-zones-or-sets"></a>Combinación de un equilibrador de carga con conjuntos o zonas de disponibilidad
Combine [Azure Load Balancer](../articles/load-balancer/load-balancer-overview.md) con un conjunto o zona de disponibilidad para aprovechar al máximo la resistencia de la aplicación. El equilibrador de carga de Azure distribuye el tráfico entre varias máquinas virtuales. El equilibrador de carga de Azure está incluido en nuestras máquinas virtuales de niveles estándar. No todos los niveles de las máquinas virtuales incluyen Azure Load Balancer. Para obtener más información sobre el equilibrio de carga en máquinas virtuales, consulte [Equilibrio de carga de máquinas virtuales](../articles/virtual-machines/linux/tutorial-load-balancer.md).

Si el equilibrador de carga no está configurado para equilibrar el tráfico entre varias máquinas virtuales, cualquier evento de mantenimiento planeado afectará a la única máquina virtual dedicada al tráfico, lo que provocará una interrupción en el nivel de la aplicación. Si se colocan varias máquinas virtuales del mismo nivel en el mismo equilibrador de carga y conjunto de disponibilidad, se permitirá tener un tráfico continuamente disponible asistido por, al menos, una instancia.

Para obtener un tutorial sobre cómo equilibrar la carga entre las zonas de disponibilidad, vea [Equilibrio de carga de máquinas virtuales en todas las zonas de disponibilidad con la CLI de Azure](../articles/load-balancer/load-balancer-standard-public-zone-redundant-cli.md).


<!-- Link references -->
[Configure varias máquinas virtuales en un conjunto de disponibilidad para la redundancia]: #configure-multiple-virtual-machines-in-an-availability-set-for-redundancy
[Combinación de un equilibrador de carga con conjuntos de disponibilidad]: #combine-a-load-balancer-with-availability-zones-or-sets
[Avoid single instance virtual machines in availability sets]: #avoid-single-instance-virtual-machines-in-availability-sets
[Uso de Managed Disks para las máquinas virtuales de un conjunto de disponibilidad]: #use-managed-disks-for-vms-in-an-availability-set
[Uso de zonas de disponibilidad para protegerse frente a errores en el nivel de centro de datos]: #use-availability-zones-to-protect-from-datacenter-level-failures
