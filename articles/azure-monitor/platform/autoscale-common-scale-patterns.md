---
title: Información general sobre los patrones comunes de escalado automático
description: Obtenga información sobre algunos de los patrones comunes de escalado automático de recursos en Azure.
ms.topic: conceptual
ms.date: 05/07/2017
ms.subservice: autoscale
ms.openlocfilehash: adbfecd05bfd4330e7c7c972cfdb9fe7021eec31
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87073508"
---
# <a name="overview-of-common-autoscale-patterns"></a>Información general sobre los patrones comunes de escalado automático
En este artículo se describen algunos de los patrones comunes para escalar recursos en Azure.

La escalabilidad automática de Azure Monitor solo se aplica a [Virtual Machine Scale Sets](https://azure.microsoft.com/services/virtual-machine-scale-sets/), [Cloud Services](https://azure.microsoft.com/services/cloud-services/), [App Service - Web Apps](https://azure.microsoft.com/services/app-service/web/) y los [servicios de API Management](../../api-management/api-management-key-concepts.md).

## <a name="lets-get-started"></a>Introducción

En este artículo se asume que está familiarizado con el escalado automático. Puede [comenzar aquí a escalar los recursos][1]. A continuación, se indican algunos de los patrones comunes de escalado.

## <a name="scale-based-on-cpu"></a>Escala en función de la CPU

Tiene una aplicación web (rol de VMSS o de servicio en la nube) y:

- Desea escalar o reducir horizontalmente en función de la CPU.
- Además, desea asegurarse de que hay un número mínimo de instancias.
- También quiere asegurarse de que establece un límite máximo del número de instancias al que puede escalar.

![Escala en función de la CPU][2]

## <a name="scale-differently-on-weekdays-vs-weekends"></a>Escalado distinto entre los días de la semana y los fines de semana

Tiene una aplicación web (rol de VMSS o de servicio en la nube) y:

- Desea 3 instancias de forma predeterminada (en días de la semana).
- No espera que haya tráfico durante los fines de semana y, por tanto, desea reducir verticalmente a 1 instancia para los fines de semana.

![Escalado distinto entre los días de la semana y los fines de semana][3]

## <a name="scale-differently-during-holidays"></a>Escalado distinto durante festividades

Tiene una aplicación web (rol de VMSS o de servicio en la nube) y:

- Desea escalar o reducir verticalmente en función del uso de CPU de forma predeterminada.
- Sin embargo, durante Navidad u otros días específicos importantes para su negocio, desea reemplazar los valores predeterminados y disponer de más capacidad.

![Escalado distinto durante festividades][4]

## <a name="scale-based-on-custom-metric"></a>Escalado en función de métricas personalizadas

Tiene un front-end web y un nivel de API que se comunica con el back-end.

- Quiere escalar el nivel de API en función de los eventos personalizados en el front-end (ejemplo: Quiere escalar el proceso de finalización de la compra en función del número de artículos en el carro de la compra).

![Escalado en función de métricas personalizadas][5]

<!--Reference-->
[1]: ./autoscale-get-started.md
[2]: ./media/autoscale-common-scale-patterns/scale-based-on-cpu.png
[3]: ./media/autoscale-common-scale-patterns/weekday-weekend-scale.png
[4]: ./media/autoscale-common-scale-patterns/holidays-scale.png
[5]: ./media/autoscale-common-scale-patterns/custom-metric-scale.png
