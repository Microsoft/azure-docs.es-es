---
title: Monitor de rendimiento en Network Performance Monitor
description: La funcionalidad Monitor de rendimiento de Network Performance Monitor ayuda a supervisar la conectividad de red en varios puntos de esta. Puede supervisar la conectividad de red en implementaciones en la nube y ubicaciones locales, varios centros de datos y sucursales, y aplicaciones o microservicios críticos de varios niveles.
ms.subservice: logs
ms.topic: conceptual
author: abshamsft
ms.author: absha
ms.date: 02/20/2018
ms.openlocfilehash: 2c20f668c57e8c2e74c320ca535aea18e633daf8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326127"
---
# <a name="network-performance-monitor-solution-performance-monitoring"></a>Solución Network Performance Monitor: Supervisión de rendimiento

La funcionalidad Monitor de rendimiento de [Network Performance Monitor](network-performance-monitor.md) ayuda a supervisar la conectividad de red en varios puntos de esta. Puede supervisar la conectividad de red en implementaciones en la nube y ubicaciones locales, varios centros de datos y sucursales, y aplicaciones o microservicios críticos de varios niveles. Con el Monitor de rendimiento, puede detectar problemas de red antes de recibir quejas de los usuarios. Las principales ventajas son que se puede: 

- Supervisar la latencia y la pérdida en varias redes virtuales, y establecer alertas.
- Supervisar todas las rutas de acceso (incluidas las redundantes) de la red.
- Solucionar problemas de red transitorios y puntuales que son difíciles de replicar.
- Determinar el segmento específico de la red responsable de la disminución del rendimiento.
- Supervisar el mantenimiento de la red, sin necesidad de SNMP.


![Monitor de rendimiento de red](media/network-performance-monitor-performance-monitor/npm-performance-monitor.png)

## <a name="configuration"></a>Configuración
Para abrir la configuración de Network Performance Monitor, abra la [solución Network Performance Monitor](network-performance-monitor.md) y seleccione **Configurar**.

![Configuración de Network Performance Monitor](media/network-performance-monitor-performance-monitor/npm-configure-button.png)

### <a name="create-new-networks"></a>Creación de nuevas redes

En Network Performance Monitor, una red es un contenedor lógico de subredes. Le ayuda a organizar la supervisión de la infraestructura de red según sus necesidades de supervisión. Puede crear una red con un nombre descriptivo y agregar subredes según la lógica de negocios. Por ejemplo, puede crear una red denominada Londres y agregar todas las subredes de su centro de datos de Londres. O bien puede crear una red denominada *ContosoFrontEnd* y agregar a esta red todas las subredes denominadas Contoso que actúen de front-end de la aplicación. La solución crea automáticamente una red predeterminada que contiene todas las subredes detectadas en su entorno. 

Siempre que cree una red, agregue una subred. A continuación, esa subred se quita de la red predeterminada. Si elimina una red, se devuelven automáticamente todas sus subredes a la red predeterminada. Así pues, la red predeterminada actúa como contenedor de todas las subredes que no se incluyan en alguna red definida por el usuario. No puede editar ni eliminar la red predeterminada. Siempre permanece en el sistema. Sin embargo, puede crear todas las redes personalizadas que necesite. En la mayoría de los casos, las subredes de la organización se organizan en más de una red. Cree una o más redes para agrupar las subredes para la lógica de negocios.

Para crear una nueva red:


1. Seleccione la pestaña **Redes**.
1. Seleccione  **Agregar red** y escriba el nombre de red y la descripción. 
2. Seleccione una o varias subredes y **Agregar**. 
3. Para guardar la configuración, seleccione **Guardar**. 


### <a name="create-monitoring-rules"></a>Creación de reglas de supervisión 

El Monitor de rendimiento genera eventos de estado cuando se supera el umbral del rendimiento de las conexiones de red entre dos subredes o entre dos redes. El sistema puede obtener información sobre estos umbrales automáticamente. También puede proporcionar umbrales personalizados. El sistema crea también automáticamente una regla predeterminada, que genera un evento de mantenimiento siempre que la pérdida o la latencia entre cualquier par de vínculos de red o subred supere el umbral que conoce el sistema. Este proceso ayuda a la solución a supervisar la infraestructura de red hasta que haya creado reglas explícitas de supervisión. Si está habilitada la regla predeterminada, todos los nodos envían transacciones sintéticas al resto de nodos que se han habilitado para la supervisión. La regla predeterminada es útil con redes pequeñas. Por ejemplo, en un escenario donde tiene un número pequeño de servidores que ejecutan un microservicio y desea asegurarse de que todos los servidores tengan conectividad entre ellos.

>[!NOTE]
> Es recomendable que deshabilite la regla predeterminada y cree reglas de supervisión personalizadas, especialmente en el caso de las redes de gran tamaño, donde se usa un número importante de nodos para la supervisión. Las reglas de supervisión personalizadas reducen el tráfico que genera la solución y ayuda a organizar la supervisión de la red.

Cree reglas de supervisión de acuerdo con la lógica de negocios. Por ejemplo, si desea supervisar el rendimiento de la conectividad de red entre dos oficinas y la central. Agrupe todas las subredes de la oficina 1 en la red O1. Después, agrupe todas las subredes de la oficina 2 en la red O2. Por último, agrupe todas las subredes de la central en la red H. Cree dos reglas de supervisión: una entre O1 y H, y la otra entre O2 y H. 

Para crear reglas de supervisión personalizadas:

1. Seleccione **Agregar regla** en la pestaña **Supervisión**, y escriba el nombre y la descripción de la regla.
2. Seleccione en la lista el par de vínculos de red o subred que desee supervisar. 
3. Seleccione la red que contiene las subredes que desee de la lista desplegable de redes. A continuación, seleccione las subredes en la lista desplegable de subredes correspondiente. Seleccione **Todas las subredes** si desea supervisar todas las subredes de un vínculo de red. De forma similar, seleccione las demás subredes que le interesen. Seleccione **Agregar excepción** para excluir de la supervisión determinados vínculos de red de la selección que ha realizado. 
4. Elija entre los protocolos ICMP y TCP para ejecutar transacciones sintéticas. 
5. Si no desea crear eventos de estado para los elementos que ha seleccionado, desactive **Habilitar Seguimiento de estado en los vínculos que abarca esta regla**. 
6. Elija las condiciones de supervisión. Para establecer umbrales personalizados para la generación de eventos de mantenimiento, escriba valores de umbral. Siempre que el valor de la condición supere el umbral seleccionado para el par de red/subred seleccionado, se generará un evento de mantenimiento. 
7. Para guardar la configuración, seleccione **Guardar**. 

Después de guardar una regla de supervisión, puede integrar esa regla con Alert Management; para ello, seleccione **Crear alerta**. Con la consulta de búsqueda se crea automáticamente una regla de alerta. Otros parámetros necesarios se rellenan automáticamente. Mediante una regla de alerta, puede recibir alertas de correo electrónico, además de las alertas existentes dentro de Network Performance Monitor. Las alertas también pueden desencadenar acciones correctoras con runbooks o se pueden integrar con soluciones de administración de servicios existentes mediante webhooks. Seleccione **Administrar alertas** para editar la configuración de las alertas. 

Ahora puede crear más reglas de supervisión del rendimiento o trasladarse al panel de la solución para comenzar a usar esta funcionalidad.

### <a name="choose-the-protocol"></a>Selección del protocolo

Network Performance Monitor utiliza transacciones sintéticas para calcular métricas de rendimiento de red, como la latencia de vínculo y la pérdida de paquetes. Para comprender mejor este concepto, piense en un agente de Network Performance Monitor conectado a un extremo de un vínculo de red. Este agente de Network Performance Monitor envía paquetes de sondeo a un segundo agente de Network Performance Monitor conectado a otro extremo de la red. El segundo agente responde con paquetes de respuesta. Este proceso se repite varias veces. Al medir el número de respuestas y el tiempo necesario para recibir cada respuesta, el primer agente de Network Performance Monitor evalúa la latencia del vínculo y la pérdida de paquetes. 

El formato, el tamaño y la secuencia de estos paquetes se determinan mediante el protocolo que elige al crear reglas de supervisión. En función del protocolo de los paquetes, los dispositivos de red intermedios (enrutadores, conmutadores, etc.) procesan estos paquetes de forma diferente. Por consiguiente, la elección del protocolo afecta a la precisión de los resultados. Asimismo, el protocolo que escoja también determinará si es necesario completar algunos pasos manuales más tras implementar la solución Network Performance Monitor. 

Network Performance Monitor permite elegir entre los protocolos ICMP y TCP para ejecutar transacciones sintéticas. Si elige ICMP al crear una regla de transacciones sintéticas, los agentes de Network Performance Monitor utilizan mensajes de eco ICMP para calcular la latencia de red y la pérdida de paquetes. El eco ICMP usa el mismo mensaje que se envía mediante la utilidad de ping convencional. Al usar TCP como protocolo, los agentes de Network Performance Monitor envían paquetes TCP SYN a través de la red. A este paso le sigue un protocolo de enlace TCP y la conexión se elimina con paquetes RST. 

Antes de elegir un protocolo, tenga en cuenta la siguiente información: 

* **Detección de varias rutas de red.** TCP es más preciso cuando se detectan varias rutas y necesita menos agentes en cada subred. Por ejemplo, uno o dos agentes que utilicen TCP pueden detectar todas las rutas redundantes entre subredes. Se necesitan varios agentes que utilicen ICMP para lograr resultados similares. Con ICMP, si tiene una cantidad de rutas entre dos subredes, necesitará una cantidad 5 veces superior de agentes en una subred de origen o de destino.

* **Precisión de los resultados.** Los enrutadores y conmutadores tienden a asignar una prioridad menor a paquetes de eco ICMP en comparación con los paquetes TCP. En determinadas situaciones, cuando los dispositivos de red están muy cargados, los datos obtenidos mediante TCP reflejan más fielmente la pérdida y la latencia que experimentan las aplicaciones. Esto ocurre porque la mayoría del tráfico de las aplicaciones fluye a través de TCP. En tales casos, ICMP proporciona resultados menos precisos en comparación con TCP. 

* **Configuración del firewall.** El protocolo TCP requiere que los paquetes TCP se envíen a un puerto de destino. El puerto predeterminado que usan los agentes de Network Performance Monitor es 8084. Puede cambiar el puerto al configurar los agentes. Asegúrese de que los firewalls de red o las reglas de grupo de seguridad de red (en Azure) permitan el tráfico en el puerto. También debe asegurarse de que el firewall local en los equipos en los que están instalados los agentes esté configurado para permitir el tráfico en este puerto. Puede utilizar scripts de PowerShell para configurar reglas de firewall en los equipos que ejecutan Windows, pero debe configurar el firewall de red manualmente. En cambio, ICMP no funciona con puertos. En la mayoría de los escenarios empresariales, se permite el tráfico ICMP mediante los firewalls para posibilitar el uso de herramientas de diagnóstico de red, como la utilidad de ping. Por lo tanto, si puede hacer ping a una máquina desde otra, podrá usar el protocolo ICMP sin necesidad de configurar los firewalls manualmente.

>[!NOTE] 
> Algunos firewall pueden bloquear ICMP, lo cual podría provocar una retransmisión que produciría un gran número de eventos en el sistema de administración de eventos e información de seguridad. Asegúrese de que el protocolo que elija no está bloqueado por un firewall de red o grupo de seguridad de red. En caso contrario, Network Performance Monitor no podrá supervisar el segmento de red. Se recomienda usar TCP para la supervisión. Use ICMP en escenarios donde no sea posible utilizar TCP, como los siguientes: 
>
> - Al usar nodos basados en clientes de Windows, dado que los sockets TCP sin formato no se permiten en los clientes de Windows.
> - Cuando el firewall de red o el grupo de seguridad de red bloquea TCP.
> - Si no sabe cambiar el protocolo.

Si decide utilizar ICMP durante la implementación, puede cambiar a TCP en cualquier momento editando la regla de supervisión predeterminada.

1. Vaya a **Rendimiento de red** > **Supervisión** > **Configurar** > **Supervisión**. A continuación, seleccione  **Regla predeterminada**. 
2. Desplácese hasta la sección **Protocolo** y seleccione el protocolo que desee utilizar. 
3. Seleccione **Guardar** para aplicar el cambio. 

Aunque la regla predeterminada use un protocolo específico, puede crear nuevas reglas con un protocolo diferente. Incluso puede crear una combinación de reglas en la que algunas usen ICMP y otras, TCP. 

## <a name="walkthrough"></a>Tutorial 

Ahora examinemos una investigación sencilla de la causa principal de un evento de mantenimiento.

En el panel de la solución, un evento de mantenimiento muestra un vínculo de red con estado incorrecto. Para investigar el problema, seleccione el icono **Vínculos de red que se están supervisando**.

En la página de exploración en profundidad se muestra que el vínculo de red **DMZ2 DMZ1** es incorrecto. Seleccione **View subnet links** (Ver vínculos de subred) de este vínculo de red. 


En la página de exploración en profundidad se muestran todos los vínculos de subred del vínculo de red **DMZ2-DMZ1**. En el caso de ambos vínculos de subred, la latencia ha superado el umbral, lo que provoca que el vínculo de red esté incorrecto. También puede ver las tendencias de latencia de ambos vínculos de subred. Utilice el control de selección de tiempo del grafo para centrarse en el intervalo de tiempo pertinente. Puede ver la hora del día en la que la latencia ha alcanzado su punto máximo. Después, busque en los registros este período para investigar el problema. Seleccione **Ver vínculos de nodo** para profundizar aún más. 
 
 ![Página de vínculos de subred](media/network-performance-monitor-performance-monitor/subnetwork-links.png) 

Como sucedía en la página anterior, en la página de exploración en profundidad del vínculo de subred concreto se enumeran los vínculos de nodo que lo componen. Aquí puede efectuar acciones similares a las realizadas en el paso anterior. Seleccione **Ver topología** para ver la topología entre los dos nodos. 
 
 ![Página de vínculos de nodo](media/network-performance-monitor-performance-monitor/node-links.png) 

Todas las rutas de acceso entre los dos nodos seleccionados estarán trazadas en el mapa de topología. Puede visualizar la topología de salto a salto de las rutas entre dos nodos en el mapa de topología. Ofrece una perspectiva clara de cuántas rutas existen entre los dos nodos y qué rutas de acceso emplean los paquetes de datos. Los cuellos de botella de rendimiento de red aparecen en rojo. Para encontrar una conexión o dispositivo de red defectuosos, examine los elementos de color rojo presentes en el mapa de topología. 

 ![Panel de topología con el mapa de topología](media/network-performance-monitor-performance-monitor/topology-dashboard.png) 

Puede consultar la pérdida, la latencia y el número de saltos de cada ruta de acceso en el panel **Acción**. Utilice la barra de desplazamiento para ver los detalles de las rutas de acceso incorrectas. Use los filtros para seleccionar las rutas de acceso con el salto incorrecto de modo que se pueda trazar la topología únicamente de las rutas de acceso seleccionadas. Para acercar o alejar el mapa de topología, utilice la rueda del mouse. 

En la siguiente imagen, la causa principal de las áreas problemáticas de la sección específica de la red aparecen en las rutas de acceso de red y los saltos. Seleccione un nodo en el mapa de topología t se mostrarán las propiedades del nodo, incluido el nombre de dominio completo y la dirección IP. Al seleccionar un salto, se muestra su dirección IP. 
 
![Mapa de topología con propiedades del nodo seleccionadas](media/network-performance-monitor-performance-monitor/topology-dashboard-root-cause.png) 

## <a name="next-steps"></a>Pasos siguientes
[Buscar registros](../log-query/log-query-overview.md) para ver datos detallados de rendimiento de red.

