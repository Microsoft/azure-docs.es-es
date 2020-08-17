---
title: Introducción a los grupos de seguridad de red de Azure
titlesuffix: Azure Virtual Network
description: Información acerca de los grupos de seguridad de red. Los grupos de seguridad de red le ayudan a filtrar el tráfico de red entre los recursos de Azure.
services: virtual-network
documentationcenter: na
author: KumudD
ms.service: virtual-network
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/27/2020
ms.author: kumud
ms.reviewer: kumud
ms.openlocfilehash: 60c350b10fb3db82af47551591d95e87cacd63a4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87065020"
---
# <a name="network-security-groups"></a>Grupos de seguridad de red
<a name="network-security-groups"></a>

Puede usar el grupo de seguridad de red de Azure para filtrar el tráfico de red hacia y desde los recursos de Azure de una red virtual de Azure. Un grupo de seguridad de red contiene [reglas de seguridad](#security-rules) que permiten o deniegan el tráfico de red entrante o el tráfico de red saliente de varios tipos de recursos de Azure. Para cada regla, puede especificar un origen y destino, un puerto y un protocolo.

En este artículo se describen las propiedades de una regla de grupo de seguridad de red, las [reglas de seguridad predeterminadas](#default-security-rules) que se aplican y las propiedades de regla que se pueden modificar para crear una [regla de seguridad aumentada](#augmented-security-rules).

## <a name="security-rules"></a><a name="security-rules"></a> Reglas de seguridad

Un grupo de seguridad de red puede contener cero reglas, o tantas reglas como desee, siempre que esté dentro de los [límites](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits) de la suscripción de Azure. Cada regla especifica las siguientes propiedades:

|Propiedad  |Explicación  |
|---------|---------|
|Nombre|Un nombre único dentro del grupo de seguridad de red.|
|Prioridad | Un número entre 100 y 4096. Las reglas se procesan en orden de prioridad. Se procesan primero las reglas con los números más bajos ya que estos tienen más prioridad. Si el tráfico coincide con una regla, se detiene el procesamiento. Como resultado, las reglas con menor prioridad (números más altos) que tengan los mismos atributos que las reglas con una prioridad mayor no se procesarán.|
|Origen o destino| Cualquiera, una dirección IP individual, un bloque CIDR de enrutamiento entre dominios sin clases (10.0.0.0/24, por ejemplo), una etiqueta de servicio o un grupo de seguridad de aplicaciones. Si especifica una dirección para un recurso de Azure, especifique la dirección IP privada asignada al recurso. Las grupos de seguridad de red se procesan después de que Azure traduzca una dirección IP pública a una dirección IP privada para el tráfico de entrada y antes de que Azure traduzca una dirección IP privada a una dirección IP pública para el tráfico de salida. . La especificación de un intervalo, una etiqueta de servicio o grupo de seguridad de aplicaciones le permite crear menos reglas de seguridad. La posibilidad de especificar varias direcciones IP individuales e intervalos (no puede especificar varias etiquetas de servicio ni grupos de aplicaciones) en una regla se conoce como [reglas de seguridad aumentada](#augmented-security-rules). Las reglas de seguridad aumentada solo se pueden generar en los grupos de seguridad de red creados mediante el modelo de implementación de Resource Manager. No puede especificar varias direcciones IP ni intervalos de ellas en grupos de seguridad de red creados mediante el modelo de implementación clásica.|
|Protocolo     | TCP, UDP, ICMP o Cualquiera.|
|Dirección| Si la regla se aplica al tráfico entrante o al saliente.|
|Intervalo de puertos     |Puede especificar un puerto individual o un intervalo de puertos. Por ejemplo, puede especificar 80 o 10000-10005. La especificación de intervalos le permite crear menos reglas de seguridad. Las reglas de seguridad aumentada solo se pueden generar en los grupos de seguridad de red creados mediante el modelo de implementación de Resource Manager. No puede especificar varios puertos ni intervalos de ellos en la misma regla de seguridad de los grupos de seguridad de red creados mediante el modelo de implementación clásica.   |
|Acción     | Permitir o denegar        |

Las reglas de seguridad de los grupos de seguridad de red se evalúan por prioridad mediante información en tuplas de 5 elementos (origen, puerto de origen, destino, puerto de destino y protocolo) para permitir o denegar el tráfico. Se crea un registro de flujo para las conexiones existentes. Se permite o deniega la comunicación en función del estado de conexión del registro de flujo. El registro de flujo permite que un grupo de seguridad de red sea con estado. Por ejemplo, si especifica una regla de seguridad de salida para cualquier dirección a través del puerto 80, no será necesario especificar una regla de seguridad de entrada para la respuesta al tráfico saliente. Solo debe especificar una regla de seguridad de entrada si la comunicación se inicia de forma externa. Lo contrario también es cierto. Si se permite el tráfico entrante a través de un puerto, no es necesario especificar una regla de seguridad de salida para responder al tráfico a través del puerto.

No es posible interrumpir las conexiones existentes cuando se elimina una regla de seguridad que habilitó el flujo. Los flujos de tráfico se interrumpen cuando se detienen las conexiones y no fluye ningún tráfico en ambas direcciones durante al menos unos minutos.

Hay límites en el número de reglas de seguridad que puede crear en un grupo de seguridad de red. Para más información, consulte el artículo acerca de los [límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

### <a name="default-security-rules"></a><a name="default-security-rules"></a>Reglas de seguridad predeterminadas

Azure crea las siguientes reglas predeterminadas en cada grupo de seguridad de red que cree:

#### <a name="inbound"></a>Entrada

##### <a name="allowvnetinbound"></a>AllowVNetInBound

|Priority|Source|Puertos de origen|Destination|Puertos de destino|Protocolo|Acceso|
|---|---|---|---|---|---|---|
|65000|VirtualNetwork|0-65535|VirtualNetwork|0-65535|Any|Allow|

##### <a name="allowazureloadbalancerinbound"></a>AllowAzureLoadBalancerInBound

|Priority|Source|Puertos de origen|Destination|Puertos de destino|Protocolo|Acceso|
|---|---|---|---|---|---|---|
|65001|AzureLoadBalancer|0-65535|0.0.0.0/0|0-65535|Any|Allow|

##### <a name="denyallinbound"></a>DenyAllInbound

|Priority|Source|Puertos de origen|Destination|Puertos de destino|Protocolo|Acceso|
|---|---|---|---|---|---|---|
|65500|0.0.0.0/0|0-65535|0.0.0.0/0|0-65535|Any|Denegar|

#### <a name="outbound"></a>Salida

##### <a name="allowvnetoutbound"></a>AllowVnetOutBound

|Priority|Source|Puertos de origen| Destination | Puertos de destino | Protocolo | Acceso |
|---|---|---|---|---|---|---|
| 65000 | VirtualNetwork | 0-65535 | VirtualNetwork | 0-65535 | Any | Allow |

##### <a name="allowinternetoutbound"></a>AllowInternetOutBound

|Priority|Source|Puertos de origen| Destination | Puertos de destino | Protocolo | Acceso |
|---|---|---|---|---|---|---|
| 65001 | 0.0.0.0/0 | 0-65535 | Internet | 0-65535 | Any | Allow |

##### <a name="denyalloutbound"></a>DenyAllOutBound

|Priority|Source|Puertos de origen| Destination | Puertos de destino | Protocolo | Acceso |
|---|---|---|---|---|---|---|
| 65500 | 0.0.0.0/0 | 0-65535 | 0.0.0.0/0 | 0-65535 | Any | Denegar |

En las columnas **Origen** y **Destino**, *VirtualNetwork*, *AzureLoadBalancer* e *Internet* son [etiquetas de servicios](service-tags-overview.md), en lugar de direcciones IP. En la columna de protocolos, **Cualquiera** abarca TCP, UDP e ICMP. Al crear una regla, puede especificar TCP, UDP, ICMP o Cualquiera. *0.0.0.0/0* en las columnas **Origen** y **Destino** representa todas las direcciones. Los clientes, como Azure Portal, la CLI de Azure o PowerShell, pueden usar * o any en esta expresión.
 
Las reglas predeterminadas no se pueden quitar, pero puede reemplazarlas con reglas de prioridad más alta.

### <a name="augmented-security-rules"></a><a name="augmented-security-rules"></a> Reglas de seguridad aumentada

Las reglas de seguridad aumentada permiten simplificar la definición de seguridad para las redes virtuales, lo que le permitirá definir directivas de seguridad de red más grandes y complejas, con menos reglas. Puede combinar varios puertos y varias direcciones IP explícitas e intervalos en una única regla de seguridad de fácil comprensión. Use reglas aumentadas en los campos de origen, destino y puerto de una regla. Para simplificar el mantenimiento de la definición de la regla de seguridad, combine las reglas de seguridad aumentada con [etiquetas de servicio](service-tags-overview.md) o [grupos de seguridad de aplicaciones](#application-security-groups). Hay límites en el número de direcciones, intervalos y puertos que se pueden especificar en una regla. Para más información, consulte el artículo acerca de los [límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

#### <a name="service-tags"></a>Etiquetas de servicio

Una etiqueta de servicio representa un grupo de prefijos de direcciones IP de un servicio de Azure determinado. Ayuda a minimizar la complejidad de las actualizaciones frecuentes de las reglas de seguridad de red.

Para más información, consulte [Etiquetas de servicio de Azure](service-tags-overview.md). Para ver un ejemplo de cómo usar la etiqueta del servicio Storage para restringir el acceso a la red, consulte [Restricción del acceso de red a los recursos de PaaS](tutorial-restrict-network-access-to-resources.md).

#### <a name="application-security-groups"></a>Grupos de seguridad de aplicaciones

Los grupos de seguridad de aplicaciones le permiten configurar la seguridad de red como una extensión natural de la estructura de una aplicación, lo que le permite agrupar máquinas virtuales y directivas de seguridad de red basadas en esos grupos. Puede reutilizar la directiva de seguridad a escala sin mantenimiento manual de direcciones IP explícitas. Para más información, consulte [Grupos de seguridad de aplicaciones](application-security-groups.md).

## <a name="how-traffic-is-evaluated"></a>Cómo se evalúa el tráfico

Puede implementar recursos de varios servicios de Azure en una red virtual de Azure. Para obtener una lista completa, consulte [Servicios que se pueden implementar en una red virtual](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network). Puede asociar cero o un grupo de seguridad de red a cada [subred](virtual-network-manage-subnet.md#change-subnet-settings) e [interfaz de red](virtual-network-network-interface.md#associate-or-dissociate-a-network-security-group) de la red virtual en una máquina virtual. El mismo grupo de seguridad de red se puede asociar a tantas interfaces de red y subredes como se desee.

La siguiente imagen ilustra los diferentes escenarios de cómo se podrían implementar grupos de seguridad de red para permitir el tráfico de red hacia y desde Internet a través del puerto TCP 80:

![Procesamiento del grupo de seguridad de red](./media/security-groups/nsg-interaction.png)

Observe la imagen anterior, junto con el siguiente texto, para entender cómo procesa Azure las reglas entrantes y salientes para los grupos de seguridad de red:

### <a name="inbound-traffic"></a>Tráfico entrante

Para el tráfico entrante, Azure procesa las reglas de un grupo de seguridad de red asociadas a una subred en primer lugar, si hay alguna y, a continuación, las reglas de un grupo de seguridad de red asociadas a la interfaz de red, si hay alguna.

- **VM1**: las reglas de seguridad de *NSG1* se procesan, ya que está asociado a *Subnet1* y *VM1* está en *Subnet1*. A menos que haya creado una regla que permita el puerto 80 de entrada, la regla de seguridad predeterminada [DenyAllInbound](#denyallinbound) deniega el tráfico y *NSG2* nunca lo evalúa, ya que *NSG2* está asociado a la interfaz de red. Si *NSG1* tiene una regla de seguridad que permite el puerto 80, *NSG2* procesa el tráfico. Para permitir el puerto 80 para la máquina virtual, tanto *NSG1* como *NSG2* deben tener una regla que permita el puerto 80 desde Internet.
- **VM2**: las reglas de *NSG1* se procesan porque *VM2* también está en *Subnet1*. Puesto que *VM2* no tiene un grupo de seguridad de red asociado a su interfaz de red, recibe todo el tráfico permitido por *NSG1* o se deniega todo el tráfico denegado por *NSG1*. El tráfico se permite o deniega a todos los recursos de la misma subred cuando un grupo de seguridad de red está asociado a una subred.
- **VM3**: dado que no hay ningún grupo de seguridad de red asociado a *Subnet2*, se permite el tráfico en la subred y *NSG2* lo procesa, porque *NSG2* está asociado a la interfaz de red conectada a *VM3*.
- **VM4**: se permite el tráfico a *VM4*, porque no hay un grupo de seguridad de red asociado a *Subnet3* ni a la interfaz de red de la máquina virtual. Si no tienen un grupo de seguridad de red asociado, se permite todo el tráfico de red a través de una subred y una interfaz de red.

### <a name="outbound-traffic"></a>Tráfico saliente

Para el tráfico saliente, Azure procesa las reglas de un grupo de seguridad de red asociadas a una interfaz de red en primer lugar, si hay alguna y, a continuación, las reglas de un grupo de seguridad de red asociadas a la subred, si hay alguna.

- **VM1**: se procesan las reglas de seguridad de *NSG2*. A menos que cree una regla de seguridad que deniegue el puerto 80 de salida a Internet, la regla de seguridad predeterminada [AllowInternetOutbound](#allowinternetoutbound) permite el tráfico de *NSG1* y *NSG2*. Si *NSG2* tiene una regla de seguridad que deniega el puerto 80, el tráfico se deniega y *NSG1* nunca lo evalúa. Para denegar el puerto 80 desde la máquina virtual, uno o ambos de los grupos de seguridad de red deben tener una regla que deniegue el puerto 80 a Internet.
- **VM2**: se envía todo el tráfico a través de la interfaz de red a la subred, ya que la interfaz de red conectada a *VM2* no tiene un grupo de seguridad de red asociado. Se procesan las reglas de *NSG1*.
- **VM3**: si *NSG2* tiene una regla de seguridad que deniega el puerto 80, también se deniega el tráfico. Si *NSG2* tiene una regla de seguridad que permite el puerto 80, dicho puerto tiene permitida la salida a Internet, ya que no hay un grupo de seguridad de red asociado a *Subnet2*.
- **VM4**: se permite todo el tráfico de red desde *VM4*, porque no hay un grupo de seguridad de red asociado a la interfaz de red conectada a la máquina virtual ni a *Subnet3*.


### <a name="intra-subnet-traffic"></a>Tráfico dentro de la subred

Es importante tener en cuenta que las reglas de seguridad de un NSG asociado a una subred pueden afectar la conectividad entre las máquinas virtuales dentro de ella. Por ejemplo, si se agrega una regla a *NSG1* que deniega todo el tráfico entrante y saliente, *VM1* y *VM2* ya no podrán comunicarse entre sí. Otra regla tendría que agregarse específicamente para permitirlo. 

Puede ver fácilmente las reglas agregadas que se aplican a una interfaz de red mediante la visualización de las [reglas de seguridad vigentes](virtual-network-network-interface.md#view-effective-security-rules) de una interfaz de red. También puede usar la funcionalidad [Comprobación del flujo de IP](../network-watcher/diagnose-vm-network-traffic-filtering-problem.md?toc=%2fazure%2fvirtual-network%2ftoc.json) de Azure Network Watcher para determinar si se permite la comunicación hacia una interfaz de red o desde esta. El flujo IP le indica si se permite o deniega una comunicación y qué regla de seguridad de red permite o deniega el tráfico.

> [!NOTE]
> Los grupos de seguridad de red se asocian a las subredes o a las máquinas virtuales y a los servicios en la nube que se implementan en el modelo de implementación clásica, y en las subredes o interfaces de red del modelo de implementación de Resource Manager. Para más información sobre los modelos de implementación de Azure, consulte [Descripción de los modelos de implementación de Azure](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

> [!TIP]
> A menos que tenga un motivo concreto, se recomienda que asocie un grupo de seguridad de red a una subred o a una interfaz de red, pero no a ambas. Puesto que las reglas de un grupo de seguridad de red asociado a una subred pueden entrar en conflicto con las reglas de un grupo de seguridad de red asociado a una interfaz de red, puede tener problemas de comunicación inesperados que necesiten solución.

## <a name="azure-platform-considerations"></a>Consideraciones de la plataforma Azure

- **Dirección IP virtual del nodo de host**: los servicios de infraestructura básica, como DHCP, DNS, IMDS y seguimiento de estado se proporcionan mediante las direcciones IP de host virtualizadas 168.63.129.16 y 169.254.169.254. Estas direcciones IP pertenecen a Microsoft y son la únicas direcciones IP virtualizadas que se usarán en todas las regiones con este fin. Las reglas de seguridad y las rutas vigentes no incluirán estas reglas de plataforma. Para invalidar esta comunicación de la infraestructura básica, puede crear una regla de seguridad para denegar el tráfico usando las siguientes [etiquetas de servicio](service-tags-overview.md) en las reglas del grupo de seguridad de red: AzurePlatformDNS, AzurePlatformIMDS, AzurePlatformLKM. Aprenda sobre el [diagnóstico del filtrado de tráfico de red](diagnose-network-traffic-filter-problem.md) y el [diagnóstico del enrutamiento de red](diagnose-network-routing-problem.md).
- **Licencias (Servicio de administración de claves)** : las imágenes de Windows que se ejecutan en máquinas virtuales deben tener licencia. Para garantizar que se usen licencias, se envía una solicitud a los servidores host del Servicio de administración de claves que administran dichas consultas. La solicitud de salida se realiza a través del puerto 1688. Para implementaciones que usan la configuración de la [ruta predeterminada 0.0.0.0/0](virtual-networks-udr-overview.md#default-route), se deshabilitará esta regla de plataforma.
- **Máquinas virtuales en grupos de carga equilibrada**: el puerto y el intervalo de direcciones de origen aplicados proceden del equipo de origen, no del equilibrador de carga. El puerto y el intervalo de direcciones de destino son los del equipo de destino, no los del equilibrador de carga.
- **Instancias de servicio de Azure**: instancias de varios servicios de Azure, como HDInsight, App Service Environment y Virtual Machine Scale Sets, implementados en subredes de la red virtual. Para ver una lista completa de los servicios que puede implementar en redes virtuales, consulte el artículo sobre la [Red virtual para los servicios de Azure](virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network). Asegúrese de que conoce los requisitos de puertos de cada servicio antes de aplicar un grupo de seguridad de red a la subred en la que se implementa el recurso. Si deniega los puertos que el servicio requiere, este no funcionará correctamente.
- **Envío de correo electrónico saliente**: Microsoft recomienda usar servicios de retransmisión SMTP autenticados (que normalmente se conectan a través del puerto TCP 587, pero a menudo también de otros) para enviar correo electrónico desde Azure Virtual Machines. Los servicios de retransmisión SMTP se especializan en la reputación del remitente, con el fin de minimizar la posibilidad de que proveedores de correo electrónico de terceros rechacen los mensajes. Estos servicios de retransmisión de SMTP incluyen Exchange Online Protection y SendGrid, pero no se limitan a ellos. El uso de servicios de retransmisión de SMTP no tiene ninguna restricción en Azure, independientemente del tipo de suscripción. 

  Si ha creado la suscripción a Azure antes del 15 de noviembre de 2017, además de poder usar los servicios de retransmisión de SMTP, puede enviar el correo electrónico directamente a través del puerto TCP 25. Si ha creado la suscripción después del 15 de noviembre de 2017, es posible que no pueda enviar correo electrónico directamente a través del puerto 25. El comportamiento de la comunicación saliente a través del puerto 25 depende del tipo de suscripción que tenga, como se indica a continuación:

     - **Contrato Enterprise**: se permite la comunicación saliente a través del puerto 25. Puede enviar un correo electrónico saliente directamente desde las máquinas virtuales a los proveedores de correo electrónico externos, sin las restricciones de la plataforma de Azure. 
     - **Pago por uso**: la comunicación saliente a través del puerto 25 está bloqueada en todos los recursos. Si necesita enviar correo electrónico desde una máquina virtual directamente a proveedores de correo electrónico externos (que no usan retransmisión SMTP autenticada), puede realizar una solicitud para quitar la restricción. Las solicitudes se revisan y aprueban a discreción de Microsoft y solo se conceden una vez que se han realizado las comprobaciones contra fraudes. Para realizar una solicitud, abra un caso de soporte técnico con el tipo de problema *Técnico*, *Conectividad de red virtual*, *No se puede enviar correo electrónico (SMTP/puerto 25)* . En su caso de soporte técnico, indique los motivos por los que su suscripción tiene que enviar correo electrónico directamente a los proveedores de correo electrónico, en lugar de pasar por una retransmisión SMTP autenticada. Si la suscripción está exenta, las únicas máquinas virtuales que pueden establecer comunicación saliente a través del puerto 25 son las creadas después de la fecha de exención.
     - **MSDN, Pase para Azure, Azure bajo licencia Open, Education, BizSpark y evaluación gratuita**: la comunicación saliente a través del puerto 25 está bloqueada en todos los recursos. No se pueden realizar solicitudes para quitar la restricción, ya que no se conceden solicitudes. Si necesita enviar correo electrónico desde la máquina virtual, debe usar un servicio de retransmisión SMTP.
     - **Proveedor de servicios en la nube**: los clientes que consumen recursos de Azure a través de un proveedor de servicios en la nube pueden crear una incidencia de soporte técnico con su proveedor de servicios en la nube y solicitar que el proveedor cree un caso de desbloqueo en su nombre si no se puede usar una retransmisión SMTP segura.

  Si Azure le permite enviar correo electrónico a través del puerto 25, Microsoft no garantiza de proveedores de correo electrónico vayan a aceptar correo electrónico entrante procedente de la máquina virtual. Si un proveedor específico rechaza el correo electrónico de la máquina virtual, tendrá que trabajar directamente con él para resolver los problemas de entrega de mensajes o de filtrado de correo no deseado o bien utilizar un servicio de retransmisión SMTP autenticado.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información acerca de qué recursos de Azure se pueden implementar en una red virtual y pueden tener grupos de seguridad de red asociados, consulte [Integración de redes virtuales para los servicios de Azure](virtual-network-for-azure-services.md).
* Si nunca ha creado un grupo de seguridad de red, puede seguir un [tutorial](tutorial-filter-network-traffic.md) rápido para obtener alguna experiencia en la creación de uno. 
* Si está familiarizado con los grupos de seguridad de red y necesita administrarlos, consulte [Administración de un grupo de seguridad de red](manage-network-security-group.md). 
* Si tiene problemas con las comunicaciones y necesita solucionar problemas de los grupos de seguridad de red, consulte [Diagnóstico de un problema de filtro de tráfico de red de una máquina virtual](diagnose-network-traffic-filter-problem.md). 
* Aprenda a habilitar los [registros de flujo de los grupos de seguridad de red](../network-watcher/network-watcher-nsg-flow-logging-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para analizar el tráfico de red hacia y desde los recursos que tienen un grupo de seguridad de red asociado.
