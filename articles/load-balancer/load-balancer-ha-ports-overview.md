---
title: Introducción a los puertos de alta disponibilidad en Azure
titleSuffix: Azure Load Balancer
description: Conozca el equilibrio de carga de los puertos de alta disponibilidad en una instancia interna de Load Balancer.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/19/2019
ms.author: allensu
ms.openlocfilehash: 62c3dad980b65c3e8cfc64c98ba173f667e0b5fe
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87447055"
---
# <a name="high-availability-ports-overview"></a>Introducción a los puertos de alta disponibilidad

Azure Load Balancer Estándar le ayuda a equilibrar la carga de los flujos TCP y UDP en todos los puertos a la vez cuando se usa un equilibrador de carga interno. 

Una regla de equilibrio de carga de puertos de alta disponibilidad (HA) es una variante de una regla de equilibrio de carga configurada en una instancia interna de Standard Load Balancer. Puede simplificar el uso de un equilibrador de carga si proporciona una única regla para equilibrar la carga de todos los flujos TCP y UDP que llegan a todos los puertos de una instancia interna de Load Balancer Estándar. La decisión de equilibrio de carga se toma por cada flujo. Esta decisión se toma en función de la conexión de cinco tuplas siguiente: dirección IP de origen, puerto de origen, dirección IP de destino, puerto de destino y protocolo.

Las reglas de equilibrio de carga de puertos de alta disponibilidad le ayudan a la hora de usar escenarios críticos como aquellos con alta disponibilidad y escalabilidad para dispositivos virtuales de red (NVA) que estén en redes virtuales. La característica también ayuda cuando hay que equilibrar la carga de un gran número de puertos. 

Las reglas de equilibrio de carga de puertos de alta disponibilidad se configuran al establecer los puertos de front-end y back-end en **0** y el protocolo en **Todos**. Es entonces cuando el equilibrador de carga interno equilibra todos los flujos TCP y UDP, independientemente del número de puerto.

## <a name="why-use-ha-ports"></a>¿Por qué usar puertos de alta disponibilidad?

### <a name="network-virtual-appliances"></a><a name="nva"></a>Dispositivos virtuales de red

Puede usar dispositivos virtuales de red (NVA) para proteger la carga de trabajo de Azure frente a varios tipos de amenazas de seguridad. Cuando se usan NVA en estos escenarios, deben ser confiables y de alta disponibilidad y escalar horizontalmente a petición.

Puede lograr estos objetivos si agrega instancias de NVA al grupo de back-end del equilibrador de carga interno y configura una regla de equilibrador de carga en puertos de alta disponibilidad.

En los escenarios de HA de NVA, los puertos de alta disponibilidad proporcionan las siguientes ventajas:
- Proporcionan conmutación por error rápida para instancias en buen estado con sondeos de estado por instancia
- Garantizan un mayor rendimiento con escalado horizontal para instancias *n*-activas
- Proporcionan escenarios *n*-activos y activo-pasivos
- Eliminan la necesidad de usar soluciones complejas, como los nodos de Apache ZooKeeper, para la supervisión de dispositivos

En el diagrama siguiente se muestra una implementación de la red virtual de tipo hub-and-spoke. Los radios dirigen su tráfico a la red virtual del concentrador y a través de los NVA, antes de abandonar el espacio de confianza. Los NVA están detrás de una instancia interna de Load Balancer estándar con una configuración de puertos de alta disponibilidad. En consecuencia, se puede procesar y reenviar todo el tráfico. Cuando se configura como se muestra en el diagrama siguiente, una regla de equilibrio de carga de puertos de alta disponibilidad proporciona además simetría de flujo para el tráfico de entrada y salida.

<a node="diagram"></a>
![Diagrama de la red virtual de tipo hub-and-spoke con los NVA implementados en modo de alta disponibilidad](./media/load-balancer-ha-ports-overview/nvaha.png)

>[!NOTE]
> Si va a usar NVA, confirme con su proveedor cómo aprovechar mejor los puertos de alta disponibilidad y cuáles son los escenarios admitidos.

### <a name="load-balancing-large-numbers-of-ports"></a>Equilibrado de carga de un gran número de puertos

También puede usar puertos de alta disponibilidad para aplicaciones que requieran equilibrar la carga de un gran número de puertos. Igualmente, puede simplificar estos escenarios mediante una instancia interna de [Load Balancer estándar](load-balancer-standard-overview.md) con puertos de alta disponibilidad. Una regla de equilibrio de carga única sustituye a varias reglas individuales de equilibrio de carga, una por cada puerto.

## <a name="region-availability"></a>Disponibilidad en regiones

La característica de puertos de alta disponibilidad está disponible en todas las regiones globales de Azure.

## <a name="supported-configurations"></a>Configuraciones admitidas

### <a name="a-single-non-floating-ip-non-direct-server-return-ha-ports-configuration-on-an-internal-standard-load-balancer"></a>Una configuración de puertos HA con una única dirección IP no flotante (sin Direct Server Return) en una instancia interna de Load Balancer Estándar

Esta es una configuración básica de puertos HA. Para configurar una regla de equilibrio de carga de puertos HA en una sola dirección IP de front-end, haga lo siguiente:
1. Al configurar Load Balancer Estándar, active la casilla **Puertos HA** en la configuración de reglas de equilibrador de carga.
2. En **IP flotante**, seleccione **Deshabilitado**.

Esta configuración no permite ninguna otra configuración de reglas de equilibrio de carga en el equilibrador de carga actual. Tampoco permite ninguna otra configuración del equilibrador de carga interno para el conjunto especificado de instancias de back-end.

Pero, además de esta regla de puertos HA, puede configurar una instancia pública de Load Balancer Estándar para las instancias de back-end.

### <a name="a-single-floating-ip-direct-server-return-ha-ports-configuration-on-an-internal-standard-load-balancer"></a>Una configuración de puertos HA con una única dirección IP flotante (Direct Server Return) en una instancia interna de Load Balancer Estándar

De igual forma, puede configurar el equilibrador de carga para usar una regla de equilibrio de carga con **Puerto HA** con un único front-end si establece **IP flotante** en **Habilitado**. 

Esta configuración permite agregar más reglas de equilibrio de carga con IP flotante o un equilibrador de carga público. Pero no puede usar una configuración de equilibrio de carga de puertos HA con IP no flotante encima de esta configuración.

### <a name="multiple-ha-ports-configurations-on-an-internal-standard-load-balancer"></a>Varias configuraciones de puertos HA en una instancia interna de Load Balancer Estándar

Si el escenario requiere que configure más de un front-end de puertos HA para el mismo grupo de back-end, puede hacer lo siguiente: 
- Configure más de una dirección IP privada de front-end para un único recurso interno Load Balancer Standard.
- Configure varias reglas de equilibrio de carga, donde cada una tenga seleccionada una única dirección IP de front-end.
- Seleccione la opción **Puertos HA** y establezca **IP flotante** en **Habilitado** en todas las reglas de equilibrio de carga.

### <a name="an-internal-load-balancer-with-ha-ports-and-a-public-load-balancer-on-the-same-back-end-instance"></a>Un equilibrador de carga interno con puertos HA y un equilibrador de carga público en la misma instancia de back-end

Puede configurar *un* recurso de una instancia pública de Load Balancer estándar para los recursos de back-end, junto con una única instancia interna de Load Balancer estándar con puertos HA.

## <a name="limitations"></a>Limitaciones

- Las reglas de equilibrio de carga de puertos de alta disponibilidad solo están disponibles para la instancia interna de Standard Load Balancer.
- No se admite la combinación de una regla de equilibrio de carga de puertos HA y una regla de equilibrio de carga de puertos no HA que apuntan a las mismas IPconfiguration de back-end, salvo que ambas tengan IP flotante habilitado.
- Los fragmentos IP existentes se reenviarán mediante reglas de equilibrio de carga de puertos de alta disponibilidad al mismo destino que el primer paquete.  No se admite la fragmentación de IP en un paquete UDP o TCP.
- La simetría de flujo (principalmente en escenarios de NVA) se admite con instancias de back-end y una NIC única (y una sola configuración de IP), solo si se usa como se muestra en el diagrama anterior y mediante reglas de equilibrio de carga de los puertos de alta disponibilidad. No se proporciona para ningún otro escenario. Esto significa que dos o más recursos de Load Balancer y sus respectivas reglas toman decisiones independientes y nunca se coordinan. Consulte la descripción y el diagrama de los [dispositivos virtuales de red](#nva). Si usa varias NIC o sitúa la aplicación virtual de red entre una instancia pública y una privada de Load Balancer, la simetría de flujo no está disponible.  Para solucionar este problema, puede alterar el origen del primer paquete del flujo de entrada a la IP de la aplicación para permitir que las respuestas lleguen a la misma NVA.  Sin embargo, se recomienda encarecidamente utilizar una sola NIC y la arquitectura de referencia que se ha mostrado en el diagrama anterior.


## <a name="next-steps"></a>Pasos siguientes

- [Aprenda a configurar los puertos de alta disponibilidad para su ILB mediante Azure Portal](tutorial-load-balancer-standard-internal-portal.md#create-a-load-balancer-rule), [PowerShell](load-balancer-get-started-ilb-arm-ps.md#create-the-configuration-rules-probe-and-load-balancer), [la CLI](load-balancer-get-started-ilb-arm-cli.md#create-the-load-balancer-rule)o [plantillas](load-balancer-get-started-ilb-arm-template.md).
- [Más información sobre Load Balancer estándar](load-balancer-standard-overview.md)
