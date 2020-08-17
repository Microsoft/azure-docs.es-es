---
title: Recuperación ante desastres geográfica de Azure Spring Cloud | Microsoft Docs
description: Más información sobre cómo proteger su aplicación de Spring Cloud ante interrupciones regionales
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 10/24/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 6af23f566e8768dd1213095402b7af029d494476
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87076040"
---
# <a name="azure-spring-cloud-disaster-recovery"></a>Recuperación ante desastres de Azure Spring Cloud

En este artículo se explican algunas estrategias que puede usar para proteger las aplicaciones de Azure Spring Cloud contra tiempos de inactividad.  Cualquier centro de datos o región puede experimentar tiempos de inactividad causados por desastres regionales, pero una planeación cuidadosa puede mitigar el impacto en los clientes.

## <a name="plan-your-application-deployment"></a>Planeamiento de la implementación de la aplicación

Las aplicaciones de Azure Spring Cloud se ejecutan en una región específica.  Azure funciona en varias ubicaciones geográficas del mundo. Una ubicación geográfica de Azure es un área definida del mundo que contiene al menos una región de Azure. Una región de Azure es un área dentro de una ubicación geográfica que contiene uno o varios centros de datos.  Cada región de Azure se empareja con otra región de la misma zona geográfica, que juntas forman un emparejamiento regional. Azure serializa las actualizaciones de plataforma (mantenimiento planeado) entre pares regionales, lo que garantiza que solo se actualice una región de cada par cada vez. Si se produce una interrupción que afecte a varias regiones, se dará prioridad a la recuperación de al menos una región de cada par.

Para garantizar la alta disponibilidad y la protección ante desastres, tiene que implementar las aplicaciones de Spring Cloud en varias regiones.  Azure proporciona una lista de [regiones emparejadas](../best-practices-availability-paired-regions.md) para que se puedan planear las implementaciones de Spring Cloud en pares regionales.  Se recomienda tener en cuenta tres factores clave al diseñar la arquitectura de microservicio: disponibilidad en regiones, regiones emparejadas de Azure y disponibilidad del servicio.

*  Disponibilidad en regiones:  Elija un área geográfica cercana a sus usuarios para minimizar el retraso de la red y el tiempo de transmisión.
*  Regiones emparejadas de Azure:  Elija regiones emparejadas dentro del área geográfica elegida para garantizar las actualizaciones coordinadas de la plataforma y los esfuerzos de recuperación prioritarios si es necesario.
*  Disponibilidad del servicio:   Decida si los niveles de acceso de las regiones emparejadas deben ser frecuente/frecuente, frecuente/normal o frecuente/poco frecuente.

## <a name="use-azure-traffic-manager-to-route-traffic"></a>Uso de Azure Traffic Manager para enrutar el tráfico

[Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) ofrece un equilibrio de carga de tráfico basado en DNS y puede distribuir el tráfico de red entre varias regiones.  Use Azure Traffic Manager para dirigir a los clientes a la instancia del servicio Azure Spring Cloud más cercana.  Para conseguir los máximos niveles de rendimiento y redundancia, dirija todo el tráfico de aplicaciones a través de Azure Traffic Manager antes de enviarlo al servicio Azure Spring Cloud.

Si tiene aplicaciones de Azure Spring Cloud en varias regiones, use Azure Traffic Manager para controlar el flujo de tráfico a las aplicaciones en cada región.  Defina un punto de conexión de Azure Traffic Manager para cada servicio con la dirección IP del servicio. Los clientes deben conectarse a un nombre DNS de Azure Traffic Manager que apunte al servicio Azure Spring Cloud.  Azure Traffic Manager equilibra el tráfico a través de los puntos de conexión definidos.  Si se produce un desastre en un centro de datos, Azure Traffic Manager dirigirá el tráfico de esa región a su par, lo que garantiza la continuidad del servicio.

## <a name="create-azure-traffic-manager-for-azure-spring-cloud"></a>Creación de Azure Traffic Manager para Azure Spring Cloud

1. Cree Azure Spring Cloud en dos regiones diferentes.
Necesitará dos instancias de servicio de Azure Spring Cloud implementadas en dos regiones diferentes (Este de EE. UU. y Oeste de Europa). Inicie una aplicación de Azure Spring Cloud existente con Azure Portal para crear dos instancias de servicio. Cada una de ellas servirá como los puntos de conexión principal y de conmutación por error de Traffic. 

**Información de las dos instancias de servicio:**

| Nombre de servicio | Location | Application |
|--|--|--|
| service-sample-a | Este de EE. UU. | gateway / auth-service / account-service |
| service-sample-b | Oeste de Europa | gateway / auth-service / account-service |

2. Configure el dominio personalizado para el servicio siguiendo lo indicado en el [documento de dominio personalizado](spring-cloud-tutorial-custom-domain.md) para configurar el dominio personalizado de estas dos instancias de servicio existentes. Después de que la configuración se complete correctamente, ambas instancias estarán enlazadas al dominio personalizado: bcdr-test.contoso.com

3. Cree un administrador de tráfico y dos puntos de conexión: [Creación de un perfil de Traffic Manager mediante Azure Portal](https://docs.microsoft.com/azure/traffic-manager/quickstart-create-traffic-manager-profile).

Este es el perfil de Traffic Manager:
* Nombre DNS de Traffic Manager: `http://asc-bcdr.trafficmanager.net`
* Perfiles de punto de conexión: 

| Perfil | Tipo | Destino | Priority | Configuración del encabezado personalizado |
|--|--|--|--|--|
| Perfil de punto de conexión A | Punto de conexión externo | service-sample-a.asc-test.net | 1 | host: bcdr-test.contoso.com |
| Perfil de extremo B | Punto de conexión externo | service-sample-b.asc-test.net | 2 | host: bcdr-test.contoso.com |

4. Cree un registro CNAME en la zona DNS: bcdr-test.contoso.com CNAME asc-bcdr.trafficmanager.net. 

5. El entorno ahora está completamente configurado. Los clientes deben poder acceder a la aplicación a través de: bcdr-test.contoso.com