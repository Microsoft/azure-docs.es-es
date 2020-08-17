---
title: Arquitectura de red de SAP HANA en Azure (instancias grandes) | Microsoft Docs
description: Arquitectura de red para implementar SAP HANA en Azure (instancias grandes).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 07/15/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b99e744fb949f707467286c3d79de0f4e76a49c6
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835517"
---
# <a name="sap-hana-large-instances-network-architecture"></a>Arquitectura de red de SAP HANA (instancias grandes)

La arquitectura del servicio de red de Azure es un componente clave para implementar las aplicaciones de SAP de forma correcta en HANA (instancias grandes). Por lo general, las implementaciones de SAP HANA en Azure (Instancias grandes) tienen un entorno de SAP más amplio con varias soluciones SAP distintas y diversos tamaños de base de datos, consumo de recursos de CPU y utilización de memoria. Es probable que no todos los sistemas de TI ya se encuentren en Azure. Con frecuencia, el entorno de SAP es híbrido también desde el punto de vista de DBMS y de la aplicación SAP cuando se usa una combinación de NetWeaver, S/4HANA y SAP HANA y otros DBMS. Azure ofrece distintos servicios que permiten ejecutar los diferentes sistemas DBMS, NetWeaver y S/4HANA en Azure. Azure también ofrece tecnología de red, de forma que se asemeja a un centro de datos virtual para las implementaciones de software locales.

A no ser que los sistemas de TI completos se hospeden en Azure. La funcionalidad de red de Azure se usa para conectar el mundo local con los recursos de Azure para que Azure se parezca a uno de sus centros de datos virtuales. La funcionalidad de red de Azure que se usa es la siguiente: 

- Las redes virtuales de Azure se conectan al circuito [ExpressRoute](https://azure.microsoft.com/services/expressroute/) que conecta con los recursos de red locales.
- Un circuito ExpressRoute que conecta el entorno local a Azure debería tener un ancho de banda mínimo de [1 Gbps o más](https://azure.microsoft.com/pricing/details/expressroute/). Este ancho de banda mínimo permite suficiente ancho de banda para la transferencia de datos entre los sistemas locales y los sistemas que se ejecutan en máquinas virtuales. También permite un ancho de banda adecuado para la conexión de los usuarios locales a los sistemas de Azure.
- Todos los sistemas SAP de Azure se configuran en redes virtuales para poder comunicarse entre sí.
- Active Directory y el entorno local hospedado por DNS se extienden a Azure mediante ExpressRoute desde el entorno local o se ejecutan por completo en Azure.

En el caso concreto de integrar HANA (Instancias grandes) en el tejido de red del centro de datos de Azure, también se usa la tecnología ExpressRoute.


> [!NOTE] 
> Solo se puede vincular una suscripción de Azure a un inquilino en una demarcación de HANA (Instancias grandes) de una región de Azure específica. Y al contrario, un solo inquilino con la demarcación de HANA (Instancias grandes) solo se puede vincular a una suscripción de Azure. Este requisito es coherente con otros objetos facturables en Azure.

Si se implementa SAP HANA en Azure (instancias grandes) en varias regiones de Azure diferentes, se implementará un inquilino diferente en HANA (Instancias grandes). Puede ejecutar las dos en la misma suscripción de Azure, siempre que estas instancias formen parte de la misma infraestructura de SAP. 

> [!IMPORTANT] 
> La única implementación compatible con SAP HANA en Azure (Instancias grandes) es la de Azure Resource Manager.

 

## <a name="additional-virtual-network-information"></a>Información adicional sobre la red virtual

Para conectar una red virtual a ExpressRoute, se debe crear una puerta de enlace de Azure ExpressRoute. Para más información, consulte [Acerca de las puertas de enlace de ExpressRoute para ExpressRoute](../../../expressroute/expressroute-about-virtual-network-gateways.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 

Se puede usar una puerta de enlace de Azure ExpressRoute con ExpressRoute para una infraestructura fuera de Azure o para una demarcación de instancias grandes de Azure. Puede conectar la puerta de enlace de Azure ExpressRoute hasta a cuatro circuitos ExpressRoute diferentes, siempre que esas conexiones procedan de diferentes enrutadores perimetrales empresariales de Microsoft. Para más información, consulte [Infraestructura y conectividad con SAP HANA en Azure (instancias grandes)](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). 

> [!NOTE] 
> El rendimiento máximo que se puede lograr con una puerta de enlace de ExpressRoute es de 10 Gbps mediante una conexión de ExpressRoute. La copia de archivos entre una máquina virtual que resida en una red virtual y un sistema local (como una sola secuencia de copia) no consigue el rendimiento total de las diferentes SKU de puerta de enlace. Para aprovechar el ancho de banda completo de la puerta de enlace de ExpressRoute, use varias secuencias. O bien, debe copiar archivos diferentes en secuencias paralelas de un único archivo.


## <a name="networking-architecture-for-hana-large-instance"></a>Arquitectura de redes de HANA (instancias grandes)
La arquitectura de redes de HANA (instancias grandes) se puede dividir en cuatro partes distintas:

- Redes locales y conexión de ExpressRoute a Azure. Esta parte es el dominio de los clientes y se conecta a Azure a través de ExpressRoute. Este circuito ExpressRoute lo paga usted completamente como cliente. El ancho de banda debe ser lo suficientemente grande como para controlar el tráfico de red entre los recursos locales y la región de Azure con la que se conecta. Consulte la parte inferior derecha en la ilustración siguiente.
- Servicios de red de Azure, como se explicó anteriormente, que requieren que se agreguen puertas de enlace de ExpressRoute. Esta parte es un área en la que debe buscar los diseños adecuados para los requisitos de las aplicaciones, de seguridad y de cumplimiento. Si usa HANA (instancias grandes) es otro punto a tener en cuenta en cuanto al número de redes virtuales y de SKU de puerta de enlace de Azure entre los que elegir. Observe la parte superior derecha de la ilustración.
- Conectividad de HANA (instancias grandes) a través de la tecnología de ExpressRoute en Azure. Microsoft implementa y controla esta parte. Todo lo que necesita hacer es proporcionar algunos intervalos de direcciones IP después de la implementación de los recursos de HANA (instancias grandes) que conectan el circuito ExpressRoute a las redes virtuales. Para más información, consulte [Infraestructura y conectividad con SAP HANA en Azure (instancias grandes)](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Como cliente, no se le cobra nada adicional por la conectividad entre el tejido de red del centro de datos de Azure y las unidades de HANA (Instancias grandes).
- Las redes dentro de la demarcación de HANA (Instancias grandes), lo que resulta casi transparente para usted.

![Red virtual conectada a SAP HANA en Azure (instancias grandes) y localmente](./media/hana-overview-architecture/image1-architecture.png)

El hecho de que use HANA (instancias grandes) no cambia el requisito de tener que conectar los recursos locales a Azure a través de ExpressRoute. No cambia tampoco el requisito de tener una o varias redes virtuales que ejecutan las máquinas virtuales que hospedan el nivel de aplicación que se conecta a las instancias de HANA que se hospedan en unidades de HANA (instancias grandes). 

Las diferencias en las implementaciones de SAP en Azure son:

- Las unidades de HANA (instancias grandes) del inquilino del cliente se conectan a las redes virtuales través de otro circuito ExpressRoute. Para separar las condiciones de carga, los circuitos ExpressRoute entre el entorno local y la red virtual de Azure y los circuitos entre las redes virtuales de Azure y HANA (Instancias grandes) no comparte los mismos enrutadores.
- La naturaleza del perfil de carga de trabajo entre el nivel de aplicación de SAP y HANA (instancias grandes) es diferente y tiene muchas solicitudes pequeñas y transferencias de datos como ráfagas (conjuntos de resultados) desde SAP HANA al nivel de aplicación.
- La arquitectura de aplicación de SAP es más sensible a la latencia de la red que los escenarios típicos donde los datos se intercambian entre el entorno local y Azure.
- La puerta de enlace de Azure ExpressRoute tiene al menos dos conexiones ExpressRoute. Un circuito que se ha conectado desde el entorno local y otro que se ha conectado desde HANA (Instancias grandes). Como resultado, solo queda espacio para otros dos circuitos de diferentes MSEE a los que conectarse en la puerta de enlace de ExpressRoute. Esta restricción es independiente del uso de ExpressRoute Fast Path. Todos los circuitos conectados comparten el ancho de banda máximo para los datos de entrada de la puerta de enlace de ExpressRoute.

Con la revisión 3 de las demarcaciones de HANA (grandes instancias). la latencia de red que se experimenta entre las máquinas virtuales y las unidades de HANA (instancias grandes) puede ser mayor que una latencia de red de recorrido de ida y vuelta entre máquinas virtuales. En función de la región de Azure, los valores medidos pueden superar la latencia de recorrido de ida y vuelta de 0,7 ms clasificada como por debajo del promedio en [SAP Note #1100926 - FAQ: Network performance](https://launchpad.support.sap.com/#/notes/1100926/E) (Nota de SAP 1100926: Preguntas más frecuentes sobre el rendimiento de red). Depende de la región de Azure y de la herramienta para medir la latencia de ida y vuelta de la red entre una máquina virtual de Azure y la unidad de instancias grandes de HANA; la latencia medida puede ser de hasta 2 milisegundos. Sin embargo, los clientes han implementado correctamente aplicaciones de SAP de producción basadas en SAP HANA en SAP HANA (instancias grandes). Asegúrese de probar exhaustivamente los procesos empresariales en HANA en Azure (instancias grandes). Una funcionalidad nueva, llamada ExpressRoute Fast Path, es capaz de reducir considerablemente la latencia de red entre HANA (Instancias grandes) y las máquinas virtuales de capa de aplicación de Azure (consulte a continuación). 

Con la revisión 4 de las demarcaciones de HANA (instancias grandes), se ha comprobado que la latencia de red entre las máquinas virtuales de Azure que se implementan próximas a la demarcación cumple la clasificación media o la supera, tal como se documenta en [SAP Note #1100926 - FAQ: Network Performance](https://launchpad.support.sap.com/#/notes/1100926/E) (Nota de SAP 1100926: Preguntas más frecuentes sobre el rendimiento de red) si se configura Azure ExpressRoute Fast Path (consulte a continuación). Para implementar máquinas virtuales de Azure en estrecha proximidad a las unidades de HANA (instancias grandes) de la revisión 4, debe aprovechar los [grupos de selección de ubicación de proximidad de Azure](../../linux/co-location.md). La forma en que se pueden usar los grupos de selección de ubicación de proximidad para ubicar el nivel de aplicación de SAP en el mismo centro de datos de Azure que las unidades de HANA (instancias grandes) hospedadas de la revisión 4 se describe en [Grupos de selección de ubicación de proximidad de Azure para una latencia de red óptima con aplicaciones SAP](sap-proximity-placement-scenarios.md).

Para proporcionar latencia de red determinista entre las máquinas virtuales y HANA (Instancias grandes), es fundamental la elección de la SKU de puerta de enlace de ExpressRoute. A diferencia de los patrones de tráfico entre el entorno local y las máquinas virtuales, el patrón de tráfico entre las máquinas virtuales y HANA (instancias grandes) puede generar pequeñas pero elevadas ráfagas de solicitudes y volúmenes de datos para transmitir. Para poder controlar correctamente esas ráfagas, se recomienda el uso de la SKU de puerta de enlace UltraPerformance. En el caso de la clase de SKU de HANA (Instancias grandes) de tipo II, es obligatorio el uso de la SKU de puerta de enlace UltraPerformance como puerta de enlace de ExpressRoute.

> [!IMPORTANT] 
> Dado el tráfico de red general entre la capa de base de datos y la de la aplicación de SAP, solo se admiten las SKU de puerta de enlace HighPerformance o UltraPerformance para redes virtuales para la conexión a SAP HANA en Azure (instancias grandes). Con las SKU de tipo II de HANA (Instancias grandes), solo se admite la SKU de puerta de enlace UltraPerformance como puerta de enlace de ExpressRoute. Se aplican excepciones al usar ExpressRoute Fast Path (vea a continuación).

### <a name="expressroute-fast-path"></a>ExpressRoute Fast Path
Para reducir la latencia, se introdujo y lanzó ExpressRoute Fast Path en mayo de 2019 para la conectividad específica de HANA (Instancias grandes) a redes virtuales de Azure que hospedan las máquinas virtuales de aplicaciones SAP. La principal diferencia con la solución implantada hasta el momento es que los flujos de datos entre las máquinas virtuales y HANA (Instancias grandes) ya no se enrutan a través de la puerta de enlace de ExpressRoute. Ahora, las máquinas virtuales asignadas en las subredes de la red virtual de Azure se comunican directamente con el enrutador perimetral de empresa dedicado. 

> [!IMPORTANT] 
> La funcionalidad de ExpressRoute Fast Path requiere que las subredes que ejecutan las máquinas virtuales de aplicaciones SAP estén en la misma red virtual de Azure que está conectada a HANA (Instancias grandes). Las máquinas virtuales ubicadas en redes virtuales de Azure que están emparejadas con la red virtual de Azure conectada directamente a las unidades de HANA (Instancias grandes) no se benefician de la funcionalidad ExpressRoute Fast Path. Como consecuencia de ello, en los diseños típicos de red virtual de concentrador y radio, donde los circuitos ExpressRoute se conectan con una red virtual de concentrador y se emparejan las redes virtuales que contienen la capa de aplicación de SAP (radios), la optimización de ExpressRoute Fast Path no funcionará. Además, actualmente ExpressRoute Fast Path no admite reglas de enrutamiento definidas por el usuario (UDR). Para más información, consulte [Puerta de enlace de red virtual de ExpressRoute y FastPath](../../../expressroute/expressroute-about-virtual-network-gateways.md). 


Para más información sobre cómo configurar ExpressRoute Fast Path, lea el documento [Conexión de una red virtual a instancias grandes de HANA](./hana-connect-vnet-express-route.md).    

> [!NOTE]
> Para que funcione ExpressRoute Fast Path, se requiere una puerta de enlace de ExpressRoute UltraPerformance.


## <a name="single-sap-system"></a>Único sistema SAP

La infraestructura local mostrada anteriormente se conecta a través de ExpressRoute con Azure. El circuito ExpressRoute se conecta a un enrutador perimetral de empresa de Microsoft (MSEE). Para más información, consulte [Información técnica sobre ExpressRoute](../../../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Una vez establecida la ruta, se conecta a la red troncal de Azure.

> [!NOTE] 
> Para ejecutar infraestructuras de SAP en Azure, conéctese al enrutador perimetral de empresa más cercano a la región de Azure en la infraestructura de SAP. Las demarcaciones de HANA (Instancias grandes) están conectadas a través de dispositivos enrutadores perimetrales de empresa dedicados para minimizar la latencia de red entre las máquinas virtuales de IaaS de Azure y las demarcaciones HANA (Instancias grandes).

La puerta de enlace de ExpressRoute de las VM que hospedan instancias de aplicaciones SAP está conectada a un circuito ExpressRoute que se conecta al entorno local. La misma red virtual está conectada a un enrutador perimetral de empresa independiente dedicado para la conexión con las demarcaciones de instancias grandes. Con ExpressRoute Fast Path, el flujo de datos desde HANA (instancias grandes) hasta las máquinas virtuales del nivel de aplicación de SAP ya no se enruta a través de la puerta de enlace de ExpressRoute y, de este modo, se reduce la latencia del recorrido de ida y vuelta de red.

Este sistema es un ejemplo sencillo de un único sistema SAP. El nivel de aplicación de SAP se hospeda en Azure. La base de datos de SAP HANA se ejecuta en SAP HANA en Azure (instancias grandes). Se supone que el ancho de banda de la puerta de enlace de ExpressRoute de 2 Gbps o 10 Gbps de rendimiento no representa un cuello de botella.

## <a name="multiple-sap-systems-or-large-sap-systems"></a>Varios sistemas SAP o sistemas SAP grandes

Si se implementan varios sistemas SAP o sistemas SAP grandes para la conexión a SAP HANA en Azure (Instancias grandes), el rendimiento de la puerta de enlace de ExpressRoute podría convertirse en un cuello de botella. O, si quiere aislar sistemas de producción y que no son de producción en diferentes redes virtuales de Azure. En ese caso, debe dividir los niveles de aplicación en varias redes virtuales. También puede crear una red virtual especial que se conecta a HANA (instancias grandes) para casos tales como:

- Realizar copias de seguridad directamente desde las instancias de HANA en HANA (instancias grandes) a una máquina virtual de Azure que hospeda los recursos compartidos de NFS.
- Copiar las copias de seguridad de gran tamaño u otros archivos de unidades de instancias grandes de HANA en espacio en disco administrado en Azure.

Use una red virtual independiente para hospedar las máquinas virtuales que administran el almacenamiento para la transferencia masiva de datos entre HANA (Instancias grandes) y Azure. Esta disposición evita los efectos de las transferencias de archivos o datos de gran volumen desde HANA (Instancias grandes) hasta Azure en la puerta de enlace de ExpressRoute que atiende a las máquinas virtuales que ejecutan la capa de aplicación de SAP. 

Para una arquitectura de red más escalable:

- Aproveche varias redes virtuales para un solo nivel de aplicación de SAP más grande.
- Implemente una red virtual distinta para cada sistema SAP implementado, en comparación con la combinación de estos sistemas SAP en subredes independientes en la misma red virtual.

  Una arquitectura de red más escalable para SAP HANA en Azure (Instancias grandes):

![Implementación del nivel de aplicación de SAP en varias redes virtuales](./media/hana-overview-architecture/image4-networking-architecture.png)

Las redes virtuales se deben emparejar en función de las reglas y las restricciones que quiera aplicar entre esas distintas redes virtuales que hospedan máquinas virtuales de distintos sistemas SAP. Para más información sobre el emparejamiento de redes virtuales, consulte [Emparejamiento de redes virtuales](../../../virtual-network/virtual-network-peering-overview.md).


## <a name="routing-in-azure"></a>Enrutamiento en Azure

En una implementación hay, de forma predeterminada, tres consideraciones importantes sobre el enrutamiento de red para SAP HANA en Azure (Instancias grandes):

* Solo se puede acceder a SAP HANA en Azure (Instancias grandes) mediante máquinas virtuales de Azure y la conexión ExpressRoute dedicada, no directamente desde el entorno local. El acceso directo desde local a las unidades de HANA (instancias grandes), tal como lo entrega Microsoft, no es posible inmediatamente. Las restricciones de enrutamientos transitivas son debidas a la arquitectura de red de Azure actual utilizada para SAP HANA (instancias grandes). Los clientes de administración y las aplicaciones que necesiten acceso directo, como SAP Solution Manager ejecutado de forma local, no se pueden conectar a la base de datos de SAP HANA. Para conocer las excepciones, consulte la sección "Enrutamiento directo a HANA (Instancias grandes)".

* Si tiene implementadas unidades de HANA (Instancias grandes) en dos regiones de Azure distintas para la recuperación ante desastres, se aplican las mismas restricciones del enrutamiento transitorio. En otras palabras, las direcciones IP de una unidad de HANA (Instancias grandes) en una región (por ejemplo, Oeste de EE. UU.) no se enrutaban a una unidad de HANA (Instancias grandes) implementada en otra región (por ejemplo, Este de EE. UU.). Esta restricción era independiente del uso del emparejamiento de redes de Azure entre regiones o de los circuitos ExpressRoute que conectan las unidades de HANA (Instancias grandes) con las redes virtuales. Para ver una representación gráfica, consulte la ilustración de la sección "Uso de unidades de HANA (instancias grandes) en varias regiones". Esta restricción, que se incluía con la arquitectura implementada, prohibía el uso inmediato de la replicación del sistema de HANA como funcionalidad de recuperación ante desastres. Para conocer los cambios recientes, consulte la sección "Uso de unidades de HANA (instancias grandes) en varias regiones". 

* Las unidades de SAP HANA en Azure (Instancias grandes) tienen una dirección IP asignada del intervalo de direcciones del grupo IP de servidor que ha enviado al solicitar la implementación de HANA (Instancias grandes). Para más información, consulte [Infraestructura y conectividad con SAP HANA en Azure (instancias grandes)](hana-overview-infrastructure-connectivity.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). Se puede acceder a esta dirección IP a través de las suscripciones de Azure y del circuito que conecta las redes virtuales de Azure a HANA (Instancias grandes). La dirección IP asignada fuera del intervalo de direcciones del grupo IP de servidor se asigna directamente a la unidad de hardware. Ya *no* se asigna mediante NAT, como sucedía en las primeras implementaciones de esta solución. 

### <a name="direct-routing-to-hana-large-instances"></a>Enrutamiento directo a HANA (Instancias grandes)

De forma predeterminada, el enrutamiento transitivo no funciona en estos escenarios:

* Entre unidades de instancias grandes de HANA y una implementación local.

* Entre el enrutamiento de instancias grandes de HANA que están implementadas en dos regiones diferentes.

Hay tres maneras de habilitar el enrutamiento transitivo en esos escenarios:

- Un servidor proxy inverso para enrutar los datos, desde y hacia. Por ejemplo, F5 BIG-IP y NGINX con Traffic Manager implementado en una red virtual de Azure que se conecta a HANA (Instancias grandes) y al entorno local como una solución de enrutamiento virtual para el firewall y el tráfico.
- Usar [reglas de IPTables](http://www.linuxhomenetworking.com/wiki/index.php/Quick_HOWTO_%3a_Ch14_%3a_Linux_Firewalls_Using_iptables#.Wkv6tI3rtaQ) en una máquina virtual Linux para habilitar el enrutamiento entre las ubicaciones locales y las unidades de HANA (instancias grandes) o entre unidades de HANA (instancias grandes) en regiones diferentes. La VM que ejecuta IPTables debe implementarse en la red virtual de Azure que se conecta a HANA (Instancias grandes) y al entorno local. Se debe ajustar el tamaño de la máquina virtual según corresponda, de forma que el rendimiento de red de la máquina virtual sea suficiente para el tráfico de red esperado. Para más información sobre el ancho de banda de red de VM, consulte el artículo [Tamaños de las máquinas virtuales Linux en Azure](../../sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
- Otra solución para permitir el tráfico directo entre el entorno local y las unidades de HANA (Instancias grandes) sería [Azure Firewall](https://azure.microsoft.com/services/azure-firewall/). 

Todo el tráfico de estas soluciones se enrutaría a través de una red virtual de Azure y, como tal, los dispositivos de software o los grupos de seguridad de red de Azure podrían restringirlo adicionalmente, de forma que se podría bloquear o permitir expresamente el acceso a HANA (Instancias grandes) para determinadas direcciones IP o intervalos de direcciones IP del entorno local. 

> [!NOTE]  
> Tenga en cuenta que Microsoft no proporciona la implementación y soporte técnico para las soluciones personalizadas que tienen dispositivos de red de terceros o IPTables. El proveedor del componente utilizado o el integrador es el que debe proporcionar el soporte técnico. 

#### <a name="express-route-global-reach"></a>ExpressRoute Global Reach
Microsoft presentó una nueva funcionalidad llamada [ExpressRoute Global Reach](../../../expressroute/expressroute-global-reach.md). Global Reach puede usarse con HANA (Instancias grandes) en dos escenarios:

- Permitir el acceso directo desde el entorno local hasta las unidades de HANA (Instancias grandes) implementadas en distintas regiones
- Permitir la comunicación directa entre las unidades de HANA (Instancias grandes) implementadas en distintas regiones


##### <a name="direct-access-from-on-premises"></a>Acceso directo desde el entorno local
En las regiones de Azure donde se ofrece Global Reach, puede solicitar que se habilite para el circuito de ExpressRoute que conecta la red local a la red virtual de Azure que se conecta también a las unidades de HANA (Instancias grandes). Hay algunos costos relacionados con el entorno local del circuito ExpressRoute. Para información sobre los precios, consulte el [complemento Global Reach](https://azure.microsoft.com/pricing/details/expressroute/). No hay ningún costo adicional relacionado con el circuito que conecta las unidades de HANA (Instancias grandes) a Azure. 

> [!IMPORTANT]  
> En el caso de uso de Global Reach para permitir el acceso directo entre las unidades de HANA (Instancias grandes) y los recursos locales, los datos de red y el flujo de control **no se enrutan a través de redes virtuales de Azure**, sino directamente entre los enrutadores de intercambio de empresa de Microsoft. Como resultado, las reglas NSG o ASG, o cualquier tipo de firewall, NVA o proxy que haya implementado en una red virtual de Azure, no se modifican. **Si usa ExpressRoute Global Reach para permitir el acceso directo entre el entorno local y las unidades de HANA (instancias grandes), es necesario definir restricciones y permisos para acceder a ellas en los firewalls del lado local**. 

##### <a name="connecting-hana-large-instances-in-different-azure-regions"></a>Conexión de HANA (Instancias grandes) a diferentes regiones de Azure
De igual forma, como ExpressRoute Global Reach se puede usar para conectar el entorno local a unidades de HANA (Instancias grandes), se puede emplear para conectar dos inquilinos de HANA (Instancias grandes) que estén implementados en dos regiones diferentes. El aislamiento son los circuitos ExpressRoute que usan los inquilinos de HANA (Instancias grandes) para conectarse a Azure en ambas regiones. No hay ningún cargo adicional por conectar dos inquilinos de HANA (Instancias grandes) que están implementados en dos regiones diferentes. 

> [!IMPORTANT]  
> El flujo de datos y el flujo de control del tráfico de red entre los distintos inquilinos de HANA (Instancias grandes) no se enrutará a través de redes de Azure. Como resultado, no se puede usar la funcionalidad de Azure o aplicaciones virtuales de red para aplicar restricciones de comunicación entre los dos inquilinos de HANA (Instancias grandes). 

Para más información sobre cómo habilitar ExpressRoute Global Reach, lea el documento [Conexión de una red virtual a instancias grandes de HANA](./hana-connect-vnet-express-route.md).


## <a name="internet-connectivity-of-hana-large-instance"></a>Conectividad a Internet de HANA (instancias grandes)
HANA (instancias grandes) *NO* tiene conectividad directa a Internet. Por ejemplo, esta limitación puede restringir la capacidad de registrar la imagen del sistema operativo directamente con el proveedor del sistema operativo. Debe trabajar con la herramienta de administración de suscripciones de SUSE Linux Enterprise Server local o con el administrador de suscripciones de Red Hat Enterprise Linux.

## <a name="data-encryption-between-vms-and-hana-large-instance"></a>Cifrado de datos entre las máquinas virtuales y HANA (instancias grandes)
Los datos transferidos entre HANA (instancias grandes) y las máquinas virtuales no se cifran. Sin embargo, solo para el intercambio entre el DBMS de HANA y las aplicaciones basadas en JDBC y ODBC, puede habilitar el cifrado del tráfico. Para más información, consulte [esta documentación de SAP](https://help.sap.com/viewer/102d9916bf77407ea3942fef93a47da8/1.0.11/en-US/dbd3d887bb571014bf05ca887f897b99.html).

## <a name="use-hana-large-instance-units-in-multiple-regions"></a>Uso de unidades de HANA (instancias grandes) en varias regiones

Para realizar configuraciones de recuperación ante desastres, debe tener unidades de SHANA (Instancias grandes) en varias regiones de Azure. Incluso con el uso de Azure [emparejamiento de Vnet global], el enrutamiento transitivo no funciona de forma predeterminada entre los inquilinos de HANA (Instancias grandes) de dos regiones diferentes. Sin embargo, Global Reach abre la ruta de comunicación entre las unidades de HANA (Instancias grandes) que se han aprovisionado en dos regiones diferentes. Este escenario de uso de ExpressRoute Global Reach permite:

 - La replicación del sistema HANA sin firewalls ni servidores proxy adicionales.
 - La copia de seguridad entre las unidades de HANA (Instancias grandes) en dos regiones diferentes para realizar copias o actualizaciones del sistema.


![Redes virtuales conectadas a demarcaciones de instancias grandes de Azure en diferentes regiones de Azure](./media/hana-overview-architecture/image8-multiple-regions.png)

En la ilustración se muestra cómo las distintas redes virtuales de ambas regiones están conectadas a dos circuitos ExpressRoute diferentes que se usan para conectarse a SAP HANA en Azure (Instancias grandes) en ambas regiones de Azure (líneas grises). La razón de esta dos conexiones cruzadas es proteger frente a una interrupción de los MSEE en ambos lados. Se supone que el flujo de comunicación entre las dos redes virtuales de las dos regiones de Azure diferentes debe controlarse a través del [emparejamiento global](/archive/blogs/azureedu/how-to-setup-global-vnet-peering-in-azure) de estas dos redes virtuales (línea azul de puntos). La línea roja gruesa describe la conexión de ExpressRoute Global Reach, que permite que las unidades de HANA (Instancias grandes) de los inquilinos de dos regiones diferentes se comuniquen entre sí. 

> [!IMPORTANT] 
> Si se usan varios circuitos ExpressRoute, deben utilizarse las configuraciones de anteposición de AS Path y de preferencia local de BGP para garantizar el enrutamiento adecuado del tráfico.

**Pasos siguientes**
- Consulte [SAP HANA (Large Instances) storage architecture](hana-storage-architecture.md) (Arquitectura de almacenamiento de SAP HANA [instancias grandes]).