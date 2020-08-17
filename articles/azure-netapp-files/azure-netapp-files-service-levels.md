---
title: Niveles de servicio de Azure NetApp Files | Microsoft Docs
description: Se describe el rendimiento de los niveles de servicio de Azure NetApp Files.
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
ms.date: 01/09/2019
ms.author: b-juche
ms.openlocfilehash: 639f1e09fdb5603965209e5b5ee6c224ad238b76
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533128"
---
# <a name="service-levels-for-azure-netapp-files"></a>Niveles de servicio para Azure NetApp Files
Los niveles de servicio son un atributo de los grupos de capacidad. Los niveles de servicio se definen y se diferencian por el rendimiento máximo permitido en un volumen en el grupo de capacidad en función de la cuota asignada a ese volumen.

## <a name="supported-service-levels"></a>Niveles de servicio admitidos

Azure NetApp Files admite tres niveles de servicio: *Ultra*, *Premium* y *estándar*. 

* <a name="Ultra"></a>Almacenamiento Ultra

    La capa de almacenamiento Ultra ofrece hasta 128 MiB/s de rendimiento por 1 TiB de cuota de volumen asignado. 

* <a name="Premium"></a>Almacenamiento Premium

    La capa de almacenamiento Premium ofrece hasta 64 MiB/s de rendimiento por 1 TiB de cuota de volumen asignado. 

* <a name="Standard"></a>Almacenamiento Standard

    La capa de almacenamiento Estándar ofrece hasta 16 MiB/s de rendimiento por 1 TiB de cuota de volumen asignado.

## <a name="throughput-limits"></a>Límites de rendimiento

El límite de rendimiento de un volumen viene determinado por una combinación de los siguientes factores:
* El nivel de servicio del grupo de capacidad al que pertenece el volumen.
* La cuota asignada al volumen.  

Este concepto se ilustra en el siguiente diagrama:

![Ilustración de nivel de servicio](../media/azure-netapp-files/azure-netapp-files-service-levels.png)

En el ejemplo 1 anterior, un volumen de un grupo de capacidad con el nivel de almacenamiento Premium y una cuota de 2 TiB tendrá asignado un límite de rendimiento de 128 MiB/s (2 TiB x 64 MiB/s). Este escenario será así independientemente del tamaño del grupo de capacidad o del consumo de volumen real.

En el ejemplo 2 anterior, un volumen de un grupo de capacidad con el nivel de almacenamiento Premium y una cuota de 100 GiB tendrá asignado un límite de rendimiento de 6,25 MiB/s (0,09765625 TiB x 64 MiB/s). Este escenario será así independientemente del tamaño del grupo de capacidad o del consumo de volumen real.

## <a name="next-steps"></a>Pasos siguientes

- [Página de precios de Azure NetApp Files](https://azure.microsoft.com/pricing/details/storage/netapp/)
- [Modelo de costo de Azure NetApp Files](azure-netapp-files-cost-model.md) 
- [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)
- [Acuerdo de Nivel de Servicio (SLA) para Azure NetApp Files](https://azure.microsoft.com/support/legal/sla/netapp/)
- [Cambio dinámico del nivel de servicio de un volumen](dynamic-change-volume-service-level.md) 