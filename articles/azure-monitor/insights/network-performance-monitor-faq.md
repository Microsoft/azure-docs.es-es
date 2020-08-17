---
title: 'Preguntas más frecuentes: Solución Network Performance Monitor en Azure | Microsoft Docs'
description: En este artículo se incluyen las preguntas más frecuentes sobre Network Performance Monitor en Azure. Network Performance Monitor (NPM) ayuda a supervisar el rendimiento de sus redes (en tiempo casi real) para detectar y localizar los cuellos de botella de rendimiento que afectan a la red.
ms.subservice: logs
ms.topic: conceptual
author: vinynigam
ms.author: vinigam
ms.date: 10/12/2018
ms.openlocfilehash: 45a10ddce165626bfbadb0ba0b3d68b81709c3bb
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326144"
---
# <a name="network-performance-monitor-solution-faq"></a>Preguntas más frecuentes sobre la solución Network Performance Monitor.

![Símbolo de Network Performance Monitor](media/network-performance-monitor-faq/npm-symbol.png)

En este artículo se incluyen las preguntas más frecuentes (P+f) sobre Network Performance Monitor (NPM) en Azure.

[Network Performance Monitor](../../networking/network-monitoring-overview.md) es una solución de [supervisión de red](./network-performance-monitor-performance-monitor.md) híbrida basada en la nube que le ayuda a supervisar el rendimiento de red entre distintos puntos en la infraestructura de red. También le permite supervisar la conectividad de red con los [puntos de conexión del servicio y la aplicación](./network-performance-monitor-service-connectivity.md) y [supervisar el rendimiento de Azure ExpressRoute](./network-performance-monitor-expressroute.md). 

Network Performance Monitor detecta posibles problemas de red, como enrutamiento del tráfico a "agujeros negros", errores de enrutamiento y problemas que los métodos de supervisión de red convencionales no pueden encontrar. La solución genera alertas y le notifica cómo y cuándo se supera un umbral en un vínculo de red. También garantiza la detección oportuna de los problemas de rendimiento de red y localiza el origen del problema en un segmento o dispositivo de red determinado. 

Hay más información sobre las diferentes funcionalidades compatibles con [Network Performance Monitor](../../networking/network-monitoring-overview.md) disponible en línea.

## <a name="set-up-and-configure-agents"></a>Instalar y configurar agentes

### <a name="what-are-the-platform-requirements-for-the-nodes-to-be-used-for-monitoring-by-npm"></a>¿Cuáles son los requisitos de plataforma para los nodos que se van a usar para la supervisión por parte de NPM?
A continuación se muestran los requisitos de plataforma para diversas funcionalidades de NPM:

- Las funcionalidades Monitor de rendimiento y Monitor de conectividad de servicio de NPM admiten tanto Windows Server como escritorios/sistemas operativos cliente Windows. Las versiones admitidas del sistema operativo Windows Server son 2008 SP1 y posteriores. Las versiones admitidas de cliente o escritorios de Windows son Windows 10, Windows 8.1, Windows 8 y Windows 7. 
- La funcionalidad Supervisión de ExpressRoute de NPM solo admite el sistema operativo Windows Server (2008 SP1 o posterior).

### <a name="can-i-use-linux-machines-as-monitoring-nodes-in-npm"></a>¿Se pueden usar máquinas Linux como nodos de supervisión en NPM?
La funcionalidad de supervisión de redes con nodos basados en Linux se encuentra actualmente en versión preliminar. Póngase en contacto con el administrador de cuentas para obtener más información. Los agentes de Linux solo ofrecen funcionalidad de supervisión con la funcionalidad Monitor de rendimiento de NPM y no está disponible para las funcionalidades Monitor de conectividad de servicio y Supervisión de ExpressRoute.

### <a name="what-are-the-size-requirements-of-the-nodes-to-be-used-for-monitoring-by-npm"></a>¿Cuáles son los requisitos de tamaño de los nodos que se van a usar para la supervisión por parte de NPM?
Para ejecutar la solución NPM en máquinas virtuales de nodo para supervisar redes, los nodos deben tener al menos 500 MB de memoria y un núcleo. No es necesario usar nodos independientes para ejecutar NPM. La solución pueda ejecutarse en nodos en los que haya otras cargas de trabajo en ejecución. La solución tiene la capacidad de detener el proceso de supervisión si usa más del 5 % de CPU.

### <a name="to-use-npm-should-i-connect-my-nodes-as-direct-agent-or-through-system-center-operations-manager"></a>Para usar NPM, ¿debo conectar mis nodos como agente directo o a través de System Center Operations Manager?
Tanto la funcionalidad Monitor de rendimiento como Monitor de conectividad de servicio admite nodos [conectados como agentes directos](../platform/agent-windows.md) y [conectados a través de Operations Manager](../platform/om-agents.md).

En la funcionalidad Supervisión de ExpressRoute, los nodos de Azure solo deben conectarse como agentes directos. Los nodos de Azure, que se conectan a través de Operations Manager, no se admiten. En los nodos locales, se admiten los nodos conectados como agentes directos y a través de Operations Manager para supervisar un circuito ExpressRoute.

### <a name="which-protocol-among-tcp-and-icmp-should-be-chosen-for-monitoring"></a>Entre TCP e ICMP, ¿qué protocolo debe elegirse para la supervisión?
Si va a supervisar su red mediante nodos basados en Windows Server, se recomienda usar TCP como protocolo de supervisión, ya que ofrece mayor precisión. 

Se recomienda ICMP para los nodos basados en el sistema operativo de cliente o escritorios de Windows. Esta plataforma no admite que se envíen datos TCP a través de sockets sin procesar, que NPM usa para detectar la topología de red.

Puede obtener más detalles sobre las ventajas relativas de cada protocolo [aquí](./network-performance-monitor-performance-monitor.md#choose-the-protocol).

### <a name="how-can-i-configure-a-node-to-support-monitoring-using-tcp-protocol"></a>¿Cómo se puede configurar un nodo para que admita la supervisión con el protocolo TCP?
Para que el nodo admita la supervisión con el protocolo TCP: 
* Asegúrese de que la plataforma de nodos sea Windows Server (2008 SP1 o posterior).
* Ejecute el script [EnableRules.ps1](https://aka.ms/npmpowershellscript) de Powershell en el nodo. Vea las [instrucciones](./network-performance-monitor.md#configure-log-analytics-agents-for-monitoring) para obtener más detalles.


### <a name="how-can-i-change-the-tcp-port-being-used-by-npm-for-monitoring"></a>¿Cómo se puede cambiar el puerto TCP que NPM utiliza para la supervisión?
Para cambiar el puerto TCP que NPM utiliza para la supervisión, ejecute el script [EnableRules.ps1](https://aka.ms/npmpowershellscript). Debe especificar el número de puerto que va a utilizar como parámetro. Por ejemplo, para habilitar TCP en el puerto 8060, ejecute `EnableRules.ps1 8060`. Asegúrese de utilizar el mismo puerto TCP en todos los nodos que se usan para la supervisión.

El script configura solo el firewall de Windows localmente. Si tiene reglas de firewall de red o de grupo de seguridad de red (NSG), asegúrese de que permiten el tráfico destinado al puerto TCP que NPM utiliza.

### <a name="how-many-agents-should-i-use"></a>¿Cuántos agentes se deben usar?
Debe usar a al menos un agente para cada subred que quiera supervisar.

### <a name="what-is-the-maximum-number-of-agents-i-can-use-or-i-see-error--youve-reached-your-configuration-limit"></a>¿Cuál es el número máximo de agentes que puedo usar o, si no, aparece el error "… se ha alcanzado el límite de configuración"?
NPM limita el número de direcciones IP a direcciones 5000 por área de trabajo. Si un nodo tiene direcciones IPv4 e IPv6, esto contará como dos direcciones IP para ese nodo. Por lo tanto, el límite de 5000 direcciones IP podría decidir el límite superior en el número de agentes. Puede eliminar los agentes inactivos en la pestaña Nodos en NPM >> Configurar. NPM también mantiene el historial de todas las direcciones IP que se han asignado a la máquina virtual que hospeda el agente. Cada una se cuenta como una dirección IP independiente que contribuye a ese límite de 5000 direcciones IP. Para liberar direcciones IP para el área de trabajo, puede usar la página Nodos para eliminar las direcciones IP que no están en uso.

## <a name="monitoring"></a>Supervisión

### <a name="how-are-loss-and-latency-calculated"></a>Cómo se calculan los valores de pérdida y latencia
Los agentes de origen envían solicitudes SYN de TCP (si se elige TCP como protocolo de supervisión) o solicitudes de eco ICMP (si se elige ICMP como protocolo de supervisión) a la dirección IP de destino a intervalos regulares para asegurarse de que se cubren todas las rutas de acceso en la combinación de direcciones IP de origen y de destino. Para calcular la pérdida y latencia de cada ruta de acceso, se mide el porcentaje de paquetes recibidos y el tiempo de ida y vuelta de los paquetes. Estos datos se agregan a lo largo del intervalo de sondeo y de todas las rutas de acceso para obtener los valores agregados de pérdida y latencia para la combinación de direcciones IP del intervalo de sondeo determinado.

### <a name="with-what-frequency-does-the-source-agent-send-packets-to-the-destination-for-monitoring"></a>¿Con qué frecuencia envía el agente de origen paquetes al destino para su monitorización?
En las funcionalidades Monitor de rendimiento y Supervisión de ExpressRoute, el origen envía paquetes cada cinco segundos y registra las mediciones de red. Estos datos se agregan durante un intervalo de sondeo de tres minutos para calcular los valores promedio y máximo de pérdida y latencia. En la funcionalidad Monitor de conectividad de servicio, la frecuencia de envío de los paquetes para la medición de red viene determinada por la frecuencia especificada por el usuario para la prueba específica durante la configuración de la prueba.

### <a name="how-many-packets-are-sent-for-monitoring"></a>¿Cuántos paquetes se envían para su supervisión?
El número de paquetes enviados por el agente de origen al destino en un sondeo es adaptable y se decide con nuestro algoritmo propietario, que puede ser diferente para las distintas topologías de red. Cuanto mayor sea el número de rutas de acceso de red en la combinación de direcciones IP de origen y destino, mayor será el número de paquetes que se envían. El sistema garantiza que se cubren todas las rutas de acceso en la combinación de direcciones IP de origen y de destino.

### <a name="how-does-npm-discover-network-topology-between-source-and-destination"></a>¿Cómo detecta NPM la topología de red entre el origen y destino?
NPM usa un algoritmo propietario que se basa en Traceroute para detectar todas las rutas de acceso y los saltos entre el origen y destino.

### <a name="does-npm-provide-routing-and-switching-level-info"></a>¿Ofrece NPM información de nivel de conmutación y enrutamiento? 
Aunque NPM puede detectar todas las posibles rutas entre el agente de origen y el destino, no ofrece visibilidad de la ruta que tomaron los paquetes enviados por las cargas de trabajo específicas. La solución puede ayudarle a identificar las rutas de acceso y los saltos de red subyacentes, que agregan más latencia de la que se espera.

### <a name="why-are-some-of-the-paths-unhealthy"></a>¿Por qué algunas de las rutas de acceso son incorrectas?
Pueden existir distintas rutas de acceso de red entre las direcciones IP de origen y destino, y cada ruta de acceso puede tener un valor distinto de pérdida y latencia. NPM marca esas rutas de acceso como incorrectas (como indica el color rojo) para las que los valores de pérdida o latencia son mayores que el correspondiente umbral establecido en la configuración de supervisión.

### <a name="what-does-a-hop-in-red-color-signify-in-the-network-topology-map"></a>¿Qué significa un salto en color rojo en el mapa de topología de red?
Si un salto aparece en rojo, indica que forma parte de al menos una ruta de acceso incorrecta. NPM solo marca las rutas de acceso como incorrectas, no segrega el estado de mantenimiento de cada ruta de acceso. Para identificar los saltos problemáticos, puede ver la latencia de salto a salto y separar los que agregan más latencia de la que se espera.

### <a name="how-does-fault-localization-in-performance-monitor-work"></a>¿Cómo funciona la localización de errores en el Monitor de rendimiento?
NPM usa un mecanismo probabilístico para asignar las probabilidades de error a cada ruta de acceso de red, segmento de red, y a los saltos de red que los componen en función del número de rutas de acceso incorrectas de las que forman parte. A medida que los segmentos de red y los saltos engrosan el número de rutas de acceso incorrectas, aumenta la probabilidad de error asociada a ellos. Este algoritmo funciona mejor cuando se tienen muchos nodos con el agente de NPM conectados entre sí, ya que esto aumenta los puntos de datos para calcular las probabilidades de error.

### <a name="how-can-i-create-alerts-in-npm"></a>¿Cómo se pueden crear alertas en NPM?
Se están produciendo errores al crear alertas desde la interfaz de usuario de NPM debido a un problema. Cree alertas manualmente.

### <a name="what-are-the-default-log-analytics-queries-for-alerts"></a>¿Cuáles son las consultas de Log Analytics predeterminadas para alertas?
Consulta del monitor de rendimiento

```kusto
NetworkMonitoring
 | where (SubType == "SubNetwork" or SubType == "NetworkPath") 
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy") and RuleName == "<<your rule name>>"
```

Consulta del monitor de conectividad de servicio

```kusto
NetworkMonitoring
 | where (SubType == "EndpointHealth" or SubType == "EndpointPath")
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or ServiceResponseHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy") and TestName == "<<your test name>>"
```

Consultas del monitor de ExpressRoute: Consulta de circuitos

```kusto
NetworkMonitoring
 | where (SubType == "ERCircuitTotalUtilization") and (UtilizationHealthState == "Unhealthy") and CircuitResourceId == "<<your circuit resource ID>>"
```

Emparejamiento privado

```kusto
NetworkMonitoring
 | where (SubType == "ExpressRoutePeering" or SubType == "ERVNetConnectionUtilization" or SubType == "ExpressRoutePath")   
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or UtilizationHealthState == "Unhealthy") and CircuitName == "<<your circuit name>>" and VirtualNetwork == "<<vnet name>>"
```

Emparejamiento de Microsoft

```kusto
NetworkMonitoring
 | where (SubType == "ExpressRoutePeering" or SubType == "ERMSPeeringUtilization" or SubType == "ExpressRoutePath")
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or UtilizationHealthState == "Unhealthy") and CircuitName == ""<<your circuit name>>" and PeeringType == "MicrosoftPeering"
```

Consulta común

```kusto
NetworkMonitoring
 | where (SubType == "ExpressRoutePeering" or SubType == "ERVNetConnectionUtilization" or SubType == "ERMSPeeringUtilization" or SubType == "ExpressRoutePath")
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or UtilizationHealthState == "Unhealthy")
```

### <a name="can-npm-monitor-routers-and-servers-as-individual-devices"></a>¿Puede NPM supervisar los enrutadores y los servidores como los dispositivos individuales?
NPM solo identifica el nombre de host y de IP de los saltos de red subyacentes (conmutadores, enrutadores, servidores, etc.) entre las direcciones IP de origen y destino. También identifica la latencia entre estos saltos identificados. No supervisa individualmente estos saltos subyacentes.

### <a name="can-npm-be-used-to-monitor-network-connectivity-between-azure-and-aws"></a>¿Puede NPM usarse para supervisar la conectividad de red entre Azure y AWS?
Sí. Consulte el artículo [Monitor Azure, AWS, and on-premises networks using NPM](/archive/blogs/msoms/monitor-on-premises-cloud-iaas-and-hybrid-networks-using-oms-network-performance-monitor) (Supervisar Azure, AWS y redes locales con NPM) para obtener más detalles.

### <a name="is-the-expressroute-bandwidth-usage-incoming-or-outgoing"></a>¿El uso de ancho de banda de ExpressRoute es entrante o saliente?
El uso de ancho de banda corresponde al total de ancho de banda entrante y saliente. Se expresa en bits por segundo.

### <a name="can-we-get-incoming-and-outgoing-bandwidth-information-for-the-expressroute"></a>¿Se puede obtener información de ancho de banda entrante y saliente de ExpressRoute?
Se pueden capturar los valores entrantes y salientes del ancho de banda principal y secundario.

Para obtener información de nivel de emparejamiento de Microsoft, utilice la consulta que se menciona a continuación en la búsqueda de registros.

```kusto
NetworkMonitoring
 | where SubType == "ERMSPeeringUtilization"
 | project CircuitName,PeeringName,BitsInPerSecond,BitsOutPerSecond 
```

Para obtener información de nivel de emparejamiento privada, utilice la consulta que se menciona a continuación en la búsqueda de registros.

```kusto
NetworkMonitoring
 | where SubType == "ERVNetConnectionUtilization"
 | project CircuitName,PeeringName,BitsInPerSecond,BitsOutPerSecond
```

Para obtener información de nivel de circuito, utilice la consulta que se menciona a continuación en la búsqueda de registros.

```kusto
NetworkMonitoring
 | where SubType == "ERCircuitTotalUtilization"
 | project CircuitName, BitsInPerSecond, BitsOutPerSecond
```

### <a name="which-regions-are-supported-for-npms-performance-monitor"></a>¿Qué regiones se admiten con Monitor de rendimiento de NPM?
NPM puede supervisar la conectividad entre redes en cualquier parte del mundo, desde un área de trabajo que se hospede en una de las [regiones admitidas](./network-performance-monitor.md#supported-regions).

### <a name="which-regions-are-supported-for-npms-service-connectivity-monitor"></a>¿Qué regiones se admiten con Monitor de conectividad de servicio de NPM?
NPM puede supervisar la conectividad con servicios en cualquier parte del mundo, desde un área de trabajo que se hospede en una de las [regiones admitidas](./network-performance-monitor.md#supported-regions).

### <a name="which-regions-are-supported-for-npms-expressroute-monitor"></a>¿Qué regiones se admiten con Supervisión de ExpressRoute de NPM?
NPM puede supervisar los circuitos ExpressRoute ubicados en cualquier región de Azure. Para incorporarse a NPM, requerirá un área de trabajo de Log Analytics que debe hospedarse en una de Las [regiones admitidas](../../expressroute/how-to-npm.md)

## <a name="troubleshoot"></a>Solución de problemas

### <a name="why-are-some-of-the-hops-marked-as-unidentified-in-the-network-topology-view"></a>¿Por qué algunos de los saltos están marcados como no identificados en la vista de topología de red?
NPM usa una versión modificada del comando traceroute para detectar la topología desde el agente de origen al destino. Un salto no identificado indica que el salto de red no respondió a la solicitud de traceroute del agente de origen. Si tres saltos de red consecutivos no responden al comando traceroute del agente, la solución marca los saltos que no responden como no identificados y no intenta detectar más saltos.

Es posible que un salto no responda a un comando traceroute en uno o varios de los escenarios siguientes:

* Los enrutadores se han configurado para no revelar su identidad.
* Los dispositivos de red no permiten el tráfico ICMP_TTL_EXCEEDED.
* Un firewall bloquea la respuesta ICMP_TTL_EXCEEDED del dispositivo de red.

Cuando cualquiera de los puntos de conexión se encuentra en Azure, traceroute muestra saltos no identificados, ya que la infraestructura de Azure no revela la identidad a traceroute. 

### <a name="i-get-alerts-for-unhealthy-tests-but-i-do-not-see-the-high-values-in-npms-loss-and-latency-graph-how-do-i-check-what-is-unhealthy"></a>Recibo alertas para las pruebas con un estado incorrecto, pero no veo los valores altos en el gráfico de pérdida y latencia de NPM. ¿Cómo compruebo qué elementos tienen un estado incorrecto?
NPM genera una alerta si la latencia de extremo a extremo entre el origen y el destino cruza el umbral en cualquiera de las rutas de acceso entre ellos. Algunas redes tienen varias rutas de acceso que conectan el mismo origen y destino. NPM genera una alerta si el estado de alguna de las rutas de acceso es incorrecto. La pérdida y latencia que se muestran en los gráficos es el valor medio de todas las rutas de acceso, por lo que puede que no muestre el valor exacto de una única ruta de acceso. Para saber dónde se ha superado el umbral, busque la columna "SubType" en la alerta. Si el problema se debe a una ruta de acceso, el valor de SubType será NetworkPath (para las pruebas del Monitor de rendimiento), EndpointPath (para las pruebas del Monitor de conectividad de servicio) y ExpressRoutePath (para las pruebas del Monitor de ExpressRotue). 

Consulta de ejemplo para buscar la ruta de acceso incorrecta:

```kusto
NetworkMonitoring
 | where ( SubType == "ExpressRoutePath")
 | where (LossHealthState == "Unhealthy" or LatencyHealthState == "Unhealthy" or UtilizationHealthState == "Unhealthy") and CircuitResourceID =="<your ER circuit ID>" and ConnectionResourceId == "<your ER connection resource id>"
 | project SubType, LossHealthState, LatencyHealthState, MedianLatency
```

### <a name="why-does-my-test-show-unhealthy-but-the-topology-does-not"></a>¿Por qué mi prueba muestra un estado incorrecto pero la topología no? 
NPM supervisa la topología, la latencia y la pérdida de un extremo a otro en intervalos diferentes. La pérdida y la latencia se miden una vez cada cinco segundos y se agregan cada tres minutos (para el Monitor de rendimiento y la Supervisión de ExpressRoute), mientras que la topología se calcula mediante el comando traceroute una vez cada diez minutos. Por ejemplo, entre las 03:44 y las 04:04, la topología podría actualizarse tres veces (a las 03:44, 03:54 y 04:04), pero la pérdida y la latencia se actualizan unas siete veces (a las 03:44, 03:47, 03:50, 03:53, 03:56, 03:59 y 04:02). La topología que se genera a las 03:54 se representará para la pérdida y la latencia calculadas a las 03:56, 03:59 y 04:02. Supongamos que recibe una alerta que indica que el circuito ER se encontraba en un estado incorrecto a las 03:59. Inicia sesión en NPM e intenta establecer la hora de la topología a las 03:59. NPM representará la topología que se generó a las 03:54. Para entender la última topología conocida de la red, compare los campos TimeProcessed (hora a la que se calcularon la pérdida y la latencia) y TracerouteCompletedTime (hora a la que se calculó la topología). 

### <a name="what-is-the-difference-between-the-fields-e2emedianlatency-and-avghoplatencylist-in-the-networkmonitoring-table"></a>¿Cuál es la diferencia entre los campos E2EMedianLatency y AvgHopLatencyList en la tabla NetworkMonitoring?
E2EMedianLatency es la latencia actualizada cada tres minutos después de agregar los resultados de las pruebas de ping basadas en TCP, mientras que AvgHopLatencyList se actualiza cada diez minutos en función del comando traceroute. Para entender la hora exacta a la que se calculó E2EMedianLatency, use el campo TimeProcessed. Para entender la hora exacta a la que el comando traceroute completó y actualizó AvgHopLatencyList, use el campo TracerouteCompletedTime.

### <a name="why-does-hop-by-hop-latency-numbers-differ-from-hoplatencyvalues"></a>¿Por qué las cifras de latencia de salto a salto son diferentes de HopLatencyValues? 
HopLatencyValues son el origen de un punto de conexión.
Por ejemplo: saltos: A,B,C; AvgHopLatency: 10,15,20. Esto significa que el origen de la latencia A es 10, el origen de la latencia B es 15 y el origen de la latencia C es 20. La interfaz de usuario calculará que la latencia del salto A-B es 5 en la topología.

### <a name="the-solution-shows-100-loss-but-there-is-connectivity-between-the-source-and-destination"></a>La solución muestra una pérdida del 100 % pero hay conectividad entre el origen y destino
Esto puede ocurrir si el firewall del host o el firewall intermedio (firewall de red o Azure NSG) bloquea la comunicación entre el agente de origen y el destino a través del puerto que NPM utiliza para la supervisión (de manera predeterminada el puerto es el 8084, a menos que el cliente lo haya cambiado).

* Para comprobar que el firewall del host no bloquea la comunicación en el puerto requerido, vea el estado de mantenimiento de los nodos de origen y destino desde la siguiente vista: Network Performance Monitor -> Configuración -> Nodos. 
  Si son incorrectos, vea las instrucciones y realice acciones correctivas. Si los nodos son correctos, siga en el paso b. a continuación.
* Para comprobar que un firewall de red intermedio o NSG de Azure no bloquean la comunicación en el puerto necesario, emplee la utilidad de terceros PsPing con las instrucciones siguientes:
  * La utilidad psping está disponible para su descarga [aquí](/sysinternals/downloads/psping). 
  * Ejecute el siguiente comando en el nodo de origen.
    * psping -n 15 \<destination node IPAddress\>:númeroPuerto De forma predeterminada, NPM usa el puerto 8084. En caso de que lo haya cambiado explícitamente mediante el script EnableRules.ps1, escriba el número de puerto personalizado que utiliza). Se trata de un ping de la máquina de Azure al entorno local.
* Compruebe si los pings son correctos. Si no es así, indica que un firewall de red intermedio o un grupo de seguridad de red de Azure bloquean el tráfico en este puerto.
* Ahora, ejecute el comando desde la dirección IP del nodo de destino a la del nodo de origen.


### <a name="there-is-loss-from-node-a-to-b-but-not-from-node-b-to-a-why"></a>Hay pérdida del nodo A a B, pero no del nodo B a A. ¿Por qué?
Como las rutas de acceso de red entre A y B pueden ser diferentes de las rutas de acceso de red entre B y A, se pueden observar diferentes valores de pérdida y latencia.

### <a name="why-are-all-my-expressroute-circuits-and-peering-connections-not-being-discovered"></a>¿Por qué no se detectan todos mis circuitos ExpressRoute y las conexiones de emparejamiento?
NPM ahora detecta los circuitos ExpressRoute y las conexiones de emparejamiento en todas las suscripciones a la que el usuario tiene acceso. Elija todas las suscripciones donde los recursos de Express Route están vinculados y habilite la supervisión de cada recurso detectado. NPM busca objetos de conexión durante la detección de un emparejamiento privado, por tanto, compruebe si hay una red virtual asociada con el emparejamiento. NPM no detecta los circuitos y el emparejamiento que se encuentran en otro inquilino del área de trabajo de Log Analytics.

### <a name="the-er-monitor-capability-has-a-diagnostic-message-traffic-is-not-passing-through-any-circuit-what-does-that-mean"></a>La funcionalidad del monitor ER tiene un mensaje de diagnóstico "Traffic is not passing through ANY circuit" (El tráfico no pasa por NINGÚN circuito). ¿Qué implica esto?

Puede que haya un escenario donde exista una conexión correcta entre los nodos locales y de Azure, pero el tráfico no pase a través del circuito ExpressRoute configurado para que NPM lo supervise. 

Esto puede suceder si:

* El circuito ER está inactivo.
* Los filtros de ruta se configuran de tal manera que dan prioridad a otras rutas (por ejemplo, una conexión VPN u otro circuito ExpressRoute) a través del circuito ExpressRoute previsto. 
* Los nodos locales y de Azure elegidos para supervisar el circuito ExpressRoute en la configuración de supervisión no tienen conectividad entre sí a través del circuito ExpressRoute previsto. Asegúrese de que ha elegido nodos correctos que tengan conectividad entre sí a través del circuito ExpressRoute que va a supervisar.

### <a name="why-does-expressroute-monitor-report-my-circuitpeering-as-unhealthy-when-it-is-available-and-passing-data"></a>¿Por qué el monitor de ExpressRoute notifica mi circuito o emparejamiento como incorrecto cuando está disponible y pasa datos?
El monitor de ExpressRoute compara los valores de rendimiento de red (pérdida, latencia y uso de ancho de banda) notificados por los agentes o servicios con los umbrales establecidos durante la configuración. En el caso de un circuito, si el uso de ancho de banda indicado es mayor que el umbral establecido en la configuración, el circuito se marca como incorrecto. En el caso de los emparejamientos, si la pérdida, la latencia o el uso de ancho de banda notificado es mayor que el umbral establecido en la configuración, el emparejamiento se marca como incorrecto. NPM no usa métricas ni ningún otro tipo de datos para decidir el estado de mantenimiento.

### <a name="why-does-expressroute-monitorbandwidth-utilisation-report-a-value-differrent-from-metrics-bits-inout"></a>¿Por qué el uso de ancho de banda del monitor de ExpressRoute notifica un valor distinto de los bits de métrica de entrada y salida?
Para el monitor de ExpressRoute, el uso de ancho de banda es el promedio de ancho de banda de entrada y salida de los últimos 20 minutos que se expresa en bits por segundo. En el caso de las métricas de ExpressRoute, los bits de entrada y salida son por puntos de datos por minuto. De manera interna, se usa el mismo conjunto de datos, pero la agregación varía entre las métricas de NPM y ER. Para obtener una supervisión detallada por minuto y alertas rápidas, se recomienda establecer las alertas directamente en las métricas de ER

### <a name="while-configuring-monitoring-of-my-expressroute-circuit-the-azure-nodes-are-not-being-detected"></a>Al configurar la supervisión de mi circuito ExpressRoute, no se detectan los nodos de Azure.
Esto puede suceder si los nodos de Azure se conectan a través de Operations Manager. La funcionalidad Supervisión de ExpressRoute solo admite los nodos de Azure que se conectan como agentes directos.

### <a name="i-cannot-discover-by-expressroute-circuits-in-the-oms-portal"></a>No se puede detectar mediante los circuitos ExpressRoute en el portal de OMS
Aunque NPM se puede usar tanto en Azure Portal como en el portal de OMS, la detección de circuitos de la funcionalidad Supervisión de ExpressRoute solo funciona a través de Azure Portal. Una vez que los circuitos se detectan en Azure Portal, se puede usar la funcionalidad en cualquiera de los dos portales. 

### <a name="in-the-service-connectivity-monitor-capability-the-service-response-time-network-loss-as-well-as-latency-are-shown-as-na"></a>En la funcionalidad Monitor de conectividad de servicio, el tiempo de respuesta del servicio, la pérdida de red y la latencia aparecen como N/D.
Esto puede suceder si se cumple una o más de las siguientes condiciones:

* El servicio está inactivo.
* El nodo que se usa para comprobar la conectividad de red con el servicio está inactivo.
* El destino que se escribió en la configuración de prueba no es correcto.
* El nodo no tiene conectividad de red ninguna.

### <a name="in-the-service-connectivity-monitor-capability-a-valid-service-response-time-is-shown-but-network-loss-as-well-as-latency-are-shown-as-na"></a>En la funcionalidad Monitor de conectividad de servicio, se muestra un tiempo de respuesta del servicio válido, pero la pérdida de red y la latencia aparecen como N/D.
 Esto puede suceder si se cumple una o más de las siguientes condiciones:

* Si el nodo que se usa para comprobar la conectividad de red con el servicio es un equipo cliente Windows, independientemente de si el servicio de destino bloquea las solicitudes ICMP o un firewall de red bloquea las solicitudes ICMP que se originan del nodo.
* La casilla Realizar mediciones de red está en blanco en la configuración de prueba.

### <a name="in-the-service-connectivity-monitor-capability-the-service-response-time-is-na-but-network-loss-as-well-as-latency-are-valid"></a>En la funcionalidad Monitor de conectividad de servicio, el tiempo de respuesta del servicio aparece como N/D, pero la pérdida de red y la latencia son válidos.
Esto puede suceder si el servicio de destino no es una aplicación web pero la prueba está configurada como prueba web. Edite la configuración de prueba y elija red como tipo de prueba en lugar de web.

## <a name="miscellaneous"></a>Varios

### <a name="is-there-a-performance-impact-on-the-node-being-used-for-monitoring"></a>¿Hay algún impacto en el rendimiento del nodo que se usa para la supervisión?
El proceso de NPM está configurado para detenerse en caso de que utilice más del 5 % de los recursos de CPU del host. Con esto se garantiza que se pueden seguir usando los nodos para las cargas de trabajo habituales sin afectar al rendimiento.

### <a name="does-npm-edit-firewall-rules-for-monitoring"></a>¿Edita NPM las reglas de firewall para la supervisión?
NPM solo crea una regla local de Firewall de Windows en los nodos en los que se ejecuta el script EnableRules.ps1 de Powershell para permitir que los agentes establezcan conexiones TCP entre sí en el puerto especificado. La solución no modifica las reglas de firewall de red o de grupo de seguridad de red (NSG).

### <a name="how-can-i-check-the-health-of-the-nodes-being-used-for-monitoring"></a>¿Cómo se puede comprobar el mantenimiento de los nodos que se usan para la supervisión?
Puede ver el estado de mantenimiento de los nodos que se usan para la supervisión en la siguiente vista: Network Performance Monitor -> Configuración -> Nodos. Si un nodo es incorrecto, puede ver los detalles del error y realizar la acción sugerida.

### <a name="can-npm-report-latency-numbers-in-microseconds"></a>¿Puede NPM informar de las cifras de latencia en microsegundos?
NPM redondea los números de latencia en la interfaz de usuario y en milisegundos. Los mismos datos se almacenan con una granularidad superior (a veces hasta cuatro posiciones decimales).

## <a name="next-steps"></a>Pasos siguientes

- Puede obtener más información sobre Network Performance Monitor en [Solución Network Performance Monitor en Azure](./network-performance-monitor.md).

