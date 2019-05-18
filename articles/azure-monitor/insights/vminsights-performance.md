---
title: Cómo representar el rendimiento en gráficos con Azure Monitor para VM (versión preliminar) | Microsoft Docs
description: El rendimiento es una característica de Azure Monitor para máquinas virtuales que detecta automáticamente los componentes de la aplicación en sistemas Windows y Linux y asigna la comunicación entre servicios. En este artículo se proporciona información sobre cómo usarla en una variedad de escenarios.
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
ms.date: 05/07/2019
ms.author: magoedte
ms.openlocfilehash: c83a862a37dbf28c6933877bf4a0aecc4364e6c5
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522089"
---
# <a name="how-to-chart-performance-with-azure-monitor-for-vms-preview"></a>Cómo representar el rendimiento en gráficos con Azure Monitor para VM (versión preliminar)
Azure Monitor para VM incluye un conjunto de gráficos de rendimiento que tienen como destino varios indicadores clave de rendimiento (KPI) para ayudarle a determinar el rendimiento de una máquina virtual. Los gráficos muestran el uso de los recursos durante un período de tiempo para que pueda identificar cuellos de botella, anomalías, o cambiar a una perspectiva en la que se muestre cada máquina a fin de ver el uso de los recursos en función de la métrica seleccionada. Aunque hay varios elementos a tener en cuenta cuando se trata del rendimiento, Azure Monitor para VM supervisa los indicadores de rendimiento clave del sistema operativo relacionados con la utilización del procesador, la memoria, el adaptador de red y los discos. El rendimiento complementa la característica de supervisión de mantenimiento y ayuda a exponer los problemas que indican un posible error en el componente del sistema. Además, es compatible con la optimización y los ajustes para lograr la eficiencia, así como con el planeamiento de capacidad.  

## <a name="multi-vm-perspective-from-azure-monitor"></a>Perspectiva de varias VM de Azure Monitor
Azure monitor, la característica de rendimiento proporciona una vista de todas las máquinas virtuales supervisadas implementado a través de grupos de trabajo en las suscripciones o en su entorno. Para obtener acceso desde Azure Monitor, siga estos pasos. 

1. En Azure Portal, seleccione **Monitor**. 
2. Elija **Máquinas virtuales (versión preliminar)** en la sección **Conclusiones**.
3. Seleccione la pestaña **Rendimiento**.

![Vista de la lista de N principales de rendimiento de conclusiones de la VM](./media/vminsights-performance/vminsights-performance-aggview-01.png)

En la pestaña **Top N Charts** (Gráficos de N principales), si tiene más de un área de trabajo de Log Analytics, elija la que esté habilitada con la solución en el selector **Área de trabajo** de la parte superior de la página. El selector de **grupos** devolverá las suscripciones, los grupos de recursos, los [grupos de equipos](../platform/computer-groups.md) y los conjuntos de escalado de máquinas virtuales de los equipos relacionados con el área de trabajo seleccionada que puede usar para filtrar aún más los resultados que aparecen en las tablas de esta y otras páginas. Su selección solo se aplica a la característica de rendimiento y no se aplica a las secciones de salud o mapa.  

De forma predeterminada, los gráficos muestran las últimas 24 horas. Mediante el selector **Intervalo de tiempo**, puede consultar los intervalos de tiempo históricos de hasta 30 días para mostrar cómo como era el rendimiento en el pasado.   

Los cinco gráficos de uso de la capacidad que se muestran en la página son:

* CPU Utilization % (Porcentaje de uso de la CPU): muestra las primeras cinco máquinas con el mayor uso medio del procesador. 
* Memoria disponible: muestra las primeras cinco máquinas con la menor cantidad media de memoria disponible. 
* Logical Disk Space Used % (Porcentaje de espacio usado del disco lógico): muestra las cinco primeras máquinas con el mayor porcentaje medio de espacio en disco que se usa en todos los volúmenes del disco. 
* Bytes Sent Rate (Tasa de bytes enviados): muestra las cinco primeras máquinas con el mayor número medio de bytes enviados. 
* Bytes Receive Rate (Tasa de bytes recibidos): muestra las cinco primeras máquinas con el mayor número medio de bytes recibidos. 

Al hacer clic en el icono de anclaje en la esquina superior derecha de uno de los cinco gráficos ancle el gráfico seleccionado en el último panel Azure que vio por última vez.  En el panel, puede cambiar el tamaño y la posición del gráfico. Seleccionando el gráfico desde el panel se redirigirá a Azure Monitor para las máquinas virtuales y cargar el ámbito correcto y la vista.  

Al hacer clic en el icono situado a la izquierda del icono de pin en cualquiera de los cinco gráficos se abre el **Top N lista** vista.  Aquí puede ver el uso de los recursos para esa métrica de rendimiento por cada VM en una vista de lista y la máquina con mayor tendencia.  

![Vista de lista de N principales para una métrica de rendimiento seleccionada](./media/vminsights-performance/vminsights-performance-topnlist-01.png)

Al hacer clic en la máquina virtual, el panel **Propiedades** se expande hacia la derecha para mostrar las propiedades del elemento seleccionado, como la información del sistema notificada por el sistema operativo, las propiedades de la VM de Azure, etc. Al hacer clic en una de las opciones de la sección **Vínculos rápidos**, se le redirigirá a esa característica directamente desde la VM seleccionada.  

![Panel Propiedades de la máquina virtual](./media/vminsights-performance/vminsights-properties-pane-01.png)

Cambie a la pestaña **Aggregated Charts** (Gráficos agregados) para ver las métricas de rendimiento filtradas por las medidas de los percentiles y las medias.  

![Vista agregada del rendimiento de las conclusiones de la VM](./media/vminsights-performance/vminsights-performance-aggview-02.png)

Se proporcionan los siguientes gráficos de uso de la capacidad:

* CPU Utilization % (Porcentaje de uso de la CPU): el valor predeterminado muestra la media y el percentil 95 superior. 
* Memoria disponible: los valores predeterminados que muestra el percentil superior, promedio de 5 y 10 
* Logical Disk Space Used % (Porcentaje de espacio usado del disco lógico): el valor predeterminado muestra la media y el percentil 95. 
* Bytes Sent Rate (Tasa de bytes enviados): el valor predeterminado muestra la media de bytes enviados. 
* Bytes Sent Rate (Tasa de bytes recibidos): el valor predeterminado muestra la media de bytes recibidos.

También puede cambiar la granularidad de los gráficos en el intervalo de tiempo. Para ello, seleccione **Avg** (Promedio), **Min** (Mínimo), **Max** (Máximo), **50th** (50), **90th** (90) y **95th** (95) en el selector de percentiles.   

Para ver el uso de recursos por máquina virtual individual en una vista de lista y ver qué máquina tiene tendencia a una mayor utilización, seleccione la pestaña **Top N List** (Lista de N principales).  En la página **Top N lista** (Lista de N principales), se muestran las 20 máquinas principales ordenadas por el mayor uso según el percentil 95 para la métrica *CPU Utilization %* (Porcentaje de uso de la CPU).  Para ver más máquinas, seleccione **Cargar más**. De este modo, los resultados se ampliarán y se mostrarán las 500 máquinas principales. 

>[!NOTE]
>La lista no puede mostrar más de 500 máquinas a la vez.  
>

![Ejemplo de la página de lista de N superiores](./media/vminsights-performance/vminsights-performance-topnlist-01.png)

Para filtrar los resultados en una máquina virtual específica de la lista, escriba el nombre de su equipo en el cuadro de texto **Buscar por nombre**.  

Si prefiere ver el uso desde otra métrica de rendimiento, en la lista desplegable **Métrica**, seleccione **Memoria disponible**, **Logical Disk Space Used %** (Logical Disk Space Used %), **Network Received Byte/s** (Bytes recibidos por la red) o **Network Sent Byte/s** (Bytes enviados por la red) y la lista se actualizará para mostrar el uso con el ámbito de esa métrica.  

Al seleccionar una máquina virtual de la lista, se abre el panel **Propiedades** del lado derecho de la página y, desde aquí, puede seleccionar **Performance detail** (Detalles de rendimiento).  Se abrirá la página **Virtual Machine Detail** (Detalles de la máquina virtual) con el ámbito en esa máquina virtual, similar a la experiencia de acceso al rendimiento de las conclusiones de la VM directamente desde la VM de Azure.  

## <a name="view-performance-directly-from-an-azure-vm"></a>Ver el rendimiento directamente desde una VM de Azure
Para tener acceso directamente desde una máquina virtual, realice los pasos siguientes.

1. En Azure Portal, seleccione **Virtual Machines**. 
2. En la lista, elija una máquina virtual y, en la sección **Monitor**, elija **Conclusiones (versión preliminar)**.  
3. Seleccione la pestaña **Rendimiento**. 

Esta página no solo incluye los gráficos de uso de rendimiento, sino también una tabla para cada disco lógico detectado, su capacidad, la utilización y la media total de cada medida.  

Se proporcionan los siguientes gráficos de uso de la capacidad:

* CPU Utilization % (Porcentaje de uso de la CPU): el valor predeterminado muestra la media y el percentil 95 superior. 
* Memoria disponible: los valores predeterminados que muestra el percentil superior, promedio de 5 y 10 
* Logical Disk Space Used % (Porcentaje de espacio usado del disco lógico): el valor predeterminado muestra la media y el percentil 95. 
* Logical Disk IOPS (IOPS del disco lógico): el valor predeterminado muestra la media y el percentil 95.
* Logical Disk MB/s (MB/s del disco lógico): el valor predeterminado muestra la media y el percentil 95.
* Max Logical Disk Used % (Porcentaje de uso máximo del disco lógico): el valor predeterminado muestra la media y el percentil 95.
* Bytes Sent Rate (Tasa de bytes enviados): el valor predeterminado muestra la media de bytes enviados. 
* Bytes Sent Rate (Tasa de bytes recibidos): el valor predeterminado muestra la media de bytes recibidos.

Al hacer clic en el icono de anclaje en la esquina superior derecha de cualquiera de las patillas de gráficos del gráfico seleccionado para el último panel de Azure que vio anteriormente. En el panel, puede cambiar el tamaño y la posición del gráfico. Seleccionar el gráfico desde el panel le redirige a Azure Monitor para las máquinas virtuales y carga la vista de detalle de rendimiento de la máquina virtual.  

![Vista de rendimiento de las conclusiones de la VM directamente desde la VM](./media/vminsights-performance/vminsights-performance-directvm-01.png)

## <a name="view-performance-directly-from-an-azure-virtual-machine-scale-set"></a>Rendimiento de la vista directamente desde un conjunto de escalado de máquina virtual de Azure
Para tener acceso directamente desde un conjunto de escalado de máquina virtual de Azure, realice los pasos siguientes.

1. En el portal de Azure, seleccione **conjuntos de escalado de máquinas virtuales**.
2. En la lista, elija una máquina virtual y, en el **supervisión** sección elija **Insights (versión preliminar)** para ver el **rendimiento** ficha.

Esta página carga la vista de rendimiento de Azure Monitor, el ámbito del conjunto de escalado seleccionado. Esto le permite ver la parte superior N instancias del conjunto de escalado en todo el conjunto de métricas supervisadas, ver el rendimiento agregado en el conjunto de escalado, y ver las tendencias de las métricas seleccionadas de los n instancias individuales la escala establecido. Al seleccionar una instancia de la vista de lista le permite cargar la asignación de TI o navegue a una vista de rendimiento detallados para esa instancia.

Al hacer clic en el icono de anclaje en la esquina superior derecha de cualquiera de las patillas de gráficos del gráfico seleccionado para el último panel de Azure que vio anteriormente. En el panel, puede cambiar el tamaño y la posición del gráfico. Seleccionar el gráfico desde el panel le redirige a Azure Monitor para las máquinas virtuales y carga la vista de detalle de rendimiento de la máquina virtual.  

![Vista del conjunto de información de máquina virtual rendimiento directamente de escalado de máquinas virtuales](./media/vminsights-performance/vminsights-performance-directvmss-01.png)

>[!NOTE]
>También puede acceder a una vista de rendimiento detallados para una instancia específica de la vista de instancias para el conjunto de escalado. Vaya a **instancias** bajo el **configuración** sección y, a continuación, elija **Insights (versión preliminar)**.

## <a name="alerts"></a>Alertas  
Las métricas de rendimiento habilitadas como parte de Azure Monitor para las máquinas virtuales no incluyen reglas de alertas preconfiguradas. Hay [alertas de estado](vminsights-health.md#alerts) correspondiente a problemas de rendimiento detectados en la máquina virtual de Azure, como el uso elevado de CPU, espacio en disco disponible y de baja memoria insuficiente, etcetera.  Sin embargo, estas alertas de estado solo se aplican a todas las máquinas virtuales habilitadas para Azure Monitor para las máquinas virtuales. 

Pero solo podemos recopilar y almacenar un subconjunto de las métricas de rendimiento que se requieren en el área de trabajo de Log Analytics. Si su estrategia de supervisión requiere análisis o alertas que incluyen otras métricas de rendimiento con el fin de evaluar eficazmente la capacidad o el mantenimiento de la máquina virtual, o si necesita flexibilidad para especificar sus propios criterios o la lógica de las alertas, puede configurar la [colección de esos contadores de rendimiento](../platform/data-sources-performance-counters.md) en Log Analytics y definir [alertas de registro](../platform/alerts-log.md). Mientras que Log Analytics le permite realizar análisis complejos con otros tipos de datos y ofrece un mayor tiempo de retención para dar cabida al análisis de tendencias, las métricas, en cambio, son ligeras y capaces de soportar escenarios en tiempo casi real. El [agente de Azure Diagnostics](../../virtual-machines/windows/monitor.md) las recopila y las almacena en el almacén de métricas de Azure Monitor, lo que permite crear alertas con una latencia y un costo menores.

Revise la información general de la [colección de métricas y registros con Azure Monitor](../platform/data-platform.md) para comprender mejor las diferencias fundamentales y otras consideraciones antes de configurar la colección de estas otras métricas y reglas de alerta.  

## <a name="next-steps"></a>Pasos siguientes
Para obtener información sobre cómo usar la característica de mantenimiento, consulte el artículo sobre cómo [ver el mantenimiento de Azure Monitor para VM](vminsights-health.md), o bien, para ver las dependencias de aplicaciones detectadas, consulte el artículo para [ver la asignación de Azure Monitor para VM](vminsights-maps.md). 