---
title: Cómo ver las dependencias de aplicación con Azure Monitor para VM (versión preliminar) | Microsoft Docs
description: La asignación es una característica de Azure Monitor para máquinas virtuales que detecta automáticamente los componentes de la aplicación en sistemas Windows y Linux y asigna la comunicación entre servicios. En este artículo se proporciona información sobre cómo usarla en una variedad de escenarios.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: tysonn
ms.assetid: ''
ms.service: azure-monitor
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/09/2019
ms.author: magoedte
ms.openlocfilehash: 792c2bd02b666cd656f1df368a7a60db44ccf8c4
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522172"
---
# <a name="using-azure-monitor-for-vms-preview-map-to-understand-application-components"></a>Uso de la asignación de Azure Monitor para VM (versión preliminar) para conocer los componentes de una aplicación
Visualización de los componentes de aplicaciones detectadas en Windows y Linux máquinas virtuales que se ejecutan en Azure de su entorno se puede observar en dos formas con Azure Monitor para las máquinas virtuales, desde una máquina virtual directamente o a través de grupos de máquinas virtuales de Azure Monitor. 

En este artículo le ayudamos a comprender la experiencia entre las dos perspectivas y a usar la característica de asignación. Para obtener más información sobre cómo configurar Azure Monitor para máquinas virtuales, consulte el artículo [Enable Azure Monitor for VMs](vminsights-enable-overview.md) (Habilitar Azure Monitor para máquinas virtuales).

## <a name="sign-in-to-azure"></a>Iniciar sesión en Azure
Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="introduction-to-map-experience"></a>Introducción a la experiencia de asignación
Antes de profundizar en cómo visualizar la asignación de una máquina virtual o de un grupo de VM, es importante que brindemos una breve introducción a la característica para entender cómo se presenta la información y qué representan las visualizaciones.  

Sin importar si selecciona la característica de asignación directamente desde una máquina virtual o desde Azure Monitor, se presentará una experiencia coherente.  La única diferencia es que, desde Azure Monitor, puede ver en una sola asignación a todos los miembros de un clúster o aplicación de varios niveles.

Las asignaciones representan las dependencias de las máquinas virtuales al detectar los procesos en ejecución con conexiones de red activas entre servidores, la latencia de conexión entrante y saliente y los puertos en cualquier arquitectura conectada a TCP a través de un intervalo de tiempo determinado.  Al expandir una máquina virtual se muestran los detalles del proceso y se muestran solo los procesos que se comunican con la máquina virtual. El recuento de clientes front-end que se conectan a la máquina virtual se indica con el grupo de clientes. El recuento de servidores back-end a los que se conecta la máquina virtual se indican en los grupos de puertos de servidor. Expanda un grupo de puertos de servidor para ver una lista detallada de los servidores conectados a ese puerto.  

Al hacer clic en la máquina virtual, el panel **Propiedades** se expande hacia la derecha para mostrar las propiedades del elemento seleccionado, como la información del sistema notificada por el sistema operativo, las propiedades de la máquina virtual de Azure y un gráfico de anillo que resume las conexiones detectadas. 

![Propiedades del sistema del equipo](./media/vminsights-maps/properties-pane-01.png)

En el lado derecho del panel, haga clic en el **eventos de registro** icono para cambiar el foco del panel para mostrar una lista de tablas que recopilan datos de la máquina virtual ha enviado a Azure Monitor y está disponible para las consultas.  Al hacer clic en cualquiera de los tipos de registro enumerados se abrirá la página **Registros** para ver los resultados de ese tipo con una consulta configurada previamente que se filtra para una máquina virtual específica.  

![Lista de búsqueda de registros en el panel Propiedades](./media/vminsights-maps/properties-pane-logs-01.png)

Cerrar **registros** y volver a la **propiedades** panel y seleccione **alertas** para ver las alertas que alerta se genera para la máquina virtual de los criterios de estado. La asignación se integra con Alertas de Azure para mostrar las alertas activadas del servidor seleccionado en el intervalo de tiempo seleccionado. El servidor muestra un icono si hay alertas actuales y el panel de alertas del equipo enumera las alertas. 

![Alertas de equipo en el panel Propiedades](./media/vminsights-maps/properties-pane-alerts-01.png)

Para que la característica de asignación muestre las alertas pertinentes, cree una regla de alerta que se active para un equipo determinado. Para crear alertas apropiadas, realice estos pasos:
- Incluya una cláusula para agrupar por equipo (por ejemplo, **by Computer interval 1 minute**).
- Elija que se envíen alertas según las unidades métricas.

Para obtener más información sobre Alertas de Azure y crear reglas de alertas, consulte el artículo [Alertas unificadas en Azure Monitor](../../azure-monitor/platform/alerts-overview.md)

El opción de **leyenda** en la esquina superior derecha describe los símbolos y los roles en una asignación.  Para observar más de cerca la asignación y explorarla, los controles de zoom en la parte inferior derecha de la página determinan el nivel de zoom y ajustan el tamaño de la página al tamaño de la página actual.  

## <a name="connection-metrics"></a>Métricas de conexión
En el panel **Conexiones** se muestra la métrica de conectividad estándar para la conexión que haya seleccionado de la máquina virtual a través del puerto TCP. Las métricas incluyen el tiempo de respuesta, las solicitudes por minuto, el rendimiento del tráfico y los vínculos.  

![Ejemplo del panel de gráficos de conectividad de red](./media/vminsights-maps/map-group-network-conn-pane-01.png)  

### <a name="failed-connections"></a>Conexiones con errores
En la asignación se muestran las conexiones con errores de procesos y equipos con una línea roja discontinua que indica que un sistema cliente no alcanza un proceso o puerto. Se informa de las conexiones erróneas de cualquier sistema con el agente de dependencia si ese sistema es el que intenta la conexión con errores. La asignación mide este proceso mediante la observación de los sockets TCP que no pueden establecer una conexión. Este error puede deberse a un firewall, una configuración incorrecta en el cliente o servidor o un servicio remoto que no está disponible.

![Ejemplo de error de conexión en la asignación](./media/vminsights-maps/map-group-failed-connection-01.png)

La comprensión de las conexiones con errores puede ayudar a solucionar problemas, a la validación de la migración, al análisis de seguridad y al entendimiento de la arquitectura general del servicio. A veces las conexiones con errores son inofensivas, pero a menudo señalan directamente un problema, como un entorno de conmutación por error que de repente se convierte en inalcanzable o dos niveles de aplicación que no pueden comunicarse entre sí después de una migración a la nube.

### <a name="client-groups"></a>Grupos de clientes
Los grupos de clientes en la asignación representan a los equipos clientes que tienen conexiones con el equipo asignado. Un único grupo de clientes representa a los clientes de un proceso o equipo individual.

![Ejemplo de grupos de cliente en la asignación](./media/vminsights-maps/map-group-client-groups-01.png)

Para ver las direcciones IP y los clientes supervisados de los sistemas en un grupo de clientes, seleccione el grupo. El contenido del grupo se enumera debajo del mismo.  

![Ejemplo de una lista de direcciones IP de grupo de clientes en la asignación](./media/vminsights-maps/map-group-client-group-iplist-01.png)

Si el grupo incluye a los clientes supervisados y sin supervisión, puede seleccionar la sección correspondiente del gráfico de anillos en el grupo para filtrar a los clientes.

### <a name="server-port-groups"></a>Grupos de puertos de servidor
Los grupos de puertos de servidor representan los puertos de servidor en servidores que tienen conexiones entrantes desde el equipo asignado. El grupo contiene el puerto del servidor y un recuento del número de servidores con conexiones a ese puerto. Seleccione el grupo para que se enumeren los servidores y las conexiones individuales. 

![Ejemplo de grupo de puertos de servidor en la asignación](./media/vminsights-maps/map-group-server-port-groups-01.png)  

Si el grupo incluye a los servidores supervisados y sin supervisión, puede seleccionar la sección correspondiente del gráfico de anillos en el grupo para filtrar a los servidores.

## <a name="view-map-directly-from-a-virtual-machine"></a>Visualización de la asignación directamente desde una máquina virtual 

Para obtener acceso a Azure Monitor para máquinas virtuales directamente desde una máquina virtual, realice lo siguiente.

1. En Azure Portal, seleccione **Virtual Machines**. 
2. En la lista, elija una máquina virtual y, en la sección **Monitor**, elija **Conclusiones (versión preliminar)**.  
3. Seleccione la pestaña **Asignación**.

La asignación muestra, durante un intervalo de tiempo determinado, las máquinas virtuales que están ejecutando procesos y grupos de procesos con conexiones de red activas.  De forma predeterminada, la asignación muestra los últimos 30 minutos.  Mediante el selector **Intervalo de tiempo** de la esquina superior izquierda, se pueden consultar intervalos de tiempo históricos de hasta una hora para mostrar el aspecto de las dependencias en el pasado (por ejemplo, durante un incidente o antes de un cambio).  

![Visión general directa de una asignación de máquina virtual](./media/vminsights-maps/map-direct-vm-01.png)

## <a name="view-map-directly-from-a-virtual-machine-scale-set"></a>Visualización del mapa de escalado de máquinas virtuales establece directamente

Para obtener acceso a Azure Monitor para las máquinas virtuales directamente desde un conjunto de escalado de máquinas virtuales, realice lo siguiente.

1. En el portal de Azure, seleccione **conjuntos de escalado de máquinas virtuales**.
2. En la lista, elija una máquina virtual y, en la sección **Monitor**, elija **Conclusiones (versión preliminar)**.  
3. Seleccione la pestaña **Asignación**.

Mapa muestra todas las instancias en el conjunto de escalado como un nodo de grupo junto con las dependencias del grupo. El nodo expandido enumera las instancias del conjunto de escalado, que puede desplazarse a través de diez a la vez. Para cargar un mapa para una instancia concreta, seleccione esa instancia en el mapa y, a continuación, haga clic en los puntos suspensivos para es adecuado y elija **cargar mapa del servidor**. Esto cargará el mapa para esa instancia, lo que permite ver los grupos de procesos y los procesos con conexiones de red activas durante un intervalo de tiempo especificado. De forma predeterminada, la asignación muestra los últimos 30 minutos. Mediante el **TimeRange** selector puede consultar los intervalos de tiempo históricos de hasta una hora para mostrar cómo se buscan dependencias en el pasado (por ejemplo, durante un incidente o antes de que se ha producido un cambio).  

![Visión general directa de una asignación de máquina virtual](./media/vminsights-maps/map-direct-vmss-01.png)

>[!NOTE]
>También puede acceder a un mapa para una instancia específica de la vista de instancias para el conjunto de escalado de máquina virtual. Vaya a **instancias** bajo el **configuración** sección y, a continuación, elija **Insights (versión preliminar)**.

## <a name="view-map-from-azure-monitor"></a>Visualización de la asignación desde Azure Monitor
Desde Azure Monitor, la característica de asignación proporciona una visión global de las máquinas virtuales y sus dependencias.  Para obtener acceso a la característica de asignación de Azure Monitor, realice lo siguiente. 

1. En Azure Portal, seleccione **Monitor**. 
2. Elija **Máquinas virtuales (versión preliminar)** en la sección **Conclusiones**.
3. Seleccione la pestaña **Asignación**.

![Visión general de una asignación de varias máquinas virtuales en Azure Monitor](./media/vminsights-maps/map-multivm-azure-monitor-01.png)

Desde el selector de **áreas de trabajo** en la parte superior de la página, si tiene más de un área de trabajo de Log Analytics, elija el área de trabajo que esté habilitada con la solución y tenga máquinas virtuales que dependan de ella. El **grupo** selector devolverá las suscripciones, grupos de recursos, [grupos de equipos](../../azure-monitor/platform/computer-groups.md)y conjuntos de escalado de máquinas virtuales de los equipos relacionados con el área de trabajo seleccionada. La selección solo se aplica a la característica de asignación y no se aplica a las secciones de rendimiento o asignación.

De forma predeterminada, la asignación muestra los últimos 30 minutos. Mediante el selector **Intervalo de tiempo**, se pueden consultar intervalos de tiempo históricos de hasta una hora para mostrar el aspecto de las dependencias en el pasado (por ejemplo, durante un incidente o antes de un cambio).   

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre cómo usar la característica de mantenimiento, consulte el artículo sobre cómo [ver el estado de una máquina virtual de Azure](vminsights-health.md) o bien, para identificar los cuellos de botella y el uso general con el rendimiento de las máquinas virtuales, consulte cómo [ver el rendimiento de Azure Monitor para máquinas virtuales](vminsights-performance.md). 
