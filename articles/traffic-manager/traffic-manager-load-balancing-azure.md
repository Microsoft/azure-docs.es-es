---
title: Uso de servicios de equilibrio de carga en Azure | Microsoft Docs
description: 'En este tutorial se muestra cómo crear un escenario al usar la cartera de equilibrio de carga de Azure: Traffic Manager, Application Gateway y Load Balancer.'
services: traffic-manager
documentationcenter: ''
author: duongau
manager: kumudD
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/27/2016
ms.author: duau
ms.openlocfilehash: 431eaff9da95063648d3e80acb54be9cc5c25bc5
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89393075"
---
# <a name="using-load-balancing-services-in-azure"></a>Uso de servicios de equilibrio de carga en Azure

## <a name="introduction"></a>Introducción

Microsoft Azure proporciona varios servicios para administrar cómo se distribuye el tráfico de red y se equilibra la carga. Puede usar estos servicios de forma individual o combinar sus métodos, según sus necesidades, para crear la solución más óptima.

En este tutorial, definiremos primero el caso de uso de un cliente y veremos cómo se puede mejorar la solidez y el rendimiento mediante la siguiente cartera de equilibrio de carga de Azure: Traffic Manager, Application Gateway y Load Balancer. A continuación, se proporcionan instrucciones paso a paso para crear una implementación con redundancia geográfica que distribuya el tráfico a máquinas virtuales y ayude a administrar distintos tipos de solicitudes.

En un nivel conceptual, cada uno de estos servicios desempeña un rol distinto en la jerarquía de equilibrio de carga.

* **Traffic Manager** proporciona equilibrio de carga de DNS global. Examina las solicitudes de DNS entrantes y responde con un punto de conexión en buen estado y de conformidad con la directiva de enrutamiento que el cliente haya seleccionado. Las opciones de los métodos de enrutamiento son:
  * Enrutamiento de rendimiento para enviar al solicitante al punto de conexión más cercano en términos de latencia.
  * Enrutamiento de prioridad para dirigir todo el tráfico a un punto de conexión, con otros puntos de conexión como respaldo.
  * Enrutamiento round-robin ponderado, que distribuye el tráfico según la ponderación asignada a cada punto de conexión.
  * Enrutamiento basado en la geografía para distribuir el tráfico a los puntos de conexión de la aplicación según la ubicación geográfica del usuario.
  * Enrutamiento basado en la subred para distribuir el tráfico a los puntos de conexión de la aplicación según la subred (rango de direcciones IP) del usuario.
  * Enrutamiento de varios valores que le permite enviar las direcciones IP de más de un punto de conexión de la aplicación en una única respuesta de DNS.

  El cliente se conecta directamente al punto de conexión devuelto por Traffic Manager. Azure Traffic Manager detecta cuándo un punto de conexión está en mal estado y redirige a los clientes a otra instancia en buen estado. Consulte la [documentación de Azure Traffic Manager](traffic-manager-overview.md) para más información sobre el servicio.
* **Application Gateway** proporciona un controlador de entrega de aplicaciones (ADC) como servicio, así como diversas funcionalidades de equilibrio de carga de nivel 7 para la aplicación. Permite a los clientes optimizar la productividad de las granjas de servidores web traspasando la carga de la terminación TLS con mayor actividad de CPU a Application Gateway. Otras funcionalidades de enrutamiento de nivel 7 son la distribución round robin del tráfico entrante, la afinidad de sesiones basada en cookies, el enrutamiento basado en ruta de acceso de URL y la posibilidad de hospedar varios sitios web tras una única instancia de Application Gateway. Application Gateway puede configurarse como una puerta de enlace accesible desde Internet, una puerta de enlace solo para uso interno o una combinación de las dos. Application Gateway está completamente administrada por Azure, es escalable y tiene una elevada disponibilidad. Cuenta con un amplio conjunto de funcionalidades de diagnóstico y registro, lo que facilita su administración.
* **Load Balancer**, parte integral de la pila de Azure SDN, proporciona servicios de equilibrio de carga de capa 4 de alto rendimiento y baja latencia para todos los protocolos TCP y UDP. Administra conexiones entrantes y salientes. Puede configurar puntos de conexión con equilibrio de carga públicos e internos y definir reglas para asignar conexiones entrantes a destinos de grupo de back-end con opciones de sondeo de estado TCP y HTTP para administrar la disponibilidad del servicio.

## <a name="scenario"></a>Escenario

En este escenario de ejemplo, se usa un sitio web sencillo que sirve dos tipos de contenido: imágenes y páginas web representadas de forma dinámica. Este sitio web debe tener redundancia geográfica y debería atender a sus usuarios desde la ubicación que tengan más cercana (latencia más baja). El desarrollador de aplicaciones ha decidido que las direcciones URL que coincidan con el patrón /images/* se sirvan desde un grupo dedicado de máquinas virtuales distintas del resto de la granja de servidores web.

Además, el grupo de máquinas virtuales predeterminado que sirve el contenido dinámico debe comunicarse con una base de datos de back-end hospedada en un clúster de alta disponibilidad. Toda la implementación está configurada mediante Azure Resource Manager.

El uso de Traffic Manager, Application Gateway y Load Balancer permite que este sitio web logre estos objetivos de diseño:

* **Redundancia geográfica múltiple**: Si una región queda inactiva, Traffic Manager enruta el tráfico sin problemas a la siguiente región más adecuada sin intervención del propietario de la aplicación.
* **Menor latencia**: Dado que Traffic Manager dirige automáticamente al cliente a la región más cercana, este experimenta una menor latencia al solicitar los contenidos de la página web.
* **Escalabilidad independiente**: Como la carga de trabajo de la aplicación web está separada por el tipo de contenido, el propietario de la aplicación puede escalar las cargas de trabajo de solicitudes manteniendo la independencia entre ellas. Application Gateway garantiza que el tráfico se enrute a los grupos correctos según las reglas especificadas y el estado de la aplicación.
* **Equilibrio de carga interno**: Con Load Balancer frente al clúster de alta disponibilidad, solo se expone a la aplicación el punto de conexión en buen estado y activo correspondiente para una base de datos. Además, un administrador de base de datos puede optimizar la carga de trabajo si distribuye réplicas activas y pasivas en el clúster de forma independiente de la aplicación de front-end. Load Balancer entrega conexiones al clúster de alta disponibilidad y garantiza que las únicas bases de datos que reciban solicitudes de conexión sean las que se encuentren en buen estado.

En el siguiente diagrama se muestra la arquitectura de este escenario:

![Diagrama de la arquitectura de equilibrio de carga](./media/traffic-manager-load-balancing-azure/scenario-diagram.png)

> [!NOTE]
> Este ejemplo es solo una de las muchas configuraciones posibles de los servicios de equilibrio de carga que ofrece Azure. Traffic Manager, Application Gateway y Load Balancer se pueden combinar de la forma que mejor se adapte a sus necesidades de equilibrio de carga. Por ejemplo, si la descarga de TLS o el procesamiento de nivel 7 no son necesarios, se puede usar Load Balancer en lugar de Application Gateway.

## <a name="setting-up-the-load-balancing-stack"></a>Configuración de la pila de equilibrio de carga

### <a name="step-1-create-a-traffic-manager-profile"></a>Paso 1: Crear un perfil de Traffic Manager

1. En Azure Portal, haga clic en **Crear un recurso** > **Redes** > **Perfil de Traffic Manager** > **Crear**.
2. Escriba la información básica siguiente:

   * **Name**: Escriba un nombre de prefijo DNS para el perfil de Traffic Manager.
   * **Método de enrutamiento**: Seleccione la directiva del método de enrutamiento del tráfico. Para más información sobre los métodos de enrutamiento del tráfico, consulte [Métodos de enrutamiento de tráfico de Traffic Manager](traffic-manager-routing-methods.md).
   * **Suscripción**: Seleccione la suscripción que contiene el perfil.
   * **Grupo de recursos**: Seleccione el grupo de recursos que contiene el perfil. Puede ser un grupo de recursos nuevo o existente.
   * **Ubicación del grupo de recursos**: El servicio Traffic Manager es global y no está enlazado a una ubicación. Sin embargo, debe especificar una región para el grupo donde residen los metadatos asociados al perfil de Traffic Manager. Esta ubicación no afecta a la disponibilidad en tiempo de ejecución del perfil.

3. Haga clic en **Crear** para generar el perfil de Traffic Manager.

   ![Hoja de creación de perfil de Traffic Manager](./media/traffic-manager-load-balancing-azure/s1-create-tm-blade.png)

### <a name="step-2-create-the-application-gateways"></a>Paso 2: Crear las puertas de enlace de aplicaciones

1. En Azure Portal, vaya al panel izquierdo y haga clic en **Crear un recurso** > **Redes** > **Application Gateway**.
2. Rellene la información básica de la puerta de enlace de aplicaciones:

   * **Name**: Nombre de la puerta de enlace de aplicaciones.
   * **Tamaño de la SKU**: El tamaño de la puerta de enlace de aplicaciones, disponible como Pequeño, Mediano o Grande.
   * **Recuento de instancias**: El número de instancias, un valor de entre 2 y 10.
   * **Grupo de recursos**: El grupo de recursos que contiene la puerta de enlace de aplicaciones. Puede ser un grupo de recursos existente o uno nuevo.
   * **Ubicación**: La región de la puerta de enlace de aplicaciones, que es la misma ubicación en el grupo de recursos. La ubicación es importante ya que la red virtual y la dirección IP pública deben estar en la misma ubicación que la puerta de enlace.
3. Haga clic en **OK**.
4. Defina la red virtual, la subred, la IP de front-end y las configuraciones de agente de escucha para la puerta de enlace de aplicaciones. En este escenario, la dirección IP de front-end es **pública**, lo que permite que se agregue como punto de conexión al perfil de Traffic Manager más adelante.
5. Configure el agente de escucha con una de las siguientes opciones:
    * Si usa HTTP, no hay nada que configurar. Haga clic en **OK**.
    * Para usar HTTPS, se requiere configuración adicional. Consulte [Creación de una puerta de enlace de aplicaciones](../application-gateway/application-gateway-create-gateway-portal.md) y comience en el paso 9. Cuando se haya completado la configuración, haga clic en **Aceptar**.

#### <a name="configure-url-routing-for-application-gateways"></a>Configuración del enrutamiento de direcciones URL para puertas de enlace de aplicaciones

Cuando elige un grupo de back-end, una puerta de enlace de aplicaciones configurada con una regla basada en ruta de acceso usa un patrón de ruta de acceso de la dirección URL de solicitud junto con la distribución round robin. En este escenario, se agrega una regla basada en ruta de acceso para dirigir a cualquier dirección URL con "/images/\*" al grupo de servidores de imagen. Para más información sobre la configuración del enrutamiento basado en rutas de acceso de dirección URL para una puerta de enlace de aplicaciones, consulte [Create a path-based rule for an application gateway](../application-gateway/application-gateway-create-url-route-portal.md) (Creación de una regla basada en ruta de acceso para una puerta de enlace de aplicaciones).

![Diagrama de nivel de web de Application Gateway](./media/traffic-manager-load-balancing-azure/web-tier-diagram.png)

1. En el grupo de recursos, vaya a la instancia de la puerta de enlace de aplicaciones que creó en la sección anterior.
2. En **Configuración**, seleccione **Backend pools** (Grupos de back-end) y seleccione **Agregar** para agregar las máquinas virtuales que desee asociar a los grupos de back-end del nivel web.
3. Escriba el nombre del grupo de back-end y todas las direcciones IP de las máquinas que residen en él. En este escenario, se conectan dos grupos de servidores de back-end de máquinas virtuales.

   !["Add backend pool" (Agregar grupo de back-end) de Application Gateway](./media/traffic-manager-load-balancing-azure/s2-appgw-add-bepool.png)

4. En **Configuración** de la puerta de enlace de aplicaciones, seleccione **Reglas** y haga clic en el botón **Path based** (Basada en ruta de acceso) para agregar una nueva regla.

   ![Botón "Basada en ruta de acceso" de las reglas de Application Gateway](./media/traffic-manager-load-balancing-azure/s2-appgw-add-pathrule.png)

5. Para configurar la regla, proporcione la información siguiente.

   Configuración básica:

   + **Name**: Nombre descriptivo de la regla a la que se puede acceder en el portal.
   + **Agente de escucha**: Agente de escucha que se usa para la regla.
   + **Grupo de back-end predeterminado**: El grupo de back-end que se usará con la regla predeterminada.
   + **Configuración de HTTP predeterminada**: La configuración de HTTP que se usará con la regla predeterminada.

   Reglas basadas en ruta de acceso:

   + **Name**: El nombre descriptivo de la regla basada en la ruta de acceso.
   + **Rutas de acceso**: La regla de ruta de acceso que se usa para reenviar el tráfico.
   + **Grupo de back-end**: El grupo de back-end que se usará con esta regla.
   + **Configuración de HTTP**: La configuración de HTTP que se usará con esta regla.

   > [!IMPORTANT]
   > Rutas de acceso: Las rutas de acceso válidas deben comenzar por "/". El carácter comodín "\*" solo se permite al final. Algunos ejemplos válidos son /xyz, /xyz\*, o /xyz/\*.

   ![Hoja "Add path-based rule" (Agregar regla basada en ruta de acceso) para Application Gateway](./media/traffic-manager-load-balancing-azure/s2-appgw-pathrule-blade.png)

### <a name="step-3-add-application-gateways-to-the-traffic-manager-endpoints"></a>Paso 3: Agregar puertas de enlace de aplicaciones a los puntos de conexión de Traffic Manager

En este escenario, Traffic Manager está conectado a las puertas de enlace de aplicaciones (como se ha configurado en los pasos anteriores) que residen en regiones diferentes. Ahora que están configuradas las puertas de enlace de aplicaciones, el paso siguiente consiste en conectarlas al perfil de Traffic Manager.

1. Abra el perfil de Traffic Manager. Para ello, examine el grupo de recursos o busque el nombre del perfil de Traffic Manager en **All Resources** (Todos los recursos).
2. En el panel izquierdo, seleccione **Puntos de conexión** y luego haga clic en **Agregar** para agregar un punto de conexión.

   ![Botón "Agregar" de puntos de conexión de Traffic Manager](./media/traffic-manager-load-balancing-azure/s3-tm-add-endpoint.png)

3. Para crear un punto de conexión, escriba la información siguiente:

   * **Tipo**: Seleccione el tipo de punto de conexión para el equilibrio de carga. En este escenario, seleccione **Punto de conexión de Azure** porque vamos a conectarlo a las instancias de puerta de enlace de aplicaciones que se configuraron anteriormente.
   * **Name**: Escriba el nombre del punto de conexión.
   * **Tipo de recurso de destino**: Seleccione **Dirección IP pública** y, en **Recurso de destino**, seleccione la dirección IP pública de la instancia de Application Gateway configurada antes.

   !["Add endpoint" (Agregar punto de conexión) en Traffic Manager](./media/traffic-manager-load-balancing-azure/s3-tm-add-endpoint-blade.png)

4. Ahora puede probar la configuración accediendo a ella con el DNS de su perfil de Traffic Manager (en este ejemplo: TrafficManagerScenario.trafficmanager.net). Puede reenviar solicitudes, activar o desactivar máquinas virtuales y servidores web creados en distintas regiones y cambiar la configuración de perfil de Traffic Manager para probar la configuración.

### <a name="step-4-create-a-load-balancer"></a>Paso 4: Creación de un equilibrador de carga

En este escenario, Load Balancer distribuye conexiones desde el nivel web hasta las bases de datos dentro de un clúster de alta disponibilidad.

Si el clúster de base de datos de alta disponibilidad usa SQL Server AlwaysOn, consulte [Configuración de uno o varios agentes de escucha de grupo de disponibilidad AlwaysOn](../azure-sql/virtual-machines/windows/availability-group-listener-powershell-configure.md) para ver las instrucciones paso a paso.

Para más información cómo configurar un equilibrador de carga interno, consulte [Creación de un equilibrador de carga interno en Azure Portal](../load-balancer/load-balancer-get-started-ilb-arm-portal.md).

1. En Azure Portal, vaya al panel izquierdo y haga clic en **Crear un recurso** > **Redes** > **Equilibrador de carga**.
2. Elija un nombre para el equilibrador de carga.
3. Establezca **Tipo** en **Interno** y elija la red virtual y la subred adecuadas para que resida el equilibrador de carga.
4. En **IP address assignment** (Asignación de direcciones IP), seleccione **Dinámica** o **Estática**.
5. En **Grupo de recursos**, elija el grupo de recursos para el equilibrador de carga.
6. En **Ubicación**, elija la región adecuada para el equilibrador de carga.
7. Haga clic en **Crear** para generar el equilibrador de carga.

#### <a name="connect-a-back-end-database-tier-to-the-load-balancer"></a>Conexión de un nivel de base de datos de back-end al equilibrador de carga

1. En el grupo de recursos, busque el equilibrador de carga que se creó en los pasos anteriores.
2. En **Configuración**, haga clic en **Backend pools** (Grupos de back-end) y en **Agregar** para agregar un grupo de back-end.

   !["Add backend pool" (Agregar grupo de back-end) en Load Balancer](./media/traffic-manager-load-balancing-azure/s4-ilb-add-bepool.png)

3. Escriba el nombre del grupo de back-end.
4. Agregue máquinas individuales o un conjunto de disponibilidad al grupo de back-end.

#### <a name="configure-a-probe"></a>Configuración de un sondeo

1. En el equilibrador de carga, en **Configuración**, seleccione **Probes** (Sondeos) y luego haga clic en **Agregar** para agregar un sondeo.

   !["Add probe" (Agregar sondeo) de Load Balancer](./media/traffic-manager-load-balancing-azure/s4-ilb-add-probe.png)

2. Escriba el nombre del sondeo.
3. Seleccione un valor en **Protocolo** para el sondeo. Para una base de datos, probablemente le convenga más un sondeo TCP que uno HTTP. Para más información sobre los sondeos de Load Balancer, consulte [Sondeos de Load Balancer](../load-balancer/load-balancer-custom-probe-overview.md).
4. A continuación, escriba un valor en **Puerto** para la base de datos que se va a usar al acceder al sondeo.
5. En **Intervalo**, especifique la frecuencia de sondeo de la aplicación.
6. En **Unhealthy threshold** (Umbral incorrecto), especifique el número de errores de sondeo continuos que deben suceder para que se considere que la máquina virtual de back-end está en mal estado.
7. Haga clic en **Aceptar** para crear el sondeo.

#### <a name="configure-the-load-balancing-rules"></a>Configuración de las reglas de equilibrio de carga

1. En **Configuración** para el equilibrador de carga, seleccione **Load balancing rules** (Reglas de equilibrio de carga) y luego haga clic en **Agregar** para crear una regla.
2. Escriba el **nombre** de la regla de equilibrio de carga.
3. Elija un valor en **Frontend IP Address** (Dirección IP de front-end) para el equilibrador de carga, así como para **Protocolo** y **Puerto**.
4. En **Backend port** (Puerto back-end), especifique el puerto que se usará en el grupo de back-end.
5. Seleccione los valores de **Backend pool** (Grupo de back-end) y en **Sondeo** creados en los pasos anteriores para aplicarles la regla.
6. En **Session persistence** (Persistencia de la sesión), seleccione cómo desea que continúen las sesiones.
7. En **Idle timeouts** (Tiempo de espera de inactividad), especifique el número de minutos antes de que se produzca un tiempo de espera de inactividad.
8. En **Floating IP** (IP flotante), seleccione **Deshabilitada** o **Habilitada**.
9. Haga clic en **Aceptar** para crear la regla.

### <a name="step-5-connect-web-tier-vms-to-the-load-balancer"></a>Paso 5: Conectar VM de nivel web al equilibrador de carga

Ahora se configuran la dirección IP y el puerto de front-end del equilibrador de carga en las aplicaciones que se ejecutan en las máquinas virtuales de nivel web para cualquier conexión de base de datos. Esta configuración es específica de la aplicación que se ejecuta en estas máquinas virtuales. Para configurar la dirección IP de destino y el puerto, consulte la documentación de la aplicación. Para buscar la dirección IP del front-end, en Azure Portal, vaya al grupo de IP de font-end en la **configuración del equilibrador de carga**.

![Panel de navegación "Frontend IP pool" (Grupo de IP de font-end) de Load Balancer](./media/traffic-manager-load-balancing-azure/s5-ilb-frontend-ippool.png)

## <a name="next-steps"></a>Pasos siguientes

* [¿Qué es Traffic Manager?](traffic-manager-overview.md)
* [Introducción a Application Gateway](../application-gateway/application-gateway-introduction.md)
* [Información general sobre Azure Load Balancer](../load-balancer/load-balancer-overview.md)
