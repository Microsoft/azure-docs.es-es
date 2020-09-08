---
title: Introducción a los conjuntos de escalado de máquinas virtuales de Azure
description: Información acerca de los conjuntos de escalado de máquinas virtuales de Azure y de cómo escalar automáticamente sus aplicaciones
author: mimckitt
ms.author: mimckitt
ms.topic: overview
ms.service: virtual-machine-scale-sets
ms.subservice: ''
ms.date: 06/30/2020
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: f381afa69d1a445d2bdb14dd663341e4a18f8cd9
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88648673"
---
# <a name="what-are-virtual-machine-scale-sets"></a>¿Qué son los conjuntos de escalado de máquina virtual?
Los conjuntos de escalado de máquinas virtuales de Azure permiten crear y administrar un grupo de máquinas virtuales con equilibrio de carga. El número de instancias de máquina virtual puede aumentar o disminuir automáticamente según la demanda, o de acuerdo a una programación definida. Los conjuntos de escalado proporcionan una alta disponibilidad a las aplicaciones y le permiten administrar, configurar y actualizar de forma centralizada un gran número de máquinas virtuales. Con los conjuntos de escalado de máquinas virtuales, puede crear servicios a gran escala para áreas como proceso, macrodatos y cargas de trabajo de contenedor.


## <a name="why-use-virtual-machine-scale-sets"></a>¿Por qué utilizar conjuntos de escalado de máquinas virtuales?
Para proporcionar redundancia y mejorar el rendimiento las aplicaciones normalmente se distribuyen entre varias instancias. Los clientes pueden acceder a la aplicación mediante un equilibrador de carga que distribuye las solicitudes a una de las instancias de la aplicación. Si necesita realizar tareas de mantenimiento o actualizar una instancia de la aplicación, los clientes tienen que distribuirse a otra instancia de aplicación que esté disponible. Para hacer frente a la demanda adicional de los clientes, puede que tenga que aumentar el número de instancias de aplicación en las que se ejecuta la aplicación.

Los conjuntos de escalado de máquinas virtuales de Azure proporcionan las capacidades de administración para las aplicaciones que se ejecutan en varias máquinas virtuales, [el escalado automático de recursos](virtual-machine-scale-sets-autoscale-overview.md)y el equilibrio de carga de tráfico. Los conjuntos de escalado ofrecen las siguientes ventajas principales:

- **Facilitan la creación y administración de varias máquinas virtuales**
    - Cuando tiene muchas máquinas virtuales que ejecutan la aplicación es importante mantener una configuración coherente en todo el entorno. Para obtener un rendimiento confiable de la aplicación, el tamaño de máquina virtual, la configuración de disco y las instalaciones de la aplicación deben coincidir en todas las máquinas virtuales.
    - Con los conjuntos de escalado se crean todas las instancias de máquina virtual desde la misma configuración e imagen base de sistema operativo. Este enfoque le permite administrar fácilmente cientos de máquinas virtuales sin tareas adicionales de configuración o administración de red.
    - Los conjuntos de escalado admiten el uso de [Azure Load Balancer](../load-balancer/load-balancer-overview.md) para la distribución del tráfico de nivel 4 básico, y de [Azure Application Gateway](../application-gateway/overview.md) para las características más avanzadas de distribución de nivel 7 y terminación TLS.

- **Proporcionan alta disponibilidad y resistencia de aplicación**
    - Los conjuntos de escalado se utilizan para ejecutar varias instancias de la aplicación. Si una de estas instancias de máquina virtual tiene un problema, los clientes siguen teniendo acceso a la aplicación a través de una de las otras instancias de máquina virtual con una interrupción mínima.
    - Con el fin de tener mayor disponibilidad, puede usar [zonas de disponibilidad](../availability-zones/az-overview.md) para distribuir automáticamente las instancias de máquina virtual en un conjunto de escalado dentro de un único centro de datos o en varios centros de datos.

- **Permiten a la aplicación escalar automáticamente a medida que cambia la demanda de recursos**
    - La demanda de la aplicación por parte de los clientes puede cambiar a lo largo del día o de la semana. Para cumplir con la demanda del cliente, los conjuntos de escalado pueden aumentar o disminuir automáticamente el número de instancias de máquina virtual a medida que aumenta o disminuye la demanda de la aplicación.
    - El escalado automático reduce el número de instancias innecesarias de máquina virtual que ejecutan la aplicación cuando la demanda es baja, mientras que cuando la demanda aumenta, los clientes continúan recibiendo un nivel aceptable de rendimiento gracias a la incorporación automática de instancias adicionales de máquina virtual. Esta capacidad le ayuda a reducir los costos y a crear de forma eficaz los recursos de Azure según sea necesario.

- **Funciona a gran escala**
    - Los conjuntos de escalado admiten hasta 1000 instancias de máquina virtual. Si crea y carga sus propias imágenes de máquina virtual personalizadas, el límite es 600 instancias.
    - Para el mejor rendimiento con cargas de trabajo de producción, use [Azure Managed Disks](../virtual-machines/managed-disks-overview.md).


## <a name="differences-between-virtual-machines-and-scale-sets"></a>Diferencias entre las máquinas virtuales y los conjuntos de escalado
Los conjuntos de escalado se generan a partir de máquinas virtuales. Con los conjuntos de escalado las capas de administración y automatización sirven para ejecutar y escalar sus aplicaciones. En su lugar podría crear y administrar manualmente máquinas virtuales individuales, o integrar herramientas existentes para crear un nivel similar de automatización. En la tabla siguiente se describen las ventajas de los conjuntos de escalado en comparación con la administración manual de varias instancias de máquina virtual.

| Escenario                           | Grupo manual de máquinas virtuales                                                                    | Conjunto de escalado de máquina virtual |
|------------------------------------|----------------------------------------------------------------------------------------|---------------------------|
| Incorporación de instancias adicionales de máquina virtual        | Proceso manual para crear, configurar y garantizar el cumplimiento                             | Creación automática a partir de la configuración central |
| Equilibrio y distribución del tráfico | Proceso manual para crear y configurar una instancia de Azure Load Balancer o Application Gateway      | Posibilidad de crear e integrar automáticamente con Azure Load Balancer o Application Gateway |
| Alta disponibilidad y redundancia   | Creación manual de conjuntos de disponibilidad o distribución y seguimiento manual de las máquinas virtuales en las distintas zonas de disponibilidad | Distribución automática de instancias de máquina virtual a través de las zonas de disponibilidad o conjuntos de disponibilidad |
| Escalado de máquinas virtuales                     | Supervisión manual y Azure Automation                                                 | Escalado automático basado en las métricas de host, las métricas de invitado, Application Insights o programación |

No hay costo adicional por usar conjuntos de escalado. Solo se paga por los recursos de proceso subyacente como las instancias de máquina virtual, el equilibrador de carga o el almacenamiento de disco administrado. Las características de administración y automatización, como el escalado automático y la redundancia, no incurren en cargos adicionales sobre el uso de máquinas virtuales.

## <a name="how-to-monitor-your-scale-sets"></a>Cómo supervisar los conjuntos de escalado

Use [Azure Monitor para VM](../azure-monitor/insights/vminsights-overview.md), que tiene un proceso de incorporación sencillo y automatiza la recopilación de contadores de rendimiento de CPU, memoria, disco y red importantes de las máquinas virtuales en el conjunto de escalado. También incluye funcionalidades de supervisión adicionales y visualizaciones predefinidas que le ayudan a centrarse en la disponibilidad y el rendimiento de los conjuntos de escalado.

Habilite la supervisión de la [aplicación del conjunto de escalado de máquinas virtuales](../azure-monitor/app/azure-vm-vmss-apps.md) con Application Insights para recopilar la información detallada acerca de la aplicación, como vistas de página, solicitudes de aplicación y excepciones. Compruebe de forma más exhaustiva la disponibilidad de la aplicación configurando una [prueba de disponibilidad](../azure-monitor/app/monitor-web-app-availability.md) para simular el tráfico de usuarios.

## <a name="singapore-data-residency"></a>Residencia de datos en Singapur

En Azure, la característica que permite almacenar los datos de clientes en una sola región solo está disponible actualmente en la región de Sudeste Asiático (Singapur) de la geoárea Asia Pacífico. En todas las demás regiones, los datos del cliente se almacenan en la geoárea. Para más información, consulte el [Centro de confianza](https://azuredatacentermap.azurewebsites.net/).

## <a name="next-steps"></a>Pasos siguientes
Para empezar cree su primer conjunto de escalado de máquinas virtuales en Azure Portal.

> [!div class="nextstepaction"]
> [Creación de un conjunto de escalado en Azure Portal](quick-create-portal.md)
