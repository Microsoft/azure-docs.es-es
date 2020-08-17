---
title: Planes de servicio y cuotas para Azure Spring Cloud
description: Más información sobre las cuotas y los planes de servicio de Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: cda22c63a70c5121e6a6972c66bdc0a4bb5158fc
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089471"
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


Durante el período de versión preliminar, Azure Spring Cloud ofrece solo un nivel de servicio. Cuando alcance un límite, recibirá un error 400 que indica lo siguiente: "Quota exceeds limit for subscription *your subscription* in region *region where your Azure Spring Cloud service is created*" (La cuota supera el límite de la suscripción <su suscripción> en la región <región en la que se ha creado el servicio de Azure Spring Cloud>).

## <a name="next-steps"></a>Pasos siguientes

Algunos límites predeterminados se pueden aumentar. Si su configuración requiere un aumento, [cree una solicitud de soporte técnico](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).
