---
title: Información general de Azure Service Fabric Mesh
description: Conozca Azure Service Fabric Mesh. Con Service Fabric Mesh, puede implementar y escalar las aplicaciones sin preocuparse por las necesidades de infraestructura de estas.
author: dkkapur
ms.author: dekapur
ms.date: 10/1/2018
ms.topic: overview
ms.openlocfilehash: b7061a178a3f5a9d126afab3fc1f9aa77d5cc050
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87927035"
---
# <a name="what-is-service-fabric-mesh"></a>¿Qué es Service Fabric Mesh?

Este vídeo ofrece una introducción rápida sobre Service Fabric Mesh.
> [!VIDEO https://www.youtube.com/embed/7qWeVGzAid0]

Azure Service Fabric mesh es un servicio totalmente administrado que permite a los desarrolladores implementar aplicaciones de microservicios sin la administración de máquinas virtuales, almacenamiento o redes. Las aplicaciones hospedadas en Service Fabric Mesh se ejecutan y escalan sin preocuparse por la infraestructura que las sustenta.  Service Fabric Mesh consta de clústeres de miles de máquinas.  Todas las operaciones de clúster están ocultas para el desarrollador. Cargue el código y especifique los recursos necesarios, los requisitos de disponibilidad y los límites de recursos.  Service Fabric Mesh asigna automáticamente la infraestructura y se ocupa de los errores de dicha infraestructura, lo que garantiza que las aplicaciones presentan una elevada disponibilidad. Solo es necesario preocuparse por el mantenimiento y la capacidad de respuesta de la aplicación, no de la infraestructura.  

[!INCLUDE [preview note](./includes/include-preview-note.md)]

En este artículo se proporciona una introducción a las ventajas principales de Service Fabric Mesh.

## <a name="great-developer-experience"></a>Grata experiencia para el desarrollador

Service Fabric Mesh admite todos los lenguajes de programación o plataformas que se puedan ejecutar en un contenedor. Las herramientas de Visual Studio 2019 y Visual Studio Code ofrecen una poderosa experiencia de edición y depuración para las aplicaciones .NET y .NET Core. 

Con Service Fabric Mesh, puede:

- "Levantar y mover" aplicaciones existentes en contenedores para modernizar y ejecutar las aplicaciones actuales a escala.
- Compilar e implementar nuevas aplicaciones de microservicios a escala en Azure.  Realizar la integración con otros servicios o aplicaciones existentes de Azure que se ejecutan en contenedores. Cada microservicio es parte de una aplicación de red aislada y segura. El microservicio tiene políticas de gobernanza de recursos definidas para núcleos de CPU, memoria, espacio en disco y mucho más.
- Realizar la integración con aplicaciones existentes y ampliarlas sin efectuar cambios en esas aplicaciones. Usar su propia red virtual para conectar las aplicaciones existentes a las nuevas aplicaciones.  
- Modernizar las aplicaciones existentes de Cloud Services mediante la migración a Service Fabric Mesh.  

## <a name="simple-operational-lifecycle"></a>Ciclo de vida operativo sencillo

Administre fácilmente la ejecución, supervisión y depuración de aplicaciones en entornos de producción. Esta administración incluye las actualizaciones de aplicaciones y el control de versiones. Estas aplicaciones pueden constar de un único microservicio o de varios de estos aislados dentro de su propia red. Las aplicaciones se ejecutan de forma eficaz con tiempos rápidos de implementación, colocación y conmutación por error.

Con Service Fabric Mesh, puede:

- Implementar y administrar aplicaciones sin necesidad de aprovisionar y administrar la infraestructura de forma explícita.  Service Fabric Mesh aprovisiona, actualiza y mantiene la infraestructura subyacente, y le aplica las revisiones necesarias, sin necesidad de que intervenga el usuario.
- Configurar la integración continua mediante las herramientas integradas para empaquetar e implementar fácilmente las aplicaciones.
- Aproveche todas las características de los recursos de Azure Resource Manager. Ejemplos de estas características son el registro de auditoría y el [control de acceso basado en roles de Azure (Azure RBAC)](../role-based-access-control/overview.md). Todos los recursos que se implementan en el servicio Service Fabric Mesh de Azure son recursos de Azure Resource Manager. Estos recursos incluyen aplicaciones, servicios, secretos, etc.
- Implementar y administrar recursos mediante [Azure Portal](https://portal.azure.com), plantillas de Resource Manager o bibliotecas de la CLI de Azure o PowerShell.
- Configurar la supervisión y alerta operativas con [Application Insights](/azure/application-insights/) (o la herramienta que prefiera) para capturar los seguimientos de diagnóstico y operativos desde la plataforma.
- Acceder a la información de diagnóstico de las aplicaciones emitida desde el modelo de aplicación mediante [Application Insights](/azure/application-insights/) o la herramienta de su elección.
- Optimizar el uso de recursos mediante la especificación de reglas de escalado automático para los servicios en la definición de aplicación.

## <a name="mission-critical-platform-capabilities"></a>Funcionalidades críticas de la plataforma

Service Fabric Mesh crea una colección de clústeres que abarcan [Azure Availability Zones](../availability-zones/az-overview.md) o límites regionales geopolíticos. En Service Fabric Mesh se describen las aplicaciones con un conjunto de intenciones como escala, requisitos de hardware, requisitos de durabilidad y directivas de seguridad.  Cuando se implementa la aplicación, Service Fabric Mesh busca el lugar óptimo para ejecutarla.

Con Service Fabric Mesh, puede:

- Aprovechar las ventajas de alta disponibilidad, escalabilidad o reducción horizontales, capacidad de detección, orquestación, enrutamiento de mensajes, mensajería confiable, actualizaciones sin tiempo de inactividad, administración de seguridad y secretos, recuperación ante desastres, administración del estado, administración de configuración y transacciones distribuidas.
- Elegir entre varios modelos de aplicación al crear aplicaciones.
- Usar funcionalidades de plataforma expuestas mediante puntos de conexión de REST por medio del consumo de enlaces específicos del idioma generados mediante Swagger.
- Implementar aplicaciones en [Availability Zones](../availability-zones/az-overview.md) y varias regiones para la confiabilidad geográfica.
- Usar todas las características de seguridad y cumplimiento que proporciona Azure.

## <a name="next-steps"></a>Pasos siguientes

Para implementar un proyecto de ejemplo con Visual Studio solo son necesarios unos cuantos pasos. Para más información, consulte [Creación de un sitio web de ASP.NET Core](service-fabric-mesh-quickstart-dotnet-core.md). 

Obtenga respuestas a [preguntas comunes](service-fabric-mesh-faq.md).


<!-- Links -->

[service-fabric-overview]: ../service-fabric/service-fabric-overview.md
