---
title: Modelo de costos para los archivos de NetApp de Azure | Microsoft Docs
description: Describe el modelo de costos de Azure Files de NetApp para administrar los gastos desde el servicio.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/01/2019
ms.author: b-juche
ms.openlocfilehash: b06e3366224b90899dd3f9f9439edf897de82794
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65524226"
---
# <a name="cost-model-for-azure-netapp-files"></a>Modelo de costo de Azure NetApp Files 

Descripción del modelo de costo para los archivos de NetApp de Azure le ayuda a administrar sus gastos desde el servicio.

## <a name="calculation-of-capacity-consumption"></a>Cálculo del consumo de la capacidad

Los archivos de NetApp Azure se factura conforme a la capacidad de almacenamiento aprovisionado.  Capacidad aprovisionada se asigna mediante la creación de grupos de capacidad.  Grupos de capacidades se facturan según $/ aprovisionado-GiB/mes en incrementos de una hora. El tamaño mínimo para un grupo de capacidades único es de 4 TB, y los grupos de capacidad pueden expandirse posteriormente en incrementos de 1 TiB. Los volúmenes se crean dentro de grupos de capacidad.  Cada volumen se asigna una cuota que disminuye a partir de la capacidad de aprovisionamiento de grupos. La cuota que se puede asignar a los volúmenes intervalos entre un mínimo de 100 GB hasta un máximo de TiB 92.  

Para un volumen activo, consumo de la capacidad para cuota se basa en lógica capacidad (efectiva).

Si el consumo de la capacidad real de un volumen supera la cuota de almacenamiento, puede continuar aumentando el volumen. Escrituras todavía podrá siempre que el tamaño del volumen real es menor que el límite del sistema (100 TB).  

La capacidad total usada en un grupo de capacidades contra su cantidad es la suma de la mayor de la cuota de asignado o real consumo de todos los volúmenes del grupo: 

   ![Cálculo de la capacidad total que se usa](../media/azure-netapp-files/azure-netapp-files-total-used-capacity.png)

El diagrama siguiente muestra estos conceptos.  
* Contamos con un grupo de capacidades con 4 TB de capacidad aprovisionada.  El grupo contiene tres volúmenes.  
    * Volumen 1 se asigna una cuota de 2 TB y tiene 800 GB de consumo.  
    * Volumen 2 se asigna una cuota de 1 TiB y tiene 100 GB de consumo.  
    * Volumen 3 se asigna una cuota de 500 GB, pero tiene 800 GB de consumo (por encima del límite).  
* El grupo de capacidad se mide para 4 TB de capacidad (la cantidad aprovisionada).  
    3,8 se consume TiB de capacidad (TiB 2 y 1 TiB de cuota a partir de los volúmenes 1 y 2 y 800 GB de consumo real para el volumen 3). Y 200 GB de capacidad restante.

   ![Grupo de capacidades con tres volúmenes](../media/azure-netapp-files/azure-netapp-files-capacity-pool-with-three-vols.png)

## <a name="overage-in-capacity-consumption"></a>Por encima del límite de consumo de la capacidad  

Cuando el total de capacidad de un grupo utilizado supera su capacidad aprovisionada, todavía se permiten las escrituras de datos.  Tras el período de gracia (una hora), si la capacidad usada del grupo todavía supera su capacidad aprovisionada, a continuación, el tamaño del grupo automáticamente aumentará en incrementos de 1 TiB hasta que la capacidad aprovisionada es mayor que la capacidad total usada.  Por ejemplo, en la ilustración anterior, si volumen 3 sigue creciendo y el consumo real alcanza 1,2 TB, a continuación, tras el período de gracia, el grupo automáticamente tamaño se establecerá a 5 TB.  El resultado es que la capacidad del grupo aprovisionado (5 TB) supera la capacidad usada (TiB 4.2).  

## <a name="manual-changes-of-the-pool-size"></a>Cambios manuales, el tamaño del grupo  

Manualmente, puede aumentar o disminuir el tamaño del grupo. Sin embargo, se aplican las siguientes restricciones:
* Límites mínimos y máximos de servicio  
    Consulte el artículo sobre [los límites de recursos](azure-netapp-files-resource-limits.md).
* Un incremento de 1 TiB después de la compra mínima de 4 TiB inicial
* Un incremento de facturación mínimo de una hora
* No puede disminuir el tamaño del grupo aprovisionado a menos que el total de capacidad usada del grupo.

## <a name="behavior-of-maximum-size-pool-overage"></a>Comportamiento de por encima del límite de tamaño máximo de grupo   

El tamaño máximo de un grupo de capacidades que puede crear o cambiar el tamaño a es de 500 TB.  Cuando el total de capacidad usada en un grupo de capacidades supera 500 TiB, se producen a las situaciones siguientes:
* Escrituras de datos todavía podrán (si el volumen está por debajo del máximo del sistema de 100 TB).
* Tras el período de gracia de una hora, el grupo de tamaño se establecerá automáticamente en incrementos de 1 TB, hasta que la capacidad del grupo aprovisionado supera la capacidad total que se usa.
* El adicional había aprovisionado y factura superior a 500 TiB no se puede usar para asignar la cuota de volumen de capacidad del grupo. También se no se puede usar para ampliar los límites de calidad de servicio de rendimiento.  
    Consulte [niveles de servicio](azure-netapp-files-service-levels.md) sobre los límites de rendimiento y ajuste de tamaño de QoS.

El diagrama siguiente muestra estos conceptos:
* Contamos con un grupo de capacidades con una capa de almacenamiento Premium y una capacidad de 500 TB. El grupo contiene nueve volúmenes.
    * Volúmenes de 1 a 8 se asignan una cuota de cada 60 TB.  La capacidad total usada es 480 TB.  
        Cada volumen tiene un límite de QoS de 3,75 GiB/s de rendimiento (TiB 60 * 64 MiB/s).  
    * Volumen 9 se asigna una cuota de 20 TB.  
        Volumen 9 tiene un límite de QoS de 1,25 GiB/s de rendimiento (TiB 60 * 64 MiB/s).
* Volumen 9 es un escenario por encima del límite. Tiene 25 TiB de consumo real.  
    * Tras el período de gracia de una hora, el grupo de capacidad se ajustará a TiB 505.  
        Es decir, total utilizado capacidad = 8 * 60-cuota TiB de volúmenes de 1 a 8 y 25 TiB de consumo real de volumen 9.
    * La capacidad facturada es TiB 505.
    * No se puede aumentar la cuota de volumen para volumen 9 (debido a la cuota total asignada para el grupo no puede superar los 500 TB).
    * No se puede asignar rendimiento adicional de QoS (porque la QoS total para el grupo todavía se basa en 500 TiB).

   ![Grupo de capacidades con nueve volúmenes](../media/azure-netapp-files/azure-netapp-files-capacity-pool-with-nine-vols.png)

## <a name="capacity-consumption-of-snapshots"></a>Consumo de capacidad de instantáneas 

El consumo de la capacidad de instantáneas de Azure Files de NetApp se carga en la cuota del volumen primario.  Como resultado, comparte la misma tarifa de facturación que el grupo de capacidades a la que pertenece el volumen.  Sin embargo, a diferencia del volumen activo, el consumo de instantánea se mide según la capacidad incremental consumida.  Las instantáneas de archivos de NetApp Azure tienen diferenciales por naturaleza. Según la tasa de cambio de los datos, las instantáneas a menudo consumen mucho menos capacidad que la capacidad del volumen activo lógica. Por ejemplo, suponga que tiene una instantánea de un volumen de 500 GiB que sólo contiene 10 GB de datos diferenciales. La capacidad que se realiza el cargo en la cuota de volumen de esa instantánea sería de 10 GiB, no 500 GiB. 

## <a name="next-steps"></a>Pasos siguientes

* [Página de precios de Azure Files de NetApp](https://azure.microsoft.com/pricing/details/storage/netapp/)
* [Niveles de servicio para Azure NetApp Files](azure-netapp-files-service-levels.md)
* [Límites de recursos para Azure NetApp Files](azure-netapp-files-resource-limits.md)
