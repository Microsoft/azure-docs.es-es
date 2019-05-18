---
title: 'Virtual Network: continuidad del negocio | Microsoft Docs'
description: Obtenga información acerca de qué hacer si se produce una interrupción de un servicio de Azure que afecta a las instancias de Azure Virtual Network.
services: virtual-network
documentationcenter: ''
author: NarayanAnnamalai
manager: jefco
editor: ''
ms.assetid: ad260ab9-d873-43b3-8896-f9a1db9858a5
ms.service: virtual-network
ms.workload: virtual-network
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/16/2016
ms.author: narayan;aglick
ms.openlocfilehash: 68a9523dcc9c4dd84399c68fc7e31a692c011487
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523259"
---
# <a name="virtual-network--business-continuity"></a>Virtual Network: continuidad del negocio

## <a name="overview"></a>Información general
Una red virtual (VNet) es una representación lógica de su red en la nube. Permite definir su propio espacio de direcciones IP privadas y segmentar la red en subredes. Las redes virtuales actúan como un límite de confianza para hospedar los recursos de proceso, como Máquinas virtuales y Cloud Services (roles web y de trabajo) de Azure. Una red virtual permite la comunicación IP privada directa entre los recursos hospedados en ella. Puede vincular una red virtual a una red local mediante una instancia de VPN Gateway o por medio de ExpressRoute.

Una red virtual se crea dentro del ámbito de una región. También puede *crear* redes virtuales con el mismo espacio de direcciones en dos regiones diferentes (por ejemplo, EE.UU y oeste de EE), pero porque tienen el mismo espacio de direcciones, no puede conectarlos entre sí. 

## <a name="business-continuity"></a>Continuidad del negocio

La aplicación puede sufrir diferentes formas de interrupción. Por ejemplo, es posible que el servicio se corte por completo en una región determinada como consecuencia de un desastre natural o que se produzca un desastre parcial debido a un error en varios dispositivos o servicios. La repercusión en el servicio de red virtual es diferente en cada una de estas situaciones.

**P: Si se produce una interrupción de toda una región, ¿qué hacer? Por ejemplo, ¿y si debido a un desastre natural el servicio se ha cortado por completo en una región? ¿Qué ocurre con las redes virtuales hospedadas en la región?**

R: La red virtual y los recursos en la región afectada permanecen inaccesibles durante el tiempo de la interrupción del servicio.

![Diagrama de red virtual simple](./media/virtual-network-disaster-recovery-guidance/vnet.png)

**P: ¿Qué se puede volver a crear la misma red virtual en una región distinta?**

R: Las redes virtuales son recursos bastante ligeros. Puede invocar las API de Azure para crear una red virtual con el mismo espacio de direcciones en una región distinta. Para volver a crear el mismo entorno que estaba presente en la región afectada, tendrá que realizar llamadas API para volver a implementar los roles web y de trabajo de Cloud Services y las máquinas virtuales que tenía. Si tiene conectividad local, como una implementación híbrida, tendrá que implementar una nueva instancia de VPN Gateway y conectarse a su red local.

Para crear una red virtual, consulte [Creación de una red virtual](manage-virtual-network.md#create-a-virtual-network).

**P: ¿Una réplica de una red virtual en una región determinada se puede recrear en otra región de antemano?**

R: Sí, puede crear dos redes virtuales con el mismo espacio de direcciones IP privadas y los recursos en dos regiones diferentes antes de tiempo. Si hospedaba servicios orientados a Internet en la red virtual, podría haber configurado Traffic Manager para distribuir geográficamente el tráfico en la región que está activa. Sin embargo, dos redes virtuales que tengan el mismo espacio de direcciones no se pueden conectar con la red local, ya que se producirían problemas de enrutamiento. Si se produjera un desastre y la pérdida de una red virtual en una región, podría conectar la otra red virtual de la región disponible, con el espacio de direcciones coincidente con la red local.

Para crear una red virtual, consulte [Creación de una red virtual](manage-virtual-network.md#create-a-virtual-network).

