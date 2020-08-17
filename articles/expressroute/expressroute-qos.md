---
title: 'Azure ExpressRoute: Requisitos QoS'
description: En esta página se proporcionan requisitos detallados para configurar y administrar QoS. Se analizan Skype Empresarial y los servicios de voz.
services: expressroute
author: cherylmc
ms.service: expressroute
ms.topic: conceptual
ms.date: 04/22/2019
ms.author: cherylmc
ms.openlocfilehash: 37c8b760ada12bd8f54c78ac6a0ea228d113b6c2
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86536993"
---
# <a name="expressroute-qos-requirements"></a>Requisitos de QoS ExpressRoute
Skype Empresarial tiene varias cargas de trabajo que requieren tratamiento diferenciado de QoS. Si piensa consumir servicios de voz a través de ExpressRoute, debe cumplir los requisitos descritos a continuación.

![Diagrama que muestra los servicios de voz que pasan a través de ExpressRoute.](./media/expressroute-qos/expressroute-qos.png)

> [!NOTE]
> Los requisitos de QoS solo se aplican al emparejamiento de Microsoft. Los valores de DSCP en el tráfico de red recibido en el emparejamiento público y privado de Azure se restablecerán en 0. 
> 
> 

En la tabla siguiente se proporciona una lista de las marcas de DSCP que usan Microsoft Teams y Skype Empresarial. Consulte [Administración de QoS para Skype Empresarial](https://docs.microsoft.com/SkypeForBusiness/manage/network-management/qos/managing-quality-of-service-QoS) para obtener más información.

| **Clase de tráfico** | **Tratamiento (marcado de DSCP)** | **Cargas de trabajo de Microsoft Teams y Skype Empresarial** |
| --- | --- | --- |
| **Voz** |EF (46) |Skype/Microsoft Teams/Lync Voice |
| **Interactivo** |AF41 (34) |Video, VBSS |
| |AF21 (18) |Uso compartido de aplicaciones | 
| **Valor predeterminado** |AF11 (10) |Transferencia de archivos |
| |CS0 (0) |Nada más |

* Debe clasificar las cargas de trabajo y marcar los valores de DSCP correctos. Siga las instrucciones proporcionadas [aquí](https://docs.microsoft.com/SkypeForBusiness/manage/network-management/qos/configuring-port-ranges-for-your-skype-clients#configure-quality-of-service-policies-for-clients-running-on-windows-10) sobre cómo establecer marcados de DSCP en la red.
* Debe configurar y admitir varias colas de QoS dentro de la red. La voz debe ser una clase independiente y recibir el tratamiento EF especificado en [RFC 3246](https://www.ietf.org/rfc/rfc3246.txt). 
* Puede decidir el mecanismo de puesta en cola, la directiva de detección de congestión y la asignación de ancho de banda por clase de tráfico. Sin embargo, se debe preservar el marcado de DSCP para cargas de trabajo de Skype Empresarial. Si utiliza marcados de DSCP no enumerados anteriormente, por ejemplo, AF31 (26), debe reescribir este valor de DSCP a 0 antes de enviar el paquete a Microsoft. Microsoft solo envía paquetes marcados con el valor de DSCP que se muestra en la tabla anterior. 

## <a name="next-steps"></a>Pasos siguientes
* Consulte los requisitos de [enrutamiento](expressroute-routing.md) y [NAT](expressroute-nat.md).
* Consulte los vínculos siguientes para configurar la conexión ExpressRoute.
  
  * [Creación de un circuito ExpressRoute](expressroute-howto-circuit-classic.md)
  * [Configuración del enrutamiento](expressroute-howto-routing-classic.md)
  * [Vinculación de redes virtuales a circuitos ExpressRoute](expressroute-howto-linkvnet-classic.md)

