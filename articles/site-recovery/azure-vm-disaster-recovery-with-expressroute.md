---
title: Integración de la recuperación ante desastres de máquinas virtuales de Azure con Azure ExpressRoute y Azure Site Recovery
description: Se describe cómo configurar la recuperación ante desastres en máquinas virtuales de Azure mediante Azure Site Recovery y Azure ExpressRoute.
services: site-recovery
author: mayurigupta13
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 04/08/2019
ms.author: mayg
ms.openlocfilehash: 0e1f670f2ba5ad31f29d56b2de40acd6e2bf18a9
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88654385"
---
# <a name="integrate-expressroute-with-disaster-recovery-for-azure-vms"></a>Integración de ExpressRoute con la recuperación ante desastres en máquinas virtuales de Azure


En este artículo se describe cómo integrar Azure ExpressRoute con [Azure Site Recovery](site-recovery-overview.md) cuando se establece la recuperación ante desastres en máquinas virtuales de Azure en una región de Azure secundaria.

Site Recovery permite la recuperación ante desastres de máquinas virtuales de Azure mediante la replicación de los datos de la máquina virtual de Azure en Azure.

- Si las máquinas virtuales de Azure usan [discos administrados de Azure](../virtual-machines/managed-disks-overview.md), los datos de la máquina virtual se replican en un disco administrado replicado en la región secundaria.
- Si las máquinas virtuales de Azure no usan discos administrados, los datos de la máquina virtual se replican en una cuenta de almacenamiento de Azure.
- Los puntos de conexión de replicación son públicos, pero el tráfico de replicación de las máquinas virtuales de Azure no atraviesa Internet.

ExpressRoute permite ampliar las redes locales a la nube de Microsoft Azure a través de una conexión privada que facilita un proveedor de conectividad. Si tiene configurado ExpressRoute, se integra con Site Recovery como se indica a continuación:

- **Durante la replicación entre regiones de Azure**: el tráfico de replicación para la recuperación ante desastres de máquinas virtuales de Azure solo está dentro de Azure y no se necesita ExpressRoute ni se usa para la replicación. Sin embargo, si se va a conectar desde un sitio local a las máquinas virtuales de Azure del sitio principal de Azure, hay una serie de problemas que es necesario tener en cuenta al configurar la recuperación ante desastres en esas máquinas virtuales de Azure.
- **Conmutación por error entre regiones de Azure**: cuando se produzcan interrupciones, puede conmutar por error máquinas virtuales de Azure desde el servidor principal hasta la región secundaria de Azure. Después de la conmutación por error a una región secundaria, hay una serie de pasos que se deben seguir para acceder a las máquinas virtuales de Azure en la región secundaria mediante ExpressRoute.


## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar, asegúrese de que comprende los siguientes conceptos:

- Los [circuitos ExpressRoute](../expressroute/expressroute-circuit-peerings.md)
- Los [dominios de enrutamiento](../expressroute/expressroute-circuit-peerings.md#routingdomains) de ExpressRoute
- Las [ubicaciones](../expressroute/expressroute-locations.md) de ExpressRoute.
- La [aquitectura de replicación](azure-to-azure-architecture.md) de máquinas virtuales de Azure
- Cómo [configurar la replicación](azure-to-azure-tutorial-enable-replication.md) en máquinas virtuales de Azure.
- Cómo [conmutar por error](azure-to-azure-tutorial-failover-failback.md) máquinas virtuales de Azure.


## <a name="general-recommendations"></a>Recomendaciones generales

Como procedimiento recomendado, y para garantizar objetivos de tiempo de recuperación (RTO) eficientes para la recuperación ante desastres, se recomienda realizar estos pasos al configurar Site Recovery para integrarse con ExpressRoute:

- Aprovisionar los componentes de red antes de la conmutación por error a una región secundaria:
    - Al habilitar la replicación en máquinas virtuales de Azure, Site Recovery puede implementar automáticamente recursos de red, como redes, subredes y puertas de enlace en la región de Azure de destino, según la configuración de red de origen.
    - Site Recovery no puede configurar automáticamente recursos de red, como puertas de enlace de red virtual.
    - Se recomienda aprovisionar estos recursos de red adicionales antes de la conmutación por error. Esta implementación conlleva un breve tiempo de inactividad, que puede afectar al tiempo de recuperación global si no lo ha tenido en cuenta durante su planeamiento.
- Ejecutar exploraciones periódicas de recuperación ante desastres:
    - Mediante una exploración se valida su estrategia de replicación sin ocasionar ninguna pérdida de datos ni tiempo de inactividad, y sin afectar al entorno de producción. Ayuda a evitar problemas de configuración de última hora que pueden afectar negativamente al RTO.
    - Al ejecutar una conmutación por error de prueba para la exploración, se recomienda usar una red de máquina virtual de Azure distinta, en lugar de la red predeterminada que se ha configurado al habilitar la replicación.
- Usar diferentes espacios de direcciones IP si tiene un único circuito ExpressRoute.
    - Se recomienda usar un espacio de direcciones IP diferente para la red virtual de destino. De esta forma, se evitan problemas al establecer conexiones durante las interrupciones regionales.
    - Si no puede usar un espacio de direcciones distinto, asegúrese de ejecutar la conmutación por error de prueba de exploración de recuperación ante desastres en una red de prueba aparte con diferentes direcciones IP. No puede conectar dos redes virtuales con espacios de direcciones IP superpuestos al mismo circuito ExpressRoute.

## <a name="replicate-azure-vms-when-using-expressroute"></a>Replicación de máquinas virtuales de Azure con ExpressRoute


Si quiere configurar la replicación en máquinas virtuales de Azure de un sitio principal, y se va a conectar a estas máquinas virtuales desde su sitio local a través de ExpressRoute, esto es lo que debe hacer:

1. [Habilite la replicación](azure-to-azure-tutorial-enable-replication.md) para cada máquina virtual de Azure.
2. Opcionalmente, deje que Site Recovery configure la red:
    - Al configurar y habilitar la replicación, Site Recovery configura las redes, las subredes y las subredes de puerta de enlace de la región de Azure de destino, para que coincidan con las de la región de origen. Site Recovery también se asigna entre las redes virtuales de origen y destino.
    - Si no quiere que Site Recovery realice esta acción automáticamente, cree los recursos de red de destino antes de habilitar la replicación.
3. Cree otros elementos de red:
    - Site Recovery no crea tablas de rutas, puertas de enlace de red virtual, conexiones de puerta de enlace de red virtual, emparejamiento de VNet u otros recursos y conexiones de red en la región secundaria.
    - Deberá crear estos elementos de red adicionales en la región secundaria, en cualquier momento antes de ejecutar una conmutación por error de la región primaria.
    - Puede usar [planes de recuperación](site-recovery-create-recovery-plans.md) y scripts de automatización para configurar y conectar estos recursos de red.
1. Si tiene una aplicación virtual de red (NVA) implementada para controlar el flujo del tráfico de red, tenga en cuenta que:
    - La ruta predeterminada del sistema de Azure para la replicación de máquinas virtuales de Azure es 0.0.0.0/0.
    - Normalmente, las implementaciones de aplicaciones virtuales de red también definen una ruta predeterminada (0.0.0.0/0) que fuerza el tráfico de salida de Internet a través de la aplicación virtual de red. La ruta predeterminada se usa cuando no se puede encontrar ninguna otra configuración de ruta específica.
    - Si este es el caso, la aplicación virtual de red podría sobrecargarse si todo el tráfico de replicación pasa por ella.
    - La misma limitación se aplica también cuando se usan rutas predeterminadas para enrutar todo el tráfico de la máquina virtual de Azure a implementaciones locales.
    - En este escenario, se recomienda [crear un punto de conexión de servicio de red](azure-to-azure-about-networking.md#create-network-service-endpoint-for-storage) en la red virtual para el servicio Microsoft.Storage, para que el tráfico de replicación no salga de los límites de Azure.

## <a name="replication-example"></a>Ejemplo de replicación

Normalmente, las implementaciones empresariales tienen cargas de trabajo repartidas entre varias redes virtuales de Azure, con un concentrador de conectividad central a Internet y a sitios locales. Normalmente se usa una topología de concentrador y radio en combinación con ExpressRoute.

![Del entorno local a Azure con ExpressRoute antes de la conmutación por error](./media/azure-vm-disaster-recovery-with-expressroute/site-recovery-with-expressroute-before-failover.png)

- **Región**. Las aplicaciones se implementan en la región de Este de Asia de Azure.
- **Redes virtuales de radio**. Las aplicaciones se implementan en dos redes virtuales de radio:
    - **vNet1 de origen**: 10.1.0.0/24.
    - **vNet2 de origen**: 10.2.0.0/24.
    - Cada red virtual de radio está conectada a una **vNet de concentrador**.
- **vNet de concentrador**. Hay una vNet de concentrador que es la **vNet de concentrador de origen**: 10.10.10.0/24.
  - Esta red virtual de concentrador actúa como equipo selector.
  - Todas las comunicaciones a través de las subredes pasan por este concentrador.
    - **Subredes de vNet de concentrador**. La vNet de concentrador tiene dos subredes:
    - **Subred de NVA**: 10.10.10.0/25. Esta subred contiene una aplicación virtual de red (10.10.10.10).
    - **Subred de puerta de enlace**: 10.10.10.128/25. Esta subred contiene una puerta de enlace de ExpressRoute conectada a una conexión de ExpressRoute que se enruta al sitio local a través de un dominio de enrutamiento de emparejamiento privado.
- El centro de datos local tiene una conexión de circuito ExpressRoute a través de un servidor perimetral asociado en Hong Kong.
- Todo el enrutamiento se controla mediante tablas de rutas de Azure (UDR).
- Todo el tráfico saliente entre redes virtuales, o al centro de datos local, se enruta a través de la aplicación virtual de red.

### <a name="hub-and-spoke-peering-settings"></a>Configuración del emparejamiento tipo hub-and-spoke

#### <a name="spoke-to-hub"></a>De radio al centro

**Dirección** | **Configuración** | **State**
--- | --- | ---
De radio al centro | Permitir dirección de red virtual | habilitado
De radio al centro | Permitir tráfico reenviado | habilitado
De radio al centro | Permitir tránsito de puerta de enlace | Disabled
De radio al centro | Usar puertas de enlace remotas | habilitado

 ![Configuración del emparejamiento del radio al centro](./media/azure-vm-disaster-recovery-with-expressroute/spoke-to-hub-peering-configuration.png)

#### <a name="hub-to-spoke"></a>De centro a radio

**Dirección** | **Configuración** | **State**
--- | --- | ---
De centro a radio | Permitir dirección de red virtual | habilitado
De centro a radio | Permitir tráfico reenviado | habilitado
De centro a radio | Permitir tránsito de puerta de enlace | habilitado
De centro a radio | Usar puertas de enlace remotas | Disabled

 ![Configuración del emparejamiento del centro al radio](./media/azure-vm-disaster-recovery-with-expressroute/hub-to-spoke-peering-configuration.png)

### <a name="example-steps"></a>Pasos de ejemplo

En nuestro ejemplo, al habilitar la replicación en máquinas virtuales de Azure en la red de origen, se debe producir lo siguiente:

1. [Habilita la replicación](azure-to-azure-tutorial-enable-replication.md) en una máquina virtual.
2. Site Recovery crea réplicas de redes virtuales, subredes y subredes de puerta de enlace en la región de destino.
3. Site Recovery crea asignaciones entre las redes de origen y las réplicas de las redes de destino que crea.
4. El usuario crea manualmente puertas de enlace de red virtual, conexiones de puerta de enlace de red virtual, emparejamiento de red virtual o cualquier otro recurso o conexión de red.


## <a name="fail-over-azure-vms-when-using-expressroute"></a>Conmutación por error de máquinas virtuales de Azure cuando se usa ExpressRoute

Después de conmutar por error las máquinas virtuales de Azure a la región de Azure de destino con Site Recovery, puede acceder a ellas mediante el [emparejamiento privado](../expressroute/expressroute-circuit-peerings.md#privatepeering) de ExpressRoute.

- Deberá conectar ExpressRoute a la red virtual de destino con una nueva conexión. La conexión de ExpressRoute existente no se transfiere automáticamente.
- La manera en que se configure la conexión de ExpressRoute a la red virtual de destino depende de la topología de ExpressRoute.


### <a name="access-with-two-circuits"></a>Acceso con dos circuitos

#### <a name="two-circuits-with-two-peering-locations"></a>Dos circuitos con dos ubicaciones de emparejamiento

Esta configuración ayuda a proteger los circuitos ExpressRoute frente a desastres regionales. Si su ubicación de emparejamiento principal deja de funcionar, las conexiones pueden continuar desde la otra ubicación.

- El circuito conectado al entorno de producción suele ser el principal. El circuito secundario tiene normalmente un menor ancho de banda, que se puede aumentar en caso de desastre.
- Después de la conmutación por error, puede establecer conexiones desde el circuito de ExpressRoute secundario hasta la red virtual de destino. Como alternativa, puede tener conexiones configuradas y listas en caso de desastre, con el fin de reducir el tiempo de recuperación general.
- Con conexiones simultáneas a las redes virtuales principal y de destino, asegúrese de que en el enrutamiento local solo se usen el circuito y la conexión secundarios después de la conmutación por error.
- Tras la conmutación por error, las redes virtuales de origen y destino pueden recibir nuevas direcciones IP o mantener las mismas. En ambos casos, se pueden establecer las conexiones secundarias antes de la conmutación por error.


#### <a name="two-circuits-with-single-peering-location"></a>Dos circuitos con una ubicación de emparejamiento

Esta configuración ayuda a protegerse frente a errores del circuito ExpressRoute principal, excepto si la ubicación de emparejamiento de ExpressRoute deja de funcionar, lo que afecta a ambos circuitos.

- Puede tener conexiones simultáneas del centro de datos local a la red virtual de origen con el circuito principal y a la red virtual de destino con el circuito secundario.
- Con conexiones simultáneas a las redes virtuales principal y de destino, asegúrese de que en el enrutamiento local solo se usen el circuito y la conexión secundarios después de la conmutación por error.
-   Si los circuitos se crean en la misma ubicación de emparejamiento, no podrá conectar ambos circuitos a la misma red virtual.

### <a name="access-with-a-single-circuit"></a>Acceso con un solo circuito

En esta configuración, solo hay un circuito ExpressRoute. Aunque el circuito tiene una conexión redundante en caso de que uno deje de funcionar, un único circuito no proporciona resistencia si la región de emparejamiento deja de funcionar. Observe lo siguiente:

- Puede replicar máquinas virtuales de Azure en cualquier región de Azure de la [misma ubicación geográfica](azure-to-azure-support-matrix.md#region-support). Si la región de Azure de destino no está en la misma ubicación que la de origen, deberá habilitar ExpressRoute Premium si usa un único circuito ExpressRoute. Más información sobre [ubicaciones de ExpressRoute](../expressroute/expressroute-locations.md) y [precios de ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute/).
- No podrá conectar a la vez redes virtuales de origen y destino al circuito si se usa el mismo espacio de direcciones IP en la región de destino. En este escenario:    
    -  Desconecte la conexión del lado de origen y, luego, establezca la conexión de lado de destino. Se puede generar un script con este cambio de conexión como parte de un plan de Site Recovery. Observe lo siguiente:
        - En un error regional, si la región primaria es inaccesible, la operación de desconexión podría producir error. Este error puede afectar la creación de la conexión a la región de destino.
        - Si creó la conexión en la región de destino, y la región primaria se recupera después, podría experimentar pérdida de paquetes si dos conexiones simultáneas intentan conectarse al mismo espacio de direcciones.
        - Para evitar este problema, finalice inmediatamente la conexión principal.
        - Después de la conmutación por recuperación de la máquina virtual a la región primaria, se puede establecer de nuevo la conexión principal después de desconectar la conexión secundaria.
-   Si se usa un espacio de direcciones diferente en la red virtual de destino, puede conectar simultáneamente las redes virtuales de origen y destino desde el mismo circuito ExpressRoute.


## <a name="failover-example"></a>Ejemplo de conmutación por error

En nuestro ejemplo, estamos usando la topología siguiente:

- Dos circuitos ExpressRoute en dos ubicaciones de emparejamiento diferentes.
- Conserve las direcciones IP privadas para las máquinas virtuales de Azure después de la conmutación por error.
- La región de recuperación de destino es el Sudeste de Asia de Azure.
- Se establece una conexión secundaria a un circuito ExpressRoute mediante un servidor perimetral asociado en Singapur.

En el caso de una topología sencilla que use un único circuito ExpressRoute, con la misma dirección IP después de la conmutación por error, [revise este artículo](site-recovery-retain-ip-azure-vm-failover.md#hybrid-resources-full-failover).

### <a name="example-steps"></a>Pasos de ejemplo
Para automatizar la recuperación en este ejemplo, debe hacer lo siguiente:

1. Siga los pasos para configurar la replicación.
2. [Conmute por error las máquinas virtuales de Azure](azure-to-azure-tutorial-failover-failback.md) con estos pasos adicionales durante o después de la conmutación por error.

    a. Cree la puerta de enlace de Azure ExpressRoute en la red virtual de concentrador de la región de destino. Esto es necesario para conectar la red virtual de concentrador de destino al circuito ExpressRoute.

    b. Cree la conexión desde la red virtual de concentrador de destino hasta el circuito ExpressRoute de destino.

    c. Configure los emparejamientos de redes virtuales entre el centro de la región de destino y las redes virtuales de radio. Las propiedades de emparejamiento de la región de destino serán las mismas que aquellas de la región de origen.

    d. Configure las UDR en la red virtual de centro y en las dos redes virtuales de radio.

    - Las propiedades de las UDR del lado de destino son las mismas que aquellas en el lado de origen si usan las mismas direcciones IP.
    - Con distintas direcciones IP de destino, las UDR se deben modificar en consecuencia.


Los pasos anteriores se pueden establecer mediante script como parte de un [plan de recuperación](site-recovery-create-recovery-plans.md). Según los requisitos de conectividad de la aplicación y de tiempo de recuperación, se pueden completar los pasos anteriores antes de iniciar la conmutación por error.

#### <a name="after-recovery"></a>Después de la recuperación

Después de recuperar las máquinas virtuales y finalizar la conectividad, el entorno de recuperación es el siguiente.

![Del entorno local a Azure con ExpressRoute tras la conmutación por error](./media/azure-vm-disaster-recovery-with-expressroute/site-recovery-with-expressroute-after-failover.png)



## <a name="next-steps"></a>Pasos siguientes

Más información sobre los [planes de recuperación](site-recovery-create-recovery-plans.md) para automatizar la conmutación por error de aplicaciones.
