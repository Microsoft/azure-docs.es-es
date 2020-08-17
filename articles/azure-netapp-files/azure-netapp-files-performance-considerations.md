---
title: Consideraciones sobre el rendimiento de Azure NetApp Files | Microsoft Docs
description: Obtenga información sobre el rendimiento de Azure NetApp Files, incluida la relación de la cuota y el límite de rendimiento, y cómo aumentar o reducir la cuota de volumen de forma dinámica.
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
ms.date: 06/25/2019
ms.author: b-juche
ms.openlocfilehash: 5f88b4755c7b4c0b20f27065cf9de2351251bc1c
ms.sourcegitcommit: 29400316f0c221a43aff3962d591629f0757e780
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/02/2020
ms.locfileid: "87513883"
---
# <a name="performance-considerations-for-azure-netapp-files"></a>Consideraciones sobre el rendimiento de Azure NetApp Files

El [límite de rendimiento](azure-netapp-files-service-levels.md) de un volumen se determina mediante una combinación de la cuota asignada al volumen y el nivel de servicio seleccionado. Cuando realiza planes de rendimiento de Azure NetApp Files, debe tener en cuenta varias consideraciones. 

## <a name="quota-and-throughput"></a>Cuotas y rendimiento  

El límite de rendimiento es solo un factor determinante del rendimiento real que se realizará.  

Las consideraciones típicas de rendimiento del almacenamiento, incluyendo la mezcla de lectura y escritura, el tamaño de transferencia, los patrones aleatorios o secuenciales y muchos otros factores, contribuirán al rendimiento total entregado.  

El rendimiento empírico máximo que se ha observado en las pruebas es de 4500 MiB/s.  En la capa de almacenamiento Premium, una cuota de volumen de 70,31 TiB aprovisionará un límite de rendimiento que sea lo suficientemente alto para alcanzar ese nivel de rendimiento.  

Si quiere asignar cantidades de cuota de volumen más allá de los 70,31 TiB, puede asignar una cuota adicional a un volumen para almacenar datos adicionales. Sin embargo, la cuota adicional no dará como resultado un aumento adicional en el rendimiento real.  

## <a name="overprovisioning-the-volume-quota"></a>Aprovisionamiento excesivo de la cuota de volumen

Si el rendimiento de una carga de trabajo se limita en función del límite de rendimiento, es posible aprovisionar en exceso la cuota de volumen para establecer un nivel de rendimiento más alto y así lograr un mayor rendimiento.  

Por ejemplo, si un volumen en la capa de almacenamiento Premium tiene solo 500 GiB de datos pero requiere 128 MiB/s de rendimiento, puede establecer la cuota en 2 TiB para que el nivel de rendimiento se establezca en consecuencia (64 MiB/s por TB * 2 TiB = 128 MiB/s).  

Si aprovisiona constantemente un volumen en exceso para lograr un rendimiento más alto, puede usar un nivel de servicio más alto.  En el ejemplo anterior, puede alcanzar el mismo límite de rendimiento con la mitad de la cuota de volumen usando el nivel de almacenamiento Ultra en su lugar (128 MiB/s por TiB * 1 TiB = 128 MiB/s).

## <a name="dynamically-increasing-or-decreasing-volume-quota"></a>Aumentar o disminuir dinámicamente la cuota de volumen

Si sus requisitos de rendimiento son temporales o si ha aumentado las necesidades de rendimiento durante un período de tiempo fijo, puede aumentar o disminuir de forma dinámica la cuota de volumen para ajustar instantáneamente el límite de rendimiento.  Tenga en cuenta las siguientes consideraciones: 

* La cuota de volumen se puede aumentar o disminuir sin necesidad de pausar E/S, y el acceso al volumen no se interrumpe ni se ve afectado.  

    Puede ajustar la cuota durante una transacción activa de E/S a un volumen.  Tenga en cuenta que la cuota de volumen nunca puede reducirse por debajo de la cantidad de datos lógicos almacenados en el volumen.

* Cuando se cambia la cuota de volumen, el cambio correspondiente en el límite de rendimiento es casi instantáneo. 

    El cambio no interrumpe ni afecta el acceso al volumen o al E/S.  

* El ajuste de cuota de volumen requiere un cambio en el tamaño del grupo de capacidad.  

    El tamaño del grupo de capacidad se puede ajustar dinámicamente y sin que afecte a la disponibilidad de volumen o E/S.

## <a name="next-steps"></a>Pasos siguientes

- [Niveles de servicio para Azure NetApp Files](azure-netapp-files-service-levels.md)
- [Bancos de pruebas de rendimiento para Linux](performance-benchmarks-linux.md)