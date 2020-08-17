---
title: Azure Standard Load Balancer y zonas de disponibilidad
titleSuffix: Azure Load Balancer
description: Con esta ruta de aprendizaje, empiece a usar Azure Standard Load Balancer y Availability Zones.
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/07/2020
ms.author: allensu
ms.openlocfilehash: 55a86eeee4f819955e3f8adfcc0f55f24d58bed0
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420318"
---
# <a name="standard-load-balancer-and-availability-zones"></a>Load Balancer Estándar y zonas de disponibilidad

Azure Standard Load Balancer admite escenarios de zonas de disponibilidad. Puede usar Standard Load Balancer para aumentar la disponibilidad en todo el escenario mediante la alineación de recursos con zonas y su distribución entre ellas. Las zonas de disponibilidad, combinadas con Standard Load Balancer, son un conjunto de características ampliable y flexible que posibilita la creación de diferentes escenarios.  Revise este documento para conocer estos [conceptos](#concepts) y la [guía de diseño](#design) de un escenario básico.

## <a name="availability-zones-concepts-applied-to-load-balancer"></a><a name="concepts"></a> Conceptos de las zonas de disponibilidad aplicados a Load Balancer

Un equilibrador de carga hereda la configuración de zona de sus componentes: 

* Front-end
* Reglas
* Definición de grupo de back-end

En el contexto de las zonas de disponibilidad, el comportamiento y las propiedades de una regla de equilibrador de carga se describen como con redundancia de zona o zonales.  En el contexto del equilibrador de carga, con redundancia de zona siempre significa **varias zonas** y zonal significa aislar el servicio para una **sola zona**. Una instancia de Azure Load Balancer tiene dos tipos: pública e interna. Ambos tipos de equilibrador de carga admiten la redundancia de zona y la implementación zonal.  Además, ambos pueden dirigir el tráfico entre las zonas según sea necesario.

## <a name="frontend"></a>Front-end

Un front-end de equilibrador de carga es una configuración de IP de front-end que hace referencia a una dirección IP pública o privada dentro de la subred de una red virtual. Conforma el punto de conexión con equilibrio de carga en el que se expone su servicio.

Un recurso de equilibrador de carga puede contener reglas con front-end zonales y con redundancia de zona al mismo tiempo. Cuando se garantiza una dirección IP pública o privada para una zona, no se puede modificar la zonalidad (o la ausencia de zonalidad). Para cambiar u omitir la zonalidad de un front-end con dirección IP pública o privada, vuelva a crear la dirección IP en la zona que corresponda. 

Las zonas de disponibilidad no cambian las restricciones para varios servidores front-end. Revise el artículo [varios servidores front-end para Load Balancer](load-balancer-multivip-overview.md) para obtener más información sobre esta funcionalidad.

### <a name="zone-redundant"></a>Redundancia de zona 

En una región con zonas de disponibilidad, un front-end de Standard Load Balancer tiene redundancia de zona. Varias zonas pueden servir como entrada o salida en una región. El tráfico se sirve mediante una dirección IP única. No son necesarios esquemas de redundancia de DNS. 

Una única dirección IP de servidor front-end sobrevivirá a los errores de zona. Se puede usar la IP de front-end para llegar a todos los miembros del grupo de back-end (no afectados), independientemente de la zona. Se puede producir un error en una o más zonas de disponibilidad y la ruta de acceso de datos sobrevive siempre que una zona de la región permanezca correcta. 

La dirección IP del front-end se suministra a la vez en varias implementaciones de infraestructura independientes de varias zonas de disponibilidad. Los reintentos o el restablecimiento se realizarán correctamente en otras zonas a las que no haya afectado por el error de zona. 

<p align="center">
  <img src="./media/az-zonal/zone-redundant-lb-1.svg" alt="Figure depicts a zone-redundant standard load balancer directing traffic in three different zones to three different subnets in a zone redundant configuration." width="512" title="Virtual Network NAT">
</p>

*Ilustración: Equilibrador de carga con redundancia de zona*

### <a name="zonal"></a>Zonal

Puede optar por tener un front-end garantizado para una sola zona, que se conoce como un *front-end zonal*.  Este escenario significa que cualquier flujo de entrada o de salida es atendido por una sola zona en una región.  El front-end comparte destino con el estado de la zona.  La ruta de acceso de datos no se ve afectada por errores en zonas distintas a la garantizada. Puede utilizar front-end zonales para exponer una dirección IP por cada zona de disponibilidad.  

Además, se admite el uso de servidores front-end zonales directamente para puntos de conexión con equilibrio de carga dentro de cada zona. Puede usar esta configuración para exponer puntos de conexión con equilibrio de carga por zona para supervisar de forma individual cada zona. En el caso de los puntos de conexión públicos, puede integrarlos con un producto de equilibrio de carga de DNS como [Traffic Manager](../traffic-manager/traffic-manager-overview.md) y usar un nombre DNS único.


<p align="center">
  <img src="./media/az-zonal/zonal-lb-1.svg" alt="Figure depicts three zonal standard load balancers each directing traffic in a zone to three different subnets in a zonal configuration." width="512" title="Virtual Network NAT">
</p>

*Ilustración: Equilibrador de carga con redundancia de zona*

Si desea combinar estos conceptos (con redundancia de zona y zonal para el mismo back-end), consulte [varios front-end en Azure Load Balancer](load-balancer-multivip-overview.md).

En el caso de un servidor front-end de equilibrador de carga público, se agrega un parámetro de **zonas** a la dirección IP pública. La configuración IP del servidor front-end que usa la regla correspondiente hace referencia a esta dirección IP pública.

En el caso de un servidor front-end de equilibrador de carga interno, se agrega un parámetro de **zonas** a la configuración IP del front-end del recurso de equilibrador de carga interno. Un servidor front-end zonal garantiza una dirección IP en una subred a una zona específica.

## <a name="backend"></a>Back-end

Azure Load Balancer funciona con instancias de máquinas virtuales. Estas pueden ser independientes, conjuntos de disponibilidad o conjuntos de escalado de máquinas virtuales. Cualquier máquina virtual de una red virtual única puede formar parte del grupo de back-end, independientemente de la zona a la que esté asignada la máquina virtual.

Para alinear y garantizar el front-end y el back-end con una sola zona, basta con colocar las máquinas virtuales dentro de la misma zona en el grupo de back-end correspondiente.

Para ubicar las máquinas virtuales a través de varias zonas, coloque las máquinas virtuales de varias zonas en el mismo grupo de back-end. 

Cuando use conjuntos de escalado de máquinas virtuales, ubique uno o más conjuntos de escalado de máquinas virtuales en el mismo grupo de back-end. Los conjuntos de escalado pueden estar en una o varias zonas.

## <a name="outbound-connections"></a>Conexiones de salida

La redundancia de zona y la zonalidad se aplican a las [conexiones salientes](load-balancer-outbound-connections.md). Todas las zonas proporcionan una dirección IP pública con redundancia de zona para las conexiones salientes. Una dirección IP pública zonal solo sirve en la zona en la que está asignada. 

Las asignaciones del puerto SNAT de conexión saliente sobreviven a los errores de zona y el escenario seguirá proporcionando conectividad SNAT saliente si no se ve afectado por un error de zona. Los errores de zona pueden requerir que las conexiones se vuelvan a establecerse para escenarios con redundancia de zona si una zona afectada sirvió el tráfico. Los flujos en zonas distintas de las zonas afectadas no se ven afectados.

El algoritmo de asignación previa de puerto SNAT es el mismo, con o sin zonas de disponibilidad.

## <a name="health-probes"></a>Sondeos de estado

Las definiciones de sondeos de mantenimiento existentes permanecen tal cual estaban sin zonas de disponibilidad. Pero hemos ampliado el modelo de mantenimiento hasta un nivel de infraestructura. 

Cuando se usan servidores front-end con redundancia de zona, el equilibrador de carga expande su comprobación de estado interna. El equilibrador de carga sondea de forma independiente la disponibilidad de una máquina virtual de cada zona y cierra las rutas de acceso de las zonas en las que se produjo un error sin intervención.  

Otras zonas que puedan llegar a esta máquina virtual pueden continuar sirviendo la máquina virtual desde sus front-end respectivos. Durante los eventos de error, cada zona puede tener distribuciones diferentes de nuevos flujos al proteger el mantenimiento general de su servicio.

## <a name="design-considerations"></a><a name="design"></a> Consideraciones de diseño

Load Balancer es flexible en el contexto de las zonas de disponibilidad. Puede optar por alinearse con zonas y por la redundancia de zona para cada regla. Una mayor disponibilidad puede suponer una mayor complejidad. Al diseñar, tenga en mente la disponibilidad para lograr un rendimiento óptimo.

### <a name="zone-redundancy"></a>Redundancia de zona

Load Balancer hace sencillo tener una dirección IP única como un front-end con redundancia de zona. Una dirección IP con redundancia de zona puede servir a un recurso zonal en cualquier zona.  La IP puede sobrevivir a uno o más errores de zona siempre que una de las zonas tenga un estado correcto dentro de la región.  En su lugar, un servidor front-end zonal es una reducción del servicio a una única zona y comparte el mismo destino que la zona correspondiente.

La redundancia de zona no implica una ruta de acceso de datos sin incidencias ni un plano de control; es un plano de datos. Los flujos con redundancia de zona pueden usar cualquier zona y los flujos de un cliente utilizarán todas las zonas correctas de una región. En el caso de un error de zona, los flujos de tráfico que usan zonas correctas no se ven afectados. 

Los flujos de tráfico que están usando una zona en el momento en que se produce un error en esta pueden resultar afectados, pero las aplicaciones se pueden recuperar. El tráfico continúa por las zonas correctas de la región después de la retransmisión cuando Azure ha detectado el error en la zona.

### <a name="cross-zone-boundaries"></a><a name="xzonedesign"></a> Límites entre zonas

Es importante comprender que cuando un servicio cruza zonas, comparte su destino no con una, sino potencialmente con varias zonas. Como resultado, el servicio podría no aumentar su disponibilidad en implementaciones no zonales.

Cuando use zonas de disponibilidad, evite la introducción de dependencias entre zonas no deseadas que anulen las mejoras de disponibilidad. Si la aplicación contiene varios componentes críticos, asegúrese de que sobrevivan a un error de zona.

Un único componente crítico puede afectar a toda la aplicación si solo se encuentra en una zona distinta de las zonas supervivientes. Considere también la restauración de la zona y cómo se restablecerá la aplicación. Obtenga información sobre cómo responde la aplicación a los errores en algunas partes de la misma. Revise los puntos clave y úselos para las preguntas al pensar sobre su escenario concreto.

- Si la aplicación tiene dos componentes:

    * Dirección IP
    * Máquina virtual con disco administrado

Están configurados en las zonas 1 y 2. Cuando se produce un error en la zona 1, el servicio no sobrevive. En los escenarios zonales no se deben cruzar las zonas, a menos que entienda perfectamente que se va a crear un modo de error potencialmente perjudicial. Este escenario puede proporcionar flexibilidad.

- Si la aplicación tiene dos componentes:

    * Dirección IP
    * Máquina virtual con disco administrado

Están configurados para tener redundancia de zona y estar en la zona 1. El servicio sobrevivirá el error de la zona 2, la zona 3 o ambas, a menos que se haya producido un error en la zona 1. No obstante, perderá cierta capacidad para razonar sobre el estado del servicio si solo presta atención a la disponibilidad del servidor front-end.  Considere la posibilidad de desarrollar un modelo de mantenimiento y capacidad más amplio.  Podría usar los conceptos zonal y con redundancia de zona juntos para ampliar su visión y capacidad de administración.

- Si la aplicación tiene dos componentes:

    * Servidor front-end de Load Balancer con redundancia de zona
    * Conjunto de escalado de máquinas virtuales entre tres zonas

Los recursos de las zonas que no sean afectados por el error estarán disponibles. La capacidad del servicio puede disminuir durante el error. Desde una perspectiva de infraestructura, la implementación puede sobrevivir a uno o más errores de zona. En este escenario, surgen las siguientes preguntas:

  - ¿Conoce la respuesta de la aplicación ante estos errores y ante una capacidad disminuida?
  - ¿Necesita contar con medidas de seguridad en el servicio para forzar la conmutación por error a un par de regiones, si es necesario?
  - ¿Cómo serán la supervisión, la detección y la mitigación ante este escenario? Puede usar los diagnósticos de Standard Load Balancer para aumentar la supervisión del rendimiento del servicio. Tenga en cuenta lo que está disponible y lo que puede necesitar de un aumento.

- Las zonas pueden hacer que los errores sean más fáciles de entender y contener. Un error en la zona no es diferente a otros errores en cuanto a tiempos de espera, reintentos y algoritmos de retroceso. Azure Load Balancer proporciona rutas de acceso con redundancia de zona. El equilibrador de carga intenta recuperarse rápidamente en el nivel de paquete en tiempo real. Las retransmisiones o el restablecimiento puede producirse durante la aparición de un error. Es importante comprender cómo se enfrenta la aplicación con los errores. El esquema de equilibrio de carga sobrevivirá, pero debe planear con las siguientes preguntas en mente:

  - Cuando se produce un error en una zona, ¿el servicio lo comprende? Y si se pierde el estado, ¿cómo se recuperará?
  - Cuando vuelve la disponibilidad de una zona, ¿la aplicación sabe cómo realizar una recuperación de forma segura?

Revise [los patrones de diseño en la nube de Azure](https://docs.microsoft.com/azure/architecture/patterns/) para mejorar la resistencia de la aplicación a los escenarios de error.

## <a name="next-steps"></a>Pasos siguientes
- Aprenda más sobre [zonas de disponibilidad](../availability-zones/az-overview.md).
- Más información sobre [Load Balancer Estándar](load-balancer-standard-overview.md)
- Obtenga información sobre cómo [equilibrar la carga de las máquinas virtuales dentro de una zona con Load Balancer estándar con un front-end de zona](load-balancer-standard-public-zonal-cli.md)
- Obtenga información sobre cómo [equilibrar la carga de las máquinas virtuales en distintas zonas con Load Balancer estándar con un front-end con redundancia de zona](load-balancer-standard-public-zone-redundant-cli.md)
- Más información sobre [los patrones de diseño en la nube de Azure](https://docs.microsoft.com/azure/architecture/patterns/) para mejorar la resistencia de la aplicación a los escenarios de error.
