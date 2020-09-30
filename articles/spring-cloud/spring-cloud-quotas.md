---
title: Planes de servicio y cuotas para Azure Spring Cloud
description: Más información sobre las cuotas y los planes de servicio de Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 7c4c832819f61d208d0722823d0a74354960f182
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90904273"
---
# <a name="quotas-and-service-plans-for-azure-spring-cloud"></a>Cuotas y planes de servicio de Azure Spring Cloud

**Este artículo se aplica a:** ✔️ Java ✔️ C#

Todos los servicios de Azure establecen límites y cuotas predeterminados para recursos y características.   Azure Spring Cloud ofrece dos planes de tarifa: Básico y Estándar. En este artículo se detallan los límites de ambos niveles.

## <a name="azure-spring-cloud-service-tiers-and-limits"></a>Límites y niveles de servicio de Azure Spring Cloud

| Recurso | Básico | Estándar
------- | ------- | -------
vCPU | 1 por instancia de servicio | 4 por instancia de servicio
Memoria | 2 GB por instancia de servicio | 8 GB por instancia de servicio
Instancias del servicio Azure Spring Cloud por región y suscripción | 10 | 10
Instancias de aplicación totales por instancia del servicio Azure Spring Cloud | 25 | 500
Volúmenes persistentes | 1 GB/aplicación x 10 aplicaciones | 50 GB/aplicación x 10 aplicaciones

## <a name="next-steps"></a>Pasos siguientes

Algunos límites predeterminados se pueden aumentar. Si su configuración requiere un aumento, [cree una solicitud de soporte técnico](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).
