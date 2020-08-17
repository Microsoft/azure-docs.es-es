---
title: ¿Qué es la dirección IP 168.63.129.16? | Microsoft Docs
description: Obtenga información sobre la dirección IP 168.63.129.16, en concreto que se usa para facilitar un canal de comunicación con los recursos de la plataforma Azure.
services: virtual-network
documentationcenter: na
author: genlin
manager: dcscontentpm
editor: v-jesits
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/15/2019
ms.author: genli
ms.openlocfilehash: 0f0bfa693086a3a097df219132d696a1d04e6f56
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87286043"
---
# <a name="what-is-ip-address-1686312916"></a>¿Qué es la dirección IP 168.63.129.16?

La dirección IP 168.63.129.16 es una dirección IP pública virtual que se usa para facilitar un canal de comunicación a los recursos de la plataforma Azure. Los clientes pueden definir cualquier espacio de direcciones de su red virtual privada en Azure. Por lo tanto, los recursos de la plataforma Azure se deben presentar como una única dirección IP pública. Esta dirección IP pública virtual facilita lo siguiente:

- Permite al agente de VM comunicarse con la plataforma Azure para indicar que se encuentra en estado "Listo".
- Permite la comunicación con el servidor DNS virtual para proporcionar resolución de nombres filtrada a los recursos (como una máquina virtual) que no tienen un servidor DNS personalizado. Este filtrado garantiza que los clientes puedan resolver solo los nombres de host de sus recursos.
- Permite que los [sondeos de estado de Azure Load Balancer](../load-balancer/load-balancer-custom-probe-overview.md) determinen el estado de mantenimiento de las VM.
- Permite que la máquina virtual obtenga una dirección IP dinámica desde el servicio DHCP en Azure.
- Habilita los mensajes de latido del agente invitado para el rol PaaS.

## <a name="scope-of-ip-address-1686312916"></a>Ámbito de la dirección IP 168.63.129.16

La dirección IP pública 168.63.129.16 se utiliza en todas las regiones y en todas las nubes nacionales. Esta dirección IP pública especial es propiedad de Microsoft y no se cambiará. Se recomienda que permita esta dirección IP en las directivas de firewall locales (en la máquina virtual) en la dirección de salida. La comunicación entre esta dirección IP especial y los recursos es segura porque solo la plataforma interna de Azure puede originar un mensaje desde esta dirección IP. Si esta dirección se bloquea, puede producirse un comportamiento inesperado en una variedad de escenarios. 168.63.129.16 es una [dirección IP virtual del nodo de host](../virtual-network/security-overview.md#azure-platform-considerations) y, como tal, no está sujeta a las rutas definidas por el usuario.

- El agente de máquina virtual requiere comunicación saliente a través de los puertos 80, 443 y 32526 con WireServer (168.63.129.16). Estos deben estar abiertos en el firewall local de la máquina virtual. La comunicación en estos puertos con 168.63.129.16 no está sujeta a los grupos de seguridad de red configurados.
- 168.63.129.16 puede proporcionar servicios DNS a la máquina virtual. Si no lo desea, este tráfico puede bloquearse en el firewall local de la máquina virtual. De forma predeterminada, la comunicación de DNS no está sujeta a los grupos de seguridad de red configurados a menos que se dirija específicamente aprovechando la etiqueta de servicio [AzurePlatformDNS](../virtual-network/service-tags-overview.md#available-service-tags). Para bloquear el tráfico DNS a Azure DNS a través del grupo de seguridad de red, cree una regla de salida para denegar el tráfico a [AzurePlatformDNS](../virtual-network/service-tags-overview.md#available-service-tags) y especifique "*" como "Intervalos de puertos de destino" y "Cualquiera" como protocolo.
- Cuando la máquina virtual forma parte de un grupo de back-end del equilibrador de carga, debe permitirse que la comunicación del [sondeo de estado](../load-balancer/load-balancer-custom-probe-overview.md) se origine en 168.63.129.16. La configuración predeterminada del grupo de seguridad de red tiene una regla que permite esta comunicación. Esta regla aprovecha la etiqueta de servicio [AzureLoadBalancer](../virtual-network/service-tags-overview.md#available-service-tags). Si lo desea, puede bloquearse este tráfico mediante la configuración del grupo de seguridad de red; sin embargo, se producirán errores en los sondeos.

En un escenario de red que no es virtual (clásica), el sondeo de estado se origina desde una dirección IP privada y 168.63.129.16 no se usa.

## <a name="next-steps"></a>Pasos siguientes

- [Grupos de seguridad](security-overview.md)
- [Creación, modificación o eliminación de un grupo de seguridad de red](manage-network-security-group.md)
