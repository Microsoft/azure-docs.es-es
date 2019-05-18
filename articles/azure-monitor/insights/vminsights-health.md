---
title: Supervisión del estado de la máquina virtual con Azure Monitor para VM (versión preliminar) | Microsoft Docs
description: En este artículo se describe cómo entender el estado de la máquina virtual y el sistema operativo subyacente con Azure Monitor para las máquinas virtuales.
services: azure-monitor
documentationcenter: ''
author: mgoedtel
manager: carmonm
editor: ''
ms.assetid: ''
ms.service: azure-monitor
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/12/2019
ms.author: magoedte
ms.openlocfilehash: 45c9a8da8344aa6aaaa19b534451a7276e96911a
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522186"
---
# <a name="understand-the-health-of-your-azure-virtual-machines"></a>Conocer el estado de las máquinas virtuales de Azure

Azure incluye varios servicios que individualmente realizan una tarea o un rol específico en el espacio de supervisión, pero aún no ha proporcionado una perspectiva detallada del estado del sistema operativo hospedado en Azure virtual machines. Aunque podría supervisar para condiciones diferentes mediante Azure Monitor, se no se ha diseñado para modelar y representan el estado general de la máquina virtual o de mantenimiento de los componentes principales. Con la característica de estado de Azure Monitor para máquinas virtuales, la disponibilidad y el rendimiento del sistema operativo invitado Windows o Linux se supervisan de forma proactiva con un modelo que representa los principales componentes y sus relaciones, así como los criterios que especifican cómo se debe medir el estado de dichos componentes y, además, le avisa cuando se detecta una condición de estado incorrecto.  

El estado de mantenimiento general de la máquina virtual de Azure y el sistema operativo subyacente pueden observarse desde dos perspectivas con el estado de Azure Monitor para máquinas virtuales, ya sea directamente desde la máquina virtual o bien en todas las máquinas virtuales de un grupo de recursos de Azure Monitor.

En este artículo le ayudamos a comprender cómo puede evaluar, investigar y resolver rápidamente los problemas detectados.

Para obtener más información sobre cómo configurar Azure Monitor para máquinas virtuales, consulte el artículo [Enable Azure Monitor for VMs](vminsights-enable-overview.md) (Habilitar Azure Monitor para máquinas virtuales).

## <a name="monitoring-configuration-details"></a>Detalles de configuración de supervisión

En esta sección se describen los criterios de estado predeterminados definidos para supervisar las máquinas virtuales Linux y Windows de Azure. Todos los criterios de mantenimiento están preconfigurados para enviar una alerta cuando se dé una condición de mal estado. 

### <a name="windows-vms"></a>VM Windows

- Megabytes disponibles de memoria 
- Media de segundos del disco por escritura (disco lógico)
- Media de segundos del disco por escritura (disco)
- Media de segundos del disco lógico por lectura
- Media de segundos del disco lógico por transferencia
- Media de segundos del disco por lectura
- Media de segundos del disco por transferencia
- Longitud actual de cola de disco (disco lógico)
- Longitud actual de cola de disco (disco)
- Porcentaje de tiempo de inactividad del disco
- Error o daños en el sistema de archivos
- Poco espacio disponible en el disco lógico (%)
- Poco espacio disponible en el disco lógico (MB)
- Porcentaje de tiempo de inactividad del disco lógico
- Páginas de memoria por segundo
- Porcentaje de ancho de banda usado para la lectura
- Porcentaje total del ancho de banda usado
- Porcentaje de ancho de banda para la escritura
- Porcentaje de memoria asignada en uso
- Porcentaje de tiempo de inactividad del disco
- Service Health del cliente DHCP
- Service Health del cliente DNS
- Service Health de RPC
- Service Health de servidor
- Porcentaje de uso total de la CPU
- Service Health del registro de eventos de Windows
- Service Health del firewall de Windows
- Service Health de Administración remota de Windows

### <a name="linux-vms"></a>VM Linux
- Media del disco Segundos de disco/transferencias 
- Media del disco Segundos de disco/lecturas 
- Media del disco Segundos de disco/escrituras 
- Estado del disco
- Espacio disponible en el disco lógico
- % de espacio disponible en el disco lógico
- % de inode disponibles en el disco lógico
- Estado del adaptador de red
- Porcentaje de tiempo del procesador total
- Megabytes disponibles de memoria del sistema operativo

## <a name="sign-in-to-the-azure-portal"></a>Iniciar sesión en Azure Portal

Inicie sesión en el [Azure Portal](https://portal.azure.com). 

## <a name="introduction-to-health-experience"></a>Introducción a la experiencia del estado

Antes de profundizar en cómo usar la característica de estado para una sola máquina virtual o un grupo de VM, es importante que brindemos una breve introducción para que pueda comprender cómo se presenta la información y qué representan las visualizaciones.  

## <a name="view-health-directly-from-a-virtual-machine"></a>Visualización del estado directamente desde una máquina virtual 

Para ver el estado de una máquina virtual de Azure, seleccione **Conclusiones (versión preliminar)** desde el panel izquierdo de la máquina virtual. En la página de conclusiones de la VM, el **estado** se abre de forma predeterminada y se muestra la vista de estado de la máquina virtual.  

![Información general del estado de Azure Monitor para máquinas virtuales de una máquina virtual de Azure seleccionada](./media/vminsights-health/vminsights-directvm-health.png)

En la pestaña **Estado** de la sección **Estado de la VM invitada**, la tabla muestra el estado de mantenimiento actual de la máquina virtual y el número total de alertas de estado de la máquina virtual generado por un componente en mal estado. Para obtener más información, consulte [alertas](#alerts) para obtener más información acerca de la alerta de experiencia.  

En la tabla siguiente se describen los estados de mantenimiento definidos para una VM: 

|Icono |Estado de mantenimiento |Significado |
|-----|-------------|------------|
| |Estado correcto |El estado de mantenimiento es correcto si se encuentra dentro de las condiciones de mantenimiento definidas, lo cual indica que no se detectó ningún problema en la VM y que funciona según lo esperado. Con un monitor de acumulación primario, acumula mantenimiento y se refleja el estado de la mejor o peor del elemento secundario.|
| |Fundamental |El estado de mantenimiento es crítico si no está dentro de la condición de mantenimiento definida, lo cual indica que se han detectado uno o más problemas críticos que deben solucionarse para poder restaurar el funcionamiento normal. Con un monitor de acumulación primario, acumula mantenimiento y se refleja el estado de la mejor o peor del elemento secundario.|
| |Advertencia |El estado de mantenimiento de advertencia se activa si está entre dos umbrales de la condición de mantenimiento definida; uno de ellos indica un estado de *Advertencia* y el otro indica un estado *Crítico* (pueden configurarse tres umbrales de estado de mantenimiento). También puede activarse cuando se detecta un problema no crítico, pero que puede ocasionar problemas graves si no se resuelve. Con una acumulación primario monitor, si uno o varios de los elementos secundarios están en un estado de advertencia y, después, reflejará el elemento primario *advertencia* estado. Si hay un elemento secundario que se encuentra en estado *Crítico* y otro en estado *Advertencia*, la acumulación primaria mostrará un estado de mantenimiento *Crítico*.|
| |Desconocida |El estado de mantenimiento es *Desconocido* cuando no se puede calcular por varios motivos; por ejemplo, cuando no se pueden recopilar los datos, si no se inicializó el servicio, etc. Este estado de mantenimiento no se puede configurar.| 

Si selecciona **Ver el diagnóstico de estado**, se abre una página que muestra todos los componentes de la VM, los criterios de estado asociados, los cambios de estado y otros temas importantes detectados mediante la supervisión de componentes relacionados con la máquina virtual. Para más información, consulte [Diagnóstico de mantenimiento](#health-diagnostics). 

En la sección **Estado del componente**, la tabla muestra un estado acumulativo del mantenimiento de las categorías de rendimiento principales supervisadas según los criterios de estado de esas áreas, específicamente la **CPU**, la **memoria**, el **disco** y la **red**.  Si selecciona cualquiera de los componentes, se abre una página en la que se enumeran todos los aspectos de supervisión de los criterios de estado individuales de ese componente y el estado de mantenimiento correspondiente de cada uno.  

Al obtener acceso al estado de una máquina virtual de Azure que ejecutan el sistema operativo de Windows, el estado de mantenimiento de los principales cinco principales de Windows services se muestran en la sección **estado de los servicios básicos**.  Cuando selecciona cualquiera de los servicios, se abre una página que enumera los criterios de estado que supervisan dicho componente y su estado de mantenimiento.  Al hacer clic en el nombre de los criterios de mantenimiento, se abrirá el panel de propiedades y, desde aquí, puede revisar los detalles de configuración, por ejemplo, si los criterios de estado tienen definida una alerta de Azure Monitor correspondiente. Para más información, consulte [Health Diagnostics and working with health criteria](#health-diagnostics) (Diagnóstico de mantenimiento y trabajo con los criterios de mantenimiento).  

## <a name="aggregate-virtual-machine-perspective"></a>Perspectiva de la máquina virtual agregada

Para ver la colección de mantenimiento de todas las máquinas virtuales de un grupo de recursos, vaya a la lista de navegación del portal, seleccione **Azure Monitor** y, a continuación, seleccione **Máquinas virtuales (versión preliminar)**.  

![Vista de supervisión de conclusiones de la VM de Azure Monitor](./media/vminsights-health/vminsights-aggregate-health.png)

En las listas desplegables **Suscripción** y **Grupo de recursos**, seleccione el grupo de recursos adecuado que incluya las máquinas virtuales relacionadas con el grupo para ver el estado de mantenimiento notificado.  La selección solo se aplica a la característica de mantenimiento y no se aplica a las secciones de rendimiento o asignación.

En la pestaña **Mantenimiento**, puede aprender lo siguiente:

* ¿Cuántas VM se encuentran en un estado crítico o incorrecto, frente a cuántas están en buen estado o no envían datos (es decir, tienen un estado desconocido)?
* ¿Qué máquinas virtuales según el sistema operativo (SO) notifican un estado incorrecto y cuántas?
* ¿Cuántas VM tienen un estado incorrecto debido a un problema detectado con un procesador, un disco, la memoria o un adaptador de red, clasificados según el estado de mantenimiento?  
* ¿Cuántas VM tienen un estado incorrecto debido a un problema detectado con un procesador, un disco, la memoria o un adaptador de red, clasificados según el estado de mantenimiento?

Aquí puede identificar rápidamente los principales problemas críticos detectados por los criterios de mantenimiento que supervisan la VM de forma proactiva, así como revisar los detalles de la alerta de estado de la VM y el artículo de conocimiento asociado a fin de obtener ayuda para diagnosticar y solucionar el problema.  Seleccione cualquiera de las gravedades para abrir la página [Todas las alertas](../../azure-monitor/platform/alerts-overview.md#all-alerts-page) filtrada según esa gravedad.

En la lista **VM distribution by operating system** (Distribución de la máquina virtual por sistema operativo) se muestran las VM que enumera la edición de Windows o la distribución de Linux, junto con su versión. En cada categoría de sistema operativo, las VM se desglosan aún más según el estado de la VM. 

![Perspectiva de la distribución de la máquina virtual de conclusiones de la VM](./media/vminsights-health/vminsights-vmdistribution-by-os.png)

Puede hacer clic en cualquier elemento de columna (**Número de VM**, **Crítico**, **Advertencia**, **Correcto** o **Desconocido**) para explorar en profundidad la página **Máquinas virtuales** y ver una lista de los resultados filtrados que coincidan con la columna seleccionada. Por ejemplo, si quiere revisar todas las VM que ejecutan la versión **Red Hat Enterprise Linux 7.5**, haga clic en el valor **Número de VM** de ese sistema operativo y se abrirá la página siguiente, donde se enumerarán las máquinas virtuales que coincidan con ese filtro y su estado de mantenimiento conocido actualmente.  

![Ejemplo de la acumulación de VM Red Hat Linux](./media/vminsights-health/vminsights-rollup-vm-rehl-01.png)
 
En la página **Máquinas virtuales**, si selecciona el nombre de una VM en la columna **Nombre de VM**, se le dirigirá a la página de la instancia de la VM con más detalles acerca de los problemas con criterios de estado y alertas identificados que afectan a la VM seleccionada.  Desde aquí, puede filtrar los detalles del estado de mantenimiento. Para ello, haga clic en el icono **Estado de mantenimiento** en la esquina superior izquierda de la página para ver qué componentes son incorrectos, o bien puede ver las alertas de estado de la VM generadas por un componente con un estado incorrecto y clasificadas según la gravedad.    

Desde la vista de la lista de VM, si hace clic en el nombre de una VM, se abre la página **Mantenimiento** de esa VM seleccionada, del mismo modo que si selecciona **Conclusiones (versión preliminar)** desde la VM directamente.

![Conclusiones de una máquina virtual de Azure seleccionada](./media/vminsights-health/vminsights-directvm-health.png)

A continuación, se muestra el **estado de mantenimiento** de una acumulación de la máquina virtual y las **alertas**, clasificadas según la gravedad, que representan las alertas de estado de la VM generadas cuando el estado de mantenimiento cambia de correcto a incorrecto según los criterios de estado.  Si selecciona **VMs in critical condition** (VM en una condición crítica), se abrirá una página con una lista de una o varias VM que se encuentran en un estado de mantenimiento crítico.  Al hacer clic en el estado de mantenimiento de una de las VM de la lista, se mostrará la vista **Diagnóstico de mantenimiento** de la VM.  A continuación, puede averiguar los criterios de estado que reflejan un problema de estado de mantenimiento. Cuando se abre la página **Diagnóstico de mantenimiento**, se muestran todos los componentes de la VM y sus criterios de estado asociados con el estado de mantenimiento actual. Para obtener más información, consulte [mantenimiento diagnóstico](#health-diagnostics).  

Al seleccionar **Ver todos los criterios de estado**, se abre una página que muestra una lista de todos los criterios de mantenimiento disponibles con esta característica.  La información se puede filtrar aún más en función de las siguientes opciones:

* **Tipo**: existen tres tipos de criterios de estado para evaluar condiciones y acumular el estado de mantenimiento general de la VM supervisada.  
     a. **Unidad**: mide algún aspecto de la máquina virtual. Es posible que este tipo de criterios de estado comprueben un contador de rendimiento para determinar el rendimiento del componente, mediante la ejecución de un script para realizar una transacción sintética, o controlar un evento que indica un error.  De forma predeterminada, el filtro se establece en la unidad.  
    b. **Dependencia**: proporciona la acumulación de estado entre las distintas entidades. Los criterios de este estado permiten que el estado de una entidad dependa del estado de otro tipo de entidad en la que se basa para que la operación se realice correctamente.  
    c. **Agregado**: proporciona un estado de mantenimiento combinado de los criterios de estado similares. Los criterios de estado de unidad y de dependencia normalmente se configurarán a partir de un criterio de estado agregado. Además de proporcionar una mejor organización general de los distintos criterios de mantenimiento de una entidad, los criterios de estado agregados proporcionan un estado único de mantenimiento para las distintas categorías de las entidades.

* **Categoría**: tipo de criterios de estado que se usan para agrupar los criterios de un tipo similar con fines informativos.  Corresponden a los de **disponibilidad** o **rendimiento**.

Puede explorar en profundidad para ver las instancias que tienen un estado incorrecto. Para ello, haga clic en un valor de la columna **Unhealthy Component** (Componente con un estado incorrecto).  En la página, una tabla enumera los componentes que se encuentran en un estado de mantenimiento crítico.    

## <a name="health-diagnostics"></a>Diagnóstico de mantenimiento

La página **Diagnóstico de mantenimiento** le permite ver el modelo de mantenimiento de una máquina virtual y enumerar todos los componentes de la máquina virtual, los criterios de mantenimiento asociados, los cambios de estado y otros problemas importantes detectados mediante la supervisión de componentes relacionados con la máquina virtual.

![Ejemplo de la página Diagnóstico de mantenimiento para una VM](./media/vminsights-health/health-diagnostics-page-01.png)

Puede iniciar el diagnóstico de mantenimiento de las maneras siguientes.

* Mediante el estado de mantenimiento acumulativo de todas las VM desde la perspectiva de la VM agregada en Azure Monitor.  En la página **Mantenimiento**, haga clic en el icono del estado de mantenimiento **Crítico**, **Advertencia**, **Correcto** o **Desconocido** de la sección **Estado de la VM invitada** y explore en profundidad la página que enumera todas las VM que coincidan con esa categoría filtrada.  Al hacer clic en el valor de la columna **Estado de mantenimiento**, se abrirán los diagnósticos de mantenimiento con ámbito en esa máquina virtual concreta.      

* Mediante el sistema operativo desde la perspectiva de la VM agregada en Azure Monitor. En **VM distribution** (Distribución de VM), al seleccionar cualquiera de los valores de la columna, se abrirá la página **Máquinas virtuales** y se devolverá una lista en la tabla que coincida con la categoría filtrada.  Al hacer clic en el valor de la columna **Estado de mantenimiento**, se abre el diagnóstico de mantenimiento para la máquina virtual seleccionada.    
 
* Desde la VM invitada de la pestaña **Mantenimiento** de Azure Monitor para máquinas virtuales, seleccione **Ver el diagnóstico de estado**. 

El diagnóstico de mantenimiento organiza la información de estado en las siguientes categorías: 

* Disponibilidad
* Rendimiento
 
Todos los criterios de estado definidos para un componente específico, como el disco lógico, CPU, etc. pueden verse sin filtrar en las dos categorías (es decir, una vista total de todos los criterios) o filtrar los resultados por cualquier categoría cuando se selecciona **disponibilidad**  o **rendimiento** opciones en la página. Además, la categoría de los criterios puede verse junto a él en el **criterios de estado** columna. Si los criterios no coincide con la categoría seleccionada, mostrará el mensaje **ningún criterio de mantenimiento disponible para la categoría seleccionada** en el **criterios de estado** columna.  

El estado de los criterios de mantenimiento se define mediante uno de estos cuatro estados: *Crítico*, *Advertencia*, *Correcto* y *Desconocido*. Los tres primeros son configurables, lo que significa que puede modificar los valores de umbral de los monitores directamente desde el panel de configuración de los criterios de estado o mediante la API de REST de Azure Monitor [monitor la operación de actualización](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/monitors/update). *Desconocido*: este valor no es configurable y se reserva para escenarios específicos.  

La página de diagnóstico de mantenimiento tiene tres secciones principales:

* Modelo de componentes 
* Criterios del mantenimiento
* Cambios de estado 

![Secciones de la página Diagnóstico de mantenimiento](./media/vminsights-health/health-diagnostics-page-02.png)

### <a name="component-model"></a>Modelo de componente

La columna del extremo izquierdo de la página de diagnóstico de mantenimiento es el modelo del componente. Todos los componentes asociados a la máquina virtual y su estado de mantenimiento actual se muestran en esta columna. 

En el ejemplo siguiente, los componentes detectados son el disco, el disco lógico, el procesador, la memoria y el sistema operativo. Varias instancias de estos componentes se detectan y se muestran en esta columna. Por ejemplo, en la imagen siguiente se muestra que la máquina virtual tiene dos instancias de discos lógicos (C: y D:) que no están en buen estado.  

![Ejemplo del modelo de componente presentado en el diagnóstico de mantenimiento](./media/vminsights-health/health-diagnostics-page-component.png)

### <a name="health-criteria"></a>Criterios de mantenimiento

La columna central de la página de diagnóstico de mantenimiento es **Criterios de mantenimiento**. El modelo de estado definido para la VM se muestra en un árbol jerárquico. El modelo de mantenimiento de una máquina virtual está compuesto por los criterios de mantenimiento agregados y de unidad.  

![Ejemplo de los criterios de mantenimiento presentados en la página Diagnóstico de mantenimiento](./media/vminsights-health/health-diagnostics-page-healthcriteria.png)

Un criterio de mantenimiento mide el mantenimiento de la instancia supervisada con algunos criterios, lo que podría ser un valor de umbral, un estado de una entidad, etc. Dicho criterio tiene dos o tres umbrales configurables de estado de mantenimiento, tal y como se describió en la sección anterior. En cualquier momento dado, el criterio de mantenimiento solo puede estar en uno de sus estados posibles. 

El mantenimiento general de un destino viene determinado por el mantenimiento de cada uno de sus criterios de mantenimiento definidos en el modelo de mantenimiento. Es una combinación de criterios de estado de destino directamente en el destino, dirigidos a los componentes de acumular en el destino a través de un criterio de estado agregado de criterios de estado. Esta jerarquía se ilustra en la sección **Criterios de mantenimiento** de la página Diagnóstico de mantenimiento. La directiva de acumulación de mantenimiento es la parte de la configuración de los criterios de mantenimiento agregados (el valor predeterminado está establecido en *Worst-of*). Puede encontrar una lista del conjunto predeterminado de los criterios de estado que se ejecuta como parte de esta característica en la sección [detalles de configuración de supervisión](#monitoring-configuration-details), y se puede usar la API de REST de Azure Monitor [supervisar instancias - lista de recursos operación](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/monitorinstances/listbyresource) para obtener una lista de todos los criterios de mantenimiento y su configuración detallada que se ejecuta en el recurso de máquina virtual de Azure.  

Si quiere, puede modificar la configuración del tipo **Unidad** de los criterios de mantenimiento. Para ello, haga clic en el vínculo de elipse del lado derecho y seleccione **Show Details** (Mostrar detalles) para abrir el panel de configuración. 

![Ejemplo de la configuración de un criterio de mantenimiento](./media/vminsights-health/health-diagnostics-vm-example-02.png)

En el panel de configuración de los criterios de mantenimiento seleccionados, si se fija en el ejemplo **Average Disk Seconds Per Write** (Promedio de segundos de disco por escritura), verá que el umbral puede configurarse con un valor numérico diferente. Este es un sistema de supervisión de dos estados, lo que significa que solo cambia de Correcto a Advertencia. También puede haber un criterio de mantenimiento de tres estados, en el que puede configurar un valor para el umbral de estado de mantenimiento de advertencia y crítico. También puede modificar el umbral de uso de la API de REST de Azure Monitor [monitor la operación de actualización](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/monitors/update).

>[!NOTE]
>Si aplica el cambio de configuración de los criterios de mantenimiento a una instancia, también se aplicará a todas las instancias supervisadas.  Por ejemplo, si selecciona **Disk -1 D:** (Disco - 1 D:) y modifica el umbral **Average Disk Seconds Per Write** (Promedio de segundos de disco por escritura), este no se aplica únicamente a esa instancia, sino a todas las demás instancias del disco que se detectan y supervisan en la VM.
>

![Ejemplo de la configuración de un criterio de mantenimiento de un monitor de unidad](./media/vminsights-health/health-diagnostics-criteria-config-01.png)

Si quiere obtener más información sobre el indicador de mantenimiento, se incluyen artículos de conocimiento que le ayudarán a identificar los problemas, sus causas y las soluciones. Haga clic en el vínculo **Ver información** en la página y se abrirá una nueva pestaña en el explorador que muestra el artículo de conocimiento específico. En cualquier momento, puede revisar todos los artículos de conocimiento sobre criterios de mantenimiento incluidos con Azure Monitor para la característica de mantenimiento de máquinas virtuales [aquí](https://docs.microsoft.com/azure/monitoring/infrastructure-health/).
  
### <a name="state-changes"></a>Cambios de estado

La columna del extremo derecho de la página Diagnóstico de mantenimiento es **Cambios de estado**. Enumera todos los cambios de estado asociados con los criterios de mantenimiento que se han seleccionado en la sección **Criterios de mantenimiento** o el cambio de estado de la VM si se ha seleccionado una VM en la columna **Modelo de componente** o **Criterios de mantenimiento** de la tabla. 

![Ejemplo de los cambios de estado presentados en la página Diagnóstico de mantenimiento](./media/vminsights-health/health-diagnostics-page-statechanges.png)

Esta sección está compuesta por el estado de los criterios de mantenimiento y el tiempo asociado ordenados por el estado más reciente.   

### <a name="association-of-component-model-health-criteria-and-state-change-columns"></a>Asociación del modelo de componentes, los criterios de estado y estado Cambiar columnas 

Las tres columnas se entrelazan entre sí. Cuando selecciona una instancia detectada en la sección **Modelo de componente**, la sección **Criterios de mantenimiento** se filtra a esa vista de componente y, en consecuencia, la sección del **cambio de estado** se actualiza en función de los criterios de mantenimiento seleccionados. 

![Ejemplo de la selección de la instancia supervisada y los resultados](./media/vminsights-health/health-diagnostics-vm-example-01.png)

En el ejemplo anterior, cuando se selecciona **Disk - 1 D:** (Disco - 1 D:), se filtra el árbol de los criterios de mantenimiento como **Disk - 1D:** (Disco - 1 D:). En la columna **Cambio de estado** se muestra el cambio de estado en función de la disponibilidad de **Disk - 1 D:** (Disco - 1 D:). 

Para ver el estado de mantenimiento actualizado, puede actualizar la página de diagnóstico de mantenimiento haciendo clic en el vínculo **Actualizar**.  Si hay una actualización en el estado de mantenimiento de los criterios de mantenimiento basada en el intervalo de sondeo predefinido, esta tarea permite evitar la espera y refleja el estado de mantenimiento más reciente.  El **Estado de criterios de mantenimiento** es un filtro que le permite definir el ámbito de los resultados según el estado de mantenimiento seleccionado: *Correcto*, *Advertencia*, *Crítico*, *Desconocido* y *Todos*.  La hora de la **última actualización** de la esquina superior derecha representa la última vez que se actualizó la página de diagnóstico de mantenimiento.  

## <a name="alerts"></a>Alertas

La característica de mantenimiento de Azure Monitor para máquinas virtuales se integra con [Alertas de Azure](../../azure-monitor/platform/alerts-overview.md) y genera una alerta cuando los criterios de mantenimiento predefinidos cambian de un estado correcto a uno incorrecto y dicha condición se detecta. Las alertas se clasifican según la gravedad: de 0 a 4, donde el 0 representa el nivel de gravedad más alto. 

Las alertas no están asociadas con un grupo de acciones para avisarle cuando se ha desencadenado la alerta. El propietario de la suscripción debe configurar las notificaciones de los pasos [más adelante en esta sección](#configure-alerts).   

El número total de alertas de mantenimiento de la VM clasificadas según la gravedad está disponible en el panel **Mantenimiento** de la sección **Alertas**. Cuando selecciona el número total de alertas o el número correspondiente a un nivel de gravedad, la página **Alertas** se abre y muestra todas las alertas que coinciden con su selección.  Por ejemplo, si ha seleccionado la fila correspondiente a **Sev level 1** (Nivel de gravedad), verá lo siguiente:

![Ejemplo de todas las alertas de nivel de gravedad 1](./media/vminsights-health/vminsights-sev1-alerts-01.png)

El ámbito de la página **Alertas** no se limita a mostrar las alertas que coinciden con la selección, sino que también se filtran por **tipo de recurso** para mostrar solo las alertas de mantenimiento generadas por el recurso de la máquina virtual.  Se refleja en la lista de alertas, en la columna **recurso de destino**, donde muestra la máquina virtual de Azure se produjo la alerta para cuando se cumplió la condición de mal estado de los criterios de mantenimiento determinado.  

Las alertas de otros tipos de recursos o servicios no están diseñadas para incluirse en esta vista, como las alertas del registro basan en consultas de registro o alertas de métrica, que normalmente podría ver desde el valor predeterminado de Azure Monitor [todas las alertas](../../azure-monitor/platform/alerts-overview.md#all-alerts-page) página. 

Para filtrar esta vista, seleccione valores en los menús desplegables que aparecen en la parte superior de la página.

|columna |DESCRIPCIÓN | 
|-------|------------| 
|Subscription |Seleccione una suscripción de Azure. Solo se incluyen las alertas de la suscripción seleccionada en la vista. | 
|Grupo de recursos |Seleccione un solo grupo de recursos. Solo las alertas con destinos en el grupo de recursos seleccionado se incluyen en la vista. | 
|Tipo de recurso |Seleccione uno o varios tipos de recurso. De manare predeterminada, se seleccionan las alertas de las **máquinas virtuales** de destino y se incluyen en esta vista. Esta columna solo está disponible tras especificar un grupo de recursos. | 
|Recurso |Seleccione un recurso. Solo las alertas con ese recurso como destino se incluyen en la vista. Esta columna solo está disponible tras especificar un tipo de recurso. | 
|Severity |Seleccione una nivel de gravedad de la alerta o seleccione *Todas* para incluir las alertas de todos los niveles de gravedad. | 
|Condición del monitor |Seleccione una condición del monitor para filtrar las alertas en función de si el sistema las ha *desencadenado* o *resuelto*, en el caso de que la condición ya no esté activa. O bien seleccione *Todas* para incluir alertas de todas las condiciones. | 
|Estado de alerta |Seleccione un estado de alerta (*Nueva*, *Confirmación* o *Cerrada*), o seleccione *Todos* para incluir alertas de todos los estados. | 
|Supervisar servicio |Seleccione un servicio o seleccione *Todo* para incluir todos los servicios. Solo las alertas de *VM Insights* son compatibles con esta característica.| 
|Intervalo de tiempo| Solo las alertas activadas dentro del período de tiempo seleccionado se incluyen en la vista. Los valores compatibles son Última hora, Últimas 24 horas, Últimos 7 días y Últimos 30 días. | 

La página **Detalles de la alerta** se muestra cuando se selecciona una alerta y proporciona detalles de la alerta, lo que le permite cambiar su estado. Para aprender a administrar alertas, consulte [Creación, visualización y administración de alertas mediante Azure Monitor](../../azure-monitor/platform/alerts-metric.md).  

>[!NOTE]
>Actualmente no se admite la creación de nuevas alertas basadas en criterios de mantenimiento ni la modificación de las reglas de alerta de mantenimiento existentes en Azure Monitor desde el portal.  
>

![Panel Detalles de la alerta de una alerta seleccionada](./media/vminsights-health/alert-details-pane-01.png)

El estado de la alerta también se puede cambiar para una o varias alertas. Para ello, debe seleccionarlas y, luego, seleccionar **Cambiar estado** en la página **Todas las alertas**, en la esquina superior izquierda. En el panel **Cambiar el estado de alerta**, seleccione uno de los estados, agregue una descripción del cambio en el campo **Comentario** y, a continuación, haga clic en **Aceptar** para confirmar los cambios. Mientras se verifica la información y se aplican los cambios, puede realizar un seguimiento de su progreso en **Notificaciones** en el menú.  

### <a name="configure-alerts"></a>Configurar alertas
Ciertos administración de alertas de las tareas no pueden administrarse desde el portal de Azure y tienen que realizarse utilizando el [API de REST de Azure Monitor](https://docs.microsoft.com/rest/api/monitor/microsoft.workloadmonitor/components). Concretamente:

- Habilitar o deshabilitar una alerta para los criterios de estado 
- Configurar notificaciones para alertas de los criterios de estado 

Está usando el enfoque usado en cada ejemplo [ARMClient](https://github.com/projectkudu/armclient) en el equipo de Windows. Si no está familiarizado con este método, consulte [ARMClient utilizando](../platform/rest-api-walkthrough.md#use-armclient).  

#### <a name="enable-or-disable-alert-rule"></a>Habilitar o deshabilitar la regla de alerta

Para habilitar o deshabilitar una alerta para un criterio de mantenimiento específico, la propiedad de los criterios de mantenimiento *alertGeneration* debe modificarse con un valor de uno de ellos **deshabilitado** o **habilitado**. Para identificar el *monitorId* de criterios de un estado determinado, en el ejemplo siguiente, se mostrará cómo realizar una consulta para que el valor de los criterios **discológico\media en segundos por transferencia del disco**.

1. En una ventana de terminal, escriba **inicio de sesión de armclient.exe**. Si lo hace, le pedirá que inicie sesión en Azure.

2. Escriba el siguiente comando para recuperar todos los criterios del estado activo en una máquina virtual específica e identificar el valor de *monitorId* propiedad. 

    ```
    armclient GET "subscriptions/subscriptionId/resourceGroups/resourcegroupName/providers/Microsoft.Compute/virtualMachines/vmName/providers/Microsoft.WorkloadMonitor/monitors?api-version=2018-08-31-preview”
    ```

    El ejemplo siguiente muestra la salida del comando. Tome nota del valor de *MonitorId*. Este valor es necesario para el paso siguiente, donde se debe especificar el identificador de los criterios de estado y modificar su propiedad para crear una alerta.

    ```
    "id": "/subscriptions/a7f23fdb-e626-4f95-89aa-3a360a90861e/resourcegroups/Lab/providers/Microsoft.Compute/virtualMachines/SVR01/providers/Microsoft.WorkloadMonitor/monitors/ComponentTypeId='LogicalDisk',MonitorId='Microsoft_LogicalDisk_AvgDiskSecPerRead'",
      "name": "ComponentTypeId='LogicalDisk',MonitorId='Microsoft_LogicalDisk_AvgDiskSecPerRead'",
      "type": "Microsoft.WorkloadMonitor/virtualMachines/monitors"
    },
    {
      "properties": {
        "description": "Monitor the performance counter LogicalDisk\\Avg Disk Sec Per Transfer",
        "monitorId": "Microsoft_LogicalDisk_AvgDiskSecPerTransfer",
        "monitorName": "Microsoft.LogicalDisk.AvgDiskSecPerTransfer",
        "monitorDisplayName": "Average Logical Disk Seconds Per Transfer",
        "parentMonitorName": null,
        "parentMonitorDisplayName": null,
        "monitorType": "Unit",
        "monitorCategory": "PerformanceHealth",
        "componentTypeId": "LogicalDisk",
        "componentTypeName": "LogicalDisk",
        "componentTypeDisplayName": "Logical Disk",
        "monitorState": "Enabled",
        "criteria": [
          {
            "healthState": "Warning",
            "comparisonOperator": "GreaterThan",
            "threshold": 0.1
          }
        ],
        "alertGeneration": "Enabled",
        "frequency": 1,
        "lookbackDuration": 17,
        "documentationURL": "https://aka.ms/Ahcs1r",
        "configurable": true,
        "signalType": "Metrics",
        "signalName": "VMHealth_Avg. Logical Disk sec/Transfer"
      },
      "etag": null,
    ```

3. Escriba el siguiente comando para modificar el *alertGeneration* propiedad.

    ```
    armclient patch subscriptions/subscriptionId/resourceGroups/resourcegroupName/providers/Microsoft.Compute/virtualMachines/vmName/providers/Microsoft.WorkloadMonitor/monitors/Microsoft_LogicalDisk_AvgDiskSecPerTransfer?api-version=2018-08-31-preview "{'properties':{'alertGeneration':'Disabled'}}"
    ```   

4. Escriba el comando GET que se usa en el paso 2 para comprobar el valor de la propiedad se establece en **deshabilitado**.  

#### <a name="associate-action-group-with-health-criteria"></a>Asociar el grupo de acciones con los criterios de estado

Azure Monitor de estado de las máquinas virtuales es compatible con notificaciones de SMS y correo electrónico cuando se generan alertas cuando los criterios de estado pasa a ser incorrecto. Para configurar las notificaciones, debe tener en cuenta el nombre del grupo de acción que está configurado para enviar notificaciones de correo electrónico o SMS. 

>[!NOTE]
>Esta acción debe realizarse en cada máquina virtual que se supervisan que desea recibir una notificación.

1. En una ventana de terminal, escriba **inicio de sesión de armclient.exe**. Si lo hace, le pedirá que inicie sesión en Azure.

2. Escriba el siguiente comando para asociar un grupo de acciones a reglas de alerta.
 
    ```
    $payload = "{'properties':{'ActionGroupResourceIds':['/subscriptions/subscriptionId/resourceGroups/resourcegroupName/providers/microsoft.insights/actionGroups/actiongroupName']}}" 
    armclient PUT https://management.azure.com/subscriptions/subscriptionId/resourceGroups/resourcegroupName/providers/Microsoft.Compute/virtualMachines/vmName/providers/Microsoft.WorkloadMonitor/notificationSettings/default?api-version=2018-08-31-preview $payload
    ```

3. Para comprobar el valor de la propiedad **actionGroupResourceIds** correctamente actualizado, escriba el siguiente comando.

    ```
    armclient GET "subscriptions/subscriptionName/resourceGroups/resourcegroupName/providers/Microsoft.Compute/virtualMachines/vmName/providers/Microsoft.WorkloadMonitor/notificationSettings?api-version=2018-08-31-preview"
    ```

    La salida debe ser similar a la siguiente:
    
    ```
    {
      "value": [
        {
          "properties": {
            "actionGroupResourceIds": [
              "/subscriptions/a7f23fdb-e626-4f95-89aa-3a360a90861e/resourceGroups/Lab/providers/microsoft.insights/actionGroups/Lab-IT%20Ops%20Notify"
            ]
          },
          "etag": null,
          "id": "/subscriptions/a7f23fdb-e626-4f95-89aa-3a360a90861e/resourcegroups/Lab/providers/Microsoft.Compute/virtualMachines/SVR01/providers/Microsoft.WorkloadMonitor/notificationSettings/default",
          "name": "notificationSettings/default",
          "type": "Microsoft.WorkloadMonitor/virtualMachines/notificationSettings"
        }
      ],
      "nextLink": null
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

Para identificar cuellos de botella y el uso general con el rendimiento de sus VM, consulte el artículo sobre cómo [ver el rendimiento de la VM de Azure](vminsights-performance.md), o bien, para ver las dependencias de la aplicación detectadas, consulte cómo [ver el Azure Monitor para la asignación de VM](vminsights-maps.md). 
