---
title: 'Conceptos: Ingeniería de software sostenible en Azure Kubernetes Services (AKS)'
description: Obtenga información sobre ingeniería de software sostenible en Azure Kubernetes Services (AKS).
services: container-service
ms.topic: conceptual
ms.date: 08/26/2020
ms.openlocfilehash: bc2240b3cb8508109f6dd25c4c80ff31ba197cfd
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89082618"
---
# <a name="sustainable-software-engineering-principals-in-azure-kubernetes-service-aks"></a>Entidades de seguridad de ingeniería de software sostenible en Azure Kubernetes Services (AKS)

Las entidades de seguridad de ingeniería de software sostenible son un conjunto de competencias que le ayudarán a definir, crear y ejecutar aplicaciones sostenibles. El objetivo general es reducir la huella de carbono de cada aspecto de la aplicación. El [proyecto Principles.Green][principals-green] tiene una visión general de las entidades de seguridad de ingeniería de software sostenible.

Una idea importante para comprender el diseño de software sostenible es que es un cambio en las prioridades y el enfoque. En muchos casos, el software se diseña y se ejecuta de forma que se centra en el rendimiento rápido y la baja latencia. La ingeniería de software sostenible se centra en reducir las emisiones de carbono tanto como sea posible. En algunos casos, la aplicación de entidades de seguridad de ingeniería de software sostenible puede proporcionar un rendimiento más rápido o una menor latencia, por ejemplo, reduciendo el recorrido total de la red. En otros casos, reducir las emisiones de carbono puede ralentizar el rendimiento o aumentar la latencia, como el retraso de las cargas de trabajo de prioridad baja. Antes de considerar la posibilidad de aplicar entidades de seguridad de ingeniería de software sostenible a la aplicación, revise las prioridades, las necesidades y las ventajas e inconvenientes de la aplicación.

## <a name="measure-and-optimize"></a>Medición y optimización

Para reducir la huella de carbono de los clústeres de AKS, necesita comprender cómo se usan los recursos del clúster. [Azure Monitor][azure-monitor] proporciona detalles sobre el uso de recursos del clúster, como pueden ser el uso de memoria y la CPU. Estos datos pueden ayudarle a informar de sus decisiones para reducir la huella de carbono del clúster y observar el efecto de los cambios. También puede instalar [Microsoft Sustainability Calculator][sustainability-calculator] para ver la huella de carbono de todos los recursos de Azure.

## <a name="increase-resource-utilization"></a>Aumento de la utilización de recursos

Un enfoque para reducir la huella de carbono es reducir la cantidad de tiempo de inactividad de los recursos de proceso. Reducir el tiempo de inactividad implica aumentar el uso de los recursos de proceso. Por ejemplo, si tuviera cuatro nodos en el clúster y cada uno de ellos se ejecutara al 50 % de su capacidad, los cuatro nodos tendrán un 50 % de capacidad sin usar que queda inactiva. Si ha reducido el clúster a tres nodos, la misma carga de trabajo haría que los tres nodos se ejecutaran al 67 % de su capacidad, lo que reduce la capacidad no usada al 33 % en cada nodo y aumenta el uso.

> [!IMPORTANT]
> Al considerar la posibilidad de realizar cambios en los recursos del clúster, compruebe que los [grupos del sistema][system-pools] tengan suficientes recursos para mantener la estabilidad de los componentes principales del sistema del clúster. No reduzca nunca los recursos del clúster hasta el punto en el que el clúster pueda volverse inestable.

Después de revisar el uso del clúster, considere la posibilidad de usar las características que [varios grupos de nodos][multiple-node-pools] ofrecen. Puede usar el [ajuste de tamaño de nodo][node-sizing] para definir grupos de nodos con perfiles de CPU y de memoria específicos, lo que le permite adaptar los nodos a sus necesidades de carga de trabajo. El ajuste de tamaño de los nodos a sus necesidades de carga de trabajo puede ayudarlo a ejecutar pocos nodos con un uso mayor. También puede configurar el modo en que el clúster [escala][scale] y usar el [escalador automático de pod horizontal][scale-horizontal] y el [escalador automático de clúster][scale-auto] para escalar el clúster automáticamente en función de la configuración. Controlar el modo en que el clúster escala puede ayudarlo a mantener todos los nodos en ejecución con un uso elevado y, al mismo tiempo, mantener los cambios en la carga de trabajo del clúster. En los casos en los que una carga de trabajo es tolerante a interrupciones o finalizaciones repentinas, puede usar [grupos de acceso puntual][spot-pools] para aprovechar la capacidad inactiva en Azure. Por ejemplo, los grupos de acceso puntual pueden funcionar bien para los trabajos por lotes o los entornos de desarrollo.

Aumentar la utilización también puede reducir el exceso de nodos, lo que disminuye la energía consumida por las [reservas de recursos en cada nodo][resource-reservations].

Revise también las *solicitudes* y los *límites* de la CPU y la memoria en los manifiestos de Kubernetes de las aplicaciones. A medida que reduce esos valores para memoria y la CPU, hay más memoria y CPU disponibles para el clúster para ejecutar otras cargas de trabajo. A medida que ejecuta más cargas de trabajo con menor CPU y memoria, su clúster se asigna más densamente, lo que aumenta su utilización. Al reducir la CPU y la memoria para las aplicaciones, el comportamiento de estas se puede degradar o pasar a ser inestable si establece estos valores demasiado bajos. Antes de cambiar las *solicitudes* y los *límites* de la CPU y la memoria, considere la posibilidad de ejecutar algunas pruebas comparativas para saber si estos valores están establecidos correctamente. Además, nunca reduzca estos valores hasta el punto de que la aplicación se vuelva inestable.

## <a name="reduce-network-travel"></a>Reducción del viaje de red

La reducción de la distancia que las solicitudes y respuestas tienen que recorrer hasta el clúster y desde este, generalmente disminuye el consumo de electricidad de los dispositivos de red y las emisiones de carbono. Después de revisar el tráfico de red, considere la posibilidad de crear clústeres [en regiones][regions] más cerca del origen del tráfico de red. También puede usar [Azure Traffic Manager][azure-traffic-manager] para ayudar a enrutar el tráfico al clúster más cercano y los [grupos con ubicación por proximidad][proiximity-placement-groups] para ayudar a reducir la distancia entre los recursos de Azure.

> [!IMPORTANT]
> Al considerar la posibilidad de realizar cambios en las redes del clúster, no reduzca nunca el viaje de red a costa de cumplir los requisitos de la carga de trabajo. Por ejemplo, el uso de [zonas de disponibilidad][availability-zones] provoca más viaje de red en el clúster, pero es posible que sea necesario usar esa característica para controlar los requisitos de la carga de trabajo.

## <a name="demand-shaping"></a>Modelado de la demanda

Siempre que sea posible, considere la posibilidad de cambiar la demanda de los recursos del clúster a las horas o regiones en las que puede usar exceso de capacidad. Por ejemplo, considere la posibilidad de cambiar la hora o la región de un trabajo por lotes para que ejecute o use [grupos de acceso puntual][spot-pools]. Considere también la posibilidad de refactorizar la aplicación para usar una cola para diferir las cargas de trabajo en ejecución que no necesitan un procesamiento inmediato.

## <a name="next-steps"></a>Pasos siguientes

Más información acerca de las siguientes características de AKS mencionadas en este artículo:

* [Varios grupos de nodos][multiple-node-pools]
* [Ajuste de tamaño del nodo][node-sizing]
* [Escala de un clúster][scale]
* [Escalador automático horizontal de pod][scale-horizontal]
* [Cluster Autoscaler][scale-auto]
* [Grupos de acceso puntual][spot-pools]
* [Grupos del sistema][system-pools]
* [Reservas de recursos][resource-reservations]
* [Grupos con ubicación por proximidad][proiximity-placement-groups]
* [Zonas de disponibilidad][availability-zones]

[availability-zones]: availability-zones.md
[azure-monitor]: ../azure-monitor/insights/container-insights-overview.md
[azure-traffic-manager]: ../traffic-manager/traffic-manager-overview.md
[proiximity-placement-groups]: reduce-latency-ppg.md
[regions]: faq.md#which-azure-regions-currently-provide-aks
[resource-reservations]: concepts-clusters-workloads.md#resource-reservations
[scale]: concepts-scale.md
[scale-auto]: concepts-scale.md#cluster-autoscaler
[scale-horizontal]: concepts-scale.md#horizontal-pod-autoscaler
[spot-pools]: spot-node-pool.md
[multiple-node-pools]: use-multiple-node-pools.md
[node-sizing]: use-multiple-node-pools.md#specify-a-vm-size-for-a-node-pool
[sustainability-calculator]: https://azure.microsoft.com/blog/microsoft-sustainability-calculator-helps-enterprises-analyze-the-carbon-emissions-of-their-it-infrastructure/
[system-pools]: use-system-pools.md
[principals-green]: https://principles.green/