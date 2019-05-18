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
ms.date: 04/22/2019
ms.author: b-juche
ms.openlocfilehash: 1f9c427045c9d42f6a11cc4bcc798cfc47a4428c
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523106"
---
# <a name="service-levels-for-azure-netapp-files"></a>Niveles de servicio para Azure NetApp Files
Los niveles de servicio son un atributo de un grupo de capacidades. Los niveles de servicio se definen y se diferencian por el rendimiento máximo permitido para un volumen en el grupo de capacidad en función de la cuota que se asigna al volumen.

## <a name="supported-service-levels"></a>Niveles de servicio admitidos

Azure Files de NetApp admite tres niveles de servicio: *Ultra*, *Premium*, y *estándar*. 

* <a name="Ultra"></a>Almacenamiento Ultra

    La capa de almacenamiento Ultra ofrece hasta 128 MiB/s de rendimiento por 1 TiB de cuota de volumen asignado. 

* <a name="Premium"></a>Almacenamiento Premium

    La capa de almacenamiento Premium ofrece hasta 64 MiB/s de rendimiento por 1 TiB de cuota de volumen asignado. 

* <a name="Standard"></a>Almacenamiento Standard

    La capa de almacenamiento estándar proporciona hasta 16 MiB/s de rendimiento por 1 TiB de cuota de volumen asignado.

## <a name="throughput-limits"></a>Límites de rendimiento

El límite de rendimiento para un volumen viene determinada por la combinación de los siguientes factores:
* El nivel de servicio del grupo de capacidades a la que pertenece el volumen
* La cuota asignada al volumen  

Este concepto se ilustra en el diagrama siguiente:

![Ilustración de nivel de servicio](../media/azure-netapp-files/azure-netapp-files-service-levels.png)

En el ejemplo 1 anterior, se le asignará un volumen de un grupo de capacidades con el nivel de almacenamiento Premium que se asigna a 2 TiB de cuota de un límite de rendimiento de 128 MiB/s (2 TiB * 64 MiB/s). Este escenario se aplica independientemente del tamaño del grupo de capacidad o el consumo real del volumen.

En el ejemplo 2 anterior, se le asignará un volumen de un grupo de capacidades con el nivel de almacenamiento Premium que se asigna a 100 GB de cuota de un límite de rendimiento de 6,25 MiB/s (TiB 0.09765625 * 64 MiB/s). Este escenario se aplica independientemente del tamaño del grupo de capacidad o el consumo real del volumen.

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [página de precios de Azure NetApp Files](https://azure.microsoft.com/pricing/details/storage/netapp/) para conocer el precio de los diferentes niveles de servicio
- Consulte [modelo de costos de Azure Files de NetApp](azure-netapp-files-cost-model.md) para el cálculo del consumo de capacidad en un grupo de capacidades 
- [Configuración de un grupo de capacidad](azure-netapp-files-set-up-capacity-pool.md)