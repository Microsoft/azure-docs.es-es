---
title: Visualización de las dependencias de aplicación con Azure Monitor para VM
description: Asignación es una característica de Azure Monitor para VM. Detecta automáticamente los componentes de la aplicación en sistemas Windows y Linux y asigna la comunicación entre servicios. En este artículo se proporciona información sobre cómo usar la característica de asignación en diferentes escenarios.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 03/20/2020
ms.openlocfilehash: 3227585b62583d0a05f8dbc0444ee5bb4a27df77
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320517"
---
# <a name="use-the-map-feature-of-azure-monitor-for-vms-to-understand-application-components"></a>Uso de la característica de asignación de Azure Monitor para VM para conocer los componentes de una aplicación
En Azure Monitor para VM, puede ver los componentes de la aplicación detectados en las máquinas virtuales (VM) de Windows y Linux que se ejecutan en Azure o en su entorno. Puede observar las VM de dos maneras. Ver una asignación directamente desde una VM o ver un asignación desde Azure Monitor para ver los componentes de los grupos de VM. En este artículo le ayudamos a comprender estos dos métodos de visualización y a usar la característica de asignación. 

Para obtener más información sobre cómo configurar Azure Monitor para máquinas virtuales, consulte el artículo [Enable Azure Monitor for VMs](vminsights-enable-overview.md) (Habilitar Azure Monitor para máquinas virtuales).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure
Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="introduction-to-the-map-experience"></a>Introducción a la experiencia de asignación
Antes de profundizar en la experiencia de asignación, debe entender cómo se presenta y se visualiza la información. Sin importar si selecciona la característica de asignación directamente desde una máquina virtual o desde Azure Monitor, se presentará una experiencia coherente. La única diferencia es que, desde Azure Monitor, una sola asignación muestra todos los miembros de un clúster o aplicación de varios niveles.

La característica de asignación visualiza las dependencias de VM al detectar los procesos en ejecución que tienen: 

- Conexiones de red activas entre servidores.
- Latencia de conexión entrante y saliente.
- Puertos en cualquier arquitectura con conexión TCP a través de un intervalo de tiempo especificado.  
 
Al expandir una VM se muestran los detalles del proceso y se muestran solo los procesos que se comunican con la máquina virtual. El grupo de clientes muestra el número de clientes front-end que se conectan a la VM. Los grupos del puerto de servidor muestran el número de servidores back-end a los que se conecta la VM. Expanda un grupo de puertos de servidor para ver una lista detallada de los servidores que se conectan a través de ese puerto.  

Cuando se selecciona la VM, el panel **Propiedades** de la derecha muestra las propiedades de la VM. Las propiedades incluyen información del sistema que proporciona el sistema operativo, propiedades de la VM de Azure y un gráfico de anillos que resume las conexiones detectadas. 

![Panel Propiedades](./media/vminsights-maps/properties-pane-01.png)

En el lado derecho del panel, seleccione **Eventos registro** para mostrar una lista de los datos que la VM ha enviado a Azure Monitor. Estos datos están disponibles para su consulta.  Seleccione cualquier tipo de registro para abrir la página **Registros**, donde verá los resultados para ese tipo de registro. También verá una consulta preconfigurada que se filtra en la VM.  

![Panel Eventos de registro](./media/vminsights-maps/properties-pane-logs-01.png)

Cierre la página **Registros** y vuelva al panel **Propiedades**. Una vez allí, seleccione **Alertas** para ver las alertas de los criterios de estado de la VM. La característica de asignación se integra con Alertas de Azure para mostrar las alertas del servidor seleccionado en el intervalo de tiempo seleccionado. El servidor muestra un icono para las alertas actuales y el panel **Alertas del equipo** enumera las alertas. 

![Panel Alertas](./media/vminsights-maps/properties-pane-alerts-01.png)

Para hacer que la característica de asignación muestre las alertas pertinentes, cree una regla de alerta que se aplique a un equipo determinado:

- Incluya una cláusula para agrupar alertas por equipo (por ejemplo, **by Computer interval 1 minute**).
- Basar la alerta en una métrica.

Para obtener más información sobre Alertas de Azure y crear reglas de alertas, consulte el artículo [Alertas unificadas en Azure Monitor](../platform/alerts-overview.md).

En la esquina superior derecha, la opción **Leyenda** describe los símbolos y roles en la asignación. Para obtener una visión más detallada en el mapa y moverlo, use los controles de zoom de la esquina inferior derecha. Puede establecer el nivel de zoom y ajustar el mapa al tamaño de la página.  

## <a name="connection-metrics"></a>Métricas de conexión
En el panel **Conexiones** se muestra la métrica estándar para la conexión que haya seleccionado de la máquina virtual a través del puerto TCP. Las métricas incluyen el tiempo de respuesta, las solicitudes por minuto, el rendimiento del tráfico y los vínculos.  

![Gráficos de Conectividad de red en el panel Conexiones](./media/vminsights-maps/map-group-network-conn-pane-01.png)  

### <a name="failed-connections"></a>Conexiones con errores
El mapa muestra las conexiones erróneas para procesos y equipos. Una línea discontinua roja indica que un sistema cliente no puede conectarse con un proceso o puerto. Para sistemas que usan Dependency Agent, el agente informa de los intentos de conexión erróneos. La característica de asignación supervisa un proceso mediante la observación de los sockets TCP que no pueden establecer una conexión. Este error puede deberse a un firewall, a una configuración incorrecta en el cliente o servidor o a un servicio remoto que no está disponible.

![Conexión errónea en la asignación](./media/vminsights-maps/map-group-failed-connection-01.png)

La comprensión de las conexiones con errores puede ayudar a solucionar problemas, validar la migración, analizar la seguridad y comprender la arquitectura general del servicio. Las conexiones con errores a veces son inofensivas, pero a menudo indican un problema. Las conexiones pueden producir un error, por ejemplo, cuando un entorno de conmutación por error de repente se convierte en inalcanzable o dos capas de aplicaciones no pueden comunicarse entre sí después de una migración en la nube.

### <a name="client-groups"></a>Grupos de clientes
En la asignación, los grupos de clientes representan a los equipos clientes que tienen conexiones con el equipo asignado. Un único grupo de clientes representa a los clientes de un proceso o equipo individual.

![Grupo de clientes en la asignación](./media/vminsights-maps/map-group-client-groups-01.png)

Para ver las direcciones IP y los clientes supervisados de los sistemas en un grupo de clientes, seleccione el grupo. El contenido del grupo se muestra a continuación.  

![Lista de direcciones IP de un grupo de clientes en la asignación](./media/vminsights-maps/map-group-client-group-iplist-01.png)

Si el grupo incluye a los clientes supervisados y sin supervisión, puede seleccionar la sección correspondiente del gráfico de anillos del grupo para filtrar a los clientes.

### <a name="server-port-groups"></a>Grupos del puerto de servidor
Los grupos del puerto de servidor representan los puertos en servidores que tienen conexiones entrantes desde el equipo asignado. El grupo contiene el puerto del servidor y un recuento del número de servidores con conexiones a ese puerto. Seleccione el grupo para ver los servidores y las conexiones individuales. 

![Grupo del puerto de servidor en la asignación](./media/vminsights-maps/map-group-server-port-groups-01.png)  

Si el grupo incluye a los servidores supervisados y sin supervisión, puede seleccionar la sección correspondiente del gráfico de anillos del grupo para filtrar a los servidores.

## <a name="view-a-map-from-a-vm"></a>Ver un mapa de una VM 

Para acceder a Azure Monitor para VM directamente desde una VM:

1. En Azure Portal, seleccione **Virtual Machines**. 
2. En la lista, elija una VM. En la sección **Supervisión**, seleccione **Insights**.  
3. Seleccione la pestaña **Asignación**.

La asignación muestra las dependencias de la VM mediante la detección de procesos y grupos de procesos que se están ejecutando en conexiones de red activas durante un determinado intervalo de tiempo.  

De forma predeterminada, la asignación muestra los últimos 30 minutos. Si quiere ver el aspecto de las dependencias en el pasado, puede consultar los intervalos de tiempo históricos de hasta hace una hora. Para ejecutar la consulta, use el selector **TimeRange** en la esquina superior izquierda. Puede ejecutar una consulta, por ejemplo, durante un incidente o para ver el estado antes de un cambio.  

![Visión general directa de una asignación de máquina virtual](./media/vminsights-maps/map-direct-vm-01.png)

## <a name="view-a-map-from-a-virtual-machine-scale-set"></a>Visualización de una asignación desde un conjunto de escalado de máquinas virtuales

Para obtener acceso a Azure Monitor para VM directamente desde un conjunto de escalado de máquinas virtuales:

1. En Azure Portal, seleccione **Conjuntos de escalado de máquinas virtuales**.
2. En la lista, elija una VM. Luego, en la sección **Supervisión**, seleccione **Insights**.  
3. Seleccione la pestaña **Asignación**.

La asignación muestra todas las instancias del conjunto de escalado como un nodo de grupo junto con las dependencias del grupo. El nodo expandido enumera las instancias del conjunto de escalado. Puede desplazarse a través de estas instancias (10 a la vez). 

Para cargar un asignación para una determinada instancia, seleccione primero esa instancia en la asignación. A continuación, seleccione el botón **puntos suspensivos** (…) a la derecha y elija **Cargar mapa del servidor**. En la asignación que aparece, puede ver los procesos y grupos de procesos con conexiones de red activas durante un intervalo de tiempo especificado. 

De forma predeterminada, la asignación muestra los últimos 30 minutos. Si quiere ver el aspecto de las dependencias en el pasado, puede consultar los intervalos de tiempo históricos de hasta hace una hora. Para ejecutar la consulta, use el selector **TimeRange**. Puede ejecutar una consulta, por ejemplo, durante un incidente o para ver el estado antes de un cambio.

![Visión general directa de una asignación de máquina virtual](./media/vminsights-maps/map-direct-vmss-01.png)

>[!NOTE]
>También puede acceder a una asignación de una instancia específica desde la vista **Instancias** de su conjunto de escalado de máquinas virtuales. En la sección **Configuración**, vaya a **Instancias** > **Insights**.

## <a name="view-a-map-from-azure-monitor"></a>Visualización de una asignación desde Azure Monitor

Desde Azure Monitor, la característica de asignación proporciona una visión global de las VM y sus dependencias. Para obtener acceso a la característica de asignación en Azure Monitor:

1. En Azure Portal, seleccione **Monitor**. 
2. En la sección **Insights**, elija **Máquinas virtuales**.
3. Seleccione la pestaña **Asignación**.

   ![Mapa de información general de Azure Monitor de varias VM](./media/vminsights-maps/map-multivm-azure-monitor-01.png)

Elija un área de trabajo con el selector **Área de trabajo** en la parte superior de la página. Si tiene más de un área de trabajo de Log Analytics, elija el área de trabajo que está habilitado con la solución y que tiene VM que dependen de él. 

El selector de **grupos** devuelve las suscripciones, los grupos de recursos, los [grupos de equipos](../platform/computer-groups.md) y los conjuntos de escalado de máquinas virtuales de los equipos relacionados con el área de trabajo seleccionada. La selección solo se aplica a la característica de asignación y no se aplica a las secciones de rendimiento o estado.

De forma predeterminada, la asignación muestra los últimos 30 minutos. Si quiere ver el aspecto de las dependencias en el pasado, puede consultar los intervalos de tiempo históricos de hasta hace una hora. Para ejecutar la consulta, use el selector **TimeRange**. Puede ejecutar una consulta, por ejemplo, durante un incidente o para ver el estado antes de un cambio.  

## <a name="next-steps"></a>Pasos siguientes

Para identificar los cuellos de botella, comprobar el rendimiento y comprender el uso general de las VM, consulte [View performance status for Azure Monitor for VMs](vminsights-performance.md) (Ver el estado del rendimiento de Azure Monitor para VM). 

