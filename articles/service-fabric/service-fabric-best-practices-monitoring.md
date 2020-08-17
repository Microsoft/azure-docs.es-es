---
title: Procedimientos recomendados para la supervisión de Azure Service Fabric
description: Procedimientos recomendados y consideraciones de diseño para supervisar los clústeres y las aplicaciones mediante Azure Service Fabric.
author: peterpogorski
ms.topic: conceptual
ms.date: 01/23/2019
ms.author: pepogors
ms.openlocfilehash: a7b1c1b3fc3196557b862c488ee01af8b8e1f04f
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86529257"
---
# <a name="monitoring-and-diagnostic-best-practices-for-azure-service-fabric"></a>Procedimientos recomendados de supervisión y diagnóstico para Azure Service Fabric

La [supervisión y el diagnóstico](./service-fabric-diagnostics-overview.md) son fundamentales para el desarrollo, las pruebas y la implementación de cargas de trabajo en cualquier entorno de la nube. Por ejemplo, puede realizar un seguimiento de cómo se usan las aplicaciones, de las acciones que realiza la plataforma Service Fabric, del uso de recursos con los contadores de rendimiento y del estado general del clúster. Puede utilizar esta información para diagnosticar y corregir cualquier problema, así como para prevenir que se repita en el futuro.

## <a name="application-monitoring"></a>Supervisión de aplicaciones

La supervisión de aplicaciones realiza un seguimiento del uso de las características y componentes de una aplicación. Supervise sus aplicaciones para asegurarse de que detecta los problemas que afectan a sus usuarios. La supervisión de las aplicaciones es responsabilidad de quienes desarrollan la aplicación y sus servicios, ya que es única para la lógica de negocios de la aplicación. Se recomienda configurar la supervisión de aplicaciones con [Application Insights](./service-fabric-tutorial-monitoring-aspnet.md), la herramienta de supervisión de aplicaciones de Azure.

## <a name="cluster-monitoring"></a>Supervisión de clústeres

Uno de los objetivos de Service Fabric es que las aplicaciones sean resistentes a los errores de hardware. Esto se logra gracias a la capacidad que tienen los servicios del sistema de la plataforma de detectar problemas de infraestructura y conmutar por error las cargas de trabajo rápidamente a otros nodos del clúster. Pero, ¿qué ocurre si son los propios servicios del sistema quienes tienen problemas? ¿O, si al intentar implementar o mover una carga de trabajo, se infringen las reglas de ubicación de los servicios? Service Fabric proporciona diagnóstico para estos y otros problemas, lo que le pide tener la certeza de que está informado acerca de cómo interactúa la plataforma de Service Fabric con sus aplicaciones, servicios, contenedores y nodos.

En el caso de los clústeres de Windows, se recomienda que configure la supervisión de clústeres con el [agente de Diagnostics](./service-fabric-diagnostics-event-aggregation-wad.md) y [registros de Azure Monitor](./service-fabric-diagnostics-oms-setup.md).

En el caso de los clústeres de Linux, los registros de Azure Monitor son también la herramienta recomendada para la supervisión de la infraestructura y plataforma de Azure. Los diagnósticos de la plataforma Linux requieren otra configuración, como se indicó en [Eventos de clúster Linux de Service Fabric en Syslog](./service-fabric-diagnostics-oms-syslog.md).

## <a name="infrastructure-monitoring"></a>Supervisión de infraestructura

Los [registros de Azure Monitor](./service-fabric-diagnostics-oms-agent.md) se recomiendan para la supervisión de eventos de nivel de clúster. Una vez que configure el agente de Log Analytics con su área de trabajo como se describe en el vínculo anterior, podrá recopilar métricas de rendimiento como el uso de la CPU, contadores de rendimiento de .NET como el uso de la CPU a nivel de proceso, contadores de rendimiento de Service Fabric contadores como el número de excepciones de un servicio confiable y métricas de contenedor como el uso de la CPU.  Tendrá que escribir registros de contenedor en stdout o stderr, con el fin de estén disponibles en los registros de Azure Monitor.

## <a name="watchdogs"></a>Guardianes

Por lo general, un guardián es un servicio independiente que vigila el mantenimiento y la carga de los servicios, hace ping a los puntos de conexión e informa acerca de los eventos de mantenimiento inesperados del clúster. Esto puede ayudar a evitar errores que se podrían pasar pasan por alto si solo se tuviera en cuenta el rendimiento de un servicio. Los guardianes son también un buen lugar para hospedar código que realiza acciones de subsanación que no requieran interacción del usuario, como limpiar los archivos de registro de almacenamiento a determinados intervalos de tiempo. Vea la implementación de un servicio guardián de ejemplo en [Eventos de clúster Linux de Service Fabric en Syslog](https://github.com/Azure-Samples/service-fabric-watchdog-service).

## <a name="next-steps"></a>Pasos siguientes

* Empiece a instrumentar las aplicaciones: [Generación de eventos y registros de nivel de aplicación](service-fabric-diagnostics-event-generation-app.md).
* Consulte los pasos necesarios para configurar Application Insights para una aplicación en [Supervisión y diagnóstico de una aplicación de ASP.NET Core de Service Fabric](service-fabric-tutorial-monitoring-aspnet.md).
* Obtenga más información acerca de la supervisión de la plataforma y los eventos que proporciona Service Fabric: [Generación de eventos y registros de nivel de plataforma](service-fabric-diagnostics-event-generation-infra.md).
* Configure la integración de los registros de Azure Monitor con Service Fabric: [Configurar registros de Azure Monitor para un clúster](service-fabric-diagnostics-oms-setup.md)
* Aprenda a configurar registros de Azure Monitor para supervisar contenedores: [Supervisión de contenedores Windows en Service Fabric con Log Analytics](service-fabric-tutorial-monitoring-wincontainers.md).
* Vea un ejemplo de problemas en el diagnóstico y sus soluciones con Service Fabric: [diagnóstico de escenarios comunes](service-fabric-diagnostics-common-scenarios.md)
* Más información acerca de las recomendaciones generales de supervisión para los recursos de Azure: [Procedimientos recomendados: supervisión y diagnóstico](/azure/architecture/best-practices/monitoring).
