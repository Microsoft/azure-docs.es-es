---
title: Preguntas frecuentes sobre Azure Firewall
description: Preguntas frecuentes sobre Azure Firewall. Se trata de un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: conceptual
ms.date: 08/13/2020
ms.author: victorh
ms.openlocfilehash: 75435155ba1dad798d301006a30a5d5b6e96226a
ms.sourcegitcommit: cd0a1ae644b95dbd3aac4be295eb4ef811be9aaa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88611184"
---
# <a name="azure-firewall-faq"></a>Preguntas frecuentes sobre Azure Firewall

## <a name="what-is-azure-firewall"></a>¿Qué es Azure Firewall?

Azure Firewall es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Se trata de un firewall como servicio con estado completo que incorpora alta disponibilidad y escalabilidad a la nube sin restricciones. Puede crear, aplicar y registrar directivas de aplicaciones y de conectividad de red a nivel central en suscripciones y redes virtuales.

## <a name="what-capabilities-are-supported-in-azure-firewall"></a>¿Qué funcionalidades son compatibles con Azure Firewall?

Para obtener información sobre las características de Azure Firewall, consulte [Características de Azure Firewall](features.md).

## <a name="what-is-the-typical-deployment-model-for-azure-firewall"></a>¿Cuál es el modelo de implementación típico para Azure Firewall?

Puede implementar Azure Firewall en cualquier red virtual, pero los clientes habitualmente lo implementan en una red virtual central y lo emparejan con otras redes virtuales en un modelo de concentrador y radio. A continuación, puede establecer la ruta predeterminada desde las redes virtuales emparejadas para que apunte a esta red virtual de firewall central. Se admite el emparejamiento de VNet global, pero no se recomienda debido a posibles problemas de rendimiento y latencia entre regiones. Para obtener el mejor rendimiento, implemente un firewall por región.

La ventaja de este modelo es la posibilidad de ejercer control centralizado sobre varias redes virtuales de radio en distintas suscripciones. También hay un ahorro en los costos, ya que no es necesario implementar un firewall en cada red virtual por separado. Este ahorro se debe medir frente al costo de emparejamiento asociado basado en los patrones de tráfico del cliente.

## <a name="how-can-i-install-the-azure-firewall"></a>¿Cómo puedo instalar Azure Firewall?

Puede configurar Azure Firewall mediante Azure Portal, PowerShell, API REST o a través del uso de plantillas. Consulte [Tutorial: Implementación y configuración de Azure Firewall mediante Azure Portal](tutorial-firewall-deploy-portal.md) para obtener instrucciones detalladas.

## <a name="what-are-some-azure-firewall-concepts"></a>¿Cuáles son algunos de los conceptos de Azure Firewall?

Azure Firewall es compatible con las reglas y las colecciones de reglas. Una colección de reglas es una lista de reglas que comparten el mismo orden y prioridad. Las colecciones de reglas se ejecutan en su orden de prioridad. Las colecciones de reglas de red tienen mayor prioridad que las colecciones de reglas de aplicación y se finalizan todas las reglas.

Hay tres tipos de colecciones de reglas:

* *Reglas de aplicación*: configurar los nombres de dominio completo (FQDN) a los que se puede acceder desde una subred.
* *Reglas de red*: configurar reglas que contienen direcciones de origen, protocolos, puertos de destino y direcciones de destino.
* *Reglas NAT*: Configurar reglas DNAT para permitir conexiones de Internet entrantes.

## <a name="does-azure-firewall-support-inbound-traffic-filtering"></a>¿Admite Azure Firewall el filtrado del tráfico de entrada?

Azure Firewall es compatible con el filtrado entrante y saliente. La protección entrante suele usarse para los protocolos que no son HTTP/S. Por ejemplo, los protocolos RDP, SSH y FTP. Para obtener la mejor protección HTTP/S de entrada, use un firewall de aplicaciones web, como [Firewall de aplicaciones web (WAF) de Azure](../web-application-firewall/overview.md).

## <a name="which-logging-and-analytics-services-are-supported-by-the-azure-firewall"></a>¿Qué servicios de registro y análisis son compatibles con Azure Firewall?

Azure Firewall se integra con Azure Monitor para ver y analizar los registros de firewall. Los registros se pueden enviar a Log Analytics, Azure Storage o Event Hubs. Puede analizarse en Log Analytics o usando distintas herramientas, como Excel y Power BI. Para más información, consulte el [Tutorial: Supervisión de los registros de Azure Firewall](tutorial-diagnostics.md).

## <a name="how-does-azure-firewall-work-differently-from-existing-services-such-as-nvas-in-the-marketplace"></a>¿Cómo Azure Firewall funciona distinto de servicios existentes como NVA que ya están en el mercado?

Azure Firewall es un servicio de firewall básica que puede abordar determinados escenarios de cliente. Se espera que tenga una combinación de NVA de terceros y Azure Firewall. Trabajar mejor juntos es una prioridad fundamental.

## <a name="what-is-the-difference-between-application-gateway-waf-and-azure-firewall"></a>¿Cuál es la diferencia entre WAF de Application Gateway y Azure Firewall?

Firewall de aplicaciones web (WAF) es una característica de Application Gateway que proporciona a las aplicaciones una protección de entrada centralizada contra ataques y puntos vulnerables comunes. Azure Firewall proporciona protección de entrada para protocolos no HTTP/S (como RDP, SSH y FTP), protección de salida a nivel de red para todos los puertos y protocolos, y protección a nivel de aplicación para las salidas HTTP/S.

## <a name="what-is-the-difference-between-network-security-groups-nsgs-and-azure-firewall"></a>¿Cuál es la diferencia entre los grupos de seguridad de red (NSG) y Azure Firewall?

El servicio Azure Firewall complementa la funcionalidad de grupo de seguridad de red. Juntos proporcionan una mejor seguridad de red de "defensa en profundidad". Los grupos de seguridad de red proporcionan filtrado del trafico de capas de red distribuida para limitar el tráfico a los recursos dentro de las redes virtuales en cada suscripción. Azure Firewall es un firewall de red centralizada como servicio con estado completo que proporciona protección a nivel de red y de aplicación en todas las distintas suscripciones y redes virtuales.

## <a name="are-network-security-groups-nsgs-supported-on-the-azurefirewallsubnet"></a>¿Se admiten grupos de seguridad de red (NSG) en AzureFirewallSubnet?

Azure Firewall es un servicio administrado con varias capas de protección, como la protección de la plataforma con NSG de nivel de NIC (no visible).  Los NSG no son obligatorios en AzureFirewallSubnet y están deshabilitados para garantizar que no se produzcan interrupciones en el servicio.

## <a name="how-do-i-set-up-azure-firewall-with-my-service-endpoints"></a>¿Cómo puedo configurar Azure Firewall con mis puntos de conexión de servicio?

Para un acceso seguro a los servicios de PaaS, se recomiendan los puntos de conexión de servicio. Puede elegir habilitar puntos de conexión de servicio en la subred de Azure Firewall y deshabilitarlos en las redes virtuales de radios conectadas. De este modo se beneficia de ambas características: la seguridad de los puntos de conexión y el registro central de todo el tráfico.

## <a name="what-is-the-pricing-for-azure-firewall"></a>¿Cuál es el precio de Azure Firewall?

Consulte [Precios de Azure Firewall](https://azure.microsoft.com/pricing/details/azure-firewall/).

## <a name="how-can-i-stop-and-start-azure-firewall"></a>¿Cómo puedo detener e iniciar Azure Firewall?

Puede usar los métodos *deallocate* y *allocate* de Azure PowerShell.

Por ejemplo:

```azurepowershell
# Stop an existing firewall

$azfw = Get-AzFirewall -Name "FW Name" -ResourceGroupName "RG Name"
$azfw.Deallocate()
Set-AzFirewall -AzureFirewall $azfw
```

```azurepowershell
# Start a firewall

$azfw = Get-AzFirewall -Name "FW Name" -ResourceGroupName "RG Name"
$vnet = Get-AzVirtualNetwork -ResourceGroupName "RG Name" -Name "VNet Name"
$publicip1 = Get-AzPublicIpAddress -Name "Public IP1 Name" -ResourceGroupName "RG Name"
$publicip2 = Get-AzPublicIpAddress -Name "Public IP2 Name" -ResourceGroupName "RG Name"
$azfw.Allocate($vnet,@($publicip1,$publicip2))

Set-AzFirewall -AzureFirewall $azfw
```

> [!NOTE]
> Debe reasignar un firewall y una dirección IP pública al grupo de recursos y la suscripción originales.

## <a name="what-are-the-known-service-limits"></a>¿Cuáles son los límites conocidos del servicio?

Para ver los límites del servicio Azure Firewall, consulte [Suscripción de Azure y límites de servicio, cuotas y restricciones](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-firewall-limits).

## <a name="can-azure-firewall-in-a-hub-virtual-network-forward-and-filter-network-traffic-between-two-spoke-virtual-networks"></a>¿Azure Firewall en la red virtual de un concentrador puede reenviar y filtrar el tráfico de red entre dos redes virtuales de radio?

Sí, puede usar Azure Firewall en la red virtual de un concentrador para enrutar y filtrar el tráfico de red entre dos redes virtuales de radio. Las subredes de cada una de las redes virtuales de radio deben tener una ruta definida por el usuario (UDR) que apunte a la instancia de Azure Firewall como puerta de enlace predeterminada para que este escenario funcione correctamente.

## <a name="can-azure-firewall-forward-and-filter-network-traffic-between-subnets-in-the-same-virtual-network-or-peered-virtual-networks"></a>¿Azure Firewall puede reenviar y filtrar el tráfico de red entre subredes de la misma red virtual o redes virtuales emparejadas?

Sí. Sin embargo, es necesario prestar una atención especial al configurar las UDR para redirigir el tráfico entre subredes de la misma red virtual. Aunque basta con usar el intervalo de direcciones de red virtual como prefijo de destino para la UDR, también se enruta todo el tráfico de una máquina a otra de la misma subred mediante la instancia de Azure Firewall. Para evitar este problema, incluya una ruta para la subred en la UDR con un tipo de próximo salto **VNET**. La administración de estas rutas podría ser problemática y estar sujeta a errores. El método recomendado para la segmentación de redes internas es usar grupos de seguridad de red, que no necesitan UDR.

## <a name="does-azure-firewall-outbound-snat-between-private-networks"></a>¿Azure Firewall aplica SNAT de salida entre redes privadas?

Azure Firewall no aplica SNAT cuando la dirección IP de destino es un intervalo de direcciones IP privadas por [IANA RFC 1918](https://tools.ietf.org/html/rfc1918). Si su organización usa un intervalo de direcciones IP públicas para las redes privadas, Azure Firewall aplicará SNAT al tráfico para una de las direcciones IP privadas de firewall en AzureFirewallSubnet. Puede configurar Azure Firewall de modo que **no** aplique SNAT al intervalo de direcciones IP públicas. Para obtener más información, vea [Aplicación de SNAT por parte de Azure Firewall a intervalos de direcciones IP privadas](snat-private-range.md).

## <a name="is-forced-tunnelingchaining-to-a-network-virtual-appliance-supported"></a>¿Se admite la tunelización o encadenamiento forzados a una aplicación virtual de red?

La tunelización forzada se admite cuando se crea un nuevo Firewall. No se puede configurar un firewall existente para la tunelización forzada. Para más información, consulte [Tunelización forzada de Azure Firewall](forced-tunneling.md).

Azure Firewall debe tener conectividad directa a Internet. Si AzureFirewallSubnet aprende una ruta predeterminada a la red local mediante BGP, debe reemplazarla por una UDR 0.0.0.0/0 con el valor **NextHopType** establecido como **Internet** para mantener la conectividad directa a Internet.

Si la configuración requiere tunelización forzada a una red local y puede determinar los prefijos de las direcciones IP de destino de los destinos de Internet, puede configurar estos intervalos con la red local como el próximo salto mediante una ruta definida por el usuario en AzureFirewallSubnet. O bien, puede usar BGP para definir estas rutas.

## <a name="are-there-any-firewall-resource-group-restrictions"></a>¿Existe alguna restricción de grupo de recursos de firewall?

Sí. El firewall, la red virtual y la dirección IP pública deben estar en el mismo grupo de recursos.

## <a name="when-configuring-dnat-for-inbound-internet-network-traffic-do-i-also-need-to-configure-a-corresponding-network-rule-to-allow-that-traffic"></a>Al configurar DNAT para el tráfico de Internet entrante, ¿es necesario que también configure una regla de red correspondiente para permitir ese tráfico?

No. Las reglas NAT agregan de forma implícita una regla de red correspondiente implícita para permitir el tráfico traducido. Para invalidar este comportamiento, agregue explícitamente una colección de reglas de red con reglas de denegación que coinciden con el tráfico traducido. Para más información acerca de la lógica de procesamiento de reglas Azure Firewall, consulte [Lógica de procesamiento de reglas de Azure Firewall](rule-processing.md).

## <a name="how-do-wildcards-work-in-an-application-rule-target-fqdn"></a>¿Cómo funcionan los caracteres comodín en un FQDN de destino de la regla de aplicación?

Actualmente, solo se pueden usar caracteres comodín en el lado izquierdo del nombre de dominio completo. Por ejemplo, * **.contoso.com** y ***contoso.com**.

Si configura * **.contoso.com**, permite *cualquiervalor*.contoso.com, pero no contoso.com (el vértice de dominio). Si desea permitir el vértice de dominio, debe configurarlo explícitamente como un FQDN de destino.

## <a name="what-does-provisioning-state-failed-mean"></a>¿Qué significa *Estado de aprovisionamiento: error*?

Cada vez que se aplique un cambio de configuración, Azure Firewall intenta actualizar todas sus instancias de back-end subyacentes. En raras ocasiones, una de estas instancias de back-end puede producir un error al actualizar con la nueva configuración y el proceso de actualización se detiene con un estado de aprovisionamiento con error. Azure Firewall todavía está operativo, pero es posible que la configuración aplicada se encuentre en un estado incoherente, donde algunas instancias tienen la configuración anterior y otras tienen el conjunto de reglas actualizado. Si esto sucede, intente actualizar la configuración una vez más, hasta que la operación se realice correctamente y el firewall se encuentre en un estado de aprovisionamiento *Correcto*.

## <a name="how-does-azure-firewall-handle-planned-maintenance-and-unplanned-failures"></a>¿Cómo controla Azure Firewall el mantenimiento planeado y los errores no planeados?
Azure Firewall consta de varios nodos de back-end en una configuración activo-activo.  Para cualquier mantenimiento planeado, se dispone de lógica de purga de la conexión para actualizar los nodos correctamente.  Las actualizaciones se planean durante el horario no comercial de cada una de las regiones de Azure para limitar aún más el riesgo de interrupción.  Si surgen problemas imprevistos, se crea una instancia de un nuevo nodo para reemplazar el nodo con errores.  Normalmente, la conectividad con el nuevo nodo se restablece en 10 segundos desde el momento en que se produjo el error.

## <a name="how-does-connection-draining-work"></a>¿Cómo funciona la purga de conexiones?

Para cualquier mantenimiento planeado, la lógica de purga de conexiones actualiza los nodos de back-end correctamente. Azure Firewall espera 90 segundos para que se cierren las conexiones existentes. Si es necesario, los clientes pueden volver a establecer la conectividad automáticamente con otro nodo de back-end.

## <a name="is-there-a-character-limit-for-a-firewall-name"></a>¿Hay un límite de caracteres para un nombre de firewall?

Sí. Hay un límite de 50 caracteres para un nombre de firewall.

## <a name="why-does-azure-firewall-need-a-26-subnet-size"></a>¿Por qué Azure Firewall necesita un tamaño de subred de /26?

Azure Firewall debe aprovisionar más instancias de máquinas virtuales a medida que se escalan. Un espacio de direcciones /26 garantiza que el firewall tiene suficientes direcciones IP disponibles para acomodar el escalado.

## <a name="does-the-firewall-subnet-size-need-to-change-as-the-service-scales"></a>¿Es necesario cambiar el tamaño de la subred del firewall a medida que se escala el servicio?

No. Azure Firewall no necesita una subred mayor que /26.

## <a name="how-can-i-increase-my-firewall-throughput"></a>¿Cómo puedo aumentar el rendimiento del firewall?

La capacidad de rendimiento inicial de Azure Firewall es de 2,5 a 3 Gbps, y se escala horizontalmente hasta 30 Gbps. Se escala horizontalmente de manera automática en función del uso de CPU y el rendimiento.

## <a name="how-long-does-it-take-for-azure-firewall-to-scale-out"></a>¿Cuánto tiempo tarda Azure Firewall en escalar horizontalmente?

Azure Firewall se escala gradualmente cuando el rendimiento medio o el consumo de CPU es del 60 %. Un rendimiento máximo de implementación predeterminado es aproximadamente 2,5-3 Gbps y comienza a escalar horizontalmente cuando alcanza el 60 % de esa cifra. El escalado horizontal tarda entre cinco y siete minutos. 

Cuando realice pruebas de rendimiento, asegúrese de que lleva a cabo una prueba de 10 a 15 minutos como mínimo e inicie nuevas conexiones para aprovechar los nodos de Firewall recién creados.

## <a name="does-azure-firewall-allow-access-to-active-directory-by-default"></a>¿Permite Azure Firewall el acceso a Active Directory de forma predeterminada?

No. Azure Firewall bloquea el acceso a Active Directory de forma predeterminada. Para permitir el acceso, configure la etiqueta de servicio AzureActiveDirectory. Para más información, consulte [Etiquetas de servicio de Azure Firewall](service-tags.md).

## <a name="can-i-exclude-a-fqdn-or-an-ip-address-from-azure-firewall-threat-intelligence-based-filtering"></a>¿Puedo excluir un FQDN o una dirección IP del filtrado basado en inteligencia sobre amenazas de Azure Firewall?

Sí, puede usar Azure PowerShell para ello:

```azurepowershell
# Add a Threat Intelligence allow list to an Existing Azure Firewall

## Create the allow list with both FQDN and IPAddresses

$fw = Get-AzFirewall -Name "Name_of_Firewall" -ResourceGroupName "Name_of_ResourceGroup"
$fw.ThreatIntelWhitelist = New-AzFirewallThreatIntelWhitelist `
   -FQDN @("fqdn1", "fqdn2", …) -IpAddress @("ip1", "ip2", …)

## Or Update FQDNs and IpAddresses separately

$fw = Get-AzFirewall -Name $firewallname -ResourceGroupName $RG
$fw.ThreatIntelWhitelist.IpAddresses = @($fw.ThreatIntelWhitelist.IpAddresses + $ipaddresses)
$fw.ThreatIntelWhitelist.fqdns = @($fw.ThreatIntelWhitelist.fqdns + $fqdns)


Set-AzFirewall -AzureFirewall $fw
```

## <a name="why-can-a-tcp-ping-and-similar-tools-successfully-connect-to-a-target-fqdn-even-when-no-rule-on-azure-firewall-allows-that-traffic"></a>¿Por qué un ping de TCP y herramientas similares se conectan correctamente a un FQDN de destino incluso si ninguna regla de Azure Firewall permite ese tráfico?

Un ping de TCP no se conecta realmente al FQDN de destino. Esto ocurre porque el proxy transparente de Azure Firewall escucha en el puerto 80/443 para el tráfico saliente. El ping de TCP establece una conexión con el firewall, que, a continuación, quita el paquete y registra la conexión. Este comportamiento no afecta a la seguridad. Sin embargo, para evitar confusiones, estamos investigando posibles cambios en este comportamiento.

## <a name="are-there-limits-for-the-number-of-ip-addresses-supported-by-ip-groups"></a>¿Existen límites para el número de direcciones IP admitidas por los grupos de IP?

Sí. Para más información, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-firewall-limits).

## <a name="can-i-move-an-ip-group-to-another-resource-group"></a>¿Puedo trasladar un grupo de direcciones IP a otro grupo de recursos?

No, actualmente no se permite mover un grupo de direcciones IP a otro grupo de recursos.

## <a name="what-is-the-tcp-idle-timeout-for-azure-firewall"></a>¿Cuál es el tiempo de espera de inactividad de TCP para Azure Firewall?

Un comportamiento estándar de un firewall de red es asegurarse de que las conexiones TCP se mantengan activas y cerrarlas de inmediato si no hay ninguna actividad. El tiempo de espera de inactividad de TCP de Azure Firewall es de cuatro minutos. Este valor no se puede configurar. Si un período de inactividad es mayor que el valor de tiempo de espera, no hay ninguna garantía de que todavía exista la sesión TCP o HTTP. Una práctica común es usar TCP Keep-alive. Esta práctica mantiene la conexión activa durante un periodo más largo. Para más información, consulte estos [ejemplos de .NET](https://docs.microsoft.com/dotnet/api/system.net.servicepoint.settcpkeepalive?redirectedfrom=MSDN&view=netcore-3.1#System_Net_ServicePoint_SetTcpKeepAlive_System_Boolean_System_Int32_System_Int32_).

## <a name="can-i-deploy-azure-firewall-without-a-public-ip-address"></a>¿Puedo implementar Azure Firewall sin una dirección IP pública?

No, actualmente debe implementar Azure Firewall con una dirección IP pública.

## <a name="where-does-azure-firewall-store-customer-data"></a>¿Dónde se almacenan los datos de los clientes en Azure Firewall?

Azure Firewall no mueve ni almacena los datos de los clientes fuera de la región en los que se implementan.
