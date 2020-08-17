---
title: Preguntas más frecuentes
description: Respuestas a las preguntas más frecuentes relacionadas con el servicio Azure Container Instances
author: dkkapur
ms.topic: article
ms.date: 06/02/2020
ms.openlocfilehash: 95190496df4037a973207e14292911225094782e
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421287"
---
# <a name="frequently-asked-questions-about-azure-container-instances"></a>Preguntas más frecuentes sobre Azure Container Instances

En este artículo se abordan las preguntas más frecuentes sobre Azure Container Instances.

## <a name="deployment"></a>Implementación

### <a name="how-large-can-my-container-image-be"></a>¿Qué tamaño puede tener mi imagen de contenedor?

El tamaño máximo de una imagen de contenedor implementable en Azure Container Instances es de 15 GB. Es posible que pueda implementar imágenes de mayor tamaño según la disponibilidad exacta en el momento de la implementación, pero no es seguro.

El tamaño de la imagen de contenedor afecta al tiempo que tarda en implementarse, por lo que, generalmente, le interesará que las imágenes de contenedor sean lo más pequeñas posible.

### <a name="how-can-i-speed-up-the-deployment-of-my-container"></a>¿Cómo puedo acelerar la implementación de mi contenedor?

Dado que uno de los principales factores determinantes en el tiempo de implementación es el tamaño de imagen, busque formas de reducir el tamaño. Quite las capas que no necesite o reduzca el tamaño de las capas de la imagen (seleccionando una imagen de sistema operativo de base menos pesada). Por ejemplo, si ejecuta contenedores de Linux, considere la posibilidad de usar Alpine como imagen de base en lugar de un servidor de Ubuntu completo. Asimismo, para contenedores de Windows, use una imagen de base de Nano Server si es posible. 

También debe comprobar la lista de imágenes previamente almacenadas en caché en Azure Container Images, disponible a través de la API [List Cached Images](/rest/api/container-instances/location/listcachedimages). Es posible que pueda desactivar una capa de imagen de una de las imágenes previamente almacenadas en caché. 

Consulte una [guía más detallada](container-instances-troubleshooting.md#container-takes-a-long-time-to-start) sobre cómo reducir el tiempo de inicio del contenedor.

### <a name="what-windows-base-os-images-are-supported"></a>¿Qué imágenes de sistema operativo de base de Windows se admiten?

> [!NOTE]
> Debido a problemas de compatibilidad con versiones anteriores tras las actualizaciones de Windows en 2020, las siguientes versiones de imagen incluyen el número de versión mínimo que se recomienda usar en la imagen de base. Las implementaciones actuales que usan versiones de imagen anteriores no se ven afectadas, pero las nuevas implementaciones deben adaptarse a las siguientes imágenes de base. 

#### <a name="windows-server-2016-base-images"></a>Imágenes de base de Windows Server 2016

* [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): `sac2016`, `10.0.14393.3506` o más reciente
* [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): `ltsc2016`, `10.0.14393.3506` o más reciente

> [!NOTE]
> No se admiten las imágenes de Windows basadas en las versiones de Canal semianual 1709 o 1803.

#### <a name="windows-server-2019-and-client-base-images-preview"></a>Imágenes de base de Windows Server 2019 y cliente (versión preliminar)

* [Nano Server](https://hub.docker.com/_/microsoft-windows-nanoserver): `1809`, `10.0.17763.1040` o más reciente
* [Windows Server Core](https://hub.docker.com/_/microsoft-windows-servercore): `ltsc2019`, `1809`, `10.0.17763.1040` o más reciente
* [Windows](https://hub.docker.com/_/microsoft-windows): `1809`, `10.0.17763.1040` o más reciente

### <a name="what-net-or-net-core-image-layer-should-i-use-in-my-container"></a>¿Qué capas de imagen .NET o .NET Core debo usar en mi contenedor? 

Use la imagen más pequeña que satisfaga sus requisitos. Para Linux, puede usar una imagen *runtime-alpine* de .NET Core, que se admite desde el lanzamiento de .NET Core 2.1. Para Windows, si está utilizando .NET Framework completo, deberá usar una imagen de Windows Server Core (imagen runtime-only, como, por ejemplo, *4.7.2-windowsservercore-ltsc2016*). Las imágenes runtime-only son más pequeñas pero no admiten cargas de trabajo que requieran el SDK de .NET.

## <a name="availability-and-quotas"></a>Disponibilidad y cuotas

### <a name="how-many-cores-and-memory-should-i-allocate-for-my-containers-or-the-container-group"></a>¿Cuántos núcleos y qué memoria debo asignar para mis contenedores o para el grupo de contenedores?

Realmente, esto depende de su carga de trabajo. Empiece poco a poco y pruebe el rendimiento para ver cómo funcionan los contenedores. [Supervise el uso de recursos de CPU y memoria](container-instances-monitor.md) y, a continuación, agregue núcleos o memoria en función del tipo de procesos que implementa en el contenedor.

Asegúrese de comprobar también la [disponibilidad de recursos](container-instances-region-availability.md#availability---general) para la región de implementación para los límites superiores en los núcleos de CPU y memoria disponibles para cada grupo de contenedores. 

> [!NOTE]
> La infraestructura subyacente al servicio utiliza una pequeña cantidad de los recursos de un grupo de contenedores. Los contenedores pueden acceder a la mayoría de los recursos asignados al grupo, pero no a todos. Por este motivo, debe planear un pequeño búfer de recursos al solicitar los recursos para los contenedores del grupo.

### <a name="what-underlying-infrastructure-does-aci-run-on"></a>¿En qué infraestructura subyacente se ejecuta ACI?

Azure Container Instances pretende ser un servicio de contenedores a petición sin servidor, por lo que queremos que se centre en el desarrollo de los contenedores y no en la infraestructura. Para aquellos que tengan curiosidad o quieran hacer comparaciones de rendimiento, ACI se ejecuta en conjuntos de máquinas virtuales de Azure de varios SKU, principalmente de las series F y D. Esperamos que esto cambie en el futuro, ya que seguimos desarrollando y optimizando el servicio. 

### <a name="i-want-to-deploy-thousand-of-cores-on-aci---can-i-get-my-quota-increased"></a>Deseo implementar miles de núcleos en ACI: ¿puedo aumentar mi cuota?
 
Sí (a veces). Consulte el artículo sobre [cuotas y límites](container-instances-quotas.md) para conocer las cuotas actuales y qué límites se pueden aumentar por solicitud.

### <a name="can-i-deploy-with-more-than-4-cores-and-16-gb-of-ram"></a>¿Puedo implementar con más de 4 núcleos y 16 GB de RAM?

Todavía no. Actualmente, estos son los valores máximos para un grupo de contenedores. Póngase en contacto con el soporte técnico de Azure para trasladarle solicitudes o requisitos específicos. 

### <a name="when-will-aci-be-in-a-specific-region"></a>¿Cuándo estará ACI en una región específica?

La disponibilidad regional actual se encuentra publicada [aquí](container-instances-region-availability.md#availability---general). Si tiene un requisito para una región específica, póngase en contacto con el soporte técnico de Azure.

## <a name="features-and-scenarios"></a>Características y escenarios

### <a name="how-do-i-scale-a-container-group"></a>¿Cómo se puede escalar un grupo de contenedores?

Actualmente, el escalado no está disponible para contenedores o grupos de contenedores. Si necesita ejecutar más instancias, use nuestra API para automatizar y crear más solicitudes de creación de grupos de contenedores en el servicio. 

### <a name="what-features-are-available-to-instances-running-in-a-custom-vnet"></a>¿Qué características están disponibles para las instancias que se ejecutan en una red virtual personalizada?

Puede [implementar grupos de contenedores en una red virtual de Azure](container-instances-vnet.md) que elija y delegar las direcciones IP privadas a los grupos de contenedores para redirigir el tráfico dentro de la red virtual a través de los recursos de Azure. Para conocer los escenarios y las limitaciones de red con Azure Container Instances, consulte [Escenarios y recursos de redes virtuales](container-instances-virtual-network-concepts.md).

## <a name="pricing"></a>Precios

### <a name="when-does-the-meter-start-running"></a>¿Cuándo empieza a ejecutarse el medidor?

La duración del grupo de contenedores se calcula desde el momento en que empezamos a extraer la imagen del primer contenedor (para una implementación nueva) o se reinicia el grupo de contenedores (si ya se ha implementado), y hasta que se detenga el grupo de contenedores. Consulte los detalles en [Precios de Container Instances](https://azure.microsoft.com/pricing/details/container-instances/).

### <a name="do-i-stop-being-charged-when-my-containers-are-stopped"></a>¿Dejo de recibir cargos cuando se detienen los contenedores?

Los medidores dejan de ejecutarse cuando se detiene el grupo de contenedores completo. Mientras que un contenedor del grupo de contenedores se esté ejecutando, mantenemos los recursos por si desea iniciar los contenedores de nuevo. 

## <a name="next-steps"></a>Pasos siguientes

* [Más información](container-instances-overview.md) sobre Azure Container Instances.
* [Solución de problemas habituales](container-instances-troubleshooting.md) de Azure Container Instances.
