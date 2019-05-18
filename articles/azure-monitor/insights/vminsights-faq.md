---
title: 'Azure Monitor para VM (versión preliminar): preguntas más frecuentes | Microsoft Docs'
description: Azure Monitor para máquinas virtuales es una solución de Azure que combina la supervisión de rendimiento y de mantenimiento del sistema operativo de la VM de Azure, así como la detección automática de las dependencias y los componentes de aplicación con otros recursos, y asigna la comunicación entre ellos. En este artículo se responden preguntas comunes.
services: azure-monitor
author: mgoedtel
manager: carmonm
editor: tysonn
ms.service: azure-monitor
ms.topic: article
ms.workload: infrastructure-services
ms.date: 01/09/2018
ms.author: magoedte
ms.openlocfilehash: 420ba9d74532095c2d028fef8f549d532e5dfa05
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522201"
---
# <a name="azure-monitor-for-vms-preview-frequently-asked-questions"></a>Azure Monitor para VM (versión preliminar): preguntas más frecuentes
En este artículo de preguntas frecuentes de Microsoft, se presenta una lista con las preguntas frecuentes sobre Azure Monitor para VM. Si tiene alguna otra pregunta sobre esta solución, vaya al [foro de discusión](https://feedback.azure.com/forums/34192--general-feedback) y publíquela. Si una pregunta es frecuente, se agrega a este artículo para que se pueda encontrar de forma rápida y sencilla.

## <a name="can-i-onboard-to-an-existing-workspace"></a>¿Puede incorporarse a un área de trabajo existente?
Si las máquinas virtuales ya están conectadas a un área de trabajo de Log Analytics, puede seguir usando esa área de trabajo cuando se incorpore a Azure Monitor para máquinas virtuales, siempre que se encuentre en una de las regiones compatibles enumeradas [aquí](vminsights-enable-overview.md#prerequisites).

Durante la incorporación, configuramos los contadores de rendimiento para el área de trabajo que harán que todas las máquinas virtuales que estén generando informes para el área de trabajo comiencen a recopilar esta información para su presentación y análisis en Azure Monitor para máquinas virtuales.  Como resultado, verá los datos de rendimiento de todas las máquinas virtuales conectadas al área de trabajo que seleccionó.  Solo se habilitan las características de mantenimiento y de asignación para las máquinas virtuales que ha especificado para la incorporación.

Para obtener más información sobre qué rendimiento se habilitan los contadores, consulte nuestra [habilitar información general sobre](vminsights-enable-overview.md#performance-counters-enabled) artículo.

## <a name="can-i-onboard-to-a-new-workspace"></a>¿Puede incorporarse a una nueva área de trabajo? 
Si las máquinas virtuales no están conectadas actualmente a un área de trabajo de Log Analytics existente, deberá crear un área de trabajo para almacenar los datos. Un área de trabajo predeterminada se crea automáticamente si configura una sola máquina virtual de Azure para Azure Monitor para máquinas virtuales a través de Azure Portal.

Si decide usar el método basado en la secuencia de comandos, se tratan estos pasos en el [habilitar Azure Monitor para las máquinas virtuales (versión preliminar) mediante la plantilla de Resource Manager o Azure PowerShell](vminsights-enable-at-scale-powershell.md) artículo. 

## <a name="what-do-i-do-if-my-vm-is-already-reporting-to-an-existing-workspace"></a>¿Qué puedo hacer si mi máquina virtual ya está generando informes para un área de trabajo?
Si ya está recopilando datos de las máquinas virtuales, es posible que ya las haya configurado para que generen ubfirnes de datos a un área de trabajo de Log Analytics existente.  Siempre y cuando el área de trabajo se encuentre en una de nuestras regiones admitidas, podrá habilitar Azure Monitor para máquinas virtuales en esa área de trabajo preexistente.  Si el área de trabajo que ya está usando no está en una de nuestras regiones admitidas, no podrá incorporar Azure Monitor para máquinas virtuales en este momento.  Estamos trabajando para admitir más regiones.

>[!NOTE]
>Configuramos los contadores de rendimiento para el área de trabajo que afecta a todas las máquinas virtuales que se comunican con el área de trabajo, independientemente de si ha elegido incorporarlas a Azure Monitor para máquinas virtuales. Para obtener más detalles sobre cómo se configuran los contadores de rendimiento para el área de trabajo, consulte nuestra [documentación](../../azure-monitor/platform/data-sources-performance-counters.md). Para obtener información acerca de los contadores configurados para Azure Monitor para las máquinas virtuales, consulte nuestra [habilitar Azure Monitor para máquinas virtuales](vminsights-enable-overview.md#performance-counters-enabled) artículo.  

## <a name="why-did-my-vm-fail-to-onboard"></a>¿Por qué no se pudo incorporar mi máquina virtual?
Al incorporar una máquina virtual de Azure desde Azure Portal, se producen los pasos siguientes:

* Se crea un área de trabajo de Log Analytics predeterminada si se ha seleccionado la opción.
* Los contadores de rendimiento se configuran para el área de trabajo seleccionada. Si se produce un error en este paso, verá que algunas de las tablas y gráficos de rendimiento no muestran datos para la máquina virtual que incorporó. Puede solucionar este problema si ejecuta el script de PowerShell documentado [aquí](vminsights-enable-at-scale-powershell.md#enable-performance-counters).
* Se instala el agente de Log Analytics en las máquinas virtuales de Azure mediante la extensión de máquina virtual si se determina que es necesario.  
* Se instala el agente de la dependencia de asignación de Azure Monitor para máquinas virtuales mediante una extensión si se determina que es necesario.  
* De ser necesario, se configuran los componentes de Azure Monitor que admiten la característica de estado y la máquina virtual se configura para notificar sobre los datos de estado.

Durante el proceso de incorporación, comprobamos el estado de cada uno de los pasos anteriores para devolver un estado de notificación en el portal. La configuración del área de trabajo y la instalación del agente normalmente tarda de 5 a 10 minutos. La visualización de los datos de supervisión y estado en el portal tarda otros 5 o 10 minutos.  

Si ha iniciado la incorporación y puede ver los mensajes que indican que la máquina virtual debe incorporarse, espere hasta 30 minutos para que la máquina virtual pueda completar el proceso. 

## <a name="i-only-enabled-azure-monitor-for-vms-why-do-i-see-all-my-vms-monitored-by-the-health-feature"></a>Solo habilité Azure Monitor para VM, ¿por qué veo que la característica de mantenimiento supervisa todas las máquinas virtuales?
La característica de mantenimiento está habilitada para todas las máquinas virtuales conectadas al área de trabajo de Log Analytics, incluso cuando la acción se inicia para una sola máquina virtual.

## <a name="can-i-modify-the-schedule-for-when-health-criteria-evaluates-a-condition"></a>¿Puedo modificar la programación de cuándo los criterios de mantenimiento evalúan una condición?
No, el período y la frecuencia de los criterios de mantenimiento no se pueden modificar con esta versión. 

## <a name="can-i-disable-health-criteria-for-a-condition-i-dont-need-to-monitor"></a>¿Puedo deshabilitar los criterios de mantenimiento para una condición que no es necesario supervisar?
Los criterios de mantenimiento no se pueden deshabilitar en esta versión.

## <a name="are-the-health-alert-severities-configurable"></a>¿Es posible configurar la gravedad de las alertas de mantenimiento?  
No se puede modificar la gravedad de la alerta de mantenimiento, solo se habilitar o deshabilitar. Además, las gravedades de algunas alertas se actualizan según el estado de los criterios de mantenimiento. 

## <a name="if-i-reconfigure-the-settings-of-a-particular-health-criteria-can-it-be-scoped-to-a-specific-instance"></a>Si vuelvo a configurar los valores de un criterio de mantenimiento determinado, ¿lo puedo limitar a una instancia específica?  
Si modifica alguna configuración de un criterio de mantenimiento, se modifican todos los criterios de estado del mismo tipo en la máquina virtual de Azure. Por ejemplo, si se modifica el umbral del criterio de mantenimiento del espacio libre en el disco correspondiente al disco lógico C:, este umbral se aplicará a todos los demás discos lógicos detectados y supervisados para la misma máquina virtual.

## <a name="does-the-health-feature-monitor-logical-processors-and-cores"></a>¿La característica de mantenimiento supervisa los núcleos y procesadores lógicos?
No, los criterios de mantenimiento de nivel de procesador individual y de procesador lógico no se incluyen para Windows, solo se supervisa el uso de CPU total de manera predeterminada para evaluar de manera eficaz la presión de CPU en función del número total de CPU lógicas disponibles para la máquina virtual de Azure. 

## <a name="are-all-health-criteria-thresholds-configurable"></a>¿Es posible configurar todos los umbrales de los criterios de mantenimiento?  
Los umbrales de los criterios de mantenimiento que tienen como destino una máquina virtual Windows no son modificables, puesto que su estado está establecido como *en ejecución* o *disponible*. Cuando se consulta el estado de mantenimiento desde la [API Workload Monitor](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/components), muestra el valor *comparisonOperator* de **LessThan** o **GreaterThan** con un valor de *threshold* de **4** para el servicio o la entidad si:
   - Service Health del cliente DNS: el servicio no se está ejecutando. 
   - Service Health del cliente DHCP: el servicio no se está ejecutando. 
   - Service Health de RPC: el servicio no se está ejecutando. 
   - Service Health del firewall de Windows: el servicio no se está ejecutando.
   - Service Health del registro de eventos de Windows: el servicio no se está ejecutando. 
   - Service Health del servidor: el servicio no se está ejecutando. 
   - Service Health del servicio de administración remota de Windows: el servicio no se está ejecutando. 
   - Error o daño del sistema de archivos: el disco lógico no está disponible.

Los umbrales de los siguientes criterios de mantenimiento de Linux no son modificables, ya que su estado de mantenimiento ya está establecido en *true*. El estado de mantenimiento muestra *comparisonOperator* con un valor **LessThan** y un valor de *threshold* de **1** cuando la API Workload Monitor realiza una consulta sobre la entidad en función de su contexto:
   - Estado del disco lógico: el disco lógico no está en línea ni está disponible
   - Estado del disco: el disco no está en línea ni está disponible
   - Estado del adaptador de red: el adaptador de red está deshabilitado

## <a name="how-do-i-modify-alerts-that-are-included-with-the-health-feature"></a>¿Cómo puedo modificar las alertas que se incluyen con la característica de mantenimiento?
Las reglas de alertas que se definen para cada criterio de mantenimiento no se muestran en Azure Portal. Solo puede habilitar o deshabilitar una regla de alerta de estado en la [API Workload Monitor](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/components). Además, no puede asignar un [grupo de acciones de Azure Monitor](../../azure-monitor/platform/action-groups.md) para las alertas de estado desde Azure Portal. Solo puede usar la API de configuración de notificaciones para configurar un grupo de acciones que se desencadene cada vez que se desencadena una alerta de estado. Actualmente, puede asignar grupos de acciones en una máquina virtual, de tal forma que todas las *alertas de estado* desencadenadas en la máquina virtual desencadenen los mismos grupos de acciones. A diferencia de las alertas de Azure tradicionales, no existe el concepto de un grupo de acciones independiente para cada regla de alertas de mantenimiento. Además, solo se admiten los grupos de acciones configurados para enviar notificaciones mediante un correo electrónico o SMS cuando se desencadenan alertas de estado. 

## <a name="i-dont-see-some-or-any-data-in-the-performance-charts-for-my-vm"></a>No veo algunos o ninguno de los datos en el gráfico de rendimiento de la máquina virtual
Si no ve los datos de rendimiento en la tabla del disco o en algunos de los gráficos de rendimiento, es posible que los contadores de rendimiento en el área de trabajo no estén configurados. Para solucionar este problema, ejecute el siguiente [script de PowerShell](vminsights-enable-at-scale-powershell.md#enable-with-powershell).

## <a name="how-is-azure-monitor-for-vms-map-feature-different-from-service-map"></a>¿En qué se diferencian la característica de asignación de Azure Monitor para máquinas virtuales y Service Map?
La característica de asignación de Azure Monitor para máquinas virtuales está basada en Service Map, pero se diferencia en los siguientes aspectos:

* Se puede acceder a la vista de asignación desde la hoja de máquina virtual y desde Azure Monitor para máquinas virtuales en Azure Monitor.
* Ahora se puede hacer clic sobre las conexiones en la asignación y, en el panel lateral, muestran una vista de los datos de métricas de la conexión seleccionada.
* Hay una nueva API que se usa para crear las asignaciones, lo que ofrece una mejor compatibilidad con asignaciones más complejas.
* Ahora se incluyen máquinas virtuales supervisadas en el nodo de grupo del cliente y el gráfico de anillos muestra la proporción de máquinas virtuales no supervisadas frente a las supervisadas en el grupo.  También puede usarse para filtrar la lista de máquinas cuando el grupo está expandido.
* Ahora se incluyen las máquinas virtuales supervisadas en los nodos de grupo de los puertos del servidor, y el gráfico de anillos muestra la proporción de máquinas supervisadas frente a las no supervisadas en el grupo.  También puede usarse para filtrar la lista de máquinas cuando el grupo está expandido.
* El estilo de la asignación se actualizó para que sea más coherente con el mapa de aplicación de Application Insights.
* Se han actualizado los paneles laterales y no tiene el conjunto completo de integración que se admitían en Service Map - Update Management, Change Tracking, seguridad y servicio de asistencia. 
* La opción para elegir los grupos y máquinas que se asignarán se ha actualizado y ahora es compatible con las suscripciones, grupos de recursos, conjuntos de escalado de máquinas virtuales de Azure y servicios en la nube.
* No puede crear grupos de máquinas de Service Map en la característica de asignación de Azure Monitor para máquinas virtuales.  

## <a name="why-do-my-performance-charts-show-dotted-lines"></a>¿Por qué mi gráficos de rendimiento muestran líneas punteadas?
Esto puede ocurrir por varios motivos.  En los casos donde hay una discontinuidad en la recopilación de datos, las líneas se muestran punteadas.  Si ha modificado la frecuencia de muestreo de datos para los contadores de rendimiento habilitados (el valor predeterminado es recopilar datos cada 60 segundos), podrá ver líneas punteadas en el gráfico si elige un intervalo de tiempo reducido para el gráfico y su frecuencia de muestreo es menor que el tamaño de depósito utilizado en el gráfico (por ejemplo, la frecuencia de muestreo es cada 10 minutos y cada depósito en el gráfico es de 5 minutos).  En este caso, elegir un intervalo de tiempo más amplio para la visualización debe hacer que las líneas del gráfico sean sólidas en lugar de punteadas.

## <a name="are-groups-supported-with-azure-monitor-for-vms"></a>¿Los grupos son compatibles con Azure Monitor para máquinas virtuales?
Sí, una vez que se instala Dependency Agent recopilamos información de las máquinas virtuales para mostrar los grupos por suscripción, grupo de recursos,conjunto de escalado de máquinas virtuales y servicios en la nube.  Si ha usado Service Map y ha creado grupos de máquinas, también se muestran.  Los grupos de equipos también aparecerán en el filtro de grupos si los ha creado para el área de trabajo que ve. 

## <a name="how-do-i-see-the-details-for-what-is-driving-the-95th-percentile-line-in-the-aggregate-performance-charts"></a>¿Cómo puedo ver los detalles de lo que está aumentando la línea del percentil 95 en los gráficos de rendimiento agregado?
De forma predeterminada, la lista está ordenada para mostrar las máquinas virtuales con el valor de percentil 95 más alto de la métrica seleccionada, con la excepción del gráfico de memoria disponible, que muestra las máquinas con el valor más bajo de percentil 5.  Al hacer clic en el gráfico, se abrirá la vista **Top N List** (Lista de N más altos) con la métrica adecuada seleccionada.

## <a name="how-does-the-map-feature-handle-duplicate-ips-across-different-vnets-and-subnets"></a>¿Cómo administra la característica de asignación las direcciones IP duplicadas entre distintas redes virtuales y subredes?
Si va a duplicar los intervalos de IP con máquinas virtuales o conjuntos de escalado de máquinas virtuales de Azure entre subredes y redes virtuales, puede hacer que la asignación de Azure Monitor para máquinas virtuales muestre información incorrecta. Se trata de un problema conocido y estamos investigando opciones para mejorar esta experiencia.

## <a name="does-map-feature-support-ipv6"></a>¿La característica de asignación es compatible con IPv6?
Por el momento, la característica de asignación solo es compatible con IPv4 y estamos investigando la compatibilidad con IPv6. También es compatible con IPv4 que se tuneliza dentro de IPv6.

## <a name="when-i-load-a-map-for-a-resource-group-or-other-large-group-the-map-is-difficult-to-view"></a>Cuando cargo una asignación para un grupo de recursos o algún otro grupo grande me resulta difícil verla.
Aunque hemos realizado mejoras a la asignación para que controle configuraciones grandes y complejas, somos conscientes de que una asignación puede tener una gran cantidad de nodos, conexiones y nodos que funcionen como un clúster.  Nos comprometemos a seguir mejorando la compatibilidad para aumentar la escalabilidad.   

## <a name="why-does-the-network-chart-on-the-performance-tab-look-different-than-the-network-chart-on-the-azure-vm-overview-page"></a>¿Por qué el gráfico de red de la pestaña Rendimiento es distinta al gráfico de red de la página Información general de la máquina virtual de Azure?

La página de información general de una máquina virtual de Azure muestra gráficos basados en la medición de actividad de la máquina virtual invitada que realiza el host.  En el gráfico de red de la información general de la máquina virtual de Azure, solo se muestra el tráfico de red que se facturará.  Esto no incluye el tráfico entre redes virtuales.  Los datos y gráficos que se muestran en Azure Monitor para máquinas virtuales se basan en los datos de la máquina virtual invitada, y el gráfico de red muestra todo el tráfico TCP/IP entrante y saliente de esa máquina virtual, incluido el que fluye entre redes virtuales.

## <a name="how-is-response-time-measured-for-data-stored-in-vmconnection-and-displayed-in-the-connection-panel-and-workbooks"></a>¿Cómo se mide el tiempo de respuesta para los datos almacenados en VMConnection y aparece en el panel de la conexión y los libros?

Tiempo de respuesta es una aproximación. Puesto que no se instrumenta el código de la aplicación, que no sabemos realmente cuando comienza una solicitud y cuando llegue la respuesta. En su lugar, observamos que se envían en una conexión de datos y, a continuación, los datos devueltos en esa conexión. Nuestro agente realiza un seguimiento de estos envía y recibe e intenta emparejarlas: recibe una secuencia de envíos, seguido de una secuencia de se interpreta como un par de solicitud/respuesta. Tiempo que transcurre entre estas operaciones es el tiempo de respuesta. Se incluyen la latencia de red y el tiempo de procesamiento del servidor.

Esta aproximación funciona bien para los protocolos que se basa en solicitud/respuesta: una única solicitud sale de la conexión y recibe una respuesta única. Este es el caso de HTTP (S) (sin la canalización), pero no cumplida para otros protocolos.

## <a name="are-their-limitations-if-i-am-on-the-log-analytics-free-pricing-plan"></a>¿Existen limitaciones si estoy en el plan de tarifa gratis de Log Analytics?
Si ha configurado Azure Monitor con un área de trabajo de Log Analytics mediante el plan de tarifa *gratis*, la característica de asignación de Azure Monitor para máquinas virtuales solo admitirá cinco máquinas conectadas. Si tiene cinco máquinas virtuales conectadas a un área de trabajo gratuita, desconecte una para poder conectar otra nueva. La nueva máquina virtual que conecte no se supervisará ni se reflejará en la página de asignación.  

En esta condición, verá la opción **Probar ahora** al abrir la máquina virtual y seleccionar la opción **Insights (versión preliminar)** en el panel izquierdo, incluso después de que se haya instalado en la máquina virtual.  Sin embargo, no se le presentarán opciones como ocurriría si estas VM no estuvieran incorporadas en Azure Monitor para VM. 

## <a name="next-steps"></a>Pasos siguientes
Revisión [habilitar Azure Monitor para las máquinas virtuales](vminsights-enable-overview.md) para comprender los requisitos y los métodos para habilitar la supervisión de las máquinas virtuales.
