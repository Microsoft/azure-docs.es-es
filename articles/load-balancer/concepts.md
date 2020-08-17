---
title: Conceptos de Azure Load Balancer
description: Información general acerca de los conceptos de Azure Load Balancer
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/13/2020
ms.author: allensu
ms.openlocfilehash: 40b738c0f074f06b2f15a260cacda876aa78daf6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87060802"
---
# <a name="azure-load-balancer-concepts"></a>Conceptos de Azure Load Balancer

Load Balancer proporciona varias funcionalidades para las aplicaciones UDP y TCP. 

## <a name="load-balancing-algorithm"></a>Algoritmo de equilibrio de carga
Se puede crear una regla de equilibrio de carga para distribuir el tráfico del front-end a un grupo del back-end. Azure Load Balancer usa un algoritmo hash para la distribución de los flujos (no bytes) entrantes. Load Balancer reescribe los encabezados de los flujos en las instancias del grupo del back-end. Un servidor está disponible para recibir nuevos flujos cuando el sondeo de estado indica un punto de conexión de back-end correcto.

De forma predeterminada, Load Balancer usa un hash de tupla de cinco elementos.

El hash incluye:

- **Dirección IP de origen**
- **Puerto de origen**
- **Dirección IP de destino**
- **Puerto de destino**
- **Número de protocolo IP para asignar flujos a servidores disponibles**

La afinidad con una dirección IP de origen se crea mediante un hash de tuplas de dos o tres elementos. Todos los paquetes del mismo flujo de paquetes llegan a la misma instancia detrás del front-end con equilibrio de carga. 

El puerto de origen cambia cuando un cliente inicia un nuevo flujo desde Cuando el cliente inicia un nuevo flujo desde la misma dirección IP de origen. En consecuencia, el hash de tupla de cinco elementos puede hacer que el tráfico vaya a otro punto de conexión de back-end.
Para más información, consulte [Configurar el modo de distribución para Azure Load Balancer](./load-balancer-distribution-mode.md).

En la siguiente imagen se muestra la distribución basada en hash:

![Distribución basada en hash](./media/load-balancer-overview/load-balancer-distribution.png)

*Ilustración: distribución basada en hash*

## <a name="application-independence-and-transparency"></a>Independencia y transparencia de aplicaciones

Load Balancer no interactúa directamente con TCP ni UDP ni con la capa de aplicación. Se puede admitir cualquier escenario de aplicación TCP o UDP. Load Balancer no cierra ni origina flujos, ni interactúa con la carga del flujo. Load Balancer no proporciona la funcionalidad de puerta de enlace de la capa de aplicación. Los protocolos de enlace de protocolo siempre se producen directamente entre el cliente y la instancia del grupo de back-end. Una respuesta a un flujo de entrada siempre es una respuesta de una máquina virtual. Cuando llega el flujo a la máquina virtual, también se conserva la dirección IP de origen original.

* A cada punto de conexión le responde una máquina virtual. Por ejemplo, un protocolo de enlace TCP se realiza entre el cliente y la máquina virtual de back-end seleccionada. Una respuesta a una solicitud a un front-end es una respuesta generada por una máquina virtual de back-end. Cuando se valida correctamente la conectividad con un front-end, se valida la conectividad de un extremo a otro con al menos una máquina virtual de back-end.
* Las cargas útiles de las aplicaciones son transparentes para Load Balancer. Se puede admitir cualquier aplicación UDP o TCP.
* Como Load Balancer no interactúa con la carga de TCP y proporciona descarga de TLS, puede crear escenarios cifrados completos. Con Load Balancer se obtiene una mayor escalabilidad horizontal de las aplicaciones TLS, ya que la conexión TLS finaliza en la propia máquina virtual. Por ejemplo, la capacidad para claves de sesión TLS solo está limitada por el tipo y número de máquinas virtuales que se agregan al grupo de servidores back-end.

## <a name="outbound-connections"></a>Conexiones de salida 

Los flujos del grupo de back-end a direcciones IP públicas se asignan al front-end. Azure traduce las conexiones salientes a la dirección IP pública del front-end a través de la regla de salida de equilibrio de carga. Esta configuración tiene las siguientes ventajas. Actualización sencilla y recuperación ante desastres de servicios, ya que el front-end se puede asignar dinámicamente a otra instancia del servicio. Facilita la administración de la lista de control de acceso (ACL). Las ACL que se expresan como direcciones IP de front-end no cambian a medida que los servicios se escalan o bajan horizontalmente o se vuelve a implementar. La traducción de las conexiones salientes a un número menor de direcciones IP que las máquinas reduce la carga de la implementación de listas de destinatarios seguras. Para más información sobre la traducción de direcciones de red de origen (SNAT) y Azure Load Balancer, consulte [SNAT y Azure Load Balancer](load-balancer-outbound-connections.md).

## <a name="availability-zones"></a>Zonas de disponibilidad 

Standard Load Balancer admite otras funcionalidades en las regiones donde Availability Zones está disponible. Las configuraciones de Availability Zones están disponibles para ambos tipos de instancias de Standard Load Balancer, públicas e internas. Un front-end con redundancia de zona sobrevive a los errores de la zona mediante una infraestructura dedicada en todas las zonas al mismo tiempo. Además, puede garantizar un front-end en una zona específica. Un front-end zonal es atendido por una infraestructura dedicada en una zona individual. El equilibrio de carga entre zonas está disponible para el grupo de back-end. Cualquier recurso de máquina virtual de una red virtual puede formar parte de un grupo de back-end. Basic Load Balancer no admite zonas. Revise la [explicación detallada de las capacidades relacionadas con las zonas de disponibilidad](load-balancer-standard-availability-zones.md) y la [introducción a las zonas de seguridad](../availability-zones/az-overview.md) para más información.

## <a name="ha-ports"></a>Puertos de alta disponibilidad

Puede configurar reglas de equilibrio de carga en puertos de alta disponibilidad para que una aplicación sea escalable y muy confiable. Estas reglas proporcionan un equilibrio de carga por flujo a los puertos transitorios de la IP del front-end del equilibrador de carga interno. La característica es útil cuando no es práctico o no se desea especificar puertos individuales. Una regla de puertos de alta disponibilidad le permite crear n+1 escenarios de activo-pasivo o activo-activo. Estos escenarios son para aplicaciones virtuales de red y cualquier aplicación, lo que requiere grandes rangos de puertos de entrada. Se puede usar un sondeo de mantenimiento para determinar qué back-ends deberían recibir nuevos flujos.  Puede usar un grupo de seguridad de red para emular un escenario de intervalo de puertos. Basic Load Balancer no admite puertos de alta disponibilidad. Consulte la [explicación detallada sobre los puertos HA](load-balancer-ha-ports-overview.md).

## <a name="multiple-frontends"></a>Varios servidores front-end 

Load Balancer admite varias reglas con varios front-ends.  Standard Load Balancer amplía esta funcionalidad a los escenarios de salida. Las reglas de salida son lo opuesto a las de entrada. La regla de salida crea una asociación para las conexiones de salida. Standard Load Balancer usa todos los front-ends asociados a un recurso de máquina virtual mediante una regla de equilibrio de carga. Además, un parámetro en la red de equilibrio de carga permite suprimir una red de equilibrio de carga para la conectividad de salida, lo que permite elegir front-ends específicos o, incluso, ninguno. Para la comparación, Basic Load Balancer selecciona un solo front-end de forma aleatoria. No hay ninguna capacidad que controle qué front-end se ha seleccionado.

## <a name="floating-ip"></a>Dirección IP flotante

Algunos escenarios de aplicación prefieren o requieren que varias instancias de la aplicación usen el mismo puerto en una sola máquina virtual en el grupo back-end. Entre los ejemplos comunes de reutilización de puertos se incluyen la agrupación en clústeres para alta disponibilidad, dispositivos de red virtuales y la exposición de varios puntos de conexión TLS sin volver a cifrar. Si desea reutilizar el puerto back-end en varias reglas, debe habilitar la IP flotante en la definición de la regla.

La **IP flotante** es el término de Azure para referirse a una parte de lo que se conoce como Direct Server Return (DSR). DSR consta de dos partes: 

- Topología de flujo
- Esquema de asignación de direcciones IP

En un nivel de plataforma, Azure Load Balancer siempre funciona en una topología de flujo DSR independientemente de si la dirección IP flotante está habilitada o no. Esto significa que la parte de salida de un flujo siempre se reescribe correctamente para que se dirija de nuevo al origen.
Sin una IP flotante, Azure expone un esquema de asignación de direcciones IP de equilibrio para facilitar el uso (la IP de las instancias de máquina virtual). Habilitar la dirección IP flotante cambia el esquema de asignación de direcciones IP a la IP de servidor front-end del equilibrador de carga para permitir una flexibilidad adicional. Obtenga más información [aquí](load-balancer-multivip-overview.md).


## <a name="limitations"></a><a name = "limitations"></a>Limitaciones

- La IP flotante no se admite actualmente en las configuraciones de IP secundarias para los escenarios de equilibrio de carga internos.

- Una regla de equilibrador de carga no puede abarcar dos redes virtuales.  Los front-end y sus instancias de back-end deben estar ubicados en la misma red virtual.  

- A los roles de trabajo web sin una red virtual y otros servicios de plataforma de Microsoft solo se puede acceder desde instancias que estén detrás de una instancia interna de Standard Load Balancer. No dependa de esta accesibilidad, ya que tanto el propio servicio como la plataforma subyacente pueden cambiar sin previo aviso. Si se requiere conectividad saliente al usar una instancia interna de Standard Load Balancer, se debe configurar la [conectividad saliente](load-balancer-outbound-connections.md).

- Load Balancer proporciona equilibrio de carga y reenvío de puertos para protocolos TCP o UDP concretos. Las reglas de equilibrio de carga y las reglas NAT de entrada admiten TCP y UDP, pero no otros protocolos IP, incluido ICMP.

- No se generará el flujo saliente de una máquina virtual del back-end a un front-end de un equilibrador de carga interno.

- No se admite el reenvío de fragmentos IP en reglas de equilibrio de carga. No se admite la fragmentación IP de paquetes UDP y TCP en reglas de equilibrio de carga. Las reglas de equilibrio de carga de los puertos de alta disponibilidad se pueden usar para reenviar fragmentos de IP existentes. Para más información, consulte [Información general sobre los puertos de alta disponibilidad](load-balancer-ha-ports-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- Consulte [Creación de una instancia de Standard Load Balancer](quickstart-load-balancer-standard-public-portal.md) para empezar a usar una instancia de Load Balancer: crear una, crear máquinas virtuales con una extensión de IIS personalizada instalada y equilibrar la carga de la aplicación web entre las máquinas virtuales.
- Información sobre [Conexiones salientes de Azure Load Balancer](load-balancer-outbound-connections.md).
- Más información sobre [Azure Load Balancer](load-balancer-overview.md).
- Más información sobre [sondeos de mantenimiento](load-balancer-custom-probe-overview.md).
- Más información acerca de los [diagnósticos de Load Balancer Estándar](load-balancer-standard-diagnostics.md).
- Más información sobre los [grupos de seguridad de red](../virtual-network/security-overview.md).
