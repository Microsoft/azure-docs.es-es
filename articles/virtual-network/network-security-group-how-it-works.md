---
title: 'Grupo de seguridad de red: funcionamiento'
titlesuffix: Azure Virtual Network
description: Obtenga información sobre cómo los grupos de seguridad de red le ayudan a filtrar el tráfico de red entre los recursos de Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/24/2020
ms.author: kumud
ms.reviewer: kumud
ms.openlocfilehash: e60cdfb00d0dc9d446bd52a72e9fd15676acd285
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89458202"
---
# <a name="how-network-security-groups-filter-network-traffic"></a>Cómo filtran el tráfico de red los grupos de seguridad de red
<a name="network-security-groups"></a>

Puede usar el grupo de seguridad de red de Azure para filtrar el tráfico de red hacia y desde los recursos de Azure de una red virtual de Azure. Un grupo de seguridad de red contiene [reglas de seguridad](https://docs.microsoft.com/azure/virtual-network/security-overview#security-rules) que permiten o deniegan el tráfico de red entrante o el tráfico de red saliente de varios tipos de recursos de Azure. Para cada regla, puede especificar un origen y destino, un puerto y un protocolo.

Puede implementar recursos de varios servicios de Azure en una red virtual de Azure. Para obtener una lista completa, consulte [Servicios que se pueden implementar en una red virtual](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network). Puede asociar cero o un grupo de seguridad de red a cada [subred](virtual-network-manage-subnet.md#change-subnet-settings) e [interfaz de red](virtual-network-network-interface.md#associate-or-dissociate-a-network-security-group) de la red virtual en una máquina virtual. El mismo grupo de seguridad de red se puede asociar a tantas interfaces de red y subredes como se desee.

La siguiente imagen ilustra los diferentes escenarios de cómo se podrían implementar grupos de seguridad de red para permitir el tráfico de red hacia y desde Internet a través del puerto TCP 80:

![Procesamiento del grupo de seguridad de red](./media/network-security-group-how-it-works/network-security-group-interaction.png)

Observe la imagen anterior, junto con el siguiente texto, para entender cómo procesa Azure las reglas entrantes y salientes para los grupos de seguridad de red:

## <a name="inbound-traffic"></a>Tráfico entrante

Para el tráfico entrante, Azure procesa las reglas de un grupo de seguridad de red asociadas a una subred en primer lugar, si hay alguna y, a continuación, las reglas de un grupo de seguridad de red asociadas a la interfaz de red, si hay alguna.

- **VM1**: las reglas de seguridad de *NSG1* se procesan, ya que está asociado a *Subnet1* y *VM1* está en *Subnet1*. A menos que haya creado una regla que permita el puerto 80 de entrada, la regla de seguridad predeterminada [DenyAllInbound](https://docs.microsoft.com/azure/virtual-network/security-overview#denyallinbound) deniega el tráfico y *NSG2* nunca lo evalúa, ya que *NSG2* está asociado a la interfaz de red. Si *NSG1* tiene una regla de seguridad que permite el puerto 80, *NSG2* procesa el tráfico. Para permitir el puerto 80 para la máquina virtual, tanto *NSG1* como *NSG2* deben tener una regla que permita el puerto 80 desde Internet.
- **VM2**: las reglas de *NSG1* se procesan porque *VM2* también está en *Subnet1*. Puesto que *VM2* no tiene un grupo de seguridad de red asociado a su interfaz de red, recibe todo el tráfico permitido por *NSG1* o se deniega todo el tráfico denegado por *NSG1*. El tráfico se permite o deniega a todos los recursos de la misma subred cuando un grupo de seguridad de red está asociado a una subred.
- **VM3**: dado que no hay ningún grupo de seguridad de red asociado a *Subnet2*, se permite el tráfico en la subred y *NSG2* lo procesa, porque *NSG2* está asociado a la interfaz de red conectada a *VM3*.
- **VM4**: se permite el tráfico a *VM4*, porque no hay un grupo de seguridad de red asociado a *Subnet3* ni a la interfaz de red de la máquina virtual. Si no tienen un grupo de seguridad de red asociado, se permite todo el tráfico de red a través de una subred y una interfaz de red.

## <a name="outbound-traffic"></a>Tráfico saliente

Para el tráfico saliente, Azure procesa las reglas de un grupo de seguridad de red asociadas a una interfaz de red en primer lugar, si hay alguna y, a continuación, las reglas de un grupo de seguridad de red asociadas a la subred, si hay alguna.

- **VM1**: se procesan las reglas de seguridad de *NSG2*. A menos que cree una regla de seguridad que deniegue el puerto 80 de salida a Internet, la regla de seguridad predeterminada [AllowInternetOutbound](https://docs.microsoft.com/azure/virtual-network/security-overview#allowinternetoutbound) permite el tráfico de *NSG1* y *NSG2*. Si *NSG2* tiene una regla de seguridad que deniega el puerto 80, el tráfico se deniega y *NSG1* nunca lo evalúa. Para denegar el puerto 80 desde la máquina virtual, uno o ambos de los grupos de seguridad de red deben tener una regla que deniegue el puerto 80 a Internet.
- **VM2**: se envía todo el tráfico a través de la interfaz de red a la subred, ya que la interfaz de red conectada a *VM2* no tiene un grupo de seguridad de red asociado. Se procesan las reglas de *NSG1*.
- **VM3**: si *NSG2* tiene una regla de seguridad que deniega el puerto 80, también se deniega el tráfico. Si *NSG2* tiene una regla de seguridad que permite el puerto 80, dicho puerto tiene permitida la salida a Internet, ya que no hay un grupo de seguridad de red asociado a *Subnet2*.
- **VM4**: se permite todo el tráfico de red desde *VM4*, porque no hay un grupo de seguridad de red asociado a la interfaz de red conectada a la máquina virtual ni a *Subnet3*.


## <a name="intra-subnet-traffic"></a>Tráfico dentro de la subred

Es importante tener en cuenta que las reglas de seguridad de un NSG asociado a una subred pueden afectar la conectividad entre las máquinas virtuales dentro de ella. Por ejemplo, si se agrega una regla a *NSG1* que deniega todo el tráfico entrante y saliente, *VM1* y *VM2* ya no podrán comunicarse entre sí. Otra regla tendría que agregarse específicamente para permitirlo. 

Puede ver fácilmente las reglas agregadas que se aplican a una interfaz de red mediante la visualización de las [reglas de seguridad vigentes](virtual-network-network-interface.md#view-effective-security-rules) de una interfaz de red. También puede usar la funcionalidad [Comprobación del flujo de IP](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure Network Watcher para determinar si se permite la comunicación hacia una interfaz de red o desde esta. El flujo IP le indica si se permite o deniega una comunicación y qué regla de seguridad de red permite o deniega el tráfico.

> [!NOTE]
> Los grupos de seguridad de red se asocian a las subredes o a las máquinas virtuales y a los servicios en la nube que se implementan en el modelo de implementación clásica, y en las subredes o interfaces de red del modelo de implementación de Resource Manager. Para más información sobre los modelos de implementación de Azure, consulte [Descripción de los modelos de implementación de Azure](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

> [!TIP]
> A menos que tenga un motivo concreto, se recomienda que asocie un grupo de seguridad de red a una subred o a una interfaz de red, pero no a ambas. Puesto que las reglas de un grupo de seguridad de red asociado a una subred pueden entrar en conflicto con las reglas de un grupo de seguridad de red asociado a una interfaz de red, puede tener problemas de comunicación inesperados que necesiten solución.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información acerca de qué recursos de Azure se pueden implementar en una red virtual y pueden tener grupos de seguridad de red asociados, consulte [Integración de redes virtuales para los servicios de Azure](virtual-network-for-azure-services.md).
* Si nunca ha creado un grupo de seguridad de red, puede seguir un [tutorial](tutorial-filter-network-traffic.md) rápido para obtener alguna experiencia en la creación de uno.
* Si está familiarizado con los grupos de seguridad de red y necesita administrarlos, consulte [Administración de un grupo de seguridad de red](manage-network-security-group.md). 
* Si tiene problemas con las comunicaciones y necesita solucionar problemas de los grupos de seguridad de red, consulte [Diagnóstico de un problema de filtro de tráfico de red de una máquina virtual](diagnose-network-traffic-filter-problem.md). 
* Aprenda a habilitar los [registros de flujo de los grupos de seguridad de red](../network-watcher/network-watcher-nsg-flow-logging-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para analizar el tráfico de red hacia y desde los recursos que tienen un grupo de seguridad de red asociado.
