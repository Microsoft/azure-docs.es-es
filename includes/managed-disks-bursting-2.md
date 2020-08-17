---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: albecker1
ms.service: virtual-machines
ms.topic: include
ms.date: 04/27/2020
ms.author: albecker1
ms.custom: include file
ms.openlocfilehash: cc0dd5e76227c0814659b74afc2ac46c8ca4de73
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87102991"
---
## <a name="common-scenarios"></a>Escenarios frecuentes
Los siguientes escenarios pueden beneficiarse en gran medida de la expansión:
- **Mejora de los tiempos de arranque**: con la expansión, la instancia se iniciará a una velocidad considerablemente más rápida. Por ejemplo, el disco de SO predeterminado para las VM habilitadas para Premium es el P4, con un rendimiento aprovisionado de hasta 120 IOPS y 25 MB/s. Con la expansión, el disco P4 puede llegar hasta 3500 IOPS y 170 MB/s, lo que permite multiplicar por 6 la aceleración del tiempo de arranque.
- **Control de los trabajos por lotes**: algunas cargas de trabajo de la aplicación son cíclicas por naturaleza y requieren un rendimiento de base de referencia la mayor parte del tiempo, así como un alto rendimiento durante un breve período de tiempo. Un ejemplo de esto es un programa de contabilidad que procesa transacciones diariamente y que requiere una pequeña cantidad de tráfico de disco. Después, al finalizar el mes, se concilian los informes que requieren una cantidad mucho mayor de tráfico de disco.
- **Preparación para los picos de tráfico**: los servidores web y sus aplicaciones pueden experimentar picos de tráfico en cualquier momento. Si el servidor web está respaldado por VM o discos que usan expansión, los servidores están mejor equipados para controlar los picos de tráfico. 

## <a name="bursting-flow"></a>Flujo de expansión
El sistema de crédito de expansión se aplica de la misma manera en el nivel de máquina virtual y en el de disco. El recurso, ya sea una VM o un disco, se iniciará con un gran abastecimiento de créditos. Estos créditos le permitirán expansiones de 30 minutos a la velocidad máxima de expansión. Los créditos de expansión se acumulan cuando el recurso se ejecuta por debajo de sus límites de almacenamiento en disco de rendimiento. Por todas las IOPS y MB/s que el recurso use por debajo del límite de rendimiento, empezará a acumular créditos. Si el recurso ha acumulado créditos para usarlos para la expansión y la carga de trabajo necesita un rendimiento adicional, el recurso puede usar esos créditos para superar el límite de rendimiento y proporcionar el rendimiento de E/S de disco que necesita para satisfacer la demanda.



![Diagrama de cubos de expansión](media/managed-disks-bursting/bucket-diagram.jpg)

Cómo quiere usar los 30 minutos de expansión es una decisión exclusivamente suya. Puede usar la expansión durante 30 minutos consecutivos o esporádicamente a lo largo del día. Cuando el producto se implementa, está listo para recibir créditos completos y, cuando se agotan, se tarda menos de un día en reabastecer completamente todos los créditos. Puede acumular y gastar sus créditos de expansión como crea adecuado y no es necesario que el depósito de 30 minutos se llene de nuevo para realizar una expansión. Una cuestión que hay que tener en cuenta sobre la acumulación de la expansión es que es diferente para cada recurso, ya que se basa en las IOPS no utilizadas y en los MB/s por debajo de sus cantidades de rendimiento. Esto significa que los productos de mayor rendimiento de base de referencia pueden acumular sus cantidades de expansión más rápido que los productos con un rendimiento de base de referencia inferior. Por ejemplo, un disco P1 inactivo acumulará 120 IOPS por segundo, mientras que un disco P20 acumula 2300 IOPS por segundo si está inactivo.

## <a name="bursting-states"></a>Estados de expansión
Hay tres estados que puede tener el recurso con la expansión habilitada:
- **Acumulación** : el tráfico de E/S del recurso tiene un rendimiento inferior al de destino. La acumulación de créditos de expansión para IOPS y MB/s se realiza de forma independiente. El recurso puede acumular créditos de IOPS y gastar créditos de MB/s o viceversa.
- **Expansión**: el tráfico del recurso tiene un rendimiento superior al de destino. El tráfico de ráfaga consumirá créditos de IOPS o ancho de banda de forma independiente.
- **Constante**: el tráfico del recurso tiene un rendimiento igual al de destino.

## <a name="examples-of-bursting"></a>Ejemplos de expansión
En los siguientes ejemplos se muestra cómo funciona la expansión con varias combinaciones de máquinas virtuales y discos. Para facilitar el seguimiento de los ejemplos, nos centraremos en los MB/s, pero se aplica la misma lógica independientemente de las IOPS.

### <a name="non-burstable-virtual-machine-with-burstable-disks"></a>Máquina virtual no expansible con discos expansibles
**Combinación de VM y discos:** 
- Standard_D8as_v4 
    - MB/s no almacenados en caché: 192
- Disco de SO P4
    - MB/s aprovisionados: 25
    - Máximo de MB/s de expansión: 170 
- 2 discos de datos P10 
    - MB/s aprovisionados: 25
    - Máximo de MB/s de expansión: 170

 Cuando la VM se inicie, recuperará los datos del disco del SO. Puesto que el disco de SO forma parte de una VM que se está iniciando, el disco del SO estará lleno de créditos de expansión. Estos créditos permitirán expandir el inicio del disco del SO a 170 MB/s segundos, tal como se muestra a continuación:

![Inicio del disco de expansión de VM sin expansión](media/managed-disks-bursting/nonbursting-vm-busting-disk/nonbusting-vm-bursting-disk-startup.jpg)

Una vez completado el arranque, se ejecuta una aplicación en la VM y tiene una carga de trabajo no crítica. Esta carga de trabajo requiere 15 MB/s que se distribuyen uniformemente entre todos los discos:

![Inactividad del disco de expansión de VM sin expansión](media/managed-disks-bursting/nonbursting-vm-busting-disk/nonbusting-vm-bursting-disk-idling.jpg)

A continuación, la aplicación necesita procesar un trabajo por lotes que requiere 192 MB/s. El disco del SO usa 2 MB/s y el resto se divide uniformemente entre los discos de datos:

![Expansión del disco de expansión de VM sin expansión](media/managed-disks-bursting/nonbursting-vm-busting-disk/nonbusting-vm-bursting-disk-bursting.jpg)

### <a name="burstable-virtual-machine-with-non-burstable-disks"></a>Máquina virtual expansible con discos no expansibles
**Combinación de VM y discos:** 
- Standard_L8s_v2 
    - MB/s no almacenados en caché: 160
    - Máximo de MB/s de expansión: 1 280
- Disco de SO P50
    - MB/s aprovisionados: 250 
- 2 discos de datos P10 
    - MB/s aprovisionados: 250

 Una vez completado el arranque inicial, se ejecuta una aplicación en la VM y tiene una carga de trabajo no crítica. Esta carga de trabajo requiere 30 MB/s que se distribuyen uniformemente entre todos los discos: ![Inactividad del disco sin expansión de la VM de expansión](media/managed-disks-bursting/bursting-vm-nonbursting-disk/burst-vm-nonbursting-disk-normal.jpg)

A continuación, la aplicación necesita procesar un trabajo por lotes que requiere 600 MB/s. Standard_L8s_v2 se expande para satisfacer esta demanda y, a continuación, las solicitudes a los discos se distribuyen uniformemente para los discos P50:

![Expansión del disco sin expansión de VM de expansión](media/managed-disks-bursting/bursting-vm-nonbursting-disk/burst-vm-nonbursting-disk-bursting.jpg)
### <a name="burstable-virtual-machine-with-burstable-disks"></a>Máquina virtual expansible con discos expansibles
**Combinación de VM y discos:** 
- Standard_L8s_v2 
    - MB/s no almacenados en caché: 160
    - Máximo de MB/s de expansión: 1 280
- Disco de SO P4
    - MB/s aprovisionados: 25
    - Máximo de MB/s de expansión: 170 
- 2 discos de datos P4 
    - MB/s aprovisionados: 25
    - Máximo de MB/s de expansión: 170 

Cuando arranca la VM, se expande para solicitar su límite de ráfagas de 1280 MB/s del disco de SO y este disco responde con su rendimiento de expansión de 170 MB/s:

![Inicio del disco de expansión de VM de expansión](media/managed-disks-bursting/bursting-vm-bursting-disk/burst-vm-burst-disk-startup.jpg)

Una vez completado el arranque, se ejecuta una aplicación en la VM. La aplicación tiene una carga de trabajo no crítica que requiere 15 MB/s que se distribuyen uniformemente entre todos los discos:

![Inactividad del disco de expansión de VM de expansión](media/managed-disks-bursting/bursting-vm-bursting-disk/burst-vm-burst-disk-idling.jpg)

A continuación, la aplicación necesita procesar un trabajo por lotes que requiere 360 MB/s. Standard_L8s_v2 se expande para satisfacer esta demanda y, a continuación, realiza una solicitud. El disco del SO solo necesita 20 MB/s. Los 340 MB/s restantes se administran mediante los discos de datos P4 de expansión:  

![Expansión del disco de expansión de VM de expansión](media/managed-disks-bursting/bursting-vm-bursting-disk/burst-vm-burst-disk-bursting.jpg)
