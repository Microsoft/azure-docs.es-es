---
title: Diseño de redes virtuales con recursos de puertas de enlace de NAT
titleSuffix: Azure Virtual Network NAT
description: Aprenda a diseñar redes virtuales con los recursos de puertas de enlace de NAT.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: nat
Customer intent: As an IT administrator, I want to learn more about how to design virtual networks with NAT gateway resources.
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/11/2020
ms.author: allensu
ms.openlocfilehash: f6e0009a1e1df57298884097cac076ef3a344714
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88135834"
---
# <a name="designing-virtual-networks-with-nat-gateway-resources"></a>Diseño de redes virtuales con recursos de puertas de enlace de NAT

Los recursos de puerta de enlace de NAT forman parte de [Virtual Network NAT](nat-overview.md) y proporcionan conectividad saliente a Internet para una o varias subredes de una red virtual. La subred de la red virtual indica qué puerta de enlace de NAT se usará. NAT proporciona traducción de direcciones de red de origen (SNAT) para una subred.  Los recursos de puerta de enlace de NAT especifican las direcciones IP estáticas que usan las máquinas virtuales al crear flujos de salida. Las direcciones IP estáticas proceden de recursos de direcciones IP públicas, recursos de prefijos IP públicos, o ambos. Si se usa un recurso de prefijo de dirección IP pública, un recurso de puerta de enlace NAT consume todas las direcciones IP de todos los recursos con prefijo de dirección IP pública. Un recurso de puerta de enlace de NAT puede usar un máximo de 16 direcciones IP desde cualquiera de ellas.


<p align="center">
  <img src="media/nat-overview/flow-direction1.svg" alt="Figure depicts a NAT gateway resource that consumes all IP addresses for a public IP prefix and directs that traffic to and from two subnets of virtual machines and a virtual machine scale set." width="256" title="Virtual Network NAT para la salida a Internet">
</p>

*Ilustración: Virtual Network NAT para la salida a Internet*

## <a name="how-to-deploy-nat"></a>Implementación de NAT

La configuración y el uso de la puerta de enlace de NAT se ha hecho sencilla a propósito:  

Recurso de puerta de enlace de NAT:
- Cree un recurso de puerta de enlace de NAT regional o con aislamiento de zona.
- Asigne direcciones IP.
- Si es necesario, modifique el tiempo de espera de inactividad de TCP (opcional).  Revise los [temporizadores](#timers) <ins>antes de</ins> cambiar el valor predeterminado.

Red virtual:
- Configure una subred de red virtual para que use una puerta de enlace de NAT.

No se necesitan rutas definidas por el usuario.

## <a name="resource"></a>Recurso

El recurso se ha diseñado para que sea muy sencillo, como se puede ver en el siguiente ejemplo de Azure Resource Manager en un formato de tipo plantilla.  Este formato se muestra aquí para ilustrar los conceptos y la estructura.  Modifique el ejemplo para adecuarlo a sus necesidades.  No se pretende que este documento sea un tutorial.

En el siguiente diagrama se muestran las referencias en cuanto a lo que se puede escribir entre los distintos recursos de Azure Resource Manager.  La flecha indica la dirección de la referencia y parte del lugar desde el que se puede escribir. Revisar 

<p align="center">
  <img src="media/nat-overview/flow-map.svg" alt="Figure depicts a NAT receiving traffic from internal subnets and directing it to a public IP and an IP prefix." width="256" title="Modelo de objetos de Virtual Network NAT">
</p>

*Ilustración: Modelo de objetos de Virtual Network NAT*

NAT se recomienda para la mayoría de las cargas de trabajo, salvo que se tenga una dependencia concreta de la [conectividad de salida de Load Balancer basada en grupos](../load-balancer/load-balancer-outbound-connections.md).  

Puede migrar desde escenarios del equilibrador de carga estándar, incluidas las [reglas salientes](../load-balancer/load-balancer-outbound-rules-overview.md), a una puerta de enlace de NAT. Para realizar la migración, mueva los recursos de IP pública y de prefijo de IP pública desde los servidores front-end de Load Balancer a la puerta de enlace de NAT. No se requieren direcciones IP nuevas para la puerta de enlace de NAT. Se pueden reutilizar tanto los recursos de la dirección IP pública como el recurso del prefijo de la dirección IP pública, siempre que el total no supere las 16 direcciones IP. Planee la migración y tenga en cuenta la interrupción del servicio durante la transición.  Si el proceso se automatiza, el periodo de interrupción se reduce considerablemente. Pruebe la migración en un entorno de ensayo primero.  Durante la transición, los flujos de entrada originados no resultan afectados.


El siguiente ejemplo es un fragmento de código de una plantilla de Azure Resource Manager.  Esta plantilla implementa varios recursos, incluida una puerta de enlace NAT.  La plantilla tiene los parámetros siguientes en este ejemplo:

- **natgatewayname**: nombre de la puerta de enlace NAT.
- **location**: región de Azure donde se encuentra el recurso.
- **publicipname**: nombre de la dirección IP pública saliente asociada a la puerta de enlace NAT.
- **vnetname**: nombre de la red virtual.
- **subnetname**: nombre de la subred asociada a la puerta de enlace NAT.

El número total de direcciones IP proporcionado por todas las direcciones IP y recursos de prefijo no pueden superar un total de 16 direcciones IP. Se permite cualquier número de direcciones IP entre 1 y 16.

:::code language="json" source="~/quickstart-templates/101-nat-gateway-vnet/azuredeploy.json" range="81-96":::

Cuando se haya creado el recurso de puerta de enlace de NAT, se puede usar en una o varias subredes de una red virtual. Especifique qué subredes usan este recurso de puerta de enlace de NAT. Una puerta de enlace de NAT no puede abarcar más de una red virtual. No es preciso asignar la misma puerta de enlace de NAT a todas las subredes de una red virtual. Se pueden configurar subredes individuales con diferentes recursos de puerta de enlace de NAT.

Los escenarios que no usen zonas de disponibilidad serán regionales (sin zona especificada). Si usa zonas de disponibilidad, puede especificar una de ellas para aislar NAT en una zona concreta. No se admite la redundancia de zonas. Examine las [zonas de disponibilidad](#availability-zones) de NAT.

:::code language="json" source="~/quickstart-templates/101-nat-gateway-vnet/azuredeploy.json" range="1-146" highlight="81-96":::

Las puertas de enlace de NAT se definen con una propiedad en una subred de una red virtual. Los flujos que creen las máquinas virtuales en la subred **subnetname** de la red virtual **vnetname** usarán la puerta de enlace NAT. Toda la conectividad de salida usará las direcciones IP asociadas con **natgatewayname** como dirección IP de origen.

Para más información sobre la plantilla de Azure Resource Manager usada en este ejemplo, consulte:

- [Inicio rápido: Creación de una puerta de enlace NAT: plantilla de Resource Manager](quickstart-create-nat-gateway-template.md)
- [Virtual Network NAT](https://azure.microsoft.com/resources/templates/101-nat-gateway-1-vm/)

## <a name="design-guidance"></a>Guía de diseño

Lea esta sección para familiarizarse con las consideraciones para diseñar redes virtuales con NAT.  

1. [Optimización de costos](#cost-optimization)
1. [Coexistencia de entrada y salida](#coexistence-of-inbound-and-outbound)
2. [Administración de recursos básicos](#managing-basic-resources)
3. [Zonas de disponibilidad](#availability-zones)

### <a name="cost-optimization"></a>Optimización de costos

Los [puntos de conexión de servicio](virtual-network-service-endpoints-overview.md) y el [vínculo privado](../private-link/private-link-overview.md) son opciones que deben tenerse en cuenta para optimizar el costo. No es necesario NAT para estos servicios. La NAT de la red virtual no procesa el tráfico dirigido a los puntos de conexión de servicio o a un vínculo privado.  

Los puntos de conexión de servicio enlazan los recursos de servicio de Azure a su red virtual y controlan el acceso a los recursos de servicio de Azure. Por ejemplo, cuando se accede a Azure Storage, se usa un punto de conexión de servicio para almacenamiento, con el fin de evitar los cargos de NAT de datos procesados. Los puntos de conexión de servicio con gratuitos.

Un vínculo privado expone el servicio PaaS de Azure (o cualquier otro servicio hospedado con vínculo privado) como punto de conexión privado dentro de una red virtual.  Un vínculo privado se factura en función de la duración y de los datos procesados.

Evalúe si alguno de estos dos métodos (o ambos) se adapta bien a su escenario y úselo según sea necesario.

### <a name="coexistence-of-inbound-and-outbound"></a>Coexistencia de entrada y salida

La puerta de enlace de NAT es compatible con:

 - Equilibrador de carga estándar
 - IP pública estándar
 - Prefijo de IP pública estándar

Cuando desarrolle una implementación nueva, comience con SKU estándar.

<p align="center">
  <img src="media/nat-overview/flow-direction1.svg" alt="Figure depicts a NAT gateway that supports outbound traffic to the internet from a virtual network." width="256" title="Virtual Network NAT para la salida a Internet">
</p>

*Ilustración: Virtual Network NAT para la salida a Internet*

El escenario de solo salida a Internet que proporciona la puerta de enlace de NAT se puede expandir con la funcionalidad de entrada desde Internet. Todos los recursos saben la dirección en que se origina un flujo. En una subred con una puerta de enlace de NAT, esta sustituye todos los escenarios de salida a Internet. Por su parte, los escenarios de entrada desde Internet los proporciona el recurso respectivo.

#### <a name="nat-and-vm-with-instance-level-public-ip"></a>NAT y máquina virtual con IP pública de nivel de instancia

<p align="center">
  <img src="media/nat-overview/flow-direction2.svg" alt="Figure depicts a NAT gateway that supports outbound traffic to the internet from a virtual network and inbound traffic with an instance-level public IP." width="300" title="Virtual Network NAT y máquina virtual con IP pública de nivel de instancia">
</p>

*Ilustración: Virtual Network NAT y máquina virtual con IP pública de nivel de instancia*

| Dirección | Recurso |
|:---:|:---:|
| Entrada | Máquina virtual con IP pública de nivel de instancia |
| Salida | NAT Gateway |

La máquina virtual usará una puerta de enlace de NAT para la salida.  La entrada originada no resulta afectada.

#### <a name="nat-and-vm-with-public-load-balancer"></a>NAT y máquina virtual con equilibrador de carga público

<p align="center">
  <img src="media/nat-overview/flow-direction3.svg" alt="Figure depicts a NAT gateway that supports outbound traffic to the internet from a virtual network and inbound traffic with a public load balancer." width="350" title="Virtual Network NAT y máquina virtual con equilibrador de carga público">
</p>

*Ilustración: Virtual Network NAT y máquina virtual con equilibrador de carga público*

| Dirección | Recurso |
|:---:|:---:|
| Entrada | Equilibrador de carga público |
| Salida | NAT Gateway |

La puerta de enlace de NAT sustituye la configuración de salida de una regla de equilibrio de carga o las reglas de salida.  La entrada originada no resulta afectada.

#### <a name="nat-and-vm-with-instance-level-public-ip-and-public-load-balancer"></a>NAT y máquina virtual con IP pública de nivel de instancia y equilibrador de carga pública

<p align="center">
  <img src="media/nat-overview/flow-direction4.svg" alt="Figure depicts a NAT gateway that supports outbound traffic to the internet from a virtual network and inbound traffic with an instance-level public IP and a public load balancer." width="425" title="Virtual Network NAT y máquina virtual con IP pública de nivel de instancia y equilibrador de carga pública">
</p>

*Ilustración: Virtual Network NAT y máquina virtual con IP pública de nivel de instancia y equilibrador de carga pública*

| Dirección | Recurso |
|:---:|:---:|
| Entrada | Máquina virtual con IP pública de nivel de instancia y equilibrador de carga pública |
| Salida | NAT Gateway |

La puerta de enlace de NAT sustituye la configuración de salida de una regla de equilibrio de carga o las reglas de salida.  La máquina virtual también usará una puerta de enlace de NAT para la salida.  La entrada originada no resulta afectada.

### <a name="managing-basic-resources"></a>Administración de recursos básicos

El equilibrador de carga estándar, la IP pública y el prefijo de IP pública son compatibles con la puerta de enlace de NAT. Las puertas de enlace de NAT operan en el ámbito de una subred. La SKU básica de estos servicios se debe implementar en una subred sin una puerta de enlace de NAT. Esta separación permite que las dos variantes de SKU coexistan en la misma red virtual.

Las puertas de enlace de NAT tienen prioridad sobre los escenarios de salida de la subred. El equilibrador de carga básico o la IP pública (y todos los servicios administrados integrados con ellos) no pueden ajustarse con las traducciones correctas. La puerta de enlace de NAT toma el control sobre el tráfico de salida a Internet en una subred. El tráfico de entrada al equilibrador de carga básico y a la IP pública no está disponible. El tráfico de entrada a un equilibrador de carga básico o a una IP pública configurada en una máquina virtual no estará disponible.

### <a name="availability-zones"></a>Zonas de disponibilidad

#### <a name="zone-isolation-with-zonal-stacks"></a>Aislamiento de zona con pilas de zonas

<p align="center">
  <img src="media/nat-overview/az-directions.svg" alt="Figure depicts three zonal stacks, each of which contains a NAT gateway and a subnet." width="425" title="Virtual Network NAT con aislamiento de zona, creación de varias "zonal stacks"">
</p>

*Ilustración: Virtual Network NAT con aislamiento de zona, creación de "pilas de zonas"*

Incluso sin zonas de disponibilidad, NAT es resistente y puede sobrevivir a los errores de varios componentes de la infraestructura.  Las zonas de disponibilidad se basan en esta resistencia con escenarios de aislamiento de zona para NAT.

Las redes virtuales y sus subredes son construcciones regionales.  Las subredes no están restringidas a una zona.

Existe un compromiso de aislamiento de zona cuando una instancia de máquina virtual que usa un recurso de puerta de enlace NAT está en la misma zona que el recurso de puerta de enlace NAT y sus direcciones IP públicas. El patrón que quiere usar para el aislamiento de zona crea una "pila de zona" por zona de disponibilidad.  Esta "pila de zona" se compone de instancias de máquina virtual, recursos de puerta de enlace NAT, dirección IP pública o recursos de prefijo en una subred que se supone que atiende solo a la misma zona.   Las operaciones del plano de control y el plano de datos están restringidos a la zona especificada. 

Se espera que los errores que se produzcan en una zona que no sea aquella en la que existe el escenario no tengan ningún impacto en NAT. El tráfico de salida de las máquinas virtuales de la misma zona generará un error debido al aislamiento de la zona.  

#### <a name="integrating-inbound-endpoints"></a>Integración de puntos de conexión de entrada

Si su escenario requiere puntos de conexión de entrada, tiene dos opciones:

| Opción | Patrón | Ejemplo | Pros | Contras |
|---|---|---|---|---|
| (1) | **Alinee** los puntos de conexión de entrada con las **pilas de zona** correspondientes que está creando para salida. | Cree de un equilibrador de carga estándar con el front-end de zona. | Mismo modelo de estado y el modo de error para entrada y salida. Más sencillo de operar. | Es posible que sea necesario enmascarar las direcciones IP individuales por zona con un nombre DNS común. |
| (2) | **Superponga** las pilas de zona con un punto de conexión de entrada **entre zonas**. | Cree un equilibrador de carga estándar con redundancia de zona en el front-end. | Dirección IP única para el punto de conexión de entrada. | Modelo de estado variable y modos de error de entrada y salida.  Más complejo de operar. |

>[!NOTE]
> Una puerta de enlace de NAT con una zona aislada requiere que las direcciones IP coincidan con la zona de la puerta de enlace de NAT. No se admiten recursos de la puerta de enlace de NAT con direcciones IP de otra zona o que no tengan ninguna zona.

#### <a name="cross-zone-outbound-scenarios-not-supported"></a>No se admiten escenarios de salida entre zonas

<p align="center">
  <img src="media/nat-overview/az-directions2.svg" alt="Figure depicts three zonal stacks, each of which contains a NAT gateway and a subnet, with the connections between to of the gateways and their subnets broken." width="425" title="Virtual Network NAT no es compatible con la subred que abarca varias zonas">
</p>

*Ilustración: Virtual Network NAT no es compatible con la subred que abarca varias zonas*

No se puede lograr un compromiso de aislamiento de zona con recursos de puerta de enlace NAT cuando las instancias de máquina virtual se implementan en varias zonas dentro de la misma subred.   E incluso si había varias puertas de enlace NAT de zona conectadas a una subred, la instancia de máquina virtual no sabría qué recurso de puerta de enlace NAT seleccionar.

No existe un compromiso de aislamiento de zona cuando a) la zona de una instancia de máquina virtual y las zonas de una puerta de enlace NAT de zona no están alineadas, o b) se usa un recurso de puerta de enlace NAT regional con instancias de máquina virtual de zona.

Aunque parece que el escenario funciona, su modelo de estado y modo de error no están definidos desde el punto de vista de la zona de disponibilidad. Considere la posibilidad de trabajar con pilas de zonas o todas regionales en su lugar.

>[!NOTE]
>La propiedad de zonas de un recurso de puerta de enlace NAT no es mutable.  Vuelva a implementar el recurso de la puerta de enlace de NAT con la preferencia regional o de zona pretendida.

>[!NOTE] 
>Las direcciones IP en sí no tienen redundancia de zona si no se especifica ninguna zona.  El servidor front-end de un [equilibrador de carga estándar tiene redundancia de zona](../load-balancer/load-balancer-standard-availability-zones.md#frontend) si una dirección IP no se crea en una zona concreta.  Esto no se aplica a la NAT.  Solo se admite el aislamiento regional o de zona.

## <a name="performance"></a>Rendimiento

Cada recurso de puerta de enlace NAT puede proporcionar hasta 50 Gbps de rendimiento. Puede dividir las implementaciones en varias subredes y asignar a cada subred o grupos de subredes una puerta de enlace NAT para realizar escalar horizontalmente.

Cada puerta de enlace NAT puede admitir 64 000 conexiones por dirección IP de salida asignada.  Revise la siguiente sección sobre la traducción de direcciones de red de origen (SNAT) para más información, así como el [artículo de solución de problemas ](https://docs.microsoft.com/azure/virtual-network/troubleshoot-nat) para obtener instrucciones específicas sobre la resolución de problemas.

## <a name="source-network-address-translation"></a>Traducción de direcciones de red de origen

La traducción de direcciones de red de origen (SNAT) reescribe el origen de un flujo para que parta de otra dirección IP.  Los recursos de la puerta de enlace de NAT usan una variante de SNAT que habitualmente se denomina traducción de direcciones de puertos (PAT). PAT reescribe la dirección de origen y el puerto de origen. Con SNAT, no hay una relación fija entre el número de direcciones privadas y sus direcciones públicas traducidas.  

### <a name="fundamentals"></a>Aspectos básicos

Examinemos un ejemplo de cuatro flujos que explican el concepto básico.  La puerta de enlace de NAT utiliza el recurso de dirección IP pública 65.52.0.2.

| Flujo | Tupla de origen | Tupla de destino |
|:---:|:---:|:---:|
| 1 | 192.168.0.16:4283 | 65.52.0.1:80 |
| 2 | 192.168.0.16:4284 | 65.52.0.1:80 |
| 3 | 192.168.0.17.5768 | 65.52.0.1:80 |
| 4 | 192.168.0.16:4285 | 65.52.0.2:80 |

Estos flujos pueden tener el siguiente aspecto después de que se haya producido la traducción de puertos de origen:

| Flujo | Tupla de origen | Tupla de origen con SNAT | Tupla de destino | 
|:---:|:---:|:---:|:---:|
| 1 | 192.168.0.16:4283 | 65.52.0.2:234 | 65.52.0.1:80 |
| 2 | 192.168.0.16:4284 | 65.52.0.2:235 | 65.52.0.1:80 |
| 3 | 192.168.0.17.5768 | 65.52.0.2:236 | 65.52.0.1:80 |
| 4 | 192.168.0.16:4285 | 65.52.0.2:237 | 65.52.0.2:80 |

El destino verá el origen del flujo como 65.52.0.2 (tupla de origen con SNAT) con el puerto asignado que se muestra.  PAT como se muestra en la tabla anterior también se denomina SNAT de enmascaramiento de puertos.  Varios orígenes privados se enmascaran detrás de una IP y un puerto.

No asuma una dependencia de la forma concreta en que se asignan los puertos de origen.  Lo anterior es una ilustración solo del concepto fundamental.

El SNAT que proporciona NAT es diferente del [equilibrador de carga](../load-balancer/load-balancer-outbound-connections.md) en varios aspectos.

### <a name="on-demand"></a>A petición

NAT proporciona puertos SNAT a petición para nuevos flujos de tráfico de salida. Todos los puertos SNAT disponibles en el inventario los usa la máquina virtual en las subredes configuradas con NAT. 

<p align="center">
  <img src="media/nat-overview/lb-vnnat-chart.svg" alt="Figure depicts inventory of all available SNAT ports used by any virtual machine on subnets configured with N A T." width="550" title="SNAT de salida a petición de Virtual Network NAT">
</p>

*Ilustración: SNAT de salida a petición de Virtual Network NAT*

Todas las configuraciones de IP de una máquina virtual pueden crear flujos de salida a petición si fuera necesario.  No es necesario realizar un planeamiento de cada instancia previo a la asignación, ni siquiera el sobreaprovisionamiento por instancia para el caso más desfavorable.  

<p align="center">
  <img src="media/nat-overview/exhaustion-threshold.svg" alt="Figure depicts inventory of all available SNAT ports used by any virtual machine on subnets configured with N A T with exhaustion threshold." width="550" title="Diferencias en escenarios de agotamiento">
</p>

*Ilustración: Diferencias en escenarios de agotamiento*

Una vez que se libera un puerto SNAT, este está disponible para que lo use cualquier máquina virtual de las subredes configuradas con NAT.  La asignación a petición permite que las cargas de trabajo dinámicas y divergentes en subredes usen los puertos SNAT a medida que se necesiten.  Mientras haya un inventario de puertos SNAT disponible, los flujos de SNAT funcionarán correctamente. Las zonas activas de los puertos SNAT se benefician de que el inventario sea mayor. Los puertos SNAT se quedan sin usar en las máquinas virtuales que no los necesiten de forma activa.

### <a name="scaling"></a>Ampliación

El escalado de NAT es primordialmente una función de la administración del inventario de puertos SNAT compartidos disponibles. NAT necesita inventario de puertos SNAT suficiente para afrontar el flujo máximo de salida previsto en todas las subredes conectadas a un recurso de puerta de enlace de NAT.  Puede usar recursos de direcciones IP públicas, recursos de prefijos IP públicos, o ambos para crear un inventario de puertos SNAT.  

>[!NOTE]
>Si va a asignar un recurso de prefijo de dirección IP pública, se usará todo el prefijo de dirección IP pública.  No se puede asignar un recurso de prefijo de dirección IP pública y, después, descomponer las direcciones IP individuales para asignarlas a otros recursos.  Si desea asignar direcciones IP individuales de un prefijo de dirección IP pública a varios recursos, debe crear direcciones IP públicas individuales desde el recurso de prefijo de dirección IP pública y asignarlas según sea necesario en lugar del propio recurso de prefijo de dirección IP pública.

SNAT asigna direcciones privadas a una o varias direcciones IP públicas y reescribe la dirección de origen y el puerto de origen en los procesos. Un recurso de puerta de enlace de NAT usará 64 000 puertos (puertos SNAT) por dirección IP pública para esta traducción. Los recursos de puerta de enlace de NAT se pueden escalar verticalmente hasta un máximo de 16 direcciones IP y un millón de puertos SNAT. Si se proporciona un recurso de prefijo de dirección IP pública, cada dirección IP dentro del prefijo proporciona un inventario de puertos SNAT. Y agregar más direcciones IP públicas aumenta los puertos SNAT del inventario disponibles. TCP y UDP son inventarios de puertos SNAT independientes y no están relacionados.

Los recursos de puerta de enlace de NAT reutilizan los puertos de origen. Antes de realizar el escalado debe asumir que cada flujo requiere un nuevo puerto SNAT y escalar el número total de direcciones IP disponibles para el tráfico de salida.

### <a name="protocols"></a>Protocolos

Los recursos de puerta de enlace de NAT interactúan con la IP y los encabezados de transporte de IP de los flujos UDP y TCP, y son independientes de las cargas de la capa de aplicaciones.  No se admiten otros protocolos de IP.

### <a name="timers"></a>Temporizadores

>[!IMPORTANT]
>Un temporizador de inactividad prolongado puede aumentar innecesariamente la probabilidad de agotamiento de SNAT. Cuanto mayor sea el temporizador que especifique, más tiempo retendrá la NAT los puertos SNAT hasta que finalmente se agote el tiempo de espera. Si los flujos están inactivos, producirán un error en este momento y consumirán innecesariamente el inventario de puertos SNAT.  Los flujos que producen un error a las dos horas también lo habrían producido al valor predeterminado de cuatro minutos. Aumentar el tiempo de espera de inactividad es una opción de último recurso que debe usarse con moderación. Si un flujo nunca está inactivo, no se verá afectado por el temporizador de inactividad.

El tiempo de espera de inactividad de TCP se puede ajustar de cuatro minutos (valor predeterminado) a 120 minutos (dos horas) para todos los flujos.  Además, puede restablecer el temporizador inactivo con tráfico del flujo.  Un patrón recomendado para actualizar las largas conexiones inactivas y para detectar la ejecución de puntos de conexión es usar mensajes TCP Keepalive.  Estos mensajes aparecen como ACK duplicados en los puntos de conexión, tienen una sobrecarga baja y son invisibles para la capa de aplicaciones.

Para la liberación de los puertos SNAT se usan los siguientes temporizadores:

| Timer | Value |
|---|---|
| TCP FIN | 60 segundos |
| TCP RST | 10 segundos |
| TCP semiabierto | 30 segundos |

Los puertos SNAT están disponibles para volver a usarlos con la misma dirección IP y puerto de destino a los 5 segundos.

>[!NOTE] 
>Esta configuración del temporizador está sujeta a cambios. Los valores se proporcionan como ayuda para la solución de problemas y no se debe asumir una dependencia de temporizadores concretos en este momento.

## <a name="limitations"></a>Limitaciones

- NAT es compatible con la dirección IP pública de la SKU estándar, el prefijo de IP pública y los recursos del equilibrador de carga.   Ni los recursos básicos (por ejemplo, el equilibrador de carga básico) ni los productos derivados de ellos son compatibles con NAT.  Los recursos básicos se deben colocar en una subred que no esté configurada con NAT.
- Se admite la familia de direcciones IPv4.  NAT no interactúa con la familia de direcciones IPv6.  NAT no se puede implementar en una subred con un prefijo IPv6.
- NAT no puede abarcar varias redes virtuales.

## <a name="suggestions"></a>Sugerencias

Queremos saber cómo podemos mejorar el servicio. ¿Falta una funcionalidad? Proponga lo que debemos crear después en [UserVoice para NAT](https://aka.ms/natuservoice).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre [Virtual Network NAT](nat-overview.md).
* Obtenga información acerca de las [métricas y alertas de los recursos de puerta de enlace NAT](nat-metrics.md).
* Obtenga información sobre la [solución de los problemas de los recursos de la puerta de enlace de NAT](troubleshoot-nat.md).
* Tutorial para validar una puerta de enlace de NAT
  - [CLI de Azure](tutorial-create-validate-nat-gateway-cli.md)
  - [PowerShell](tutorial-create-validate-nat-gateway-powershell.md)
  - [Portal](tutorial-create-validate-nat-gateway-portal.md)
* Inicio rápido para implementar recursos de puerta de enlace de NAT
  - [CLI de Azure](./quickstart-create-nat-gateway-cli.md)
  - [PowerShell](./quickstart-create-nat-gateway-powershell.md)
  - [Portal](./quickstart-create-nat-gateway-portal.md)
  - [Plantilla](./quickstart-create-nat-gateway-template.md)
* Información acerca de la API de recursos de la puerta de enlace de NAT
  - [REST API](https://docs.microsoft.com/rest/api/virtualnetwork/natgateways)
  - [CLI de Azure](https://docs.microsoft.com/cli/azure/network/nat/gateway?view=azure-cli-latest)
  - [PowerShell](https://docs.microsoft.com/powershell/module/az.network/new-aznatgateway)
* Información acerca de las [zonas de disponibilidad](../availability-zones/az-overview.md).
* Información acerca del [equilibrador de carga estándar](../load-balancer/load-balancer-standard-overview.md).
* Información sobre las [zonas de disponibilidad y el equilibrador de carga estándar](../load-balancer/load-balancer-standard-availability-zones.md).
* [Indíquenos qué crear a continuación para Virtual Network NAT en UserVoice](https://aka.ms/natuservoice).


