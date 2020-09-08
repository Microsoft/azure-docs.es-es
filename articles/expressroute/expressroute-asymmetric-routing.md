---
title: 'Azure ExpressRoute: Ruta asimétrica'
description: Este artículo lo guía a través de los problemas a los que se puede enfrentar con el enrutamiento asimétrico en una red que tiene varios vínculos a un destino.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: article
ms.date: 10/10/2016
ms.author: duau
ms.openlocfilehash: aa3465940088d3a66f23dfd5d58a6ec3fd3053de
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397719"
---
# <a name="asymmetric-routing-with-multiple-network-paths"></a>Enrutamiento asimétrico con varias rutas de acceso de red
En este artículo se explica cómo el tráfico de red de reenvío y de retorno puede tomar distintas rutas cuando hay varias rutas de acceso disponibles entre el origen y el destino de la red.

Para comprender el enrutamiento asimétrico, es importante entender dos conceptos. Uno es el efecto que produce tener varias rutas de acceso de red. El otro es ver cómo ciertos dispositivos, como un firewall, mantienen el estado. Estos tipos de dispositivos se denominan dispositivos con estado. Una combinación de estos dos factores permite crear escenarios en los que un dispositivo con estado interrumpe el tráfico de red porque no detecta que el tráfico se origina con el dispositivo mismo.

## <a name="multiple-network-paths"></a>Varias rutas de acceso de red
Cuando una red empresarial solo tiene un vínculo a Internet a través de su proveedor de servicios de Internet, todo el tráfico hacia Internet y desde él viaja por la misma ruta de acceso. A menudo, las empresas adquieren varios circuitos, como rutas de acceso redundantes, para mejorar el tiempo de actividad de la red. Cuando esto sucede, es posible que el tráfico que sale de la red, hacia Internet, vaya a través de un vínculo mientras que el tráfico de retorno viene a través de un vínculo diferente. Normalmente, esto se conoce como enrutamiento asimétrico. En el enrutamiento asimétrico, el tráfico de red de retorno toma una ruta de acceso distinta a la del flujo original.

![Red con varias rutas de acceso](./media/expressroute-asymmetric-routing/AsymmetricRouting3.png)

Aunque se produce principalmente en Internet, el enrutamiento asimétrico también se aplica a otras combinaciones de varias rutas de acceso. Se aplica, por ejemplo, a una ruta de acceso de Internet y a una ruta de acceso privada que van al mismo destino, o a varias rutas de acceso privadas que van al mismo destino.

Cada enrutador del recorrido, desde el origen hasta el destino, calcula la mejor ruta para llegar al destino. La determinación por parte del enrutador de la mejor ruta de acceso posible se basa en dos factores principales:

* El enrutamiento entre redes externas se basa en un protocolo de enrutamiento llamado Border Gateway Protocol (BGP). BGP toma anuncios de vecinos, ejecuta una serie de pasos con ellos para determinar la mejor ruta de acceso al destino deseado. Y almacena la mejor ruta de acceso en su tabla de enrutamiento.
* La longitud de una máscara de subred asociada con una ruta influye en las rutas de acceso de los enrutamientos. Si un enrutador recibe varios anuncios para la misma dirección IP pero con diferentes máscaras de subred, el enrutador prefiere el anuncio con la máscara de subred más larga porque se considera una ruta más específica.

## <a name="stateful-devices"></a>Dispositivos con estado
Los enrutadores miran el encabezado IP de un paquete para fines de enrutamiento. Algunos dispositivos miran de forma aún más profunda dentro del paquete. Normalmente, estos dispositivos miran los encabezados de la capa (4) (Protocolo de control de transmisión o TCP, o Protocolo de datagramas de usuario o UDP) o incluso de la capa 7 (capa de aplicación). Estas variantes de dispositivos son dispositivos de seguridad o dispositivos de optimización de ancho de banda. 

Un firewall es un ejemplo común de dispositivo con estado. Un firewall permite o rechaza que un paquete pase a través de sus interfaces basándose en diversos campos, como protocolo, puerto TCP o UDP y encabezados de dirección URL. Este nivel de inspección de paquete coloca una pesada carga de procesamiento en el dispositivo. Para mejorar el rendimiento, el firewall inspecciona el primer paquete de un flujo. Si se permite que el paquete continúe, la información de flujo se mantiene en su tabla de estado. Todos los paquetes posteriores relacionados con este flujo se permiten en función de la determinación inicial. Un paquete que forma parte de un flujo existente puede llegar al firewall. Si el firewall no tiene una información de estado anterior del paquete, lo descartará.

## <a name="asymmetric-routing-with-expressroute"></a>Enrutamiento asimétrico con ExpressRoute
Cuando se conecta a Microsoft a través de Azure ExpressRoute, se producen los siguientes cambios en la red:

* Tiene varios vínculos con Microsoft. Un vínculo es la conexión existente de Internet y el otro se realiza a través de ExpressRoute. Una parte del tráfico a Microsoft puede ir a través de Internet pero volver a través de ExpressRoute o viceversa.
* Recibe direcciones IP más específicas a través de ExpressRoute. Por lo tanto, para el tráfico de la red a Microsoft de los servicios ofrecidos a través de ExpressRoute, los enrutadores siempre darán preferencia a ExpressRoute.

Para comprender el efecto que estos dos cambios tienen en una red, veamos algunos escenarios. Por ejemplo, supongamos que solo tiene un circuito a Internet y consume todos los servicios de Microsoft a través de Internet. El tráfico de la red a Microsoft y el de vuelta recorre el mismo vínculo de Internet y pasa a través del firewall. El firewall registra el flujo según ve el primer paquete y los paquetes de retorno se permiten ya que el flujo existe en la tabla de estado.

![Enrutamiento asimétrico con ExpressRoute](./media/expressroute-asymmetric-routing/AsymmetricRouting1.png)

A continuación, ya puede activar ExpressRoute y consumir los servicios ofrecidos por Microsoft a través de ExpressRoute. Todos los demás servicios de Microsoft se consumen a través de Internet. Implemente un servidor de firewall independiente en su perímetro que se conecte a ExpressRoute. Microsoft anuncia prefijos más específicos en la red a través de ExpressRoute para servicios específicos. La infraestructura de enrutamiento elige ExpressRoute como ruta de acceso preferida para estos prefijos. Si no va a anunciar direcciones IP públicas en Microsoft a través de ExpressRoute, Microsoft se comunicará con las direcciones IP públicas a través de Internet. Por lo tanto, al reenviar el tráfico de la red a Microsoft, se usará ExpressRoute mientras que el tráfico inverso desde Microsoft utilizará Internet. Si el firewall en el perímetro ve un paquete de respuesta de un flujo que no se encuentra en la tabla de estado, descartará el tráfico de retorno.

Si decide mostrar el mismo grupo de traducción de direcciones de red (NAT) para ExpressRoute e Internet, tendrá problemas similares con los clientes en las direcciones IP privadas de la red. La solicitud de servicios como Windows Update pasará a través de Internet ya que las direcciones IP de estos servicios no se anuncian a través de ExpressRoute. Sin embargo, el tráfico de retorno volverá a través de ExpressRoute. Si Microsoft recibe una dirección IP con la misma máscara de subred de Internet y ExpressRoute, dará preferencia a ExpressRoute en lugar de a Internet. Si un firewall u otro dispositivo con estado en el perímetro de la red, orientado a ExpressRoute, no tiene ninguna información previa sobre el flujo, descartará los paquetes que pertenecen a ese flujo.

## <a name="asymmetric-routing-solutions"></a>Soluciones de enrutamiento asimétrico
Tiene dos opciones principales para solucionar el problema del enrutamiento asimétrico. Una es a través del enrutamiento y la otra es mediante el uso de NAT basada en origen (SNAT).

### <a name="routing"></a>Enrutamiento
Asegúrese de que las direcciones IP públicas se anuncian en los vínculos de red de área extensa (WAN) adecuados. Por ejemplo, si desea utilizar Internet para el tráfico de autenticación y ExpressRoute para el tráfico de correo, no debe anunciar las direcciones IP públicas de los Servicios de federación de Active Directory (AD FS) en ExpressRoute. De forma similar, asegúrese de no exponer un servidor local de AD FS a direcciones IP que el enrutador reciba a través de ExpressRoute. Las rutas recibidas a través de ExpressRoute son más específicas, por lo que ExpressRoute será la ruta preferida para el tráfico de autenticación a Microsoft. Esto provocará un enrutamiento asimétrico.

Si desea usar ExpressRoute para la autenticación, asegúrese de que va a anunciar direcciones IP públicas de AD FS a través de ExpressRoute sin NAT. De este modo, el tráfico que se origina en Microsoft y se dirige a un servidor local de AD FS se produce a través de ExpressRoute. El tráfico de retorno del cliente a Microsoft utiliza ExpressRoute porque es la ruta preferida en lugar de Internet.

### <a name="source-based-nat"></a>NAT basada en origen
Otra manera de resolver los problemas de enrutamiento asimétrico es mediante SNAT. Por ejemplo, no ha anunciado la dirección IP pública de un servidor local de Protocolo simple de transferencia de correo (SMTP) en ExpressRoute ya que pretende utilizar Internet para este tipo de comunicación. Una solicitud que se origina con Microsoft y va, a continuación, al servidor SMTP local recorre Internet. Realice una SNAT con la solicitud entrante a una dirección IP interna. El tráfico inverso desde el servidor SMTP va al firewall perimetral (que utiliza para NAT) en lugar de a través de ExpressRoute. El tráfico de retorno vuelve a través de Internet.

![Configuración de red con NAT basada en origen](./media/expressroute-asymmetric-routing/AsymmetricRouting2.png)

## <a name="asymmetric-routing-detection"></a>Detección de enrutamiento asimétrico
Traceroute es la mejor manera de asegurarse de que el tráfico de la red recorra la ruta de acceso prevista. Si espera que el tráfico del servidor SMTP local a Microsoft tome la ruta de acceso de Internet, realice el traceroute previsto del servidor SMTP a Office 365. El resultado validará que el tráfico sale de hecho de la red hacia Internet y no hacia ExpressRoute.

