---
title: Métricas de Azure NetApp Files | Microsoft Docs
description: Azure NetApp Files proporciona métricas sobre el almacenamiento asignado, el uso de almacenamiento real, el número de IOPS del volumen y la latencia. Use estas métricas para comprender el uso y el rendimiento.
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
ms.date: 03/17/2020
ms.author: b-juche
ms.openlocfilehash: 7e6ab90010d4379c1640f73a8deeba874e601daf
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513815"
---
# <a name="metrics-for-azure-netapp-files"></a>Métricas de Azure NetApp Files

Azure NetApp Files proporciona métricas sobre el almacenamiento asignado, el uso de almacenamiento real, el número de IOPS del volumen y la latencia. Mediante el análisis de estas métricas, puede comprender mejor el patrón de uso y el rendimiento de volumen de las cuentas de NetApp.  

## <a name="usage-metrics-for-capacity-pools"></a><a name="capacity_pools"></a>Métricas de uso de grupos de capacidad

<!-- 
- *Pool Provisioned Size*  
    The logical space (GiB) the capacity pool is provisioned with.  
    This size is the size you selected during capacity pool creation. 
--> 
- *Grupo asignado al tamaño del volumen*  
    Cuota de volumen (GiB) total en un grupo de capacidad determinado (es decir, el total de los tamaños aprovisionados de los volúmenes en el grupo de capacidad).  
    Es el tamaño que se selecciona durante la creación del volumen.  
- *Tamaño consumido del grupo*  
    Espacio lógico (GiB) total usado en todos los volúmenes de un grupo de capacidad.  
<!-- 
- *Pool Consumed Snapshot Size*  
    The total of logical space (GiB) used by snapshots across all volumes in a capacity pool. 
-->

## <a name="usage-metrics-for-volumes"></a><a name="volumes"></a>Métricas de uso de volúmenes

<!--
- *Volume Quota Size*    
    The quota size (GiB) the volume is provisioned with.   
    This size is the size you selected during capacity pool creation. 
-->
- *Tamaño consumido del volumen*   
    Espacio lógico total usado en un volumen (GiB).  
    Este tamaño incluye el espacio lógico usado por las instantáneas y los sistemas de archivos activos.  
- *Tamaño de instantánea de volumen*   
   Espacio lógico incremental usado por las instantáneas de un volumen.  

## <a name="performance-metrics-for-volumes"></a>Métricas de rendimiento de los volúmenes

- *AverageReadLatency*   
    Tiempo promedio de las lecturas del volumen en milisegundos.
- *AverageWriteLatency*   
    Tiempo promedio de las escrituras del volumen en milisegundos.
- *ReadIops*   
    Número de lecturas en el volumen por segundo.
- *WriteIops*   
    Número de escrituras en el volumen por segundo.

## <a name="next-steps"></a>Pasos siguientes

* [Información sobre la jerarquía del almacenamiento de Azure NetApp Files](azure-netapp-files-understand-storage-hierarchy.md)
* [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)
* [Creación de un volumen de Azure NetApp Files](azure-netapp-files-create-volumes.md)
