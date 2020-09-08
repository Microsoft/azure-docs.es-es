---
title: Solución Network Performance Monitor en Azure | Microsoft Docs
description: Network Performance Monitor en Azure ayuda a supervisar el rendimiento de las redes, casi en tiempo real, para detectar y localizar cuellos de botella de rendimiento.
ms.subservice: logs
ms.topic: conceptual
author: vinynigam
ms.author: vinigam
ms.date: 02/20/2018
ms.openlocfilehash: 34e81076c27086ad838cca23de0e150a3c1b076c
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88798914"
---
# <a name="network-performance-monitor-solution-in-azure"></a>Solución Network Performance Monitor en Azure

![Símbolo de Network Performance Monitor](./media/network-performance-monitor/npm-symbol.png)


Network Performance Monitor es una solución de supervisión de red híbrida basada en la nube que le ayuda a supervisar el rendimiento de red entre distintos puntos en la infraestructura de red. También le permite supervisar la conectividad de red con los puntos de conexión del servicio y la aplicación, y supervisar el rendimiento de Azure ExpressRoute. 

Network Performance Monitor detecta posibles problemas de red, como enrutamiento del tráfico a "agujeros negros", errores de enrutamiento y problemas que los métodos de supervisión de red convencionales no pueden encontrar. La solución genera alertas y le notifica cómo y cuándo se supera un umbral en un vínculo de red. También garantiza la detección oportuna de los problemas de rendimiento de red y localiza el origen del problema en un segmento o dispositivo de red determinado. 

Network Performance Monitor ofrece tres amplias funcionalidades: 

* [Monitor de rendimiento](network-performance-monitor-performance-monitor.md): puede supervisar la conectividad de red en implementaciones en la nube y ubicaciones locales, varios centros de datos y sucursales, y aplicaciones o microservicios críticos de varios niveles. Con el Monitor de rendimiento, puede detectar problemas de red antes de recibir quejas de los usuarios.

* [Monitor de conectividad de servicio](network-performance-monitor-service-connectivity.md): puede supervisar la conectividad de los usuarios a los servicios que le interesen, determinar qué infraestructura se encuentra en la ruta de acceso e identificar dónde se producen los cuellos de botella en la red. Puede conocer las interrupciones antes que sus usuarios y ver el lugar exacto de los problemas a lo largo de la ruta de acceso de la red. 

    Esta funcionalidad le ayuda a realizar pruebas basadas en HTTP, HTTPS, TCP e ICMP con el fin de supervisar, casi en tiempo real o históricamente, la disponibilidad y el tiempo de respuesta del servicio. También puede supervisar la contribución de la red en latencia y pérdida de paquetes. Con un mapa de topología de red, puede aislar las ralentizaciones de la red. Puede identificar las zonas problemáticas a lo largo de la ruta de acceso de la red, desde el nodo al servicio, con datos de latencia en cada salto. Con las pruebas integradas, puede supervisar la conectividad de red a Office 365 y Dynamics CRM sin ninguna configuración previa. Con esta funcionalidad, puede supervisar la conectividad de red a cualquier punto de conexión compatible con TCP, como sitios web, aplicaciones SaaS, aplicaciones PaaS y bases de datos SQL.

* [Supervisión de ExpressRoute](network-performance-monitor-expressroute.md): supervise el rendimiento y la conectividad de un extremo a otro entre las sucursales y Azure a través de Azure ExpressRoute.  

Hay más información sobre las diferentes funcionalidades compatibles con [Network Performance Monitor](../../networking/network-monitoring-overview.md) disponible en línea.
 
## <a name="supported-regions"></a>Regiones admitidas
NPM puede supervisar la conectividad entre redes y aplicaciones en cualquier parte del mundo, desde un área de trabajo que se hospede en una de las siguientes regiones:
* Norte de Europa
* Oeste de Europa
* Norte de Suiza
* Centro de Francia
* Norte de Sudáfrica
* Centro de Canadá
* Oeste de EE. UU.
* Centro-Oeste de EE. UU.
* Centro-Norte de EE. UU
* Centro-sur de EE. UU.
* Centro de EE. UU.
* Este de EE. UU.
* Este de EE. UU. 2
* Oeste de EE. UU. 2
* Este de Japón
* Sudeste de Asia
* Sudeste de Australia
* Centro de Australia
* Este de Australia
* Sur de Reino Unido
* Este de Asia
* Centro de Corea del Sur
* Centro de la India
* US Gov Virginia
* US Gov Arizona
* Este de China 2


La lista de regiones admitidas para la Supervisión de ExpressRoute está disponible en la [documentación](../../expressroute/how-to-npm.md?utm_swu=8117).


## <a name="set-up-and-configure"></a>Instalación y configuración

> [!NOTE]
> Debe saber que el agente de Log Analytics también se denomina a veces Microsoft Monitoring Agent (MMA) o agente de OMS para Linux.

### <a name="install-and-configure-agents"></a>Instalación y configuración de agentes 

Use los procesos básicos para instalar agentes en [Conexión de equipos Windows a Azure Monitor](../platform/agent-windows.md) y [Conexión de Operations Manager con Azure Monitor](../platform/om-agents.md).

### <a name="where-to-install-the-agents"></a>Dónde instalar los agentes 

* **Monitor de rendimiento**: instale agentes de Log Analytics como mínimo en un nodo conectado a cada subred desde la que supervisará la conectividad de red a otras subredes.

    Para supervisar un vínculo de red, instale agentes en ambos puntos de conexión del vínculo en cuestión. Si no está seguro de la topología de la red, instale los agentes en servidores con cargas de trabajo críticas entre las que desee supervisar el rendimiento de red. Por ejemplo, si quiere supervisar la conexión de red entre un servidor web y un servidor que ejecuta SQL, instale un agente en ambos servidores. Los agentes supervisan la conectividad de red (vínculos) entre los hosts, no los hosts propiamente dichos. 

* **Monitor de conectividad de servicio**: instale un agente de Log Analytics en cada nodo desde el que supervisará la conectividad de red al punto de conexión de servicio. Un ejemplo es si desea supervisar la conectividad de red a Office 365 desde sus oficinas etiquetadas O1, O2 y O3. Instale el agente de Log Analytics como mínimo en un nodo en O1, O2 y O3. 

* **Supervisión de ExpressRoute**: instale como mínimo un agente de Log Analytics en la red virtual de Azure. Asimismo, instale al menos un agente en la subred local, que se conecta mediante el emparejamiento privado de ExpressRoute.  

### <a name="configure-log-analytics-agents-for-monitoring"></a>Configurar agentes de Log Analytics para la supervisión 

Network Performance Monitor usa transacciones sintéticas para supervisar el rendimiento de red entre los agentes de origen y de destino. Puede elegir entre los protocolos TCP e ICMP para la supervisión en las funcionalidades de Monitor de rendimiento y Monitor de conectividad de servicio. Solo TCP está disponible como el protocolo de supervisión para Supervisión de ExpressRoute. Asegúrese de que el firewall permita la comunicación entre los agentes de Log Analytics usados para supervisar en el protocolo que ha elegido. 

* **Protocolo TCP**: si ha elegido el protocolo TCP para la supervisión, abra el puerto de firewall en los agentes que se usan en Network Performance Monitor y Supervisión de ExpressRoute, para asegurarse de que estos pueden conectarse entre sí. Para abrir el puerto, ejecute el script de PowerShell [EnableRules.ps1](https://aka.ms/npmpowershellscript) sin parámetros en una ventana de PowerShell con privilegios administrativos.

    El script crea las claves del Registro que requiere la solución. También crea reglas de firewall de Windows para permitir que los agentes creen conexiones TCP entre sí. Las claves del Registro que crea el script especifican si se deben escribir los registros de depuración y la ruta del archivo de registro. Asimismo, el script define el puerto TCP del agente empleado para establecer la comunicación. El script establece automáticamente los valores de estas claves. No cambie manualmente estas claves. El puerto que se abre de forma predeterminada es 8084. Puede utilizar un puerto personalizado especificando el parámetro portNumber en el script. Utilice el mismo puerto en todos los equipos en los que se ejecute el script. 

    >[!NOTE]
    > El script configura solo el firewall de Windows localmente. Si tiene un firewall de red, asegúrese de que permite el tráfico destinado al puerto TCP que Network Performance Monitor esté utilizando.

    >[!NOTE]
    > No es necesario ejecutar el script de PowerShell [EnableRules.ps1](https://aka.ms/npmpowershellscript ) para el Monitor de conectividad de servicio.

    

* **Protocolo ICMP**: si ha elegido el protocolo ICMP para la supervisión, habilite las reglas de firewall siguientes para usar ICMP de forma confiable:
    
   ```
   netsh advfirewall firewall add rule name="NPMDICMPV4Echo" protocol="icmpv4:8,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV6Echo" protocol="icmpv6:128,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV4DestinationUnreachable" protocol="icmpv4:3,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV6DestinationUnreachable" protocol="icmpv6:1,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV4TimeExceeded" protocol="icmpv4:11,any" dir=in action=allow 
   netsh advfirewall firewall add rule name="NPMDICMPV6TimeExceeded" protocol="icmpv6:3,any" dir=in action=allow 
   ```
 

### <a name="configure-the-solution"></a>Configuración de la solución 

1. Agregue la solución Network Performance Monitor al área de trabajo desde [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.NetworkMonitoringOMS?tab=Overview). También puede usar el proceso que se describe en cómo [agregar soluciones de Azure Monitor desde la Galería de soluciones](./solutions.md). 
2. Abra el área de trabajo de Log Analytics y seleccione el icono **Overview** (Información general). 
3. Seleccione el icono denominado **Network Performance Monitor** con el mensaje *La solución necesita una configuración adicional*.

   ![Icono Monitor de rendimiento de red](media/network-performance-monitor/npm-config.png)

4. En la página **Configuración**, verá la opción para instalar agentes de Log Analytics y configurar los agentes para la supervisión en la vista **Configuración común**. Como se ha explicado anteriormente, si ha instalado y configurado los agentes de Log Analytics, seleccione la vista **Configuración** para configurar la función que quiere usar. 

   **Monitor de rendimiento**: elija el protocolo que se va a usar para las transacciones sintéticas en la regla del Monitor de rendimiento **Default** (Predeterminada) y seleccione **Save & Continue** (Guardar y continuar). Esta selección de protocolo contiene únicamente para la regla predeterminada generada por el sistema. Debe elegir el protocolo cada vez que cree explícitamente una regla de monitor de rendimiento. Siempre puede mover la configuración de reglas **Default** (Predeterminada) en la pestaña **Performance Monitor** (Monitor de rendimiento) (que aparece después de completar la configuración del día 0) y cambiar el protocolo más adelante. Si no quiere la funcionalidad Performance Monitor, puede deshabilitar la regla predeterminada en la configuración de reglas **Default** (Predeterminada) de la pestaña **Performance Monitor** (Monitor de rendimiento).

   ![Vista Supervisión del rendimiento](media/network-performance-monitor/npm-synthetic-transactions.png)
    
   **Monitor de conectividad de servicio**: esta funcionalidad proporciona pruebas preconfiguradas integradas para supervisar la conectividad de red a Office 365 y Dynamics 365 desde los agentes. Elija los servicios de Office 365 y Dynamics 365 que desea supervisar seleccionando las casillas junto a ellos. Para elegir los agentes desde los que desea realizar la supervisión, seleccione el botón **Add Agents** (Agregar agentes). Si no desea utilizar esta funcionalidad o si desea configurarla más adelante, no seleccione nada y haga clic en **Save & Continue** (Guardar y continuar).

   ![Vista del Monitor de conectividad de servicio](media/network-performance-monitor/npm-service-endpoint-monitor.png)

   **Supervisión de ExpressRoute**: Seleccione **Detectar ahora** para detectar todos los emparejamientos privados de ExpressRoute que están conectados a las redes virtuales en la suscripción de Azure vinculada a esta área de trabajo de Azure Log Analytics. 

   ![Vista Supervisión de ExpressRoute](media/network-performance-monitor/npm-express-route.png)

   Después de finalizar la detección, los emparejamientos y circuitos detectados se muestran en una tabla. 

   ![Página Configuración de Network Performance Monitor](media/network-performance-monitor/npm-private-peerings.png)
    
La supervisión de estos emparejamientos y circuitos está inicialmente en estado deshabilitado. Seleccione todos los recursos que desee supervisar y configure su supervisión en la vista de detalles de la derecha. Para guardar la configuración, seleccione **Guardar**. Para más información, consulte el artículo "Configure ExpressRoute monitoring" (Configuración de la supervisión de ExpressRoute). 

Una vez finalizada la configuración, se tarda entre 30 minutos y una hora en rellenar los datos. Mientras la solución agrega datos de la red, verá el mensaje *Solution requires additional configuration* (La solución necesita una configuración adicional) en el icono **Overview** (Información general) de Network Performance Monitor. Después de que los datos se recopilen e indexen, el icono **Overview** (Información general) cambia y le informa sobre el mantenimiento de la red en un resumen. Puede editar la supervisión de los nodos donde se han instalado los agentes de Log Analytics, así como las subredes detectadas desde el entorno.

#### <a name="edit-monitoring-settings-for-subnets-and-nodes"></a>Edición de la configuración de supervisión de subredes y nodos 

Todas las subredes que tienen instalado al menos un agente se muestran en la pestaña  **Subredes** de la página de configuración. 


Para habilitar o deshabilitar la supervisión de subredes determinadas, siga estos pasos:

1. Active o desactive la casilla junto a **Identificador de subred**. Después, asegúrese de que la opción **Usar para la supervisión** esté activada o desactivada, según corresponda. Puede seleccionar o borrar varias subredes. Cuando esta casilla está desactivada, no se supervisan las subredes y los agentes se actualizan para que dejen de hacer ping a los demás agentes. 
2. Elija los nodos que desea supervisar en una subred determinada. Seleccione la subred de la lista y mueva los nodos necesarios entre las listas que contienen nodos no supervisados y supervisados. Puede agregar una descripción personalizada a la subred.
3. Para guardar la configuración, seleccione **Guardar**. 

#### <a name="choose-nodes-to-monitor"></a>Elección de los nodos que desee supervisar

En la pestaña **Nodes** (Nodos) se muestran todos los nodos que tienen un agente instalado. 

1. Active o desactive los nodos que desee supervisar o dejar de supervisar. 
2. Seleccione **Use for Monitoring** (Usar para la supervisión) o desactívelo, según corresponda. 
3. Seleccione **Guardar**. 


Configure las funcionalidades que desee:

- [Supervisión del rendimiento](network-performance-monitor-performance-monitor.md#configuration)
- [Monitor de conectividad de servicio](network-performance-monitor-performance-monitor.md#configuration)
- [Supervisión de ExpressRoute](network-performance-monitor-expressroute.md#configuration)

 

## <a name="data-collection-details"></a>Detalles de la recopilación de datos
Para recopilar información de pérdida y latencia, Network Performance Monitor usa los paquetes de protocolo de enlace TCP SYN-SYNACK-ACK cuando se elige TCP como protocolo. Network Performance Monitor usa ICMP ECHO ICMP ECHO REPLY cuando se elige ICMP como protocolo. Traceroute también se usa para obtener información sobre la topología.

En la siguiente tabla se muestran los métodos de recopilación de datos y otros detalles sobre cómo se reúnen los datos para Monitor de rendimiento de red.

| Plataforma | Agente directo | Agente de System Center Operations Manager | Azure Storage | ¿Se requiere Operations Manager? | Se envían los datos del agente de Operations Manager a través del grupo de administración | Frecuencia de recopilación |
| --- | --- | --- | --- | --- | --- | --- |
| Windows | &#8226; | &#8226; |  |  |  |Protocolos de enlace TCP/mensajes ICMP ECHO cada 5 segundos, datos enviados cada 3 minutos |
 

 
La solución usa transacciones sintéticas para evaluar el estado de la red. Los agentes de Log Analytics instalados en diversos puntos de la red intercambian entre sí paquetes TCP o eco ICMP. El hecho de que los agentes utilicen paquetes TCP o ICMP Echo depende del protocolo seleccionado para la supervisión. En el proceso, los agentes conocen el tiempo de ida y vuelta y se enteran de la pérdida de paquetes, si la hay. Periódicamente, cada agente también realiza un seguimiento de la ruta hasta los demás agentes para descubrir las diversas rutas de la red que se deben comprobar. Con estos datos, los agentes pueden deducir la latencia de red y las cifras de pérdida de paquetes. Las pruebas se repiten cada cinco segundos. Los agentes agregan los datos durante aproximadamente tres minutos antes de que se carguen en el área de trabajo de Log Analytics en Azure Monitor.



>[!NOTE]
> A pesar de que los agentes se comunican entre sí con frecuencia, no generan una gran cantidad de tráfico de red mientras llevan a cabo las pruebas. Los agentes utilizan exclusivamente los paquetes de protocolo de enlace TCP SYN-SYNACK-ACK para determinar la pérdida y la latencia. No se intercambian paquetes de datos. Durante este proceso, los agentes se comunican entre sí solo si es necesario. La topología de comunicación del agente está optimizada para reducir el tráfico de red.

## <a name="use-the-solution"></a>Uso de la solución 

### <a name="network-performance-monitor-overview-tile"></a>Icono Información general de Network Performance Monitor 

Una vez que ha habilitado la solución Network Performance Monitor, el icono de la solución en la página **Overview** (Información general) ofrece una visión general rápida del mantenimiento de red. 

 ![Icono Información general de Network Performance Monitor](media/network-performance-monitor/npm-overview-tile.png)

### <a name="network-performance-monitor-dashboard"></a>Panel Monitor de rendimiento de red 

* **Eventos de mantenimiento de red principales**: esta página proporciona una lista de los eventos de mantenimiento y las alertas más recientes del sistema, así como el tiempo que llevan activos los eventos. Cada vez que el valor de la métrica seleccionada (pérdida, latencia, tiempo de respuesta o utilización de ancho de banda) para la regla de supervisión supera el umbral, se genera un evento de mantenimiento o una alerta. 

* **Supervisión de ExpressRoute**: esta página proporciona resúmenes de mantenimiento para las distintas conexiones de emparejamiento de ExpressRoute que supervisa la solución. El icono **Topology** (Topología) muestra el número de rutas de acceso de red mediante los circuitos de ExpressRoute que se supervisan en la red. Seleccione este icono para ir a la vista **Topology** (Topología).

* **Monitor de conectividad de servicio**: esta página proporciona resúmenes de mantenimiento de las distintas pruebas creadas. El icono **Topology** (Topología) indica al número de puntos de conexión que se supervisan. Seleccione este icono para ir a la vista **Topology** (Topología).

* **Monitor de rendimiento**: esta página proporciona resúmenes de mantenimiento para los vínculos **Network** (Red) y **Subnetwork** (Subred) que supervisa la solución. El icono **Topology** (Topología) indica al número de rutas de acceso de red que se supervisan en la red. Seleccione este icono para ir a la vista **Topology** (Topología). 

* **Consultas comunes**: esta página contiene un conjunto de consultas de búsqueda que capturan directamente los datos de supervisión de la red sin procesar. Puede usar estas consultas como punto de partida a fin de crear las suyas propias para generar informes personalizados. 

   ![Panel Monitor de rendimiento de red](media/network-performance-monitor/npm-dashboard.png)

 

### <a name="drill-down-for-depth"></a>Exploración en profundidad 

Puede seleccionar los diversos vínculos del panel de la solución para profundizar más en cualquier área de interés. Por ejemplo, cuando vea aparecer en el panel una alerta o un vínculo de red incorrecto, selecciónelo para investigar más. Una página enumera todos los vínculos de subred del vínculo de red concreto. Puede ver la pérdida, la latencia y el estado de mantenimiento de cada vínculo de subred. Puede averiguar rápidamente qué vínculo de subred provoca problemas. Seleccione **Ver vínculos de nodo** para ver todos los vínculos de nodo del vínculo de subred incorrecto. Además, puede ver los vínculos individuales de nodo a nodo y encontrar los vínculos de nodo incorrectos. 

Seleccione **Ver topología** para ver la topología de salto a salto de las rutas entre los nodos de origen y de destino. Las rutas con un mantenimiento incorrecto se muestran en rojo. Puede ver la latencia a la que ha contribuido cada salto y así identificar rápidamente el problema para una parte determinada de la red.

 

### <a name="network-state-recorder-control"></a>Control de la Grabadora de estado de la red

Cada vista muestra una instantánea del estado de su red en un momento determinado del tiempo. De forma predeterminada, se muestra el estado más reciente. La barra en la parte superior de la página muestra el punto en el tiempo para el que se muestra el estado. Para ver una instantánea del mantenimiento de red en un momento anterior, seleccione **Actions** (Acciones). También puede habilitar o deshabilitar la actualización automática de cualquier página mientras ve el estado más reciente. 

 ![Grabadora de estado de la red](media/network-performance-monitor/network-state-recorder.png)

 

### <a name="trend-charts"></a>Gráficos de tendencias 

En cada nivel que explore en profundidad, podrá ver la tendencia de la métrica aplicable. Puede ser pérdida, latencia, tiempo de respuesta o uso de ancho de banda. Para cambiar el intervalo de tiempo empleado para la tendencia, utilice el control del tiempo de la parte superior del gráfico. 

Los gráficos de tendencias muestran una perspectiva histórica del rendimiento de una métrica de rendimiento. Algunos problemas de red son transitorios por naturaleza y, por lo tanto, resultan difíciles de encontrar examinando únicamente el estado actual de la red. Los problemas pueden manifestarse rápidamente y desaparecer antes de que nadie se percate para volver a surgir más adelante. Estos problemas transitorios también pueden ser difíciles para los administradores de aplicaciones. Los problemas aparecen a menudo como aumentos inexplicables en el tiempo de respuesta de la aplicación, incluso aunque parezca que todos los componentes de la aplicación se ejecutan sin problemas. 

Puede detectar fácilmente estos tipos de problemas examinando un gráfico de tendencias. El problema aparece como un pico repentino de latencia o pérdida de paquetes en la red. Para investigar el problema, utilice el control de la Grabadora de estado de la red para ver la instantánea y la topología de red de ese punto en el tiempo en el que se produjo el problema.

 
![Gráficos de tendencias](media/network-performance-monitor/trend-charts.png)
 

### <a name="topology-map"></a>Mapa de topología 

Network Performance Monitor muestra la topología salto a salto de rutas entre el punto de conexión de origen y de destino en un mapa de topología interactivo. Para ver el mapa de topología, seleccione el icono **Topology** (Topología) en el panel de la solución. También puede seleccionar el vínculo **View topology** (Ver topología) en las páginas de exploración en profundidad. 

En el mapa de topología se muestra el número de rutas que existen entre el origen y el destino y cuáles son las rutas que toman los paquetes de datos. También se puede ver la latencia a la que contribuye cada salto de red. Todas las rutas de acceso en las que la latencia total está por encima del umbral (definido en la regla de supervisión correspondiente) se muestran en rojo. 

Cuando seleccione un nodo o pase el mouse por encima de él encima del mapa de topología, verá las propiedades del nodo, como el FQDN o la dirección IP. Seleccione un salto para ver su dirección IP. Puede identificar el salto de red problemático si observa la latencia a la que contribuye. Para filtrar rutas determinadas, utilice los filtros que aparecen en el panel de acciones contraíble. Para simplificar las topologías de red, oculte los saltos intermedios con el control deslizante del panel de acciones. Puede acercar o alejar el mapa de topología con la rueda del mouse. 

La topología mostrada en el mapa corresponde al nivel 3 y no contiene ningún dispositivo ni conexión del nivel 2. 

 
![Mapa de topología](media/network-performance-monitor/topology-map.png)
 

## <a name="log-queries-in-azure-monitor"></a>Consultas de registro en Azure Monitor

Todos los datos expuestos mediante gráficos en las páginas de exploración en profundidad y en el panel de Network Performance Monitor también se encuentran disponibles de forma nativa en las [consultas de registro](../log-query/log-query-overview.md). Puede realizar un análisis interactivo de los datos en el repositorio y correlacionar datos procedentes de distintos orígenes. También puede crear vistas y alertas personalizadas, y exportar los datos a Excel, Power BI o un vínculo que se pueda compartir. El área **Consultas comunes** del panel incorpora una serie de consultas útiles que puede usar como punto de partida para crear sus propios informes y consultas. 

## <a name="alerts"></a>Alertas

Network Performance Monitor usa las capacidades de alerta de [Azure Monitor](../platform/alerts-overview.md).

Esto quiere decir que todas las notificaciones se administran con [grupos de acciones](../platform/action-groups.md).  

Si es un usuario de NPM que crea una alerta mediante Log Analytics: 
1. Verá un vínculo que le redirigirá a Azure Portal. Haga clic en él para obtener acceso al portal.
2. Haga clic en el icono de la solución Network Performance Monitor. 
3. Navegue hasta Configurar.  
4. Seleccione la prueba para la que quiere crear una alerta y siga los pasos mencionados a continuación.

Si es un usuario NPM que va a crear una alerta a través de Azure Portal:  
1. Puede escribir directamente el correo electrónico o puede crear alertas a través de grupos de acciones.
2. Si decide escribir directamente el correo electrónico, se crea un grupo de acciones con el nombre **NPM Email ActionGroup** (Grupo de acciones de correo electrónico NPM) y se agrega el id. de correo electrónico a ese grupo de acciones.
3. Si decide usar grupos de acciones, tendrá que seleccionar un grupo de acciones creado anteriormente. Puede aprender a crear un grupo de acciones [aquí.](../platform/action-groups.md#create-an-action-group-by-using-the-azure-portal) 
4. Una vez que se ha creado correctamente la alerta, puede usar el vínculo Administrar alertas para administrar las alertas. 

Cada vez que cree una alerta, NPM creará una regla de alertas de registro basada en consultas en Azure Monitor. Esta consulta se desencadena de forma predeterminada cada cinco minutos. Azure Monitor no realiza ningún cargo por las primeras 250 reglas de alertas de registro creadas; pero las reglas de alertas por encima del límite de 250 reglas de alertas de registro se facturan según los [Precios de alertas en la página de precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).
Las notificaciones se cobran por separado según los [Precios de notificaciones en la página de precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).


## <a name="pricing"></a>Precios

La información sobre los precios está disponible [en línea](network-performance-monitor-pricing-faq.md).

## <a name="provide-feedback"></a>Envío de comentarios 

* **UserVoice:** puede publicar sus ideas sobre las características de Network Performance Monitor en las que le gustaría que trabajásemos. Visite la [página de UserVoice](https://feedback.azure.com/forums/267889-log-analytics/category/188146-network-monitoring). 

* **Únase a nuestra cohorte**. Siempre estamos interesados en que se unan nuevos clientes a nuestra cohorte. Como parte de todo ello, podrá obtener un acceso anticipado a las nuevas características y tendrá la oportunidad de ayudarnos a mejorar Network Performance Monitor. Si está interesado en unirse, rellene esta [encuesta rápida](https://aka.ms/npmcohort). 

## <a name="next-steps"></a>Pasos siguientes 
Más información sobre [Monitor de rendimiento](network-performance-monitor-performance-monitor.md), [Monitor de conectividad de servicio](network-performance-monitor-performance-monitor.md) y [Supervisión de ExpressRoute](network-performance-monitor-expressroute.md). 

