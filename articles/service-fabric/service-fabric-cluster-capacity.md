---
title: Consideraciones de planeación de capacidad del clúster de Service Fabric
description: Tipos de nodos, durabilidad, confiabilidad y otros aspectos que se deben tener en cuenta al planear el clúster de Service Fabric.
ms.topic: conceptual
ms.date: 05/21/2020
ms.author: pepogors
ms.custom: sfrev
ms.openlocfilehash: 28a01bbc54f752ffc1f25b57dcf2eca566aa635a
ms.sourcegitcommit: 6fc156ceedd0fbbb2eec1e9f5e3c6d0915f65b8e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88718108"
---
# <a name="service-fabric-cluster-capacity-planning-considerations"></a>Consideraciones de planeación de capacidad del clúster de Service Fabric

El planeamiento de la capacidad del clúster es importante para cada entorno de producción de Service Fabric. Las consideraciones clave incluyen:

* **Número inicial y propiedades de los *tipos de nodo*** de clúster

* Nivel de ***durabilidad* de cada tipo de nodo**, que determina los privilegios de Service Fabric VM en la infraestructura de Azure

* Nivel de ***confiabilidad* del clúster**, que determina la estabilidad de los servicios del sistema de Service Fabric la función general del clúster

Este artículo le guiará a través de los puntos de decisión significativos para cada una de estas áreas.

## <a name="initial-number-and-properties-of-cluster-node-types"></a>Número inicial y propiedades de los tipos de nodo de clúster

Un *tipo de nodo* define el tamaño, el número y las propiedades de un conjunto de nodos (máquinas virtuales) en el clúster. Cada tipo de nodo que se define en un clúster de Service Fabric se asigna a con un [conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/overview.md).

Dado que cada tipo de nodo es un conjunto de escalado distinto, se puede escalar o reducir verticalmente de forma independiente, tener diferentes conjuntos de puertos abiertos y tener distintas métricas de capacidad. Para más información sobre la relación entre los tipos de nodo y los conjuntos de escalado de máquinas virtuales, consulte los [tipos de nodo de clúster de Service Fabric](service-fabric-cluster-nodetypes.md).

Cada clúster requiere un **tipo de nodo principal**, que ejecuta servicios críticos del sistema que proporcionan capacidades de la plataforma Service Fabric. Aunque también es posible usar los tipos de nodo principal para ejecutar las aplicaciones, se recomienda dedicarlos únicamente a la ejecución de servicios del sistema.

**Los tipos de nodo no principales** pueden usarse para definir roles de aplicación (como *front-end* y servicios de *back-end*) y aislar físicamente los servicios dentro de un clúster. Los clústeres de Service Fabric pueden tener cero o más tipos de nodo no principal.

El tipo de nodo principal se configura con el atributo `isPrimary` de la definición de tipo de nodo en la plantilla de implementación de Azure Resource Manager. Vea el [objeto NodeTypeDescription](/azure/templates/microsoft.servicefabric/clusters#nodetypedescription-object) para obtener la lista completa de las propiedades de tipo de nodo. Por ejemplo, use cualquier archivo *AzureDeploy.json* en [ejemplos de clúster de Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/) y *Buscar en la página* busque el objeto `nodetTypes`.

### <a name="node-type-planning-considerations"></a>Consideraciones de planeación de tipos de nodo

El número de tipos de nodos iniciales depende del propósito de su clúster y de las aplicaciones y servicios que se ejecutan en él. Tenga en cuenta las preguntas siguientes:

* ***¿La aplicación tiene varios servicios y cualquiera de ellos debe ser público o accesible desde Internet?***

    Las aplicaciones típicas contienen un servicio front-end de puerta de enlace que recibe la entrada de un cliente, así como uno o varios servicios back-end que se comunican con los servicios front-end, con redes independientes entre los servicios front-end y back-end. Estos casos suelen requerir tres tipos de nodo: un tipo de nodo principal y dos tipos de nodo no principales (uno para el servicio front-end y back-end).

* ***¿Los servicios que componen la aplicación tienen diferentes necesidades de infraestructura, como una RAM mayor o más ciclos de CPU?***

    A menudo, el servicio front-end puede ejecutarse en máquinas virtuales más pequeñas (tamaños de máquina virtual como D2) que tienen puertos abiertos a Internet.  Es posible que los servicios back-end de cómputo intensivo deban ejecutarse en máquinas virtuales más grandes (con tamaños de máquina virtual como D4, D6, D15) que no son accesibles desde Internet. La definición de distintos tipos de nodo para estos servicios le permite hacer un uso más eficaz y seguro de las máquinas virtuales Service Fabric subyacentes y les permite escalarlas de forma independiente. Para más información sobre la estimación de la cantidad de recursos que necesita, consulte [Planeamiento de capacidad para aplicaciones de Service Fabric](service-fabric-capacity-planning.md)

* ***¿Alguno de sus servicios de aplicación necesitará escalar más allá de 100 nodos?***

    Un tipo de nodo único no puede escalar de manera confiable más allá de 100 nodos por conjunto de escalado de máquinas virtuales para aplicaciones Service Fabric. La ejecución de más de 100 nodos requiere conjuntos de escalado de máquinas virtuales adicionales (y por consiguiente, tipos de nodo adicionales).

* ***¿El clúster abarcará Availability Zones?***

    Service Fabric admite clústeres que abarcan [Availability Zones](../availability-zones/az-overview.md) con la implementación de tipos de nodo anclados a zonas específicas, lo que garantiza una alta disponibilidad de las aplicaciones. Availability Zones requiere planeación de tipo de nodo adicional y requisitos mínimos. Para más información, consulte [Topología recomendada para el tipo de nodo principal de clústeres de Service Fabric que se distribuyen en Availability Zones](service-fabric-cross-availability-zones.md#recommended-topology-for-primary-node-type-of-azure-service-fabric-clusters-spanning-across-availability-zones). 

A la hora de determinar el número y las propiedades de los tipos de nodo para la creación inicial del clúster, tenga en cuenta que siempre se pueden agregar, modificar o quitar tipos de nodo (no principales) una vez implementado el clúster. [Los tipos de nodo principal también se pueden modificar](service-fabric-scale-up-primary-node-type.md) en clústeres en ejecución (aunque tales operaciones requieren una gran cantidad de planeación y precaución en entornos de producción).

Una consideración más detallada para las propiedades de tipo de nodo es el nivel de durabilidad, que determina los privilegios que tienen las máquinas virtuales de un tipo de nodo dentro de la infraestructura de Azure. Use el tamaño de las máquinas virtuales que elija para el clúster y el recuento de instancias que asigna para los tipos de nodo individuales para ayudar a determinar el nivel de durabilidad adecuado para cada uno de los tipos de nodo, tal y como se describe enseguida.

## <a name="durability-characteristics-of-the-cluster"></a>Características de durabilidad del clúster

El *nivel de durabilidad* designa los privilegios que tienen las máquinas virtuales de Service Fabric con la infraestructura subyacente de Azure. Este privilegio permite a Service Fabric pausar cualquier solicitud de infraestructura de nivel de máquina virtual (por ejemplo, reinicio, restablecimiento de imagen inicial o migración) que afecte los requisitos de quórum para los servicios del sistema de Service Fabric y sus servicios con estado.

> [!IMPORTANT]
> El nivel de durabilidad se establece por tipo de nodo. Si no se especifica ninguno, se usará el nivel *Bronce*, pero no proporciona actualizaciones automáticas del sistema operativo. La durabilidad de *Plata* u *Oro* se recomienda para las cargas de trabajo de producción.

En la siguiente tabla se enumeran los niveles de durabilidad de Service Fabric, sus requisitos y las prestaciones.

| Nivel de durabilidad  | Número mínimo de máquinas virtuales | Tamaños de máquina virtual admitidos                                                                  | Actualizaciones realizadas en el conjunto de escalado de máquinas virtuales                               | Actualizaciones y mantenimiento iniciados por Azure                                                              | 
| ---------------- |  ----------------------------  | ---------------------------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Oro             | 5                              | Tamaños de nodo completo dedicados a un solo cliente (por ejemplo, L32s, GS5, G5, DS15_v2, D15_v2) | Se puede retrasar hasta que lo apruebe el clúster de Service Fabric | Se puede pausar durante 2 horas por dominio de actualización para permitir el tiempo adicional para que las réplicas se recuperen de errores anteriores |
| Plata           | 5                              | Máquinas virtuales de un solo núcleo o varios con al menos 50 GB de SSD local                      | Se puede retrasar hasta que lo apruebe el clúster de Service Fabric | No se puede retrasar una cantidad de tiempo significativa                                                    |
| Bronce          | 1                              | Máquinas virtuales con al menos 50 GB de SSD local                                              | El clúster de Service Fabric no lo retrasará           | No se puede retrasar una cantidad de tiempo significativa                                                    |

> [!WARNING]
> La actualización automática de la imagen del sistema operativo no está disponible con durabilidad Bronce. Si bien la [Aplicación de orquestación de parches](service-fabric-patch-orchestration-application.md) (pensada solo para clústeres alojados que no sean de Azure) *no se recomienda* para niveles de durabilidad Plata o mayores, es su única opción para automatizar las actualizaciones de Windows con respecto a la actualización de dominios de Service Fabric.

> [!IMPORTANT]
> Independientemente del nivel de durabilidad, la ejecución de una operación de [Desasignación](/rest/api/compute/virtualmachinescalesets/deallocate) en un conjunto de escalado de máquinas virtuales destruirá el clúster.

### <a name="bronze"></a>Bronce

Los tipos de nodos que se ejecutan con durabilidad Bronce no obtienen ningún privilegio. Esto significa que los trabajos de infraestructura que afectan a las cargas de trabajo con estado no se detendrán ni se retrasarán. Use la durabilidad Bronce para los tipos de nodo que solo ejecutan cargas de trabajo sin estado. Para las cargas de trabajo de producción, se recomienda utilizar el nivel Silver o superiores.

### <a name="silver-and-gold"></a>Plata y Oro

Use la durabilidad Plata u Oro para todos los tipos de nodo que hospedan servicios con estado que espera que se escalen con frecuencia y donde desea que las operaciones de implementación se retrasen y se reduzca la capacidad en favor de simplificar el proceso. Los escenarios de escalado horizontal no deben afectar a la elección del nivel de durabilidad.

#### <a name="advantages"></a>Ventajas

* Reduce el número de pasos necesarios para las operaciones de escalado horizontal (se llama automáticamente a la desactivación de nodos y a Remove-ServiceFabricNodeState).
* Reduce el riesgo de pérdida de datos debido a las operaciones de cambio de tamaño de máquina virtual en contexto y a las operaciones de infraestructura de Azure.

#### <a name="disadvantages"></a>Inconvenientes

* Las implementaciones en conjuntos de escalado de máquinas virtuales y otros recursos de Azure relacionados pueden agotar el tiempo de espera, retrasar o bloquearse completamente por problemas en el clúster o en el nivel de infraestructura.
* Aumenta el número de [eventos de los ciclos de vida de las réplicas](service-fabric-reliable-services-lifecycle.md) (p. ej. intercambios principales) debidos a las desactivaciones automáticas de nodos durante las operaciones en la infraestructura de Azure.
* Deja los nodos fuera de servicio durante los períodos de tiempo en los que tienen lugar las actualizaciones de software de la plataforma de Azure o las actividades de mantenimiento de hardware. Durante estas actividades, puede ver los nodos con un estado que indica que se están deshabilitando o que están deshabilitados. Esto reduce la capacidad del clúster temporalmente, pero no debería afectar a la disponibilidad de dicho clúster o de las aplicaciones.

#### <a name="best-practices-for-silver-and-gold-durability-node-types"></a>Procedimientos recomendados para los tipos de nodos de durabilidad Plata y Oro

Siga estas recomendaciones para administrar tipos de nodo con durabilidad Plata u Oro:

* Mantenga el clúster y las aplicaciones en buen estado en todo momento, y asegúrese de que las aplicaciones responden a todos los [eventos de los ciclos de vida de las réplicas del servicio](service-fabric-reliable-services-lifecycle.md) (como que la réplica en compilación está bloqueada) en el momento adecuado.
* Adopte formas más seguras de cambiar el tamaño de una máquina virtual (escalar o reducir verticalmente). Cambiar el tamaño de la máquina virtual de un conjunto de escalado de máquinas virtuales requiere una cuidadosa planeación y precaución. Para más información, consulte [Escalado vertical de un tipo de nodo de Azure Service Fabric](service-fabric-scale-up-primary-node-type.md)
* Mantenga un mínimo de cinco nodos en todos los conjuntos de escalado de máquinas virtuales que tengan habilitados los niveles de durabilidad Gold o Silver. El clúster entrará en estado de error si escala por debajo de este umbral y deberá limpiar manualmente el estado (`Remove-ServiceFabricNodeState`) de los nodos eliminados.
* Cada conjunto de escalado de máquinas virtuales con el nivel de durabilidad Silver o Gold tiene que asignarse a su propio tipo de nodo en el clúster de Service Fabric. La asignación de varios conjuntos de escalado de máquinas virtuales a un único tipo de nodo impedirá que la coordinación entre el clúster de Service Fabric y la infraestructura de Azure funcione correctamente.
* No elimine instancias de máquina virtual aleatorias, use siempre la característica de reducción horizontal del conjunto de escalado de máquinas virtuales. La eliminación de instancias de máquina virtual aleatorias tiene la posibilidad de crear desequilibrios en la instancia de máquina virtual distribuida en [dominios de actualización](service-fabric-cluster-resource-manager-cluster-description.md#upgrade-domains) y [dominios de error](service-fabric-cluster-resource-manager-cluster-description.md#fault-domains). Este desequilibrio puede afectar negativamente a la capacidad de los sistemas de equilibrar la carga correctamente entre las instancias de servicio o las réplicas de servicio.
* Si se usa el escalado automático, establezca las reglas de modo que se realicen las operaciones de escalado (eliminación de instancias de máquina virtual) en un solo nodo cada vez. No es seguro reducir verticalmente más de una instancia a la vez.
* Si elimina o desasigna máquinas virtuales en el tipo de nodo principal, nunca reduzca el número de máquinas virtuales asignadas por debajo de lo que requiere el nivel de confiabilidad. Estas operaciones se bloquearán indefinidamente en un conjunto de escalado con un nivel de durabilidad Silver o Gold.

### <a name="changing-durability-levels"></a>Cambio de los niveles de durabilidad

Dentro de ciertas restricciones, el nivel de durabilidad del tipo de nodo se puede ajustar:

* Los tipos de nodo con los niveles de durabilidad Plata u Oro no se pueden degradar a Bronce.
* El cambio de bronce a plata u oro tarda unas horas.
* Al cambiar el nivel de durabilidad, asegúrese de actualizarlo tanto en la configuración de la extensión de Service Fabric del recurso del conjunto de escalado de máquinas virtuales como en la definición del tipo de nodo del recurso de clúster de Service Fabric. Estos valores deben coincidir.

Otro aspecto que se debe tener en cuenta en el planeamiento de capacidad es el nivel de confiabilidad del clúster, que determina la estabilidad de los servicios del sistema y el clúster global, como se describe en la siguiente sección.

## <a name="reliability-characteristics-of-the-cluster"></a>Características de confiabilidad del clúster

El *nivel de confiabilidad* del clúster determina el número de réplicas de servicios del sistema que se ejecutan en el tipo de nodo principal del clúster. Cuanto mayor sea el número de réplicas, más confiables serán los servicios del sistema (y por lo tanto, el clúster como un todo).

> [!IMPORTANT]
> El nivel de confiabilidad se establece en el nivel de clúster y determina el número mínimo de nodos del tipo de nodo principal. Las cargas de trabajo de producción requieren un nivel de confiabilidad Plata (mayor o igual que cinco nodos) o superior.  

El nivel de confiabilidad puede adoptar los siguientes valores:

* **Platinum**: los servicios del sistema se ejecutan con el recuento de nueve del conjunto de réplicas.
* **Gold**: los servicios del sistema se ejecutan con el recuento de siete del conjunto de réplicas.
* **Silver**: los servicios del sistema se ejecutan con el recuento de cinco del conjunto de réplicas.
* **Bronze**: los servicios del sistema se ejecutan con el recuento de tres del conjunto de réplicas.

Esta es la recomendación sobre cómo elegir el nivel de confiabilidad. El número de nodos raíz también se establece en el número mínimo de nodos para un nivel de confiabilidad.


| **Número de nodos** | **Nivel de confiabilidad** |
| --- | --- |
| 1 | *No especifique el parámetro de `reliabilityLevel`: el sistema lo calcula.* |
| 3 | Bronce |
| 5 o 6| Plata |
| 7 u 8 | Oro |
| 9 y superiores | Platinum |

Al aumentar o disminuir el tamaño del clúster (la suma de las instancias de máquina virtual en todos los tipos de nodo), considere la posibilidad de actualizar la confiabilidad del clúster de un nivel a otro. Esto desencadena las actualizaciones de clúster necesarias para cambiar el recuento de conjunto de réplicas de los servicios del sistema. Espere a que finalice la actualización en curso antes de realizar otros cambios en el clúster, como agregar nodos.  Puede supervisar el progreso de la actualización en Service Fabric Explorer o puede ejecutar [Get-ServiceFabricClusterUpgrade](/powershell/module/servicefabric/get-servicefabricclusterupgrade?view=azureservicefabricps).

### <a name="capacity-planning-for-reliability"></a>Planeamiento de capacidad de confiabilidad

Las necesidades de capacidad del clúster se determinarán según sus requisitos específicos de carga de trabajo y confiabilidad. En esta sección se proporcionan una guía general para ayudarlo a empezar con el planeamiento de capacidad.

#### <a name="virtual-machine-sizing"></a>Tamaño de la máquina virtual

**En las cargas de trabajo de producción, el tamaño de máquina virtual recomendado (SKU) es [Standard D2_V2](../virtual-machines/dv2-dsv2-series.md) (o equivalente) con un mínimo de 50 GB de SSD local, 2 núcleos y 4 GiB de memoria.** Se recomienda un mínimo de 50 GB de SSD local, pero algunas cargas de trabajo (como las que ejecutan contenedores de Windows) requerirán discos de mayor tamaño. Al elegir otros [tamaños de máquina virtual](../virtual-machines/sizes-general.md) para cargas de trabajo de producción, tenga en cuenta las siguientes restricciones:

- No se admiten tamaños de máquina virtual de base parcial como A0 estándar.
- *La serie A* de tamaños de máquina virtual no se admiten por motivos de rendimiento.
- No se admiten máquinas virtuales de prioridad baja.

#### <a name="primary-node-type"></a>Tipo de nodo principal

**Las cargas de trabajo de producción** en Azure requieren un mínimo de cinco nodos principales (instancias de máquina virtual) y el nivel de confiabilidad Plata. Se recomienda dedicar el tipo de nodo principal del clúster a los servicios del sistema y usar restricciones de selección de ubicación para implementar la aplicación en los tipos de nodo secundarios.

**Las cargas de trabajo de prueba** en Azure pueden ejecutar un mínimo de uno o tres nodos principales. Para configurar un clúster de un nodo, asegúrese de que la configuración de `reliabilityLevel` se ha omitido por completo en la plantilla del Administrador de recursos (la especificación de un valor de cadena vacía para `reliabilityLevel` no es suficiente). Si configura el clúster de un nodo configurado con Azure Portal, esta configuración se realiza automáticamente.

> [!WARNING]
> Los clústeres de un nodo se ejecutan con una configuración especial sin confiabilidad y donde no se admite la escalabilidad horizontal.

#### <a name="non-primary-node-types"></a>Tipos de nodos no principales

El número mínimo de nodos para un tipo de nodo no principal depende del [nivel de durabilidad](#durability-characteristics-of-the-cluster) específico del tipo de nodo. Debe planear el número de nodos (y el nivel de durabilidad) en función del número de réplicas de aplicaciones o servicios que desea ejecutar para el tipo de nodo, y en función de si la carga de trabajo es con estado o sin estado. Tenga en cuenta que puede aumentar o disminuir el número de máquinas virtuales en un tipo de nodo en cualquier momento después de haber implementado el clúster.

##### <a name="stateful-workloads"></a>Cargas de trabajo con estado

En el caso de cargas de trabajo de producción con estado con Service Fabric [colección de confianza o actor de confianza](service-fabric-choose-framework.md), se recomienda un número mínimo de réplicas de destino de cinco. Con esto, en estado estable, termina con una réplica (de un conjunto de réplicas) en cada dominio de error y dominio de actualización. En general, use el nivel de confiabilidad que estableció para los servicios del sistema como guía para el número de réplicas que usa para los servicios con estado.

##### <a name="stateless-workloads"></a>Cargas de trabajo sin estado

En las cargas de trabajo de producción sin estado, el tamaño mínimo admitido del tipo de nodo no principal es tres para mantener el cuórum, pero se recomienda un tamaño de tipo de nodo de cinco.

## <a name="next-steps"></a>Pasos siguientes

Antes de configurar el clúster, revise las `Not Allowed` [directivas de actualización de clúster](service-fabric-cluster-fabric-settings.md) para mitigar la necesidad de volver a crear el clúster más adelante debido a las opciones de configuración del sistema que, de otro modo, no cambian.

Para más información sobre la planeación de clústeres, consulte:

* [Planeación y escalado de proceso](service-fabric-best-practices-capacity-scaling.md)
* [Planeación de la capacidad para las aplicaciones de Service Fabric](service-fabric-capacity-planning.md)
* [Planeamiento de recuperación ante desastres](service-fabric-disaster-recovery.md)

<!--Image references-->
[SystemServices]: ./media/service-fabric-cluster-capacity/SystemServices.png
