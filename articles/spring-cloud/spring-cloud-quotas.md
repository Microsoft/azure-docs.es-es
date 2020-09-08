---
title: Planes de servicio y cuotas para Azure Spring Cloud
description: Más información sobre las cuotas y los planes de servicio de Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 7d47311385848f96a12a50c1dcf8b8284869c71d
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89260363"
---
# <a name="quotas-and-service-plans-for-azure-spring-cloud"></a>Cuotas y planes de servicio de Azure Spring Cloud

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
