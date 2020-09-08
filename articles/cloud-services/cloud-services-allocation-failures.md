---
title: Solución de errores de asignación de servicio en la nube | Microsoft Docs
description: Solucione un error de asignación al implementar Azure Cloud Services. Obtenga información sobre cómo funciona la asignación y por qué se pueden producir errores en la asignación.
services: azure-service-management, cloud-services
documentationcenter: ''
author: simonxjx
manager: dcscontentpm
editor: ''
tags: top-support-issue
ms.assetid: 529157eb-e4a1-4388-aa2b-09e8b923af74
ms.service: cloud-services
ms.workload: na
ms.tgt_pltfrm: ibiza
ms.topic: troubleshooting
ms.date: 06/15/2018
ms.author: v-six
ms.openlocfilehash: cba3f47015072f16112ef981d2f59d0c73cb69c2
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88142492"
---
# <a name="troubleshooting-allocation-failure-when-you-deploy-cloud-services-in-azure"></a>Solución de errores de asignación al implementar Cloud Services de Azure
## <a name="summary"></a>Resumen
Al implementar instancias en un servicio en la nube o agregar nuevas instancias de rol de trabajo o web, Microsoft Azure asigna recursos de proceso. En ocasiones, es posible que reciba errores a realizar estas operaciones incluso antes de llegar a los límites de la suscripción de Azure. En este artículo se explican las causas de algunos de los errores de asignación más comunes y se sugieren posibles soluciones. La información también puede ser útil si tiene pensado realizar la implementación de sus servicios.

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

### <a name="background--how-allocation-works"></a>Información de contexto: cómo funciona la asignación
Los servidores de los centros de datos de Azure están particionados en clústeres. Se intenta una nueva solicitud de asignación de servicio en la nube en varios clústeres. Cuando se implementa la primera instancia a un servicio en la nube (ya sea en ensayo o producción), ese servicio en la nube se ancla a un clúster. Todas las implementaciones posteriores para el servicio en la nube tendrán lugar en el mismo clúster. En este artículo, haremos referencia a esto como que la solicitud está "anclada a un clúster". En el diagrama 1 siguiente se ilustra el caso de una asignación normal que se intenta en varios clústeres; en el diagrama 2 se ilustra el caso de una asignación que está anclada al clúster 2 porque es ahí donde se hospeda el servicio en la nube CS_1.

![Diagrama de asignación](./media/cloud-services-allocation-failure/Allocation1.png)

### <a name="why-allocation-failure-happens"></a>¿Por qué se producen errores de asignación?
Cuando una solicitud de asignación está anclada a un clúster, existe una posibilidad menor de encontrar recursos libres dado que el grupo de recursos disponible se limita a un clúster. Además, si la solicitud de asignación está anclada a un clúster pero el tipo de recurso que solicita no se admite en ese clúster, la solicitud dará error aunque el clúster tenga recursos libres. En el diagrama 3 a continuación se ilustra el caso en el que una asignación anclada da error porque el único clúster candidato no tiene recursos libres. En el diagrama 4 se ilustra el caso en el que una asignación anclada da error porque el único clúster candidato no admite el tamaño de VM solicitado, a pesar de que el clúster tiene recursos libres.

![Error de asignaciones ancladas](./media/cloud-services-allocation-failure/Allocation2.png)

## <a name="troubleshooting-allocation-failure-for-cloud-services"></a>Solución de errores de asignación para servicios en la nube
### <a name="error-message"></a>Mensaje de error
Puede ver el siguiente mensaje de error:

> Error "Operación de Azure" {operation id}" con el código Compute.ConstrainedAllocationFailed. Detalles: Error en la asignación; no se pudieron satisfacer las restricciones de la solicitud. La nueva implementación del servicio solicitada está enlazada a un grupo de afinidad, su destino es una instancia de Virtual Network o hay una implementación existente bajo este servicio hospedado. Todas estas condiciones restringen la nueva implementación a recursos específicos de Azure. Inténtelo de nuevo más tarde o pruebe a reducir el tamaño de la máquina virtual o el número de instancias de rol. También puede quitar, si es posible, las restricciones mencionadas o intentar realizar la implementación en otra región.

### <a name="common-issues"></a>Problemas comunes
A continuación se presentan los escenarios de asignación comunes que ocasionan que una solicitud de asignación quede anclada a un solo clúster.

* Implementación en la ranura de ensayo - Si un servicio en la nube tiene una implementación en cualquier ranura, entonces todo el servicio en la nube se ancla a un clúster concreto.  Esto significa que si una implementación ya existe en la ranura de producción, una nueva implementación de ensayo solo se puede asignar en el mismo clúster que la ranura de producción. Si la capacidad del clúster se está agotando, se puede producir un error en la solicitud.
* Escalado - La incorporación de nuevas instancias a un servicio en la nube existente se debe asignar en el mismo clúster.  Normalmente, las solicitudes de escalado pequeño se pueden asignar, pero no siempre. Si la capacidad del clúster se está agotando, se puede producir un error en la solicitud.
* Grupo de afinidad - Una nueva implementación en un servicio en la nube vacío se puede asignar mediante el tejido en cualquier clúster de esa región, a menos que el servicio en la nube esté anclado a un grupo de afinidad. Las implementaciones en el mismo grupo de afinidad se intentarán en el mismo clúster. Si la capacidad del clúster se está agotando, se puede producir un error en la solicitud.
* Virtual Network del grupo de afinidad - Las redes virtuales más antiguas estaban asociadas a grupos de afinidad en lugar de a regiones y los servicios en la nube de estas redes virtuales se anclarían al clúster del grupo de afinidad. Las implementaciones en este tipo de red virtual se intentarán en el clúster anclado. Si la capacidad del clúster se está agotando, se puede producir un error en la solicitud.

## <a name="solutions"></a>Soluciones
1. Volver a implementar un nuevo servicio en la nube - Esta solución es probable que sea más exitosa, ya que permite a la plataforma elegir entre todos los clústeres de esa región.

   * Implementar la carga de trabajo en un nuevo servicio en la nube.  
   * Actualizar el registro CNAME o A para que apunte el tráfico al nuevo servicio en la nube.
   * Una vez que ya no se dirige tráfico al sitio antiguo, puede eliminar el servicio en la nube antiguo. Esta solución debe incurrir en tiempo de inactividad cero.
2. Eliminar las ranuras de producción y ensayo - Esta solución conservará el nombre de DNS existente, pero provocará tiempo de inactividad en la aplicación.

   * Eliminar las ranuras de producción y ensayo de un servicio en la nube existente para que dicho esté vacío y después
   * Crear una nueva implementación en el servicio en la nube existente. Esto volverá a intentar la asignación en todos los clústeres de la región. Asegúrese de que el servicio en la nube no está asociado a un grupo de afinidad.
3. Dirección IP reservada: esta solución conservará su dirección IP existente, pero provocará tiempo de inactividad en la aplicación.  

   * Crear una dirección IP reservada para la implementación mediante Powershell.

     ```
     New-AzureReservedIP -ReservedIPName {new reserved IP name} -Location {location} -ServiceName {existing service name}
     ```
   * Siga el punto 2 anterior y asegúrese de especificar la nueva dirección IP reservada en el CSCFG del servicio.
4. Quitar el grupo de afinidad para nuevas implementaciones - Ya no se recomienda utilizar grupos de afinidad. Siga los pasos del punto 1 anterior para implementar un nuevo servicio en la nube. Asegúrese de que el servicio en la nube no se encuentra en un grupo de afinidad.
5. Convertir a una red virtual regional - Consulte [Migración de grupos de afinidad a una red virtual regional](../virtual-network/virtual-networks-migrate-to-regional-vnet.md).
