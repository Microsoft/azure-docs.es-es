---
title: Azure Traffic Manager | Microsoft Docs
description: Este artículo ofrece una introducción sobre Azure Traffic Manager. Averigüe si es la elección correcta para equilibrar la carga de tráfico de usuario de la aplicación.
services: traffic-manager
author: duongau
manager: twooley
ms.service: traffic-manager
customer intent: As an IT admin, I want to learn about Traffic Manager and what I can use it for.
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2019
ms.author: duau
ms.openlocfilehash: 830700fb4a5ac57405877364e9cc4828e5d1a5a4
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89392551"
---
# <a name="what-is-traffic-manager"></a>¿Qué es el Administrador de tráfico?
Azure Traffic Manager es un equilibrador de carga de tráfico basado en DNS que le permite distribuir el tráfico de forma óptima a servicios de regiones de Azure globales, al tiempo que proporciona una alta disponibilidad y capacidad de respuesta.

Traffic Manager usa DNS para dirigir las solicitudes del cliente al punto de conexión de servicio más adecuado en función de un método de enrutamiento del tráfico y el mantenimiento de los puntos de conexión. Un punto de conexión es cualquier servicio accesible desde Internet hospedado dentro o fuera de Azure. Traffic Manager proporciona una serie de [métodos de enrutamiento del tráfico](traffic-manager-routing-methods.md) y [opciones de supervisión del punto de conexión](traffic-manager-monitoring.md) para satisfacer las distintas necesidades de las aplicaciones y los modelos de conmutación automática por error. Traffic Manager es resistente a errores, incluidos los que afecten a toda una región de Azure.

>[!NOTE]
> Azure ofrece un conjunto de soluciones de equilibrio de carga completamente administradas para sus escenarios. Si desea finalización con el protocolo de seguridad de la capa de transporte (TLS) ("descarga SSL") procesamiento de niveles de aplicación por solicitud HTTP/HTTPS, revise [Application Gateway](../application-gateway/application-gateway-introduction.md). Si desea conseguir equilibrio de carga en la región, consulte [Load Balancer](../load-balancer/load-balancer-overview.md). Sus escenarios integrales pueden beneficiarse de la combinación de estas soluciones según sea necesario.
>
> Si desea ver una comparación de las distintas opciones de equilibrio de carga de Azure, consulte [Información general sobre las opciones de equilibrio de carga en Azure](https://docs.microsoft.com/azure/architecture/guide/technology-choices/load-balancing-overview).

Traffic Manager ofrece las siguientes características:

## <a name="increase-application-availability"></a>Aumento de la disponibilidad de la aplicación

Traffic Manager ofrece alta disponibilidad para las aplicaciones más importantes al supervisar los puntos de conexión y proporcionar la conmutación automática por error cuando un punto de conexión deja de funcionar.
    
## <a name="improve-application-performance"></a>Mejora del rendimiento de las aplicaciones

Azure permite ejecutar servicios en la nube en los centros de datos ubicados en todo el mundo. Traffic Manager mejora la capacidad de respuesta de las aplicaciones al dirigir el tráfico al punto de conexión con la menor latencia de red para el cliente.

## <a name="perform-service-maintenance-without-downtime"></a>Realizar el mantenimiento del servicio sin tiempo de inactividad

Puede realizar operaciones de mantenimiento planeado en sus aplicaciones sin tiempo de inactividad. Traffic Manager puede dirigir el tráfico a puntos de conexión alternativos mientras el mantenimiento está en curso.

## <a name="combine-hybrid-applications"></a>Combinación de aplicaciones híbridas

Traffic Manager admite puntos de conexión externos (que no forman parte de Azure) que pueden emplearse en implementaciones locales y de nube híbrida, entre ellos los escenarios de [implementación](https://azure.microsoft.com/overview/what-is-cloud-bursting/), migración y conmutación por error a la nube.

## <a name="distribute-traffic-for-complex-deployments"></a>Distribución del tráfico para implementaciones complejas

Mediante los [perfiles anidados de Traffic Manager](traffic-manager-nested-profiles.md), se pueden combinar varios métodos de enrutamiento del tráfico para crear reglas complejas y flexibles que satisfagan las necesidades de escalado de las implementaciones más grandes y complejas.

## <a name="pricing"></a>Precios

Para obtener información de precios, consulte la página de [precios de Traffic Manager](https://azure.microsoft.com/pricing/details/traffic-manager/).


## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [crear un perfil de Traffic Manager](traffic-manager-create-profile.md).
- Consulte [Cómo funciona Traffic Manager](traffic-manager-how-it-works.md).
- Vea las [preguntas más frecuentes](traffic-manager-FAQs.md) sobre Traffic Manager.




