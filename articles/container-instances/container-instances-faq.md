---
title: 'Azure Container Instances: preguntas más frecuentes'
description: Respuestas a las preguntas más frecuentes relacionadas con el servicio de Azure Container Instances
services: container-instances
author: dkkapur
manager: jeconnoc
ms.service: container-instances
ms.topic: article
ms.date: 4/25/2019
ms.author: dekapur
ms.openlocfilehash: 99882bd23d7b94afc550247172e5b70deb23bec9
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65791394"
---
# <a name="frequently-asked-questions-about-azure-container-instances"></a>Preguntas más frecuentes sobre Azure Container Instances

Este artículo tratan las preguntas más frecuentes sobre Azure Container Instances.

## <a name="deployment"></a>Implementación

### <a name="how-large-can-my-container-image-be"></a>¿Tamaño pueden mi imagen de contenedor?

El tamaño máximo de una imagen de contenedor que se pueden implementar en Azure Container Instances es 15 GB. Es posible que pueda implementar imágenes de mayor tamaño según la disponibilidad en el momento exacta implementa, pero esto no está garantizado.

El tamaño de la imagen de contenedor afecta a cuánto tarda en implementar, por lo general, que desea mantener las imágenes de contenedor más pequeño posible.

### <a name="how-can-i-speed-up-the-deployment-of-my-container"></a>¿Cómo puedo acelerar la implementación de mi contenedor?

Dado que uno de los factores determinantes principales de tiempos de implementación es el tamaño de imagen, busque formas de reducir el tamaño. Quite las capas que no necesite o reducir el tamaño de las capas en la imagen (seleccionando una imagen de sistema operativo base más clara). Por ejemplo, si ejecutas contenedores de Linux, considere el uso Alpine como su imagen base en lugar de un servidor de Ubuntu completo. Asimismo, para contenedores de Windows, use una imagen de base de Nano Server si es posible. 

También debe comprobar la lista de imágenes previamente almacenado en caché en las imágenes de contenedor de Azure, disponible a través de la [lista en caché de imágenes](/rest/api/container-instances/listcachedimages) API. Puede desactivar una capa de imagen para una de las imágenes previamente almacenado en caché. 

Ver más [orientación detallada](container-instances-troubleshooting.md#container-takes-a-long-time-to-start) en reducir el tiempo de inicio del contenedor.

### <a name="what-windows-base-os-images-are-supported"></a>¿Qué imágenes de sistema operativo bases Windows son compatibles?

#### <a name="windows-server-2016-base-images"></a>Imágenes de base de Windows Server 2016

* [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): `10.0.14393.x`, `sac2016`
* [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): `ltsc2016`,  `10.0.14393.x`

> [!NOTE]
> No se admiten imágenes de Windows basadas en canal semianual versión 1709 o 1803.

#### <a name="windows-server-2019-and-client-base-images-preview"></a>Imágenes base 2019 de Windows Server y cliente (versión preliminar)

* [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): `1809`, `10.0.17763.x`
* [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): `ltsc2019`, `1809`, `10.0.17763.x`
* [Windows](https://hub.docker.com/_/microsoft-windows): `1809`, `10.0.17763.x` 

### <a name="what-net-or-net-core-image-layer-should-i-use-in-my-container"></a>¿Qué capas de imagen .NET o .NET Core debo usar en mi contenedor? 

Use la imagen más pequeña que satisface sus requisitos. Para Linux, puede usar un *en tiempo de ejecución alpine* imagen de .NET Core, que ha sido compatible desde el lanzamiento de .NET Core 2.1. Para Windows, si está utilizando .NET Framework completo, a continuación, deberá usar una imagen de Windows Server Core (imagen sólo en tiempo de ejecución, como *4.7.2-windowsservercore-ltsc2016*). Solo en tiempo de ejecución de las imágenes son más pequeñas, pero no admiten las cargas de trabajo que requieren el SDK de .NET.

## <a name="availability-and-quotas"></a>Disponibilidad y las cuotas

### <a name="how-many-cores-and-memory-should-i-allocate-for-my-containers-or-the-container-group"></a>¿Número de núcleos y memoria deben asignarse a los contenedores o el grupo de contenedores?

Esto depende de la carga de trabajo. Empiece poco a poco y probar el rendimiento para ver cómo se hacen los contenedores. [Supervisar el uso de recursos de CPU y memoria](container-instances-monitor.md)y, a continuación, agregue núcleos o más memoria en función del tipo de procesos que se implementan en el contenedor. 

Asegúrese de que también comprobar la [disponibilidad de recursos](container-instances-region-availability.md#availability---general) para la región de implementación para los límites superiores en los núcleos de CPU y memoria disponible para cada grupo de contenedores. 

### <a name="what-underlying-infrastructure-does-aci-run-on"></a>¿Qué infraestructura subyacente ejecuta ACI?

Azure Container Instances pretende ser un servicio de contenedores a petición sin servidor, por lo que queremos que se centra en el desarrollo de los contenedores y no preocuparse por la infraestructura. Para los que tiene curiosidad, o si quiere hacer comparaciones en el rendimiento, ACI se ejecuta en conjuntos de máquinas virtuales de Azure de diferentes SKU, principalmente a partir de la serie D y de la F. Esperamos que esta opción para cambiar en el futuro, puesto que seguimos desarrollar y optimizar el servicio. 

### <a name="i-want-to-deploy-thousand-of-cores-on-aci---can-i-get-my-quota-increased"></a>Deseo implementar miles de núcleos en ACI: ¿puedo aumentar mi cuota?
 
Sí (a veces). Consulte la [cuotas y límites](container-instances-quotas.md) artículo para las cuotas actuales y qué límites se pueden aumentar por solicitud.

### <a name="can-i-deploy-with-more-than-4-cores-and-16-gb-of-ram"></a>¿Puedo implementar con más de 4 núcleos y 16 GB de RAM?

Todavía no. Actualmente, estos son los valores máximos para un grupo de contenedores. Póngase en contacto con soporte técnico de Azure con solicitudes o requisitos específicos. 

### <a name="when-will-aci-be-in-a-specific-region"></a>¿Cuándo estará ACI en una región específica?

Disponibilidad por región actual se publica [aquí](container-instances-region-availability.md#availability---general). Si tiene un requisito para una región específica, póngase en contacto con soporte técnico de Azure.

## <a name="features-and-scenarios"></a>Características y escenarios

### <a name="how-do-i-scale-a-container-group"></a>¿Cómo se puede escalar un grupo de contenedores?

Actualmente, el escalado no está disponible para contenedores o los grupos de contenedores. Si necesita ejecutar varias instancias, usar nuestra API para automatizar y crear más solicitudes de creación del contenedor de grupo para el servicio. 

### <a name="what-features-are-available-to-instances-running-in-a-custom-vnet"></a>¿Qué características están disponibles para las instancias que se ejecutan en una red virtual personalizada?

Puede implementar grupos de contenedores en una red virtual de Azure de su elección y delegar las direcciones IP privadas a los grupos de contenedor para enrutar el tráfico dentro de la red virtual a través de los recursos de Azure. Implementación de un grupo de contenedores en una red virtual está actualmente en versión preliminar y pueden cambiar algunos aspectos de esta característica antes de la disponibilidad general (GA). Consulte [obtener una vista previa de las limitaciones](container-instances-vnet.md#preview-limitations) para obtener información actualizada.

## <a name="pricing"></a>Precios

### <a name="when-does-the-meter-start-running"></a>¿Cuando inicia el medidor de la ejecución?

Duración del grupo de contenedor se calcula desde el momento en que se inician extraer imagen del primer contenedor (para una nueva implementación) o el grupo de contenedores se reinicia (si ya ha implementado), hasta que se detenga el grupo de contenedores. Consulte los detalles en [precios de Container Instances](https://azure.microsoft.com/pricing/details/container-instances/).

### <a name="do-i-stop-being-charged-when-my-containers-are-stopped"></a>¿Se impide que se le cobrará cuando se detienen los contenedores?

Los medidores dejan de ejecutarse cuando se detiene el grupo de contenedores completa. Se está ejecutando un contenedor en el grupo de contenedores, siempre y cuando se mantenga los recursos en caso de que desea que los contenedores para volver a iniciar. 

## <a name="next-steps"></a>Pasos siguientes

* [Obtenga más información](container-instances-overview.md) sobre Azure Container Instances.
* [Solucionar problemas comunes de](container-instances-troubleshooting.md) en Azure Container Instances.