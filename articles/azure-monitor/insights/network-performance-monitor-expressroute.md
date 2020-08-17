---
title: Solución Monitor de rendimiento de red en Azure Log Analytics | Microsoft Docs
description: Use la funcionalidad de Supervisión de ExpressRoute en Network Performance Monitor para supervisar la conectividad de un extremo a otro y el rendimiento entre sus sucursales y Azure, a través de Azure ExpressRoute.
ms.subservice: logs
ms.topic: conceptual
author: abshamsft
ms.author: absha
ms.date: 11/27/2018
ms.openlocfilehash: a9f268e5080a5c04c5cdb4767f2db5fd2d77cd3c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87326161"
---
# <a name="expressroute-monitor"></a>Supervisión de ExpressRoute

Puede usar la funcionalidad de Supervisión de Azure ExpressRoute en [Network Performance Monitor](network-performance-monitor.md) para supervisar la conectividad de un extremo a otro y el rendimiento entre sus sucursales y Azure, a través de Azure ExpressRoute. Las principales ventajas son: 

- Detección automática de circuitos ExpressRoute asociados a una suscripción.
- Seguimiento de uso, pérdida y latencia de ancho de banda en el nivel de circuito, emparejamiento y Azure Virtual Network para ExpressRoute.
- Detección de topología de red de los circuitos ExpressRoute.

![Supervisión de ExpressRoute](media/network-performance-monitor-expressroute/expressroute-intro.png)

## <a name="configuration"></a>Configuración 
Para abrir la configuración de Network Performance Monitor, abra la [solución de Network Performance Monitor](network-performance-monitor.md) y seleccione **Configurar**.

### <a name="configure-network-security-group-rules"></a>Configuración de las reglas del grupo de seguridad de red 
En el caso de los servidores de Azure que se utilizan para la supervisión a través de Network Performance Monitor, configure las reglas del grupo de seguridad de red (NSG) para permitir el tráfico TCP en un puerto utilizado por Network Performance Monitor para las transacciones sintéticas. El puerto predeterminado es 8084. Esta configuración permite al agente de Log Analytics instalado en máquinas virtuales de Azure comunicarse con un agente de supervisión local. 

Para más información sobre NSG, consulte  [Grupos de seguridad de red](../../virtual-network/manage-network-security-group.md). 

>[!NOTE]
> Antes de continuar con este paso, instale el agente del servidor local y el agente del servidor de Azure, y ejecute el script de PowerShell EnableRules.ps1. 

 
### <a name="discover-expressroute-peering-connections"></a>Detección de conexiones de emparejamiento de ExpressRoute 
 
1. Seleccione la vista **Emparejamientos de ExpressRoute**.
2. Seleccione **Detectar ahora** para detectar todos los emparejamientos privados de ExpressRoute que están conectados a las redes virtuales en la suscripción de Azure vinculada a esta área de trabajo de Azure Log Analytics.

    >[!NOTE]
    > Actualmente, la solución detecta solo emparejamientos privados de ExpressRoute. 

    >[!NOTE]
    > Solo se detectan los emparejamientos privados conectados a las redes virtuales asociadas a la suscripción vinculada con esta área de trabajo de Log Analytics. Si la instancia de ExpressRoute está conectada a redes virtuales situadas fuera de la suscripción vinculada a esta área de trabajo, cree un área de trabajo de Log Analytics en esas suscripciones. A continuación, use Network Performance Monitor para supervisar esos emparejamientos. 

    ![Configuración de la Supervisión de ExpressRoute](media/network-performance-monitor-expressroute/expressroute-configure.png)
 
   Una vez completada la detección, las conexiones de los emparejamientos privados detectados se muestran en una tabla. La supervisión en estos emparejamientos está inicialmente en estado deshabilitado. 

### <a name="enable-monitoring-of-the-expressroute-peering-connections"></a>Habilitación de la supervisión de las conexiones de emparejamiento de ExpressRoute 

1. Seleccione la conexión de emparejamiento privado que desea supervisar.
2. En el panel de la derecha, active la casilla **Supervisar este emparejamiento**. 
3. Si piensa crear eventos de estado para esta conexión, active **Habilitar el seguimiento de estado para este emparejamiento**. 
4. Elija las condiciones de supervisión. Puede establecer umbrales personalizados para la generación de eventos de estado escribiendo valores de umbral. Siempre que el valor de la condición supere el umbral seleccionado para la conexión del emparejamiento, se generará un evento de estado. 
5. Seleccione **Agregar agentes** para elegir los agentes de supervisión que desea utilizar para la supervisión de esta conexión de emparejamiento. Asegúrese de agregar agentes en ambos extremos de la conexión. Necesita al menos un agente en la red virtual conectada a este emparejamiento. También necesita al menos un agente local conectado a este emparejamiento. 
6. Para guardar la configuración, seleccione **Guardar**. 

   ![Configuración de la Supervisión de ExpressRoute](media/network-performance-monitor-expressroute/expressroute-configure-discovery.png)


Después de habilitar las reglas y seleccionar valores y agentes, espere entre 30 y 60 minutos a que se rellenen los valores y a que aparezcan los iconos **Supervisión de ExpressRoute**. Si se fija en los iconos de supervisión, Network Performance Monitor supervisa ahora los circuitos ExpressRoute y los recursos de conexión. 

>[!NOTE]
> Esta función funciona de manera confiable en áreas de trabajo que se han actualizado para el nuevo lenguaje de consulta.

## <a name="walkthrough"></a>Tutorial 

El panel de Network Performance Monitor muestra un resumen del estado de los circuitos ExpressRoute y las conexiones de emparejamiento. 

![Panel Monitor de rendimiento de red](media/network-performance-monitor-expressroute/npm-dashboard-expressroute.png) 

### <a name="circuits-list"></a>Lista de circuitos 

Para ver una lista de todos los circuitos ExpressRoute supervisados, seleccione el icono Circuitos ExpressRoute. Puede seleccionar un circuito y ver su estado de mantenimiento, gráficos de tendencias de pérdida de paquetes, uso de ancho de banda y latencia. Los gráficos son interactivos. Puede seleccionar un período de tiempo personalizado para trazar los gráficos. Arrastre el mouse sobre un área del gráfico para acercar y ver los puntos de datos con precisión. 

![Lista de circuitos ExpressRoute](media/network-performance-monitor-expressroute/expressroute-circuits.png) 

### <a name="trends-of-loss-latency-and-throughput"></a>Tendencias de rendimiento, latencia y pérdida 

Los gráficos de pérdida, la latencia y el uso de ancho de banda son interactivos. Puede acercar cualquier sección de estos gráficos con los controles de mouse. También puede ver los datos de pérdida, la latencia y el ancho de banda para otros intervalos. En la parte superior izquierda, bajo el botón **Acciones**, seleccione  **Fecha y hora**. 

![Latencia de ExpressRoute](media/network-performance-monitor-expressroute/expressroute-latency.png) 

### <a name="peerings-list"></a>Lista de emparejamientos 

Para mostrar una lista de todas las conexiones a redes virtuales a través de emparejamientos privados, seleccione el icono **Emparejamientos privados** en el panel. Aquí, puede seleccionar una conexión de red virtual y ver su estado de mantenimiento, gráficos de tendencias de pérdida de paquetes, uso de ancho de banda y latencia. 

![Emparejamientos de ExpressRoute](media/network-performance-monitor-expressroute/expressroute-peerings.png) 

### <a name="circuit-topology"></a>Topología de circuito 

Para ver la topología de circuito, seleccione el icono **Topología**. Esta acción le llevará a la vista de topología del circuito o emparejamiento seleccionado. El diagrama de topología proporciona la latencia para cada segmento de la red y cada salto de tres capas se representa mediante un nodo del diagrama. Si se selecciona un salto, se mostrarán más detalles sobre él. Para aumentar el nivel de visibilidad para incluir saltos locales, mueva el control deslizante debajo de **FILTROS**. Si se mueve el control deslizante hacia la izquierda o derecha, se aumenta o disminuye el número de saltos en el gráfico de topología. La latencia por cada segmento está visible, lo que permite un aislamiento más rápido de los segmentos de latencia elevada en la red.

![Topología de ExpressRoute](media/network-performance-monitor-expressroute/expressroute-topology.png)

### <a name="detailed-topology-view-of-a-circuit"></a>Vista de topología detallada de un circuito 

En esta vista se muestran conexiones de red virtual. 

![Conexiones de red virtual de ExpressRoute](media/network-performance-monitor-expressroute/expressroute-vnet.png)
 
## <a name="diagnostics"></a>Diagnóstico 

Network Performance Monitor le ayuda a diagnosticar diferentes problemas de conectividad en los circuitos. A continuación se enumeran algunos de los problemas que se pueden ver.

Puede ver los códigos de notificación y establecer alertas para ellos a través de **LogAnalytics**. En la página de **diagnósticos de NPM** puede ver las descripciones de cada mensaje de diagnóstico que se desencadena.

| Código de notificación (registros) | Descripción |
| --- | --- |
| 5501 | No se puede atravesar una conexión secundaria del circuito ExpressRoute. |
| 5502 | No se puede atravesar una conexión primaria del circuito ExpressRoute. |
| 5503 | No se encuentra ningún circuito para la suscripción vinculada al área de trabajo. | 
| 5508 | No se puede determinar si el tráfico pasa a través de un circuito para la ruta de acceso. |
| 5510 | El tráfico no pasa a través del circuito previsto. | 
| 5511 | El tráfico no pasa a través de la red virtual prevista. | 

**El circuito está inactivo.** Network Performance Monitor le avisa cuando se pierde la conectividad entre los recursos locales y las redes virtuales de Azure. Esta notificación le ayuda a adoptar medidas preventivas antes de recibir las escalaciones de usuario y reducir el tiempo de inactividad.

![El circuito ExpressRoute está inactivo](media/network-performance-monitor-expressroute/expressroute-circuit-down.png)
 

**El tráfico no fluye a través del circuito previsto.** Network Performance Monitor le avisa cada vez que no fluye el tráfico a través del circuito ExpressRoute previsto. Este problema puede producirse si el circuito está inactivo y el tráfico fluye a través de la ruta de la copia de seguridad. También puede producirse si hay un problema de enrutamiento. Esta información le ayuda a administrar de manera proactiva los problemas de configuración en las directivas de enrutamiento y a asegurarse de que se utiliza la ruta más óptima y segura. 

 

**El tráfico no fluye a través del circuito principal.** Network Performance Monitor le avisa cuando el tráfico fluye a través del circuito ExpressRoute secundario. Aunque en este caso no experimentará problemas de conectividad, la solución proactiva de los problemas del circuito le ayudan a estar mejor preparado. 

 
![Flujo de tráfico de ExpressRoute](media/network-performance-monitor-expressroute/expressroute-traffic-flow.png)


**Degradación debido a uso intenso.** Puede poner en correlación la tendencia de utilización del ancho de banda con la tendencia de latencia para identificar si la degradación de la carga de trabajo de Azure se debe a un pico en la utilización del ancho de banda o no. A continuación, puede tomar medidas en consecuencia.

![Uso de ancho de banda de ExpressRoute](media/network-performance-monitor-expressroute/expressroute-peak-utilization.png)

 

## <a name="next-steps"></a>Pasos siguientes
[Buscar registros](../log-query/log-query-overview.md) para ver datos detallados de rendimiento de red.

