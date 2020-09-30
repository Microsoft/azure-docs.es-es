---
title: Replicación entre regiones de volúmenes de Azure NetApp Files | Microsoft Docs
description: Describe lo que hace la replicación entre regiones de Azure NetApp Files, los pares de regiones admitidos, los objetivos de nivel de servicio, la durabilidad de los datos y el modelo de costos.
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
ms.date: 09/16/2020
ms.author: b-juche
ms.openlocfilehash: e7d424435c29eb3e7a6779a7036816a7bedd2085
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90708443"
---
# <a name="cross-region-replication-of-azure-netapp-files-volumes"></a>Replicación entre regiones de volúmenes de Azure NetApp Files

La funcionalidad de replicación de Azure NetApp Files proporciona protección de datos a través de la replicación de volúmenes entre regiones. Puede replicar datos de forma asincrónica desde un volumen de Azure NetApp Files (origen) de una región a otro volumen Azure NetApp Files (destino) de otra región.  Esta funcionalidad le permite realizar la conmutación por error de la aplicación crítica en caso de un desastre o una interrupción en toda la región.

> [!IMPORTANT]
> La característica de replicación entre regiones se encuentra actualmente en versión preliminar pública. Debe enviar una solicitud de lista de espera para acceder a la característica a través de la [página de envío de lista de espera de replicación entre regiones de Azure NetApp Files](https://aka.ms/anfcrrpreviewsignup). Antes de utilizar la característica de replicación entre regiones, debe esperar un correo electrónico de confirmación oficial del equipo de Azure NetApp Files.

## <a name="supported-region-pairs"></a>Pares de regiones admitidas

La replicación de volúmenes de Azure NetApp Files está disponible actualmente en los siguientes pares de regiones fijas:  

* Oeste de EE. UU. y Este de EE. UU.
* Oeste de EE. UU. 2 y Este de EE. UU. 
* Centro y Sur de EE. UU. y Centro de EE. UU. 
* Centro y Sur de EE. UU. y Este de EE. UU.
* Centro y Sur de EE. UU. y Este de EE. UU. 2 
* Este de EE. UU. 2 y Centro de EE. UU. 
* Norte de Europa y Oeste de Europa
* Sur de Reino Unido y Oeste de Reino Unido
* Este de Australia y Sudeste de Australia
* Centro de Canadá y Este de Canadá
* Japón Oriental y Japón Occidental
* Sur de Reino Unido y Centro-oeste de Alemania
* Sudeste de Asia y Este de Australia

## <a name="service-level-objectives"></a>Objetivos de nivel de servicio

Los objetivos de punto de recuperación (RPO), o la pérdida de datos máxima tolerable, se define como dos veces la programación de replicación.  El RPO real observado puede variar en función de factores como el tamaño total del conjunto de datos junto con la tasa de cambio, el porcentaje de sobrescrituras de datos y el ancho de banda de replicación disponible para la transferencia.   

* Para la programación de replicación de 10 minutos, el RPO máximo es de 20 minutos.  
* Para la programación de replicación por hora, el RPO máximo es de dos horas.  
* Para la programación de replicación diaria, el RPO máximo es de dos días.  

El objetivo de tiempo de recuperación (RTO), o el tiempo de inactividad máximo tolerable de la aplicación empresarial, viene determinado por factores en la presentación de la aplicación y el aprovisionamiento de acceso a los datos en el segundo sitio. Se prevé que la parte del almacenamiento del RTO para romper la relación de emparejamiento con la finalidad de activar el volumen de destino y proporcionar acceso de lectura y escritura de datos en el segundo sitio se complete en un minuto.

## <a name="cost-model-for-cross-region-replication"></a>Modelo de costos de la replicación entre regiones  

Con la replicación entre regiones de Azure NetApp Files, solo se paga por la cantidad de datos que se replican. No se aplican cargos de configuración ni una cuota mínima de uso. El precio de replicación se basa en la frecuencia de replicación y en la región del volumen de *destino* que elija durante la configuración de replicación inicial. Para más información, consulte la página [Precios de Azure NetApp Files](https://azure.microsoft.com/pricing/details/netapp/).  

El cargo de la capacidad de almacenamiento de Azure NetApp Files regular se aplica al volumen de destino de replicación (también denominado volumen *protección de datos*). 

### <a name="pricing-examples"></a>Ejemplos de precios

La cantidad de replicación entre regiones facturada en un mes se basa en la cantidad de datos replicados a través de la característica de replicación entre regiones durante ese mes. La cantidad de datos replicados se mide en GiB. Representa la suma de los datos replicados en dos regiones durante todas las replicaciones normales de los volúmenes de origen a los volúmenes de destino y durante todas las replicaciones de resincronización de los volúmenes de destino a los volúmenes de origen.

#### <a name="example-1-month-1-baseline-replication-and-incremental-replications"></a>Ejemplo 1: Replicación de base de referencia del mes 1 y replicaciones incrementales

Considere las situaciones siguientes:

* El volumen de *origen* corresponde al nivel de servicio *Premium* de Azure NetApp Files. Tiene un tamaño de cuota de volumen de 1000 GiB y un tamaño de volumen consumido de 500 GiB al principio del primer día de un mes. El volumen se encuentra en la región *Centro y Sur de EE. UU.*
* El volumen de *destino* corresponde al nivel de servicio *Standard* de Azure NetApp Files. Está en la región *Este de EE. UU. 2*.
* Ha configurado una replicación entre regiones *por hora* entre los dos volúmenes anteriores. Por lo tanto, el precio de la replicación es de 0,12 USD por GiB.
* Por motivos de simplicidad, supongamos que el volumen de origen tiene un cambio de datos constante de 0,5 GiB cada hora, pero el tamaño total consumido del volumen no crece (permanece en 500 GiB). 

Después de la instalación inicial, la replicación de base de referencia se produce inmediatamente.  

* Cantidad de datos replicada durante la replicación de base de referencia: `500 GiB`
* Cargos de replicación de base de referencia: `500 GiB * $0.12 = $60`

Después de la replicación de base de referencia, solo se replican los bloques modificados. Por lo tanto, solo se replicarán 0,5 GiB de datos cada hora en las sucesivas réplicas incrementales.

* Suma de la cantidad de datos replicada entre las replicaciones incrementales durante un mes de 30 días: `0.5 GiB * 24 hours * 30 days = 360 GiB`
* Cargos de replicación incremental: `360 GiB * $0.12 = $43.2`

Al final del mes 1, el cargo total de la replicación entre regiones es el siguiente:  

*  Cargo total de replicación entre regiones del mes 1: `$60 + $43.2 = $103.2`

El cargo normal de la capacidad de almacenamiento de Azure NetApp Files se aplica al volumen de destino. Sin embargo, el volumen de destino puede usar una capa de almacenamiento diferente de la del nivel de volumen de origen (y más económica).

#### <a name="example-2-month-2-incremental-replications-and-resync-replications"></a>Ejemplo 2: Replicaciones incrementales del mes 2 y replicaciones de resincronización  

Supongamos que tiene un volumen de origen, un volumen de destino y una relación de replicación entre los dos configurados como se describe en el ejemplo 1. Durante 29 días del segundo mes (un mes de 30 días), las replicaciones por hora se produjeron según lo previsto.

* Suma de la cantidad de datos replicados entre las replicaciones incrementales durante 29 días: `0.5 GiB * 24 hours * 29 days = 348 GiB`

Supongamos que el último día del mes, se produjo una interrupción no planeada en la región de origen y que realizó la conmutación por error en el volumen de destino. Después de 2 horas, la región de origen se recuperó y realizó una replicación de resincronización del volumen de destino al volumen de origen. Durante las 2 horas, se produjeron 0,8 GiB de cambios de datos en el volumen de destino y fue necesaria la resincronización en el origen.

* Suma de la cantidad de datos replicados entre replicaciones normales durante 22 horas el último día: `0.5 GiB * 22 hours = 11 GiB`
* Cantidad de datos replicados durante una replicación de resincronización: `0.8 GiB`

Por lo tanto, al final del mes 2, el cargo total de la replicación entre regiones es el siguiente:  

* Cargo total de replicación entre regiones del mes 2: `(348 GiB + 11 GiB + 0.8 GiB) * $0.12 = $43.18`

El cargo normal de la capacidad de almacenamiento de Azure NetApp Files del mes 2 se aplica al volumen de destino.

## <a name="next-steps"></a>Pasos siguientes
* [Requisitos y consideraciones del uso de la replicación entre regiones](cross-region-replication-requirements-considerations.md)
* [Creación de emparejamiento de replicación](cross-region-replication-create-peering.md)
* [Visualización del estado de mantenimiento de la relación de la replicación](cross-region-replication-display-health-status.md)
* [Administración de la recuperación ante desastres](cross-region-replication-manage-disaster-recovery.md)
* [Métricas de replicación de volúmenes](azure-netapp-files-metrics.md#replication)
* [Solución de problemas de la replicación entre regiones](troubleshoot-cross-region-replication.md)


