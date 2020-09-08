---
title: 'Optimización del rendimiento en máquinas virtuales de la serie Lsv2 de Azure: Almacenamiento'
description: Aprenda a optimizar el rendimiento de la solución en las máquinas virtuales de la serie Lsv2.
author: sasha-melamed
ms.service: virtual-machines-windows
ms.subservice: sizes
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/17/2019
ms.author: joelpell
ms.openlocfilehash: 82554982cd55b6c5fb2b96b2752b00401cb896d8
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88653637"
---
# <a name="optimize-performance-on-the-lsv2-series-windows-virtual-machines"></a>Optimización del rendimiento en las máquinas virtuales Windows de la serie Lsv2

Las máquinas virtuales de la serie Lsv2 admiten una variedad de cargas de trabajo que necesitan una elevada cantidad de E/S y rendimiento en el almacenamiento local en una amplia gama de sectores y aplicaciones.  La serie Lsv2 es ideal para macrodatos, SQL, bases de datos NoSQL, almacenamiento de datos y bases de datos transaccionales de gran tamaño, como Cassandra, MongoDB, Cloudera y Redis.

El diseño de las máquinas virtuales de la serie Lsv2 maximiza el procesador AMD EPYC™ 7551 para proporcionar el mejor rendimiento entre el procesador, la memoria, las máquinas virtuales y los dispositivos NVMe. Además de maximizar el rendimiento del hardware, las máquinas virtuales de la serie Lsv2 están diseñadas para responder a las necesidades de los sistemas operativos Windows y Linux para mejorar el rendimiento con el hardware y el software.

El ajuste del hardware y el software dio lugar a la versión optimizada de [Windows Server 2019 Datacenter](https://www.microsoft.com/cloud-platform/windows-server-pricing), que se publicó a principios de diciembre de 2018 en Azure Marketplace, que admite el rendimiento máximo en los dispositivos NVMe en las máquinas virtuales de la serie Lsv2.

En este artículo se proporcionan consejos y sugerencias para asegurarse de que las cargas de trabajo y las aplicaciones alcanzan el máximo rendimiento diseñado en las máquinas virtuales. La información de esta página se actualizará continuamente a medida que se agreguen más imágenes optimizadas de Lsv2 a Azure Marketplace.

## <a name="amd-eypc-chipset-architecture"></a>Arquitectura del conjunto de chips AMD EYPC™

Las máquinas virtuales de la serie Lsv2 usan procesadores de servidor AMD EYPC™ basados en la microarquitectura Zen. AMD desarrolló Infinity Fabric (IF) para EYPC™ como interconexión escalable para su modelo NUMA que podría usarse para las comunicaciones en la placa, en el paquete y en comunicaciones de varios paquetes. En comparación con QPI (Quick-Path Interconnect) y UPI (Ultra-Path Interconnect), que se usan en los procesadores de placa monolítica moderna de Intel, la arquitectura de muchas placas pequeñas de NUMA de AMD aporta beneficios de rendimiento, por un lado, y diferentes retos por el otro. La repercusión real de las restricciones de ancho de banda y latencia de memoria puede variar según el tipo de cargas de trabajo.

## <a name="tips-for-maximizing-performance"></a>Sugerencias para maximizar el rendimiento

* El hardware de las máquinas virtuales de la serie Lsv2 utiliza dispositivos NVMe con ocho pares de cola de E/S. Cada cola de E/S de dispositivo NVMe es realmente un par: una cola de envío y una cola de finalización. El controlador de NVMe está configurado para optimizar el uso de estos ocho pares de cola de E/S distribuyendo la E/S en una programación de round robin. Para obtener un rendimiento máximo, ejecute ocho trabajos por dispositivo para hacerlos coincidir.

* Evite mezclar los comandos de administrador de NVMe (por ejemplo, la consulta de información SMART de NVMe, etc.) con comandos de E/S de NVMe durante las cargas de trabajo activas. Los dispositivos NVMe de la serie Lsv2 están respaldados por la tecnología NVMe Direct de Hyper-V, que se activa en "modo de baja velocidad" cada vez que haya comandos de administrador de NVMe pendientes. Los usuarios de la serie Lsv2 podrían experimentar una importante caída en el rendimiento de E/S de NVMe si esto sucede.

* Los usuarios de Lsv2 no deben depender de la información de NUMA del dispositivo (todo 0) que se notifica en la máquina virtual para las unidades de datos para decidir la afinidad de NUMA para sus aplicaciones. La manera recomendada de mejorar el rendimiento es distribuir las cargas de trabajo entre las CPU, si es posible. 

* La profundidad máxima de cola admitida por cada par de cola de E/S para un dispositivo NVMe de una máquina virtual de la serie Lsv2 es 1024 (frente al límite de profundidad de cola de 32 de Amazon i3). Los usuarios de Lsv2 deben limitar sus cargas de trabajo de pruebas comparativas (sintéticas) a la profundidad de cola de 1024 o inferior para evitar que se desencadenen condiciones de cola completa, que pueden reducir el rendimiento.

## <a name="utilizing-local-nvme-storage"></a>Uso de almacenamiento local de NVMe

El almacenamiento local en el disco de NVMe de 1,92 TB de todas las máquinas virtuales de Lsv2 es efímero. Durante un reinicio estándar correcto de la máquina virtual, se conservarán los datos del disco local de NVMe. No se conservarán los datos en el NVMe si la máquina virtual se vuelve a implementar, se desasigna o se elimina. No se conservarán los datos si algún otro problema provoca un error en el estado de la máquina virtual o el hardware en que se está ejecutando. Cuando esto sucede, se borran de forma segura los datos en el host antiguo.

También habrá casos en que la máquina virtual debe moverse a una máquina de un host diferente; por ejemplo, durante una operación de mantenimiento planeado. Las operaciones de mantenimiento planeado y algunos errores de hardware pueden anticiparse con los [eventos programados](scheduled-events.md). Scheduled Events debe usarse para mantenerse actualizado en todas las operaciones de recuperación y mantenimiento previstas.

En el caso de que un evento de mantenimiento planeado requiera que la máquina virtual se vuelva a crear en un nuevo host con discos locales vacíos, los datos se deben volver a sincronizar (de nuevo, borrando de forma segura los datos del host antiguo). Esto ocurre porque las máquinas virtuales de la serie Lsv2 no admiten actualmente la migración en vivo en el disco local de NVMe.

Existen dos modos de realizar el mantenimiento planeado.

### <a name="standard-vm-customer-controlled-maintenance"></a>Mantenimiento controlado por el cliente de máquina virtual estándar

- La máquina virtual se mueve a un host actualizado durante un período de 30 días.
- Los datos del almacenamiento local de Lsv2 podrían perderse, por lo que se recomienda realizar copias de seguridad datos antes del evento.

### <a name="automatic-maintenance"></a>Mantenimiento automático

- Se produce si el cliente no ejecuta el mantenimiento controlado por el cliente o en caso de procedimientos de emergencias, como un evento de seguridad de día cero.
- Diseñado para conservar los datos del cliente, pero hay un pequeño riesgo de bloqueo o reinicio de la máquina virtual.
- Los datos del almacenamiento local de Lsv2 podrían perderse, por lo que se recomienda realizar copias de seguridad datos antes del evento.

Para los próximos eventos de servicio, utilice el proceso de mantenimiento controlado para seleccionar un momento más conveniente para la actualización. Antes del evento, puede realizar una copia de seguridad de los datos en almacenamiento premium. Una vez completado el evento de mantenimiento, puede devolver los datos al almacenamiento de NVMe local de las máquinas virtuales Lsv2 actualizadas.

Entre los escenarios que mantienen los datos en discos NVMe locales están los siguientes:

- La máquina virtual se está ejecutando y se encuentra en buen estado.
- La máquina virtual se reinicia in situ (por usted o Azure).
- La máquina virtual se pausa (detenida sin desasignación).
- La mayoría de las operaciones de servicio del mantenimiento planeado.

Los escenarios que borran de forma segura los datos para proteger al cliente incluyen:

- La máquina virtual se vuelve a implementar, se detiene (desasigna) o se elimina (por usted).
- La máquina virtual pasa a un estado incorrecto y es necesario recurrir a otro nodo debido a un problema de hardware.
- Un pequeño número de las operaciones de servicio de mantenimiento planeado que requiere que la VM se reasigne a otro host para el servicio.

Para más información acerca de las opciones de la copia de seguridad de datos en el almacenamiento local, consulte [Copia de seguridad y recuperación ante desastres para discos IaaS de Azure](../backup-and-disaster-recovery-for-azure-iaas-disks.md).

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

* **¿Cómo se puede iniciar la implementación de máquinas virtuales de la serie Lsv2?**  
   Al igual que cualquier otra máquina virtual, use el [Portal](quick-create-portal.md), la [CLI de Azure](quick-create-cli.md) o [PowerShell](quick-create-powershell.md) para crear una máquina virtual.

* **¿Un único error de disco NVMe provocará un error en todas las máquinas virtuales del host?**  
   Si se detecta un error de disco en el nodo de hardware, el hardware está en estado de error. Cuando esto ocurre, se anula la asignación de todas las máquinas virtuales del nodo y estas se mueven a un nodo en buen estado. En el caso de las máquinas virtuales de la serie Lsv2, esto significa que los datos del cliente en el nodo con error se borran de forma segura y el cliente tendrá que volver a crearlos en el nuevo nodo. Como se indicó, antes de que la migración en vivo esté disponible en Lsv2, los datos del nodo con error se moverán proactivamente con las máquinas virtuales a medida que se transfieran a otro nodo.

* **¿Es preciso realizar ajustes de sondeo en Windows Server 2012 o Windows Server 2016?**  
   El sondeo de NVMe solo está disponible en Windows Server 2019 en Azure.  

* **¿Puedo volver a un modelo de rutina de servicio de interrupción (ISR) tradicional?**  
   Máquinas virtuales de la serie Lsv2 están optimizadas para el sondeo de NVMe. Se proporcionan continuamente actualizaciones para mejorar el rendimiento del sondeo.

* **¿Se puede ajustar la configuración del sondeo en Windows Server 2019?**  
   El usuario no puede ajustar la configuración del sondeo.
   
## <a name="next-steps"></a>Pasos siguientes

* Consulte las especificaciones de todas las [máquinas virtuales optimizadas para el rendimiento del almacenamiento](../sizes-storage.md) en Azure
