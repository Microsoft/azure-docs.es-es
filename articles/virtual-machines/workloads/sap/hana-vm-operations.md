---
title: Configuraciones y operaciones de infraestructura de SAP HANA en Azure | Microsoft Docs
description: Guía de operaciones para los sistemas SAP HANA que se implementan en Azure Virtual Machines.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-linux
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 10/01/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: b5438132f32117e0ec48a6f985c3b9d2045a9da2
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88649693"
---
# <a name="sap-hana-infrastructure-configurations-and-operations-on-azure"></a>Configuraciones y operaciones de infraestructura de SAP HANA en Azure
En este documento se proporcionan instrucciones para configurar la infraestructura de Azure y sobre el funcionamiento de los sistemas SAP HANA que se implementaron en máquinas virtuales nativas de Azure. En el documento también se incluye información sobre la configuración de la escalabilidad horizontal de SAP HANA para la SKU de máquinas virtuales M128s. Este documento no pretende reemplazar ninguna documentación estándar de SAP, incluido el contenido siguiente:

- [Guía de administración de SAP](https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.02/330e5550b09d4f0f8b6cceb14a64cd22.html)
- [Guías de instalación de SAP](https://service.sap.com/instguides)
- [Notas de SAP](https://service.sap.com/notes)

## <a name="prerequisites"></a>Prerrequisitos
Para utilizar esta guía, necesita un conocimiento básico de los siguientes componentes de Azure:

- [Máquinas virtuales de Azure](../../linux/tutorial-manage-vm.md)
- [Redes de azure y Azure Virtual Network](../../linux/tutorial-virtual-network.md)
- [Almacenamiento de Azure](../../linux/tutorial-manage-disks.md)

Para obtener más información acerca de SAP NetWeaver y otros componentes de SAP en Azure, consulte la sección [SAP en Azure](./get-started.md) de la [documentación de Azure](../../../index.yml).

## <a name="basic-setup-considerations"></a>Consideraciones básicas sobre la configuración
En las siguientes secciones se describen las consideraciones de configuración básicas para implementar sistemas SAP HANA en máquinas virtuales de Azure.

### <a name="connect-into-azure-virtual-machines"></a>Conectarse a Azure Virtual Machines
Como se documenta en la [guía de planeamiento de Azure Virtual Machines](./planning-guide.md), existen dos métodos básicos para conectarse a máquinas virtuales de Azure:

- Conéctese a través de Internet y de los puntos de conexión públicos en una máquina virtual de Jump o en la máquina virtual que ejecuta SAP HANA.
- Conéctese a través de una [VPN](../../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md) o de Azure [ExpressRoute](https://azure.microsoft.com/services/expressroute/).

La conectividad de sitio a sitio a través de VPN o ExpressRoute es necesaria para los escenarios de producción. Este tipo de conexión también es necesario para escenarios de no producción que se incorporan en escenarios de producción cuando se usa el software SAP. La imagen siguiente muestra un ejemplo de conectividad entre sitios:

![Conectividad entre sitios](media/virtual-machines-shared-sap-planning-guide/300-vpn-s2s.png)


### <a name="choose-azure-vm-types"></a>Elegir tipos de máquina virtual de Azure
Los tipos de máquina virtual de Azure que pueden utilizarse para escenarios de producción se muestran en la [documentación de SAP para IAAS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html). En el caso de los escenarios que no son de producción, existe una variedad más amplia de tipos de máquinas virtuales de Azure disponible.

>[!NOTE]
> Para escenarios no de producción, use los tipos de máquinas virtuales que se enumeran en la [nota de SAP 1928533](https://launchpad.support.sap.com/#/notes/1928533). Para el uso de máquinas virtuales de Azure en escenarios de producción, compruebe las máquinas virtuales certificadas para SAP HANA en el documento publicado por SAP con la [lista de plataformas de IaaS certificadas](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure).

Implemente las máquinas virtuales en Azure mediante:

- Azure Portal.
- Los cmdlets de Azure PowerShell.
- La CLI de Azure.

También puede implementar una plataforma SAP HANA completamente instalada en los servicios de máquina virtual de Azure a través de la [plataforma en la nube de SAP](https://cal.sap.com/). El proceso de instalación se describe en [Implementación de SAP S/4HANA o BW/4HANA en Azure](./cal-s4h.md), o con la automatización publicada [aquí](https://github.com/AzureCAT-GSI/SAP-HANA-ARM).

>[!IMPORTANT]
> Para usar máquinas virtuales M208xx_v2, es preciso seleccionar la imagen de Linux en la galería de imágenes de máquina virtual de Azure. Para leer los detalles, consulte el artículo [Tamaños de máquina virtual optimizada para memoria](../../mv2-series.md).
> 


### <a name="storage-configuration-for-sap-hana"></a>Configuración del almacenamiento en SAP HANA
Para las configuraciones de almacenamiento y los tipos de almacenamiento que se usan con SAP HANA en Azure, lea el documento [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md)


### <a name="set-up-azure-virtual-networks"></a>Configurar Azure Virtual Network
Si tiene conectividad de sitio a sitio a Azure por medio de VPN o de ExpressRoute, debe tener como mínimo una red de Azure Virtual Network conectada a través de una puerta de enlace virtual al circuito VPN o ExpressRoute. En las implementaciones sencillas, la puerta de enlace virtual puede implementarse en una subred de la instancia de Azure Virtual Network (VNet) que también hospeda las instancias de SAP HANA. Para instalar SAP HANA, debe crear dos subredes adicionales en la instancia de Azure Virtual Network. Una subred hospeda las máquinas virtuales para ejecutar las instancias de SAP HANA. La otra subred ejecuta Jumpbox o máquinas virtuales de administración para hospedar SAP HANA Studio, otro software de administración o el software de la aplicación.

> [!IMPORTANT]
> Además de por la funcionalidad y, lo que es más importante, por motivos de rendimiento, no se puede configurar [Aplicaciones virtuales de red de Azure](https://azure.microsoft.com/solutions/network-appliances/) en la ruta de comunicación entre la aplicación de SAP y la capa DBMS de un sistema SAP basado en SAP NetWeaver, Hybris o S/4HANA. La comunicación entre la capa de la aplicación de SAP y la capa de DBMS debe ser directa. La restricción no incluye [reglas ASG ni NSG de Azure](../../../virtual-network/security-overview.md), siempre y cuando dichas reglas ASG y NSG permitan una comunicación directa. Más escenarios donde no se admiten los NVA se dan en las rutas de comunicación entre las máquinas virtuales de Azure que representan los nodos de clúster de Linux Pacemaker y los dispositivos SBD, según se describe en [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en SUSE Linux Enterprise Server para SAP Applications](./high-availability-guide-suse.md). O bien, en las rutas de comunicación establecidas entre las máquinas virtuales de Azure y Windows Server SOFS configuradas como se describe en [Agrupación de una instancia de ASCS/SCS de SAP en un clúster de conmutación por error de Windows con un recurso compartido de archivos en Azure](./sap-high-availability-guide-wsfc-file-share.md). Las aplicaciones virtuales de red en las rutas de comunicación pueden duplicar fácilmente la latencia de red entre dos socios de comunicación y restringir el rendimiento en las rutas críticas entre la capa de la aplicación de SAP y la capa de DBMS. En algunos escenarios que se han observado con los clientes, las aplicaciones virtuales de red pueden ocasionar que los clústeres Pacemaker Linux produzcan un error cuando las comunicaciones entre los nodos del clúster de Linux Pacemaker necesiten comunicarse con su dispositivo SBD mediante una NVA.  
> 

> [!IMPORTANT]
> Otro diseño que **NO** se admite es la segregación de la capa de la aplicación de SAP ni la capa de DBMS en diferentes redes virtuales de Azure que no están [emparejadas](../../../virtual-network/virtual-network-peering-overview.md) entre sí. Se recomienda separar la capa de la aplicación de SAP y la capa de DBMS con subredes dentro de una red virtual de Azure, en lugar de usar diferentes redes virtuales de Azure. Si decide no seguir la recomendación y, en su lugar, separa las dos capas en redes virtuales diferentes, las dos redes virtuales deben estar [emparejadas](../../../virtual-network/virtual-network-peering-overview.md). Tenga en cuenta que el tráfico entre dos redes virtuales de Azure [emparejadas](../../../virtual-network/virtual-network-peering-overview.md) está sujeto a costes de transferencia. Debido al intercambio de un gran volumen de datos en terabytes entre la capa de la aplicación de SAP y la capa de DBMS, pueden acumularse costos sustanciales si la capa de la aplicación de SAP y la capa de DBMS se separan entre dos redes virtuales de Azure emparejadas. 

Al instalar las máquinas virtuales para ejecutar SAP HANA, estas necesitan lo siguiente:

- Dos NIC virtuales instaladas: una para conectarse a la subred de administración y la otra para conectarse ya sea desde la red local o desde otras redes a la instancia de SAP HANA de la máquina virtual de Azure.
- Direcciones IP privadas estáticas implementadas para ambas NIC virtuales.

> [!NOTE]
> Debería asignar direcciones IP estáticas mediante Azure a NIC virtuales individuales. No debería asignar direcciones IP estáticas dentro del sistema operativo invitado a una NIC virtual. Algunos servicios de Azure como el servicio Azure Backup se basan en el hecho de que al menos el vNIC principal está establecido en DHCP y no en direcciones IP estáticas. Consulte también el documento [Solución de problemas de copia de seguridad de máquinas virtuales de Azure](../../../backup/backup-azure-vms-troubleshoot.md#networking). Si necesita asignar varias direcciones IP estáticas a una máquina virtual, tiene que asignar varias NIC virtuales a una máquina virtual.
>
>

Sin embargo, para implementaciones permanentes, deberá crear una arquitectura de red del centro de datos virtual en Azure. Esta arquitectura recomienda separar la instancia de Azure VNet Gateway que se conecta a entornos locales en una instancia independiente de Azure Virtual Network. Esta red virtual independiente debe hospedar todo el tráfico que sale a entornos locales o a Internet. Este enfoque permite implementar software para la auditoría y el registro de tráfico que entra en el centro de datos virtual de Azure en esta red virtual hub independiente. De esta forma, dispondrá de una red virtual que hospeda todo el software y las configuraciones relacionados con el tráfico de entrada y salida de la implementación de Azure.

En los artículos [Centro de datos virtual de Azure: una perspectiva de red](/azure/architecture/vdc/networking-virtual-datacenter) y [Centro de datos virtual de Azure y el plano de control empresarial](/azure/architecture/vdc/) se ofrece más información sobre el enfoque del centro de datos virtual y el diseño relacionado de Azure Virtual Network.


>[!NOTE]
>El tráfico que fluye entre una red virtual hub y otra spoke mediante [emparejamiento de redes virtuales de Azure](../../../virtual-network/virtual-network-peering-overview.md) supone [costos](https://azure.microsoft.com/pricing/details/virtual-network/) adicionales. Según estos costos, puede que deba plantearse la opción de asumir compromisos entre la ejecución de un diseño de red hub-and-spoke estricto y la ejecución de varias [puertas de enlace de Azure ExpressRoute](../../../expressroute/expressroute-about-virtual-network-gateways.md) que conecta a los "radios" para omitir el emparejamiento de redes virtuales. Sin embargo, las puertas de enlace de Azure ExpressRoute también generan [costos](https://azure.microsoft.com/pricing/details/vpn-gateway/) adicionales. También puede incurrir en costos adicionales por el software de terceros que usa para el registro, la auditoría y la supervisión del tráfico de red. En función de los costos derivados del intercambio de datos mediante el emparejamiento de redes virtuales, por una parte, y de los costos que generan las puertas de enlace de Azure ExpressRoute adicionales y las licencias de software adicionales, por otra, puede que deba plantearse la microsegmentación dentro de una red virtual con el uso de subredes como unidad de aislamiento en lugar de utilizar redes virtuales.


Para obtener información general sobre los distintos métodos para asignar direcciones IP, consulte [Tipos de direcciones IP y métodos de asignación en Azure](../../../virtual-network/public-ip-addresses.md). 

Para máquinas virtuales que ejecutan SAP HANA, debe trabajar con direcciones IP estáticas asignadas. La razón es que algunos atributos de configuración de HANA hacen referencia a direcciones IP.

Los [grupos de seguridad de red (NSG) de Azure](../../../virtual-network/virtual-network-vnet-plan-design-arm.md) se usan para dirigir el tráfico que se enruta a la instancia de SAP HANA o al Jumpbox. Los NSG y los posibles [grupos de seguridad de aplicaciones](../../../virtual-network/security-overview.md#application-security-groups) se asocian a la subred de SAP HANA y a la subred de administración.

En la siguiente imagen se muestra un resumen de un esquema de implementación aproximado para SAP HANA según una arquitectura de red virtual tipo hub-and-spoke:

![Esquema de implementación aproximado de SAP HANA](media/hana-vm-operations/hana-simple-networking-dmz.png)

Para implementar SAP HANA en Azure sin una conexión de sitio a sitio, todavía debe blindar la instancia de SAP HANA desde la red pública de Internet y ocultarla detrás de un proxy de reenvío. En este escenario básico, la implementación depende de los servicios DNS integrados en Azure para resolver los nombres de host. En una implementación más compleja en la que se usan direcciones IP de acceso público, los servicios DNS integrados en Azure son especialmente importantes. Use los grupos de seguridad de red de Azure y las [aplicaciones virtuales de red de Azure](https://azure.microsoft.com/solutions/network-appliances/) para controlar y supervisar el enrutamiento desde Internet a la arquitectura de Azure Virtual Network. En la imagen siguiente se muestra un esquema aproximado de implementación de SAP HANA sin una conexión de sitio a sitio en una arquitectura de red virtual tipo hub-and-spoke:
  
![Esquema de implementación aproximado de SAP HANA sin una conexión de sitio a sitio](media/hana-vm-operations/hana-simple-networking-dmz.png)
 

Se puede consultar otra descripción sobre cómo usar las aplicaciones virtuales de red de Azure para controlar y supervisar el acceso desde Internet sin la arquitectura de red virtual tipo hub-and-spoke en el artículo [Implementación de aplicaciones virtuales de red de alta disponibilidad](/azure/architecture/reference-architectures/dmz/nva-ha).


## <a name="configuring-azure-infrastructure-for-sap-hana-scale-out"></a>Configuración de la infraestructura de Azure para la escalabilidad horizontal de SAP HANA
Para averiguar los tipos de máquina virtual de Azure que están certificadas para la escalabilidad horizontal de OLAP o la escalabilidad horizontal de S/4HANA, consulte el [directorio de hardware de SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Una marca de verificación en la columna "Clustering" (Agrupación en clústeres) indica compatibilidad con la escalabilidad horizontal. La columna "Application Type" (Tipo de aplicación) indica si se admite la escalabilidad horizontal de OLAP o la escalabilidad horizontal de S/4HANA. Para más información sobre los nodos certificados en la escalabilidad horizontal de cada una de las máquinas virtuales, compruebe los detalles de las entradas en la SKU de máquina virtual específica que aparece en el directorio de hardware de SAP HANA.

Para conocer las versiones mínimas del sistema operativo para implementar configuraciones de escalabilidad horizontal en máquinas virtuales de Azure, compruebe los detalles de las entradas de la SKU de máquina virtual específica que aparecen en el directorio de hardware de SAP HANA. De una configuración de escalabilidad horizontal de OLAP de n nodos, un nodo funciona como nodo maestro. Los demás nodos hasta el límite de la certificación funcionan como nodos de trabajo. Los nodos en espera adicionales no cuentan en el número de nodos certificados.

>[!NOTE]
> Las implementaciones de escalabilidad horizontal de máquinas virtuales de Azure de SAP HANA con un nodo en espera solo son posibles mediante el almacenamiento [Azure NetApp Files](https://azure.microsoft.com/services/netapp/). Ningún otro almacenamiento de Azure certificado para SAP HANA permite la configuración de nodos en espera de SAP HANA.
>

En el caso de /hana/shared, también se recomienda el uso de [Azure NetApp Files](https://azure.microsoft.com/services/netapp/). 

El diseño básico habitual de un solo nodo en una configuración de escalabilidad horizontal será similar a la siguiente:

![Conceptos básicos de escalabilidad horizontal de un solo nodo](media/hana-vm-operations/scale-out-basics-anf-shared.PNG)

La configuración básica de un nodo de máquina virtual para escalabilidad horizontal de SAP HANA presenta este aspecto:

- Con **/hana/shared**, usará el servicio NFS nativo que se proporciona mediante Azure NetApp Files. 
- Todos los demás volúmenes de disco no se comparten entre los diferentes nodos y no se basan en NFS. Más adelante en este documento se proporcionan las configuraciones de instalación y los pasos para la escalabilidad horizontal de instalaciones HANA con los recursos no compartidos **/hana/data** y **/hana/log**. Para conocer el almacenamiento certificado para HANA que se puede usar, consulte el artículo [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md).


Para ajustar el tamaño de los volúmenes o discos, debe consultar el documento [Requisitos de almacenamiento de TDI para SAP HANA](https://www.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html). El tamaño necesario depende del número de nodos de trabajo. En el documento se incluye una fórmula que debe aplicar para obtener la capacidad necesaria del volumen.

El otro criterio de diseño que se muestra en los gráficos de la configuración de un único nodo para una máquina virtual de SAP HANA de escalabilidad horizontal es la red virtual, o mejor dicho la configuración de subred. SAP recomienda encarecidamente una separación del tráfico orientado al cliente y a la aplicación de las comunicaciones entre los nodos HANA. Como se muestra en los gráficos, esto se logra al tener dos VNIC diferentes conectados a la máquina virtual. Ambos VNIC están en subredes diferentes, con dos direcciones IP distintas. Puede controlar el flujo del tráfico con reglas de enrutamiento mediante grupos de seguridad de red o rutas definidas por el usuario.

Especialmente en Azure, no hay ningún medio o método para exigir la calidad de servicio y las cuotas en VNIC específicos. Como resultado, la separación de las comunicaciones entre nodos y aquellas orientadas al cliente y a la aplicación no ofrece ninguna oportunidad de dar prioridad a un flujo de tráfico con respecto a otro. En su lugar, la separación sigue siendo una medida de seguridad al blindar las comunicaciones entre nodos de las configuraciones de escalabilidad horizontal.  

>[!NOTE]
>SAP recomienda separar el tráfico de red hacia el cliente o la aplicación y el tráfico entre nodos como se describe en este documento. Por tanto, resulta muy conveniente establecer una arquitectura como la que se muestra en los últimos gráficos. Si tiene requisitos que se desvían de la recomendación, consulte también al equipo de seguridad y cumplimiento. 
>

Desde el punto de vista de las redes, la arquitectura de red mínima requerida sería:

![Conceptos básicos de escalabilidad horizontal de un solo nodo](media/hana-vm-operations/overview-scale-out-networking.png)



### <a name="installing-sap-hana-scale-out-n-azure"></a>Instalación de la escalabilidad horizontal de SAP HANA en Azure
Para instalar una configuración de SAP para escalabilidad horizontal, debe seguir estos pasos:

- Implementación de una nueva infraestructura de red virtual de Azure o adaptación de una existente
- Implementación de las nuevas máquinas virtuales mediante Premium Storage administrado de Azure, volúmenes de discos Ultra y/o volúmenes NFS basados en ANF
- - Adaptar el enrutamiento de red para asegurarse de que, por ejemplo, la comunicación entre nodos de las máquinas virtuales no se enruta mediante una [NVA](https://azure.microsoft.com/solutions/network-appliances/). 
- Instalar el nodo maestro de SAP HANA.
- Adaptar los parámetros de configuración del nodo maestro de SAP HANA.
- Continuar con la instalación de los nodos de trabajo de SAP HANA.

#### <a name="installation-of-sap-hana-in-scale-out-configuration"></a>Instalación de SAP HANA en la configuración de escalabilidad horizontal
Como se implementa la infraestructura de máquina virtual de Azure y se realizan todos los demás preparativos, deberá instalar las configuraciones de escalabilidad horizontal de SAP HANA en estos pasos:

- Instale el nodo maestro de SAP HANA según la documentación de SAP.
- En caso de que se use Azure Premium Storage o almacenamiento de discos Ultra con discos no compartidos de /hana/data y /hana/log, deberá cambiar el archivo global.ini y agregarle el parámetro "basepath_shared = no". Este parámetro habilita SAP HANA para que se ejecute la escalabilidad horizontal sin volúmenes **/hana/data** y **/hana/log** "compartidos" entre los nodos. Los detalles están documentados en la [nota de SAP n.º 2080991](https://launchpad.support.sap.com/#/notes/2080991). Si usa volúmenes NFS basados en ANF para /hana/data y /hana/log, no es necesario que realice este cambio.
- Después del cambio definitivo en el parámetro global.ini, reinicie la instancia de SAP HANA.
- Agregue nodos de trabajo adicionales. Consulte también <https://help.sap.com/viewer/6b94445c94ae495c83a19646e7c3fd56/2.0.00/en-US/0d9fe701e2214e98ad4f8721f6558c34.html>. Especifique la red interna para la comunicación entre nodos de SAP HANA durante la instalación o después con, por ejemplo, el hdblcm local. Para obtener documentación más detallada, vea también la [nota de SAP n.º 2183363](https://launchpad.support.sap.com/#/notes/2183363). 

Los detalles para configurar un sistema de escalabilidad horizontal de SAP HANA con nodos en espera en SUSE Linux se describen de forma detallada en [Implementación de un sistema de escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en SUSE Linux Enterprise Server](./sap-hana-scale-out-standby-netapp-files-suse.md). Puede encontrar documentación equivalente para Red Hat en el artículo [Implementación de un sistema de escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en Red Hat Enterprise Linux](./sap-hana-scale-out-standby-netapp-files-rhel.md). 


## <a name="sap-hana-dynamic-tiering-20-for-azure-virtual-machines"></a>Dynamic Tiering 2.0 de SAP HANA para máquinas virtuales de Azure

Además de las certificaciones de SAP HANA en máquinas virtuales de la serie M de Azure, Dynamic Tiering 2.0 de SAP HANA también se admite en Microsoft Azure (consulte el vínculo de la documentación de Dynamic Tiering de SAP HANA, más abajo). Aunque no hay ninguna diferencia en la instalación del producto o en su funcionamiento, por ejemplo, a través de SAP HANA Cockpit dentro de una máquina virtual de Azure, hay algunos elementos importantes que son obligatorios para el soporte técnico oficial en Azure. A continuación se describen estos puntos clave. En este artículo, se va a usar la abreviatura "DT 2.0" en lugar del nombre completo Dynamic Tiering 2.0.

Dynamic Tiering 2.0 de SAP HANA no se admite en SAP BW ni en S4HANA. Los casos de uso principales son ahora las aplicaciones nativas de HANA.


### <a name="overview"></a>Información general

La ilustración siguiente proporciona información general sobre la compatibilidad de DT 2.0 en Microsoft Azure. Hay un conjunto de requisitos obligatorios, que deben seguirse para cumplir con la certificación oficial:

- DT 2.0 debe instalarse en una máquina virtual de Azure dedicada. No se puede ejecutar en la misma máquina virtual donde se ejecuta SAP HANA.
- Las máquinas virtuales de DT 2.0 y de SAP HANA deben implementarse dentro de la misma red virtual de Azure.
- Las máquinas virtuales de DT 2.0 y de SAP HANA deben implementarse con las redes aceleradas de Azure habilitadas.
- El tipo de almacenamiento para las máquinas virtuales de DT 2.0 debe ser Azure Premium Storage.
- Deben conectarse varios discos de Azure a la máquina virtual de DT 2.0.
- Es necesario crear una matriz redundante de discos independientes de software o volumen seccionado (ya sea a través de lvm o de mdadm) mediante la creación de bandas en los discos de Azure.

Esto se va a explicar con más detalle en las secciones siguientes.

![Introducción a la arquitectura de DT 2.0 de SAP HANA](media/hana-vm-operations/hana-data-tiering.png)



### <a name="dedicated-azure-vm-for-sap-hana-dt-20"></a>Máquina virtual de Azure dedicada para DT 2.0 de SAP HANA

En Azure IaaS, DT 2.0 solo se admite en una máquina virtual dedicada. No se permite ejecutar DT 2.0 en la misma máquina virtual de Azure donde se está ejecutando la instancia de HANA. Inicialmente, se pueden usar dos tipos de máquina virtual para ejecutar DT 2.0 de SAP HANA:

- M64-32ms 
- E32sv3 

Vea la descripción del tipo de máquina virtual [aquí](../../sizes-memory.md).

Dada la idea básica de DT 2.0, que es la descarga de datos "semiactivos" con el fin de ahorrar costos, tiene sentido usar los tamaños de máquina virtual correspondientes. Sin embargo, no hay ninguna regla estricta referente a las combinaciones posibles. Depende de la carga de trabajo específica del cliente.

Las configuraciones recomendadas serían:

| Tipo de máquina virtual de SAP HANA | Tipo de máquina virtual de DT 2.0 |
| --- | --- | 
| M128ms | M64-32ms |
| M128s | M64-32ms |
| M64ms | E32sv3 |
| M64s | E32sv3 |


Son posibles todas las combinaciones de máquinas virtuales de la serie M certificadas para SAP HANA con máquinas virtuales admitidas de DT 2.0 (M64-32ms, E32sv3).


### <a name="azure-networking-and-sap-hana-dt-20"></a>Redes de Azure y DT 2.0 de SAP HANA

Instalar DT 2.0 en una máquina virtual dedicada requiere un rendimiento de la red entre las máquinas virtuales de DT 2.0 y de SAP HANA de 10 Gb, como mínimo. Por lo tanto, es imprescindible colocar todas las máquinas virtuales dentro de la misma red virtual de Azure y habilitar las redes aceleradas de Azure.

Consulte información adicional sobre las redes aceleradas de Azure [aquí](../../../virtual-network/create-vm-accelerated-networking-cli.md).

### <a name="vm-storage-for-sap-hana-dt-20"></a>Almacenamiento de máquinas virtuales para DT 2.0 de SAP HANA

Según las instrucciones de procedimientos recomendados de DT 2.0, el rendimiento de E/S de disco debe ser de 50 MB/s por núcleo físico, como mínimo. Al observar la especificación de los dos tipos de máquina virtual de Azure, que son compatibles con DT 2.0, el límite máximo de rendimiento de E/S de disco para la máquina virtual es:

- E32sv3:   768 MB/s (no almacenado en caché), lo que significa una proporción de 48 MB/s por núcleo físico
- M64-32 ms:  1000 MB/s (no almacenado en caché), lo que significa una proporción de 62,5 MB/s por núcleo físico

Es necesario conectar varios discos de Azure a la máquina virtual de DT 2.0 y crear una matriz redundante de discos independientes de software (seccionado) en el nivel del sistema operativo para lograr el límite máximo de rendimiento de disco por máquina virtual. Un único disco de Azure no puede proporcionar el rendimiento para alcanzar el límite máximo para las máquinas virtuales en este sentido. Se requiere Azure Premium Storage para ejecutar DT 2.0. 

- [Aquí](../../disks-types.md) puede encontrar detalles sobre los tipos de disco de Azure disponibles.
- [Aquí](../../linux/configure-raid.md) puede encontrar detalles sobre la creación de una matriz redundante de discos independientes de software a través de mdadm.
- Los detalles sobre la configuración de LVM para crear un volumen seccionado con el máximo rendimiento se encuentran [aquí](../../linux/configure-lvm.md).

Según los requisitos de tamaño, existen diferentes opciones para alcanzar el máximo rendimiento de una máquina virtual. Estas son las configuraciones de disco para volúmenes de datos posibles para cada tipo de máquina virtual de DT 2.0 a fin de alcanzar el límite superior de rendimiento para las máquinas virtuales. La máquina virtual E32sv3 debe considerarse como un nivel de entrada para las cargas de trabajo menores. En caso de que resultara no ser suficientemente rápida, podría ser necesario cambiar el tamaño de la máquina virtual a M64-32ms estándar.
Como la máquina virtual M64-32ms estándar tiene mucha memoria, la carga de E/S podría no alcanzar el límite, especialmente para cargas de trabajo con muchas lecturas. Por lo tanto, podrían ser suficientes menos discos en el conjunto seccionado, según la carga de trabajo específica del cliente. Sin embargo, por seguridad, se eligieron las configuraciones de disco siguientes a fin de garantizar el máximo rendimiento:


| SKU de la máquina virtual | Configuración de disco 1 | Configuración de disco 2 | Configuración de disco 3 | Configuración de disco 4 | Configuración de disco 5 | 
| ---- | ---- | ---- | ---- | ---- | ---- | 
| M64-32ms | 4 x P50 -> 16 TB | 4 x P40 -> 8 TB | 5 x P30 -> 5 TB | 7 x P20 -> 3,5 TB | 8 x P15 -> 2 TB | 
| E32sv3 | 3 x P50 -> 12 TB | 3 x P40 -> 6 TB | 4 x P30 -> 4 TB | 5 x P20 -> 2,5 TB | 6 x P15 -> 1,5 TB | 


Especialmente en el caso de que la carga de trabajo suponga muchas lecturas, podría aumentar el rendimiento de la E/S si se activa la memoria caché de host de Azure "de solo lectura", como se recomienda para los volúmenes de datos de software de base de datos. Sin embargo, la memoria caché del disco host de Azure del registro de transacciones debe ser "none" (ninguna). 

En relación con el tamaño del volumen de registro, el punto de partida recomendado es el valor heurístico del 15 % del tamaño de datos. La creación del volumen de registro puede llevarse a cabo con diferentes tipos de disco de Azure, según los requisitos de costo y rendimiento. Para el volumen de registro, se requiere una capacidad de proceso de E/S alta.  En el caso de usar el tipo de máquina virtual M64-32ms, se recomienda habilitar el [Acelerador de escritura](../../how-to-enable-write-accelerator.md). Este Acelerador de escritura de Azure proporciona la latencia de escritura en disco óptima para el registro de transacciones (solo disponible para la serie M). Sin embargo, hay algunos elementos que deben tenerse en cuenta, como el número máximo de discos por tipo de máquina virtual. Puede encontrar detalles sobre el Acelerador de escritura [aquí](../../how-to-enable-write-accelerator.md).


Estos son algunos ejemplos sobre cómo cambiar el tamaño del volumen de registro:

| tipo de disco y el tamaño del volumen de datos | configuración de tipo de disco y volumen de registro 1 | configuración de tipo de disco y volumen de registro 2 |
| --- | --- | --- |
| 4 x P50 -> 16 TB | 5 x P20 -> 2,5 TB | 3 x P30 -> 3 TB |
| 6 x P15 -> 1,5 TB | 4 x P6 -> 256 GB | 1 x P15 -> 256 GB |


Al igual que para el escalado horizontal de SAP HANA, el directorio /hana/shared tiene que compartirse entre la máquina virtual de SAP HANA y la máquina virtual de DT 2.0. Se recomienda la misma arquitectura que para la escalabilidad horizontal de SAP HANA con las máquinas virtuales, que actúan como un servidor NFS de alta disponibilidad. Con el fin de proporcionar un volumen compartido de copia de seguridad, se puede usar un diseño idéntico. Pero es decisión del cliente si se requiere una elevada disponibilidad o si basta con usar una máquina virtual dedicada con suficiente capacidad de almacenamiento para que actúe como un servidor de copia de seguridad.



### <a name="links-to-dt-20-documentation"></a>Vínculos a la documentación de DT 2.0 

- [SAP HANA Dynamic Tiering installation and update guide](https://help.sap.com/viewer/88f82e0d010e4da1bc8963f18346f46e/2.0.03/en-US) (Guía de instalación y actualización de Dynamic Tiering de SAP HANA)
- [SAP HANA Dynamic Tiering tutorials and resources](https://help.sap.com/viewer/fb9c3779f9d1412b8de6dd0788fa167b/2.0.03/en-US) (Tutoriales y recursos de Dynamic Tiering de SAP HANA)
- [SAP HANA Dynamic Tiering PoC](https://blogs.sap.com/2017/12/08/sap-hana-dynamic-tiering-delivering-on-low-tco-with-impressive-performance/) (Poc de Dynamic Tiering de SAP HANA)
- [SAP HANA 2.0 SPS 02 dynamic tiering enhancements](https://blogs.sap.com/2017/07/31/sap-hana-2.0-sps-02-dynamic-tiering-enhancements/) (Mejoras de Dynamic Tiering de SAP HANA 2.0 SPS 02)




## <a name="operations-for-deploying-sap-hana-on-azure-vms"></a>Operaciones de implementación de SAP HANA en máquinas virtuales de Azure
En las siguientes secciones se describen algunas de las operaciones relacionadas con la implementación de sistemas SAP HANA en máquinas virtuales de Azure.

### <a name="back-up-and-restore-operations-on-azure-vms"></a>Operaciones de copia de seguridad y restauración en máquinas virtuales Azure
En los documentos siguientes se describe cómo realizar la copia de seguridad y la restauración de una implementación de SAP HANA:

- [SAP HANA: información general sobre copias de seguridad](./sap-hana-backup-guide.md)
- [Azure Backup de SAP HANA en el nivel de archivo](./sap-hana-backup-file-level.md)
- [Copia de seguridad de SAP HANA basada en instantáneas de almacenamiento](./sap-hana-backup-guide.md)


### <a name="start-and-restart-vms-that-contain-sap-hana"></a>Iniciar y reiniciar máquinas virtuales que contienen SAP HANA
Una característica fundamental de la nube pública de Azure es que solo se le cobra por los minutos de computación. Por ejemplo, cuando se apaga una máquina virtual que ejecuta SAP HANA, se le facturan solo los costos de almacenamiento durante ese tiempo. Otra característica está disponible cuando se especifican las direcciones IP estáticas de las máquinas virtuales de la implementación inicial. Cuando se reinicia una máquina virtual con SAP HANA, lo hace con sus direcciones IP anteriores. 


### <a name="use-saprouter-for-sap-remote-support"></a>Usar SAPRouter para la compatibilidad remota de SAP
Si tiene una conexión de sitio a sitio entre sus ubicaciones locales y Azure, y ejecuta componentes de SAP, es probable que ya está ejecutando SAProuter. En este caso, realice las siguientes acciones para obtener compatibilidad remota:

- Mantener la dirección IP estática y privada de la máquina virtual que hospeda SAP HANA en la configuración de SAProuter.
- Configurar el NSG de la subred que hospeda la máquina virtual de HANA para permitir el tráfico a través del puerto TCP/IP 3299.

Si se está conectando a Azure a través de Internet y no tiene un enrutador SAP para la máquina virtual con SAP HANA, tiene que instalar el componente. Instale SAProuter en una máquina virtual independiente en la subred de administración. La imagen siguiente muestra un esquema aproximado de implementación de SAP HANA sin una conexión de sitio a sitio y con SAProuter:

![Esquema de implementación aproximado para SAP HANA sin conexión de sitio a sitio y SAPRouter](media/hana-vm-operations/hana-simple-networking-saprouter.png)

Asegúrese de instalar SAPRouter en otra máquina virtual y no en la máquina virtual de JumpBox. La máquina virtual independiente debe tener una dirección IP estática. Para conectar la instancia de SAProuter a la instancia de SAProuter hospedada en SAP, póngase en contacto con SAP para obtener una dirección IP. (La instancia de SAProuter hospedada en SAP es el equivalente de la instancia de SAProuter que se instala en la máquina virtual). Utilice la dirección IP de SAP para configurar la instancia de SAProuter. En la configuración, el único puerto necesario es el puerto TCP 3299.

Para más información sobre cómo configurar y mantener conexiones de compatibilidad remota a través de SAPRouter, consulte la [documentación de SAP](https://support.sap.com/en/tools/connectivity-tools/remote-support.html).

### <a name="high-availability-with-sap-hana-on-azure-native-vms"></a>Alta disponibilidad con SAP HANA en máquinas virtuales nativas de Azure
Si ejecuta SUSE Linux Enterprise Server o Red Hat, puede establecer un clúster Pacemaker con dispositivos STONITH. Puede usar los dispositivos para establecer una configuración de SAP HANA que utilice la replicación sincrónica con conmutación por error automática y replicación del sistema de HANA. Para más información, consulte la sección Pasos siguientes.

## <a name="next-steps"></a>Pasos siguientes
Familiarícese con los artículos que se muestran.
- [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](./hana-vm-operations-storage.md)
- [Implementación de un sistema de escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en SUSE Linux Enterprise Server](./sap-hana-scale-out-standby-netapp-files-suse.md)
- [Implementación de un sistema de escalabilidad horizontal de SAP HANA con nodo en espera en VM de Azure mediante Azure NetApp Files en Red Hat Enterprise Linux](./sap-hana-scale-out-standby-netapp-files-rhel.md)
- [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en SUSE Linux Enterprise Server](./sap-hana-high-availability.md)
- [Alta disponibilidad de SAP HANA en máquinas virtuales de Azure en Red Hat Enterprise Linux](./sap-hana-high-availability-rhel.md)

 
