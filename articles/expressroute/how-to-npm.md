---
title: 'Azure ExpressRoute: Configuración de NPM para circuitos'
description: Configure la supervisión de red basada en la nube (NPM) de circuitos Azure ExpressRoute. Aborda la supervisión a través de emparejamiento privado de ExpressRoute y emparejamiento de Microsoft.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 01/25/2019
ms.author: duau
ms.openlocfilehash: 0783675450bb6ddf2de7d012a6870f5e4c90408d
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393685"
---
# <a name="configure-network-performance-monitor-for-expressroute"></a>Configuración de Network Performance Monitor para ExpressRoute

Este artículo lo ayuda a configurar una extensión de Network Performance Monitor para supervisar ExpressRoute. Network Performance Monitor es una solución de supervisión de red en la nube que supervisa la conectividad entre las implementaciones de nube de Azure y las ubicaciones locales (sucursales, etcétera). NPM forma parte de los registros de Azure Monitor. NPM ofrece una extensión para ExpressRoute que le permite supervisar el rendimiento de la red a través de circuitos ExpressRoute configurados para usar el emparejamiento privado o emparejamiento de Microsoft. Cuando configura Network Performance Monitor para ExpressRoute, puede detectar problemas de red para identificar y eliminar. Este servicio también está disponible para Azure Government Cloud.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

Puede:

* Supervisar la latencia y pérdida a través de varias redes virtuales y establecer alertas

* Supervisar todas las rutas de acceso (incluidas las rutas de acceso redundantes) en la red

* Solucionar problemas de red transitorios y a un momento dato que son difíciles de replicar

* Ayudar a determinar un segmento específico de la red que es responsable de la degradación del rendimiento

* Obtener el rendimiento por red virtual (si tiene agentes instalados en cada red virtual)

* Ver el estado del sistema ExpressRoute desde un momento dado anterior

## <a name="workflow"></a><a name="workflow"></a>Flujo de trabajo

Se instalan agentes de supervisión en varios servidores, tanto en el entorno local como en Azure. Los agentes se comunican entre sí, pero no envían datos, envían paquetes de protocolo de enlace TCP. La comunicación entre los agentes permite a Azure asignar la topología de red y la ruta de acceso que puede tomar el tráfico.

1. Cree un área de trabajo de NPM. Es lo mismo que un área de trabajo de Log Analytics.
2. Instalación y configuración de agentes de software. (Si solo desea supervisar el Emparejamiento de Microsoft, no es necesario instalar ni configurar agentes de software): 
    * Instale agentes de supervisión en los servidores locales y las máquinas virtuales de Azure (para emparejamiento privado).
    * Configure las opciones en los servidores de agente de supervisión para permitir que estos agentes se comuniquen. (Por ejemplo, abrir puertos del firewall).
3. Configure las reglas de grupo de seguridad de red (NSG) para permitir que el agente de supervisión instalado en las máquinas virtuales de Azure se comuniquen con agentes de supervisión locales.
4. Configure la supervisión de la detección automática y administre las redes que están visibles en Network Performance Monitor.

Si ya usa Network Performance Monitor para supervisar otros servicios u objetos y ya dispone de un área de trabajo en una de las regiones admitidas, puede omitir el paso 1 y 2 y empezar la configuración en el paso 3.

## <a name="step-1-create-a-workspace"></a><a name="configure"></a>Paso 1: Crear un área de trabajo

Cree un área de trabajo en la suscripción que tiene las redes virtuales vinculadas a los circuitos ExpressRoute.

1. En [Azure Portal](https://portal.azure.com), seleccione la suscripción que tiene las redes virtuales emparejadas al circuito de ExpressRoute. A continuación, busque "Network Performance Monitor" en la lista de servicios de **Marketplace**. Cuando se devuelva el resultado, haga clic para abrir la página **Network Performance Monitor**.

   >[!NOTE]
   >Puede crear una nueva área de trabajo o usar una existente. Si desea usar un área de trabajo existente, debe asegurarse de que se haya migrado el área de trabajo al nuevo lenguaje de consulta. [Más información...](https://docs.microsoft.com/azure/log-analytics/log-analytics-log-search-upgrade)
   >

   ![portal](./media/how-to-npm/3.png)<br><br>
2. En la parte inferior de la página **Network Performance Monitor** principal, haga clic en **Crear** para abrir la página **Network Performance Monitor - Crear nueva solución**. Haga clic en **Área de trabajo de Log Analytics - Seleccionar un área de trabajo** para abrir la página Áreas de trabajo. Haga clic en **+Crear área de trabajo nueva** para abrir la página Área de trabajo.
3. En la página **Área de trabajo de Log Analytics**, seleccione **Crear nuevo** y configure las siguientes opciones:

   * Área de trabajo de Log Analytics: escriba un nombre para el área de trabajo.
   * Suscripción : si tiene varias suscripciones, elija la que desea asociar con el área de trabajo nueva.
   * Grupo de recursos: cree un grupo de recursos o use uno existente.
   * Ubicación: esta ubicación se utiliza para especificar la ubicación de la cuenta de almacenamiento que se usa para los registros de conexión del agente.
   * Plan de tarifa: seleccione el plan de tarifa.
  
     >[!NOTE]
     >El circuito ExpressRoute puede estar en cualquier lugar del mundo. No tiene que estar en la misma región que el área de trabajo.
     >
  
     ![área de trabajo](./media/how-to-npm/4.png)<br><br>
4. Haga clic en **Aceptar** para guardar e implementar la plantilla de configuración. Una vez validada la plantilla, haga clic en **Crear** para implementar el área de trabajo.
5. Una vez implementado el área de trabajo, vaya al recurso **NetworkMonitoring(name)** que ha creado. Valide la configuración y haga clic en **La solución necesita una configuración adicional**.

   ![configuración adicional](./media/how-to-npm/5.png)

## <a name="step-2-install-and-configure-agents"></a><a name="agents"></a>Paso 2: Instalación y configuración de agentes

### <a name="21-download-the-agent-setup-file"></a><a name="download"></a>2.1: Descarga del archivo de instalación del agente

1. Vaya a la pestaña **Configuración común** de la página **Configuración de Monitor de rendimiento de red** del recurso. Haga clic en el agente que se corresponda con el procesador de su servidor en la sección **Instalar los agentes de Log Analytics** y descargue el archivo de instalación.
2. Después, copie los valores de **Id. de área de trabajo** y **Clave principal** en el Bloc de notas.
3. En la sección **Configuración de los agentes de Log Analytics para la supervisión mediante el protocolo TCP**, descargue el script de Powershell. El script de PowerShell le ayuda a abrir el puerto de firewall pertinente para las transacciones TCP.

   ![Script de PowerShell](./media/how-to-npm/7.png)

### <a name="22-install-a-monitoring-agent-on-each-monitoring-server-on-each-vnet-that-you-want-to-monitor"></a><a name="installagent"></a>2.2: Instalación de un agente de supervisión en cada servidor de supervisión (en cada red virtual que quiera supervisar)

Le recomendamos que instale al menos dos agentes en cada lado de la conexión de ExpressRoute para obtener redundancia (por ejemplo, en el entorno local y en las redes virtuales de Azure). El agente debe instalarse en Windows Server (2008 SP1 o posterior). No se admite la supervisión de circuitos ExpressRoute mediante el sistema operativo de escritorio Windows y el sistema operativo Linux. Use los pasos siguientes para instalar agentes:
   
  >[!NOTE]
  >Puede que los agentes que inserta SCOM (lo que incluye [MMA](https://technet.microsoft.com/library/dn465154(v=sc.12).aspx)) no sean capaces de detectar su ubicación de forma coherente si están hospedados en Azure. No se recomienda usar estos agentes en redes virtuales de Azure para supervisar ExpressRoute.
  >

1. Ejecute **el programa de instalación** para instalar el agente en cada servidor que va a utilizar para la supervisión de ExpressRoute. El servidor que se usa para la supervisión puede ser una máquina virtual o encontrase en un entorno local, y debe tener acceso a Internet. Debe instalar al menos un agente local y uno en cada segmento de red que desea supervisar en Azure.
2. En la página **principal**, haga clic en **Siguiente**.
3. En la página **Términos de licencia**, lea la licencia y haga clic en **Acepto**.
4. En la página **Carpeta de destino**, cambie o mantenga la carpeta de instalación predeterminada y haga clic en **Siguiente**.
5. En la página **Opciones de instalación del agente**, puede decidir si quiere conectar el agente a los registros de Azure Monitor o a Operations Manager. O bien, puede dejar las opciones en blanco si desea configurar el agente más adelante. Tras efectuar la selección, haga clic en **Siguiente**.

   * Si decide conectar el agente a **Azure Log Analytics**, pegue el **identificador del área de trabajo** y la **clave del área de trabajo** (clave principal) que copió en el Bloc de notas en la sección anterior. A continuación, haga clic en **Siguiente**.

     ![Identificador y clave](./media/how-to-npm/8.png)
   * Si decide conectar el agente a **Operations Manager**, en la página **Configuración del grupo de administración**, escriba el **nombre del grupo de administración**, el **servidor de administración** y el **puerto del servidor de administración**. A continuación, haga clic en **Siguiente**.

     ![Operations Manager](./media/how-to-npm/9.png)
   * En la página **Cuenta de acción del agente**, seleccione la cuenta de **sistema local** o un **dominio o cuenta del equipo local**. A continuación, haga clic en **Siguiente**.

     ![Cuenta](./media/how-to-npm/10.png)
6. En la página **Preparado para instalar**, revise las opciones seleccionadas y haga clic en **Instalar**.
7. En la página **La configuración finalizó correctamente**, haga clic en **Finalizar**.
8. Una vez completado el proceso, Microsoft Monitoring Agent aparece en el Panel de control. Ahí puede revisar la configuración y comprobar que el agente está conectado a los registros de Azure Monitor. Cuando se conecta, el agente muestra un mensaje que indica: **The Microsoft Monitoring Agent has successfully connected to the Microsoft Operations Management Suite service** (Microsoft Monitoring Agent se ha conectado correctamente al servicio Microsoft Operations Management Suite).

9. Repita este procedimiento con cada red virtual que quiera que se supervise.

### <a name="23-configure-proxy-settings-optional"></a><a name="proxy"></a>2.3: Configuración de los valores de proxy (opcional)

Si está utilizando un proxy web para acceder a Internet, siga estos pasos para configurar el proxy para Microsoft Monitoring Agent. Realice estos pasos para cada servidor. Si tiene muchos servidores que necesite configurar, le resultará más fácil usar un script para automatizar este proceso. Si es así, consulte [Para configurar el proxy para Microsoft Monitoring Agent mediante un script](../log-analytics/log-analytics-windows-agent.md).

Para configurar el proxy para Microsoft Monitoring Agent mediante el Panel de control:

1. Abra el **Panel de control**.
2. Abra **Microsoft Monitoring Agent**.
3. Haga clic en la pestaña **Configuración de proxy**.
4. Seleccione **Usar un servidor proxy** y escriba la dirección URL y el número de puerto, si uno es necesario. Si el servidor proxy requiere autenticación, escriba el nombre de usuario y la contraseña para acceder al servidor proxy.

   ![proxy](./media/how-to-npm/11.png)

### <a name="24-verify-agent-connectivity"></a><a name="verifyagent"></a>2.4: Comprobación de la conectividad del agente

Puede comprobar fácilmente si los agentes se están comunicando.

1. En un servidor con el agente de supervisión, abra el **Panel de Control**.
2. Abra **Microsoft Monitoring Agent**.
3. Haga clic en la pestaña **Azure Log Analytics**.
4. En la columna **Estado**, verá que el agente se conectó correctamente a los registros de Azure Monitor.

   ![status](./media/how-to-npm/12.png)

### <a name="25-open-the-firewall-ports-on-the-monitoring-agent-servers"></a><a name="firewall"></a>2.5: Apertura de los puertos de firewall en los servidores de agente de supervisión

Para usar el protocolo TCP, debe abrir los puertos de firewall a fin de asegurarse de que los agentes de supervisión puedan comunicarse.

Puede ejecutar un script de PowerShell para crear las claves del Registro que son necesarias para Network Performance Monitor. Este script también crea reglas del Firewall de Windows para permitir que los agentes de supervisión creen conexiones TCP entre sí. Las claves del Registro que crea el script especifican si se deben escribir los registros de depuración y la ruta del archivo de registro. Asimismo, define el puerto TCP del agente empleado para establecer la comunicación. El script establece automáticamente los valores de estas claves. No debería cambiar manualmente estas claves.

El puerto 8084 se abre de forma predeterminada. Puede utilizar un puerto personalizado especificando el parámetro "portNumber" en el script. Sin embargo, si lo hace, debe especificar el mismo puerto para todos los servidores en los que se ejecuta el script.

>[!NOTE]
>El script de PowerShell "EnableRules" solo configura reglas del firewall de Windows en el servidor donde se ejecuta. Si tiene un firewall de red, debe asegurarse de que permite el tráfico destinado al puerto TCP que esté utilizando Monitor de rendimiento de red.
>
>

En los servidores de agente, abra una ventana de PowerShell con privilegios administrativos. Ejecute el script [EnableRules](https://aka.ms/npmpowershellscript) de PowerShell (que descargó anteriormente). No utilice ningún parámetro.

![PowerShell_Script](./media/how-to-npm/script.png)

## <a name="step-3-configure-network-security-group-rules"></a><a name="opennsg"></a>Paso 3: Configuración de las reglas del grupo de seguridad de red

Para supervisar los servidores de agente que se encuentran en Azure, debe configurar las reglas del grupo de seguridad de red (NSG) para permitir el tráfico TCP en un puerto utilizado por Network Performance Monitor para las transacciones sintéticas. El puerto predeterminado es 8084. Esto permite que un agente de supervisión instalado en una VM de Azure se comunique con un agente de supervisión local.

Para más información, consulte [Grupos de seguridad de red](../virtual-network/virtual-networks-create-nsg-arm-portal.md).

>[!NOTE]
>Asegúrese de que ha instalado los agentes (tanto el agente de servidor local como el agente de servidor de Azure) y que ha ejecutado el script de PowerShell antes de continuar con este paso.
>

## <a name="step-4-discover-peering-connections"></a><a name="setupmonitor"></a>Paso 4: Detección de las conexiones de emparejamiento

1. Navegue hasta el icono de información general de Network Performance Monitor, vaya a la página **Todos los recursos** y haga clic en el área de trabajo de NPM que se encuentra en la lista blanca.

   ![área de trabajo de Network Performance Monitor](./media/how-to-npm/npm.png)
2. Haga clic en el icono de introducción de **Network Performance Monitor** para que aparezca el panel. El panel contiene una página de ExpressRoute, que muestra que ExpressRoute está en un estado "no configurado". Haga clic en **Configuración de características** para abrir la página de configuración de Network Performance Monitor.

   ![configuración de características](./media/how-to-npm/npm2.png)
3. En la página de configuración, navegue a la pestaña "Emparejamientos de ExpressRoute", ubicada en el panel del lado izquierdo. A continuación, haga clic en **Detectar ahora**.

   ![detectar](./media/how-to-npm/13.png)
4. Cuando se completa la detección, verá una lista que contiene los siguientes elementos:
   * Todas las conexiones de emparejamiento de Microsoft en los circuitos ExpressRoute que están asociados con esta suscripción.
   * Todas las conexiones de emparejamiento privadas que se conectan a las redes virtuales asociadas con esta suscripción.
            
## <a name="step-5-configure-monitors"></a><a name="configmonitor"></a>Paso 5: Configuración de los monitores

En esta sección, configurará los monitores. Siga los pasos para el tipo de emparejamiento que desea supervisar: **emparejamiento privado** o **emparejamiento de Microsoft**.

### <a name="private-peering"></a>Emparejamiento privado

Para el emparejamiento privado, cuando se complete la detección, verá las reglas para el **Nombre del circuito** y el **Nombre de red virtual** exclusivos. Inicialmente, estas reglas están deshabilitadas.

![reglas](./media/how-to-npm/14.png)

1. Marque la casilla **Supervisar este emparejamiento**.
2. Seleccione la casilla **Habilitar el seguimiento de mantenimiento para este emparejamiento**.
3. Elija las condiciones de supervisión. Puede establecer umbrales personalizados para generar eventos de estado escribiendo valores de umbral. Siempre que el valor de la condición supere el umbral seleccionado para el par de red/subred seleccionado, se generará un evento de estado.
4. Haga clic en **Agregar agentes** en AGENTES LOCALES para agregar los servidores locales desde los cuales supervisar la conexión de emparejamiento privado. Asegúrese de solo elegir los agentes que tengan conectividad con el punto de conexión de servicio de Microsoft que especificó en la sección del paso 2. Los agentes locales deben ser capaces de alcanzar el punto de conexión mediante la conexión de ExpressRoute.
5. Guarde la configuración
6. Después de habilitar las reglas y de seleccionar los valores y los agentes que desea supervisar, hay una espera entre 30 y 60 minutos aproximadamente para que los valores que se va a empezar a llenarse y los iconos de **Supervisión de ExpressRoute** están disponibles.

### <a name="microsoft-peering"></a>Emparejamiento de Microsoft

Para el emparejamiento de Microsoft, haga clic en las conexiones de emparejamiento de Microsoft que desea supervisar y configure las opciones.

1. Marque la casilla **Supervisar este emparejamiento**. 
2. (Opcional) Puede cambiar el punto de conexión de servicio de Microsoft de destino. De manera predeterminada, NPM elige un punto de conexión de servicio de Microsoft como destino. NPM supervisa la conectividad de los servidores locales a este punto de conexión de destino a través de ExpressRoute. 
    * Para cambiar este punto de conexión de destino, haga clic en el vínculo **(editar)** en **Destino:** y seleccione otro punto de conexión de servicio de Microsoft de destino en la lista de direcciones URL.
      ![editar destino](./media/how-to-npm/edit_target.png)<br>

    * Puede usar una dirección URL o IP personalizada. Esta opción es especialmente pertinente si usa el emparejamiento de Microsoft para establecer una conexión con los servicios de PaaS de Azure, como Azure Storage, bases de datos SQL y sitios web que se ofrecen en direcciones IP públicas. Para ello, haga clic en el vínculo **(Usar la dirección IP o URL personalizada en su lugar)** en la parte inferior de la lista de direcciones URL, a continuación, escriba el punto de conexión público de su servicio de PaaS de Azure que está conectado a través del emparejamiento de Microsoft de ExpressRoute.
    ![URL personalizada](./media/how-to-npm/custom_url.png)<br>

    * Si usa esta configuración opcional, asegúrese de que solo el punto de conexión de servicio de Microsoft esté seleccionado aquí. El punto de conexión debe estar conectado a ExpressRoute, y los agentes locales deben poder acceder a él.
3. Seleccione la casilla **Habilitar el seguimiento de mantenimiento para este emparejamiento**.
4. Elija las condiciones de supervisión. Puede establecer umbrales personalizados para generar eventos de estado escribiendo valores de umbral. Siempre que el valor de la condición supere el umbral seleccionado para el par de red/subred seleccionado, se generará un evento de estado.
5. Haga clic en **Agregar agentes** en AGENTES LOCALES para agregar los servidores locales desde los cuales supervisar la conexión de emparejamiento de Microsoft. Asegúrese de solo elegir los agentes que tengan conectividad con los puntos de conexión de servicio de Microsoft que especificó en la sección del paso 2. Los agentes locales deben ser capaces de alcanzar el punto de conexión mediante la conexión de ExpressRoute.
6. Guarde la configuración
7. Después de habilitar las reglas y de seleccionar los valores y los agentes que desea supervisar, hay una espera entre 30 y 60 minutos aproximadamente para que los valores que se va a empezar a llenarse y los iconos de **Supervisión de ExpressRoute** están disponibles.

## <a name="step-6-view-monitoring-tiles"></a><a name="explore"></a>Paso 6: Visualización de los iconos de supervisión

Cuando se vean los iconos de supervisión, los circuitos ExpressRoute y los recursos de conexión van a estar supervisados por Network Performance Monitor. Puede hacer clic en el icono Emparejamiento de Microsoft para explorar en profundidad el estado de las conexiones de Emparejamiento de Microsoft.

![iconos de supervisión](./media/how-to-npm/15.png)

### <a name="network-performance-monitor-page"></a><a name="dashboard"></a>Página Network Performance Monitor

La página Network Performance Monitor contiene una página de ExpressRoute que muestra una introducción del estado de emparejamientos y circuitos de ExpressRoute.

![Panel](./media/how-to-npm/dashboard.png)

### <a name="list-of-circuits"></a><a name="circuits"></a>Lista de circuitos

Para ver una lista de todos los circuitos ExpressRoute supervisados, haga clic en el icono **Circuitos ExpressRoute**. Puede seleccionar un circuito y ver su estado de mantenimiento, gráficos de tendencias de pérdida de paquetes, uso de ancho de banda y latencia. Los gráficos son interactivos. Puede seleccionar un período de tiempo personalizado para trazar los gráficos. Puede arrastrar el mouse sobre un área del gráfico para acercar y ver los puntos de datos con precisión.

![circuit_list](./media/how-to-npm/circuits.png)

#### <a name="trend-of-loss-latency-and-throughput"></a><a name="trend"></a>Tendencia de rendimiento, latencia y pérdida

Los gráficos de ancho de banda, latencia y pérdida son interactivos. Puede acercar cualquier sección de estos gráficos con los controles de mouse. También puede ver los datos de ancho de banda, latencia y pérdida para otros intervalos; para ello, haga clic en **Fecha/hora**, que se encuentra debajo del botón Acciones en la parte superior izquierda.

![tendencia](./media/how-to-npm/16.png)

### <a name="peerings-list"></a><a name="peerings"></a>Lista de emparejamientos

Para ver una lista de todas las conexiones a redes virtuales a través de emparejamientos privados, haga clic en el icono **Emparejamientos privados** en el panel. Aquí, puede seleccionar una conexión de red virtual y ver su estado de mantenimiento, gráficos de tendencias de pérdida de paquetes, uso de ancho de banda y latencia.

![lista de circuitos](./media/how-to-npm/peerings.png)

### <a name="nodes-view"></a><a name="nodes"></a>Vista de nodos

Para ver la lista de todos los vínculos entre los nodos locales y los puntos de conexión de servicio de Microsoft/VM de Azure para la conexión de emparejamiento de ExpressRoute seleccionada, haga clic en **Ver vínculos de nodo**. Puede ver el estado de mantenimiento de cada vínculo, así como la tendencia de pérdida y latencia asociada con ellos.

![vista de nodos](./media/how-to-npm/nodes.png)

### <a name="circuit-topology"></a><a name="topology"></a>Topología de circuito

Para ver la topología de circuito, haga clic en el icono **Topología**. Esto le llevará a la vista de topología del circuito o emparejamiento seleccionado. El diagrama de topología proporciona la latencia para cada segmento de la red. Cada salto de capa 3 se representa mediante un nodo del diagrama. Si se hace clic en un salto, se mostrarán más detalles sobre él.

Para aumentar el nivel de visibilidad para incluir saltos locales, mueva el control deslizante debajo de **Filtros**. Si se mueve el control deslizante hacia la izquierda o derecha, se aumenta o disminuye el número de saltos en el gráfico de topología. La latencia por cada segmento está visible, lo que permite un aislamiento más rápido de los segmentos de latencia elevada en la red.

![filters](./media/how-to-npm/topology.png)

#### <a name="detailed-topology-view-of-a-circuit"></a>Vista de topología detallada de un circuito

Esta vista muestra conexiones de red virtual.
![topología detallada](./media/how-to-npm/17.png)
