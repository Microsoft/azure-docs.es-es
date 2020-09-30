---
title: Introducción a Azure Spring Cloud
description: Aprenda las características y ventajas de Azure Spring Cloud para implementar y administrar aplicaciones Java Spring en Azure.
author: bmitchell287
ms.service: spring-cloud
ms.topic: overview
ms.date: 09/08/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: f622042c9a2e8cc5bc11c8dc909d02afe38e5c78
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90908355"
---
# <a name="what-is-azure-spring-cloud"></a>¿Qué es Azure Spring Cloud?

Azure Spring Cloud facilita la implementación de aplicaciones de microservicio basadas en Spring Boot en Azure sin necesidad de cambiar el código.  Azure Spring Cloud administra la infraestructura de las aplicaciones de Spring Cloud, con el fin de que los desarrolladores puedan centrarse en el código.  Spring Cloud proporciona administración del ciclo de vida mediante el uso de una supervisión y un diagnóstico completos, administración de la configuración, detección de servicios, integración de CI/CD e implementaciones blue-green, entre otros.

Azure Spring Cloud es compatible con aplicaciones de [Spring Boot](https://spring.io/projects/spring-boot) para Java y de [Steeltoe](https://steeltoe.io/) para ASP.NET Core. Steeltoe se ofrece actualmente como versión preliminar pública. Las ofertas de versión preliminar pública le permiten experimentar con nuevas características antes de su publicación oficial.  Los servicios y las características en versión preliminar pública no están diseñados para su uso en producción.  Para obtener más información, consulte las [preguntas frecuentes](https://azure.microsoft.com/support/faq/) o envía una [solicitud de soporte técnico](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).

Al formar parte del ecosistema de Azure, Azure Spring Cloud facilita el enlace con otros servicios de Azure, como almacenamiento, bases de datos, supervisión, etc.

En esta introducción se describen las siguientes funcionalidades de Azure Spring Cloud:

* Servidor de configuración
* Implementaciones Blue/Green
* Escalado de aplicaciones
* Integración con Azure DevOps
* Supervisión de aplicaciones

## <a name="spring-cloud-config-server"></a>Spring Cloud Config Server

Azure Spring Cloud Config Server ofrece una configuración externalizada de un sistema distribuido con compatibilidad con cliente y servidor.  Azure Spring Cloud Config Server es una ubicación central para administrar las propiedades de la aplicación en todos los entornos. Para más información al respecto, consulte la [Guía de referencia de Spring Cloud Config Server](https://spring.io/projects/spring-cloud-config.md). 

## <a name="bluegreen-deployments"></a>Implementaciones Blue/Green

Azure Spring Cloud admite implementaciones blue/green para liberar y actualizar el código en entornos de producción.  Este patrón de administración de cambios permite a los desarrolladores implementar características y cambios de código con la seguridad de que se realizará una reserva inmediata cuando sea necesario.  Los desarrolladores pueden concentrarse en escribir código con varios entornos de producción para actualizar o revertir los cambios realizados en el código sin interrumpir la aplicación.  Para más información acerca de los entornos de ensayo y las implementaciones blue/green, visite este [artículo de procedimientos](spring-cloud-howto-staging-environment.md).

## <a name="cicd-pipeline-automation"></a>Automatización de canalizaciones de CI/CD

Azure Spring Cloud proporciona integración con Azure DevOps mediante la CLI de Azure.  Azure DevOps permite automatizar la integración e implementación del código en la aplicación Spring.  Para más información, visite [este artículo](spring-cloud-howto-cicd.md).

## <a name="application-scaling"></a>Escalado de aplicaciones

Azure Spring Cloud permite escalar fácilmente los microservicios en el panel de Azure Spring Cloud.  Tanto el número de vCPU como la cantidad de memoria disponible para los microservicios se pueden escalar o reducir verticalmente para ajustarlas a sus necesidades.  El escalado surte efecto en segundos y no requiere cambios en el código ni la reimplementación de los microservicios.  Para más información, complete este [tutorial](spring-cloud-tutorial-scale-manual.md).

## <a name="application-monitoring"></a>Supervisión de aplicaciones

Las herramientas de seguimiento distribuido de Spring Cloud permiten a los desarrolladores depurar y supervisar las complejas interconexiones entre los microservicios de una aplicación.  Mediante la integración de [Spring Cloud Sleuth](https://spring.io/projects/spring-cloud-sleuth) en [Azure Application Insights](../azure-monitor/insights/insights-overview.md) Azure proporciona una eficaz funcionalidad de seguimiento distribuido directamente desde Azure Portal.  Para más información, complete este [tutorial](spring-cloud-tutorial-distributed-tracing.md).

## <a name="next-steps"></a>Pasos siguientes

Para empezar, complete la [guía de inicio rápido de Spring Cloud](spring-cloud-quickstart.md)

Los ejemplos están disponibles en GitHub: [Ejemplos de Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/).
